# NamiShelf — Extension Capability Catalogue and Permission Profiles

**Status:** Engineering proposal aligned with owner-approved profile model; profile names/default grants require product review before implementation freeze.

## 1. Authorization model

Capabilities are host-defined, versioned identifiers. An extension can declare requirements but cannot grant itself permissions. Effective permission is calculated by the host from product security policy, installed extension/source identity, user-selected profile and overrides, current consent state, and operation-specific constraints.

Default deny. A denied capability returns CAPABILITY_DENIED; the extension cannot substitute a broader API.

## 2. Capability catalogue

| Capability | Scope | Sensitivity | Default |
|---|---|---|---|
| source.search | Current source | Routine | Standard Source |
| source.details | Current source | Routine | Standard Source |
| source.chapters | Current source | Routine | Standard Source |
| network.fetch | Declared source origins only | Routine, constrained | Standard Source |
| content.stream.read | Handle issued for current request | Routine, quota-bound | Standard Source |
| extension.settings.read | Extension namespace only | Routine | Standard Source |
| extension.settings.write | Extension namespace only | Routine | Standard Source |
| source.auth | Credentials for this source only | Sensitive | Explicit consent |
| host.notification | Typed notification only | Optional | Disabled until enabled |
| content.pick | User-selected file/URI, one operation | Sensitive | User gesture plus consent |
| source.external-origin | Additional declared origin | Sensitive | Explicit per-origin approval |
| host.clipboard.read | Clipboard contents | High sensitivity | Not available in release 1 |
| host.filesystem | Arbitrary host paths | Prohibited | Never granted |
| database.raw, backup.raw, sync.keys, tracking.credentials, provider.credentials, shell.exec | Core secrets/state/host | Prohibited | Never exposed |

Capability scopes include extension ID and source ID where relevant. A permission granted to one source must not implicitly apply to another source from the same package.

## 3. Initial profiles

Proposed defaults for release-1 UX:

### Standard Source (default)
- source.search, source.details, source.chapters
- network.fetch to source-declared, host-validated HTTPS origins
- content.stream.read for broker-issued handles
- extension.settings.read/write in the extension private namespace

### Authenticated Source
Standard Source plus source.auth for the selected source. Requires explicit user consent and explains that credentials are stored by NamiShelf and never exposed as global secrets. Cookie/session operations remain broker-managed.

### Restricted / Review
No network or source-auth capability until the user reviews and enables the source. Useful for imported/unverified extensions and troubleshooting.

Profiles are templates, not trust scores. Artifact provenance/signature status is displayed separately. A signed artifact is not automatically safe.

## 4. User experience requirements

Show requested, granted, denied, and effective capabilities before install/enable. Explain source and origin scope in ordinary language. Let the user inspect and override individual capabilities. Require explicit prompts for source authentication, additional origins, local-file picker, notifications if enabled, and other sensitive capabilities.

On update, show newly requested capabilities and require renewed consent; never silently expand. On revocation, cancel in-flight operations and invalidate related handles/cookies/tokens as appropriate. Provide per-extension disable/stop and a global extension kill switch. Maintain a local permission history without secret values.

## 5. Source authentication boundary

The extension may request login/session operations through a typed API. NamiShelf owns credential collection/storage, cookie jar, refresh, and logout. The extension receives only the minimum source-scoped response needed for its contract; it must not receive raw vault keys, other sources' cookies, provider tokens, sync keys, tracker credentials, or general-purpose secret-store access.

Browser-mediated login and CAPTCHA interaction require a separate UX/security design; do not assume Android WebView compatibility.

## 6. Capability profile versioning

Profiles carry a policy version. A profile upgrade is not permission expansion. New capabilities are denied until reviewed. A downgrade/revocation takes effect immediately for new operations and cancels active work where possible.

## 7. Audit events

Record permission request, grant, denial, override, revocation, profile change, extension update requesting new capabilities, and denied operation. Events contain identifiers and timestamps, not payloads or secret material.

## 8. Acceptance tests

Verify cross-source isolation, denied-method behavior, consent persistence, no silent expansion, immediate revocation, handle invalidation, settings namespace isolation, source-auth vault boundaries, profile migration, and malformed capability identifiers.

## 9. Owner input still needed

Confirm whether the proposed profile names and default grants match the intended user experience. No capability implementation should broaden these defaults without explicit policy review.
