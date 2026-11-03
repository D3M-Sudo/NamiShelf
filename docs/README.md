# NamiShelf Documentation Index

Technical documentation is organized by subject. Start here, then use the folder matching the subsystem.

## Directory map

| Directory | Contents |
|---|---|
| [01-project-governance](01-project-governance/) | Architecture decision records, consolidated owner decisions, agent handoff |
| [02-audit-and-licensing](02-audit-and-licensing/) | Downstream audit, licensing and dependency review |
| [03-platform-and-linux](03-platform-and-linux/) | Linux platform design, runtime, credentials, native dependencies, XDG, security lifecycle and distribution architecture |
| [04-core-architecture](04-core-architecture/) | Application architecture, domain/persistence, desktop UI, service orchestration, upstream integration and reader pipeline |
| [05-extensions](05-extensions/) | Extension security, IPC, broker, Android/Mihon API compatibility and capabilities |
| [06-backup](06-backup/) | Native backup, Mihon compatibility, restore safety and conformance |
| [07-synchronization](07-synchronization/) | Provider-neutral sync, E2EE, GitHub and WebDAV |
| [08-development-planning](08-development-planning/) | Pre-code implementation blueprint and milestone sequence |

## Key documents

- [Architecture Decision Record](01-project-governance/02-architecture-decision-record.md) — authoritative ADR history, including ADR-011.
- [Consolidated Architecture Decisions](01-project-governance/15-architecture-decisions-required.md) — owner-confirmed decisions and remaining owner inputs.
- [Agent Handoff](01-project-governance/07-agent-handoff.md) — current constraints and development handoff.
- [Downstream Audit](02-audit-and-licensing/01-downstream-audit.md) — candidate Mihon Desktop audit findings and validation gaps.
- [Extension IPC Protocol v1](05-extensions/29-extension-ipc-protocol-v1.md) — normative v1 wire protocol proposal.
- [Extension Capability Catalogue](05-extensions/30-extension-capability-catalogue.md) — proposed permission profiles and broker policy.
- [Android/Mihon Compatibility Matrix](05-extensions/27-android-mihon-extension-api-compatibility-matrix.md) — extension API compatibility scope.
- [Native Backup v1 Profile](06-backup/32-native-backup-v1-implementation-profile.md) — proposed ZIP/JSON format and restore contract.
- [Provider-Neutral Sync Protocol](07-synchronization/33-provider-neutral-sync-domain-protocol.md) — sync domain, conflict requirements and provider contract.
- [Pre-Code Implementation Blueprint](08-development-planning/31-pre-code-implementation-blueprint.md) — ordered workstreams and M0 gate.
- [Application Architecture Blueprint](04-core-architecture/34-application-architecture-blueprint.md) — module boundaries, dependency rules and lifecycle.
- [Domain Data and Persistence](04-core-architecture/35-domain-data-and-persistence-architecture.md) — logical entities, identities, XDG storage and migration.
- [Desktop UI and Navigation](04-core-architecture/36-desktop-ui-and-navigation-architecture.md) — information architecture, shell, reader UX and state ownership.
- [Service Integration](04-core-architecture/37-service-integration-and-feature-orchestration.md) — source, reader, downloads, tracking, backup, sync and extension workflows.
- [Security, Lifecycle and Distribution](03-platform-and-linux/38-security-lifecycle-and-distribution-architecture.md) — trust zones, credentials, process lifecycle and package gates.

## Documentation status

The preliminary five-area architecture baseline is documented. It remains a proposal pending owner review and M0 feasibility evidence. This repository does not claim that the candidate base has passed Linux feasibility, that the specifications have been implemented, or that security/cryptographic review is complete.

No application code implementation, build, dependency installation, extension execution, sync test, packaging test, PR or merge is implied by the documentation work.
