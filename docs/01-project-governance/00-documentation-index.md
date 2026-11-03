# NamiShelf Documentation Index

This index is stored with project governance documents so that every document under `docs/` belongs to a thematic subdirectory. The repository-root README links here.

## Directory map

| Directory | Scope |
|---|---|
| [01-project-governance](./) | ADR history, authoritative decision register, handoff and documentation governance |
| [02-audit-and-licensing](../02-audit-and-licensing/) | Downstream audit, licensing and dependency review |
| [03-platform-and-linux](../03-platform-and-linux/) | Linux platform design, runtime, credentials, native dependencies, XDG, security and distribution |
| [04-core-architecture](../04-core-architecture/) | Upstream strategy, reader pipeline, domain/persistence, UI, storage and application services |
| [05-extensions](../05-extensions/) | Extension security, IPC, Android/Mihon compatibility, broker and capability profiles |
| [06-backup](../06-backup/) | Native backup, Mihon compatibility, restore safety and conformance |
| [07-synchronization](../07-synchronization/) | Provider-neutral sync, E2EE, GitHub and WebDAV |
| [08-development-planning](../08-development-planning/) | Pre-M0 planning, M0 scope and implementation gates |

## Authoritative documents

- [Architecture Decision Record](02-architecture-decision-record.md) — append-only history of approved architecture decisions, ADR-001 through ADR-027. ADR-011 is intentionally unassigned; protocol v1 is an engineering specification, not an owner-approved ADR.
- [Architecture Decision Register](15-architecture-decisions-required.md) — current consolidated owner decisions, deferred engineering specifications and the only remaining owner authorization gate.
- [Agent Handoff](07-agent-handoff.md) — operational constraints and current project state.
- [Pre-Code Implementation Blueprint](../08-development-planning/31-pre-code-implementation-blueprint.md) — M0 boundary, entry/exit criteria and subsequent workstream order.

## Technical specifications

### Audit and licensing
- [Downstream audit](../02-audit-and-licensing/01-downstream-audit.md)
- [License and dependency review](../02-audit-and-licensing/06-license-and-dependency-review.md)

### Linux and platform
- [Linux platform design](../03-platform-and-linux/04-linux-platform-design.md)
- [Runtime and dependency risk](../03-platform-and-linux/11-linux-runtime-and-dependency-risk.md)
- [Credential storage assessment](../03-platform-and-linux/12-linux-credential-storage-assessment.md)
- [Native dependency and architecture matrix](../03-platform-and-linux/13-linux-native-dependency-and-architecture-matrix.md)
- [XDG filesystem and migration](../03-platform-and-linux/17-xdg-filesystem-and-migration-design.md)
- [Linux release and reader acceptance](../03-platform-and-linux/18-linux-release-and-reader-acceptance.md)
- [Security, lifecycle and distribution architecture](../03-platform-and-linux/38-security-lifecycle-and-distribution-architecture.md)

### Core architecture
- [Upstream synchronization strategy](../04-core-architecture/03-upstream-sync-strategy.md)
- [Reader and image pipeline assessment](../04-core-architecture/10-reader-and-image-pipeline-assessment.md)
- [Application architecture](../04-core-architecture/34-application-architecture-blueprint.md)
- [Domain data and persistence](../04-core-architecture/35-domain-data-and-persistence-architecture.md)
- [Desktop UI and navigation](../04-core-architecture/36-desktop-ui-and-navigation-architecture.md)
- [Reader runtime and page pipeline](../04-core-architecture/37-reader-runtime-and-page-pipeline.md)
- [Service integration](../04-core-architecture/37-service-integration-and-feature-orchestration.md)
- [Content-addressed chapter storage](../04-core-architecture/38-content-addressed-storage-and-chapter-manifests.md)
- [XDG storage and download location](../04-core-architecture/39-xdg-storage-layout-and-download-location.md)
- [Linux platform support and packaging matrix](../04-core-architecture/40-linux-platform-support-and-packaging-matrix.md)
- [Application update strategy](../04-core-architecture/41-application-update-and-release-channel-strategy.md)

### Extensions
- [Extension security model](../05-extensions/05-extension-security-model.md)
- [IPC transport and broker design](../05-extensions/26-extension-ipc-transport-and-broker-design.md)
- [Android/Mihon API compatibility matrix](../05-extensions/27-android-mihon-extension-api-compatibility-matrix.md)
- [Broker wire protocol proposal — superseded for v1 framing by protocol v1](../05-extensions/28-broker-wire-protocol-proposal.md)
- [Extension IPC protocol v1](../05-extensions/29-extension-ipc-protocol-v1.md)
- [Extension capability catalogue](../05-extensions/30-extension-capability-catalogue.md)

### Backup
- [Backup interoperability assessment](../06-backup/08-backup-interoperability.md)
- [Backup format and restore safety](../06-backup/14-backup-format-and-restore-safety.md)
- [Mihon backup compatibility architecture](../06-backup/19-mihon-backup-compatibility-architecture.md)
- [Backup workflow and conformance plan](../06-backup/20-backup-workflow-and-conformance-plan.md)
- [Native backup container options](../06-backup/21-native-backup-container-options.md)
- [Native backup format contract draft — superseded by v1 profile](../06-backup/22-native-backup-format-contract-draft.md)
- [Native backup v1 implementation profile](../06-backup/32-native-backup-v1-implementation-profile.md)

### Synchronization
- [Sync architecture assessment](../07-synchronization/09-sync-architecture-assessment.md)
- [Provider-neutral sync protocol](../07-synchronization/16-provider-neutral-sync-protocol.md)
- [Sync provider strategy](../07-synchronization/23-sync-provider-strategy.md)
- [E2EE and key management](../07-synchronization/24-sync-e2ee-and-key-management.md)
- [Self-hosted adapter options](../07-synchronization/25-self-hosted-sync-adapter-options.md)
- [Provider-neutral sync domain protocol](../07-synchronization/33-provider-neutral-sync-domain-protocol.md)

## Documentation status

- Owner-approved decisions are recorded in ADRs and summarized in the decision register.
- Older assessments and proposals remain as design history; where superseded, their status must identify the current authoritative specification.
- Deferred technical choices are not missing owner decisions unless the register explicitly says owner authorization is required.
- M0 has not been authorized or executed. No application implementation, build, dependency installation, extension execution, synchronization, packaging, PR or merge is implied by this documentation.
