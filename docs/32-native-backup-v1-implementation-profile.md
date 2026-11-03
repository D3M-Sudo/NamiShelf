# NamiShelf — Native Backup Format v1 Implementation Profile

**Status:** Proposed engineering profile based on owner-approved ZIP + JSON + optional encryption decisions. Cryptographic envelope remains subject to independent review.

## 1. File identity

- Proposed extension: .nsbackup.
- Container: ZIP, ZIP64 where necessary.
- Proposed MIME type: application/vnd.d3m-sudo.namishelf-backup+zip.
- Format identifier: org.d3m-sudo.namishelf.backup.
- Container format version is independent from NamiShelf application version.
- A deterministic NamiShelf magic/header must be checked before ZIP parsing so arbitrary ZIP files are not mistaken for NamiShelf backups. Exact header bytes are fixed in the implementation schema.

## 2. Canonical archive layout

    backup.nsbackup
    ├── manifest.json
    ├── payload/
    │   ├── library.json
    │   ├── chapters.json
    │   ├── categories.json
    │   ├── history.json
    │   ├── tracking.json
    │   └── settings/
    │       ├── application.json
    │       └── sources.json
    └── integrity/
        └── checksums.json

The manifest is first and small. Payloads are domain-oriented, not database table dumps. Only selected settings categories are present. Excluded or unselected payloads are absent, not represented by misleading empty files.

For encrypted backups, the archive contains a versioned encrypted envelope and only the minimum metadata required to derive and authenticate the payload. Do not expose library titles, source names, or user identifiers in cleartext metadata unless a later explicit design chooses that trade-off.

## 3. JSON and schema rules

- UTF-8, RFC 8259 JSON.
- Canonical serialization: RFC 8785 JSON Canonicalization Scheme (JCS).
- Schema validation: JSON Schema Draft 2020-12 or a stricter equivalent.
- Reject duplicate keys, non-finite numbers, invalid UTF-8, unpaired surrogates, excessive nesting, and unknown required features.
- Timestamps use UTC RFC 3339; IDs are stable opaque strings; numeric precision is explicitly bounded.
- Missing and null are distinct and defined per field.
- Unknown optional fields may be preserved only where schema explicitly permits; unknown required features stop restore.
- Never serialize database row IDs as public stable identities unless explicitly part of the domain contract.

## 4. Manifest minimum fields

format_id, format_version, producer_id, producer_version, created_at, library_scope (initially all), settings_categories, payload inventory (relative path, media type, schema version, byte length, SHA-256 digest), integrity algorithm/profile, and encryption envelope descriptor when enabled.

Never include passwords, keys, tokens, cookies, credential identifiers, absolute local paths, device IDs that identify a user, or extension signing secrets.

## 5. Integrity and encryption

Plaintext backup:
- SHA-256 per canonical payload.
- Verify declared size and digest before restore preview.
- Clearly state that unkeyed hashes detect accidental corruption but do not authenticate the publisher.

Encrypted backup:
- Use an authenticated encryption envelope; do not use legacy ZIP encryption.
- Authenticate format version, envelope version, and payload inventory as associated data.
- Bound KDF parameters before running the KDF.
- Fresh random salt and nonce according to the selected primitive.
- Fail closed on authentication failure; never fall back to plaintext parsing.
- No password escrow or silent recovery.
- Cryptographic primitive/library/KDF selection requires independent review of JVM support, Linux packaging, known-answer vectors, and resource-exhaustion limits before implementation freeze.

## 6. ZIP parser profile and safety

- Permit only STORE and DEFLATE; ZIP64 permitted.
- Reject encrypted ZIP entries (NamiShelf encryption is the outer envelope), symlinks, special files, absolute paths, parent traversal, backslash ambiguity, duplicate normalized paths, duplicate entry names, and unlisted entries.
- Never extract archive entries directly to arbitrary paths.
- Stream payloads to bounded temporary storage; validate names and sizes before opening output.
- Reject compression ratios above a conservative configured threshold; do not trust ZIP metadata alone.
- Initial engineering ceilings for fixture validation: 4 GiB total archive, 2 GiB total decompressed content, 1 GiB per entry, 100,000 entries, 1 MiB manifest, 1 MiB individual filename, compression ratio 100:1. These are configurable safety ceilings, not guarantees that a device can restore an archive of that size; lower effective limits may be required on constrained hardware.
- Enforce cumulative and per-entry limits during actual decompression, not only from central-directory metadata.
- Detect CRC/digest mismatch, truncation, overlapping entry data, and trailing unexpected content.

## 7. Restore transaction

1. Detect NamiShelf magic and format version.
2. Bound input size and parse container metadata without extraction.
3. If encrypted, validate KDF parameters against hard bounds, derive key, authenticate, then decrypt to bounded temporary storage.
4. Validate ZIP structure, canonical paths, entry allowlist, sizes and compression limits.
5. Parse manifest and validate every schema and digest.
6. Build a read-only restore preview: new/updated/conflicting/skipped records, source mapping, settings categories, unsupported fields.
7. Create a recovery checkpoint.
8. Apply bounded batches in a transaction strategy that supports rollback or a clearly reported partial result.
9. Report imported, merged, skipped, failed, and unresolved counts; retain diagnostic correlation IDs without content secrets.
10. Clean temporary files and key buffers on success, failure, cancellation, or crash recovery.

No persistent profile mutation is allowed before the user confirms the preview.

## 8. Portable settings and trust boundary

- Portable settings are selected by category.
- OS paths, window geometry tied to a monitor, cache state, device-local preferences, credentials, cookies, sync state, provider tokens, E2EE keys, and extension binaries are excluded.
- Imported source settings and repository metadata are untrusted input. Never auto-enable repositories or execute extensions after restore.
- Restore settings with a preview and category-level confirmation.

## 9. Compatibility adapters

- .tachibk remains a separate legacy import/export adapter based on the last compatible Mihon v0.21.3 protobuf/gzip implementation.
- Legacy export produces a field-level loss report.
- Downloads remain a separate archive/migration operation with independent identity/checksum/relink semantics.
- Future Mihon backup adapters are added only against an inspectable official implementation and fixtures.

## 10. Required conformance fixtures

Malformed ZIP headers; duplicate names and normalized paths; traversal; symlinks; ZIP64; unsupported compression; compression bombs; truncated streams; CRC/digest mismatch; malformed/duplicate-key JSON; unknown required schema version; wrong password; tampered ciphertext/metadata; KDF parameter abuse; cancellation during each restore stage; disk-full; interrupted checkpoint/apply; partial restore; old/new schema migration; legacy .tachibk import/export and loss report.

No backup implementation, build, restore, or fixture execution has been performed.
