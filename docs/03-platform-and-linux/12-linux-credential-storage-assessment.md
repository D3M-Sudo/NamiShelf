# NamiShelf — Linux Credential Storage Assessment

**Status:** Static source review; no credential commands executed. Owner policy is fixed by ADR-023.  
**Reviewed candidate:** `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Date:** 2026-11-03

## 1. Executive assessment

The candidate has a Linux credential backend using the freedesktop Secret Service through `secret-tool`. It deliberately has no plaintext preferences/file fallback. This is a sound security direction, but Linux native integration is not sufficiently evidenced by the inspected tests, and Flatpak/session availability must be validated.

## 2. Observed design

- `DesktopSyncSecureStore` stores encrypted records under the desktop config directory.
- `DesktopRecordCipher` uses AES-GCM with a random 256-bit wrapping key.
- The wrapping key is stored through `CredentialBackend`, not beside ciphertext.
- `PlatformCredentialBackend` maps Linux operations to `secret-tool store`, `lookup`, and `clear`.
- Credential namespaces separate tracker, app-lock, and sync secrets.
- The encrypted file store uses per-record associated data, bounded record size, file locking, and atomic replacement.
- The backend fails closed if the credential command is unavailable; no plaintext fallback is implemented.

## 3. Positive controls

- No plaintext fallback.
- AES/GCM authenticated encryption for persisted sync records.
- Wrapping key stored separately in OS credential service.
- Loss of wrapping key does not silently generate a new key over existing ciphertext.
- Secret values are redacted from several `toString` paths and temporary character/byte buffers are cleared where implemented.
- The Linux command interface uses argument arrays rather than shell command concatenation and has bounded process output/timeouts.

## 4. Evidence gaps and risks

### CRED-01 — No native Linux round-trip evidence in the inspected desktop sync test

The inspected `DesktopSyncSecureStoreTest` performs native backend round-trip only on Windows and macOS; Linux returns before executing that test. `PlatformCredentialBackendTest` exercises Linux command selection using a recording fake runner, not a real Secret Service provider.

**Impact:** The existence of `secret-tool` calls does not prove reliable operation on Linux Mint, GNOME Keyring, KWallet, headless sessions, or Flatpak.

### CRED-02 — Secret Service availability is a runtime dependency

The backend requires `secret-tool` and a functioning Secret Service provider/session. It intentionally fails closed if unavailable. Flatpak D-Bus permissions and provider availability may differ from a host DEB/AppImage launch.

**Impact:** Sync authorization and other features using this backend may be unavailable in minimal desktop sessions or sandboxed packages unless the credential service is reachable.

### CRED-03 — Wrapping-key recovery is intentionally strict

The wrapping key identity is derived from the canonical secure-store directory path. Moving the application data directory without migrating the matching OS credential item makes existing ciphertext unreadable. The code avoids silently regenerating a key when encrypted records already exist.

**Impact:** Backup/restore, profile relocation, migration between package formats, and reinstall flows must preserve or explicitly reauthorize credentials. Do not copy encrypted records as if they were self-contained.

## 5. NamiShelf requirements

1. Secret Service is the only persistent credential backend; fail-closed with no fallback is owner-approved under ADR-023. No plaintext, file, app-encrypted, database, environment-variable or session-memory fallback is permitted.
2. Credential-dependent operations remain unavailable when Secret Service is absent, locked or inaccessible; unrelated local features remain usable.
3. Validate real Linux Mint 22.3 sessions with the supported Secret Service provider(s), including locked/unavailable/error states.
4. Validate Flatpak D-Bus/portal access and DEB/AppImage behavior independently at their packaging gates.
5. Define wrapping-key migration when config paths or package formats change without weakening the fail-closed policy.
6. Keep secrets out of backup files, extension IPC, logs, diagnostics and process arguments.
7. Add Linux native integration tests in a controlled environment; fake command tests are not a substitute.

## 6. Recommendation

Secret Service is the sole persistent credential backend under ADR-023, with fail-closed behavior and no fallback. Treat provider availability and Flatpak access as release gates. Prefer a clear unavailable-state UX over weakening storage guarantees.

## 7. Source references

- [DesktopCredentialStore.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/platform/DesktopCredentialStore.kt)
- [DesktopSyncSecureStore.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/sync/DesktopSyncSecureStore.kt)
- [DesktopSyncSecureStoreTest.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/test/kotlin/mihon/desktop/sync/DesktopSyncSecureStoreTest.kt)
- [PlatformCredentialBackendTest.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/test/kotlin/mihon/desktop/platform/PlatformCredentialBackendTest.kt)
