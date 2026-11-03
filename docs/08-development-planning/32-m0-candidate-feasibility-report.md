# NamiShelf — M0 Candidate Feasibility Report

**Status:** M0 in progress — static source review completed; executable validation blocked by the current isolated environment  
**Assessment date:** 2026-11-03  
**Candidate:** `AltairCardinal/mihon-desktop`  
**Pinned candidate commit:** `701080c251c8fd8b35d0b57a481d93b9a9eb3701` (`main`, 2026-11-02)  
**Scope:** Read-only inspection and baseline feasibility only; no candidate or NamiShelf application code changes

## 1. Executive assessment

**Preliminary disposition: NO-GO as an immediately usable Linux desktop base; CONDITIONAL candidate for a separately authorized downstream Linux-enablement port.**

The repository contains a real Kotlin/JVM Compose Desktop application and some Linux-specific integration code. However, its own build entrypoint explicitly rejects Linux desktop packaging, its native distribution configuration targets only DMG and MSI, its Desktop CI runs on Windows, and its Linux data paths default to the legacy `~/.mihon` directory rather than the owner-approved NamiShelf XDG layout.

The source-level evidence is sufficient to reject a drop-in Linux reuse assumption. It is not sufficient to reject the codebase as a possible starting point for a substantial downstream port: `:app-desktop:jvmTest` and GUI/runtime behavior have not been executed in this environment.

M0 is therefore **not complete**. A final GO/NO-GO requires a source checkout and executable validation on a network-enabled isolated environment, preferably the primary Linux Mint 22.3 x86_64 reference system.

## 2. Source identity and provenance

| Item | Evidence |
|---|---|
| Repository | [AltairCardinal/mihon-desktop](https://github.com/AltairCardinal/mihon-desktop) |
| Repository description | “Mihon Desktop downstream project, synchronized with mihonapp/mihon.” |
| GitHub fork flag | `false`; it is a separate downstream repository, not a GitHub fork object |
| Default branch | `main` |
| Pinned HEAD | `701080c251c8fd8b35d0b57a481d93b9a9eb3701` |
| Commit date/message | 2026-11-02; “Merge reader nonblocking pairing and Android reread fixes into main” |
| Commit signature | GitHub reports `verified: false`, reason `unsigned` |
| GitHub-reported repository size | 188,289 KB |
| Root license | Apache License 2.0 |
| Main branch protection | GitHub branch metadata reports `protected: true` |

The repository contains an `upstream/main` reference. GitHub compare metadata for `upstream/main...main` reports a diverged history: 2,185 commits ahead and 275 behind at the assessment point. This is a substantial downstream delta and requires a maintained provenance/delta ledger before adopting the codebase.

The pinned commit is reproducible as a source identity, but its unsigned status means the commit itself does not provide a verified author signature. Preserve the exact SHA and independently verify any downloaded source archive before relying on it.

## 3. Release and platform evidence

The README describes Windows as the primary desktop target, macOS as an additional build path, and says Linux desktop packaging/validation are not configured.

The README's statement that no public release packages exist is stale: the GitHub Releases API lists desktop release assets. The desktop artifacts found are Windows ZIPs, not Linux packages.

| Release evidence | Result |
|---|---|
| Desktop release | `desktop-v0.11.20.6`, published 2026-10-15 |
| Desktop artifact | `Mihon-Desktop-0.11.20.6.4f1483b-windows.zip` |
| Artifact size | 195,109,770 bytes |
| Published SHA-256 | `6993af10f127f512bf406cd319384113876675c8bc72df5af37af1418d044dd0` |
| Other desktop preview | Windows ZIP, prerelease |
| Linux desktop artifacts | None found in the releases inspected |
| Android artifacts | Separate APK releases; not evidence of Linux desktop support |

Sources: [README at pinned commit](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/README.md), [desktop release](https://github.com/AltairCardinal/mihon-desktop/releases/tag/desktop-v0.11.20.6).

## 4. Build-system and packaging findings

### F-01 — Linux desktop build entrypoint explicitly aborts

**Severity:** Blocker for direct Linux use  
**Status:** Confirmed by source inspection; not executed

In `scripts/build-desktop.sh`, the `Linux*` case either dispatches to the Windows PowerShell build if `powershell.exe` exists or exits with:

```text
Linux desktop packaging is not configured for this repository.
Supported platforms: macOS via this script, Windows via scripts/build-windows.ps1 dispatch.
```

This is an intentional unsupported-platform guard, not a transient build error. The official desktop build workflow cannot be used on Linux unchanged.

Source: [build-desktop.sh at pinned commit](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/scripts/build-desktop.sh).

### F-02 — Native packaging targets exclude Linux formats

**Severity:** Blocker for release-1 packaging as-is  
**Status:** Confirmed by source inspection; not executed

`app-desktop/build.gradle.kts` configures Compose Desktop native distributions with:

- `TargetFormat.Dmg`
- `TargetFormat.Msi`

There are no `Deb` or `Rpm` targets. No AppImage build/update pipeline was found in the desktop build entrypoint. The distribution output root also selects a macOS or Windows-specific path; it has no Linux-specific output configuration.

The project therefore has no ready Linux DEB, Flatpak or AppImage delivery path. This conflicts with NamiShelf's approved release strategy and would require a downstream packaging workstream.

Source: [app-desktop/build.gradle.kts at pinned commit](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/build.gradle.kts).

### F-03 — No Linux desktop CI acceptance lane

**Severity:** High  
**Status:** Confirmed by workflow inspection

- `.github/workflows/desktop-ci.yml` is `workflow_dispatch` only and runs on `windows-2022`.
- `.github/workflows/build.yml` runs on `ubuntu-24.04`, but installs Android SDK Platform 36 / Build Tools 36.0.0, builds the Android candidate and runs Android unit tests. It does not run Linux desktop tests or packaging.
- `.github/workflows/release.yml` is explicitly gated to `mihonapp/mihon`; it is a historical upstream Android release workflow, not this downstream desktop release pipeline.

The latest product Desktop CI and Build & Test runs found on `main` were both failures on 2026-10-13 at commit `3367aed7a88626aa68c8beb40c6839ded35e394d`. Desktop CI failed at `Run Desktop JVM tests`; Build & Test failed at `Run unit tests`. Later runs through 2026-11-02 were documentation/feedback workflows, not a new desktop product validation. These historical failures are not proof that the current commit fails, but there is no current Linux desktop CI evidence.

Sources: [Desktop CI workflow](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/.github/workflows/desktop-ci.yml), [Build & Test workflow](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/.github/workflows/build.yml), [Desktop CI run](https://github.com/AltairCardinal/mihon-desktop/actions/runs/38425586545), [Build & Test run](https://github.com/AltairCardinal/mihon-desktop/actions/runs/38425586446).

### F-04 — Linux data paths use legacy `~/.mihon`, not XDG

**Severity:** High; direct conflict with NamiShelf ADR-004/020 and XDG design  
**Status:** Confirmed by source inspection; filesystem side effects not executed

`DesktopPlatformPaths.resolve()` uses `~/.mihon` as the config and local-data root for all non-Windows platforms. On Linux this places the database, network cache, downloads, extensions, covers, logs and backups below the legacy hidden directory. It does not honor the NamiShelf-approved separation of XDG config/data/cache directories.

The Linux URI-handler registration separately honors `XDG_DATA_HOME` for its desktop-entry installation path. That isolated use of XDG does not make the application's data layout XDG-compliant.

NamiShelf must not launch this candidate with the normal user profile as-is: its startup path resolves the default platform paths and may create/use the legacy Mihon directory. A future downstream port must define a distinct NamiShelf application identity and XDG roots, and migration must remain explicit opt-in.

Sources: [DesktopPlatformPaths.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/platform/DesktopPlatformPaths.kt), [Main.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/Main.kt).

## 5. Linux-specific code that does exist

The repository is not devoid of Linux awareness. Several platform adapters are present, but their presence is not equivalent to a validated Linux product.

| Area | Static evidence | Assessment |
|---|---|---|
| Desktop framework | Kotlin Multiplatform `jvm()`; Compose Desktop dependency uses `compose.desktop.currentOs` | JVM/Compose source may be portable; not compiled here |
| Credential storage | Linux backend calls `secret-tool store/lookup/clear` | Design direction matches Secret Service; runtime integration not tested |
| Credential failure | Tests cover absent CLI and locked/permission error behavior | Fail-closed behavior is represented in tests; tests not executed |
| URI scheme | Linux `.desktop` entry; `XDG_DATA_HOME` or `~/.local/share/applications`; `update-desktop-database` and `xdg-mime` | Partial desktop integration; requires packaged jpackage layout and host tools |
| File picker | Swing `JFileChooser` | Generic JVM integration; portal-native behavior not demonstrated |
| File watching | Dependency comment explicitly references inotify on Linux | Linux-aware dependency; runtime not tested |
| Native integration | JNA and 7-Zip JNI all-platforms dependency | Linux x86_64 native artifact availability/loading still requires runtime validation |
| Windows/macOS adapters | Windows-specific native calls and macOS native share integration are present | Requires strict platform dispatch review; no Linux conclusion inferred |

The Linux credential backend is a positive compatibility signal for ADR-023. It does not prove that Secret Service is available in every target DEB/Flatpak/AppImage environment, nor that the application is wired and tested end-to-end on Linux.

## 6. Toolchain and dependency profile

The README specifies:

- JDK 21;
- Python 3.11 or newer;
- Android SDK Platform 36 and Build Tools 36.0.0 for Android modules in the shared build;
- Gradle wrapper, pinned to Gradle 8.14.4.

The desktop module includes Compose Desktop, coroutines/Swing, SQLDelight's JVM SQLite driver, JNA/JNA Platform, Coil, archive support through 7-Zip JNI, directory watching, Rhino and DEX-to-JVM tooling.

This is a substantial mixed Android/JVM repository. The root settings include both Android and desktop modules. A desktop-only task may still configure shared Android build logic; this must be verified in an executable build environment rather than assumed.

### Licensing and attribution

- Root project license is Apache-2.0.
- README attributes Mihon and Tachiyomi contributors and preserves copyright notices for Javier Tomás (2015) and Mihon Open Source Project (2024).
- Desktop dependency notices are generated into `META-INF/mihon/dependencies.json` by the AboutLibraries Gradle plugin. A desktop test expects 203 dependency notices and checks selected license contents.
- Material Symbols icons have a NOTICE file and an included Apache-2.0 license.
- The full resolved desktop dependency inventory, native transitive licenses and a reproducible SBOM were not generated in this assessment because Gradle could not be run.

Apache-2.0 does not grant trademark rights. NamiShelf must use its own name, application ID, icons, package metadata and product presentation; retain applicable copyright/license/NOTICE obligations and avoid implying Mihon affiliation.

Sources: [LICENSE](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/LICENSE), [README credits](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/README.md), [desktop license build plugin](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/buildSrc/src/main/kotlin/mihon.desktop.licenses.gradle.kts).

## 7. Execution environment and blocker

The isolated execution container is Linux x86_64, Debian 13 (trixie), not Linux Mint 22.3.

| Component | Observed |
|---|---|
| Architecture | x86_64 |
| OS | Debian GNU/Linux 13 |
| JVM | OpenJDK 21.0.12 |
| Python | 3.13.5 |
| Memory visible to container | approximately 5.8 GiB |
| Filesystem | approximately 30 GiB free at start |
| Native libraries | GTK 3, X11, Xext, Xrender, Xtst, Xi, FreeType and Fontconfig present |
| `xdg-mime` | present |
| Gradle executable/cache | absent |
| Android SDK / `ANDROID_HOME` | absent |
| `secret-tool` | absent |
| `update-desktop-database` | absent |
| Graphical X11/Wayland session | not available in this container |

A shallow Git clone was attempted into the isolated work area and failed before checkout:

```text
fatal: unable to access 'https://github.com/AltairCardinal/mihon-desktop.git/':
Could not resolve host: github.com
```

No source checkout was obtained. No dependency installation was attempted because the environment could not resolve external hosts. No Gradle build, test, GUI launch, Secret Service test or package build was run. This is an execution-environment/network blocker, not a failing candidate build.

## 8. Remaining M0 execution gates

M0 remains open until the following are completed in a network-enabled isolated Linux environment:

1. Obtain and verify a checkout/archive pinned to `701080c251c8fd8b35d0b57a481d93b9a9eb3701`; record archive checksum and clean worktree status.
2. Install/use JDK 21, Python 3.11+, Gradle wrapper dependencies and Android SDK Platform 36 / Build Tools 36.0.0 as required by project configuration.
3. Run `./gradlew :app-desktop:jvmTest`; retain XML/HTML reports and classify every failure/skip.
4. Attempt the unmodified Linux desktop runtime path under X11. Record exact command, display/session/compositor, startup/shutdown, logs and native library loading.
5. Exercise Secret Service save/load/delete and missing/locked/error behavior using a disposable test account in a controlled desktop session.
6. Verify the candidate's default path behavior in a disposable home; confirm the `~/.mihon` conflict without touching any real user profile.
7. Generate desktop dependency notices and a resolved dependency/native-library inventory; produce an SBOM where feasible and review license/NOTICE completeness.
8. Confirm Linux native archive/image dependencies load on x86_64.
9. Record Wayland behavior separately; do not claim support without compositor-specific evidence.
10. Record the exact delta required for a NamiShelf-owned XDG profile, Linux DEB/Flatpak/AppImage packaging and Linux CI. These are downstream work, not M0 implementation.

The official packaging entrypoint already exits on Linux and the Gradle distribution formats omit Linux. Do not spend time treating a Windows or macOS package task as Linux validation.

## 9. Preliminary recommendation

- **Drop-in reuse for NamiShelf Linux release 1:** NO-GO.
- **Reuse as a source-code starting point for a downstream Linux port:** CONDITIONAL; pending successful JVM test/runtime evidence, source provenance review and explicit cost/maintenance assessment.
- **Compose Desktop suitability:** unresolved until executable Linux build and X11 runtime tests.
- **SQLite/SQLDelight JVM feasibility:** plausible from dependency configuration; unverified by execution.
- **Secret Service compatibility:** positive static evidence; real session integration unverified.
- **NamiShelf XDG/profile isolation:** incompatible as-is; requires downstream path/identity replacement before application launch under a normal user account.
- **Linux packaging and updates:** absent as-is; requires downstream engineering for all three approved formats.
- **Wayland:** unassessed.

No fork/import/porting decision is made by this report. No code changes, candidate modifications, PRs, merges or releases were made. The final M0 GO/NO-GO decision is deferred until the executable gates above are satisfied.
