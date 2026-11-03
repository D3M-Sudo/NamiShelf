# Mihon Desktop — Initial Read-Only Downstream Audit

**Repository:** `AltairCardinal/mihon-desktop`  
**Reviewed ref:** `main`  
**Observed commit:** `701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Review date:** 2026-11-03  
**Mode:** Static/read-only; no build or tests executed

## Executive assessment

Mihon Desktop is a substantially more advanced starting point than a greenfield desktop port: it contains a JVM desktop application, reader, library/source integration, downloads, backup, history, tracking-related workflows, and a desktop extension adapter.

It is not yet a validated Linux desktop distribution. The inspected README identifies Windows as the primary target, macOS as an additional build path, and Linux packaging/validation as unconfigured. The inspected desktop CI workflow is manually dispatched and runs on Windows.

The conditional downstream-base decision remains appropriate. Production-base acceptance requires closure of Linux portability, extension security/compatibility, dependency, licensing, backup, and maintenance gates.

## Findings

### DSA-01 — Linux is not supported/validated in the inspected baseline

**Severity:** High for NamiShelf target; **confidence:** High.

Evidence: README states Linux packaging and validation are not configured; desktop Gradle build declares a Kotlin/JVM target; inspected desktop CI runs on `windows-2022`.

Impact: JVM targeting does not prove Linux readiness. Native libraries, file chooser integration, desktop registration, window behavior, filesystem semantics, image codecs, SQL drivers, and packaging may differ.

Required evidence:
1. Linux CI and reproducible build.
2. Runtime smoke tests on X11 and Wayland where supported.
3. File chooser, URL opening, notifications, desktop integration.
4. AppImage, Flatpak, and DEB install/upgrade/uninstall validation.

### DSA-02 — Extension execution is in-process in the candidate

**Severity:** Critical design risk for untrusted extensions; **confidence:** High.

Evidence: `DesktopExtensionLoader` uses an `ExtensionClassLoader`, `ServiceLoader`, reflection, and class scanning; `DesktopExtensionManager` owns runtime lifecycle. Artifact authentication checks repository fingerprints/signatures.

Impact: A JVM classloader is not a security sandbox. Code in the same process may inherit filesystem, environment, network, memory, and application-secret access. Signatures prove provenance/integrity relative to a key, not benign behavior.

NamiShelf owner decision: replace this execution boundary with one process per extension and NamiShelf-mediated data access over a narrow, versioned IPC/broker API.

### DSA-03 — DEX-to-JVM conversion does not guarantee Android API compatibility

**Severity:** High; **confidence:** High.

Evidence: `ApkToJarConverter.kt` converts DEX bytecode to JVM JAR form; desktop compatibility stubs/adapters are described in project documentation.

Impact: Conversion does not reproduce Android framework classes or runtime semantics, including lifecycle, preferences, WebView/CookieManager, URI/filesystem, threading, and UI services.

Owner decision: converted Android APK extensions are required in the first release. NamiShelf must define a supported Android/Mihon/Tachiyomi API subset, implement modular adapters, and test representative real APK fixtures. Do not claim universal Android compatibility.

### DSA-04 — Upstream and desktop-specific behavior need provenance tracking

**Severity:** Medium; **confidence:** High.

Evidence: `docs/roadmap/source-extension-authority-baseline.md` distinguishes upstream behavior, desktop adapters, security enhancements, and technical debt.

Impact: Downstream-only behavior must not be mislabeled as upstream behavior; Android-first upstream changes can break desktop adaptations.

Required control: feature-level provenance map and impact review for each synchronization batch.

### DSA-05 — README and release metadata need reconciliation

**Severity:** Low; **confidence:** Medium.

Evidence: README describes an early-development state and absence of public release packages, while prior repository-level observations recorded a Windows desktop release. Re-check current releases and assets before relying on maturity claims.

## Positive controls observed

- Desktop CI declares `permissions: contents: read`.
- Third-party GitHub Actions are pinned to full commit SHAs.
- FlexibleAdapter artifact is pinned with explicit SHA-256 verification.
- Extension signer fingerprints and JAR signature checks are present.
- Installation code includes transaction/rollback lifecycle concepts.
- Documentation records known compatibility debt.

These controls do not close the extension execution risk or prove Linux readiness.

## Review limitations

No build, tests, dependency resolution, SBOM, advisory scan, full transitive license review, Linux packaging/runtime validation, malicious extension fixtures, or exhaustive review of backup/sync/storage/reader/network paths was performed.

## Acceptance gates

| Gate | Required outcome | Status |
|---|---|---|
| Linux | Reproducible build and runtime smoke suite | Open |
| Extension isolation | Per-extension process + brokered data access | Direction approved; implementation open |
| Extension compatibility | Android/Mihon API matrix and real APK fixtures | Open |
| Dependencies | Resolved runtime graph, SBOM, advisory review | Open |
| Legal provenance | Licenses, NOTICE, assets/fonts review | Open |
| Upstream provenance | Feature-level baseline | Partially documented |
| Backup | Mihon import/export and round-trip tests | Open |
| Product identity | NamiShelf branding and attribution review | Open |
| Maintenance | Upstream delta and synchronization cost | Open |

## Source references

- [Mihon Desktop README](https://github.com/AltairCardinal/mihon-desktop/blob/main/README.md)
- [Desktop Gradle build](https://github.com/AltairCardinal/mihon-desktop/blob/main/app-desktop/build.gradle.kts)
- [Desktop CI workflow](https://github.com/AltairCardinal/mihon-desktop/blob/main/.github/workflows/desktop-ci.yml)
- [Desktop extension loader](https://github.com/AltairCardinal/mihon-desktop/blob/main/app-desktop/src/main/kotlin/mihon/desktop/extension/DesktopExtensionLoader.kt)
- [Desktop artifact authenticator](https://github.com/AltairCardinal/mihon-desktop/blob/main/app-desktop/src/main/kotlin/mihon/desktop/extension/DesktopArtifactAuthenticator.kt)
- [Evolution plan](https://github.com/AltairCardinal/mihon-desktop/blob/main/EVOLUTION_PLAN.md)
