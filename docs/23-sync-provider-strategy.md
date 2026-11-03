# NamiShelf — First-Release Sync Provider Strategy

**Status:** Architecture options and recommendation; provider not selected
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
- Implement one provider adapter for release 1 after the owner chooses the hosting model.
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
- Decide separately whether sync payloads require end-to-end encryption. Provider-side encryption is not equivalent to E2EE.
- Never send credentials, cookies, tracker tokens, extension binaries, or downloaded chapter content by default.
- Minimize metadata and document exactly which library/reading information leaves the device.
- Keep access tokens out of URLs, logs, crash reports, process arguments, and extension IPC.
- Support disconnect/revoke and explain what remote data remains after disconnect.
- If E2EE is selected, define device enrollment, key rotation, lost-device revocation, recovery, and multi-device key transfer before implementation.

## 7. Remaining owner decisions

1. **Hosting model:** should release 1 avoid a NamiShelf-operated service and rely on user/provider accounts, or is a NamiShelf-hosted service acceptable?
2. **First adapter:** GitHub, WebDAV, or S3-compatible storage? This is independent of the provider-neutral protocol decision.
3. **Self-hosting priority:** must a user be able to sync without a third-party cloud account in release 1?
4. **E2EE:** mandatory for release 1, optional, or deferred? Transport TLS and secure credential storage remain mandatory regardless.
5. **Sync data scope:** approve the candidate entity list as-is, or narrow release 1 to library entries, chapter read/progress state, categories, and followed authors?

## 8. Explicit non-goals

- Do not assume upstream Mihon Android compatibility.
- Do not expose provider tokens to extensions.
- Do not treat a Git repository as the canonical sync protocol.
- Do not synchronize downloaded chapter files or extension binaries by default.
- Do not promise multi-provider support until adapter conformance tests pass.

No provider credentials were used. No sync request, account creation, remote repository, build, or test was executed.