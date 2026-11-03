# NamiShelf — Security, Lifecycle and Distribution Architecture

**Status:** Owner-approved release policy baseline under ADR-027; exact distribution/runtime versions and platform evidence remain subject to M0 and release gates.

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

Never reuse one class of secret as another. Persistent secrets belong in the OS credential service. Secret Service is the only persistent credential backend and fails closed with no fallback, as required by ADR-023. No plaintext file, command-line, environment-variable, database, backup or extension-process fallback is permitted.

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

Owner-approved baseline under ADR-027:
- Linux-first; release 1 targets x86_64 only. Future platform support remains an architectural goal, not a release commitment.
- Linux Mint 22.3 x86_64 is the primary end-to-end reference environment. Debian/Ubuntu-family and Arch-based distributions are compatibility targets only when exact releases and package behavior have been tested and documented.
- Ship and control a dedicated application JVM/runtime; do not require or silently fall back to an arbitrary system Java installation.
- X11 is the mandatory first-release baseline. Wayland is a compatibility target advertised only for compositor/session combinations that pass the release matrix.
- DEB uses APT/configured repositories, Flatpak uses Flatpak remotes, and AppImage uses the dedicated per-user NamiShelf updater, as specified by ADR-022.
- Stable is the only initial public release channel. Preview/testing channels require a separate decision.
- Each package format is an independent release gate. A successful build alone is insufficient; require reproducible install/upgrade/uninstall, rollback where applicable, user-data preservation and end-to-end evidence.
- Minimum Debian/Ubuntu/Arch release versions remain open until M0 inspects actual native dependencies and runtime requirements; submit an evidence-backed matrix before packaging freeze.

Deferred M0/release engineering:
- Exact JVM distribution/version and runtime packaging mechanism.
- Minimum supported distro releases and exact tested derivative matrix.
- Flatpak runtime/application ID and portal permissions.
- AppImage updater technology, metadata/signature format and key lifecycle.
- Wayland compositor/session test matrix.
- OS-level sandbox mechanism for extension processes.

These details must preserve ADR-027 and other approved security decisions.

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
