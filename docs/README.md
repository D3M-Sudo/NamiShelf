# NamiShelf — Documentation Phase 01

**Status:** Draft / evidence-backed planning  
**Date:** 2026-11-03

## Scope

This documentation baseline records the conditional downstream-base decision, the initial read-only review of `AltairCardinal/mihon-desktop`, Linux porting implications, upstream synchronization principles, and the owner-approved extension execution and first-release requirements.

It is not a security certification, a complete source-code audit, or a guarantee of universal Android API compatibility. It does not authorize changes to Mihon/Mihon Desktop.

## Documents

- `01-downstream-audit.md` — initial evidence and risks found in Mihon Desktop.
- `02-architecture-decision-record.md` — conditional architecture decision and extension decisions.
- `03-upstream-sync-strategy.md` — upstream tracking and selective integration policy.
- `04-linux-platform-design.md` — Linux/Mint porting workstreams and validation matrix.
- `05-extension-security-model.md` — approved per-extension process model, brokered data access, conditional OS isolation, and required Android APK/DEX-to-JVM compatibility.
- `06-license-and-dependency-review.md` — preliminary licensing and dependency inventory findings.
- `07-agent-handoff.md` — constraints and workflow for the next authorized phase.

## Current decisions

| Topic | Decision |
|---|---|
| Product identity | NamiShelf, independent product inspired by Mihon |
| Candidate technical base | `AltairCardinal/mihon-desktop`, conditional |
| Upstream policy | Keep upstream reference and integrate changes selectively |
| Linux target | Linux Mint 22.3 primary; Ubuntu-, Debian-, and Arch-based distributions in scope |
| Packaging | AppImage, Flatpak, DEB |
| Extension execution | One separate process per extension |
| Extension data access | Mediated by NamiShelf through a versioned IPC/broker interface |
| Android APK extensions | DEX-to-JVM conversion plus declared Android/Mihon API compatibility required in first release |
| Third-party extensions | Included in first public release |
| OS isolation | Pursue if technically sustainable; exact mechanism may be deferred by later decision |
| Documentation publication | Authorized on an existing work branch |
| Application coding/builds/dependency installation | Not authorized |

## Evidence baseline

The downstream repository was inspected read-only on `main` at:

`701080c251c8fd8b35d0b57a481d93b9a9eb3701`

The upstream Mihon snapshot previously recorded for comparison is:

`mihonapp/mihon@e588e04359fe1058206a2cbac1b01018ad787359`

Re-check both references before future implementation work.

## Review limitations

No code was built or executed. No dependency resolution, advisory scan, full license report, Linux runtime validation, extension fixture execution, or reproducible packaging test was performed.
