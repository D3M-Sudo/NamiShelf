# NamiShelf — Linux Native Dependency and Architecture Matrix

**Status:** Static dependency/build configuration review; no dependency resolution or package builds  
**Reviewed candidate:** `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Date:** 2026-11-03

## 1. Executive assessment

The desktop target uses `compose.desktop.currentOs`, so native desktop artifacts are selected for the build host. The inspected packaging configuration declares only Windows MSI and macOS DMG formats. No Linux packaging target or Linux architecture matrix is established in the reviewed build configuration.

The candidate includes multiple native-sensitive components. Linux x86_64 should be the first validation target because it aligns with the primary desktop use case; aarch64 must remain an explicit unverified target until every native dependency and package format is evidenced.

## 2. Native-sensitive components observed

| Component | Desktop use | Linux validation required |
|---|---|---|
| Compose Desktop / Skiko / Skia | Windowing and image rendering | Native artifacts, graphics stack, glibc baseline, X11/Wayland |
| SQLDelight SQLite JDBC driver | Persistent database | SQLite native/runtime loading, architecture |
| JNA + JNA Platform | OS integration | Native library loading, architecture |
| SevenZipJBinding + all-platforms | RAR/CBR archive extraction | Linux native payload availability, x86_64/aarch64 |
| Directory Watcher | Filesystem monitoring | inotify behavior, resource limits |
| DEX conversion / APK signature verification | Extension conversion and verification | JVM-only vs native transitive dependencies |
| OkHttp and compression modules | Network and compression | TLS/native provider behavior, architecture |
| Secret Service integration | Credentials | `secret-tool`, D-Bus session, provider and sandbox permissions |

The table identifies review targets, not a completed native artifact inventory.

## 3. Build configuration observations

- `app-desktop` declares a Kotlin/JVM target.
- Desktop dependencies include `compose.desktop.currentOs`, JNA, SQLDelight's JDBC SQLite driver, Skia transitively, SevenZipJBinding platform artifacts, and filesystem watcher.
- Packaging declares `TargetFormat.Dmg` and `TargetFormat.Msi`; no Linux target format is configured in that block.
- Root settings include Android modules and Android plugin configuration alongside the desktop module.
- The root build includes JitPack and a local pinned FlexibleAdapter AAR/POM. The AAR is Android-specific and its desktop runtime/build impact must be checked through actual resolved configurations.
- No resolved desktop runtime dependency graph or Linux SBOM was produced in this phase.

## 4. Architecture coverage

| Architecture | Current evidence | NamiShelf recommendation |
|---|---|---|
| Linux x86_64 | No Linux package/runtime validation in reviewed evidence | First Linux release-validation target |
| Linux aarch64 | No complete native dependency/package evidence | Keep as planned/conditional; do not promise until validated |
| Windows x86_64 | Candidate packaging target exists | Not a NamiShelf priority |
| macOS | Candidate DMG target exists | Outside initial NamiShelf platform scope |

## 5. Required dependency evidence

For each Linux architecture, generate the resolved runtime graph and SBOM, identify native artifacts and embedded binaries, verify checksums/provenance/licenses, inspect ELF dependencies and glibc requirements, and test package contents on a clean target image.

Specific checks:
1. Skiko/Skia artifact classifier and required system libraries.
2. SQLite JDBC native binary classifier and loading behavior.
3. JNA native payload and any architecture-specific library.
4. SevenZipJBinding native payload coverage and extraction smoke fixtures.
5. Directory watcher Linux backend and descriptor/resource behavior.
6. TLS and compression behavior without accidental Android-only native dependencies.
7. APK/DEX conversion and signature verification on a clean JVM.
8. Flatpak runtime compatibility and portal/D-Bus permissions.
9. DEB dependency metadata and minimum glibc/system package baseline.
10. AppImage self-contained runtime assumptions and external library requirements.

## 6. Linux filesystem path finding

The inspected `DesktopPlatformPaths.resolve` uses `~/.mihon` for non-Windows platforms, including Linux. It does not implement XDG_CONFIG_HOME, XDG_DATA_HOME, XDG_CACHE_HOME, or XDG_STATE_HOME. This is a confirmed platform-design gap relative to NamiShelf's approved Linux design.

NamiShelf should introduce an explicit platform path adapter and a documented migration from any inherited legacy `.mihon` data. Do not simply rename or move the directory: database, cookies, downloads, extension data, logs, and encrypted credential wrapping-key identity have different migration constraints.

## 7. Recommended initial support policy

- **Release gate 1:** Linux x86_64 on Linux Mint 22.3 / supported Ubuntu base.
- **Release gate 2:** Debian and Arch-based smoke coverage after x86_64 baseline.
- **Release gate 3:** aarch64 only after native dependency, packaging, and real hardware/VM validation.
- Keep all requested package formats as goals, but publish only those with reproducible clean-install/upgrade/uninstall evidence.

## 8. Source references

- [Desktop build.gradle.kts](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/build.gradle.kts)
- [Root settings.gradle.kts](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/settings.gradle.kts)
- [Gradle version catalog](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/gradle/libs.versions.toml)
- [DesktopPlatformPaths.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/platform/DesktopPlatformPaths.kt)
