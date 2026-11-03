# NamiShelf — Backup Interoperability and Recovery Assessment

**Status:** Static assessment of the candidate; no import/export execution  
**Reviewed candidate:** `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Date:** 2026-11-03

## 1. Executive assessment

The candidate contains a desktop backup implementation that explicitly uses the same gzip-compressed protobuf `.tachibk` wire format as its Android code. This is a strong reuse opportunity, but it is not yet evidence of complete compatibility with every Mihon release, every backup section, or every Android extension's data.

The code assembles backups from repositories and preferences and includes library works, chapters, categories, history, tracker records, application preferences, source preferences, extension repositories, and an optional author archive section. It filters app-state preferences. The exact supported fields must be checked against the pinned upstream version and fixture set.

## 2. Observed implementation

### Export

`DesktopBackupCreator`:
- delegates encoding/decoding to the shared `BackupCodec`;
- writes gzip/protobuf bytes with a `.tachibk` extension;
- exports favorited works and their chapter metadata, reading/bookmark state, progress, URL identity/aliases, categories, history, tracking, and notes/memo fields;
- exports app preferences except app-state keys;
- exports per-source preferences for sources represented in the library;
- exports extension repository configuration and optional author archive data.

### Restore

`DesktopBackupRestorer` restores categories, works, chapters, history, tracking, preferences, source preferences, extension repositories, and optional author archive data. It supports a setting that excludes application preferences from restoration, reports progress, records partial errors, and wraps restore with a sync recovery observer/outcome.

The desktop file boundary accepts local paths and `file:` URIs, requires a regular `.tachibk` file, and rejects unsupported URI schemes. This is desktop-specific file handling, not Android ContentResolver/Uri parity.

## 3. Important boundaries and non-assumptions

- A shared wire format is not proof of semantic parity for every field.
- Backup is not a full device image. It does not imply that installed extensions, downloaded chapter image bytes, caches, credentials, or all OS-specific state are included.
- Extension repository configuration is not the same as exporting installed extension binaries.
- Tracking metadata in a backup is not equivalent to transferring tracker authentication tokens.
- Source preferences may refer to source IDs or source implementations unavailable on another installation.
- A successful protobuf decode does not prove the restore is semantically safe or complete.
- Restore may be partial; users need a preview and actionable per-item error report.

## 4. NamiShelf requirements

1. Maintain versioned compatibility fixtures produced by real Mihon Android releases and the candidate desktop implementation.
2. Test import of legacy `.tachibk` fixtures and round-trip export/import without claiming byte-for-byte equality where ordering or timestamps legitimately differ.
3. Document every included and excluded data category.
4. Preview counts and conflicts before applying restore.
5. Provide explicit control over restoring application settings and source settings.
6. Make restore transactional where possible, or clearly report partial commits and provide recovery/rollback strategy.
7. Test duplicate work identity, source ID remapping, chapter URL aliases, category mapping, history, tracking records, notes/memo, and malformed inputs.
8. Never restore or export global credentials as ordinary backup preferences.
9. Treat extension repositories and source preferences as untrusted configuration; revalidate endpoints, signatures, and policy before use.
10. Provide a safe pre-restore backup or recovery checkpoint.

## 5. Required acceptance matrix

| Area | Evidence required | Status |
|---|---|---|
| Mihon Android import | Real fixtures from supported releases | Open |
| NamiShelf export → Mihon import | Cross-app fixture tests | Open |
| Mihon export → NamiShelf import | Cross-app fixture tests | Open |
| Categories/library/chapter state | Field-level semantic comparison | Open |
| History and tracking | Field-level comparison | Open |
| Source preferences/repos | Validation and unavailable-source behavior | Open |
| Author archive extension section | Versioned optional-section handling | Open |
| Corrupt/oversized input | Bounded parser and failure tests | Open |
| Partial restore/recovery | Failure injection and recovery tests | Open |
| Credentials/download bytes | Explicit exclusion proof | Open |

## 6. Recommendation

Retain Mihon-compatible `.tachibk` as the primary interoperability target, but define a NamiShelf backup compatibility policy and fixture corpus before declaring compatibility. Keep NamiShelf-specific data in versioned optional sections or a separately specified extension format; do not silently alter upstream protobuf semantics.

## 7. Source references

- [DesktopBackupCreator.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/backup/DesktopBackupCreator.kt)
- [DesktopBackupRestorer.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/backup/DesktopBackupRestorer.kt)
- [Backup restore factory](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/backup/BackupRestoreScreenModelFactory.kt)
- [Mihon backup guide](https://mihon.app/docs/guides/backups)
