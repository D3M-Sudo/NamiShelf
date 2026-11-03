# NamiShelf — Backup Format Evolution and Restore Safety

**Status:** Static review; no backup files decoded or restored  
**Date:** 2026-11-03  
**Detailed architecture:** [19-mihon-backup-compatibility-architecture.md](19-mihon-backup-compatibility-architecture.md)

## 1. Format strategy

The legacy `.tachibk` gzip/ProtoBuf contract must be treated as a compatibility adapter, not automatically selected as NamiShelf's permanent native format. Mihon v0.21.3 is documented as the last version supporting Tachiyomi-format `.tachibk` restore. The next Mihon version removes that legacy support; its replacement format is not specified by the inspected public announcement.

NamiShelf should keep a canonical versioned backup domain independent of wire formats, with separate legacy Mihon, future Mihon, and NamiShelf-native adapters. The native container format and release scope remain owner decisions.

## 2. Confirmed hardening requirements

### BKP-01 — Unbounded backup input

**Priority:** High hardening requirement; **confidence:** High.

The inspected candidate reads complete file bytes and decompresses gzip into memory without visible explicit compressed/decompressed limits. Mihon v0.21.3 Android has a streaming reader/writer, which is a better implementation reference, but streaming alone does not replace strict byte/count limits.

Required: bounded streaming, compressed/decompressed byte caps, per-entry and collection limits, cancellation, malformed/truncated input handling, and fixture-based selection of legitimate limits.

### BKP-02 — Partial restore semantics

**Priority:** Medium; **confidence:** High.

The candidate restore path records per-item errors and can leave a partially applied restore. This is resilient but not globally atomic.

Required: preflight, preview, recovery checkpoint, coherent transaction boundaries where possible, idempotent retries, and explicit partial outcome reporting. Do not claim all-or-nothing restore.

### BKP-03 — Imported configuration trust

**Priority:** Medium; **confidence:** High.

Source preferences and extension repository configuration can affect later network and extension behavior.

Required: treat imported endpoints/settings as untrusted; preview repositories; validate schemes/hosts; revalidate signer fingerprints; never auto-execute extension code during restore.

## 3. Compatibility requirements

- Keep exact legacy protobuf field mappings inside the legacy adapter.
- Do not assume unknown fields survive Mihon read/write round-trips.
- Do not invent the next Mihon format; inspect its official implementation and fixtures when available.
- Maintain versioned golden fixtures for each supported Mihon generation.
- Test semantic import/export, not just successful decoding.
- Report every field omitted or approximated by legacy export.
- Exclude credentials and downloaded chapter bytes by default.

## 4. Required acceptance tests

Compressed/decompressed limit tests; expansion-ratio tests; truncated gzip; malformed protobuf; unknown fields; duplicate identities; large strings/collections; real Mihon fixtures; NamiShelf round trips; Mihon cross-import; cancellation; partial restore; recovery; malicious repository metadata; and secret-exclusion tests.

## 5. References

See [document 19](19-mihon-backup-compatibility-architecture.md) for the pinned Android source files, official guide, and version policy.
