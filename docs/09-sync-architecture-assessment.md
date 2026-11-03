# NamiShelf — Multi-Device Synchronization Assessment

**Status:** Static assessment of candidate behavior; no credentials or remote sync used  
**Reviewed candidate:** `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Date:** 2026-11-03

## 1. Executive assessment

Mihon Desktop implements a project-specific cross-device synchronization system backed by a dedicated private GitHub repository. The README describes synchronization of favorites, followed authors, read status, and reading position between compatible desktop and Android builds of that fork. It explicitly states that upstream Mihon Android does not implement this protocol and downloaded chapter images are outside the sync scope.

This is not a generic Mihon sync service and must not be presented as compatible with upstream Mihon clients.

## 2. Observed architecture

The candidate contains a domain-level sync coordinator and protocol, a local journal/outbox, remote projection/inbox, GitHub Git transport, token authorization/refresh, a secure credential-store abstraction, and encrypted batch payload structures.

The inspected `SyncRuntime` wires a `PersistentGitHubCredentialStore` over a `SyncSecureStore`, GitHub authorization, coordinator, baseline store, inbox projection, and diagnostics. The Git transport uses Git objects and content-addressed blob identifiers; its bounded blob cache documentation says it contains no credentials or durable sync state.

The inspected test tree includes storage contracts for journal, outbox, baseline, reading, metadata, and sync panel state. These are useful design evidence, but static inspection is not an independent validation of correctness or security.

## 3. Data and security boundaries

Candidate README scope includes favorites, followed authors, read status, and reading position. Do not infer that it synchronizes:
- downloaded chapter image bytes or local caches;
- extension APK/JAR binaries;
- global tracker credentials;
- arbitrary source cookies or passwords;
- full application settings or backup archives.

The credential path is abstracted through `SyncSecureStore`; the actual desktop secure-store implementation, encryption key lifecycle, file permissions, backup/restore behavior, and OS keyring integration require a separate platform review. A class named “secure store” is not itself proof of hardware-backed or OS-vault protection.

## 4. NamiShelf owner decisions and remaining design

1. **Resolved:** NamiShelf uses a provider-neutral sync domain/protocol, with provider-specific adapters (ADR-002).
2. Define account authorization, token scope, refresh/revocation, and logout semantics.
3. Specify encryption-at-rest and end-to-end encryption threat model, key derivation/recovery, and device enrollment.
4. Define conflict semantics for read status, progress, favorites, author follows, and deletes.
5. Specify multi-device concurrency, idempotency, causal ordering, retries, offline queueing, and recovery from interrupted uploads.
6. Keep credentials outside ordinary backup files and outside extension capabilities.
7. Ensure diagnostics never include tokens, private repository contents, or sensitive identifiers.
8. **Resolved direction:** do not make GitHub a protocol requirement. The first provider is still to be selected; assess self-hosted and non-GitHub options.
9. Define what happens when the user loses the sync secret or private repository access.
10. Document clearly that upstream Mihon does not speak the candidate's custom protocol.

## 5. Linux-specific validation

- Verify secure credential storage on Linux Mint 22.3 with and without a desktop keyring service.
- Test Flatpak portal/runtime behavior and whether secret persistence is available under the chosen permissions.
- Test AppImage and DEB credential storage without storing tokens in plaintext preferences.
- Verify logout removes/revokes local credential material without deleting user data unexpectedly.
- Test private repository access, token refresh, rate limits, network interruptions, clock skew, and recovery.
- Confirm no credential leakage in crash reports, debug logs, process arguments, environment variables, or extension IPC.

## 6. Owner-approved release scope

Synchronization is required in the first public release. This is a product-scope decision, not evidence that the current candidate's GitHub sync is accepted unchanged. Provider selection, credential policy, encryption, conflict rules, and device recovery remain design work.

## 7. Assessment

The sync architecture is substantial and appears intentionally designed around journals, projections, encryption, and recovery. However, adopting it would create a strong GitHub dependency and a project-specific protocol. NamiShelf should treat it as a candidate subsystem, not as an already-decided product requirement. No sync backend choice is made in this document.

## 7. Source references

- [Mihon Desktop README](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/README.md)
- [SyncRuntime.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/data/src/commonMain/kotlin/mihon/data/sync/runtime/SyncRuntime.kt)
- [GitHub Git transport](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/data/src/commonMain/kotlin/mihon/data/sync/transport/GitHubGitDatabaseClient.kt)
- [Mihon Desktop feature guide](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/website/features.md)
