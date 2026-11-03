# NamiShelf — Self-Hosted Sync Adapter Options

**Status:** Comparative design; no second adapter selected
**Date:** 2026-11-03
**Decision basis:** ADR-006 — self-hosted sync required in release 1; GitHub is first adapter

## 1. Requirement

Release 1 must support synchronization without requiring a third-party cloud account or a NamiShelf-operated service. GitHub remains the first provider adapter, but GitHub.com alone does not satisfy self-hosting. A second adapter must connect to user-managed infrastructure while implementing the same provider-neutral NamiShelf protocol and mandatory E2EE envelope.

## 2. Candidate comparison

| Criterion | WebDAV | S3-compatible object storage | Self-hosted Git server |
|---|---|---|---|
| Self-hosting availability | Broad; common NAS and server software | Broad; multiple server implementations | Broad; established Git hosting software |
| Client complexity | HTTP methods, ETags, locking behavior varies | S3 API subset, signing, multipart and conditional semantics vary | Git transport/auth, object graph, refs and pack protocol |
| Atomicity/concurrency | Depends on server support for conditional writes and MOVE/LOCK | Conditional object operations vary by implementation | Ref updates can be atomic, but domain concurrency still needs protocol logic |
| Data model fit | Arbitrary resources and collections | Immutable/versioned objects fit well | Content-addressed immutable objects fit naturally |
| Operational setup | Usually simple URL/user/password or app password | More complex endpoint, bucket, access key and secret | Requires Git account/repository and SSH/HTTPS auth |
| Credential exposure | WebDAV credential only to adapter | Access/secret keys only to adapter | SSH key or token only to adapter |
| Large object handling | Server-dependent limits | Multipart upload commonly available | Packfile/object-size behavior varies |
| Provider neutrality | Good if capabilities are explicit | Good if limited to a defined API subset | More Git-shaped; risk of transport semantics leaking into sync protocol |
| User experience | Familiar to NAS/self-hosting users | Familiar to admins, less accessible to casual users | Familiar to developers, less accessible to general users |

## 3. WebDAV assessment

Advantages: broad deployment availability, HTTPS-based access, relatively low setup burden, and no need to operate Git or object-storage infrastructure.

Risks: WebDAV server implementations differ in ETag behavior, conditional PUT, MOVE atomicity, locking, directory listing, and consistency. The adapter must probe or declare capabilities and must not assume that LOCK is universally reliable. A server lacking safe conditional update semantics may require immutable object names plus a separate compare-and-swap coordination mechanism, or be rejected as unsupported.

Minimum acceptance: TLS validation; tested conditional create/update; stable resource identity; safe pagination/listing; bounded object sizes; resumable/interrupted transfer behavior; and no credential leakage.

## 4. S3-compatible assessment

Advantages: object storage is a natural fit for immutable encrypted envelopes, versioned objects, lifecycle controls, and large payloads. Self-hosted implementations exist.

Risks: “S3-compatible” is not a single uniform behavior contract. Conditional writes, versioning, consistency, multipart upload, deletion markers, and authentication differ. A small supported API subset and adapter capability declaration are mandatory. The UI setup is more complex than WebDAV.

Minimum acceptance: verified endpoint TLS; supported signing/authentication; conditional writes or immutable object strategy; version-aware reads; multipart/size limits; pagination; and explicit retention/deletion semantics.

## 5. Self-hosted Git assessment

Advantages: the candidate already uses Git object transport; self-hosted Git servers are widely available; content-addressed objects and ref updates can support immutable payload storage and a coordination pointer.

Risks: this increases coupling to Git semantics and may duplicate substantial code in the GitHub adapter. It is not necessarily easier for nontechnical users. Server-side hooks, repository permissions, branch protection, garbage collection, shallow clones, ref races, and packfile limits add complexity. The provider-neutral sync protocol must remain above Git.

Minimum acceptance: standard authenticated HTTPS or SSH; atomic ref update behavior; protected server-side repository access; safe handling of untrusted remote objects; no reliance on GitHub-specific APIs; and cross-provider conformance tests.

## 6. Recommendation

Evaluate WebDAV as the default self-hosted adapter candidate because it offers the broadest user-managed infrastructure story without making Git a required storage primitive. Keep S3-compatible storage as a strong alternative for users who already operate object storage. Consider self-hosted Git only if reuse of the candidate Git transport materially reduces implementation risk and the user experience is acceptable.

This is a recommendation, not an owner decision. Before selecting WebDAV, verify whether its conditional write and concurrency semantics can safely implement the protocol without server-specific workarounds. If not, S3-compatible storage may be the stronger first self-hosted adapter.

## 7. Required architecture shape

- One provider-neutral sync engine and encrypted envelope.
- Separate GitHub and self-hosted adapter implementations.
- Explicit adapter capability declaration and conformance suite.
- No provider-specific IDs or Git object hashes in canonical domain entities.
- User can configure a self-hosted endpoint without a NamiShelf account.
- Credentials stored only in the OS credential service.
- Provider stores ciphertext only; key management is independent of provider authentication.
- Switching provider must not require decrypting and reauthoring domain state through the old provider; define a secure client-side transfer path.

## 8. Open owner decision

Choose the release-1 self-hosted adapter: WebDAV, S3-compatible object storage, or self-hosted Git. If choosing WebDAV or S3, decide whether a minimal supported server compatibility list is acceptable. If choosing self-hosted Git, confirm that the more technical setup experience is acceptable.

No server was contacted, no credentials were used, and no implementation or interoperability test was performed.