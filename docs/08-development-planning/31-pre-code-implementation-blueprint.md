# NamiShelf — Pre-M0 Feasibility Blueprint and Later Engineering Roadmap

**Status:** M0 owner-authorized on 2026-11-03; execution via GitHub Actions evidence workflow  
**Target:** NamiShelf release 1, Linux-first x86_64  
**Authority:** ADR-001 through ADR-027 and the consolidated Architecture Decision Register

## 1. Purpose and boundaries

This document separates the bounded M0 feasibility assessment from subsequent product implementation. M0 determines whether the conditional Mihon Desktop candidate is a viable technical base. It is not a port, fork, product skeleton, feature implementation or release-preparation phase.

The candidate remains conditional until evidence supports a go/no-go decision. The owner explicitly authorized M0 on 2026-11-03, including isolated dependency installation, build and baseline tests of the unmodified candidate. This does not authorize application-code changes, fork/import, extension execution, sync, backup restore, production packaging, PR or merge.

## 2. Approved architecture baseline

The consolidated owner-approved choices are recorded in the Architecture Decision Record and summarized in `docs/01-project-governance/15-architecture-decisions-required.md`. Key constraints relevant to feasibility:

- Linux-first; x86_64 release 1; Linux Mint 22.3 primary reference environment.
- Dedicated application JVM/runtime for release 1; exact distribution/version/packaging to be determined from M0 evidence. No arbitrary system-JVM fallback.
- X11 mandatory first-release baseline; Wayland support only for tested compositor/session combinations.
- Compose Multiplatform Desktop reference UI, subject to M0 feasibility/performance/compatibility gates.
- SQLite embedded reference local persistence, subject to candidate integration feasibility.
- Modular Monolith logical architecture; physical Gradle decomposition deferred.
- XDG-separated data/config/cache and user-selectable permanent download directory.
- Linux Secret Service only persistent credential backend, fail-closed with no fallback.
- Extension capabilities under ADR-025; no extension execution during M0.
- Native backup recovery under ADR-026; no backup restore or cryptographic implementation during M0.
- Sync E2EE mandatory, GitHub first and WebDAV second/self-hosted; historical ciphertext re-encryption after revocation under ADR-024. No sync implementation during M0.
- DEB updates via APT/repository, Flatpak updates via Flatpak, AppImage via dedicated user-space updater; packaging is later release engineering.
- Initial public release channel Stable only.

## 3. M0 — Linux feasibility and baseline

### 3.1 Preconditions

Before M0 begins, the owner must explicitly authorize:
- use of the conditional Mihon Desktop candidate for read-only inspection and baseline feasibility;
- creation/use of an isolated Linux x86_64 environment;
- dependency installation and build operations needed to evaluate the unmodified candidate;
- execution of the candidate and its existing baseline test suite, where safe.

This authorization does not include candidate code modifications or any production derivation.

### 3.2 Work plan

| Step | Activity | Required evidence |
|---:|---|---|
| 1 | Pin exact Mihon upstream and Mihon Desktop commits | Commit IDs, tags, source provenance |
| 2 | Inspect repository structure, build system, Linux support claims and build instructions | Source/build inventory and known constraints |
| 3 | Establish isolated reproducible Linux x86_64 environment on a GitHub-hosted Ubuntu runner; retain Mint 22.3 as the eventual reference-machine acceptance target | Environment manifest and reproducibility notes |
| 4 | Build and launch unmodified candidate, if possible | Commands, logs, exit status, screenshots/observations where useful |
| 5 | Run existing baseline tests where safe | Test inventory, pass/fail/skip details |
| 6 | Inventory JVM, native libraries, architecture-specific artifacts, graphics/image dependencies and desktop integration | Dependency/native compatibility matrix |
| 7 | Produce source/dependency inventory, SBOM where feasible, license/NOTICE/provenance review | Findings and unresolved obligations |
| 8 | Assess Compose Desktop, SQLite, dedicated-JVM packaging feasibility and Secret Service integration | Evidence-based feasibility matrix; no implementation |
| 9 | Inspect XDG/profile behavior and confirm no legacy Mihon profile is read or mutated | Filesystem observations and isolation evidence |
| 10 | Assess X11 baseline and record Wayland observations without broad compatibility claims | Session/compositor observations |
| 11 | Produce go/no-go report | Recommendation, blockers, risks and post-M0 decisions |

### 3.3 Explicit non-goals

M0 does not:
- edit candidate or NamiShelf application code;
- create a NamiShelf fork, import selected source or establish a production derivation;
- add an architecture skeleton or new application modules;
- execute third-party extensions or Android APK conversions;
- implement the broker, IPC, sandbox, backup, restore, sync, E2EE or key management;
- create AppImage, Flatpak or DEB packages or configure release channels;
- change branches, commit, push, open PRs or merge.

If a step requires candidate modification, destructive/privileged action, external account credential, or a scope expansion, stop and request separate authorization.

### 3.5 Remote execution workflow

The owner approved using GitHub Actions rather than a local Linux machine for the remaining build/test evidence. The dedicated workflow is `.github/workflows/m0-linux-feasibility.yml` and checks out the external candidate at the immutable SHA recorded in the M0 report. It uses a GitHub-hosted `ubuntu-24.04` runner, JDK 21, read-only repository permissions, and uploads logs/reports as a time-limited artifact. It must not receive application secrets or write access to the external candidate.

The workflow currently covers toolchain preflight, desktop JVM compilation, `:app-desktop:jvmTest`, runtime dependency reporting and a non-gating Linux native-distribution probe. A workflow definition is not test evidence: M0 remains open until a completed run and artifacts are reviewed. Graphical X11 launch, Secret Service, URI integration, native-library closure, full SBOM/license validation and final GO/NO-GO still require separate evidence. A GitHub-hosted Ubuntu result also does not replace final compatibility validation on the approved Linux Mint 22.3 reference environment.

### 3.4 M0 exit criteria

M0 is complete only when the report contains:
- exact source commit provenance;
- reproducible build/start result or a precise blocker;
- baseline test outcomes and known failures;
- JVM/runtime and native dependency inventory;
- license/NOTICE/SBOM/provenance assessment;
- XDG/profile isolation findings;
- feasibility assessment against the approved Compose Desktop, SQLite, dedicated JVM and Secret Service constraints;
- X11 baseline and Wayland observations;
- go/no-go recommendation and explicit post-M0 owner decisions.

A successful build alone does not mean the candidate is accepted.

## 4. Post-M0 roadmap — not authorized by M0

The following workstreams are ordered by dependency. They require a separate implementation authorization and do not begin automatically when M0 completes.

| Order | Workstream | Depends on | Exit evidence |
|---:|---|---|---|
| 1 | Base strategy and downstream ownership decision | M0 go/no-go | Owner-approved fork/import/derivation plan, provenance and delta ledger |
| 2 | NamiShelf application skeleton and XDG profile | 1 | Separate app identity, clean profile, migration boundaries |
| 3 | Core domain and SQLite persistence | 2 | Domain mapping, schema/migrations and recovery tests |
| 4 | Secret Service credential backend | 2–3 | Fail-closed round-trip and missing/locked/error tests |
| 5 | Library, source and reader adaptation | 2–3 | Functional regression and measured memory behavior |
| 6 | Extension process supervisor and IPC | 2 | Protocol conformance and process lifecycle |
| 7 | Broker, capability profiles and network enforcement | 6 | Origin isolation, auth boundaries, revocation and abuse tests |
| 8 | DEX-to-JVM and Android/Mihon compatibility | 6–7 | Fixture-backed API coverage and deterministic unsupported behavior |
| 9 | Native backup and Mihon legacy adapters | 3 | Native round-trip, .tachibk import/export and loss reports |
| 10 | Provider-neutral sync and E2EE core | 3–4 | Independent crypto review, state/conflict protocol, recovery/revocation tests |
| 11 | GitHub and WebDAV adapters | 10 | Provider conformance and concurrent/interruption tests |
| 12 | Packaging and release engineering | 4–11 | Independent AppImage, Flatpak and DEB acceptance |
| 13 | Full security, licensing and regression audit | all | Release evidence and residual-risk register |

Do not parallelize work that violates dependencies. Extension runtime, sync cryptography, backup restore and packaging are security-sensitive and require independent review.

## 5. Later engineering details and owner gates

These are not missing owner decisions that block M0. They must be resolved with evidence before their respective implementation/release workstreams:

- Exact JVM distribution/version and packaging mechanism.
- Minimum Debian/Ubuntu/Arch versions and tested distribution matrix.
- Flatpak runtime/application ID and portal permissions.
- AppImage updater technology, signing format, key custody/rotation and rollback health checks.
- Compose/Kotlin versions and concrete navigation/state/DI libraries.
- SQLite driver, SQL access layer, schema and concurrency configuration.
- Physical Gradle module decomposition.
- Content digest, manifest serialization, garbage collection and migration.
- Extension IPC v1 framing, limits, method registry and Android/Mihon API coverage.
- OS-level sandbox mechanism and its release commitment; process separation alone is not a sandbox.
- Backup JSON/ZIP profile, integrity, parser limits and independently reviewed cryptographic envelope.
- Sync key hierarchy/cryptographic suite, causal/conflict model, provider conformance and revocation fixtures.
- Exact reader cache/prefetch budgets based on profiling.

ADR-011 is intentionally unassigned. Document 29 is an engineering protocol proposal, not an owner-approved ADR. Do not cite ADR-011 as authority.

## 6. Operational guardrails

- No changes to Mihon upstream or the external Mihon Desktop repository.
- Preserve all license notices, attribution and provenance.
- No release claim without reproducible evidence.
- No secrets in logs, IPC diagnostics, command-line arguments, environment variables, backups or extension processes.
- No arbitrary extension code in the main process.
- No custom cryptography; independent review is mandatory.
- An architecture decision is not authorization to implement, build, install dependencies, execute extensions, package, merge or publish.
