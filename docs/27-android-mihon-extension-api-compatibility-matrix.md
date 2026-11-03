# NamiShelf — Android/Mihon Extension API Compatibility Matrix

**Status:** Planning matrix; no API family is declared implemented or validated  
**Date:** 2026-11-03

## 1. Compatibility contract

DEX-to-JVM conversion changes bytecode representation; it does not provide Android framework classes, lifecycle semantics, resources, package services, or Mihon runtime behavior. NamiShelf must define a versioned compatibility profile, not claim arbitrary APK extensions run unchanged.

Each API is classified as:
- **Supported:** implemented and fixture-verified.
- **Limited:** works with documented semantic differences.
- **Unsupported:** deterministic explicit failure; never a silent no-op.
- **Prohibited:** denied for security/product-boundary reasons.
- **Unassessed:** not advertised as supported.

Every entry must record API level, signatures, semantic differences, broker capability, failure behavior, and fixture evidence.

## 2. Runtime layers

1. APK intake: inspect manifest, signer/provenance, SDK levels, permissions, components, ABI/native libraries.
2. DEX-to-JVM conversion: conversion success is not compatibility evidence.
3. JVM Android shim: expose only explicitly supported Android classes.
4. Mihon source API adapter: pin an upstream API baseline and implement declared source contracts.
5. Broker bridge: all privileged operations use the same stdio IPC broker as native JVM extensions.
6. Resource and lifecycle adapters: implement only the declared subset.

## 3. API family matrix

| API family | Initial classification | Strategy and constraints |
|---|---|---|
| Kotlin/JVM, coroutines, serialization | Supported candidate | Pin runtime dependencies; verify ABI, cancellation, dispatchers and serializer versions using fixtures |
| Mihon Source / HttpSource | Required core target | Versioned adapter for search, details, listings and chapters; pin upstream revision |
| OkHttp | Limited | Host-owned network broker; explicitly define redirects, cookies, TLS, headers, cache, interceptors, timeouts and streaming |
| android.content.Context | Partial | Narrow shim for approved app-scoped services; never expose unrestricted host context |
| SharedPreferences | Partial | Extension-scoped settings broker; typed values, quotas and migration |
| Mihon PreferenceScreen / Android preference UI | Unsupported as native UI | Translate known descriptors to desktop settings; custom Android Views remain unsupported |
| android.net.Uri | Partial | Value/parser compatibility only; no filesystem or network authority |
| ContentResolver / ContentProvider | Mostly unsupported | Only explicit brokered content operations; no arbitrary providers or document URI authority |
| android.graphics.Bitmap / BitmapFactory | Partial | Map to host image pipeline; verify pixel formats, density, color profiles and memory ownership |
| Android resources/assets/XML | Partial | Narrow resource lookup; themes, qualifiers and Android UI resources not equivalent |
| Activity, Fragment, Intent, lifecycle | Unsupported for source extensions | Map to host lifecycle/events; arbitrary Android UI flows unsupported |
| WebView | Unsupported initially | No Android WebView emulation; browser-mediated auth needs separate design |
| PackageManager / APK install APIs | Prohibited | Extension installation and artifact management belong to NamiShelf |
| AccountManager / Android account credentials | Prohibited | Source-scoped broker credentials only; no global or NamiShelf sync/tracking secrets |
| DownloadManager / filesystem APIs | Prohibited/partial | Explicit host download/content operations only; no arbitrary paths |
| Android services, broadcasts, Binder | Unsupported initially | Only named typed host events via IPC |
| Native .so / JNI | Unassessed; default unsupported | DEX conversion does not convert native code; ABI-specific execution adds risk |
| Reflection, class loading, Java serialization | Restricted | Allowlist packages/classes; reject unsafe deserialization and arbitrary dynamic loading |

## 4. Broker capability mapping

| Legacy operation | Capability candidate | Default |
|---|---|---|
| Search/details/chapter listing | source.query | Bound to installed extension/source identity |
| HTTP fetch | network.fetch | HTTPS, origin and redirect policy enforced by host |
| Source login/session | source.auth | Source-scoped vault only |
| Extension preferences | extension.settings.read/write | Extension namespace only |
| Image/page retrieval | content.fetch / content.stream | Bounded streaming, quotas and cancellation |
| Notifications | host.notification | Narrow typed events |
| User-selected local content | content.pick | Explicit user gesture and scoped handle |
| Arbitrary filesystem/database access | None | Prohibited |
| Sync, backup, tracking tokens | None | Prohibited |

## 5. Versioning

- Pin the Mihon extension API baseline by tag/commit.
- Version the compatibility profile independently of NamiShelf.
- Show required/optional APIs and support status before installation.
- Reject activation if required APIs are unsupported.
- Optional APIs return documented capability errors.
- Never broaden permissions silently during upgrades.
- Maintain a fixture corpus for each supported profile.

## 6. Fixture plan

Subject to implementation authorization, use legally redistributable representative fixtures for: simple source, custom headers/redirects, cookie/login, preferences, large image/page payloads, coroutine cancellation, unsupported Android APIs, malformed APKs, unsupported native libraries, missing classes and API-version mismatch.

During this planning phase, do not download, install, convert, or execute extension APKs.

## 7. Security constraints

- One process per extension; stdio IPC is the only host control/data interface.
- Android shims never return privileged Context/service handles.
- No raw host paths, arbitrary file descriptors, environment secrets, provider tokens, sync keys, tracking credentials, backup handles or database connections.
- Network passes through the broker under the owner-approved source-declared-origin policy: validate every redirect hop, enforce TLS, isolate cookies by source, strip sensitive headers across origins, and deny private/local destinations by default.
- Capabilities are granted through editable predefined profiles; sensitive capabilities require explicit additional consent and upgrades never silently broaden grants.
- Large payloads use temporary opaque broker handles bound to process/session/request, with expiry, quotas, cancellation and backpressure.
- Treat APK metadata, manifests, resources and extension responses as untrusted.
- Process separation is not an OS sandbox; document actual package-specific isolation.

## 8. Release-1 acceptance gates

1. Exact Mihon API baseline and method-level matrix published.
2. Representative APK conversion and class resolution verified.
3. Source contracts, OkHttp mapping, coroutines, preferences and image behavior tested.
4. Every privileged operation broker-authorized.
5. Unsupported APIs fail deterministically.
6. No secrets or privileged descriptors leak across IPC.
7. Process lifecycle, cancellation, crash handling and quotas verified under AppImage, Flatpak and DEB.
8. Runtime components and fixtures pass license/provenance review.
9. Independent security review and malformed/malicious fixture testing completed before enabling third-party extensions by default.

## 9. Open design work

Exact Mihon API baseline; Android shim strategy; OkHttp/cookie/redirect semantics; preference descriptor schema; bitmap interoperability; native library policy; browser-mediated login; capability consent UX; per-process resource limits and package-specific sandbox.

No extension has been downloaded, converted, installed or executed. No API family in this matrix has been runtime-validated.


## 10. Owner-approved broker policy

The owner selected source-declared origin restrictions, editable capability profiles, and broker-issued streaming handles (ADR-010). Converted Android/Mihon APIs must map to these same policies; no Android compatibility shim may bypass them.

- HTTP operations are restricted to the declared and host-approved source origins. Redirects are re-authorized hop by hop.
- Cookies and source authentication are isolated by source. Sensitive headers are not forwarded across origin changes.
- Local/private destinations are denied by default.
- Effective capabilities come from the host profile and user overrides, not extension self-declaration.
- Large image/page results use temporary opaque broker handles with scoped lifetime and quotas.
