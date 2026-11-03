# NamiShelf — Extension Broker Wire Protocol Proposal

**Status:** Design proposal; wire encoding and envelope are not owner-approved  
**Date:** 2026-11-03  
**Constraints:** One process per extension; stdio pipes; versioned framed IPC; host-mediated capabilities; source-origin network policy; editable capability profiles; broker-issued streaming handles

## 1. Design goals

The protocol must be unambiguous under partial reads/writes and process termination; versioned independently of NamiShelf releases; strictly bounded before parsing or allocation; suitable for requests, responses, host events, cancellation and health checks; easy to validate and fuzz; and incapable of granting authority based on extension-supplied claims.

The protocol is a local trust boundary, not a security sandbox. Every operation still requires host-side authorization.

## 2. Encoding options

| Option | Advantages | Costs / risks |
|---|---|---|
| UTF-8 JSON envelope | Human-readable, easy diagnostics and fixtures, broad language support, mature validators | Larger payloads and parsing overhead; duplicate-key, numeric and depth handling require strict rules |
| CBOR | Compact, typed binary values, efficient structured data | Less inspectable; canonicalization and implementation consistency need care |
| Protocol Buffers | Strong schema evolution and generated types | Code generation/runtime dependency; schema compatibility discipline required |
| MessagePack | Compact and broadly supported | Schema discipline remains external; extension types and parser limits require policy |

### Preliminary recommendation

Use a **4-byte unsigned big-endian length prefix followed by a UTF-8 JSON control envelope**. Keep control messages small and bounded. Transfer large binary data only through broker-issued streaming handles; never embed image/chapter blobs in JSON frames.

This is a proposal, not an approved encoding decision. Benchmarking and implementation are not authorized in this phase.

## 3. Framing

- Read exactly four bytes for the frame length, then exactly that many payload bytes.
- Reject zero-length frames and frames above a configured maximum before allocating the payload buffer.
- Define a smaller maximum for handshake/control messages.
- Treat EOF mid-prefix or mid-payload as a protocol failure.
- Do not attempt stream resynchronization after malformed framing; terminate the extension session.
- Serialize writes per channel to prevent frame interleaving.
- Standard output is protocol-only. Non-frame output is a protocol violation; diagnostics go to stderr and are bounded/redacted.
- JSON parsing must reject duplicate keys, invalid UTF-8, excessive nesting, oversized strings/arrays, non-finite numbers and unknown critical fields.

Exact byte limits remain to be selected from measured workloads and memory budgets.

## 4. Envelope proposal

Every control frame contains:
- protocol major/minor negotiated version;
- kind: hello, hello_ack, request, response, event, cancel, or error;
- unique correlation ID where applicable;
- allowlisted method name for requests/events;
- capability being exercised;
- method-specific typed payload;
- host-issued session nonce or opaque session identifier.

The extension does not choose its own identity, session, granted capabilities, or policy. The host binds these to the launched process and authenticated handshake.

Avoid bearer secrets in general envelope fields. Credential-bearing operations use narrowly scoped broker calls with source identity and redaction guarantees.

## 5. Handshake state machine

1. Host starts the process with a minimal environment and only required standard handles.
2. Host sends hello with supported protocol versions, host build/profile version, process session, and effective capability grants.
3. Extension responds with hello_ack, declaring runtime identity, compatibility profile, and required/optional API features.
4. Host validates the declaration against installed artifact metadata and granted policy.
5. Host sends activation result. Only then may the extension issue normal requests.
6. Requests before activation, duplicate handshakes, version downgrade, invalid sessions, or unsupported required features fail closed.

The extension's declared requirements never expand granted capabilities. The host computes the intersection of product policy, user profile, source policy and extension requirements.

## 6. Request lifecycle

- Every request has a unique ID scoped to the process session.
- Host enforces per-extension in-flight limits and per-method deadlines.
- Cancellation references the original request ID and is idempotent.
- Responses for unknown, cancelled, expired or completed IDs are rejected or discarded according to a documented rule.
- Retries are explicit and method-specific; non-idempotent operations are not retried automatically.
- Process exit invalidates all outstanding requests and all handles.
- Restart creates a new session; old request IDs and handles cannot be reused.

## 7. Error model

Use stable machine-readable error codes plus safe user-facing messages. Do not return stack traces, filesystem paths, credentials, cookies, or raw upstream response bodies by default.

Candidate classes: PROTOCOL_INVALID, PROTOCOL_VERSION_UNSUPPORTED, SESSION_NOT_ACTIVE, CAPABILITY_DENIED, ORIGIN_NOT_ALLOWED, RATE_LIMITED, QUOTA_EXCEEDED, REQUEST_TIMEOUT, REQUEST_CANCELLED, RESOURCE_EXPIRED, UPSTREAM_NETWORK_ERROR, UPSTREAM_HTTP_ERROR, UNSUPPORTED_API, EXTENSION_FAILURE.

Errors should include retryability and a correlation ID, but not sensitive request metadata.

## 8. Streaming-handle contract

The broker issues opaque handles only after authorizing the originating operation. A handle is:
- bound to extension identity, process session and originating request;
- limited to one declared content type and maximum byte count;
- short-lived and non-transferable;
- readable only through explicit stream operations;
- invalidated on cancellation, expiry, revocation, process exit or policy change.

The protocol should support bounded chunk reads, explicit end-of-stream, cancellation and backpressure. Resume is not assumed; if needed later, it requires explicit range/token design that cannot cross sessions. Handle identifiers are not filesystem paths and reveal no storage location.

## 9. Compatibility and evolution

- Major version changes may break wire compatibility; minor versions add backward-compatible optional features.
- Unknown optional fields may be ignored only where the schema explicitly marks them optional.
- Unknown methods, capabilities, critical fields and enum values fail closed.
- Capability identifiers are stable and independently versioned.
- Native JVM and Android compatibility shims use the same method schemas.
- The host may disable individual API methods without changing transport version, but must expose the effective profile to the extension.

## 10. Abuse resistance

- Strict frame and parser limits before allocation.
- Per-extension request, byte, handle, concurrency and time quotas.
- Backpressure; no unbounded queues.
- No arbitrary method dispatch or reflection-based host invocation.
- No generic filesystem, database, shell, sync, backup or tracking endpoints.
- Structured audit events with secret redaction.
- Fuzz framing, parser, state machine, cancellation and handle lifecycle before release.

## 11. Decisions still required

1. Choose JSON, CBOR, Protocol Buffers or another encoding.
2. Confirm frame maximums and parser limits.
3. Confirm protocol major/minor negotiation and backward-compatibility policy.
4. Confirm exact handshake identity and extension attestation fields.
5. Confirm error taxonomy and which errors are user-visible.
6. Confirm stream chunking, expiry, quota and resume semantics.
7. Confirm whether a source may declare multiple origins and how catalog provenance is validated.

No implementation, benchmark, extension execution, or build has been performed.
