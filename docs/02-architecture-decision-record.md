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
