# NamiShelf — Agent Handoff and Operational Guardrails

**Status:** Planning-only handoff  
**Date:** 2026-11-03

## 1. Product objective

Prepare NamiShelf as an independent Linux desktop application inspired by Mihon, with `AltairCardinal/mihon-desktop` as a conditional technical-base candidate.

## 2. Confirmed owner decisions

- Product: NamiShelf; independent/inspired, not official.
- Repository: `D3M-Sudo/NamiShelf`; documentation publication authorized on existing branch `feature`.
- Candidate base: conditional adoption of `AltairCardinal/mihon-desktop`.
- Upstream policy: retain upstream references and selectively integrate changes.
- Synchronization: provider-neutral domain/protocol; no mandatory NamiShelf-hosted service; GitHub is first adapter; self-hosting is required in release 1 and therefore requires WebDAV as second adapter; E2EE mandatory; recovery via recovery key plus authorized-device transfer; device revocation rotates keys for future data; release-1 scope limited to library, chapter read/progress, categories, followed authors (owner-approved 2026-11-03).
- Backup: NamiShelf-native versioned format is the default; ZIP container (ZIP64 where required), canonical JSON payloads, optional per-backup encryption, and full-library selection by default (owner-approved 2026-11-03). Legacy Mihon/Tachiyomi .tachibk import and export are required in release 1; downloads use a separate export/migration flow; portable settings are selected by category.
- Linux filesystem: clean NamiShelf XDG profile with explicit opt-in migration/import; no silent profile adoption (owner-approved 2026-11-03).
- Linux release architecture: x86_64 first; aarch64 after validation (owner-approved 2026-11-03).
- Reader acceptance: prioritize stability and bounded memory over aggressive prefetch (owner-approved 2026-11-03).

- UI: desktop-first hybrid UX inspired by Mihon.
- Scope: reader, sources, library, tracking, backup, extensions, downloads, history.
- Packaging: AppImage, Flatpak, DEB.
- Platforms: Linux Mint 22.3 primary; Ubuntu-, Debian-, and Arch-based targets.
- Documentation: technical English.
- Extensions: one process per extension; stdio pipes with versioned framed IPC (ADR-008); brokered data access; Android APK DEX-to-JVM compatibility and third-party extensions required in first release; OS sandbox pursued if technically sustainable, otherwise exact mechanism/commitment deferred by later decision.

## 3. Non-negotiable constraints

Unless separately authorized:
- Do not fork or mirror Mihon/Mihon Desktop.
- Do not modify Mihon upstream or Mihon Desktop.
- Do not modify application code, build, install dependencies, run setup, or execute state-mutating tests.
- Do not publish packages/releases.
- Do not claim Linux, extension, backup, or sync compatibility without evidence.
- Do not create additional branches, tags, or pull requests unless explicitly requested.

The owner has authorized publication of the planning documents to the existing NamiShelf `feature` branch. This authorization does not extend to application implementation.

## 4. Required sequence before implementation

1. Complete remaining static review of extension security, backup interoperability, sync data flows, reader/image stack, and platform dependencies.
2. Resolve IPC schema, network broker policy, consent/capability UX, sandbox feasibility, and Android/Mihon API compatibility matrix.
3. Complete dependency/SBOM/license analysis in an explicitly authorized environment.
4. Define Linux runtime and packaging acceptance criteria.
5. Present final architecture recommendation and unresolved trade-offs.
6. Obtain explicit authorization before coding, builds, dependency installation, or changes to Mihon/Mihon Desktop.

## 5. Evidence discipline

Every audit finding must include stable ID, severity/confidence, affected path/behavior, evidence, user impact, remediation, verification method, and status (confirmed/suspected/not assessed). Never call an unexecuted test passed. Never infer security from signatures or classloaders.

## 6. Extension implementation constraints

- Keep third-party extension code out of the NamiShelf main process.
- Run one separate process per extension.
- Mediate data access through a narrow, versioned, validated IPC/broker interface.
- Do not expose core databases, backups, tracking/sync credentials, arbitrary user files, or generic process execution.
- Support converted Android APK extensions in the first release, with a declared Android/Mihon API compatibility subset; DEX conversion alone is insufficient.
- Include third-party extensions in the first public release.
- Pursue OS sandboxing where technically sustainable; if disproportionate/unreliable, obtain a later owner decision on deferral. Process separation and brokered access remain mandatory.
- Escalate disproportionately expensive Android compatibility features rather than silently dropping or pretending to support them.


## 7. IPC and compatibility planning status

- IPC transport is approved: stdio pipes; stdout is protocol-only and stderr is bounded diagnostics.
- Wire schema, handshake, capability catalogue, network policy, streaming, and package-specific supervision remain design tasks.
- Converted Android APK extensions must use the same broker boundary as native JVM extensions.
- Do not claim general Mihon extension compatibility until the API compatibility matrix and representative extension fixtures are defined and validated.


## 8. Broker policy decisions confirmed

- Network: source-declared origins only; broker revalidates redirects and blocks local/private destinations by default.
- Capability UX: editable predefined profiles, per-capability overrides, and explicit extra consent for sensitive capabilities.
- Streaming: temporary opaque broker handles scoped to extension process/session/request, with expiry, quotas, cancellation and backpressure.
- Record these in ADR-010. Do not implement or execute extension code without separate authorization.


## 9. Final pre-code specification baseline

The following documents are the implementation-planning references and supersede older unresolved design notes where they conflict:

- ADR-011 records UTF-8 JSON control frames plus binary stream frames over stdio.
- Document 29 is the normative IPC v1 target.
- Document 30 defines the proposed capability catalogue and profiles; exact profile names/defaults remain owner-review items.
- Document 31 defines ordered implementation workstreams and the M0 Linux feasibility gate.
- Document 32 defines the proposed native backup v1 profile; encryption primitive/KDF selection remains subject to independent review.
- Document 33 defines the provider-neutral sync domain and adapter requirements; causal conflict model and cryptographic profile remain to be frozen after review.
- Document 15 is the consolidated decision register and authoritative list of owner inputs.

The next authorized development activity should be a bounded Linux x86_64 feasibility spike on the conditional Mihon Desktop candidate. It must not assume production-base acceptance, enable extension execution, or enable sync. Obtain explicit owner authorization before starting that code/build phase.


## 10. Preliminary application architecture baseline

The five-area architecture proposal is now published:
- 34 — application architecture and module boundaries;
- 35 — domain data and persistence;
- 36 — desktop UI and navigation;
- 37 — service integration and orchestration;
- 38 — security, lifecycle and distribution.

These documents are proposals pending owner review and M0 evidence. Do not translate a proposed database, dependency injection, JVM packaging, sandbox or module split into an implementation choice without checking its stated gate. No M0, build, installation, application coding or tests have been authorized by this documentation update.
