# NamiShelf — Mihon Backup Compatibility Architecture

**Status:** Owner-approved architecture; format implementation details remain open  
**Research date:** 2026-11-03  
**Mihon reference:** `mihonapp/mihon@e588e04359fe1058206a2cbac1b01018ad787359` (v0.21.3 source baseline)

## 1. Executive conclusion

NamiShelf must not adopt the legacy `.tachibk` file format as its permanent native backup contract.

Mihon officially identifies v0.21.3 as the last major version able to restore Tachiyomi-format `.tachibk` backups. The next version is announced to drop that compatibility. The public announcement explains the removal but does not publish a replacement format specification. The inspected public v0.21.3 source still creates gzip-compressed Protocol Buffers backups with a `.tachibk` filename.

Therefore:
- treat `.tachibk` as a **legacy interoperability format**, not NamiShelf's canonical native format;
- use the owner-approved NamiShelf-native ZIP container with canonical JSON payloads, optional per-backup encryption, and full-library selection by default; portable settings remain category-selectable;
- retain a dedicated import/export adapter for the last compatible Mihon/Tachiyomi generation;
- do not invent or assume the future Mihon format;
- inspect the first Mihon release/source that implements the replacement before specifying a future-format adapter;
- keep NamiShelf's internal backup model independent of either wire format.

## 2. What Mihon v0.21.3 actually implements

The inspected Android implementation has these characteristics:

1. The file payload is Protocol Buffers serialized with Kotlin serialization's ProtoBuf support.
2. The output stream is gzip-compressed.
3. The standard filename ends in `.tachibk`.
4. The top-level protobuf message stores repeated library entries as field 1, followed by metadata fields.
5. The streaming writer serializes each `BackupManga` entry independently, writes its field tag and length, then writes the remaining top-level metadata.
6. The reader scans the protobuf stream, exposes one library entry at a time, and retains non-entry metadata for later decoding.
7. The decoder detects gzip by magic bytes and also accepts uncompressed protobuf; JSON-like legacy input is rejected.
8. Restore processes library entries in chunks of 100 and retries individual entries if a batch restore fails.

The v0.21.3 changelog reports backup creation/restoration improvements focused on speed and memory. This streaming design is material: NamiShelf should study the current Android implementation rather than rely only on the older desktop candidate codec.

## 3. Mihon backup data contract

The official guide and source models identify these data groups:

| Group | Data represented | Important caveats |
|---|---|---|
| Library entries | Source ID, source URL, title, artist/author, description, genres, status, cover URL, favorite state/date | Source identity may not resolve on desktop |
| Chapters | URL, name, scanlator, read/bookmark state, page progress, fetch/upload dates, chapter number/order, memo | Downloaded chapter image files are not included |
| Categories | Name, order, ID, flags | Category IDs/order need semantic mapping |
| Tracking | Tracker ID, remote media/library IDs, URL/title, progress, score/status, dates, privacy flag | Tracker credentials are excluded by default |
| History | Chapter URL, last-read timestamp, read duration | Entries depend on chapter/source identity |
| Source metadata | Source ID and display name | Does not install the source implementation |
| App settings | Typed primitive preferences | Device/platform-specific settings may not be portable |
| Source settings | Per-source typed preferences | Requires compatible source and preference schema |
| Extension stores | Repository metadata | Does not include installed extension APKs/binaries |
| Sensitive settings | Tracker login tokens when explicitly selected | Excluded by default; NamiShelf must apply stricter secret exclusions |

The official guide says backups do not include extensions, downloaded chapter files (including local-source chapters), or custom covers. These exclusions must be visible in NamiShelf's backup UX.

## 4. Important compatibility details in the wire schema

The current model uses explicit protobuf field numbers. For example:
- top-level `Backup`: library entries field 1, categories field 2, sources field 101, app preferences field 104, source preferences field 105, extension stores field 106;
- `BackupManga`: source field 1, URL field 2, descriptive metadata fields 3–9, date added field 13, viewer field 14, chapters field 16, categories field 17, tracking field 18, and newer Mihon-specific fields in the 100+ range;
- `BackupChapter`: URL/name fields 1–2, read/bookmark/progress and date/number fields 4–10, memo field 13;
- `BackupTracking`: tracker ID field 1, library ID field 2, legacy media ID field 3, tracking URL/title/progress/status/date fields 4–12, newer media ID field 100.

These numbers are wire-contract details, not a recommended NamiShelf database schema. Preserve them only inside the legacy adapter. Do not reuse them as NamiShelf domain identifiers.

The Android models explicitly retain compatibility aliases and fields for older 1.x/0.x backups, including renamed values, old field numbers, and deprecated representations. This is evidence that compatibility requires semantic mapping, not merely protobuf decoding.

## 5. Recommended NamiShelf backup architecture

### 5.1 Canonical domain model

Define a versioned, platform-neutral NamiShelf backup domain independent of Mihon protobuf classes. It should model:
- library identities and source references;
- chapter state and reading progress;
- categories;
- history;
- tracker associations without authentication secrets;
- portable preferences through an explicit allowlist;
- source configuration through a trust-aware import model;
- NamiShelf-specific data such as reader settings or desktop metadata, each explicitly classified as portable or device-local.

Use stable NamiShelf identifiers and preserve source-native identifiers as external references. Do not assume Mihon numeric source IDs are stable across unrelated installations.

### 5.2 Format adapters

Keep format handling at the boundary:

- **Mihon legacy adapter:** import and export the v0.21.3-compatible gzip/ProtoBuf `.tachibk` contract, with a clearly labeled compatibility mode and field-level loss report.
- **Future Mihon adapter:** add only after the new Mihon release/source provides an inspectable, versioned format contract and fixtures.
- **NamiShelf native adapter:** serialize the canonical domain in a NamiShelf-owned, explicitly versioned container. Its extension, manifest, schema evolution, integrity, and optional encryption remain detailed design tasks within the approved NamiShelf-native direction.

Do not make the canonical NamiShelf data model equal to any one adapter DTO.

### 5.3 Import/export direction

Support these as separate capabilities rather than claiming one undifferentiated "Mihon compatible" feature:

| Direction | Proposed capability | Gate |
|---|---|---|
| Mihon v0.21.3 `.tachibk` → NamiShelf | Legacy import | Golden fixtures and semantic mapping |
| NamiShelf → Mihon v0.21.3 `.tachibk` | Legacy export | Mihon restore fixtures; explicit unsupported-field report |
| Future Mihon backup → NamiShelf | Future import | New official format/spec and fixtures |
| NamiShelf → future Mihon | Future export | New official format/spec and fixtures |
| NamiShelf native backup ↔ NamiShelf | Full-fidelity native round trip | Schema migration and recovery tests |

A legacy export must never silently discard NamiShelf-only data. Report every omitted or approximated field. Do not place NamiShelf-only fields in unknown protobuf fields unless preservation through Mihon import/export is demonstrated.

## 6. Compatibility policy

- Keep legacy `.tachibk` support as an explicit migration/interchange path for users on Mihon v0.21.3 and compatible forks.
- Do not promise that the legacy file can be restored by future Mihon versions.
- Do not claim compatibility with the upcoming Mihon backup format until it is publicly specified and tested.
- Track Mihon backup changes by release/tag and source commit, not by filename alone.
- Maintain independent fixture sets for every supported Mihon format generation.
- Keep NamiShelf native backup readable across NamiShelf upgrades through explicit schema migrations.

## 7. Security, reliability, and restore requirements

- Stream gzip and protobuf parsing with strict compressed/decompressed byte limits, per-entry limits, count limits, and cancellation.
- Validate magic/signature and schema version before mutation.
- Preview entry counts, source/tracker availability, settings, and unsupported fields.
- Create a recovery checkpoint before restore.
- Restore in bounded batches with per-item diagnostics and a clear partial-restore result.
- Treat imported repository URLs, source preferences, and settings as untrusted data; do not auto-enable repositories or execute extension code during restore.
- Never export OS credentials, sync tokens, cookies, passwords, extension signing secrets, or wrapping keys.
- Keep downloaded chapter files and caches outside the portable backup by default; provide a separately documented downloads migration if required.
- Make restore idempotent where possible and test duplicate identities, URL changes, source remapping, and interrupted restores.

## 8. Evidence and references

Official Mihon documentation:
- [Backups guide](https://mihon.app/docs/guides/backups)
- [v0.21.3 changelog](https://mihon.app/changelogs/v0.21.3)
- [Changelogs](https://mihon.app/changelogs/)

Pinned Android implementation paths:
- [BackupStream.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/BackupStream.kt)
- [BackupDecoder.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/BackupDecoder.kt)
- [BackupCreator.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/create/BackupCreator.kt)
- [BackupRestorer.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/restore/BackupRestorer.kt)
- [Backup.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/models/Backup.kt)
- [BackupManga.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/models/BackupManga.kt)
- [BackupChapter.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/models/BackupChapter.kt)
- [BackupTracking.kt](https://github.com/mihonapp/mihon/blob/e588e04359fe1058206a2cbac1b01018ad787359/app/src/main/java/eu/kanade/tachiyomi/data/backup/models/BackupTracking.kt)

## 9. Remaining design work (no longer owner-level format choice)

1. Select the native container encoding and file extension; specify manifest, schema versioning, integrity checks, and optional encryption.
2. Define the precise portable-setting categories and device-local exclusions.
3. Define a separate downloads export/migration archive, including source identity, checksums, partial-download handling, and re-linking semantics.
4. Define loss-report format and UI for legacy .tachibk export.
5. Obtain fixtures from Mihon v0.21.3 and representative compatible forks; test import and export against real Mihon restore behavior.
6. Inspect the next Mihon release's replacement backup implementation before designing that adapter.

No backup implementation, build, restore, or fixture execution has been performed.
