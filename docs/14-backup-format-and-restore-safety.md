# NamiShelf — Backup Format Evolution and Restore Safety

**Status:** Static codec/restore review; no backup files decoded or restored  
**Reviewed candidate:** `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Date:** 2026-11-03

## 1. Executive assessment

The candidate uses the shared Mihon `BackupCodec` for gzip-compressed protobuf backup files and adds a desktop backup model/restore flow. This is the correct starting point for interoperability. However, the inspected codec reads the entire decompressed gzip stream into memory without an explicit decompressed-size or expansion-ratio limit, and the desktop file reader uses `File.readBytes()` without a visible file-size limit.

This is a confirmed hardening requirement for NamiShelf because backup files are user-supplied input and may be malformed or intentionally oversized. This finding is not a claim that a vulnerability has been exploited or that Mihon upstream is uniquely affected; the codec is shared code.

## 2. Observed format behavior

- Encode: protobuf serialization followed by gzip.
- Decode: rejects empty/JSON-like input; accepts gzip or uncompressed protobuf; normalizes wire compatibility for the Mihon Backup serializer; validates that the decoded backup contains restorable data.
- Desktop reader: requires a regular `.tachibk` file and decodes the full file bytes.
- Restore: processes preferences, source preferences, extension repository records, categories, library entries, chapters, history, tracking, and optional author archive data; it accumulates per-item errors and reports partial outcomes.
- Restore supports excluding application preferences, but other settings and source preferences remain part of the import path.

## 3. Findings

### BKP-01 — Unbounded compressed and decompressed backup input

**Severity:** High hardening priority; **confidence:** High.

Evidence: `DesktopBackupCreator.readBackupFile` reads the complete file with `readBytes()`; `BackupCodec.decode` decompresses gzip with `readByteArray()`. No explicit maximum compressed size, maximum decompressed size, or expansion-ratio bound is visible in these paths.

Impact: a very large or highly compressible backup can cause excessive heap use or process termination before protobuf semantic validation. A malicious backup can also impose excessive CPU/memory cost.

Required remediation direction: bounded streaming input, compressed-size cap, decompressed-size cap, protobuf collection/string limits where possible, and cancellation/timeout. Establish limits using legitimate large-library fixtures before choosing product defaults.

### BKP-02 — Restore is not an all-or-nothing transaction

**Severity:** Medium; **confidence:** High.

Evidence: restore applies categories/preferences/repositories and then iterates through library entries, catching per-item exceptions and recording partial failures. It reports `PARTIAL` outcome when errors exist. The observed flow is intentionally resilient but is not a single atomic transaction across all imported data.

Impact: after a partial failure, the database may contain a mixture of restored and pre-existing state. Users need a preview, checkpoint, clear partial report, and recovery guidance.

Required remediation: preflight validation, snapshot/backup before restore, conflict preview, transaction boundaries for coherent units, idempotent retry semantics, and explicit partial-restore UX. Do not claim global atomicity.

### BKP-03 — Source preferences and extension repositories are executable-adjacent configuration

**Severity:** Medium; **confidence:** High.

Evidence: restore writes per-source preferences and extension repository configuration; repository restore checks signing-key fingerprint conflicts. These values can affect later network/source/extension behavior.

Impact: a backup is not merely passive reading progress data. Imported repository URLs and source settings can influence future trust and network operations.

Required remediation: show imported repositories before activation, validate schemes/hosts, re-run signer verification, preserve existing trust on fingerprint mismatch, and keep network credentials out of the backup.

## 4. Format evolution recommendation

Retain the Mihon `.tachibk` format as the interoperability layer. Do not fork or silently mutate the canonical protobuf schema.

For NamiShelf-specific data:
- prefer a versioned, namespaced optional protobuf extension section only if the wire contract can preserve unknown fields safely across Mihon versions;
- otherwise use a separate NamiShelf companion manifest/file in a documented archive/container;
- never make ordinary Mihon restore depend on NamiShelf-only sections;
- define downgrade behavior and loss reporting;
- include a schema/version migration policy and fixture corpus.

The exact extension mechanism is a design decision, not settled by this assessment.

## 5. Required acceptance tests

1. Reject compressed files above a documented input limit before allocating full content.
2. Reject decompressed streams above a documented output limit.
3. Test high expansion-ratio gzip inputs.
4. Test truncated gzip, malformed protobuf, unknown fields, duplicate identities, and oversized strings/collections.
5. Test real Mihon backups from each supported compatibility release.
6. Test NamiShelf export/import round trips and Mihon cross-import.
7. Test partial restore, cancellation, retry, and pre-restore recovery.
8. Test untrusted extension repository URLs and signer fingerprint conflicts.
9. Prove tokens, cookies, and sync credentials are excluded.
10. Verify backup preview and clear reporting of data not included.

## 6. Recommendation

Use Mihon protobuf/gzip as the base interoperability contract, but harden parser resource limits before release and avoid extending the canonical format until the owner chooses the NamiShelf-specific data strategy.

## 7. Source references

- [BackupCodec.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/data/src/commonMain/kotlin/tachiyomi/data/backup/BackupCodec.kt)
- [DesktopBackupCreator.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/backup/DesktopBackupCreator.kt)
- [DesktopBackupRestorer.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/backup/DesktopBackupRestorer.kt)
