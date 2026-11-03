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
- UI: desktop-first hybrid UX inspired by Mihon.
- Scope: reader, sources, library, tracking, backup, extensions, downloads, history.
- Packaging: AppImage, Flatpak, DEB.
- Platforms: Linux Mint 22.3 primary; Ubuntu-, Debian-, and Arch-based targets.
- Documentation: technical English.
- Extensions: one process per extension; brokered data access; Android APK DEX-to-JVM compatibility and third-party extensions required in first release; OS sandbox pursued if technically sustainable, otherwise exact mechanism/commitment deferred by later decision.

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
