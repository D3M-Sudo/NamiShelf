# NamiShelf — Pre-Code Implementation Blueprint and Readiness Gates

**Status:** Planning baseline; no application code implementation authorized by this document  
**Target:** NamiShelf release 1, Linux x86_64 first

## 1. Purpose

This blueprint converts the approved architecture and current design documents into an ordered engineering programme. It is the handoff contract for the first code-development phase. It does not claim the candidate base has passed its acceptance gates.

## 2. Workstream order and dependencies

| Order | Workstream | Depends on | Exit evidence |
|---:|---|---|---|
| 0 | Re-baseline candidate and upstream | Owner confirms code phase | Fresh commits, license/provenance, full source/API diff |
| 1 | Linux build/runtime feasibility | 0 | Reproducible Linux x86_64 build, startup, native dependency inventory |
| 2 | Core domain and XDG storage | 0–1 | Domain mapping, migrations, clean NamiShelf profile |
| 3 | Secure credential backend | 1–2 | Real Secret Service round-trip on Mint; Flatpak/DEB/AppImage behavior |
| 4 | Extension process supervisor + IPC v1 | 1–2 | Protocol conformance, process lifecycle, crash isolation |
| 5 | Broker + capability profiles + network enforcement | 4 | Origin policy, source auth, settings isolation, revocation tests |
| 6 | DEX-to-JVM and Android/Mihon shim | 4–5 | Fixture-backed compatibility profile and deterministic unsupported behavior |
| 7 | Library/reader/download Linux adaptation | 1–2 | Functional regression and bounded-memory evidence |
| 8 | Native backup + Mihon legacy adapters | 2 | Native round-trip, .tachibk import/export, loss reports |
| 9 | Provider-neutral sync + E2EE core | 2–3 | Crypto review, state/conflict protocol, recovery/revocation tests |
| 10 | GitHub + WebDAV adapters | 9 | Provider conformance, concurrent writer/interruption tests |
| 11 | Packaging and release engineering | 3–10 | AppImage, Flatpak, DEB independent acceptance |
| 12 | Full security, license, regression audit | all | Signed-off release evidence and unresolved risk register |

Do not parallelize work that violates dependencies. Extension runtime, sync cryptography, backup restore, and packaging are security-sensitive subsystems and require independent review.

## 3. First code-phase bootstrap checklist

Before coding:
1. Confirm the exact downstream repository and commit to use as a base.
2. Confirm whether to create a NamiShelf-owned fork or import selected source; preserve attribution and license notices.
3. Pin upstream Mihon and downstream Mihon Desktop snapshots.
4. Generate source inventory, license/NOTICE inventory, dependency graph and SBOM.
5. Define branch protection and PR workflow for NamiShelf; no changes to upstream repositories.
6. Establish Linux Mint 22.3 x86_64 baseline and a clean reproducible build environment.
7. Record baseline tests and known failures before modifications.
8. Create a porting delta ledger mapping upstream/downstream files to NamiShelf-owned changes.

## 4. First implementation milestone recommendation

**M0 — Reproducible Linux baseline and architecture skeleton**
- Build and launch the unmodified candidate on Linux x86_64.
- Document all blockers and native dependencies.
- Establish XDG path abstraction and NamiShelf app identity in a separate branch.
- Do not enable third-party extension execution or sync until their broker/crypto gates pass.
- Preserve a rollback path to the untouched candidate snapshot.

Exit criteria: clean reproducible build; startup/shutdown; no writes into legacy .mihon profile; dependency/license inventory; automated smoke test; documented native dependency risks.

## 5. Implementation policies

- No direct editing of Mihon upstream or the external Mihon Desktop repository.
- Keep NamiShelf changes in its own repository and branches.
- Preserve license headers, NOTICE and attribution.
- Every subsystem change has tests and a migration/rollback story.
- No secret in logs, IPC diagnostics, command-line arguments, environment variables, backups or extension process state.
- No arbitrary extension code in the main process.
- No release claims without evidence from the actual target package and environment.
- Security-critical crypto and archive parsing require dedicated review.

## 6. First-release scope traceability

Release 1 requirements:
- Linux x86_64; Linux Mint 22.3 primary validation environment.
- AppImage, Flatpak, DEB packaging goals, each independently gated.
- Reader, sources, library, tracking, backup, extensions, downloads, history.
- Third-party extensions, including Android APK DEX-to-JVM conversion and declared Mihon API compatibility.
- One process per extension, stdio IPC, host broker, origin-restricted networking, capability profiles, streaming handles.
- Native ZIP + canonical JSON backup, optional encryption, complete library default, category-selectable portable settings.
- .tachibk import and export; downloads separate.
- Provider-neutral sync in core scope; GitHub first adapter and WebDAV self-hosted adapter; mandatory E2EE; both recovery paths; device revocation rotates future-data keys.

## 7. Blocking decisions and owner inputs

The following cannot be silently guessed:
1. **Base-code authorization:** the conditional Mihon Desktop candidate has not passed Linux, dependency, license, extension-security, or full code review. Confirm whether the first code phase may begin with a Linux feasibility spike on the candidate, without yet authorizing a fork or production derivation.
2. **Credential prerequisite:** confirm the recommended fail-closed policy: persistent sync is unavailable when Secret Service is unavailable; no plaintext or session-only fallback.
3. **Extension profiles:** confirm proposed profile names and grants in document 30.
4. **Sync cryptography:** approve a dedicated cryptographic design review before selecting primitives/library. No custom cryptography.
5. **Historical data after device revocation:** choose retain encrypted under old keys, re-encrypt for remaining devices, or delete when feasible. Future data rotation alone does not revoke historical copies.
6. **Platform product policy:** minimum Debian/Ubuntu/Arch support window, system vs bundled JVM, Wayland release criteria, Flatpak runtime/application ID, AppImage update/signature policy.
7. **Native backup UX:** confirm file extension and password recovery wording; encrypted backup passwords cannot be recovered unless a separate recovery feature is designed.
8. **Tracking scope:** current sync scope excludes tracking; confirm no request to include tracking associations in release-1 sync.

## 8. Non-goals before the code phase

No code, build, installation, dependency resolution, tests, repository fork/import, merge, or PR is performed by this planning blueprint. The first code-phase authorization must be explicit and limited to the selected workstream.
