# NamiShelf — Native Backup Container Options Assessment

**Status:** Owner-approved native format choices; implementation details remain open
**Date:** 2026-11-03

## 1. Requirements derived from ADR-003

The native backup must be independent of Mihon and of the live application database. It must be versioned, streamable, verifiable before restore, forward-migratable, and safe to inspect without mutating the profile. It stores library metadata, chapter state, categories, history, tracker associations, and user-selected portable settings. Downloaded chapter files are a separate export/migration operation.

## 2. Candidate comparison

| Criterion | ZIP64 container | TAR + Zstandard | Custom framed envelope |
|---|---|---|---|
| Tooling and inspection | Excellent; ubiquitous tools | Good; common Unix tooling | Requires NamiShelf tooling |
| Streaming write | Good with sequential entries | Excellent | Excellent if designed correctly |
| Random access | Good with central directory | Limited without index | Design-dependent |
| Large archive support | ZIP64 required | Strong | Design-dependent |
| Metadata and path safety | Must reject traversal, duplicate names, symlinks | Must reject traversal, links, special files | Explicit records, fewer path semantics |
| Corruption recovery | Central directory can be lost on truncation | Stream may be partially recoverable | Depends on framing/index |
| Encryption integration | Use an outer authenticated-encryption envelope; avoid legacy ZIP encryption | Same outer envelope approach | Can integrate framing, but crypto design risk remains |
| Ecosystem compatibility | Highest | High on Linux | Lowest |
| Implementation complexity | Moderate | Moderate | High

## 3. Recommendation

Use a ZIP-based container for the first native backup generation. Use ZIP64 extensions where required for large archives. It provides broad tooling, straightforward file inspection, a standard manifest-plus-payload layout, and adequate support for the relatively compact library/settings backup. ZIP64 avoids classic ZIP size limits.

Suggested logical layout (names are illustrative, not yet a frozen contract):

- `manifest.json` — format identifier, schema version, producer version, creation metadata, payload inventory, lengths, and cryptographic digests.
- `data/` — structured canonical backup payloads grouped by domain, not by live database table.
- Optional future attachments — only if a later owner decision authorizes them; downloaded chapter files remain a separate export.

Use a stable canonical serialization for the structured payload. JSON is easier to inspect and migrate; a binary schema such as Protocol Buffers can be more compact but requires stricter schema governance and tooling. The owner selected canonical JSON for structured payloads. Define a canonicalization profile for deterministic serialization and hashing; do not reuse Mihon's protobuf DTOs as the NamiShelf domain.

## 4. Mandatory ZIP safety rules

- Accept only regular files with normalized relative paths; reject absolute paths, `..`, duplicate normalized paths, symlinks, and special file entries.
- Enforce limits on archive size, compressed and decompressed bytes, entry count, individual entry size, path length, and compression ratio.
- Never trust manifest sizes or hashes until actual stream lengths and digests have been checked.
- Reject duplicate manifest keys and ambiguous encodings.
- Parse and validate all entries before applying restore mutations.
- Use temporary output and atomic replacement where supported; preserve the previous valid backup on interruption.
- Do not extract archives directly to arbitrary filesystem paths.
- Keep credentials, tokens, cookies, keys, and local absolute paths out of the payload.

## 5. Integrity and encryption

Integrity is mandatory. The manifest should include per-payload length and digest, plus a final integrity structure covering the manifest and payload inventory. An unkeyed digest detects accidental corruption but does not authenticate the creator.

Encryption is optional per backup, as owner-approved. When enabled, use a reviewed authenticated-encryption envelope rather than ZIP's historical encryption modes. Specify KDF parameters, nonce handling, authentication failure behavior, metadata leakage, and recovery policy in a separate security ADR. Do not reuse OS keyring or sync keys.

## 6. Schema evolution

- Use an explicit format identifier and major/minor schema version.
- Unknown optional fields should be preservable or safely ignorable; unknown required features must block restore with a clear message.
- Migrations operate on canonical backup domain objects, not live database migrations.
- Keep fixtures for every released NamiShelf schema and test oldest-supported-to-current restore.
- Record source application version and backup schema version separately.

## 7. Owner-approved choices and remaining design work

Approved on 2026-11-03: ZIP container (ZIP64 when needed); canonical JSON payloads; optional encryption per backup; complete library selected by default, with settings selectable by category.

Still to specify: canonical JSON profile, ZIP feature subset, manifest schema, integrity semantics, authenticated-encryption profile, KDF parameters, metadata leakage, password recovery UX, and exact settings allowlist.

No implementation, build, backup creation, restore, or fixture execution has been performed.