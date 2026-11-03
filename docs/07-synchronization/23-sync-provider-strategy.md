# NamiShelf — First-Release Sync Provider Strategy

**Status:** Provider choices and E2EE requirement owner-approved; adapter conformance and cryptographic engineering remain open
**Date:** 2026-11-03
**Basis:** ADR-002, ADR-004, and the static candidate sync assessment

## 1. Fixed requirements

- Synchronization is required in the first public release.
- The sync domain and operation protocol remain provider-neutral.
- GitHub may be an adapter, but GitHub repositories, Git objects, and GitHub token semantics must not define the protocol.
- Local reading/library use must remain fully functional offline.
- Credentials stay in the OS secure credential service; no plaintext fallback.
- Extensions cannot access sync credentials, journal, or raw remote payloads.
- Downloads, extension binaries, cookies, passwords, and tracker credentials are excluded by default.
- Provider capabilities and consistency limits must be explicit.

## 2. Provider models to compare

| Model | Advantages | Costs and risks |
|---|---|---|
| **A. GitHub private repository adapter** | Reuses the candidate's existing transport work; no NamiShelf-operated sync service; users may already have accounts | GitHub account/API dependency, token lifecycle and quotas; repository/Git semantics must stay isolated; not self-hosted; unsuitable as the only provider contract |
| **B. WebDAV adapter** | Broad server availability; self-hosting possible; ordinary HTTPS infrastructure; no Git dependency | Server behavior and locking/conditional-write support vary; concurrency semantics must be implemented by NamiShelf; provider-specific testing burden |
| **C. S3-compatible object storage adapter** | Scalable object storage; multiple vendors and self-hosted implementations; object versioning/conditional operations may be available | Credentials and configuration complexity; API compatibility varies; object stores are not transactional databases; consistency and conditional-write capabilities must be declared |
| **D. NamiShelf sync service/API** | Full control over protocol, accounts, quotas, conflict coordination, and future clients | Requires operating infrastructure, authentication, privacy policy, monitoring, backups, abuse controls, cost management, and long-term service commitment |
| **E. User-selected local/network folder** | Simple manual transfer and user-controlled storage; useful for offline migration | Not reliable multi-device sync by itself; concurrent writes and remote filesystem semantics are hazardous; should be treated as export/import or a later advanced adapter |

## 3. Architectural distinction

Keep three layers separate:

1. **Sync protocol:** canonical entities, operation IDs, causal metadata, idempotency, conflict semantics, encryption envelope, and schema migration.
2. **Provider adapter:** authentication, object read/write/list, conditional operations, pagination, limits, retries, and capability declarations.
3. **Account/service policy:** who hosts the data, who pays, account recovery, quotas, deletion, retention, and support.

Choosing a provider must not force changes to the canonical sync entity model.

## 4. Recommendation for release 1

Do not build a hosted NamiShelf service as a hidden prerequisite for the first release unless the owner explicitly accepts the operational commitment. Do not make GitHub the sole protocol or storage assumption.

Recommended staged approach:
- Define and test the provider-neutral sync protocol first.
- Implement the owner-selected GitHub and WebDAV adapters after the provider-neutral contract and security design are ready.
- Keep the adapter contract capable of supporting at least one alternative provider without changing domain entities.
- If minimizing service operations is the priority, compare GitHub adapter and user-managed WebDAV as the first two concrete implementations.
- If broad infrastructure flexibility is the priority, evaluate S3-compatible object storage but do not assume it provides database-like transactions.
- Treat local/network folder sync as manual export/import until locking, atomicity, and concurrent-writer semantics are demonstrated.

## 5. Provider capability checklist

Every adapter must explicitly declare and test:
- authentication type, token scope, refresh, revocation, and logout;
- TLS requirements and certificate validation;
- read-after-write behavior and consistency guarantees;
- conditional writes/ETags/version preconditions;
- atomic object replacement and maximum object size;
- listing, pagination, object deletion, and tombstone retention;
- rate limits, retry hints, quotas, and storage cost visibility;
- account deletion/export and data retention behavior;
- self-hosting support and provider lock-in;
- credential storage behavior in DEB, AppImage, and Flatpak;
- recovery after interrupted upload/download and stale-device reconnect.

## 6. Security and privacy baseline

- Transport security is mandatory; never disable certificate validation.
- End-to-end encryption is mandatory for all release-1 synchronized user payloads. Provider-side encryption is not equivalent to E2EE.
- Never send credentials, cookies, tracker tokens, extension binaries, or downloaded chapter content by default.
- Minimize metadata and document exactly which library/reading information leaves the device.
- Keep access tokens out of URLs, logs, crash reports, process arguments, and extension IPC.
- Support disconnect/revoke and explain what remote data remains after disconnect.
- Implement the owner-approved device enrollment/recovery paths, future-data key rotation and historical ciphertext re-encryption under ADR-007/024; exact wire protocol and cryptographic construction require independent review before implementation.

## 7. Owner-approved decisions and remaining design

1. **Resolved — hosting:** no mandatory NamiShelf-hosted service in release 1; users use their own provider accounts/infrastructure.
2. **Resolved — first adapter:** GitHub. Its API, repository layout, and Git object model remain behind the provider adapter.
3. **Resolved — self-hosting:** required in release 1. WebDAV is the approved second adapter alongside GitHub. Its server compatibility and conditional-write guarantees must be validated.
4. **Resolved — E2EE and revocation:** mandatory for all synced user payloads. Recovery uses both a user-held recovery key and authenticated transfer from an authorized device. Device revocation rotates future-data keys and triggers historical re-encryption for remaining devices plus best-effort provider deletion (ADR-024). Cryptographic construction, exact key lifecycle protocol and metadata minimization remain engineering/security gates.
5. **Resolved — scope:** core only: library entries, chapter read/progress state, categories, and followed authors. History, tracking, preferences, downloads, and extension data are excluded from release 1.

## 8. Architecture implications and explicit non-goals

A GitHub-only first release would contradict the self-hosting requirement. The release architecture therefore needs two provider adapters: GitHub plus WebDAV. The provider-neutral protocol remains common to both. Do not silently reinterpret GitHub Enterprise or a GitHub account as satisfying self-hosting. WebDAV servers must pass capability and concurrency conformance checks.



- Do not assume upstream Mihon Android compatibility.
- Do not expose provider tokens to extensions.
- Do not treat a Git repository as the canonical sync protocol.
- Do not synchronize downloaded chapter files or extension binaries by default.
- Do not promise multi-provider support until adapter conformance tests pass.

No provider credentials were used. No sync request, account creation, remote repository, build, or test was executed.