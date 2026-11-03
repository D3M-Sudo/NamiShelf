# NamiShelf — Extension Security and Compatibility Model

**Status:** Owner-approved architecture direction; implementation details remain open  
**Decision date:** 2026-11-03

## 1. Owner decisions

- Third-party extension code must never execute inside the NamiShelf main process.
- Use **one separate process per extension**.
- Extension access to application data is mediated by NamiShelf through a narrow, versioned IPC/broker interface.
- Android APK extensions converted from DEX to JVM-compatible artifacts are required in the first public release.
- Third-party extensions are included in the first public release.
- Pursue OS-level isolation where technically sustainable. If it proves disproportionately complex or unreliable, defer the exact mechanism and its release commitment to a later explicit owner decision.

## 2. Security objective

The Extension Host boundary must reduce the consequences of a compromised, malicious, buggy, or resource-exhausting extension.

NamiShelf remains the authority for library/history persistence, tracking credentials, backups, downloads, user preferences, extension lifecycle, and sensitive capability approval. Extensions must not receive unrestricted access merely because they are installed or signed.

## 3. Approved target architecture

```text
+-------------------------------------------------------+
|                      NamiShelf                        |
| UI / Domain / Library / Reader / Backup / Downloads   |
|                                                       |
| Extension Manager + policy enforcement + data broker |
+--------------------------+----------------------------+
                           | versioned, validated IPC
                 +---------+----------+
                 |                    |
          +------v------+      +------v------+
          | Extension A |      | Extension B |
          | own process |      | own process |
          +-------------+      +-------------+
```

One OS process per extension is the approved target topology. It improves fault containment and limits the blast radius compared with shared workers. The implementation must assess startup latency, memory overhead, process supervision, and behavior with many installed extensions. If overhead is excessive on target hardware, present measured alternatives for a new owner decision; do not silently switch to shared workers.

Extension data access is mediated by NamiShelf. Extensions request narrowly scoped operations through the broker/IPC API; they do not receive direct access to application databases, backup archives, global credentials, or arbitrary user paths.

## 4. Mandatory security properties

### 4.1 Process separation

- Extension code must not execute inside the NamiShelf application process.
- The application must terminate and restart an extension host independently.
- A crash or hang in one extension must not crash or indefinitely block the main UI.
- The host must not inherit unnecessary application secrets or privileged file descriptors.

### 4.2 Narrow IPC contract

- Define a versioned, typed IPC protocol.
- Validate every message, field, length, enum, and identifier at the receiving boundary.
- Reject unknown mandatory protocol versions and malformed payloads safely.
- Apply request timeouts, cancellation, message-size limits, and bounded concurrency.
- Do not expose arbitrary shell execution, unrestricted filesystem paths, or generic application service access.
- Treat all extension responses as untrusted input.

### 4.3 Capability-based access

Candidate capabilities include source catalogue/search/detail/chapter metadata requests, brokered source HTTP requests, extension-specific settings, and a dedicated extension cache. Do not expose tracking credentials, backup archives, arbitrary user files, full library database access, unrestricted download directories, application configuration/environment variables, or generic process execution by default.

The final capability list and user-consent UX remain open.

### 4.4 Network and credentials

Prefer brokered network access over unrestricted host networking. Scope cookies and authentication state to the relevant source/extension. Never pass global tracking or sync tokens to extension processes. Define redirect, TLS, DNS, proxy, and request-header policies. Redact credentials and sensitive query parameters from logs.

### 4.5 Filesystem

Give each extension a dedicated data/cache directory. Avoid arbitrary host paths over IPC. Use opaque handles or brokered operations where file access is required. Keep backups, library databases, and downloaded chapters outside extension-writable locations. Reject archive traversal and unexpected file types.

### 4.6 Resource controls

Bound memory, CPU, open files, concurrent requests, and response sizes where supported. Enforce deadlines and cancellation. Detect unresponsive hosts and provide controlled restart/disable behavior.

### 4.7 Artifact integrity

Retain repository signing and artifact hash verification. Signatures establish provenance/integrity, not benign behavior. Support key rotation/revocation, make signer changes visible, and define policy for local unsigned artifacts.

## 5. Linux OS isolation — conditional on technical sustainability

Evaluate unprivileged processes, restricted environment/file-descriptor inheritance, namespaces, seccomp, resource limits, and packaging-compatible sandbox facilities.

If a robust and maintainable OS sandbox proves disproportionately complex or unreliable, defer the exact mechanism and its first-release commitment to a later explicit owner decision. Do not describe process separation alone as a sandbox. Deferring OS sandboxing must not weaken the mandatory per-extension process boundary, IPC validation, least-privilege broker, or prohibition on direct access to sensitive application data.

## 6. Android APK / DEX-to-JVM compatibility — first-release requirement

The intended third-party extension ecosystem primarily depends on Android APK extensions. NamiShelf must support APK extensions converted from DEX to JVM-compatible artifacts in the first public release.

DEX-to-JVM conversion does not reproduce Android framework classes or runtime semantics. NamiShelf therefore needs a modular compatibility/runtime adapter for a declared subset of Android and Mihon/Tachiyomi extension APIs.

Requirements:
- Define a supported Android/Mihon/Tachiyomi extension API compatibility matrix.
- Verify APK provenance, signatures, and hashes before conversion and installation.
- Perform conversion and execute the resulting artifact only inside that extension's isolated process.
- Implement explicit adapters for supported Android APIs; reject unsupported calls deterministically.
- Maintain representative real-world APK fixtures and tests for conversion, class loading, source discovery, networking, preferences, lifecycle, and error handling.
- Provide per-extension compatibility diagnostics and redact secrets from logs.
- Describe compatibility as support for a declared API subset, not arbitrary Android application compatibility.
- Keep adapters modular. If a requested Android behavior is disproportionately costly or unsafe to emulate, document the gap and obtain a later owner decision; do not silently claim support.

## 7. Failure handling

The Extension Manager owns lifecycle and recovery: verify artifacts and policy before start; perform an IPC health handshake; apply startup/request timeouts; use bounded restart policy; disable repeatedly crashing or protocol-violating extensions; preserve transactional installation state; and avoid data loss when disabling/removing extensions.

## 8. Security acceptance tests required later

These are future requirements; none were run in this documentation phase.

1. Extension process cannot read NamiShelf databases, backups, or unrelated user files.
2. Extension process cannot access tracking/sync credentials.
3. Extension cannot invoke shell commands through IPC.
4. Malformed/oversized IPC messages are rejected safely.
5. Hung requests time out and can be cancelled.
6. Killing a host does not crash the main process or corrupt persisted data.
7. A compromised extension cannot write outside assigned storage.
8. Network policy and credential scoping are enforced.
9. Resource exhaustion is contained within configured limits where the OS permits.
10. Signature mismatch, revoked signer, and key rotation are handled safely.
11. Flatpak and non-Flatpak deployments enforce the documented posture.
12. Diagnostics redact credentials, cookies, tokens, and sensitive URLs.

## 9. Remaining engineering details

Exact IPC technology/schema, sandbox mechanism, network broker policy, user-consent UX/capability catalogue, and the precise Android/Mihon API compatibility matrix remain open. Resolve these before implementing the extension subsystem. If any Android compatibility feature is disproportionately expensive, present it for a later owner decision rather than silently dropping it.


## 12. Approved IPC transport

The owner selected stdio pipes for extension runtime IPC (ADR-008, 2026-11-03). NamiShelf launches and supervises each extension process and communicates through a length-prefixed, versioned frame protocol. Standard output is protocol-only; standard error is bounded diagnostics. This transport choice does not replace capability checks, process separation, or optional OS-level sandboxing.

Large content must use explicit brokered streaming with quotas, deadlines, cancellation, and backpressure. Native JVM and converted DEX-to-JVM extensions use the same broker and policy boundary.
