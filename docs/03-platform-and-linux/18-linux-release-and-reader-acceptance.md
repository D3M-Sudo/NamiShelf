# NamiShelf — Linux Release and Reader Acceptance Plan

**Status:** Proposed acceptance gates aligned with owner-approved ADR-021/ADR-027; no runtime validation performed  
**Date:** 2026-11-03

## Platform strategy

Linux x86_64 on Linux Mint 22.3 is the primary reference target. Ubuntu-, Debian- and Arch-based x86_64 distributions are target families only when exact releases and package combinations are tested and documented. ARM64 is outside the release-1 support commitment.

## Packaging gates

| Format | Required evidence |
|---|---|
| DEB | Clean install/upgrade/uninstall, dependencies, desktop integration, data preservation |
| Flatpak | Runtime/SDK compatibility, portals, permissions, Secret Service/D-Bus, extension process policy |
| AppImage | Clean launch, native loading, glibc baseline, desktop integration, update/signature policy, stable data paths |

A successful packaging task alone does not establish support. Each format requires clean-environment smoke, upgrade, and uninstall verification. X11 is the mandatory first-release baseline; Wayland is supported only for tested compositor/session combinations. Stable is the only initial public release channel.

## Architecture policy

x86_64 is the first release gate. aarch64 is published only after native dependency and packaging validation; otherwise it remains clearly marked planned. Record minimum glibc and system-library requirements. A single distribution test does not establish support for every derivative.

## Reader reference profile

Use the primary Linux Mint 22.3 environment as the constrained reference profile: 8 GB RAM, 1920×1080 at 60 Hz, older AMD A-series hardware. Prioritize bounded memory and stability over aggressive prefetch. Numeric thresholds must come from authorized measurements, not static code inspection.

## Required measurements

Cold/warm startup; chapter-open to first visible page; page-turn latency; peak resident/native image memory; decoded and tile cache sizes; decoder concurrency and prefetch depth; memory release after chapter switch/close; low-memory behavior; CPU/GPU use and fallback; long-session stability.

## Required fixtures

Large JPEG/PNG/WebP; EXIF orientation and mirroring; alpha and color profiles; corrupt/truncated files; very tall/wide scans; two-page spreads; high-resolution region decode; animated formats only if explicitly supported.

## Release acceptance

No unbounded decode/prefetch allocation; release native resources on success, cancellation, failure, navigation, and shutdown; no crash or persistent memory growth in long sessions; validate the primary X11/Wayland session; test HiDPI/fullscreen; verify CPU fallback; publish reproducible budgets measured on the reference profile.

Numeric thresholds remain open pending runtime measurements.
