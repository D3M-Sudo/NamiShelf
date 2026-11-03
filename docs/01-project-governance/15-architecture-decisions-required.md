# NamiShelf — Final Architecture Decision Register and Pre-Code Gates

**Status:** Consolidated owner decisions and remaining implementation blockers  
**Date:** 2026-11-03  
**Scope:** Architecture and specification only; no application code implementation is performed by this register.

This document supersedes older "open decision" lists where later ADRs or owner confirmations resolved the question. In case of conflict, the latest numbered ADR and explicit owner confirmation take precedence.

## 1. Product and technical base

- Product: NamiShelf, independent Linux desktop application inspired by Mihon; not an official Mihon product.
- Repository: `D3M-Sudo/NamiShelf`.
- Technical base: `AltairCardinal/mihon-desktop` remains a **conditional candidate**, not yet accepted as the production base.
- Upstream strategy: maintain Mihon upstream as a reference and selectively integrate relevant changes; preserve a documented downstream delta.
- UI: desktop-first hybrid UX, not a direct Android UI port.
- Documentation: technical English.
- Owner: D3M-Sudo.

**Blocking gate:** the candidate has not passed Linux build/runtime feasibility, resolved dependency/SBOM, full licensing/provenance, extension security, or native integration validation. A code phase must start with a bounded feasibility spike, not assume the base is already accepted.

## 2. First-release product scope and Linux targets

Confirmed:
- Reading, sources, library, tracking, backup, extensions, downloads, and history.
- Third-party extensions included in the first public release.
- Linux Mint 22.3 is the primary validation environment.
- Linux x86_64 is the first release gate; aarch64 follows only after native dependency and package validation.
- AppImage, Flatpak, and DEB are packaging goals, each independently gated.
- Clean NamiShelf XDG profile; migration/import from Mihon Desktop is explicit opt-in and must not silently adopt or mutate the old profile.
- Stability and bounded memory take priority over aggressive reader prefetch. Numeric budgets require real measurements.

## 3. Extensions and security

Owner-confirmed:
- One OS process per extension.
- All privileged data access mediated by NamiShelf broker; no direct core DB, backup, sync credential/key, tracking credential, or arbitrary host-file access.
- stdio is the IPC transport.
- Source-origin-restricted network policy.
- Editable predefined capability profiles.
- Large payload transfer via broker-issued temporary handles.
- Android APK extensions converted DEX-to-JVM are required in release 1.
- Third-party extensions are required in release 1.
- OS-level sandboxing should be pursued where technically sustainable; if not, the exact mechanism/release commitment must be deferred to an explicit owner decision. Process separation is not a sandbox.

Engineering defaults now specified:
- Protocol v1.0: UTF-8 JSON control frames plus binary stream frames, four-byte big-endian length prefix and explicit frame type (ADR-011 / document 29).
- Default-deny capability evaluation; host-computed intersection of product policy, profile, user overrides, source policy and method requirement.
- Standard Source, Authenticated Source, and Restricted/Review profiles are proposed, not yet owner-approved UX names/defaults (document 30).
- Network: HTTPS, per-source cookie/auth isolation, redirect reauthorization at each hop, no cross-origin credential forwarding, local/private/link-local/metadata destinations denied by default, DNS rebinding/SSRF defenses.
- Handles: random opaque identifiers bound to extension/process/session/request, short expiry, byte/concurrency quotas, cancellation, backpressure and revocation invalidation.
- Unsupported required Android/Mihon APIs block activation; unsupported optional APIs fail explicitly. No silent no-ops.

Documents: 05, 26, 27, 29, 30.

## 4. Backup architecture

Owner-confirmed:
- NamiShelf-native format is canonical; Mihon formats are interoperability adapters.
- Native container: ZIP/ZIP64.
- Structured payload: JSON, with canonicalization profile to be finalized.
- Encryption optional per backup.
- Complete library selected by default.
- Portable settings selected by category.
- Legacy Mihon/Tachiyomi `.tachibk` import and export required in release 1.
- Downloads are a separate export/migration operation.
- Future Mihon format adapter only after official implementation/format is inspectable and fixture-testable.

Required safety:
- ZIP traversal/duplicate path/symlink rejection, decompression and entry limits, streaming, validation before mutation, restore preview, checkpoint, bounded batches, cancellation and explicit partial-restore report.
- No credentials, cookies, provider tokens, E2EE keys, OS wrapping keys, or extension signing secrets in backups.
- Legacy export must disclose omitted/approximated fields; no silent data loss.

Still to finalize technically: canonical JSON standard, manifest/schema, integrity construction, AEAD/KDF profile after cryptographic review, hard parser limits, exact settings allowlist, download archive layout, loss-report format and file extension. These are engineering specifications, not unresolved product direction.

Documents: 14, 19, 20, 21, 22.

## 5. Synchronization architecture

Owner-confirmed:
- Provider-neutral NamiShelf sync domain/protocol (strategy B).
- No mandatory NamiShelf-hosted service in release 1.
- GitHub is the first provider adapter.
- Self-hosting is required in release 1; WebDAV is the second adapter.
- E2EE mandatory for all synchronized user payloads.
- Sync scope: core only — library entries, chapter read/progress state, categories, followed authors.
- Recovery supports both a user-held recovery key and authenticated transfer from an already-authorized device.
- Device revocation rotates keys for future synchronized data.

Explicit exclusions from release-1 sync: history, tracking associations, general preferences, downloads, extension state/binaries, provider credentials, tracker credentials, cookies and passwords.

Technical gates:
- GitHub and WebDAV adapter conformance, concurrency and interruption semantics.
- Cryptographic suite/key hierarchy must receive dedicated independent review; no custom cryptography and no primitive selection by guesswork.
- Device enrollment, key rotation, stale-device behavior, replay/rollback prevention and recovery need protocol fixtures.
- Historical ciphertext after revocation remains unresolved: retain under old keys, re-encrypt for remaining devices, or delete where feasible. Rotation cannot erase plaintext or ciphertext already copied by a revoked device.

Documents: 16, 23, 24, 25.

## 6. Linux credentials and platform policy

Evidence-based recommendation:
- Keep Secret Service as persistent credential backend and fail closed; never introduce a plaintext fallback.
- Native Linux and Flatpak credential service behavior has not been validated.

Owner confirmation still needed: whether persistent sync must be unavailable whenever Secret Service is unavailable, with no session-only fallback. Recommended default: yes.

Other platform policy details requiring owner input before release packaging freeze:
- minimum Debian/Ubuntu/Arch support window;
- bundled versus system JVM;
- Wayland support/release criteria;
- Flatpak runtime and application ID;
- AppImage update/signature policy.

These do not block writing implementation code for a Linux feasibility spike.

## 7. Chapter storage decision

Owner-approved under ADR-019: content-addressed filesystem storage, stable logical manga/chapter identifiers and a versioned per-chapter manifest. SQLite remains authoritative for structured metadata; permanent page payloads remain filesystem artifacts. Staging, permanent downloads, disposable disk cache and decoded-memory cache are separate retention classes. Hash algorithm, manifest serialization, directory layout, garbage collection and migration details remain open for M0. See `docs/04-core-architecture/38-content-addressed-storage-and-chapter-manifests.md`.

## 8. Reader page-loading decision

Owner-approved under ADR-018: asynchronous lazy loading, a byte-bounded LRU decoded-page cache and controlled directional prefetch. I/O and decoding stay off the UI thread; visible pages outrank speculative work; cache eviction never deletes durable downloads or reading progress. Numeric budgets, concurrency, prefetch radius, decoder and disk-cache policy require M0 inspection and measured reader profiling. See `docs/04-core-architecture/37-reader-runtime-and-page-pipeline.md`.

## 9. Presentation state model decision

Owner-approved under ADR-017: MVVM combined with Unidirectional Data Flow. Screens expose immutable observable state; user interactions are typed events; presentation invokes application use cases; one-off effects are separate from durable state. Long-running work remains service-owned and survives navigation/screen disposal. Specific ViewModel/state, navigation, coroutine and DI libraries are deferred until M0.

## 10. Window model decision

Owner-approved under ADR-016: one top-level application window for release 1. Library, feature destinations, contextual detail and reader mode transition within that window. Native transient dialogs/file pickers are permitted. Long-running background operations remain independent of screen lifecycle. Detached reader windows and broader multi-window support are not release-1 requirements.

## 11. Desktop navigation decision

Owner-approved under ADR-015: hybrid shell with collapsible persistent sidebar, central content workspace, optional contextual detail pane, and dedicated reader workspace. Reader entry/exit restores the previous navigation context; background operations continue independently. Exact responsive breakpoints, pane widths, minimum window size, tab/multi-window behavior and shortcut assignments remain prototype-level details.

## 12. UI framework decision

Owner-approved under ADR-014: Compose Multiplatform Desktop is the reference UI framework. The UX remains desktop-first rather than an Android UI port. Linux build/runtime, memory, sustained reader rendering, HiDPI/fractional scaling, accessibility, X11/Wayland policy and AppImage/Flatpak/DEB constraints are mandatory M0 gates. Kotlin/Compose versions and navigation/state libraries remain deferred. A failed critical gate requires evidence and owner review; do not silently replace the framework.

## 13. Local persistence decision

Owner-approved under ADR-013: SQLite embedded is the reference technology for local structured persistence. Large page/download artifacts and caches remain in managed filesystem storage. Database driver, SQL access layer, ORM/query framework, concurrency configuration and physical schema are deferred until after M0 inspection. If the candidate's SQLite stack proves unsuitable, present evidence and alternatives for explicit owner review; do not silently switch databases.

## 14. Preliminary application architecture baseline

The five-area architecture baseline is now documented in:

- `docs/04-core-architecture/34-application-architecture-blueprint.md` — modular monolith, logical module responsibilities, dependency direction and lifecycle.
- `docs/04-core-architecture/35-domain-data-and-persistence-architecture.md` — bounded contexts, stable identity, persistence recommendation, XDG layout and migration.
- `docs/04-core-architecture/36-desktop-ui-and-navigation-architecture.md` — desktop information architecture, navigation, reader UX and UI state ownership.
- `docs/04-core-architecture/37-service-integration-and-feature-orchestration.md` — sources, reader, downloads, tracking, backup, sync and extension orchestration.
- `docs/03-platform-and-linux/38-security-lifecycle-and-distribution-architecture.md` — trust zones, authorization, credentials, process lifecycle and packaging gates.

These are preliminary engineering proposals, not implementation evidence. Physical module count, SQLite driver/SQL access layer, Compose versions/navigation-state libraries, JVM packaging, OS sandbox mechanism and cryptographic suite remain conditional on M0 inspection or later owner decisions.

## 15. Explicitly unresolved owner decisions

Only the following require owner choice or confirmation rather than ordinary engineering specification:

1. **Authorize the first code phase:** permit a bounded Linux x86_64 feasibility spike on the conditional Mihon Desktop candidate, without authorizing a production fork/derivation or extension/sync release enablement. This is the next required authorization.
2. **Credential prerequisite:** confirm fail-closed Secret Service requirement and no session-only fallback.
3. **Capability UX:** approve or revise the proposed profile names/default grants in document 30.
4. **Historical ciphertext after device revocation:** choose retention under old keys, re-encryption for remaining devices, or deletion where feasible. Engineering recommendation: re-encrypt for remaining devices, then request provider deletion of superseded objects where supported; document that provider versioning/backups may retain old ciphertext and a revoked device may already possess it.
5. **Encrypted backup recovery:** confirm that loss of the backup password means the encrypted backup is unrecoverable; no hidden recovery escrow is proposed.
6. **Platform release policy:** confirm minimum distro/JVM/Wayland/Flatpak/AppImage policy before packaging freeze.

Everything else in the approved scope can proceed as engineering specification with secure defaults, fixture validation, and documented evidence. The five-area preliminary architecture baseline is documented; it still requires owner review before architecture freeze.

## 16. First code phase recommendation

**M0 — Linux feasibility and baseline, not full product implementation**

1. Reconfirm exact candidate and upstream commit snapshots.
2. Create a reproducible Linux x86_64 environment on Mint 22.3 or a clean compatible Ubuntu base.
3. Build and launch the unmodified candidate; record baseline tests and blockers.
4. Resolve runtime dependency graph, native artifacts, SBOM, license/NOTICE and provenance.
5. Verify clean NamiShelf XDG profile without touching legacy Mihon Desktop data.
6. Test Secret Service integration in the actual Linux session.
7. Produce a go/no-go report for adopting the candidate.

Do not enable third-party extension execution or synchronization in M0. Do not port or implement security-sensitive subsystems until their specifications and review gates are complete.

## 17. Operational guardrails

- No changes to Mihon upstream or the external Mihon Desktop repository.
- NamiShelf work remains in its own repository and branches.
- Preserve all license notices, attribution and provenance.
- No release claim without reproducible evidence.
- No secrets in logs, IPC diagnostics, arguments, environment, backups or extension processes.
- No arbitrary extension code in the main process.
- Security-critical crypto and archive parsing require independent review.
- A design decision is not an authorization to build, install dependencies, execute extensions, merge branches, or publish releases.
