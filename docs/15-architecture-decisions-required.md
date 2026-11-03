# NamiShelf — Architecture Decisions Required Before Implementation

**Status:** Decision brief for owner review  
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

**Owner decision required:** choose:
- **A — GitHub-first:** adopt the candidate protocol initially and document GitHub as a hard service dependency.
- **B — Provider-neutral:** define protocol/provider boundary now; GitHub is one adapter.
- **C — Defer sync:** ship local-only first and add sync after initial release.

Suggested default: **B**, if sync is a first-release feature; otherwise **C** is safer than coupling the product to a protocol without a committed requirement.

## 3. Backup format extension

**Evidence:** Mihon uses a canonical gzip/protobuf backup. Candidate desktop adds optional data such as author archive state. The codec accepts gzip or plain protobuf and decompresses the entire stream to memory without a visible explicit compressed/decompressed byte cap. Restore is per-item and can be partial. Imported source preferences and extension repositories can influence later network and extension behavior.

**Recommendation:** preserve canonical Mihon `.tachibk` interoperability unchanged. Put NamiShelf-only data in a separately versioned companion manifest/archive unless a protobuf extension is proven to preserve unknown fields through Mihon round-trips. Do not silently extend core protobuf fields.

**Owner decision required:** choose:
- **A — Strict Mihon-compatible only:** no NamiShelf-only fields in the first release.
- **B — Companion extension:** standard `.tachibk` plus optional versioned NamiShelf companion data.
- **C — Separate NamiShelf archive:** independent format with explicit Mihon import/export conversion.

Suggested default: **B**, while keeping the Mihon backup usable without the companion data.

**Mandatory regardless of choice:** enforce compressed and decompressed size limits, bounded parsing, pre-restore preview/checkpoint, and explicit partial-restore reporting before release.

## 4. Linux filesystem layout and migration

**Evidence:** candidate Linux path resolution currently uses legacy `~/.mihon` rather than XDG config/data/cache/state locations. Credential wrapping-key identity is derived from the canonical secure-store directory path, so moving the directory without migrating its OS credential item makes encrypted sync records unreadable.

**Recommendation:** use XDG directories for new NamiShelf installations, with explicit migration from legacy Mihon Desktop paths. Separate config, data, cache, state, logs, downloads, extensions, and backups. Migration must treat secure credential material specially; do not copy ciphertext alone or silently generate a new wrapping key.

**Owner decision required:** choose:
- **A — Clean NamiShelf profile:** new XDG paths; import user data via backup/migration wizard.
- **B — In-place migration:** migrate existing Mihon Desktop data automatically on first run.
- **C — Compatibility mode:** preserve legacy paths initially, move to XDG in a later release.

Suggested default: **A** for an independent product, with an explicit opt-in migration/import assistant.

## 5. Linux architectures and packaging release policy

**Evidence:** candidate declares Windows MSI and macOS DMG, not Linux packaging. Native-sensitive components include Skiko/Skia, SQLite JDBC, JNA, SevenZipJBinding, and filesystem watching. No resolved Linux runtime graph or complete x86_64/aarch64 artifact inventory was produced.

**Recommendation:** make Linux x86_64 the first release gate on Mint 22.3/Ubuntu base. Keep aarch64 as a planned target but do not promise it until all native dependencies and package formats are validated. Keep AppImage, Flatpak, and DEB as goals, but permit release-format staging if one cannot meet security/maintenance criteria.

**Owner decision required:** is aarch64 required for the first public release or can it follow after x86_64? Suggested default: **x86_64 first; aarch64 after validation**.

## 6. Extension sandbox commitment

The owner already approved one process per extension, brokered data access, DEX-to-JVM support, and third-party extensions in release 1. The remaining OS-level sandbox mechanism is conditional on sustainability.

**Recommendation:** preserve the current conditional commitment. Do not claim that process separation alone is an OS sandbox. Require a documented security posture for each package format and a later explicit release decision if full OS sandboxing cannot be delivered.

## 7. Reader performance budgets

**Evidence:** candidate has Skia decode/downsampling/region decode and memory accounting concepts, but no Linux measurements on NamiShelf's target hardware.

**Recommendation:** define acceptance budgets before optimization decisions: startup, first-page latency, decoded-image memory, cache memory, prefetch concurrency, and post-chapter memory release. Use the primary Linux Mint 22.3 environment as the reference profile; do not infer performance from code comments.

**Owner input required:** whether first-release reader acceptance should prioritize low-memory stability over aggressive prefetch/performance. Suggested default: **stability and bounded memory first**.

## 8. Implementation readiness gates

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
