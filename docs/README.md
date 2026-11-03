# NamiShelf — Documentation Index

**Status:** Evidence-backed planning documentation  
**Date:** 2026-11-03

## Scope

This documentation records the conditional downstream-base assessment, approved product/architecture directions, Linux porting requirements, extension security model, backup interoperability, synchronization design, and outstanding owner decisions. It is not a security certification or runtime compatibility claim.

## Documents

- `01-downstream-audit.md` — initial evidence and risks in Mihon Desktop.
- `02-architecture-decision-record.md` — conditional base decision and owner-approved extension, sync, backup, and first-release scope ADRs.
- `03-upstream-sync-strategy.md` — upstream tracking and selective integration policy.
- `04-linux-platform-design.md` — Linux/Mint porting workstreams and validation matrix.
- `05-extension-security-model.md` — per-extension process model, brokered data access, conditional OS isolation, Android APK compatibility.
- `06-license-and-dependency-review.md` — preliminary licensing and dependency inventory.
- `07-agent-handoff.md` — confirmed decisions and operational guardrails.
- `08-backup-interoperability.md` — legacy Mihon compatibility boundaries and restore acceptance.
- `09-sync-architecture-assessment.md` — candidate GitHub-backed sync and credential boundaries.
- `10-reader-and-image-pipeline-assessment.md` — Skia reader, native runtime risks, and memory validation.
- `11-linux-runtime-and-dependency-risk.md` — desktop build feasibility and Linux packaging gates.
- `12-linux-credential-storage-assessment.md` — Secret Service integration and Linux test gaps.
- `13-linux-native-dependency-and-architecture-matrix.md` — native dependencies, architecture coverage, and XDG gap.
- `14-backup-format-and-restore-safety.md` — format evolution, resource limits, and partial restore semantics.
- `19-mihon-backup-compatibility-architecture.md` — pinned Android v0.21.3 backup implementation, legacy wire contract, and approved NamiShelf adapter strategy.
- `20-backup-workflow-and-conformance-plan.md` — backup UX, download export separation, loss reporting, and format conformance plan.
- `21-native-backup-container-options.md` — approved ZIP/JSON/encryption choices and remaining schema/security details.
- `22-native-backup-format-contract-draft.md` — manifest, canonical JSON, integrity, optional encryption, ZIP safety, and restore state machine.
- `23-sync-provider-strategy.md` — approved GitHub-first, self-hosted, E2EE sync requirements and remaining adapter/key-management choices.
- `24-sync-e2ee-and-key-management.md` — E2EE threat boundaries, key lifecycle, device enrollment, recovery, and provider ciphertext requirements.
- `25-self-hosted-sync-adapter-options.md` — WebDAV selection and server compatibility requirements.
- `26-extension-ipc-transport-and-broker-design.md` — comparison of stdio, Unix sockets, and D-Bus; broker contract and security requirements.
- `15-architecture-decisions-required.md` — remaining owner decisions.
- `16-provider-neutral-sync-protocol.md` — sync domain/protocol, provider adapters, conflicts, and security.
- `17-xdg-filesystem-and-migration-design.md` — Linux directory layout and safe migration.
- `18-linux-release-and-reader-acceptance.md` — staged platform/package policy and reader acceptance.

## Confirmed owner decisions

| Topic | Decision |
|---|---|
| Product identity | NamiShelf, independent product inspired by Mihon |
| Candidate technical base | `AltairCardinal/mihon-desktop`, conditional |
| Upstream policy | Retain upstream reference and integrate selectively |
| Sync architecture | Provider-neutral domain/protocol; provider-specific adapters |
| Sync release scope | Required in release 1; core entities only |
| Sync hosting | No mandatory NamiShelf-hosted service |
| First sync provider | GitHub adapter |
| Self-hosted sync | WebDAV adapter required in release 1 |
| Sync encryption | Mandatory E2EE |
| E2EE recovery | Recovery key plus authorized-device transfer |
| Device revocation | Rotate keys for future synchronized data |
| Sync entity scope | Library, chapter read/progress, categories, followed authors |
| Linux profile | Clean XDG profile; explicit opt-in migration |
| CPU architecture | x86_64 first; aarch64 after validation |
| Reader priority | Stability and bounded memory over aggressive prefetch |
| Backup architecture | NamiShelf-native versioned format; Mihon formats are adapters |
| Native backup container | ZIP (ZIP64 when required) |
| Native backup payload | Canonical JSON |
| Native backup encryption | Optional per backup |
| Library selection | Full library selected by default; settings selectable by category |
| Legacy Mihon backup | .tachibk import and export required in release 1 |
| Downloads backup | Separate export/migration operation |
| Portable settings | Selected by backup category; secrets/device-local state excluded |
| Extension execution | One separate process per third-party extension |
| Extension data access | Mediated by NamiShelf through versioned IPC/broker |
| Android APK extensions | DEX-to-JVM plus declared Android/Mihon API compatibility required in first release |
| Third-party extensions | Included in first public release |
| OS isolation | Pursue if sustainable; exact mechanism may be deferred by later decision |
| Linux target | Linux Mint 22.3 primary; Ubuntu-, Debian-, and Arch-based targets in scope |
| Packaging goals | AppImage, Flatpak, DEB |
| Implementation | Not authorized; documentation/planning only |

## Evidence baseline

- Mihon Desktop: `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`
- Mihon upstream: `mihonapp/mihon@e588e04359fe1058206a2cbac1b01018ad787359`

Re-check before implementation. No build, test execution, dependency resolution, advisory scan, complete license report, Linux runtime validation, extension fixture execution, or reproducible package test has been performed.
