# NamiShelf — Native Backup Format Contract Draft

**Status:** Technical draft based on ADR-003 and ADR-005; not a frozen implementation contract
**Date:** 2026-11-03

## 1. Approved constraints

- ZIP container; ZIP64 features where required.
- Canonical JSON for structured payloads.
- Encryption optional per backup.
- Full library selected by default; portable settings selected by category.
- Downloaded chapter files exported/migrated separately.
- Mihon `.tachibk` remains a separate legacy import/export adapter.
- No credentials, cookies, tokens, signing keys, OS key material, or device-local absolute paths in any backup.

## 2. Proposed logical archive layout

```text
<backup-file>
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
```

Names are provisional. The archive must not mirror database tables; payloads represent stable domain concepts. Omit payloads for unselected settings categories rather than writing empty or misleading placeholders.

## 3. Manifest contract proposal

The manifest should identify the format independently from the application release and payload schema. Proposed fields:

| Field | Purpose |
|---|---|
| `format_id` | Stable identifier such as `org.d3m-sudo.namishelf.backup` |
| `format_version` | Major/minor container contract version |
| `producer` | Application identifier and version |
| `created_at` | UTC timestamp in a documented RFC 3339 representation |
| `library_scope` | `all` for the approved default; future partial scope only after a separate decision |
| `settings_categories` | Explicit portable settings categories included |
| `payloads` | Path, media type, schema version, byte length, and digest for each payload |
| `integrity` | Digest algorithm and integrity-record location |
| `encryption` | Absent for plaintext archives; otherwise envelope version and non-secret KDF/AEAD parameters |

Do not place passwords, keys, tokens, credential identifiers, or sensitive source authentication data in the manifest.

## 4. Canonical JSON profile — requirements

Select and document a recognized canonical JSON profile rather than inventing ad-hoc key sorting. The profile must define UTF-8 handling, object key ordering, number serialization, escaping, Unicode normalization policy, duplicate-key rejection, and treatment of absent versus null fields.

All payloads must be validated against versioned schemas before restore. Reject duplicate object keys and non-finite numeric values. Preserve unknown optional fields only where the schema explicitly allows it; unknown required features must stop restore with a compatibility error.

## 5. Integrity model

- Calculate a digest for each payload over its exact canonical serialized bytes.
- Record payload byte lengths and digests in the manifest/integrity structure.
- Define a deterministic archive-level integrity value over a canonical representation of the manifest and ordered payload inventory.
- Verify all lengths and digests before exposing the restore preview as trusted.
- Clearly distinguish corruption detection from publisher authenticity: unkeyed hashes do not prove who created a backup.
- If encrypted, authenticated encryption must cover the complete logical payload set and relevant metadata; define which metadata remains visible.

Exact digest algorithm, manifest self-reference avoidance, and authenticated archive-level construction require a security review before freeze.

## 6. ZIP interoperability and parser safety

Specify the permitted ZIP compression methods, data descriptor behavior, ZIP64 support, UTF-8 filename rules, and whether archive comments/extras are allowed. Prefer a narrow feature subset to reduce parser ambiguity.

Reject absolute paths, parent traversal, backslashes where they create platform ambiguity, duplicate normalized paths, symlinks, special files, encrypted ZIP entries outside the NamiShelf envelope, unsupported compression methods, and unexpected entries. Never extract directly to arbitrary filesystem paths.

Enforce configurable but hard maximums for compressed bytes, decompressed bytes, entry count, individual entry size, filename length, and compression ratio. Limits must be selected from representative fixture measurements and target hardware, not guessed here.

## 7. Optional encryption envelope

Encryption is an explicit per-backup choice. Plaintext archives remain valid NamiShelf backups, but the UI must clearly indicate the security implications before creation.

When enabled:
- Encrypt the logical archive using a reviewed authenticated-encryption construction, not legacy ZIP encryption.
- Use a reviewed password-based KDF with parameters stored in the envelope; parameters must be bounded before KDF execution to prevent resource-exhaustion attacks.
- Generate a fresh random salt and nonce as required by the selected algorithms.
- Authenticate format/version metadata and ciphertext; fail closed on authentication failure.
- Do not offer silent password recovery or store the password in application settings.
- Document that a lost password means the encrypted backup cannot be recovered unless a separately designed recovery mechanism is approved.
- Keep key material out of logs, crash reports, telemetry, sync metadata, and extension IPC.

Algorithm selection, KDF profile, parameter bounds, nonce construction, metadata leakage, and password UX remain subject to a dedicated cryptographic review.

## 8. Restore state machine

1. **Identify:** inspect magic/header and format identifier without extracting.
2. **Bound:** enforce compressed-size and parser resource limits.
3. **Authenticate:** if encrypted, validate envelope parameters and authenticate before parsing plaintext.
4. **Parse:** validate ZIP structure, paths, entries, manifest, and JSON schemas.
5. **Verify:** check lengths and all payload digests.
6. **Preview:** calculate source mappings, duplicate/conflict counts, unsupported fields, and settings changes.
7. **Checkpoint:** create a recoverable pre-restore checkpoint.
8. **Apply:** restore bounded batches with cancellation and explicit conflict policy.
9. **Report:** distinguish applied, merged, skipped, failed, and unresolved records.

No profile mutation may occur during identification, parsing, verification, or preview.

## 9. Open design decisions before format freeze

1. Confirm file extension and whether encrypted archives use the same extension with a detectable envelope header.
2. Select canonical JSON standard/profile and schema-validation technology.
3. Select digest and authenticated-encryption algorithms through a threat-model review.
4. Define password KDF and bounded parameters, plus password verification and recovery UX.
5. Define exact setting-category allowlist and source-preference trust prompts.
6. Define hard parser limits from fixture corpus and reference hardware.
7. Define archive forward/backward compatibility policy and oldest-supported schema.

## 10. Explicit non-goals

- Do not embed downloaded chapter files in the ordinary backup.
- Do not reproduce Mihon's protobuf DTOs as the NamiShelf domain schema.
- Do not treat a valid checksum as proof of authenticity.
- Do not auto-enable imported extension repositories or execute imported extensions.
- Do not claim Mihon compatibility without real fixtures and cross-app tests.

No code, build, installation, fixture execution, or restore operation has been performed.