# ADR-001 — Conditional Selection of Mihon Desktop as NamiShelf's Technical Base

**Status:** Accepted conditionally; implementation authorization not granted  
**Date:** 2026-11-03

## Context

NamiShelf is an independent Linux desktop product inspired by Mihon. The candidate approaches were:
1. Build a new desktop application independently, reusing selected concepts/components.
2. Derive NamiShelf from `AltairCardinal/mihon-desktop`, itself based on Mihon.

The downstream candidate already provides a JVM desktop application and a substantial set of desktop-specific features. A greenfield implementation would duplicate significant reader, source, library, download, backup, and extension work.

## Decision

Use `AltairCardinal/mihon-desktop` as the **preferred candidate base, conditionally**. NamiShelf retains independent product identity, branding, roadmap, release process, and acceptance criteria.

This does not authorize forking or mirroring the candidate repository, modifying Mihon/Mihon Desktop code, starting builds, installing dependencies, or running tests. Documentation publication to the owner-created `D3M-Sudo/NamiShelf` repository is separately authorized.

## Conditions precedent

The candidate is not accepted as the production base until:
1. Linux build/runtime/packaging feasibility is demonstrated.
2. Extension trust and process-isolation policy is implemented and validated.
3. Resolved dependency graph, advisories, licenses, and third-party notices are reviewed.
4. Backup compatibility with Mihon is specified and tested.
5. Upstream synchronization cost and divergence are understood.

## Consequences

- Reuse of a functioning desktop product rather than reimplementation.
- NamiShelf inherits downstream architecture and maintenance constraints.
- Linux is a porting and validation workstream, not a packaging-only task.
- Upstream synchronization may create conflicts between Android-first changes and desktop-specific adaptations.

## Rejected alternative for now

A fully independent implementation is not selected because it would recreate substantial functionality. It remains a fallback if the audit shows that the downstream codebase is too difficult to secure, port, or maintain.

## ADR-001 Addendum — Extension execution and first-release scope

**Decision date:** 2026-11-03  
**Status:** Owner-approved direction; implementation details remain subject to feasibility review.

The owner confirmed:

1. **Process topology:** one separate process per extension.
2. **Data access:** mediated by NamiShelf through a narrow, versioned IPC/broker interface. Extensions must not receive direct access to core databases, backups, tracking/sync credentials, or arbitrary user files.
3. **OS-level isolation:** pursue an OS sandbox where technically sustainable across target distributions and package formats. If it proves disproportionately complex or unreliable, defer the exact mechanism and its release commitment to a later explicit decision. Deferral does not remove the mandatory per-extension process boundary or brokered data access.
4. **Android APK extensions:** support for third-party Android APK extensions converted from DEX to JVM-compatible artifacts is required. This is essential to the intended extension ecosystem and is a first-release requirement. DEX conversion alone is not Android API compatibility; a documented compatibility adapter and supported API matrix are required.
5. **Third-party extensions:** included in the first public release.

Still open: exact IPC technology and schema, sandbox implementation, network broker policy, user-consent/permission UX, and the precise Android/Mihon API compatibility matrix. Any compatibility feature that proves disproportionately expensive must be surfaced for a later owner decision rather than silently omitted or approximated.

These decisions authorize documentation and planning only. They do not authorize application coding, builds, dependency installation, or changes to Mihon/Mihon Desktop.


## ADR-002 — Provider-neutral synchronization architecture

**Decision date:** 2026-11-03  
**Status:** Owner-approved direction; protocol and release scope remain open

### Context

The candidate sync system is based on a private GitHub repository and custom Git transport. Adopting it unchanged would couple NamiShelf's protocol and account lifecycle to GitHub.

### Decision

Define a provider-neutral NamiShelf sync domain and protocol. Provider-specific transport and authorization live behind adapters. GitHub may be an initial provider, but repository structure, Git objects, and GitHub token semantics are not the domain contract.

### Consequences

- Define canonical entities, schema versions, operation IDs, idempotency, conflict resolution, tombstones, and migrations independently of transport.
- Declare provider capabilities; do not assume atomic writes or equivalent consistency.
- Keep credentials in OS secure storage and outside extensions and ordinary backups.
- Require adapter conformance tests.
- Preserve offline-first local operation.

### Not decided

First-release sync scope, initial provider, end-to-end encryption, key recovery/device enrollment, entity conflict rules, quotas, retention, and service operating costs.

This ADR authorizes planning only, not implementation.


## ADR-003 — NamiShelf-native backup with Mihon interoperability adapters

**Decision date:** 2026-11-03  
**Status:** Owner-approved architecture; detailed container design remains open

### Context

Mihon v0.21.3 is the last release documented to restore legacy Tachiyomi .tachibk backups. The following Mihon version removes that legacy compatibility. The replacement format was not specified by the inspected public announcement. NamiShelf requires durable long-term backups while retaining practical interoperability with Mihon users.

### Decision

NamiShelf owns a versioned native backup domain and native backup format. Mihon backup formats are independent import/export adapters, not the canonical internal model.

The first public release must support both import and export of the legacy v0.21.3-compatible .tachibk format. Export must disclose unsupported or lossy fields. Downloads use a separate export/migration operation. Portable settings are selected by backup category; secrets and device-local state are excluded.

A future Mihon backup adapter may be added only after its official implementation/format is inspectable and validated with fixtures.

### Consequences

- NamiShelf can evolve its native backup independently of Mihon's format changes.
- Legacy interoperability requires exact protobuf compatibility and semantic mapping, not only decoding.
- Native and legacy fixtures must be versioned separately.
- Download archives need independent integrity, source identity, and relinking rules.
- Restore requires bounded streaming, preflight, checkpointing, and explicit partial-result reporting.

### Open design details

Native container encoding and extension, manifest, schema migration, integrity/encryption, exact settings categories, download archive format, and legacy loss-report UX.

This ADR authorizes documentation and planning only, not implementation.


## ADR-004 — First-release Linux and product scope decisions

**Decision date:** 2026-11-03  
**Status:** Owner-approved planning direction; implementation authorization remains separate

### Decisions

1. **Synchronization release scope:** synchronization is required in the first public release. Its domain/protocol remains provider-neutral under ADR-002. The initial provider is not yet selected.
2. **Linux profile migration:** use a clean NamiShelf XDG profile. Provide explicit opt-in import/migration; do not silently adopt or mutate an existing Mihon Desktop profile.
3. **CPU architecture release order:** x86_64 is the first release gate. aarch64 follows only after native dependencies, runtime, and package formats are validated.
4. **Reader performance priority:** prioritize stability and bounded memory over aggressive prefetch. Set numeric budgets only from measurements on the Linux Mint 22.3 reference profile.

### Consequences

- Release 1 must include a usable sync implementation, not merely a dormant protocol abstraction.
- Provider selection, credentials, encryption/recovery, conflict semantics, and operating cost require further design.
- Migration must preserve the original source profile and report unsupported data.
- aarch64 is a follow-up target rather than a first-release blocker.
- Reader optimizations must not compromise memory bounds or long-session stability.

This ADR records product direction only. It does not authorize application coding, builds, dependency installation, or changes to Mihon/Mihon Desktop.

## ADR-005 — Native backup container and serialization

**Decision date:** 2026-11-03  
**Status:** Owner-approved design direction; security profile and detailed schema remain open

### Decisions

1. **Container:** ZIP; use ZIP64 extensions where required for large archives.
2. **Structured payload:** canonical JSON, with a documented canonicalization profile to be specified before implementation.
3. **Encryption:** optional per backup. When enabled, use a reviewed authenticated-encryption envelope; do not use legacy ZIP encryption.
4. **Library selection:** the complete library is selected by default. Portable settings remain selectable by category.
5. **Downloads:** remain outside the ordinary backup and use a separate export/migration operation (ADR-003).

### Consequences

- NamiShelf owns its JSON schema and migration policy; Mihon protobuf field numbers remain isolated in the legacy adapter.
- Deterministic canonicalization is required for stable digests and reproducible fixtures.
- ZIP path normalization, duplicate-entry rejection, decompression limits, and ZIP64 interoperability are mandatory.
- Optional encryption requires explicit UX for password creation, verification, forgotten passwords, and metadata leakage.
- Credentials, tokens, cookies, signing keys, and device-local state remain excluded regardless of encryption.

### Open technical details

Canonical JSON profile, manifest schema, ZIP feature subset, archive integrity/authentication semantics, AEAD/KDF profile, recovery UX, precise settings allowlist, and downloads archive layout.

This ADR authorizes documentation and planning only, not implementation, builds, or dependency installation.


## ADR-006 — First-release synchronization provider, hosting, scope, and E2EE

**Decision date:** 2026-11-03  
**Status:** Owner-approved product constraints; provider topology and cryptographic protocol details remain open

### Decisions

1. **Hosting:** NamiShelf will not operate a mandatory hosted sync service for release 1. Users connect to supported providers using their own accounts or infrastructure.
2. **First provider:** GitHub is the first provider adapter. GitHub is a transport/provider choice only; it does not define the sync domain or wire protocol.
3. **Self-hosting:** self-hosted synchronization is a release-1 requirement. A GitHub adapter alone does not satisfy this requirement. The release plan must include a separately deployable/user-managed provider adapter (candidate: WebDAV or self-hosted S3-compatible storage) or the owner must later revise the requirement. Do not claim this requirement is met by GitHub.com.
4. **End-to-end encryption:** mandatory for all synchronized user payloads. Provider-side encryption and TLS are additional controls, not substitutes for E2EE. The provider must receive ciphertext only, apart from the minimum metadata needed for storage and transport.
5. **Release-1 sync scope:** core scope only: library entries, chapter read/progress state, categories, and followed authors. History, tracking associations, general preferences, downloads, extension state, and credentials are excluded unless separately approved.

### Consequences

- Release 1 needs at least one GitHub adapter and a self-hosted adapter; adapter order and exact self-hosted technology remain to be decided.
- Provider capability differences must be represented explicitly. Do not imply equivalent atomicity or consistency.
- E2EE key generation, device enrollment, recovery, rotation, revocation, and lost-device handling are release-blocking design requirements.
- Key material must never be stored with provider data, in ordinary backups, logs, crash reports, or extension IPC.
- Sync must remain usable offline; a provider outage must not block local reading or library operations.
- Core sync scope must be enforced by schema and tests, not only hidden UI controls.

### Open decisions

- Self-hosted provider for release 1: WebDAV, S3-compatible, or another concrete option.
- Whether a self-hosted Git server adapter is acceptable as the second adapter, or whether a non-Git object API is preferred.
- E2EE cryptographic construction, KDF, key hierarchy, device enrollment, recovery, rotation, and revocation model.
- Whether multi-device recovery uses a user-held recovery key, an explicit device-to-device transfer, or both.
- Exact canonical entity schema, conflict semantics, tombstone policy, and metadata minimization.
- GitHub authorization scopes, token lifecycle, and repository creation/selection UX.

This ADR authorizes documentation and planning only, not implementation, builds, dependency installation, or operation of a hosted service.


## ADR-007 — Self-hosted sync adapter and E2EE device lifecycle

**Decision date:** 2026-11-03  
**Status:** Owner-approved design direction; detailed protocol and cryptographic profile remain open

### Decisions

1. **Self-hosted provider:** WebDAV is the release-1 self-hosted adapter, alongside the GitHub adapter.
2. **Recovery:** support both a user-held recovery key and authenticated transfer from an already-authorized device.
3. **Device revocation:** revoking a device triggers key rotation for future synchronized data. A revoked device must not decrypt payloads created after the rotation boundary.

### Consequences and required safeguards

- WebDAV is not assumed to provide uniform conditional-write, locking, or atomic replacement semantics. The adapter must declare capabilities and only enable safe synchronization modes supported by the server.
- Recovery-key generation, display, confirmation, secure storage guidance, and loss consequences must be designed without a NamiShelf-operated recovery service.
- Device-to-device transfer must use an authenticated encrypted channel and explicit user authorization.
- Recovery key and device transfer are complementary recovery paths; neither provider credentials nor GitHub/WebDAV account access may decrypt sync data.
- Revocation requires a key-version boundary and a protocol for distributing the new key only to remaining trusted devices.
- Historical ciphertext policy after revocation (retain under old keys, re-encrypt, or delete) remains open. Rotation does not retroactively erase data already decrypted or copied by a previously authorized device.

### Open technical details

WebDAV server compatibility profile and capability probes; conflict/conditional-write protocol; recovery-key encoding and entropy requirements; device transfer channel; key hierarchy and cryptographic suite; rotation propagation; stale-device behavior; treatment of historical ciphertext; and recovery after all trusted devices are lost.

This ADR authorizes documentation and planning only, not implementation, builds, dependency installation, or provider deployment.


## ADR-008 — Extension runtime IPC transport

**Decision date:** 2026-11-03  
**Status:** Owner-approved transport choice; wire schema and runtime details remain design work

### Decision

Use **stdio pipes** between NamiShelf and each separately supervised extension process. The channel uses a versioned framed protocol. Standard output is reserved exclusively for protocol frames; standard error is a bounded diagnostic stream and must be redacted before logging.

### Required properties

- Parent-owned process launch, supervision, cancellation, termination, and restart.
- Length-prefixed frames with strict maximum sizes and schema validation.
- Protocol major/minor negotiation and explicit compatibility behavior.
- Correlation IDs, bounded in-flight requests, deadlines, cancellation, and backpressure.
- No listening TCP port, ambient socket endpoint, or unnecessary inherited file descriptors.
- Large content delivered only through explicit brokered streaming operations with quotas and cancellation; never as unbounded control-message blobs.
- Transport abstraction must not leak into extension API/domain contracts.
- Same broker and capability policy for native JVM and converted DEX-to-JVM extensions.

### Consequences

- Extension stdout cannot be used for ordinary logs; runtime adapters must redirect or capture extension diagnostics safely.
- The host must handle malformed/truncated/oversized frames, partial reads/writes, process death, restart, and stale request IDs.
- Stdio is the selected transport, not a sandbox. Per-extension process separation and broker authorization remain mandatory; OS-level isolation remains conditional under prior decisions.
- AppImage, Flatpak, and DEB process-launch/stdio behavior must be validated before implementation freeze.

### Open technical details

Frame encoding, schema language, protocol version policy, handshake state machine, timeout/quota defaults, streaming channel design, Android API mapping, network broker rules, and packaging-specific process supervision.

This ADR authorizes documentation and planning only, not implementation, builds, dependency installation, or extension execution.

## ADR-009 — Android/Mihon extension compatibility contract

**Decision date:** 2026-11-03  
**Status:** Product requirement confirmed; exact API subset remains a design and validation task

### Decision and constraints

Android APK extensions converted from DEX to JVM are required in release 1, and third-party extensions are part of the first public release. Conversion alone is not considered compatibility. NamiShelf must publish a versioned Android/Mihon API compatibility profile and explicitly classify each API as supported, supported with limitations, unsupported, prohibited, or unassessed.

All converted extension code runs in its own process and uses the same stdio IPC broker and capability policy as native JVM extensions. Android shims must not expose unrestricted Android Context, ContentResolver, PackageManager, AccountManager, filesystem, service, or network access.

Unsupported required APIs must prevent activation with a clear diagnostic. Unsupported optional APIs must return documented deterministic errors, not silent no-ops or fabricated success.

### Release-1 acceptance gates

- Pin the Mihon extension API baseline.
- Publish method-level compatibility matrix.
- Validate representative, legally usable APK fixtures.
- Verify broker authorization, process isolation, resource limits, and deterministic unsupported behavior.
- Complete license/provenance and security review before enabling third-party extensions by default.

This ADR authorizes planning and documentation only; it does not authorize extension downloads, conversion, execution, implementation, or builds.

## ADR-010 — Extension broker network, capability profiles, and streaming handles

**Decision date:** 2026-11-03  
**Status:** Owner-approved policy direction; detailed schemas and enforcement remain design work

### Decisions

1. **Network policy — source-declared origins:** extension network access is mediated by the NamiShelf broker and limited to origins declared for the installed source and allowed by host policy. Redirects are re-evaluated at every hop; cross-origin redirects do not inherit credentials, cookies, or sensitive headers. TLS validation is mandatory. Localhost, private, link-local, and other sensitive address ranges are denied by default.
2. **Capability UX — profiles:** users grant capabilities through editable, predefined profiles rather than approving every routine capability individually. The host displays the effective permissions, permits per-capability overrides, and requires explicit additional consent for sensitive capabilities. An extension cannot self-grant or silently expand its profile.
3. **Large content — brokered handles:** large images/chapter payloads use opaque, temporary handles issued by the broker, not unbounded IPC control-message blobs. Handles are scoped to the extension process/session and request, with expiry, byte quotas, cancellation, and access validation.

### Consequences

- Source manifests and catalog metadata are untrusted declarations, not authority. NamiShelf validates origins and the user-approved profile before each request.
- DNS resolution and redirect handling must prevent private-network and rebinding bypasses; policy is applied to the resolved destination and each redirect hop.
- Cookie jars and authentication state are source-scoped. Sensitive headers are stripped on origin changes and never forwarded by default.
- Capability profiles are versioned by host policy. On extension/profile upgrades, newly requested permissions require renewed consent; previously granted permissions are not broadened automatically.
- Handles are unguessable, short-lived, non-transferable across extensions, and invalidated on cancellation, process exit, revocation, or expiry. The broker enforces streaming quotas and backpressure.
- These policies apply equally to native JVM and converted DEX-to-JVM extensions.

### Open technical details

Exact source-origin declaration format and catalog trust model; DNS pinning/rebinding defense; proxy behavior; capability profile names/defaults and sensitive-capability taxonomy; consent and revocation UX; handle token format, storage, chunking, resume semantics, and quota defaults; and per-package network enforcement.

This ADR authorizes documentation and planning only, not implementation, builds, dependency installation, extension downloads, or execution.


## ADR-012 — Modular monolith as the application architecture

**Decision date:** 2026-11-03  
**Status:** Owner-approved architecture direction; physical Gradle module decomposition deferred until after M0.

### Context

NamiShelf needs clear domain boundaries and dependency control without introducing distributed-system complexity into a desktop application. The conditional Mihon Desktop candidate already has an existing code and build structure that must be inspected before deciding how to map logical boundaries to Gradle modules.

### Decision

Adopt a **modular monolith** as the reference architecture for the trusted NamiShelf application:

- one primary NamiShelf host process for trusted application code;
- explicit logical boundaries between presentation, application use cases, domain, and infrastructure adapters;
- dependency direction from presentation to application contracts, application to domain/ports, and infrastructure implementing ports;
- extension code remains outside this monolith, in one separately supervised process per extension, using the approved stdio IPC broker;
- no microservice decomposition for release 1.

The module table in document 34 is a logical decomposition, not a command to create one Gradle module per row. The physical Gradle module count, package split, build conventions and retained candidate modules are deferred until after M0 inspection and feasibility evidence.

### Consequences

- Internal module contracts and dependency rules become architecture acceptance criteria.
- Cross-feature communication uses application contracts or typed events, not direct repository reach-through.
- The application shell owns composition and lifecycle.
- Build complexity and startup overhead remain lower than a distributed-services design.
- M0 must map the candidate's current modules to the logical boundaries and identify the minimum safe refactoring plan.

### Not decided

Exact Gradle module count, DI framework, package names, database driver, event implementation and candidate refactoring scope.

This ADR records an architecture decision only. It does not authorize M0, code changes, builds, dependency installation, tests, or repository changes outside the documentation work already authorized.


## ADR-013 — SQLite as the local persistence technology

**Decision date:** 2026-11-03  
**Status:** Owner-approved technology direction; driver and SQL access layer deferred until after M0.

### Context

NamiShelf requires durable local persistence for relational library, chapter, category, reading, history, tracking and operation metadata while remaining offline-first and portable across supported Linux distributions. The candidate Mihon Desktop may already contain database abstractions or Android-derived persistence adaptations that must be inspected before selecting a JVM driver or query framework.

### Decision

Adopt **SQLite embedded** as the reference technology for NamiShelf local structured persistence.

- SQLite stores structured domain and operational metadata.
- Large chapter/image artifacts, durable downloads and disposable caches remain in managed filesystem storage, not database BLOBs by default.
- Access is mediated by repository/application boundaries; feature UI and extensions never receive direct database handles.
- Schema changes use ordered, versioned migrations with explicit failure recovery.
- Backup and sync operate on validated domain-level exports/operations, not by copying the active database file.

### Deferred decisions

The SQLite JDBC/native driver, SQLDelight or other query layer, ORM choice, connection/concurrency configuration, journal mode, schema layout and physical Gradle module mapping are deferred until after M0 inspection of the candidate's database stack, runtime/native dependencies, licensing and packaging behavior.

### Consequences

- SQLite is the local persistence baseline; another database technology is not a co-equal default.
- M0 must verify that the candidate's existing SQLite stack is suitable for JVM/Linux and all target package formats.
- If the candidate stack is unsuitable, present concrete evidence and alternatives for an explicit owner decision rather than silently substituting another database.
- Migration, backup consistency, corruption recovery and concurrent background-operation behavior are release acceptance requirements.

This ADR records an architecture decision only. It does not authorize M0, database implementation, builds, dependency installation, tests, or changes to Mihon/Mihon Desktop.


## ADR-014 — Compose Multiplatform Desktop as the UI framework

**Decision date:** 2026-11-03  
**Status:** Owner-approved framework direction; Linux feasibility and performance are mandatory M0 gates.

### Context

NamiShelf is a Kotlin/JVM desktop application derived conditionally from the Mihon Desktop candidate. The candidate already uses Compose Multiplatform Desktop. Replacing the UI framework would create substantial rewrite and integration cost. NamiShelf nevertheless requires a desktop-first UX and cannot assume that Android-oriented UI patterns or current Linux behavior are suitable.

### Decision

Adopt **Compose Multiplatform Desktop** as the reference UI framework for NamiShelf.

- Build a desktop-first hybrid UX rather than a literal Android screen port.
- Keep presentation dependent on application use cases and typed UI state, not directly on repositories or infrastructure.
- Use a shared design system for accessibility, themes, typography, spacing and component behavior.
- Treat the reader as a focused desktop workspace with bounded-memory page presentation.
- Preserve framework-independent domain and application contracts.

### M0 mandatory validation gates

The framework choice is approved as the direction, but technical feasibility is not presumed. M0 must verify on the Linux Mint 22.3 x86_64 reference environment:

- clean Linux build and startup of the candidate UI;
- runtime compatibility and native dependency resolution;
- memory footprint and sustained-session behavior;
- rendering performance for library views and image-heavy reader workloads;
- HiDPI/fractional scaling, font rendering and resizing;
- keyboard navigation, focus behavior and accessibility support;
- X11 baseline and Wayland behavior within the agreed support policy;
- compatibility with AppImage, Flatpak and DEB packaging constraints.

A failed critical gate must be documented with reproducible evidence and brought to the owner before framework replacement or major UI rewrite is considered.

### Deferred decisions

Exact Compose Multiplatform and Kotlin versions, navigation/state libraries, UI architecture library, rendering configuration, minimum window size, responsive breakpoints and visual design details remain subject to candidate inspection and UX validation.

### Consequences

- Compose Multiplatform Desktop is the reference framework; JavaFX and GTK4/Libadwaita are not parallel implementation targets.
- Linux performance and compatibility are explicit adoption gates, not assumptions.
- The downstream UI may be selectively reused, refactored or replaced within Compose after M0 evidence and desktop UX review.
- No UI coding, framework upgrade or dependency installation is authorized by this ADR.

This ADR records an architecture decision only. It does not authorize M0 execution, builds, tests, dependency installation, or code changes.


## ADR-015 — Hybrid desktop navigation

**Status:** Owner-approved direction; detailed responsive behavior remains open.

NamiShelf adopts a hybrid desktop shell: a collapsible persistent sidebar for primary destinations, a central content workspace, an optional contextual detail pane on sufficiently wide windows, and a dedicated reader workspace that can hide or minimize global navigation. The reader is opened contextually from a library entry or chapter, not as a mandatory primary sidebar destination. Returning from the reader restores the previous navigation context. Background downloads and synchronization continue independently and remain visible through a compact status surface.

Exact widths, responsive breakpoints, minimum window size, tab/multi-window behavior, and shortcut assignments remain open for prototype review. The detail pane must collapse to a navigable detail view on narrow windows; keyboard and accessibility support are mandatory. This decision does not authorize UI implementation, builds, dependency installation, or tests.


## ADR-016 — Single-window application model

**Decision date:** 2026-11-03  
**Status:** Owner-approved for the reference application architecture.

### Context

The approved hybrid shell provides a collapsible sidebar, central content workspace, optional contextual detail pane and dedicated reader workspace. A multi-window model would add lifecycle, focus, state restoration, duplicate-session and cross-window coordination complexity before a demonstrated release-1 requirement exists.

### Decision

NamiShelf uses a **single top-level application window** as its release-1 and reference architecture.

- Primary destinations, contextual details and reader mode are hosted within the same top-level window.
- Entering reader mode transitions the central workspace into the dedicated reader experience; it does not spawn a second application window.
- Exiting reader mode restores the previous navigation destination and relevant transient context.
- Background downloads, tracking and synchronization remain service-owned and continue while the user changes screens or enters/exits reader mode.
- Dialogs, menus and native file pickers may use platform-managed transient surfaces; these do not constitute application multi-window support.

### Consequences

- Window lifecycle and global navigation restoration have one authoritative owner.
- Feature screens must not assume ownership of independent top-level windows.
- Cross-feature background work must not be scoped to a screen's composition lifecycle.
- The architecture should avoid unnecessary coupling to a single-window implementation detail where doing so is inexpensive, but multi-window behavior is not a release-1 requirement or hidden commitment.

### Not decided

Exact window dimensions, minimum size, close-to-tray behavior, startup/window-state restoration, and whether a future release should add optional detached reader windows.

This ADR records an architecture decision only. It does not authorize UI implementation, builds, dependency installation, or tests.


## ADR-017 — MVVM with Unidirectional Data Flow for presentation state

**Decision date:** 2026-11-03  
**Status:** Owner-approved presentation architecture; supporting libraries deferred until M0.

### Context

NamiShelf uses Compose Multiplatform Desktop in a single-window modular monolith. Its screens must coordinate library state, reader state and long-running operations without duplicating durable domain data or coupling presentation to infrastructure.

### Decision

Adopt **MVVM combined with Unidirectional Data Flow (UDF)** as the reference presentation model.

- Each screen or cohesive feature presentation scope exposes an immutable, observable UI state.
- User interactions are represented as explicit typed events/intents.
- ViewModels or equivalent presentation state holders process events and invoke application use cases.
- State transitions are deterministic and testable; asynchronous work is represented explicitly through loading, success, partial, stale and failure states where applicable.
- One-off effects (navigation requests, file-picker requests, notifications) are represented separately from durable screen state and are consumed predictably.
- Domain entities and durable state remain authoritative in application/domain services and repositories; UI state is a projection, not a second mutable source of truth.
- Long-running downloads, tracking and synchronization are owned by application services/job coordinators and survive screen disposal and navigation.
- Compose rendering remains side-effect free: no network/database work is initiated merely by recomposition.

### Deferred decisions

Specific ViewModel/state libraries, coroutine/Flow conventions, navigation library, dependency injection framework, reducer implementation style and lifecycle integration are deferred until M0 candidate inspection. Do not add dependencies solely to conform to this ADR before evaluating the existing stack.

### Consequences

- Feature screens follow consistent state/event/effect contracts.
- UI tests can exercise state transitions without launching the full desktop UI.
- Durable business state is not duplicated in ViewModels.
- Background work lifecycle is independent of screen composition.
- M0 must map the candidate's existing presentation architecture to this model and identify reuse/adaptation cost.

This ADR records an architecture decision only. It does not authorize UI implementation, builds, dependency installation, or tests.
