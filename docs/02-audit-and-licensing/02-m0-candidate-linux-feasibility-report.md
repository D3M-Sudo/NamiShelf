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


## 11. GitHub Actions execution path (owner-authorized)

On 2026-11-03 the owner authorized completing the remaining M0 evidence using GitHub-hosted CI instead of a local machine. A dedicated workflow was added to the NamiShelf repository:

- Workflow: `.github/workflows/m0-linux-feasibility.yml`
- Runner: `ubuntu-24.04`
- JDK: Temurin 21
- Candidate: `AltairCardinal/mihon-desktop`
- Immutable candidate commit: `701080c251c8fd8b35d0b57a481d93b9a9eb3701`
- Permissions: `contents: read`; no application secrets are configured.
- Candidate checkout is detached at the pinned SHA and checked for a clean worktree.
- Planned checks: Android SDK/toolchain preflight required by the shared Gradle configuration, desktop JVM compilation, `:app-desktop:jvmTest`, an Xvfb/DBus launch smoke test with isolated HOME and XDG directories, AboutLibraries license metadata export, resolved JVM runtime dependency report, and an explicit Linux `:app-desktop:createDistributable` probe.
- Evidence is uploaded as a workflow artifact with 30-day retention.

The packaging probe is deliberately non-gating: its exit code and log are captured as evidence because the source currently declares only DMG/MSI native formats. A successful JVM compile/test must not be interpreted as Linux packaging support.

**Execution status at report revision:** workflow file committed; a completed GitHub Actions run and its artifact have not yet been inspected. No build/test pass is claimed. The Xvfb launch smoke is now part of the workflow but is not yet an observed pass. Secret Service success/fail-closed runtime behavior, URI integration, native ELF closure, complete SBOM/license review and the final GO/NO-GO remain open until evidence is collected. The workflow does not yet establish those results.

No files in the external candidate repository are modified. No NamiShelf application source, package, release, PR or merge is produced by this workflow.

## 12. Repository CI inventory — 2026-11-03

A repository-wide Actions inventory was performed after the first M0 workflow executions. At the time of inspection, the repository exposed five runs of the single workflow `.github/workflows/m0-linux-feasibility.yml` (no other workflow files were present).

| Run | Commit | Result at inspection | Evidence |
|---|---|---|---|
| #1 `41720328088` | `c35404d` | Failure | SDK preflight failed: `sdkmanager: command not found`. Candidate checkout at the pinned SHA succeeded. Compilation/tests were skipped. Artifact uploaded, but its dependency report is not evidence of successful tests. |
| #2 `41720562026` | `151f03b` | Cancelled | Superseded during workflow edits; no completed job evidence. |
| #3 `41720600337` | `8242f31` | Cancelled | Checkout and Android SDK preflight succeeded; Kotlin/JVM compile succeeded. `:app-desktop:jvmTest` was cancelled while in progress; X11 smoke was skipped. Packaging probe was entered, but run was cancelled before artifact completion. |
| #4 `41724406499` | `cfe36ff` | In progress | SDK setup succeeded; Kotlin/JVM compilation was in progress at the latest inspection. Tests and subsequent steps had not started. |
| #5 `41724989858` | `e313b92` | Pending | Created after adding bounded SDK discovery/install and clearer packaging-probe reporting. Not yet executed at inspection time. |

### CI findings and corrective actions

1. Run #1 exposed a concrete environment assumption: the workflow called `sdkmanager` without ensuring Android command-line tools were installed or locating the binary. The latest workflow now discovers the executable under `ANDROID_HOME/cmdline-tools`, emits an explicit diagnostic if absent, and bounds SDK installation to eight minutes.
2. Run #3 proved that the pinned candidate can be checked out and that `:app-desktop:compileKotlinJvm` completed successfully on Ubuntu 24.04 / Temurin JDK 21. It did **not** prove JVM tests pass.
3. The original packaging probe always returned success from its shell step, regardless of Gradle's exit code. The latest workflow now emits a warning for a non-zero Gradle result and retains the actual exit code/log as evidence. The probe remains non-gating by design.
4. The workflow now has a 45-minute job limit, 15-minute compile/test limits, 5-minute report/package-probe limits, and cancellation of superseded runs. These limits were not active in earlier run attempts.
5. The Android SDK setup, desktop compilation, test execution, Xvfb launch, AboutLibraries export and artifact upload are sequential. A failure in an early required step prevents later runtime evidence from being collected; report-generation steps are best-effort and must not be mistaken for passing checks.

### Current CI verdict

**CI infrastructure: partially stabilized; M0 evidence: incomplete.** The strongest successful result so far is desktop Kotlin/JVM compilation. No completed test pass, GUI launch pass, native packaging support, complete SBOM/license review, Secret Service runtime validation or Linux Mint 22.3 compatibility result has been established. The latest workflow revision has been committed but its run was still pending at inventory time. No GO decision can be based on these runs.

## 13. Completed run #5 analysis — 2026-11-03

Run [#41724989858](https://github.com/D3M-Sudo/NamiShelf/actions/runs/41724989858), candidate SHA `701080c251c8fd8b35d0b57a481d93b9a9eb3701`, workflow revision `e313b922d5395d097cf29a65cf8fa1fda0fc4bb8`, completed with conclusion `failure`.

Observed step results:
- JDK 21, Gradle setup, exact candidate checkout and Android SDK setup: success.
- `:app-desktop:compileKotlinJvm`: **BUILD SUCCESSFUL in 4m 25s**.
- `:app-desktop:jvmTest`: failed; the wrapper timeout returned exit code `124` after 15 minutes. Before timeout, Gradle emitted many individual failed tests.
- Xvfb/DBus installation and application launch: skipped because the test step failed.
- AboutLibraries and runtime dependency reports: invoked best-effort; their shell commands use `|| true`, so the step's green status does not prove either report succeeded.
- `:app-desktop:createDistributable`: Gradle exit code `0`; this is not proof of Linux packaging. The candidate's `app-desktop/build.gradle.kts` declares only `TargetFormat.Dmg` and `TargetFormat.Msi`, and its distribution root is selected as a macOS or Windows path. The generic task's zero exit may simply mean no Linux package target was configured. No Linux artifact was asserted or inspected.

### JVM test failure classification

The log contains at least 216 test cases marked `FAILED` before timeout. Failure signatures include:
- At least 163 log lines referencing `PlatformCredentialUnavailableException`, affecting many production-DI, reader, download-directory, extension and repository wiring tests.
- At least 29 lines referencing `AssertionFailedError`, indicating additional independent contract/assertion failures.
- Coroutine timeout failures and XML parser/transformer failures also appear.

The dominant failure is consistent with the runner lacking an active Secret Service session: the candidate's Linux credential backend invokes `secret-tool` and fails closed if the command or Secret Service is unavailable. The workflow ran the tests without provisioning a Secret Service daemon. This is an **environmental test-harness defect** that must be corrected before interpreting all credential-dependent failures as product defects. It does not explain the independent assertion, XML and coroutine failures; those remain to be triaged after a valid credential-service test environment is provided.

### Next CI correction

Commit `b79bb03d1e87c6127018c19fc71374aa60038575` updates the workflow to install `gnome-keyring`, `libsecret-tools`, DBus and Xvfb, then start an isolated DBus session, unlock an ephemeral Secret Service keyring, perform a store/lookup/clear probe, and run the JVM suite inside that session. This is a test-only CI fixture; it does not add a fallback to the application credential backend and does not alter the approved fail-closed security policy.

The new workflow run has not yet been inspected. The next decision must be based on that run's complete test report. Do not increase the timeout until the credential-service environment has been corrected and the test throughput/failures re-evaluated.

### Updated M0 assessment

**Build:** confirmed on Ubuntu 24.04 / JDK 21.
**JVM tests:** failed; current run is not a valid clean baseline because Secret Service was not provisioned and independent failures remain.
**Linux GUI:** not tested; gated behind JVM test success.
**Linux native packaging:** unsupported by the declared DMG/MSI-only target configuration; exit code zero from the generic task is not evidence of a package.
**GO/NO-GO:** no final GO. The candidate remains a conditional source-reuse candidate, not a Linux-ready desktop base.
