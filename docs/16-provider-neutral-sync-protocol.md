# NamiShelf — Provider-Neutral Synchronization Protocol

**Status:** First-release requirement; provider-neutral architecture owner-approved  
**Date:** 2026-11-03

## Goals

Define a transport-independent sync domain. GitHub may be an adapter, but Git repository layout, Git objects, branches, and GitHub token semantics must not leak into the domain contract. Local reading and library operations remain fully usable offline.

## Logical components

1. **Domain model:** canonical entities and operations; no provider SDK or transport dependencies.
2. **Local journal:** durable operation IDs, entity keys, versions, pending/acknowledged state.
3. **Sync engine:** batching, retry, idempotency, reconciliation, and conflict handling.
4. **Envelope:** schema version, payload type, integrity metadata, mandatory end-to-end encryption (E2EE).
5. **Provider adapter:** remote read/write/list and declared consistency/capability limits.
6. **Credential service:** OS secure storage; inaccessible to extensions.
7. **UI control plane:** sync status, pause, disconnect, device management, conflict resolution.

## Candidate synchronized entities

| Entity | Candidate scope | Important exclusions |
|---|---|---|
| Library entry | Add/remove/favorite and stable source identity | Local absolute paths |
| Chapter state | Read/unread, bookmark, progress | Downloaded image bytes |
| Categories | Names, ordering, membership | Device-specific presentation |
| Followed authors | Add/remove and normalized identity | Source credentials |
| History | Excluded from release-1 core scope | All history events |
| Tracking links | Excluded from release-1 core scope | Tracker associations and credentials |
| Preferences | Excluded from release-1 core scope | General preferences, window geometry, cache, local paths |

Release-1 core scope is limited to library entries, chapter read/progress state, categories, and followed authors. History, tracking associations, general preferences, extensions, downloaded content, caches, cookies, passwords, and global credentials are excluded.

## Operation envelope

Each operation should carry protocol/schema version, stable operation ID, entity type/key, operation kind, device/actor ID, causal version or logical clock, diagnostic timestamp, optional parent version, and integrity metadata. Retries must be idempotent. Wall-clock timestamps alone must not decide conflicts.

## Conflict semantics

Do not use generic last-write-wins for every entity. Read progress may preserve maximum progress while distinguishing intentional rewind. Favorites/follows need explicit add/remove semantics. Category rename/reorder conflicts may require user resolution. Deletes need tombstones and retention; absence from a partial listing is not deletion. These are proposals, not finalized product semantics.

## Provider capability contract

Adapters declare support for durable object storage, atomic replacement, conditional writes, listing/pagination, ETags/versions, maximum object size, auth refresh/revocation, server-side encryption, self-hosting, quotas, and retry hints. The engine must account for missing capabilities rather than pretending providers offer equivalent consistency.

## Security requirements

- Store provider credentials only in OS secure storage; no plaintext fallback.
- Never put secrets in logs, diagnostics, URLs, process arguments, extensions, or ordinary backups.
- Keep extension processes away from sync journal, credentials, account tokens, and raw remote payloads.
- Preserve TLS validation.
- Treat transport encryption and end-to-end payload encryption as separate decisions.
- E2EE is mandatory. Specify key generation, device enrollment, recovery, rotation, revocation, and lost-device behavior before implementation. Providers must receive encrypted payloads only.
- Minimize synchronized personal metadata and expose the data scope to users.

## Failure and conformance requirements

Handle offline devices, partial batches, duplicate delivery, rate limits, revoked credentials, corrupted objects, schema mismatch, clock skew, account deletion, and lost keys. Sync failure must not block local reading or corrupt local state. Tests must cover idempotent replay, duplicate/out-of-order operations, concurrent edits, tombstone retention, stale-device reconnect, conditional-write races, partial retry, schema migration, corrupt payloads, credential revocation, and secret-leak checks.

## Release scope and remaining decisions

Synchronization is required in the first public release. Owner decisions (2026-11-03): GitHub is the first provider adapter; no mandatory NamiShelf-hosted service; self-hosting is required in release 1; E2EE is mandatory; core scope is library entries, chapter read/progress state, categories, and followed authors. A GitHub adapter alone cannot satisfy self-hosting, so a second self-hosted adapter is required unless the owner revises that constraint. Device enrollment, recovery, cryptographic profile, and final conflict rules remain open.
