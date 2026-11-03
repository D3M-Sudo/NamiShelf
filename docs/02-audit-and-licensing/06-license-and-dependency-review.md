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
