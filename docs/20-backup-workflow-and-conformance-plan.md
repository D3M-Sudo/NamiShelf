# NamiShelf — Backup Workflow and Conformance Plan

**Status:** Planning proposal under owner-approved ADR-003  
**Date:** 2026-11-03

## 1. Scope and approved contract

This plan implements no code. It translates the approved backup decisions into UX, data contracts, test evidence, and remaining design choices.

Approved:
- NamiShelf-native versioned backup is the default and canonical full-fidelity format.
- Release 1 supports both import and export of legacy Mihon/Tachiyomi .tachibk.
- Legacy export must report unsupported, omitted, and approximated fields.
- Downloaded chapters use a separate export/migration operation.
- Portable settings are selected by backup category.
- Mihon future-format support is conditional on an official inspectable contract and fixtures.

## 2. Backup operations and UX

Keep three distinct user actions:
1. **Create NamiShelf backup** — default operation for portable library state and selected settings.
2. **Import/export Mihon backup** — explicit compatibility workflow, naming the detected Mihon format generation and direction.
3. **Export/migrate downloads** — separate, potentially large operation with independent progress, destination, verification, and resume behavior.

Do not label a legacy .tachibk export as a full NamiShelf backup. Warn before export that NamiShelf-only fields cannot be represented in the legacy format.

### Backup creation flow
- Choose backup destination.
- Select portable data categories; show default selections and exclusions.
- Display estimated item counts and approximate size where available.
- Confirm secret/device-local exclusions.
- Stream creation to a temporary file.
- Finalize manifest and integrity metadata.
- Flush and atomically move into destination where supported.
- Verify the completed archive before reporting success.
- Preserve previous valid backup if creation is interrupted.

### Restore flow
- Inspect archive without mutating application state.
- Validate container, schema, integrity, size/count limits, and compatibility.
- Show a preview: library entries, chapters, categories, history, tracking links, settings categories, source mappings, and skipped/unsupported items.
- Create a recovery checkpoint.
- Ask how to handle duplicates/conflicts.
- Restore in bounded batches with cancellation and resumable/diagnosable partial outcomes.
- Report restored, skipped, merged, failed, and unresolved counts separately.

## 3. Portable category proposal

| Category | Native backup default | Legacy .tachibk mapping |
|---|---|---|
| Library metadata and favorite state | Selected by default | Supported subset |
| Chapter read/bookmark/progress | Selected by default | Supported subset |
| Categories | Selected by default | Supported subset |
| Reading history | Selected by default | Supported subset |
| Tracker associations and progress | Selected by default | Supported metadata; no credentials |
| Portable app preferences | Opt-in category | Mihon-compatible primitive subset |
| Source preferences | Opt-in category, trust review | Compatible source subset |
| Extension repository metadata | Opt-in category, never auto-enable | Compatible subset |
| Reader/device presentation | Opt-in only if explicitly portable | Usually omitted |
| Window geometry, local paths, caches | Excluded | Omitted |
| Credentials, cookies, tokens, signing keys | Always excluded | Always excluded |
| Downloaded chapter files | Separate operation | Not in .tachibk |

The final taxonomy and per-setting allowlist must be defined from actual NamiShelf and Mihon settings, not guessed from names.

## 4. Legacy .tachibk export loss report

Before writing, compute a compatibility report containing supported/exportable item counts; fields omitted because the legacy schema has no representation; values approximated or normalized; entries skipped due to missing/unsupported source identity; source/tracker metadata that cannot be mapped; categories/settings not representable; and whether output is expected to be restorable by Mihon v0.21.3 or a named compatible fork.

Require explicit confirmation when any data is lost. Store the report alongside the operation result; embedding it in the .tachibk file is not assumed safe or compatible.

## 5. Separate downloads export/migration

The downloads archive is not an ordinary backup. It should carry stable NamiShelf manga/source identity plus original source identifiers/URLs; chapter identity, URL, ordering, and file list; per-file size and cryptographic checksum; archive manifest and schema version; and optional metadata needed to re-link files after source ID changes.

Requirements: preserve relative structure, never absolute host paths; stream large files and support cancellation/resume; detect missing, truncated, duplicate, and corrupt files; verify checksums before import; never auto-trust imported source endpoints or execute extensions; allow library restore even when download archive restore fails.

Archive compression, deduplication, encryption, and whether downloads can be selectively exported remain design choices.

## 6. Native container design options

The native format must be documented and versioned independently of the internal database. Candidate approaches:
- **ZIP-based container:** manifest plus structured data and optional attachment entries. Advantages: inspectable, tooling availability, streaming entry support. Risks: zip-bomb handling, duplicate paths, path traversal, metadata ambiguity.
- **Tar-based container with compression:** stream-friendly and suitable for large payloads. Risks: portability/tooling nuances and archive path safety.
- **Single binary envelope:** explicit header, manifest, and framed payload records. Advantages: tight control and streaming. Risks: custom tooling and harder manual inspection.

Compare streaming behavior, deterministic output, random access, recovery after truncation, encryption integration, metadata leakage, interoperability, and mature library support. Require strict path canonicalization and decompression limits regardless of container.

## 7. Integrity and encryption decision framework

Integrity is mandatory: manifest version, payload lengths, cryptographic hashes, and a final archive-level integrity check. Hashes detect corruption but do not authenticate an attacker-controlled archive.

Optional encryption is a separate decision. If selected, use a well-reviewed authenticated-encryption construction and memory-hard password key derivation or a clearly specified recipient-key model. Never reuse sync credentials or OS wrapping keys as archive keys. Define recovery behavior for forgotten passwords/lost keys before enabling encryption. Avoid claiming authenticity from an unkeyed checksum. Keep secrets and credentials out even when encryption is enabled.

The exact cryptographic profile requires a dedicated threat-model review before implementation.

## 8. Conformance fixture matrix

Maintain immutable, provenance-recorded fixtures: official Mihon v0.21.3 .tachibk produced by Android; older supported Tachiyomi/Mihon generations; representative compatible-fork backup with documented provenance; malformed/truncated gzip and protobuf; large library and metadata fixtures; duplicate source IDs and changed source URLs; unknown protobuf fields and deprecated aliases; tracker/category/history edge cases; NamiShelf native backups from each released schema version; downloads archives with missing, duplicate, corrupt, and renamed files.

For each fixture, record origin app/version, source commit if known, generation method, expected semantic contents, and redistribution/licensing constraints.

## 9. Required test layers

1. **Parser tests:** limits, truncation, malformed data, duplicate fields/paths, unknown fields, cancellation.
2. **Semantic mapping tests:** source identity, chapter progress, categories, history, tracking, preferences.
3. **Round-trip tests:** NamiShelf native create→restore; legacy import→native; native→legacy with loss report.
4. **Cross-app tests:** restore exported .tachibk in Mihon v0.21.3; import genuine Mihon fixture into NamiShelf.
5. **Recovery tests:** interrupted write, interrupted restore, checkpoint rollback/recovery, partial failure.
6. **Security tests:** path traversal, zip bombs, oversized strings/counts, malicious repository URLs, secret leakage.
7. **Migration tests:** source profile preserved; downloads independently verified and re-linked.

No compatibility claim is allowed until the corresponding fixture and cross-app tests pass.

## 10. Remaining owner decisions

The high-level backup policy is approved. Before implementation, present a focused decision on native container choice; whether native backup encryption is mandatory, optional, or deferred; exact default portable categories and source-preference trust UX; and whether downloads export is a directory/archive and supports selective manga/chapter scope.

No implementation, build, restore, or fixture execution has been performed.