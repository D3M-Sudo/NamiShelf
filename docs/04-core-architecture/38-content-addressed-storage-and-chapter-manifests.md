# Content-Addressed Storage and Chapter Manifests

**Status:** Content-addressed filesystem storage with stable logical identifiers and per-chapter manifests approved under ADR-019. Exact layout, hash algorithm and serialization remain deferred to M0 design review.

## 1. Responsibilities

- **SQLite:** authoritative structured metadata, logical IDs, relationships, source mappings, download state, manifest references and lookup indexes.
- **Permanent artifact store:** downloaded page payloads addressed by content hash.
- **Chapter manifest:** versioned ordered mapping from a stable chapter ID to page identities and content hashes.
- **Download staging:** temporary partial work, never exposed as a completed chapter.
- **Decoded-page cache:** bounded in-memory LRU owned by the reader.
- **Optional disk cache:** disposable and separate from permanent downloads.

## 2. Identity model

Manga and chapter identities must not depend on mutable titles, URLs, display labels or source metadata. Source-specific identifiers may be retained as mappings, but NamiShelf must maintain stable internal logical identifiers. Page identity/order is represented by the chapter manifest; content identity is represented by a cryptographic digest of the stored payload.

The digest algorithm, canonical byte representation and collision-handling policy must be selected during M0 after inspecting candidate code and libraries.

## 3. Chapter manifest

A manifest must describe at least:
- manifest schema/version;
- stable internal manga and chapter IDs;
- ordered page entries;
- page content hash and byte length where known;
- media type and dimensions where reliably available;
- source provenance/reference metadata where appropriate and safe;
- integrity/completion metadata.

Do not assume JSON: serialization is an explicit implementation decision. The native user backup's JSON encoding is a separate format decision and does not automatically dictate internal manifest serialization.

Unknown future manifest versions must be rejected safely or opened read-only where possible; never silently rewrite them.

## 4. Download finalization

1. Create a unique staging operation.
2. Stream each page to a temporary file while enforcing size/resource limits.
3. Compute the content digest and validate the payload.
4. Move or link the payload into its content-addressed permanent location using a crash-safe atomic operation where supported.
5. Write and validate the versioned chapter manifest.
6. Commit the SQLite metadata/state transition to complete only after required artifacts and manifest are durable.
7. Clean up staging data after successful finalization.

SQLite and filesystem operations are not one atomic transaction. Recovery logic must reconcile orphaned files, incomplete manifests and stale database states after interruption or crash.

## 5. Deduplication and garbage collection

- Identical payload hashes may share one stored artifact.
- Maintain reference accounting or derive reachability from authoritative chapter manifests and retained operations.
- Never delete an artifact while referenced by any completed chapter, active download, backup/export operation or other retained object.
- Garbage collection should use a conservative mark-and-sweep or equivalent reference-aware process, with safe handling of concurrent writes and crashes.
- Orphan cleanup must distinguish abandoned staging files from committed artifacts.

## 6. Integrity and repair

- Verify hashes when finalizing downloads.
- Support explicit integrity checks and detection of missing, truncated or mismatched artifacts.
- Represent damaged chapters as repairable states; permit page/chapter redownload without corrupting unrelated chapters.
- Do not report a chapter as complete when its manifest or required page payloads are missing.
- Keep diagnostics useful but do not log credentials, cookies, signed URLs or page contents.

## 7. Access and security boundaries

All storage access is mediated by NamiShelf application services. Extensions receive only brokered operations and scoped handles permitted by their capability profile; they do not receive arbitrary filesystem paths or direct access to the SQLite database, backup files or global credentials.

## 8. Retention classes

Permanent downloads, download staging, optional disposable disk cache and decoded in-memory LRU cache have distinct quotas and cleanup rules. Reader cache eviction must never remove permanent downloaded artifacts. User deletion and garbage collection must respect active references and explicit user intent.

## 9. Deferred design decisions

M0 must inspect the downstream candidate and determine:
- concrete directory layout and path portability;
- digest algorithm and collision strategy;
- manifest serialization and schema evolution;
- stable-ID generation/mapping strategy;
- atomic move/link behavior across supported filesystems;
- filesystem permission and symlink policy;
- encryption-at-rest requirements and backup interaction;
- reference accounting and garbage-collection implementation;
- migration from any existing candidate storage layout.

## 10. Validation scenarios

Before release, validate interrupted downloads, process crash during finalization, duplicate page payloads, renamed titles/source URLs, missing/corrupt files, manifest-version mismatch, concurrent download and reader access, backup/export during writes, garbage collection under active references, and low disk-space behavior.

No implementation, build, dependency installation or test execution is authorized by this specification.
