# Application Update and Release Channel Strategy

**Status:** Native DEB/Flatpak updates and a dedicated AppImage updater approved under ADR-022. Concrete updater technology, signing operations and UX remain deferred.

## 1. Update ownership by package format

| Package | Update owner | NamiShelf responsibility |
|---|---|---|
| DEB | APT / configured package repository | Detect/announce availability where feasible; link to package-manager workflow; never mutate system package state directly |
| Flatpak | Flatpak remote/update service | Respect Flatpak deployment and sandbox lifecycle; provide update information without self-modifying deployed files |
| AppImage | NamiShelf per-user updater | Check trusted release metadata, download, verify, stage, activate and support rollback |

The application must detect or be informed of its installation format without guessing from paths alone. If update ownership cannot be determined reliably, offer instructions rather than attempting a potentially destructive update.

## 2. AppImage update lifecycle

1. Check the configured release channel and retrieve signed release metadata.
2. Validate metadata authenticity, version policy, channel and supported architecture.
3. Download the complete artifact to a staging location.
4. Verify the artifact signature or equivalent authenticated provenance and cryptographic digest.
5. Check available disk space, installation permissions and whether another update is in progress.
6. Preserve the currently working AppImage and stage the candidate without touching user data.
7. Activate using a safe filesystem operation appropriate to the installation location.
8. Retain the previous version for rollback until the new version is confirmed usable according to a defined health criterion.
9. Report success/failure and clean stale staging artifacts conservatively.

No downloaded shell script or arbitrary update hook may be executed. Per-user AppImage updates must not require root privileges. System-wide installations may require manual administrator/package-manager action and should not be silently modified.

## 3. Trust and release provenance

- HTTPS protects transport but does not independently establish publisher authenticity.
- Release metadata and artifacts require cryptographic verification rooted in a trusted key or equivalent secure trust bootstrap.
- Key rotation and revocation must be planned before public release.
- Reject mismatched digest/signature, wrong architecture, wrong channel and malformed metadata.
- Downgrade should be rejected by default; provide a deliberate recovery procedure for a broken release.
- Do not embed a signing private key in the application or CI logs. Signing custody belongs to release infrastructure with restricted access.
- Record release version, channel, architecture, digest and verification outcome for diagnostics without exposing secrets.

## 4. Channels and compatibility

Separate stable and any future preview/testing channels. Channel switching must be explicit and visible. The initial release may expose only stable; additional channels are not implied.

Application binaries, extension packages and source/catalog metadata are distinct update domains:
- application updates use release signing and package-format ownership;
- extension updates use the extension repository authentication, compatibility and capability policy;
- catalog/metadata updates use source-specific validation and versioning.

Updating one domain must not silently change another domain's trust policy or rollback state.

## 5. User data and migration safety

Application replacement must preserve:
- SQLite database and migrations;
- permanent downloaded chapter/page artifacts and manifests;
- user configuration;
- backup archives;
- extension data/settings subject to their own compatibility rules.

A schema migration must be versioned, transactional where possible, recoverable and separately tested. Binary rollback does not automatically roll back a database schema; migration compatibility and backup/recovery policy must be designed explicitly.

## 6. Failure cases

Handle unavailable network, invalid signature, digest mismatch, interrupted download, insufficient space, read-only install directory, concurrent updater, activation failure, failed first launch, revoked signing key and unavailable previous version. Fail closed before activation when authenticity or integrity cannot be established.

## 7. Deferred M0/release engineering decisions

- AppImage updater technology and supported installation layouts.
- Release metadata schema and signing format.
- Trusted key bootstrap, custody, rotation and revocation.
- Stable-only initial channel versus preview channels.
- Rollback activation mechanism and health criterion.
- Update notification, consent and scheduling UX.
- Full versus delta downloads and offline behavior.
- Hosting/repository setup for DEB and Flatpak remotes.
- Compatibility policy for database migrations across rollback.

## 8. Validation scenarios

Before release, validate valid and invalid signatures, tampered artifacts, wrong architecture/channel, interrupted download, low disk space, read-only path, concurrent update, activation failure, rollback, no network, revoked key, user-data preservation and update behavior for each package format on its declared support matrix.

No implementation, build, dependency installation or test execution is authorized by this specification.
