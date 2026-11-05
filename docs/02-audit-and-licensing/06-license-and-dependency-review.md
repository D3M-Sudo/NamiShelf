# NamiShelf — Preliminary License and Dependency Review

**Status:** Preliminary static review; legal and dependency gates remain open  
**Date:** 2026-11-03

## 1. License baseline

The inspected Mihon Desktop repository contains an Apache License 2.0 license file. Mihon upstream is also recorded as Apache-2.0 in the project research. Apache-2.0 generally permits use, modification, and redistribution subject to its conditions, including preservation of applicable notices and license text. This is not legal advice.

NamiShelf must preserve upstream copyright/license notices, include required Apache-2.0 text in source and binary distributions, preserve applicable NOTICE content, produce third-party attributions, independently review fonts/icons/illustrations/screenshots, avoid trademark confusion, and clearly identify downstream modifications.

## 2. Preliminary dependency observations

Inspected Gradle catalogs include Kotlin 2.3.10, Compose Multiplatform 1.10.2, OkHttp 5.3.2, SQLDelight 2.2.1, SQLite 2.6.2, Voyager 1.1.0-beta03, JNA 5.19.1, Tink 1.23.0, Coil 3.4.0, and AndroidX/Android-oriented libraries.

Catalog presence does not prove inclusion in the desktop runtime. Resolve and review the actual `app-desktop` dependency graph. The project also pins a FlexibleAdapter artifact in a local Maven directory with SHA-256 verification; this is a positive integrity control, but provenance, licensing, and update ownership still require review.

## 3. Required dependency audit

Before accepting the technical base:
1. Resolve desktop runtime and build dependencies in a controlled environment.
2. Generate an SBOM (CycloneDX or SPDX).
3. Record direct/transitive dependencies, versions, repositories, and checksums.
4. Review known vulnerabilities and maintenance status.
5. Identify dynamic versions, snapshots, JitPack/Git references, and local binary artifacts.
6. Identify native libraries and architecture-specific payloads.
7. Produce a license report for packaged artifacts.
8. Verify source/binary packages contain required notices.
9. Define dependency update and vulnerability response ownership.

No dependency installation or resolution was performed in this phase.

## 4. Packaging review

- **AppImage:** inspect bundled libraries and license payload.
- **Flatpak:** inspect runtime, SDK, permissions, bundled extensions, and native dependencies.
- **DEB:** inspect package dependencies, maintainer scripts, installed paths, and copyright metadata.

## 5. Open legal questions

Verify redistribution rights for inherited assets, separate font/icon notices, completeness of third-party NOTICE inventory, NamiShelf name/logo trademark risk, and license obligations of APK-to-JAR conversion code and related libraries.

## 6. Gate

Do not distribute public packages until an SBOM and license report have been reviewed and packaged notices verified.


---

## 7. M0 evidence update — 2026-11-04

The pinned candidate is `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`.

### Static evidence confirmed

- The candidate declares Apache-2.0 at repository level.
- The desktop build applies an AboutLibraries Gradle plugin and the repository's `mihon.desktop.licenses` convention plugin.
- The convention plugin exports JVM library metadata to `META-INF/mihon/dependencies.json` and wires the export into JVM resource processing.
- The desktop test suite includes a dependency-notice contract; the earlier static report records an expectation of 203 notices.
- The desktop resources include Material Symbols NOTICE and Apache-2.0 license material.
- The Gradle version catalog declares Compose Multiplatform 1.10.2, SQLDelight 2.2.1, SQLite 2.6.2 and JNA 5.19.1. These are declarations, not proof that every component enters the packaged runtime.

### Execution evidence

The NamiShelf Linux CI run #8 completed the desktop JVM compile and reached the JVM tests, but the test task failed. The workflow's evidence collection attempts to export AboutLibraries definitions and resolve `jvmRuntimeClasspath` even when an earlier step fails. A subsequent workflow run is now executing with the GUI smoke test decoupled from the JVM test outcome.

No complete resolved runtime dependency inventory, SPDX/CycloneDX SBOM, native ELF inventory, package-level license report or clean-package notice verification is yet accepted as complete. Those gates require the current CI evidence to be inspected and checked against actual packaged files.

### Legal/dependency disposition

**Static source/license baseline: provisionally acceptable for further evaluation, not approved for redistribution.**

Before any public NamiShelf binary:
1. Capture and retain the complete resolved desktop runtime dependency graph and its repository coordinates.
2. Generate an SPDX or CycloneDX SBOM from resolved artifacts, not from the version catalog alone.
3. Include native binaries and embedded payloads in the inventory.
4. Verify every shipped license and NOTICE entry against the actual DEB, Flatpak and AppImage payloads independently.
5. Check fonts, icons, images, trademarks, inherited names and product branding separately from source-code licensing.
6. Define vulnerability monitoring, dependency update ownership and a reproducible provenance/checksum process.

The CI evidence step is an evidence collector, not a legal approval gate. Do not describe the project as fully license-audited until the generated reports and final package payloads have been reviewed.


## 8. Workflow run #9 update — 2026-11-05

The NamiShelf M0 Linux workflow [run #9](https://github.com/D3M-Sudo/NamiShelf/actions/runs/41747974947) completed its dependency/license evidence-collection step, but this is not a completed audit result:

- The AboutLibraries export command is followed by `|| true`; a green workflow step does not prove the export succeeded.
- The `jvmRuntimeClasspath` dependency report command is also followed by `|| true`; its output must be inspected and checked for a complete resolution.
- No reviewed SPDX/CycloneDX SBOM, native payload inventory or package-level license report has been accepted.
- The workflow artifact [13284233790](https://github.com/D3M-Sudo/NamiShelf/actions/runs/41747974947/artifacts/13284233790) was retrieved, but its contents have not yet been inspected in this session.

**Disposition unchanged:** static source/license baseline is provisionally acceptable for further evaluation only. The dependency/legal gate remains open; no redistribution approval is implied.
