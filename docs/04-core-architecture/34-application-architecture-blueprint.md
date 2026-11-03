# NamiShelf — Preliminary Application Architecture Blueprint

**Status:** Proposed architecture baseline; owner approval required for decisions explicitly marked as such.  
**Scope:** Release 1 architecture, before M0 and before application implementation.

## 1. Architectural objectives

NamiShelf is an independent Linux desktop application inspired by Mihon. Its architecture must preserve useful Mihon/Mihon Desktop domain behavior while providing a desktop-first UX, Linux-native integration, isolated third-party extensions, portable backup, and provider-neutral E2EE synchronization.

Primary constraints:
- Kotlin/JVM desktop candidate; Compose Desktop is the current UI baseline inherited from the conditional downstream candidate.
- Linux Mint 22.3 x86_64 is the first validation target.
- Modular monolith for trusted application code; separate OS process for every third-party extension.
- Offline-first local library and reader operation.
- Stable domain contracts independent from UI toolkit, storage implementation, provider transport, and Android compatibility shims.
- No extension receives direct database, credential, backup-root, or arbitrary host filesystem access.
- Security-sensitive operations fail closed.

## 2. Recommended top-level structure

Use a layered modular monolith with explicit dependency direction:

    Desktop Application Shell
      ├── Presentation / Navigation
      ├── Application Use Cases
      ├── Domain Modules
      ├── Ports (interfaces)
      └── Infrastructure Adapters
            ├── Local Database / Filesystem
            ├── Linux Desktop Integration
            ├── Network / Source Runtime
            ├── Extension Supervisor + IPC
            ├── Backup Adapters
            ├── Sync Engine + Provider Adapters
            └── Credential Store

The desktop shell composes modules and owns application lifecycle. Feature modules must not instantiate global infrastructure directly. Domain modules must not depend on Compose, Android APIs, provider SDKs, or concrete database drivers.

## 3. Proposed module boundaries

| Module | Owns | Must not own |
|---|---|---|
| app-shell | startup, dependency composition, window lifecycle, shutdown | feature business rules |
| navigation | routes, navigation state, deep-link dispatch | persistence or network |
| design-system | shared UI primitives, tokens, accessibility conventions | feature workflows |
| feature-library | library views and library use-case orchestration | raw SQL/provider clients |
| feature-reader | reader session UX, page presentation and controls | source transport implementation |
| feature-sources | source discovery/management UX | direct extension process access |
| feature-downloads | queue UX and download orchestration | arbitrary filesystem paths |
| feature-tracking | tracker UX and tracking use cases | tracker secrets in UI state |
| feature-history | reading history UX and queries | direct database access |
| domain-* | entities, value objects, invariants, domain services | UI/framework/transport |
| application-* | use cases, transaction boundaries, authorization orchestration | framework-specific UI |
| data-* | repositories, schema, migrations, persistence mappings | presentation state |
| platform-linux | XDG, portals, Secret Service, desktop integration | domain policy |
| source-runtime | source contracts and host-side source coordination | core DB handles exposed to extensions |
| extension-host | process supervision, protocol, broker and capability enforcement | trust in extension claims |
| backup | canonical backup model, container and adapters | direct unvalidated restore mutation |
| sync | provider-neutral journal, conflict model, E2EE envelope | provider-specific assumptions |
| sync-provider-github | GitHub storage/auth adapter | sync domain rules |
| sync-provider-webdav | WebDAV storage/auth adapter | sync domain rules |

This is a logical decomposition, not a mandate to create one Gradle module per row immediately. Begin with a small number of build modules and preserve package/API boundaries; split into physical modules when dependency enforcement and build ownership justify it.

## 4. Dependency rules

1. Presentation depends on application use cases and design-system contracts.
2. Application services depend on domain models and ports.
3. Domain depends only on the Kotlin/JVM standard library or narrowly approved pure libraries.
4. Infrastructure implements ports and may depend on approved platform/runtime libraries.
5. Feature modules do not reach into another feature's repositories; cross-feature work goes through application contracts/events.
6. Extension child processes communicate only through the versioned stdio protocol.
7. Backup and sync consume explicit export/import domain services, never raw database files.
8. Provider adapters implement a provider-neutral object-store contract.
9. Linux-specific behavior is behind platform interfaces.
10. Android/Mihon compatibility is isolated in adapters and cannot bypass host authorization.

## 5. Application lifecycle and composition

Startup sequence:
1. Resolve application identity and XDG directories.
2. Acquire single-instance/locking policy if adopted.
3. Initialize structured redacted logging.
4. Open and migrate the local database with backup/rollback safeguards.
5. Initialize platform services and credential-store capability state.
6. Load source and extension metadata without executing untrusted extension code.
7. Initialize download and sync schedulers in paused/recovery-aware state.
8. Restore navigation/window state that is safe to restore.
9. Present the desktop shell.

Shutdown sequence:
1. Stop accepting new user operations.
2. Persist reader and UI state.
3. Cancel or checkpoint downloads and sync according to their transaction semantics.
4. Request graceful extension shutdown; enforce deadlines and terminate unresponsive child processes.
5. Flush/close repositories and database.
6. Close platform resources and logging.

Startup must not silently enable an extension repository, execute newly imported extensions, or start a sync operation before required credentials and key state are available.

## 6. Communication patterns

- Direct synchronous calls for small in-process queries and commands.
- Explicit asynchronous use cases for network, file, download, backup and sync operations.
- Typed domain/application events for decoupled cross-feature notifications.
- No global mutable event bus as a substitute for ownership.
- Every long-running operation has cancellation, progress, terminal state and recoverable error representation.
- UI state is a projection of application state, not a second source of truth.

## 7. Error model and observability

Separate validation errors, domain conflicts, authorization denials, infrastructure failures, unsupported compatibility APIs, corruption, cancellation and partial completion. Errors crossing module boundaries must be typed and safe to display. Internal diagnostics use correlation IDs and redaction; never log credentials, page content, backup payloads, extension stream bytes or E2EE material.

## 8. Decisions intentionally not frozen

- Exact Gradle module count and package naming.
- DI framework versus explicit constructor composition.
- Database driver and SQLDelight/Exposed/JDBC choice.
- Event mechanism and coroutine dispatcher policy.
- Whether the candidate's existing feature structure can be retained or must be refactored.
- Exact plugin/extension host process launcher under AppImage, Flatpak and DEB.

These depend on M0 source/build inspection and must not be guessed from static architecture alone.

## 9. Acceptance criteria before implementation

- Every module has a one-sentence responsibility and dependency contract.
- No domain-to-UI/platform dependency cycles.
- Application lifecycle has startup, failure recovery and shutdown behavior.
- All privileged operations have an owning application service and authorization boundary.
- Extension, backup and sync boundaries match their dedicated specifications.
- A source-to-NamiShelf provenance map identifies reused, adapted and new behavior.

No code, build, dependency installation or executable test has been performed.
