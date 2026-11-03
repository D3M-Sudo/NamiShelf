# NamiShelf — Architecture Decision Register and Pre-M0 Readiness

**Status:** Consolidated owner-approved decisions; one remaining authorization gate before M0  
**Date:** 2026-11-03  
**Scope:** Product/architecture decisions and pre-M0 readiness. This document does not authorize implementation.

The latest applicable owner-approved ADR is authoritative. Historical assessments and drafts remain useful for context but do not override later ADRs. Deferred engineering choices are not owner decisions unless explicitly marked as such.

## 1. Product identity and technical base

- Product: NamiShelf, an independent Linux desktop application inspired by Mihon; not an official Mihon product.
- Repository: `D3M-Sudo/NamiShelf`.
- Candidate base: `AltairCardinal/mihon-desktop`, conditional only; not accepted as the production base.
- Upstream strategy: Mihon remains a reference upstream; selectively integrate changes and preserve a documented downstream delta.
- UX: desktop-first, not a direct Android UI port.
- Documentation language: technical English.
- No fork/import/production derivation has been approved. That decision is deferred until M0 produces a candidate feasibility and licensing report.

## 2. Release-1 product scope

Owner-confirmed scope:
- Library, browse/sources, reader, downloads, history, tracking, extensions, backup and synchronization.
- Third-party extensions, including the Android APK DEX-to-JVM compatibility path, are release-1 product requirements subject to feasibility, API coverage and security gates.
- A clean NamiShelf XDG profile. Mihon Desktop data migration/import is explicit opt-in and must not silently adopt or mutate the old profile.
- Stability and bounded memory take priority over aggressive reader prefetch; numeric budgets require measurements.

## 3. Linux platform and release policy

Owner-approved under ADR-021 and ADR-027:
- Linux-first; release 1 targets x86_64 only.
- Linux Mint 22.3 x86_64 is the primary reference environment.
- Ubuntu-, Debian- and Arch-based x86_64 distributions are target families only when exact releases/package combinations are tested and documented. Minimum versions are not yet frozen.
- ARM64, Windows and macOS are not release-1 support commitments; architecture should avoid unnecessary barriers to future ports.
- A dedicated application JVM/runtime is the release policy. Do not require or silently fall back to an arbitrary system Java runtime. Exact distribution/version and packaging approach are M0 engineering findings.
- X11 is the mandatory first-release baseline. Wayland may be advertised only for tested compositor/session combinations.
- Initial public channel is Stable only. Preview/testing channels require a separate decision.
- DEB updates are owned by APT/configured repositories; Flatpak updates by Flatpak remotes; AppImage uses a dedicated per-user updater. AppImage provenance/signature verification, staged activation, rollback and user-data preservation are required.
- Each package format is an independent release gate. Minimum distribution versions, Flatpak runtime/application ID, AppImage updater implementation/signing operations and detailed Wayland matrix remain release-engineering work, not pre-M0 owner blockers.

## 4. Application architecture and local data

Owner-approved:
- Modular Monolith; physical Gradle module decomposition is deferred until after M0 inspection (ADR-012).
- SQLite embedded is the reference local structured persistence technology (ADR-013). Driver, SQL access layer, ORM/query framework, concurrency configuration and schema are deferred.
- Compose Multiplatform Desktop is the reference UI framework (ADR-014), subject to M0 Linux build/runtime/performance/compatibility gates.
- Hybrid shell: collapsible sidebar, central workspace, optional contextual pane and dedicated reader workspace (ADR-015).
- Single top-level window for release 1 (ADR-016).
- MVVM + Unidirectional Data Flow (ADR-017); concrete libraries are deferred.
- Reader uses asynchronous lazy loading, byte-bounded LRU decoded-page cache and controlled directional prefetch (ADR-018); numeric budgets and decoder are measured later.
- Chapter payloads use content-addressed filesystem storage with stable logical IDs and versioned per-chapter manifests; SQLite owns structured metadata (ADR-019). Hash, manifest encoding, GC and directory layout are deferred.
- XDG-separated data/config/cache plus a user-selectable permanent download directory, including another disk (ADR-020). Cache cleanup must never delete permanent downloads. External-volume relocation must be verified, recoverable and non-destructive.

## 5. Extension architecture and authorization

Owner-confirmed:
- One OS process per extension; stdio IPC.
- All privileged data access is broker-mediated. Extensions receive no direct core database, backup, sync keys, provider/tracker credentials, general secret-store access, shell or arbitrary host-filesystem access.
- Network access is source-origin-restricted; large content uses broker-issued temporary streaming handles.
- Android APK extensions converted DEX-to-JVM are a release-1 requirement, subject to documented API compatibility.
- OS-level sandboxing should be pursued where technically sustainable. Process separation is not a sandbox. Exact mechanism and release commitment remain a later owner decision if a sustainable solution cannot be established.

Owner-approved capability policy under ADR-025:
- **Standard Source** (default): source search/details/chapters; HTTPS only to source-declared and host-validated origins; broker-issued stream handles; extension-private settings.
- **Authenticated Source:** Standard Source plus source-scoped authentication, enabled only with explicit user consent. NamiShelf owns credential collection, storage, cookies/session and refresh.
- **Restricted / Review:** no network or source-auth capability until user review and explicit enablement.
- Default deny; grants are scoped to extension and source identity; updates cannot silently expand permissions; user can inspect/revoke grants.
- Profiles are permission templates, not trust ratings. Provenance/signature status is separate.

Engineering proposal, not owner-frozen:
- Document 29 describes a candidate IPC v1 framing/schema and initial resource limits. It is not authorized by ADR-011; ADR-011 is intentionally unassigned. The framing, limits, handshake fields and method registry must be validated in the later extension implementation workstream. They do not block M0.

## 6. Native backup architecture

Owner-confirmed:
- NamiShelf-native backup is canonical; Mihon formats are interoperability adapters (ADR-003).
- ZIP/ZIP64 container, JSON structured payload, optional per-backup encryption, full library selected by default (ADR-005 and earlier owner choices).
- Portable settings selected by category.
- Legacy Mihon/Tachiyomi `.tachibk` import/export required in release 1; legacy export must disclose omissions/approximations.
- Downloads are a separate export/migration operation.
- No credentials, cookies, provider tokens, E2EE keys, OS wrapping keys or extension signing secrets in backups.

Encrypted-backup recovery approved under ADR-026:
- Both backup password and separate locally generated recovery key are supported as alternative independent recovery methods.
- Recovery key is shown once at creation/explicit regeneration, stored separately by the user, never transmitted to providers/logged/embedded in plaintext.
- Losing both methods means the backup is unrecoverable; no vendor escrow or plaintext fallback.
- Envelope/key-wrapping construction and cryptographic primitives require independent review.

Deferred engineering details: canonical JSON/schema, exact ZIP subset and parser limits, manifest/integrity model, AEAD/KDF/wrapping construction, settings allowlist, download archive layout and conformance fixtures.

## 7. Synchronization architecture

Owner-confirmed:
- Provider-neutral sync domain/protocol (strategy B); no mandatory NamiShelf-hosted service.
- GitHub is the first provider; WebDAV is the second adapter and self-hosting is required for release 1.
- E2EE is mandatory for all synchronized user payloads.
- Sync scope is core only: library entries, chapter read/progress state, categories and followed authors.
- Recovery for adding a device supports both a user-held recovery key and authenticated transfer from an already-authorized device.
- Device revocation rotates keys for future data.

Historical data after revocation approved under ADR-024:
- Re-encrypt historical synchronized data for remaining authorized devices under the new key version.
- Request deletion of superseded ciphertext from the provider where supported.
- Deletion is best-effort; provider versioning/backups/replicas may retain old ciphertext. A revoked device may already possess historical ciphertext/plaintext.

Release-1 sync exclusions: history, tracking associations, general preferences, downloads, extension state/binaries, provider/tracker credentials, cookies and passwords.

Deferred engineering/security gates: cryptographic suite/key hierarchy requires independent review; exact causal/conflict model, key rotation protocol, recovery fixtures, provider conformance and stale-device behavior must be completed before sync implementation/release. No custom cryptography.

## 8. Linux credential policy

Owner-approved under ADR-023:
- Linux Secret Service is the only persistent credential backend.
- Fail-closed, no fallback: no plaintext/file/app-encrypted/database/environment/session-memory alternative.
- If Secret Service is unavailable, locked or inaccessible, credential-dependent operations remain unavailable; unrelated local functionality remains usable.
- Validate native Linux and Flatpak integration at the relevant packaging gate, including unavailable/locked/error states and proof no alternate persistence occurs.

## 9. Authoritative technical specifications

- Application architecture: documents 34–37.
- Chapter content-addressed storage: document 38.
- XDG/download storage: document 39.
- Linux packaging matrix: document 40.
- Update/release channels: document 41.
- Extension capabilities: document 30.
- Extension IPC candidate: document 29 (engineering proposal; not ADR-011).
- Native backup v1 profile: document 32; older document 22 is a historical draft and must not override ADR-026/document 32.
- Provider-neutral sync domain protocol: document 33; older document 24 is a threat-boundary/design document whose stale owner-decision statements are superseded by ADR-024.

## 10. Remaining owner authorization before M0

**Exactly one owner authorization remains:**

1. **Authorize M0 — bounded Linux feasibility and baseline assessment** on the unmodified conditional Mihon Desktop candidate, using Linux x86_64 and Linux Mint 22.3 as the primary reference.

No decision to fork, import selected source, create a production derivation, implement NamiShelf features, enable third-party extensions, implement sync/backup cryptography, package releases, merge or publish is included in M0 authorization.

## 11. M0 scope and exit criteria

M0 is a feasibility spike, not the first product implementation milestone.

1. Pin exact Mihon upstream and Mihon Desktop candidate commits and preserve source provenance.
2. Inspect repository structure, build system, current Linux support claims and existing test/build instructions.
3. Establish an isolated, reproducible Linux x86_64 environment. Any dependency installation or environment mutation must be covered by the explicit M0 authorization.
4. Build and launch the unmodified candidate, if technically possible; record baseline test results and failures without modifying candidate code.
5. Inventory JVM/runtime requirements, native libraries, architecture-specific artifacts, graphics/image dependencies and system integration.
6. Produce source/dependency inventory, SBOM where feasible, license/NOTICE/provenance findings and unresolved obligations.
7. Assess whether the approved dedicated JVM runtime, Compose Desktop, SQLite and required Linux Secret Service integration are technically feasible; do not implement them.
8. Inspect clean-profile/XDG behavior and verify that legacy Mihon Desktop user data is not read or mutated.
9. Assess X11 as mandatory baseline and record Wayland observations without making untested compatibility claims.
10. Produce a go/no-go report with evidence, blockers, risks, recommended base strategy and explicit decisions required after M0.

M0 does **not** include NamiShelf code changes, architecture skeleton, fork/import, extension execution, sync, backup restore, release packaging, PRs or merges. If any step requires modifying the candidate or performing a destructive/privileged operation, stop and request separate authorization.

## 12. Later engineering gates (not pre-M0 owner blockers)

- Select exact JVM distribution/version and packaging method based on M0 evidence.
- Establish tested minimum Debian/Ubuntu/Arch releases and detailed X11/Wayland matrix.
- Choose Flatpak runtime/application ID and AppImage updater/signing implementation.
- Decide physical Gradle module boundaries and concrete SQLite/Compose/navigation/DI libraries.
- Finalize content hash, chapter manifest serialization, garbage collection and migration.
- Complete extension IPC v1, Android/Mihon API coverage, DEX-to-JVM feasibility and OS sandbox decision.
- Complete backup cryptographic design, parser limits, format conformance and legacy fixtures.
- Complete sync cryptographic review, conflict/causal model, provider adapters and revocation/recovery fixtures.
- Decide fork/import/production derivation strategy only after M0 evidence and separate owner review.

## 13. Operational guardrails

- No changes to Mihon upstream or the external Mihon Desktop repository.
- Keep all authorized work in NamiShelf's own repository/branches.
- Preserve license notices, attribution and provenance.
- No release claim without reproducible evidence.
- No secrets in logs, IPC diagnostics, arguments, environment, backups or extension processes.
- No arbitrary extension code in the main process.
- Security-critical cryptography and archive parsing require independent review.
- An architecture decision is not authorization to build, install dependencies, execute extensions, merge branches or publish releases.
