# NamiShelf

NamiShelf is an independent Linux desktop application project inspired by Mihon. It is not an official Mihon product.

## Project status

The project is in its architecture/specification phase. The current technical base candidate is `AltairCardinal/mihon-desktop`, subject to Linux portability, security, dependency, licensing, backup, and maintenance acceptance gates. Detailed pre-code specifications now cover extension IPC v1, capability profiles, Android/Mihon compatibility, native backup v1, provider-neutral E2EE sync, and ordered implementation gates. No application code implementation or Linux runtime validation is claimed at this stage.

## Documentation

The planning and audit baseline is maintained in [`docs/`](docs/01-project-governance/00-documentation-index.md), covering the downstream audit, architecture decisions, upstream synchronization, Linux platform design, extension security and compatibility, backup interoperability, native backup architecture and Mihon adapters, sync architecture, reader/image pipeline, Linux runtime/dependencies, credential storage, backup safety, provider-neutral synchronization, XDG migration, Linux release gates, architecture decisions, licensing, and agent handoff.

## Key architecture directions

- Independent product identity and roadmap; selective upstream synchronization.
- One separate process per third-party extension.
- Extension data access mediated by NamiShelf through a versioned, validated IPC/broker interface.
- Android APK extensions converted from DEX to JVM-compatible artifacts are a first-release requirement, subject to a documented Android/Mihon API compatibility matrix.
- OS-level sandboxing is pursued where technically sustainable; the exact mechanism may be deferred by an explicit later decision if disproportionate or unreliable.

## Current limitations

This repository currently contains planning documentation, not a completed or validated application. Builds, tests, dependency resolution, Linux packaging, and runtime compatibility checks have not yet been performed.
