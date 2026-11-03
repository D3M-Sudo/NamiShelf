# XDG Storage Layout and Configurable Download Location

**Status:** XDG-aligned data separation and user-configurable permanent download location approved under ADR-020. Platform path resolution and external-volume behavior require M0 validation.

## 1. Storage classes

| Class | Linux reference location | Contents | Disposable? |
|---|---|---|---|
| Application data | `$XDG_DATA_HOME/NamiShelf/` (default `~/.local/share/NamiShelf/`) | SQLite database, manifests, metadata and default permanent artifacts | No |
| Configuration | `$XDG_CONFIG_HOME/NamiShelf/` (default `~/.config/NamiShelf/`) | User preferences and local configuration | No |
| Cache | `$XDG_CACHE_HOME/NamiShelf/` (default `~/.cache/NamiShelf/`) | Reconstructible cache and temporary derived data | Yes |
| Alternate downloads | User-selected directory | Permanent downloaded chapter/page artifacts | No |
| Backup exports | User-selected destination | Explicit backup archives | No; governed by backup lifecycle |

The exact internal directory names and whether the SQLite database remains in XDG data or a platform-specific application-data subdirectory are subject to M0 inspection. The separation of data classes is mandatory.

## 2. Permanent downloads on another disk

Users may select a separate directory or mounted volume for permanent chapter/page artifacts. This location is not a cache and must never be included in automatic cache cleanup.

The application must persist a stable location reference and detect when the volume is unavailable. A missing volume must produce a clear unavailable/offline state; it must not be interpreted as an empty library, trigger destructive repair, or cause silent fallback to the cache. When the volume returns, the application should revalidate expected artifacts and restore access.

## 3. Safe relocation workflow

Moving permanent downloads must be explicit and recoverable:

1. Validate destination path, filesystem support, permissions and available space.
2. Identify all referenced permanent artifacts and active operations.
3. Quiesce or safely coordinate writes affecting the relocation set.
4. Copy artifacts and manifests using staging paths.
5. Verify content hashes and manifest references at the destination.
6. Commit the new storage-location mapping in SQLite only after verification.
7. Reopen/revalidate affected chapters from the new location.
8. Retain old data until the new location is confirmed usable; offer cleanup as a separate explicit action.
9. Recover safely after interruption, without marking incomplete relocation as complete.

Cross-filesystem moves must not assume atomic rename semantics. The operation must be restartable and must not destroy the only valid copy.

## 4. Cache safety

- Cache cleanup targets only explicitly disposable cache roots and entries.
- Permanent downloaded pages, chapter manifests, SQLite metadata and user backup exports are never cache-cleanup targets.
- Decoded in-memory LRU, optional disk cache, download staging and permanent downloads are separate classes.
- A cache cleanup operation must not traverse or delete a user-selected download directory.
- User-facing storage reporting should distinguish permanent offline content from reclaimable cache.

## 5. Packaging considerations

- **DEB and AppImage:** respect XDG environment variables and user-selected locations.
- **Flatpak:** use sandbox-visible app data/cache/config roots. Access to external user-selected directories must use supported portal grants or explicitly documented permissions. Do not assume arbitrary host paths are accessible.
- All packaging formats must preserve the same logical storage classes even if physical paths differ.

## 6. Failure handling

- Missing external volume: show library location unavailable; preserve metadata and references.
- Insufficient space: stop before destructive steps and report the required/available capacity where measurable.
- Permission denied: retain existing location and data; do not silently switch.
- Hash mismatch or missing artifact: mark affected content for repair; do not delete unrelated content.
- Crash during relocation: resume or roll back from recorded operation state; retain at least one verified copy.

## 7. Deferred M0 decisions

- Exact app identifier and directory naming/casing.
- Candidate's existing path-resolution abstractions and migration needs.
- Stable external-volume identity and mount-point changes.
- Symlink policy and filesystem compatibility.
- Free-space safety margins and relocation progress UX.
- Flatpak portal integration and permission persistence.
- Backup behavior when downloads are on an external volume.

## 8. Validation scenarios

Validate custom XDG roots, paths containing spaces/non-ASCII characters, external disk disconnect/reconnect, changed mount points, read-only volumes, insufficient space, permission changes, cross-filesystem relocation, process interruption during copy and metadata switch, hash mismatch, cache cleanup while external downloads are configured, and Flatpak portal revocation.

No implementation, build, dependency installation or test execution is authorized by this specification.
