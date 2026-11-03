# NamiShelf — Backup Interoperability Assessment

**Status:** Static assessment; no import/export execution  
**Date:** 2026-11-03  
**Detailed architecture:** [19-mihon-backup-compatibility-architecture.md](19-mihon-backup-compatibility-architecture.md)

## 1. Executive conclusion

Do not treat legacy `.tachibk` as NamiShelf's permanent native backup format. Mihon v0.21.3 is the last version documented to restore Tachiyomi-format `.tachibk`; the next Mihon version drops that legacy compatibility. The public announcement does not specify the replacement format. The inspected v0.21.3 Android source still writes gzip-compressed Protocol Buffers using the `.tachibk` extension.

NamiShelf should therefore maintain a canonical, versioned, platform-neutral backup domain, with independent format adapters:
- legacy Mihon/Tachiyomi `.tachibk` import/export;
- future Mihon format adapter, only after official implementation/specification and fixtures are available;
- NamiShelf-native full-fidelity backup.

**Owner decision (2026-11-03):** NamiShelf-native backup is the default; legacy .tachibk import and export are required in release 1; downloads are exported/migrated separately; portable settings are selected by category.

## 2. Current Mihon Android v0.21.3 contract

The pinned Android source uses Kotlin serialization ProtoBuf, gzip compression, and a streaming entry layout. Top-level repeated library entries are protobuf field 1; metadata fields follow. The reader supports streaming entry-by-entry processing, recognizes gzip by magic bytes, and accepts uncompressed protobuf. Restore processes entries in bounded batches and retries individual entries after a failed batch.

The current data model includes library metadata, chapters, read/bookmark/progress state, categories, history, tracking metadata, source metadata, typed app preferences, source preferences, and extension repository records.

Important exclusions/limitations in the official guide:
- installed extensions/binaries are not backed up;
- downloaded chapter files, including local-source chapters, are not included;
- custom covers are not included;
- tracker authentication is sensitive and should not be included by default;
- extension repository configuration is not equivalent to installed extension binaries.

The exact protobuf field numbers and compatibility aliases are documented in document 19 and must remain isolated inside the legacy adapter.

## 3. Compatibility boundaries

A successful protobuf decode is not semantic compatibility. Source IDs may not map across installations; source preferences require compatible source implementations; category identity/order must be mapped; tracker metadata is not tracker authentication; and desktop-only settings may not exist on Android.

NamiShelf must expose separate capabilities:
- legacy Mihon import;
- legacy Mihon export, with explicit loss reporting;
- future Mihon import/export, only after the new format is inspectable;
- NamiShelf-native full-fidelity backup/restore.

Never claim generic "Mihon compatibility" without naming the format generation and direction.

## 4. Required safeguards

1. Streaming parser with compressed/decompressed byte, per-entry, and count limits.
2. Pre-restore validation and preview.
3. Recovery checkpoint before mutation.
4. Bounded batches, idempotent retry where possible, and explicit partial-restore reporting.
5. Treat imported source preferences and repository endpoints as untrusted; do not auto-enable repositories or execute extensions during restore.
6. Exclude OS credentials, cookies, passwords, sync tokens, signing secrets, and wrapping keys.
7. Maintain real Mihon fixtures per supported release and semantic cross-app tests.
8. Report fields omitted or approximated during legacy export.

## 5. Evidence baseline

- Mihon Android source: `mihonapp/mihon@e588e04359fe1058206a2cbac1b01018ad787359`
- [Official backup guide](https://mihon.app/docs/guides/backups)
- [Mihon v0.21.3 changelog](https://mihon.app/changelogs/v0.21.3)
- [Detailed NamiShelf compatibility architecture](19-mihon-backup-compatibility-architecture.md)

No backup fixtures have been executed and no cross-app compatibility is yet demonstrated.
