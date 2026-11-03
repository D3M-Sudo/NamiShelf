# NamiShelf — Agent Handoff and Operational Guardrails

**Status:** Planning complete except explicit M0 authorization; no M0 execution or application implementation  
**Date:** 2026-11-03  
**Authoritative register:** `docs/01-project-governance/15-architecture-decisions-required.md`  
**Documentation index:** `docs/01-project-governance/00-documentation-index.md`

## 1. Product objective and candidate

NamiShelf is an independent Linux desktop application inspired by Mihon; it is not an official Mihon product. `AltairCardinal/mihon-desktop` is the preferred but conditional technical-base candidate. It has not been accepted as the production base. M0 must provide evidence before any fork/import/production-derivation decision.

## 2. Owner-approved decisions

### Product and platform
- Linux-first; release 1 x86_64 only; Linux Mint 22.3 x86_64 is the primary reference environment (ADR-021/027).
- Architecture should avoid unnecessary platform assumptions for future ports, but ARM64, Windows and macOS are not release-1 commitments.
- Dedicated application JVM/runtime; no arbitrary system-JVM fallback. Exact distribution/version and packaging mechanism follow M0 evidence (ADR-027).
- X11 is the mandatory first-release baseline; Wayland is claimed only for tested compositor/session combinations (ADR-027).
- Stable is the only initial public release channel (ADR-027).
- DEB updates use APT/configured repositories; Flatpak updates use Flatpak remotes; AppImage uses a dedicated per-user updater (ADR-022).
- Clean NamiShelf XDG profile; Mihon Desktop migration/import is explicit opt-in and must not silently adopt or mutate the old profile (ADR-004/020).
- Permanent downloads may use a user-selected alternative directory or disk; disposable cache cleanup must never delete permanent content (ADR-020).

### Core architecture and UX
- Modular Monolith; physical Gradle module decomposition deferred (ADR-012).
- SQLite embedded for local structured persistence (ADR-013).
- Compose Multiplatform Desktop as reference UI framework (ADR-014).
- Hybrid navigation with collapsible sidebar, central workspace, contextual detail and dedicated reader workspace (ADR-015).
- One top-level window in release 1 (ADR-016).
- MVVM + Unidirectional Data Flow (ADR-017).
- Reader uses asynchronous lazy loading, byte-bounded LRU decoded-page cache and controlled directional prefetch (ADR-018).
- Chapter payloads use content-addressed filesystem storage with per-chapter manifests; SQLite owns structured metadata (ADR-019).

### Extensions
- Third-party extensions are required in release 1, one OS process per extension; no extension code in the main process (ADR-001 addendum).
- Privileged access is broker-mediated; stdio is the IPC transport (ADR-008).
- Source-declared origin restrictions and broker-issued streaming handles (ADR-010).
- Capability profiles approved under ADR-025: Standard Source (default), Authenticated Source (explicit source-scoped consent), Restricted / Review (no network/auth until reviewed). Default deny; no silent permission expansion.
- Android APK DEX-to-JVM conversion and a declared Android/Mihon API compatibility subset are release-1 requirements (ADR-009); conversion alone is not compatibility.
- OS-level sandboxing is pursued where sustainable. Process separation is not a sandbox; exact mechanism/release commitment remains a later owner decision if necessary.
- Document 29 is an engineering IPC proposal, not an owner-frozen wire contract. ADR-011 is intentionally unassigned.

### Backup
- NamiShelf-native backup is canonical; Mihon formats are interoperability adapters (ADR-003).
- ZIP/ZIP64 container, canonical JSON payload, optional encryption, complete library selected by default, settings selectable by category (ADR-005).
- Legacy `.tachibk` import/export required in release 1; downloads are a separate export/migration operation.
- Encrypted backups support both password and a separate locally generated recovery key as alternative independent recovery methods (ADR-026). Losing both means unrecoverable; no vendor escrow or plaintext fallback.

### Synchronization
- Provider-neutral sync; no mandatory NamiShelf-hosted service; GitHub first adapter; WebDAV second adapter and self-hosting required (ADR-002/006/007).
- E2EE mandatory; core sync scope is library entries, chapter read/progress, categories and followed authors. History, tracking associations, general preferences, downloads, extensions and credentials are excluded.
- Device enrollment/recovery supports both a user-held recovery key and authenticated transfer from an already-authorized device (ADR-007).
- Revocation rotates future-data keys; historical ciphertext is re-encrypted for remaining authorized devices, followed by best-effort provider deletion where supported (ADR-024).
- Cryptographic construction/key hierarchy and conflict protocol require independent review before sync implementation.

### Credentials
- Linux Secret Service is the only persistent credential backend; fail-closed with no fallback of any kind (ADR-023).
- When unavailable/locked/inaccessible, credential-dependent operations remain unavailable; unrelated local functionality remains usable.

## 3. M0 authorization gate

The only remaining owner authorization before M0 is explicit permission to perform a bounded Linux x86_64 feasibility assessment on the unmodified conditional Mihon Desktop candidate.

M0 may include an isolated environment, dependency installation, build/start of the unmodified candidate and safe baseline tests only if explicitly authorized. It must not edit candidate or NamiShelf application code, create a fork/import/production derivation, implement features, execute extensions, implement backup/sync, package releases, create PRs or merge.

M0 must pin exact upstream/candidate commits; inspect build/runtime/native dependencies; produce source/dependency/SBOM/license/NOTICE/provenance findings; assess the dedicated JVM, Compose Desktop, SQLite and Secret Service feasibility; inspect clean XDG/profile isolation; assess X11 and record Wayland observations; and produce an evidence-backed go/no-go report.

Any destructive, privileged, out-of-scope or candidate-modifying step requires separate authorization.

## 4. Deferred engineering work — not owner decisions blocking M0

- Exact JVM distribution/version, runtime packaging, minimum distro releases, Flatpak runtime/application ID and AppImage updater/signing operations.
- Concrete Compose/Kotlin/navigation/state/DI libraries; SQLite driver/SQL layer/schema; physical Gradle module decomposition.
- Hash algorithm, chapter manifest serialization, directory layout, garbage collection and migration.
- Extension IPC framing/limits/method registry, Android/Mihon API coverage, DEX-to-JVM validation, OS sandbox mechanism.
- Backup canonical JSON/ZIP profile, parser limits, integrity construction and independently reviewed cryptographic envelope.
- Sync cryptographic suite/key hierarchy, causal/conflict model, provider conformance and detailed recovery/revocation protocol.
- Reader memory/cache/prefetch budgets from measured profiling.

These must be resolved before their respective implementation or release gates, but they do not require additional owner product choices before the M0 feasibility spike.

## 5. Evidence discipline

Every audit finding must include stable ID, severity/confidence, affected path/behavior, evidence, user impact, remediation, verification method and status (confirmed/suspected/not assessed). Never call an unexecuted test passed. Never infer security from signatures, process separation or classloaders alone.

## 6. Operational guardrails

- No changes to Mihon upstream or the external Mihon Desktop repository.
- Keep all authorized work in NamiShelf's own repository/branches.
- Preserve license notices, attribution and provenance.
- No release claim without reproducible evidence.
- No secrets in logs, IPC diagnostics, arguments, environment, backups or extension processes.
- No arbitrary extension code in the main process.
- No custom cryptography; independent review is mandatory.
- An architecture decision is not authorization to implement, build, install dependencies, execute extensions, package, merge or publish.
