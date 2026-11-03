# NamiShelf — Architecture Decisions Required Before Implementation

**Status:** Consolidated decision register; remaining choices only  
**Date:** 2026-11-03  
**Scope:** Product/architecture choices only; no implementation authorized

This document consolidates decisions that cannot be settled safely by static inspection alone. Recommendations are proposals, not accepted decisions.

## 1. Linux credential storage behavior

**Evidence:** the candidate uses Secret Service through `secret-tool`, encrypted AES-GCM records, and a separately stored wrapping key. It intentionally has no plaintext fallback. The inspected desktop integration test exercises native credential round-trip on Windows/macOS, but skips Linux; Linux tests use a fake command runner.

**Recommendation:** keep Secret Service as the persistent credential backend; fail closed when unavailable. Provide an explicit “secure credential service unavailable” state and disable persistent sync authorization rather than downgrade to plaintext. Decide whether an optional session-only credential mode is desirable; default recommendation is no session-only mode in the first release because it complicates refresh/restart semantics.

**Owner decision required:** Is sync allowed to operate only when an OS credential service is available, with no fallback? Proposed default: **yes**.

## 2. Sync provider and protocol

**Evidence:** candidate sync uses a custom GitHub private-repository protocol. It is not compatible with upstream Mihon clients and introduces GitHub account, API, repository, and token lifecycle dependencies.

**Recommendation:** define a provider-neutral NamiShelf sync domain/protocol and keep transport behind an adapter. GitHub may be the first provider, but do not make GitHub repository layout or Git object transport the public protocol contract. This preserves the option of self-hosting or another provider later, at additional initial design cost.

**Owner decision:** **B — Provider-neutral**, confirmed on 2026-11-03. NamiShelf will define a provider-independent sync domain/protocol and isolate transports behind adapters. GitHub may be an initial adapter, but its repository layout and Git object model will not define the public protocol. Sync is included in the first public release, as confirmed 2026-11-03. GitHub is the first provider adapter; self-hosting is also required in release 1, so a second self-hosted adapter is needed.

## 3. Backup compatibility architecture

**Owner decisions:** confirmed on 2026-11-03.

- **Native format:** NamiShelf-native, versioned format is the default; Mihon formats are interoperability adapters.
- **Container:** ZIP; use ZIP64 extensions where required for large archives.
- **Structured payload:** canonical JSON.
- **Encryption:** optional per backup.
- **Library selection:** complete library selected by default.
- **Settings:** portable settings selected by category.
- **Legacy Mihon/Tachiyomi:** both .tachibk import and export required in release 1; export must disclose omitted and approximated fields.
- **Downloads:** separate export/migration operation.
- **Future Mihon format:** implement an adapter only after official format and implementation are inspectable and fixture-testable.

Remaining design tasks: canonical JSON profile, manifest schema, ZIP feature subset, integrity/authentication model, encryption/KDF/recovery profile, settings allowlist, download archive layout, and legacy export loss-report UX.

Mandatory safeguards: bounded streaming, compressed/decompressed limits, pre-restore preview, recovery checkpoint, and explicit partial-restore reporting.

Detailed design: [19 — Mihon backup compatibility](19-mihon-backup-compatibility-architecture.md), [20 — Backup workflow and conformance](20-backup-workflow-and-conformance-plan.md), [21 — Native container options](21-native-backup-container-options.md).

## 4. Linux filesystem layout and migration

**Evidence:** candidate Linux path resolution currently uses legacy `~/.mihon` rather than XDG config/data/cache/state locations. Credential wrapping-key identity is derived from the canonical secure-store directory path, so moving the directory without migrating its OS credential item makes encrypted sync records unreadable.

**Recommendation:** use XDG directories for new NamiShelf installations, with explicit migration from legacy Mihon Desktop paths. Separate config, data, cache, state, logs, downloads, extensions, and backups. Migration must treat secure credential material specially; do not copy ciphertext alone or silently generate a new wrapping key.

**Owner decision:** **A — Clean NamiShelf profile**, confirmed 2026-11-03. Use new XDG paths and an explicit opt-in import/migration assistant; never silently adopt or mutate an existing Mihon Desktop profile.

## 5. Linux architectures and packaging release policy

**Evidence:** candidate declares Windows MSI and macOS DMG, not Linux packaging. Native-sensitive components include Skiko/Skia, SQLite JDBC, JNA, SevenZipJBinding, and filesystem watching. No resolved Linux runtime graph or complete x86_64/aarch64 artifact inventory was produced.

**Recommendation:** make Linux x86_64 the first release gate on Mint 22.3/Ubuntu base. Keep aarch64 as a planned target but do not promise it until all native dependencies and package formats are validated. Keep AppImage, Flatpak, and DEB as goals, but permit release-format staging if one cannot meet security/maintenance criteria.

**Owner decision:** x86_64 first, aarch64 after validation, confirmed 2026-11-03. AppImage, Flatpak, and DEB remain packaging goals, subject to individual release gates.

## 6. Extension sandbox commitment

The owner already approved one process per extension, brokered data access, DEX-to-JVM support, and third-party extensions in release 1. The remaining OS-level sandbox mechanism is conditional on sustainability.

**Recommendation:** preserve the current conditional commitment. Do not claim that process separation alone is an OS sandbox. Require a documented security posture for each package format and a later explicit release decision if full OS sandboxing cannot be delivered.

## 7. Reader performance budgets

**Evidence:** candidate has Skia decode/downsampling/region decode and memory accounting concepts, but no Linux measurements on NamiShelf's target hardware.

**Recommendation:** define acceptance budgets before optimization decisions: startup, first-page latency, decoded-image memory, cache memory, prefetch concurrency, and post-chapter memory release. Use the primary Linux Mint 22.3 environment as the reference profile; do not infer performance from code comments.

**Owner decision:** prioritize stability and bounded memory over aggressive prefetch, confirmed 2026-11-03. Numeric budgets must be measured on the Linux Mint 22.3 reference hardware before being fixed.

## 8. Remaining owner decisions and implementation readiness gates

The candidate remains a conditional base. Before implementation freeze:
1. Static security review of extension adapters and broker design.
2. Linux-only build feasibility and resolved dependency/SBOM review.
3. Native Linux credential integration evidence.
4. Backup resource limits and cross-version fixture plan.
5. XDG path/migration decision.
6. Sync provider decision if sync is in first-release scope.
7. First-release architecture coverage and package policy.
8. Reader memory/performance budgets.

No build, test, dependency resolution, installation, application coding, or repository changes outside the NamiShelf documentation branch have been performed in this phase.
