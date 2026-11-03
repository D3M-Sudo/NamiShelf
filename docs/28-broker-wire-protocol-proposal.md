# NamiShelf — Extension Broker Wire Protocol Proposal

**Status:** Design proposal; encoding and envelope are not owner-approved.

## Constraints

This proposal follows the approved architecture: one process per extension, stdio transport, versioned framed IPC, broker-mediated access, source-declared network origins, editable capability profiles, and temporary streaming handles.

## Encoding comparison

| Encoding | Benefits | Costs |
|---|---|---|
| UTF-8 JSON | Inspectable, easy fixtures, broad support | Larger and slower; strict parser limits required |
| CBOR | Compact typed data | Less inspectable; canonicalization discipline |
| Protocol Buffers | Generated schemas and evolution support | Runtime/code-generation dependency |
| MessagePack | Compact and widely supported | Schema discipline remains external |

**Recommendation:** 4-byte unsigned big-endian length prefix plus UTF-8 JSON control envelope. Keep control frames small; large binary payloads must use broker-issued handles. This is a proposal only.

## Framing and parser rules

- Read exactly four bytes, then exactly the declared payload length.
- Reject zero-length and oversized frames before allocation.
- Treat EOF mid-frame as a protocol violation; terminate the extension session rather than attempting resynchronization.
- Serialize writes to prevent interleaving.
- Reserve stdout exclusively for protocol frames; stderr is bounded, redacted diagnostics.
- Reject invalid UTF-8, duplicate JSON keys, excessive nesting, oversized collections, non-finite numbers, and unknown critical fields.
- Set distinct limits for handshake/control frames and payload metadata. Exact limits require workload and memory-budget analysis.

## Envelope proposal

Fields: negotiated protocol major/minor; message kind (hello, hello_ack, request, response, event, cancel, error); correlation ID; allowlisted method; capability identifier; typed method payload; host-issued process-session identifier.

The host binds extension identity, session, and granted capabilities to the launched process. Extension-supplied metadata cannot grant authority. Avoid bearer secrets in generic envelope fields.

## Handshake

1. Host launches with minimal environment and required standard handles only.
2. Host sends supported protocol versions, host/profile version, session ID, and effective capability grants.
3. Extension declares runtime identity, compatibility profile, and required/optional APIs.
4. Host validates declarations against artifact metadata and user-approved policy.
5. Host explicitly activates the session.
6. Requests before activation, invalid sessions, duplicate handshakes, unsupported required features, and downgrade attempts fail closed.

## Request lifecycle

- Unique IDs scoped to a process session.
- Per-extension in-flight limits and per-method deadlines.
- Idempotent cancellation.
- Explicit method-specific retry rules; no automatic retries for non-idempotent operations.
- Process exit invalidates requests and all handles; restart creates a new session.
- Responses to unknown, expired, or cancelled IDs are discarded or treated as protocol violations under a documented rule.

## Error taxonomy proposal

PROTOCOL_INVALID; PROTOCOL_VERSION_UNSUPPORTED; SESSION_NOT_ACTIVE; CAPABILITY_DENIED; ORIGIN_NOT_ALLOWED; RATE_LIMITED; QUOTA_EXCEEDED; REQUEST_TIMEOUT; REQUEST_CANCELLED; RESOURCE_EXPIRED; UPSTREAM_NETWORK_ERROR; UPSTREAM_HTTP_ERROR; UNSUPPORTED_API; EXTENSION_FAILURE.

Errors should include a correlation ID and retryability but exclude stack traces, credentials, cookies, filesystem paths, and sensitive upstream bodies.

## Streaming handle contract

Handles are opaque, short-lived, and bound to extension identity, process session, originating request, content type, and byte quota. They are readable only through explicit broker operations, support bounded chunks, cancellation and backpressure, and are invalidated on expiry, cancellation, process exit, revocation, or policy change. They are not paths and reveal no storage location. Resume semantics are not assumed.

## Abuse resistance and evolution

- Strict frame/parser limits before allocation.
- Per-extension request, byte, handle, concurrency and time quotas.
- No unbounded queues, arbitrary method dispatch, reflection-based host invocation, or generic filesystem/database/shell/sync/backup endpoints.
- Major versions may break compatibility; minor versions add optional backward-compatible features.
- Unknown methods, capabilities, critical fields and enum values fail closed.
- Native JVM and converted Android extensions use identical method schemas.
- Fuzz framing, parser, state machine, cancellation and handle lifecycle before release.

## Decisions still required

1. Select JSON, CBOR, Protocol Buffers, or another encoding.
2. Set frame and parser limits.
3. Confirm version negotiation and backward-compatibility rules.
4. Confirm handshake identity/attestation fields.
5. Confirm error taxonomy and user-visible diagnostics.
6. Set chunk size, expiry, quota and resume behavior for handles.
7. Define source-origin declaration format and catalog provenance validation.

No implementation, benchmark, build, or extension execution has been performed.


## Superseded status

ADR-011 and document 29 select UTF-8 JSON control frames plus a binary stream frame over the approved stdio transport. This proposal remains useful as the encoding comparison and design rationale, but document 29 is the normative v1 implementation target. Where the documents differ, ADR-011 and document 29 take precedence.
