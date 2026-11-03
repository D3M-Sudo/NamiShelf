# NamiShelf — Linux Platform Design and Validation Plan

**Status:** Proposed; no implementation selected  
**Primary validation environment:** Linux Mint 22.3  
**Additional targets:** Ubuntu-based, Debian-based, Arch-based distributions

## 1. Baseline

The candidate is Kotlin/JVM with Compose Desktop. In the reviewed baseline, Linux packaging and validation are not configured and inspected CI runs on Windows. Linux must therefore be treated as a first-class platform workstream, not merely a packaging task.

## 2. Platform layers

Keep product/domain behavior independent from distribution-specific integration.

### Core desktop runtime
- JVM runtime and application lifecycle.
- Compose Desktop windowing/input.
- Persistent storage/database.
- Network stack and cookie handling.
- Reader image decoding, caching, and prefetch.
- Download scheduler and filesystem abstraction.

### Linux integration boundary
- XDG Base Directory paths.
- Desktop entry and icon installation.
- MIME/open-with integration where applicable.
- Portal-based file/directory selection in sandboxed deployments.
- URL opening, notifications if in scope, keyboard shortcuts, window-manager behavior.
- HiDPI, fractional scaling, multi-monitor behavior.
- Wayland and X11 compatibility.

Do not hardcode a Cinnamon-only implementation. Mint 22.3 is the primary acceptance environment, not the only target.

## 3. Filesystem policy

Use XDG locations:
- Config: `$XDG_CONFIG_HOME/nami-shelf` (fallback `~/.config/nami-shelf`)
- Data: `$XDG_DATA_HOME/nami-shelf` (fallback `~/.local/share/nami-shelf`)
- Cache: `$XDG_CACHE_HOME/nami-shelf` (fallback `~/.cache/nami-shelf`)
- State: `$XDG_STATE_HOME/nami-shelf` (fallback `~/.local/state/nami-shelf`)

Never assume the working directory is writable. Keep mutable data outside executable, AppImage, and Flatpak installation paths.

## 4. Packaging targets

### AppImage
Validate bundled JVM/native libraries, desktop integration, paths with spaces/non-ASCII characters, read-only image behavior, and update strategy separately. AppImage itself does not define a secure updater.

### Flatpak
Choose application ID/runtime after product identity decisions. Minimize permissions, prefer portals over broad home access, validate network/file chooser/URL integration and persistent data. Ensure extension execution does not silently bypass sandbox assumptions.

### DEB
Define supported Debian/Ubuntu baseline and dependency policy. Validate package lifecycle, upgrades/downgrades, uninstall, configuration preservation, desktop integration, and maintainer scripts. Never run application code as root during package operations.

## 5. Linux validation matrix

| Area | Mint 22.3 | Ubuntu LTS | Debian stable | Arch-based |
|---|---|---|---|---|
| Launch/startup | Required | Required | Required | Required |
| X11 | Required | Required | Required | Required |
| Wayland | Where available | Where available | Where available | Where available |
| File chooser / URL | Required | Required | Required | Required |
| Database persistence | Required | Required | Required | Required |
| Reader image decode | Required | Required | Required | Required |
| Downloads/resume | Required | Required | Required | Required |
| Backup/restore | Required | Required | Required | Required |
| AppImage | Required | Smoke | Smoke | Smoke |
| Flatpak | Required | Required | Required | Required |
| DEB | Required | Required | Required | N/A |
| HiDPI/scaling | Required | Required | Smoke | Required |

## 6. Mint acceptance profile

Record distribution release/codename, kernel/architecture, Cinnamon version, X11/Wayland session, JVM runtime policy, GPU/driver, image rendering, memory/startup budgets, and portal/desktop integration versions. Do not assume package state or hardware capabilities beyond owner-supplied information.

## 7. Open platform decisions

Require explicit approval before locking: bundled vs system JVM, minimum supported distribution window, Wayland release criteria, Flatpak runtime/application ID, AppImage update mechanism, and notification/desktop integration scope.
