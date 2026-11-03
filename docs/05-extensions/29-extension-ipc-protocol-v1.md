# NamiShelf — Extension IPC Protocol v1

**Status:** Engineering proposal for the later extension workstream; not an owner-frozen wire contract. ADR-011 is intentionally unassigned.  
**Transport:** stdio, one child process per extension.  
**Security boundary:** host-enforced broker; IPC is not an OS sandbox.

## 1. Scope and invariants

This proposal describes a candidate first implementation target for native JVM and converted Android/Mihon extensions. Its framing, limits, handshake fields, method registry and compatibility details require engineering validation and must not be treated as owner-approved solely by appearing here. Both runtimes use identical wire messages and broker authorization.

- The host launches, identifies, supervises, and terminates each extension process.
- stdin/stdout are dedicated to IPC; stdout MUST NOT contain logs.
- stderr is diagnostics only, bounded and redacted.
- No request is authorized by extension-supplied identity, capability claims, source metadata, or payload fields alone.
- Every request is checked against process identity, active session, source identity, user-approved profile, method schema, quotas, and current policy.
- No generic filesystem, database, shell, backup, sync, tracker, credential, or arbitrary host invocation method exists.
- A malformed frame or protocol-state violation terminates the extension session.

## 2. Wire encoding and frame layout

Use UTF-8 JSON for control frames and a binary frame type for streamed payload chunks. This avoids base64 expansion while keeping control messages inspectable.

Each frame is a four-byte unsigned big-endian body length N, followed by a one-byte frame type and N-1 bytes of body. The length includes the frame-type byte and excludes the four-byte prefix.

Frame types:
- 0x01 CONTROL_JSON: body is one UTF-8 JSON object.
- 0x02 STREAM_DATA: body is a fixed binary stream header followed by payload bytes.

Reject N=0. Read exactly N bytes. EOF inside prefix/body is fatal. Never scan for a new prefix after malformed data.

### 2.1 Initial limits

These are conservative implementation defaults, not performance claims. Keep them centrally configurable and verify against fixture workloads before release.

| Limit | Initial value |
|---|---:|
| Maximum frame body | 1 MiB |
| Maximum handshake/control JSON frame | 64 KiB |
| Maximum JSON nesting depth | 32 |
| Maximum string value | 256 KiB |
| Maximum array/object members | 4,096 / 1,024 |
| Maximum concurrent requests per extension | 16 |
| Maximum open streaming handles per extension | 4 |
| Maximum stream data payload per frame | 256 KiB |
| Maximum outstanding stream bytes per extension | 8 MiB |
| Handshake deadline | 10 seconds |
| Default request deadline | 30 seconds |
| Source network operation deadline | 45 seconds |

The host rejects a frame before allocation if its declared size exceeds the limit. Limits apply independently in both directions.

### 2.2 JSON parser requirements

Use RFC 8259 JSON, UTF-8 only. Reject duplicate object keys, invalid UTF-8, unpaired surrogate input, non-finite numbers, excessive nesting, oversized strings/collections, trailing data, and unknown critical fields. Do not enable polymorphic/default class instantiation from untrusted type names. Parse into explicit DTOs and validate every field.

## 3. Control envelope

Every control message is a JSON object with:
- v: negotiated protocol version, object with major 1 and minor 0;
- kind: hello, hello_ack, activate, request, response, event, cancel, or error;
- id: 128-bit random correlation identifier encoded as lowercase hex, required for request/response/cancel;
- method: allowlisted operation name for request/event;
- capability: capability identifier required by the method;
- session: host-issued opaque session identifier;
- payload: method-specific schema object.

Do not include bearer tokens, passwords, recovery keys, E2EE keys, cookies, or arbitrary filesystem paths in generic envelopes.

## 4. Handshake and activation state machine

States: STARTING → HELLO_SENT → NEGOTIATED → ACTIVE → STOPPING → EXITED.

1. Host launches the process with a minimal environment, a working directory inside the extension private data area, and only stdin/stdout/stderr inherited. No provider credentials or application secrets in environment variables or command-line arguments.
2. Host sends hello containing supported protocol versions, host build, broker API version, compatibility profile ID, host-issued session ID, extension artifact ID, source ID, and effective capability IDs.
3. Extension replies hello_ack with runtime type, supported versions, declared API compatibility profile, and required/optional feature IDs.
4. Host validates artifact metadata, selected runtime, compatibility profile, and that every required feature is supported and permitted.
5. Host sends activate with success or a stable failure code. Only then may normal requests be sent.
6. A request before activation, duplicate handshake, unknown session, unsupported required feature, malformed version, or attempted downgrade is fatal.

The host computes effective permissions as the intersection of product policy, approved profile, user overrides, source policy, and the method declared capability. Extension declarations never expand grants.

## 5. Request/response semantics

- IDs are unique within a session and MUST NOT be reused.
- Host and extension each maintain bounded in-flight maps.
- Every method has a schema, required capability, timeout, size limits, idempotency classification, and cancellation behavior.
- Cancellation is idempotent and references the original request ID.
- A completed, cancelled, or expired request cannot be revived.
- Automatic retries are forbidden for non-idempotent operations. Retry policy for idempotent operations belongs to the host.
- Responses for unknown IDs are protocol violations; late responses for explicitly cancelled requests may be discarded once and counted.
- Process exit fails all outstanding requests and invalidates every handle.

## 6. Error object

Errors contain code, safe message, retryable, and correlation_id. Optional details must be schema-defined and non-sensitive.

Stable initial codes: PROTOCOL_INVALID, PROTOCOL_VERSION_UNSUPPORTED, SESSION_NOT_ACTIVE, CAPABILITY_DENIED, ORIGIN_NOT_ALLOWED, RATE_LIMITED, QUOTA_EXCEEDED, REQUEST_TIMEOUT, REQUEST_CANCELLED, RESOURCE_EXPIRED, UPSTREAM_NETWORK_ERROR, UPSTREAM_HTTP_ERROR, UNSUPPORTED_API, SOURCE_UNAVAILABLE, EXTENSION_FAILURE.

Never return stack traces, secrets, cookies, host paths, raw authorization headers, or unbounded upstream response bodies. Internal diagnostics use a correlation ID and redacted structured logging.

## 7. Binary stream frame

A STREAM_DATA body has:
- 16-byte request ID;
- 16-byte handle ID;
- 4-byte unsigned big-endian sequence number;
- 1-byte flags (0x01 = final chunk; all other bits reserved and rejected);
- 0..262,144 bytes of content.

The receiver checks request, handle, sequence, direction, content type, and cumulative byte quota. Sequence starts at zero and increments by one. A final frame closes the stream. No content is accepted after finalization.

Handles are opaque random identifiers, not paths or bearer credentials. They are bound to extension ID, process session, originating request, direction, declared media type, and byte quota. The broker authorizes each read/write operation. Handles expire after 60 seconds of inactivity and at most 10 minutes absolute lifetime by default; they are invalidated immediately on cancellation, revocation, process exit, or policy change. These defaults are configurable and require workload validation.

## 8. Method registry and API compatibility

No dynamic method dispatch. Maintain a versioned registry mapping each method to request/response schema, required capability, maximum payload, timeout, idempotency, cancellation semantics, API compatibility profile, and audit classification.

Unknown methods and capabilities fail closed. Native JVM and DEX-to-JVM adapters use the same registry. Android compatibility shims may translate legacy calls but cannot bypass broker checks.

## 9. Logging and observability

stdout is protocol-only. stderr capture is bounded; redact tokens, cookies, passwords, URLs with sensitive query parameters, and user content. Record lifecycle, version negotiation, capability denials, quota violations, crashes, restarts, and policy changes. Do not log page bodies, full request headers, credentials, stream payloads, or encryption material. Apply log retention and rotation limits.

## 10. Version evolution

Protocol v1.0 is the initial target. Major versions may break compatibility. Minor versions may add optional fields/methods only when older peers can safely ignore them. Unknown critical fields, methods, capabilities, enum values, or required features fail closed. No silent downgrade. Keep protocol version independent from NamiShelf app version and Android API profile version.

## 11. Conformance and release gates

Build deterministic fixtures for partial reads/writes, frame truncation, oversize declarations, duplicate JSON keys, invalid UTF-8, depth/collection abuse, ID reuse, handshake downgrade, out-of-order responses, cancellation races, late responses, stream sequence gaps, wrong-handle access, quota overflow, expiry, process exit, and stderr floods.

Before release, fuzz the parser/state machine and verify the host remains responsive and other extensions remain isolated after one child crashes or misbehaves. Test process launch and stream behavior under AppImage, Flatpak, and DEB.

## 12. Explicit non-goals

This protocol does not itself provide an OS sandbox, network firewall, cryptographic device attestation, or protection from a compromised host process. It is a constrained local IPC contract backed by process isolation and host-side authorization.


## 13. Decision authority and freeze gate

ADR-011 is intentionally unassigned; this document is not authorized by that ADR number. Owner-approved constraints are the one-process-per-extension model, stdio transport, broker-mediated access, source-origin restrictions, capability policy and broker-issued streaming handles (ADRs 008–010 and ADR-025). The exact JSON/binary framing, initial numeric limits, envelope fields, error taxonomy and method registry remain an engineering proposal to be validated in the later extension workstream before implementation freeze. This work is not part of M0.
