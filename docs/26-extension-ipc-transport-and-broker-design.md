# NamiShelf — Extension IPC Transport and Broker Design

**Status:** Stdio transport owner-approved; wire schema and broker policy remain design work
**Date:** 2026-11-03
**Decision basis:** Owner-approved one-process-per-extension topology and brokered data access

## 1. Fixed security constraints

- One OS process per third-party extension.
- Extension code never executes in the NamiShelf main process.
- NamiShelf remains authoritative for data access, policy, credentials, lifecycle, and persistence.
- All extension requests pass through a narrow, versioned, validated broker API.
- No direct access to application databases, backups, sync keys/tokens, tracking credentials, or arbitrary host paths.
- Transport choice must work with Linux Mint 22.3 and the planned AppImage, Flatpak, and DEB packaging models.
- OS sandboxing remains conditional on a sustainable cross-package implementation; IPC must not assume a sandbox exists.

## 2. Transport options

| Option | Strengths | Risks / limitations |
|---|---|---|
| **Stdio pipes** | Parent directly owns child lifecycle; no listening socket or endpoint discovery; easy to supervise; works across packaging formats | Must define framing and backpressure; stdout must be protocol-only; stderr/log separation; process restart reconnect semantics; binary streams can be awkward for large payloads |
| **Unix domain sockets** | Full-duplex bidirectional channel; filesystem permissions and peer credentials can help authenticate local endpoints; supports multiple channels | Socket path lifecycle and permissions; stale socket cleanup; namespace/Flatpak visibility; endpoint discovery and cleanup; Linux-specific unless abstracted |
| **D-Bus** | Native desktop IPC, service activation and typed interfaces; familiar on Linux | Session/system bus policy and Flatpak portal constraints; service naming/activation complexity; API/versioning ergonomics; not a portable transport for non-Linux future targets |
| **TCP loopback** | Portable and easy to integrate with common RPC stacks | Port allocation/races, local network exposure, firewall/security assumptions, peer authentication complexity; avoid unless a strong cross-platform requirement emerges |

## 3. Recommendation

Use **stdio pipes with a length-prefixed, versioned message envelope** for the first Linux release (owner-approved 2026-11-03; ADR-008). NamiShelf launches and supervises each extension process directly, so inherited stdin/stdout pipes provide a narrow private channel without exposing a local listening endpoint. Keep stdout exclusively for protocol frames and stderr for bounded diagnostic output.

Use a transport abstraction so a later platform can add Unix domain sockets or another channel without changing the broker contract. Do not use D-Bus as the extension protocol: it can be considered for desktop integration, but should not be required for extension runtime communication.

This recommendation depends on proving that the selected extension runtime and process supervisor can reliably support bidirectional framed stdio, cancellation, bounded streaming, and crash detection. If large binary chapter/image transfers are required, prefer brokered streaming handles or a separate bounded data channel rather than embedding unbounded blobs in ordinary RPC messages.

## 4. Message envelope requirements

Every frame should carry:

- protocol major/minor version;
- message type (request, response, event, cancellation, handshake);
- request/correlation identifier;
- extension identity and host session identifier;
- capability identifier and operation name;
- bounded payload length;
- typed payload validated against the negotiated schema.

Use a fixed maximum frame size for control messages. Large content must use explicit streaming operations with byte quotas, cancellation, timeouts, and per-extension concurrency limits. Reject unknown major versions, malformed frames, duplicate request IDs, invalid state transitions, and unauthorized capability use.

## 5. Handshake and lifecycle

1. NamiShelf verifies artifact provenance and extension manifest before launch.
2. Host starts with a minimal environment, dedicated working directory, and only required inherited handles.
3. NamiShelf sends a protocol handshake containing supported major/minor versions and granted capabilities.
4. Extension responds with its runtime/API compatibility declaration and supported operations.
5. Broker validates the declaration and activates only the intersection of host policy and extension requirements.
6. All subsequent requests are authenticated to the specific process session and validated against the negotiated schema.
7. On crash, timeout, protocol violation, or revocation, NamiShelf cancels outstanding requests, terminates the process, invalidates session handles, and applies bounded restart/disable policy.

An extension cannot self-grant capabilities by declaring them in its manifest or handshake.

## 6. Capability model

Capabilities should be operation-specific and scoped to an extension/source identity. Candidate groups:

- source discovery and metadata;
- search and detail retrieval;
- chapter listing and page retrieval;
- brokered HTTP requests with source-scoped cookie/auth state;
- extension-owned settings and cache;
- controlled image/content streaming;
- notifications/events to the host.

Do not expose generic database queries, arbitrary file read/write, shell/process execution, unrestricted network sockets, sync or tracking credentials, backup access, or raw application configuration.

Each request must be checked against the granted capability, user consent state, source scope, quota, and current extension lifecycle state.

## 7. Network broker boundary

Do not assume that an IPC transport decision settles network policy. Define separately whether extensions can:

- request arbitrary HTTPS URLs or only source-declared origins;
- follow redirects across origins;
- use source-scoped cookies or authentication;
- set sensitive headers;
- use proxies or custom DNS;
- download large responses or stream content;
- access localhost, private LAN ranges, or link-local endpoints.

Default-deny local/private network destinations unless a specific source capability and user-approved policy require them. Always validate TLS. Prevent credential forwarding across origin changes and redact sensitive URLs/headers from logs.

## 8. Resource and abuse controls

- Per-extension process and request quotas.
- Bounded message/frame size and parser depth.
- CPU, memory, file descriptor, and wall-clock limits where OS/package support permits.
- Request deadlines, cancellation, and bounded retries.
- Backpressure and streaming byte quotas.
- Crash-loop detection and automatic disablement.
- No inherited secrets, ambient credentials, or unnecessary file descriptors.

Document which controls are enforced by NamiShelf itself and which depend on optional OS sandboxing. Do not claim OS-level containment for application-level quotas.

## 9. Compatibility runtime interaction

DEX-to-JVM conversion and Android/Mihon API adapters execute inside the extension's dedicated process. The compatibility layer must call the same broker contract as native JVM extensions; it must not create a privileged alternate path to host files, databases, credentials, or unrestricted network access.

Android APIs that imply direct filesystem, ContentResolver, Android AccountManager, WebView, or arbitrary service access require explicit emulation, broker mapping, or deterministic unsupported behavior. Compatibility must be defined by a versioned API matrix and tested against real extension fixtures.

## 10. Acceptance gates before implementation

- Demonstrate process supervision and stdio framing under AppImage, Flatpak, and DEB constraints.
- Verify protocol isolation: extension stdout cannot inject logs or bypass frame validation.
- Test malformed, truncated, oversized, and reordered frames.
- Test cancellation, backpressure, process death, restart, and stale-session invalidation.
- Prove that no secrets or privileged descriptors are inherited.
- Test capability denial and cross-extension isolation.
- Test brokered network origin, redirect, cookie, and private-network policies.
- Exercise real converted DEX/JVM fixtures through the same broker interface.

## 11. Owner decision and status

**Resolved:** stdio pipes with a length-prefixed, versioned frame protocol are the release-1 IPC transport (ADR-008). Unix domain sockets and D-Bus are not selected as extension runtime transports.

No transport has been implemented or benchmarked. No extension code has been executed.

## 12. Owner-approved broker policy (ADR-010)

### 12.1 Network: source-declared origins

The owner selected an origin-restricted model. Extensions do not receive general-purpose network access. The broker validates each request against the installed source's declared origin set and the effective user-approved capability profile.

- Require HTTPS by default; any future exception needs a separate explicit security decision.
- Re-evaluate every redirect hop against the origin policy.
- Strip Authorization, Cookie, and other sensitive headers on origin changes; never forward credentials across origins by default.
- Deny localhost, loopback, private, link-local, multicast, and cloud metadata destinations by default.
- Validate DNS results and resolved destinations to mitigate DNS rebinding and redirect-based SSRF.
- Keep cookie jars and authentication state scoped to a source identity.
- Do not trust extension-supplied origin declarations without host/catalog validation and user-visible disclosure.

### 12.2 Capability profiles

The owner selected editable predefined permission profiles. NamiShelf owns the profile definitions and computes effective grants; extension manifests only request permissions.

- Display requested and effective capabilities before install/enable.
- Allow users to inspect and override individual capabilities.
- Require explicit additional consent for sensitive capabilities.
- Never silently grant newly requested capabilities during extension or profile upgrades.
- Revoke active requests and associated handles when a grant is withdrawn.

Exact profile names, default grants, sensitive-capability taxonomy, and UX remain open.

### 12.3 Brokered streaming handles

Large content uses opaque, temporary broker-issued handles rather than large unbounded control frames.

Each handle must be bound to the extension process/session and originating request, expire, enforce byte and concurrency quotas, support cancellation/backpressure, and become invalid on process exit, revocation, cancellation, or expiry. Extensions cannot choose arbitrary host paths or reuse handles across extension identities.

Exact token format, chunk size, resume semantics, quotas, and storage strategy remain open.
