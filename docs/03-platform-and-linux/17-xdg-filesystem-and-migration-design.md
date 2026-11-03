# NamiShelf — XDG Filesystem and Migration Design

**Status:** Proposed Linux platform design  
**Date:** 2026-11-03

## Principles

Follow XDG Base Directory conventions; separate durable data from regenerable cache; keep secrets in OS credential services; preserve data across package formats; never silently adopt or overwrite an existing Mihon Desktop profile.

## Proposed layout

| Purpose | Proposed location |
|---|---|
| Configuration | `$XDG_CONFIG_HOME/namishelf/` (default `~/.config/namishelf/`) |
| Databases and durable data | `$XDG_DATA_HOME/namishelf/` (default `~/.local/share/namishelf/`) |
| Regenerable cache | `$XDG_CACHE_HOME/namishelf/` (default `~/.cache/namishelf/`) |
| Logs and diagnostic state | `$XDG_STATE_HOME/namishelf/` (default `~/.local/state/namishelf/`) |
| Downloads/extensions | Dedicated subdirectories under the data area |
| Backups | User-selected destination |
| Credentials | Secret Service items; never plaintext XDG files |

Exact subdirectory names and permissions remain implementation details. Flatpak must use its sandbox-visible XDG paths and declared permissions rather than hardcoded host paths.

## Legacy profile handling

The candidate currently uses `~/.mihon` on non-Windows platforms. NamiShelf should detect potential legacy profiles read-only and show the user the discovered path, version, size, and supported data categories.

A migration flow should offer import, skip, or inspect-only; create a recovery checkpoint before mutation; use version-aware adapters; report skipped/failed categories; be restartable; and preserve the source profile until successful verification and explicit cleanup.

Do not assume every `~/.mihon` directory belongs to Mihon Desktop.

## Credential migration hazard

The candidate's secure-store wrapping-key identity is tied to the canonical secure-store directory path. Copying encrypted records without the matching OS credential item is insufficient and may make them unreadable.

Never copy ciphertext alone, export wrapping keys to ordinary backups, or silently generate a replacement key over existing ciphertext. If credential migration is not safe, require reauthorization/re-enrollment. Local library migration must remain independent of sync credential migration.

## Package-format invariants

DEB, AppImage, and Flatpak should resolve the same logical profile within their permitted filesystem view. Changing package format must not create a parallel library. Uninstall must preserve user data by default; data deletion requires a separate explicit action.

## Acceptance criteria

Honor XDG overrides and defaults; handle invalid/non-writable paths safely; keep secrets out of files; preserve profiles across package upgrades; make migration restartable; retain the source until verification; allow library migration if credential migration fails; and test Flatpak portal/path behavior.

**Suggested default:** clean NamiShelf XDG profile plus an explicit import wizard, never silent in-place mutation.
