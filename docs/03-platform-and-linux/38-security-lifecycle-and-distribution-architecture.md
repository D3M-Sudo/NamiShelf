# NamiShelf — Security, Lifecycle and Distribution Architecture

**Status:** Proposed security and release architecture; owner decisions and platform validation remain open.

## 1. Trust zones

NamiShelf has four distinct trust zones:

1. **Trusted host process:** domain/application services, local database, user interface and policy enforcement.
2. **Third-party extension process:** untrusted code, one process per extension, constrained IPC and broker-mediated access.
3. **External providers/services:** sources, trackers, GitHub, WebDAV and remote hosts; all responses are untrusted.
4. **OS/platform services:** Secret Service, portals, filesystem, desktop session and package runtime.

Process separation is mandatory for third-party extensions. OS-level sandboxing is pursued where reliable and maintainable across target packages; process separation alone is not represented as a sandbox.

## 2. Authorization architecture

All privileged operations require host-side authorization:
- Identify calling extension and active process session.
- Resolve requested method to a fixed registry entry.
- Evaluate required capability against product policy, assigned profile, user overrides, source policy and current revocation state.
- Validate origin, input schema, quotas and request lifecycle.
- Execute through a host-owned adapter.
- Return bounded output or opaque stream handles.

Extension-provided claims never grant permission. Unknown methods, capabilities, API versions and required features fail closed.

## 3. Credential architecture

Separate:
- source authentication/session state;
- tracker credentials;
- provider access tokens;
- E2EE key material;
- optional backup password/key derivation input.

Never reuse one class of secret as another. Persistent secrets belong in the OS credential service. The current recommendation is fail-closed when Secret Service is unavailable; no plaintext file, command-line, environment-variable or extension-process fallback. This specific availability policy still requires owner confirmation.

Extensions do not receive global credentials. Source authentication is scoped to the relevant source and origin. Logs, diagnostics, backups and crash reports redact secrets.

## 4. Data and filesystem protection

- Use clean NamiShelf XDG profile.
- Explicit migration from Mihon Desktop is read-only at source and opt-in.
- Restrict file selection through user-selected destinations and portals where available.
- Keep caches, durable downloads, backup files and database files in separate managed locations.
- Validate every imported archive, settings payload, source URL and extension artifact.
- Do not auto-run extensions or auto-enable repositories after restore.
- Set restrictive permissions on private application data and temporary files.

## 5. Process and operation lifecycle

Every extension process has an explicit lifecycle: discovered → verified → permissioned → starting → handshake → active → stopping → stopped/failed. Revocation terminates the process and invalidates all outstanding requests and stream handles.

Long-running tasks expose cancellation and recovery. Shutdown stops new work, checkpoints safe operations, terminates extensions within deadlines and closes storage cleanly.

## 6. Linux distribution architecture

Logical packaging goals:
- AppImage: portable user-space distribution; verify bundled JVM/native libraries, read-only execution path, update/signature policy and desktop integration.
- Flatpak: sandboxed deployment; use portals, minimal permissions and a declared runtime/application ID. Extension child-process launch and filesystem/network policy must be tested against Flatpak restrictions.
- DEB: system package integration with declared dependencies and lifecycle scripts; never execute app logic as root.

Packaging formats are independent release gates. A successful build of one format does not establish compatibility of another.

## 7. Runtime and compatibility policy

Still open:
- bundled versus system JVM;
- minimum supported Debian/Ubuntu/Arch release window;
- Wayland release criteria and X11 baseline;
- Flatpak runtime/application ID;
- AppImage update and signature mechanism;
- Secret Service failure behavior;
- OS-level sandbox mechanism for extensions.

The candidate's actual native dependencies and current runtime packaging must be inspected before selecting these.

## 8. Security verification gates

Before public release:
- threat model and abuse cases for each trust boundary;
- extension protocol fuzzing and malicious fixture tests;
- SSRF, redirect, DNS rebinding and cookie-isolation tests;
- credential leak checks across logs, IPC, crash reports and process metadata;
- archive traversal, decompression bomb and restore interruption tests;
- sync cryptographic review, known-answer vectors, replay/rollback and revocation tests;
- provider concurrency and interrupted transfer tests;
- package permission review and runtime integration tests;
- dependency/SBOM/license/advisory review.

## 9. Release readiness

Release only when:
- each supported package has reproducible build and install/upgrade/uninstall evidence;
- Mint 22.3 x86_64 passes end-to-end acceptance;
- critical workflows work offline and recover after interruption;
- no unresolved critical/high security issue remains without explicit risk acceptance;
- compatibility claims match tested Mihon/Android API fixtures;
- backup import/export and restore have real golden fixtures;
- sync recovery/revocation and provider adapters pass conformance;
- licenses, notices, provenance and attribution are complete.

No platform installation, build, package creation or security test has been performed.
