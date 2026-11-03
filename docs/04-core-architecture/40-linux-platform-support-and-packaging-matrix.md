# Linux Platform Support and Packaging Matrix

**Status:** Linux-first, x86_64 release-1 scope approved under ADR-021. Minimum distribution/runtime versions and exact desktop-session matrix remain open pending M0.

## 1. Release-1 support commitment

| Platform | Release-1 status | Validation expectation |
|---|---|---|
| Linux Mint 22.3 x86_64 | Primary reference platform | Full acceptance and manual validation |
| Ubuntu-based x86_64 | Target distribution family | Validate representative supported baseline |
| Debian-based x86_64 | Target distribution family | Validate representative supported baseline |
| Arch-based x86_64 | Target distribution family | Validate representative supported baseline |
| Linux ARM64 | Not committed | Future feasibility assessment |
| Windows | Not committed | Future port only |
| macOS | Not committed | Future port only |

Being based on a listed distribution family does not automatically guarantee compatibility. NamiShelf must publish a tested minimum baseline and supported package/runtime combinations.

## 2. Release-1 package formats

| Format | Intended role | Required validation |
|---|---|---|
| AppImage | Portable user-managed application package | Download, executable permissions, launch, desktop integration expectations, update strategy and removal |
| Flatpak | Sandboxed desktop distribution | Install/launch, portals, external-volume access, permissions, updates, desktop integration and uninstall |
| DEB | Native Debian/Ubuntu-family package | Install/upgrade, dependencies, desktop entry, user data preservation and purge/removal behavior |

The formats are release targets, not a claim that every distribution/version supports every package equally. The release matrix must state which package is recommended for each tested environment.

## 3. Architecture portability requirements

- Keep domain entities, application use cases and storage contracts independent of Linux desktop APIs.
- Isolate filesystem path resolution, portals, notifications, file pickers, desktop integration and platform lifecycle behind platform services.
- Avoid assuming a fixed home-directory layout; respect XDG environment variables.
- Treat external download directories as permission-sensitive resources; Flatpak must use supported portal grants or explicitly documented permissions.
- Avoid hard-coded x86_64 assumptions in business logic; native binaries and packaging assets must be architecture-scoped.
- Keep image decoding and graphics dependencies behind replaceable interfaces where platform-specific behavior may arise.
- Record any unavoidable Linux-specific implementation in the platform adapter layer.

## 4. Release gates

Before release:
1. Validate on Linux Mint 22.3 x86_64.
2. Validate at least one representative environment for each distribution family claimed as supported.
3. Test each published package format on its declared distribution matrix.
4. Verify install, first launch, restart, upgrade, data preservation, uninstall and desktop integration.
5. Verify SQLite/native library availability, Compose runtime, fonts, image decoding and graphics behavior.
6. Verify Flatpak sandbox behavior for user-selected external download locations and backup destinations.
7. Publish minimum distribution/runtime versions and known limitations.

## 5. M0 platform investigation

M0 must establish:
- candidate project's current Linux/JVM desktop portability and native dependencies;
- minimum viable JDK/JVM runtime and packaging strategy;
- glibc and system-library baseline for AppImage/DEB;
- Flatpak runtime/SDK and portal requirements;
- X11 and Wayland support expectations;
- GPU acceleration and software-rendering fallback behavior;
- x86_64 native artifacts and transitive dependency architecture;
- feasibility and cost of future ARM64, Windows and macOS ports.

Do not select minimum versions based solely on the current development machine.

## 6. Out of scope for release 1

Official Windows, macOS and Linux ARM64 support are explicitly excluded. The architecture should avoid needless blockers, but no CI, package, support or compatibility commitment exists for these targets until a separate decision.

No implementation, build, dependency installation or test execution is authorized by this specification.
