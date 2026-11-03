# NamiShelf — Provider-Neutral Sync Domain and Protocol Design

**Status:** Engineering design proposal constrained by ADR-002, ADR-006 and ADR-007. E2EE primitive selection remains blocked on independent cryptographic review.

## 1. Owner-approved constraints

- Provider-neutral NamiShelf sync protocol.
- No mandatory NamiShelf-hosted service.
- GitHub first adapter; WebDAV second adapter and required for release-1 self-hosting.
- Mandatory E2EE for every synced user payload.
- Core scope only: library entries, chapter read/progress state, categories, followed authors.
- Recovery through both a user-held recovery key and authenticated transfer from an authorized device.
- Device revocation rotates keys for future data.
- Provider credentials and encryption keys remain separate; extensions never receive either.

## 2. Layering

1. Domain model: stable NamiShelf entities and operations.
2. Sync engine: journal, causal metadata, conflict detection, tombstones, schema migration and E2EE envelope.
3. Provider-neutral object store contract: read, create, conditional replace, list, delete, capability declaration.
4. Provider adapters: GitHub and WebDAV; transport-specific authentication and concurrency behavior.
5. OS credential backend: provider tokens and local key wrapping material, isolated from sync payloads.

Neither Git object IDs nor WebDAV paths become domain identities.

## 3. Canonical release-1 entities

| Entity | Canonical fields (conceptual) | Exclusions |
|---|---|---|
| LibraryEntry | stable entity ID, source namespace, source-native URL/key, title/display metadata, favorite state, created/updated metadata | credentials, cached cover bytes |
| ChapterState | library entity ID, chapter source key, read/bookmark/progress state, last-read timestamp | chapter image bytes/downloads |
| Category | stable category ID, name, ordering, membership references | UI-only local layout |
| FollowedAuthor | stable author ID, source namespace, source-native key, display name | tracking identity and credentials |

The exact schema must be versioned and migration-tested. Source IDs are namespaced by source identity; never assume numeric Mihon source IDs are globally stable.

## 4. Operation and object model

Recommended model:
- Every local mutation creates a unique operation ID and records entity ID, schema version, device ID, monotonic logical clock, and causal parent/version metadata.
- Operations are immutable once encrypted and uploaded.
- Provider object names are random opaque identifiers or content-addressed ciphertext identifiers; do not encode titles, URLs, or personal data in object paths.
- A separately encrypted account manifest references the current operation/snapshot set.
- Upload immutable objects first; publish/update the manifest only after object verification.
- Use provider conditional writes/ETags only as transport concurrency controls, not as cryptographic freshness proof.
- Tombstones are retained long enough to prevent deleted entities reappearing on stale devices; compaction requires acknowledgement policy and recovery from long-offline devices.

This design avoids treating provider storage as a transactional database. Exact journal compaction and snapshot policy remain implementation-level design work.

## 5. Conflict policy requirements

Conflicts must be explicit, deterministic, and tested across at least three devices. Do not silently use wall-clock timestamps as the sole conflict authority.

- Preserve concurrent alternatives when automatic merge is unsafe.
- Field-level merge is permitted only for fields with defined semantics.
- Read progress may use a monotonic merge only where the user has not issued an explicit reset/mark-unread operation that causally supersedes it.
- Category ordering and renames need stable tie-break and conflict representation.
- Favorite/library removal uses tombstones; stale updates must not resurrect a tombstoned entry without a causally later explicit re-add.
- Followed-author changes follow the same add/remove semantics.
- User-visible conflict resolution must be idempotent and itself become a sync operation.

The exact causal model (per-device vector clock, dotted version vector, or another reviewed model) and field-by-field merge rules must be fixed before sync implementation.

## 6. E2EE envelope requirements

Each encrypted object authenticates protocol version, account scope, entity/operation type, schema version, object ID, key version, and causal metadata as associated data. Use an independently reviewed AEAD/KDF/key hierarchy; no custom cryptography.

- Provider sees ciphertext and minimal unavoidable object metadata only.
- Nonces obey the selected AEAD uniqueness rules.
- Detect replay, rollback, object substitution, stale key versions and malformed envelopes.
- Provider ETags/Git commit history are not cryptographic anti-replay proofs.
- No plaintext titles, source URLs, user IDs or entity payloads in object paths, filenames, logs or provider metadata.
- Key material is not stored in ordinary backup or extension IPC.

## 7. Provider adapter contract

Every adapter declares auth method, scopes, token refresh/revocation/logout, TLS/certificate validation, object create/read/list/conditional update/delete, consistency, pagination, maximum object size, atomicity, rate limits, retries, quotas, retention/versioning, interruption/resume, credential storage requirements, and deletion/export limitations.

### GitHub adapter

Use a dedicated private repository or equivalent owner-selected private storage target. Use least-privilege authorization; keep repository layout behind the adapter. Do not expose GitHub tokens to extensions or sync domain objects.

### WebDAV adapter

Use HTTPS with certificate validation. The adapter must not assume LOCK or atomic MOVE works consistently. Require conditional create/update or use an immutable-object plus compare-and-swap manifest strategy supported by the server. Servers without a safe concurrency primitive are unsupported for multi-writer sync. Validate ETag, conditional PUT, listing, deletion, size, and interrupted transfers.

## 8. Device lifecycle

- New device requires explicit authorization and either recovery key or authenticated transfer from an existing trusted device.
- New device gets a distinct device identity and local key wrapping material.
- Revocation updates authenticated account state and rotates the key version for future payloads.
- Stale devices must not publish manifests using revoked key versions.
- Historical ciphertext policy is a separate owner decision; revocation cannot erase plaintext/ciphertext already copied by a device.
- Loss of all trusted devices and recovery material means remote ciphertext is unrecoverable; explain this before enrollment.

## 9. Sync state machine

DISCONNECTED → AUTH_REQUIRED → DEVICE_ENROLLMENT → READY → FETCHING → MERGING → UPLOADING → READY; errors return to READY with retry/backoff or AUTH_REQUIRED. Revocation enters KEY_ROTATION_REQUIRED and blocks uploads under old key versions.

Sync must be pauseable and cancellable. Offline reading/library operation remains available. Never block local reads on provider availability.

## 10. Security and conformance tests

Three-device concurrent edits; simultaneous first sync; stale device reconnect; duplicate/replayed operation; manifest rollback; object substitution; ETag race; WebDAV server lacking conditional write; interrupted upload/download; token expiry/revocation; device enrollment/recovery; device revocation and key rotation; corrupted ciphertext; schema migration; tombstone compaction; provider outage; clock skew; no plaintext metadata leakage.

No cryptographic library or primitive is selected by this design. No provider was contacted, no credentials used, and no sync implementation or test was run.
