# NamiShelf — Upstream Synchronization Strategy

**Status:** Proposed policy based on the owner's `upstream` decision  
**Date:** 2026-11-03

## 1. Remotes and lineage

Conceptual lineage:

`mihonapp/mihon` → `AltairCardinal/mihon-desktop` → `D3M-Sudo/NamiShelf`

Mihon Android remains the upstream reference; Mihon Desktop is the immediate technical-base candidate. NamiShelf is an independent product and must not imply official Mihon affiliation.

The intended strategy is to retain upstream references and selectively integrate changes. Do not merge upstream `main` wholesale.

## 2. Selective integration workflow

For each synchronization batch:

1. Record immutable upstream commit ranges and release notes.
2. Identify changed modules, APIs, database schemas, source contracts, backup formats, and security-sensitive code.
3. Map affected changes to NamiShelf adapters and product behavior.
4. Classify each change:
   - **Adopt:** compatible without material adaptation.
   - **Adapt:** useful but needs desktop/Linux-specific work.
   - **Defer:** outside the current milestone or too risky.
   - **Reject:** incompatible with scope or unacceptable risk.
5. Record rationale in an integration log.
6. Integrate only after review, tests, and rollback planning.

## 3. Protected areas

Require elevated review for database schemas/migrations, source and extension APIs, backup serialization/restoration, reader state, downloads/filesystem paths, authentication/sync credentials, extension verification/loading/process boundaries, and platform abstractions/Android compatibility shims.

## 4. Feature provenance map

Maintain a feature-level record:

| Feature | Mihon upstream | Mihon Desktop adaptation | NamiShelf behavior | Classification |
|---|---|---|---|---|
| Library | Reference commit/symbols | Desktop UI/adapter | Product behavior | Shared/adapted/custom |
| Reader | Reference commit/symbols | Desktop input/rendering | Product behavior | Shared/adapted/custom |
| Sources | Source API contract | JVM loader/network | Product behavior | Shared/adapted/custom |
| Extensions | Android package/signature model | DEX/JAR adapter | Isolated process + broker | Adapted/custom |
| Backup | Protobuf/gzip format | Desktop file access | Interoperability target | Shared/adapted/custom |
| Downloads | Domain semantics | Desktop scheduler/filesystem | Product behavior | Adapted/custom |
| Tracking | Tracking model | Desktop UI/auth | Product behavior | Shared/adapted/custom |

Never label downstream-only enhancements as Mihon upstream behavior.

## 5. Cadence and triggers

Do not promise automatic merges. Review stable Mihon releases and security/API changes often enough to avoid large, unreviewable deltas. Suggested triggers: stable release, security advisory, source/extension API change, database/backup migration, or critical shared-module fix. The owner/maintainer approves each integration batch.

## 6. Integration record

Each record should include upstream repository and immutable commit range, downstream base commit, changed modules/files, affected provenance entries, adopt/adapt/defer/reject decisions, conflict/migration notes, tests and results, rollback plan, reviewer, and approval.

## 7. Guardrails

- Never push NamiShelf changes to Mihon or Mihon Desktop.
- Never cherry-pick without reviewing dependencies and consumers.
- Never silently rewrite database or backup semantics.
- Never discard NamiShelf-specific fixes during synchronization.
- Preserve immutable references for decisions and audit evidence.
