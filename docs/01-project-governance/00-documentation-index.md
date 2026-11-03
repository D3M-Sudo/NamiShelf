# NamiShelf Documentation Index

All documentation is organized in thematic subdirectories. This index is the entry point for the documentation tree.

## Directory map

| Directory | Scope |
|---|---|
| [01-project-governance](./) | ADR history, decision register, handoff and documentation governance |
| [02-audit-and-licensing](../02-audit-and-licensing/) | Downstream audit, licensing and dependency review |
| [03-platform-and-linux](../03-platform-and-linux/) | Linux platform, runtime, credentials, native dependencies, XDG, security and distribution |
| [04-core-architecture](../04-core-architecture/) | Application architecture, persistence, UI, reader, storage and integration |
| [05-extensions](../05-extensions/) | Extension security, IPC, broker, Android/Mihon compatibility and capabilities |
| [06-backup](../06-backup/) | Native backup, Mihon interoperability, restore safety and conformance |
| [07-synchronization](../07-synchronization/) | Provider-neutral sync, E2EE, GitHub and WebDAV |
| [08-development-planning](../08-development-planning/) | Pre-M0 blueprint, implementation order and acceptance gates |

## Governance and planning

- [Architecture Decision Record](02-architecture-decision-record.md) — append-only approved decisions ADR-001 through ADR-027. ADR-011 is intentionally unassigned; no approved ADR currently uses that number.
- [Architecture Decision Register](15-architecture-decisions-required.md) — consolidated decisions, deferred engineering details, remaining owner gate.
- [Agent Handoff](07-agent-handoff.md) — repository constraints and current project state.
- [Pre-Code Implementation Blueprint](../08-development-planning/31-pre-code-implementation-blueprint.md) — M0 scope, exit criteria and later workstream sequence.

## Audit and licensing

- [Downstream audit](../02-audit-and-licensing/01-downstream-audit.md)
- [License and dependency review](../02-audit-and-licensing/06-license-and-dependency-review.md)

## Platform and Linux

- [Linux platform design](../03-platform-and-linux/04-linux-platform-design.md)
- [Runtime and dependency risk](../03-platform-and-linux/11-linux-runtime-and-dependency-risk.md)
- [Credential storage assessment](../03-platform-and-linux/12-linux-credential-storage-assessment.md)
- [Native dependency and architecture matrix](../03-platform-and-linux/13-linux-native-dependency-and-architecture-matrix.md)
- [XDG filesystem and migration](../03-platform-and-linux/17-xdg-filesystem-and-migration-design.md)
- [Linux release and reader acceptance](../03-platform-and-linux/18-linux-release-and-reader-acceptance.md)
- [Security, lifecycle and distribution architecture](../03-platform-and-linux/38-security-lifecycle-and-distribution-architecture.md)

## Core architecture

- [Upstream synchronization strategy](../04-core-architecture/03-upstream-sync-strategy.md)
- [Reader and image pipeline assessment](../04-core-architecture/10-reader-and-image-pipeline-assessment.md)
- [Application architecture blueprint](../04-core-architecture/34-application-architecture-blueprint.md)
- [Domain data and persistence](../04-core-architecture/35-domain-data-and-persistence-architecture.md)
- [Desktop UI and navigation](../04-core-architecture/36-desktop-ui-and-navigation-architecture.md)
- [Reader runtime and page pipeline](../04-core-architecture/37-reader-runtime-and-page-pipeline.md)
- [Service integration and orchestration](../04-core-architecture/37-service-integration-and-feature-orchestration.md)
- [Content-addressed storage and chapter manifests](../04-core-architecture/38-content-addressed-storage-and-chapter-manifests.md)
- [XDG storage and download location](../04-core-architecture/39-xdg-storage-layout-and-download-location.md)
- [Linux platform support and packaging matrix](../04-core-architecture/40-linux-platform-support-and-packaging-matrix.md)
- [Application update and release channels](../04-core-architecture/41-application-update-and-release-channel-strategy.md)

## Extensions

- [Extension security model](../05-extensions/05-extension-security-model.md)
- [Extension IPC transport and broker design](../05-extensions/26-extension-ipc-transport-and-broker-design.md)
- [Android/Mihon extension API compatibility matrix](../05-extensions/27-android-mihon-extension-api-compatibility-matrix.md)
- [Broker wire protocol proposal](../05-extensions/28-broker-wire-protocol-proposal.md) — historical proposal; superseded where it conflicts with protocol v1.
- [Extension IPC protocol v1](../05-extensions/29-extension-ipc-protocol-v1.md) — engineering baseline; cryptographic attestation and OS sandbox are out of scope.
- [Extension capability catalogue](../05-extensions/30-extension-capability-catalogue.md) — owner-approved profiles under ADR-025.

## Backup

- [Backup interoperability assessment](../06-backup/08-backup-interoperability.md)
- [Backup format and restore safety](../06-backup/14-backup-format-and-restore-safety.md)
- [Mihon backup compatibility architecture](../06-backup/19-mihon-backup-compatibility-architecture.md)
- [Backup workflow and conformance plan](../06-backup/20-backup-workflow-and-conformance-plan.md)
- [Native backup container options](../06-backup/21-native-backup-container-options.md)
- [Native backup format contract draft](../06-backup/22-native-backup-format-contract-draft.md) — historical draft; ADR-026 and document 32 take precedence for encrypted-backup recovery.
- [Native backup v1 implementation profile](../06-backup/32-native-backup-v1-implementation-profile.md)

## Synchronization

- [Sync architecture assessment](../07-synchronization/09-sync-architecture-assessment.md)
- [Provider-neutral sync protocol](../07-synchronization/16-provider-neutral-sync-protocol.md)
- [Sync provider strategy](../07-synchronization/23-sync-provider-strategy.md)
- [E2EE and key management](../07-synchronization/24-sync-e2ee-and-key-management.md)
- [Self-hosted sync adapter options](../07-synchronization/25-self-hosted-sync-adapter-options.md)
- [Provider-neutral sync domain protocol](../07-synchronization/33-provider-neutral-sync-domain-protocol.md)

## Document authority and status rules

1. The latest applicable owner-approved ADR is authoritative for product/security decisions.
2. The architecture decision register is the consolidated index of owner choices and remaining owner authorization.
3. Current implementation profiles/protocol specifications refine decisions but cannot weaken or silently replace an ADR.
4. Earlier assessments and drafts remain as history; they must identify later superseding decisions where conflicts exist.
5. Deferred engineering details are not unresolved owner decisions unless the register explicitly marks them as requiring owner approval.
6. M0 has not been authorized or executed. No implementation, build, dependency installation, extension execution, synchronization, package creation, PR or merge is implied.
