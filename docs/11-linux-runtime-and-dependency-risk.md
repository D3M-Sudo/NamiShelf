# NamiShelf — Linux Runtime and Dependency Risk Assessment

**Status:** Static build-configuration review; no dependency resolution or build  
**Reviewed candidate:** `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Date:** 2026-11-03

## 1. Executive assessment

The candidate is a Kotlin Multiplatform repository with a JVM desktop target, but its root build includes Android modules and Android SDK requirements. The README explicitly states that Linux desktop packaging and validation are not configured. A desktop JVM target therefore does not establish that the whole repository can be built or packaged on Linux without Android tooling.

## 2. Observed build constraints

The inspected root settings include Android application/library modules alongside shared KMP modules and `:app-desktop`. The README's source-build prerequisites include JDK 21, Python 3.11+, Android SDK Platform 36, and Build Tools 36.0.0 for Android modules in the shared build.

The desktop Gradle configuration includes a JVM target and desktop dependencies such as Compose Desktop, Skia/Skiko transitively, JNA, SQLDelight's JVM SQLite driver, DEX conversion, APK signature verification, and Rhino. The version catalog also contains Android-oriented libraries and native artifacts. Catalog entries alone do not establish that each is present in the desktop runtime classpath.

The root settings contain JitPack and a locally pinned FlexibleAdapter AAR/POM with SHA-256 checks. This is a positive reproducibility/integrity measure, but Android AAR consumption and local binary provenance remain concerns for a Linux-only desktop distribution.

## 3. Key risks

### LNX-01 — Android build configuration may remain a host prerequisite

The repository includes Android modules and SDK configuration. Determine whether Gradle task configuration for a desktop-only build still requires Android SDK/platform components. Do not assume that invoking a desktop task avoids Android plugin configuration.

### LNX-02 — Native runtime coverage is unknown

Skiko/Skia, SQLite, JNA, and other native dependencies must be verified for x86_64 and aarch64 Linux, including glibc and system-library requirements.

### LNX-03 — Dependency catalog is not an SBOM

The version catalog contains Android-only and desktop dependencies. Resolve the actual desktop runtime graph, distinguish compile/test/build-only dependencies, and generate an SBOM before making packaging decisions.

### LNX-04 — Repository topology couples desktop to Android

A shared root build may increase Linux build complexity and maintenance. Assess whether desktop-only configuration can be made reliable without destabilizing Android modules. Do not split repositories or modules without a separate architecture decision.

### LNX-05 — JitPack/local binary supply chain

Review immutable coordinates, checksums, source availability, licensing, and whether the desktop artifact graph can exclude Android-only binaries such as AARs. Retain checksum verification for pinned binaries.

## 4. Required technical investigation

1. Inspect Gradle plugin application and configuration-time task dependencies.
2. Establish the minimum JDK and whether Android SDK is required for desktop-only build.
3. Resolve desktop compile/runtime/test graphs separately.
4. Generate x86_64 and aarch64 Linux dependency inventories.
5. Inspect native libraries and minimum glibc/system dependencies.
6. Test a clean, offline/reproducible build after dependency cache policy is defined.
7. Compare AppImage, Flatpak, and DEB runtime strategies.
8. Confirm no Android-only AAR/native artifacts leak into desktop runtime packaging.
9. Add Linux CI for at least one supported architecture, then expand architecture coverage.
10. Review dependency vulnerabilities and licenses against the resolved artifacts.

## 5. Acceptance gates

| Gate | Required evidence |
|---|---|
| Desktop-only configuration | Gradle config/build without Android SDK, or documented minimum SDK prerequisite |
| Linux x86_64 | Clean build and smoke test |
| Linux aarch64 | Clean build and smoke test or explicit support deferral |
| Native libraries | Complete inventory and loader verification |
| Dependency graph | SBOM and vulnerability review |
| Packaging | AppImage, Flatpak, DEB install/upgrade/uninstall tests |
| CI | Reproducible Linux workflow |
| Legal | License/NOTICE verification for actual shipped artifacts |

## 6. Recommendation

Do not select packaging technology or promise architecture coverage yet. First establish a desktop-only build boundary and inspect the resolved Linux runtime graph. Keep all three packaging targets as product goals, subject to feasibility and maintenance cost.

## 7. Source references

- [Mihon Desktop README](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/README.md)
- [Root settings.gradle.kts](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/settings.gradle.kts)
- [Desktop build.gradle.kts](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/build.gradle.kts)
- [Gradle version catalog](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/gradle/libs.versions.toml)
