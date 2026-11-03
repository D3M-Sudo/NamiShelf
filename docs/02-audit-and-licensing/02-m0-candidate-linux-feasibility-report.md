# M0 — Mihon Desktop Linux Feasibility Report

**Assessment date:** 2026-11-03  
**Status:** Static source/repository review completed; local build and runtime validation blocked by execution-environment network isolation  
**Candidate:** `AltairCardinal/mihon-desktop`  
**Candidate ref:** `main`  
**Pinned candidate commit:** `701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Upstream Mihon reference commit:** `55be95dd5df7ac985bbc68ea62a5a525611a732f`  
**Assessment host:** Debian GNU/Linux 13 (trixie), x86_64, OpenJDK 21.0.12  
**Scope:** Unmodified candidate only. No source changes, fork, packaging, PR or merge.

## 1. Executive assessment

**Preliminary result: CONDITIONAL GO for further technical reuse evaluation; NO-GO for accepting this candidate as a Linux-ready release-1 base without a dedicated Linux enablement workstream.**

The candidate has a real Kotlin/JVM Compose Desktop application module and some Linux-specific adapters (Secret Service, URI registration, Linux desktop-entry resource). This is materially more than an Android-only codebase.

However, Linux is not a configured desktop build/release target:
- the documented desktop build entrypoint explicitly rejects Linux;
- Compose native distribution targets are DMG and MSI only;
- desktop CI runs on Windows, while the Ubuntu workflow is Android-focused and does not validate the desktop application;
- the README explicitly says Linux packaging and validation are not configured;
- the desktop data-path implementation uses the legacy `~/.mihon` directory on Linux instead of the approved NamiShelf XDG layout and clean profile boundary.

The codebase therefore cannot be accepted unchanged as NamiShelf's Linux application. Whether its shared domain/reader/UI code is worth adapting remains plausible, but must be established with a reproducible Linux build and focused runtime evidence.

## 2. Source identity and provenance

- Candidate repository: <https://github.com/AltairCardinal/mihon-desktop>
- Pinned candidate commit: <https://github.com/AltairCardinal/mihon-desktop/commit/701080c251c8fd8b35d0b57a481d93b9a9eb3701>
- Upstream repository: <https://github.com/mihonapp/mihon>
- Upstream comparison reference: `55be95dd5df7ac985bbc68ea62a5a525611a732f`
- GitHub compare reports the candidate as diverged from that upstream reference, 2,185 commits ahead and 275 behind at the pinned refs. This is a substantial downstream delta; NamiShelf must not treat the candidate as a thin or automatically maintainable fork.
- Candidate repository metadata identifies Apache-2.0 as its repository license. The candidate README describes it as an independently maintained downstream project based on Mihon.

The pinned commit is unsigned according to the GitHub commit verification metadata (`verified=false`, reason `unsigned`). This is a provenance observation, not evidence of malicious content. NamiShelf would need its own source pinning, integrity and attribution process.

## 3. Build system and application structure

### Confirmed

- Gradle wrapper is configured for Gradle 8.14.4.
- The desktop application is `:app-desktop`, Kotlin Multiplatform with a JVM target.
- Desktop UI uses Compose Multiplatform; the dependency is selected with `compose.desktop.currentOs`.
- The desktop module consumes shared domain/data/presentation/source modules and includes desktop-specific adapters.
- The version catalog declares Compose Multiplatform 1.10.2, SQLDelight 2.2.1 and SQLite 2.6.2.
- The project pins JDK 21 in `.github/.java-version`.
- The README additionally requires Python 3.11+ and Android SDK Platform 36 / Build Tools 36.0.0 for the shared build.
- Desktop dependency notices are generated for the JVM source set by a dedicated AboutLibraries Gradle plugin. A desktop test asserts a packaged list of 203 notices, including license text checks. This is useful infrastructure, but actual output and completeness were not regenerated in this assessment.

### Linux blockers

- `scripts/build-desktop.sh` has an explicit `Linux*` branch that prints “Linux desktop packaging is not configured” and exits non-zero.
- `app-desktop/build.gradle.kts` configures only `TargetFormat.Dmg` and `TargetFormat.Msi`; no Linux DEB/RPM/AppImage/native target is configured.
- The desktop CI workflow uses `windows-2022` and is `workflow_dispatch` only.
- The Ubuntu Build & Test workflow builds Android and runs Android/unit-test gates; it does not build or run `:app-desktop:jvmTest` on Linux.
- The README says Linux desktop packaging and validation are not configured.

## 4. Linux platform code already present

| Capability | Source evidence | Assessment |
|---|---|---|
| Secret Service | `DesktopCredentialStore.kt` uses `secret-tool store/lookup/clear` | A real Linux adapter exists and uses stdin for secret material. Runtime integration still unverified. |
| URI registration | `DesktopUriSchemeRegistration.kt` writes a user desktop entry, then invokes `update-desktop-database` and `xdg-mime` | Linux integration exists, but requires packaged jpackage layout and external commands. |
| Desktop entry | `platform/linux/mihon-desktop.desktop` | Resource exists; not evidence of an installable Linux package. |
| Filesystem watching | Desktop build references `io.methvin:directory-watcher` with comment identifying inotify on Linux | Dependency is cross-platform; actual runtime behavior unverified. |
| Native libraries | JNA/JNA-platform and 7-Zip JNI bindings are in the desktop dependency set | Linux native artifact availability/ABI and packaging were not verified. |
| File selection | Swing `JFileChooser` implementation | Likely functional under a graphical JVM, but desktop portal integration and sandbox behavior are not demonstrated. |

## 5. Confirmed architecture mismatch: storage paths

`DesktopPlatformPaths.resolve()` uses `~/.mihon` as both config and local root for every non-Windows platform (with a separate macOS logs path). On Linux, this places the database, network cache, cookies, downloads, extensions, covers, logs and backups under the legacy hidden directory.

This conflicts with NamiShelf's approved requirements:
- independent NamiShelf application identity and clean profile;
- XDG config/data/cache separation;
- permanent downloads independently selectable by the user;
- cache cleanup must not remove permanent offline content;
- no silent adoption or mutation of the existing Mihon profile.

This is a source-level incompatibility, not a runtime test result. The candidate's recovery-profile mechanism may provide useful isolation primitives, but it does not make the default Linux path policy compliant.

## 6. Local execution environment and attempted validation

The available execution container reports:
- Debian 13 (trixie), x86_64;
- OpenJDK 21.0.12;
- Python 3.13.5;
- GTK 3, X11, Xext, Xrender, Xtst, Xi, FreeType and Fontconfig shared libraries present;
- no Gradle executable or pre-populated Gradle cache;
- no Android SDK at the checked standard locations and no `ANDROID_HOME` / `ANDROID_SDK_ROOT`;
- no `secret-tool` or `update-desktop-database` command;
- `xdg-mime` is present.

A shallow clone was attempted from the public candidate repository. It failed before transfer because the execution container could not resolve `github.com` (`Could not resolve host: github.com`). The codeload archive route was also unavailable from this execution environment.

Consequently:
- the candidate source tree was not materialized locally;
- Gradle dependency resolution was not attempted;
- no build was run;
- no application launch was attempted;
- no candidate tests were executed;
- no runtime behavior is marked passed.

This is an environment/network blocker, not a build failure attributable to candidate source.

## 7. CI and release evidence

- The current desktop CI workflow is Windows-only (`windows-2022`) and runs the desktop JVM tests and distributable task on Windows.
- The Ubuntu workflow is Android-focused and does not provide Linux desktop validation.
- The release workflow is explicitly gated to `mihonapp/mihon`; it is not an authorized release workflow for this downstream repository.
- Public desktop release artifacts inspected were Windows ZIPs; the desktop release `desktop-v0.11.20.6` published a Windows x64 ZIP and SHA-256 sidecar. No Linux desktop artifact appeared in the inspected release list.
- The README is slightly stale about the existence of a Windows desktop release, but its statement that Linux packaging/validation are not configured matches the build and CI configuration.

## 8. License, dependency and attribution assessment

- The candidate repository declares Apache-2.0 and contains a root `LICENSE`.
- The README credits Mihon and Tachiyomi and states that copyright notices are preserved.
- Desktop dependency metadata is generated by AboutLibraries for the JVM source set; a desktop test checks the packaged metadata and license content.
- A NOTICE file and Apache-2.0 license for included Material Symbols assets are present under desktop resources.
- A pinned FlexibleAdapter AAR/POM and license are retained with SHA-256 checks in the build configuration.

**Not completed:** a full resolved dependency inventory, SBOM generation, license compatibility review for the actual Linux runtime closure, native-library license/ABI attribution, and verification of the generated desktop notices. Those require a successful source checkout and Gradle dependency resolution. Apache-2.0 on the repository alone does not prove every bundled runtime dependency is distributable under NamiShelf's planned packaging.

## 9. Findings

### M0-LNX-001 — Linux desktop build and packaging are explicitly disabled
- **Severity:** High
- **Confidence:** Confirmed
- **Evidence:** `scripts/build-desktop.sh` Linux branch exits non-zero; Gradle native formats are DMG/MSI only; README and CI corroborate.
- **Impact:** The candidate cannot currently produce a supported Linux desktop build through its documented build entrypoint.
- **Required follow-up:** Add a separate, authorized Linux build/packaging workstream after the base strategy decision. Do not modify the candidate during M0.

### M0-LNX-002 — Default Linux storage paths violate NamiShelf XDG/profile requirements
- **Severity:** Critical for direct reuse without adaptation
- **Confidence:** Confirmed source-level mismatch
- **Evidence:** `DesktopPlatformPaths.resolve()` maps non-Windows config and local roots to `~/.mihon`.
- **Impact:** Risk of data collision with Mihon, failure to satisfy clean-profile guarantees, and inability to honor XDG separation/cache-vs-download requirements unchanged.
- **Required follow-up:** NamiShelf must own a new path adapter and explicit migration boundary. Do not run the candidate against a real user profile.

### M0-LNX-003 — No Linux desktop CI acceptance gate
- **Severity:** High
- **Confidence:** Confirmed
- **Evidence:** Desktop CI uses Windows; Ubuntu workflow does not invoke desktop JVM tests or desktop packaging.
- **Impact:** Linux regressions in native dependencies, graphics, filesystem, Secret Service and packaging can pass upstream CI undetected.
- **Required follow-up:** Require Linux x86_64 CI with build, JVM tests, clean-profile smoke, X11 launch, native dependency and package install/upgrade/uninstall checks before release.

### M0-LNX-004 — Linux credential and URI adapters are present but not runtime-validated
- **Severity:** Medium
- **Confidence:** Confirmed implementation presence; runtime status not assessed
- **Evidence:** `secret-tool`, `xdg-mime`, `update-desktop-database` calls and Linux desktop entry exist.
- **Impact:** Presence of code does not prove correct behavior on Mint, under Flatpak, with locked/missing Secret Service, or from a packaged runtime.
- **Required follow-up:** Test missing, locked, inaccessible and working Secret Service; test URI registration and cleanup on the actual supported desktop sessions and package formats.

### M0-LNX-005 — Native runtime dependency closure is not yet proven
- **Severity:** High
- **Confidence:** Suspected risk, not a confirmed failure
- **Evidence:** JNA/JNA-platform and 7-Zip JNI bindings are dependencies; repository contains no committed Linux native binaries and Linux packaging is absent.
- **Impact:** Missing or incompatible native libraries can break archive handling, graphics or startup on supported distributions.
- **Required follow-up:** Resolve and inspect the actual Linux runtime dependency graph, inspect ELF architecture/ABI requirements, and smoke-test on clean Mint/Ubuntu environments.

### M0-LNX-006 — Candidate has substantial downstream divergence
- **Severity:** Medium
- **Confidence:** Confirmed
- **Evidence:** GitHub compare reports 2,185 commits ahead and 275 behind the pinned upstream reference.
- **Impact:** Upstream synchronization and security maintenance require a managed downstream delta; simple rebasing/cherry-picking assumptions are unsafe.
- **Required follow-up:** Produce a component-level provenance/delta map before any fork/import/production derivation.

## 10. M0 decision and next steps

**Current decision:** Do not accept Mihon Desktop as a Linux-ready NamiShelf base. Keep it as a conditional source of reusable shared Kotlin/domain/reader code pending a successful Linux feasibility build and a component-level delta review.

The next M0 work required to close the evidence gap is:
1. Obtain the exact pinned source archive in an environment with GitHub network access, without modifying the source.
2. Reproduce the checkout at `701080c251c8fd8b35d0b57a481d93b9a9eb3701` and verify clean working tree.
3. Install the documented JDK/Python/Android SDK requirements only in an isolated build environment; install Gradle dependencies through the wrapper.
4. Inspect Gradle configuration and dependency resolution; run at minimum `:app-desktop:jvmTest` and the least invasive Linux JVM build target that does not invoke the Linux-disabled packaging script.
5. If the JVM tests/build succeed, launch the unmodified app in an isolated temporary home/profile under X11; do not point it at any real `~/.mihon` data.
6. Capture runtime linkage, graphics, fonts, archive/7-Zip, filesystem watcher, Secret Service and URI integration evidence.
7. Generate the resolved dependency inventory/SBOM and verify notices and native artifact licenses.
8. Reassess GO/NO-GO against NamiShelf's XDG, dedicated runtime, packaging, security and release constraints.

No candidate code, NamiShelf application code, build configuration, branch, release, PR or merge was changed as part of this assessment.
