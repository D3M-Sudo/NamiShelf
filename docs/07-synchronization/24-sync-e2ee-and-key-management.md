# NamiShelf — Sync E2EE and Key Management Design

**Status:** Threat-boundary and key-management design; owner decisions ADR-024 supersede the historical-ciphertext statements below. Cryptographic suite not selected
**Date:** 2026-11-03
**Decision basis:** ADR-006 — E2EE mandatory for release 1

## 1. Security objective

All user synchronization payloads must be encrypted on the client before upload. The provider adapter must store and transport ciphertext and must not receive plaintext domain records or content-encryption keys. TLS remains mandatory but is an additional transport control, not the E2EE boundary.

E2EE protects payload confidentiality from the storage provider and passive infrastructure operators. It does not hide all metadata: provider identifiers, object sizes, timestamps, access patterns, account identity, and network metadata may remain observable unless the protocol deliberately pads or minimizes them.

## 2. Threat model

### In scope
- Compromise or malicious operation of the storage provider.
- Disclosure of stored objects, backups of provider storage, or provider API credentials.
- Network observation and interception (TLS remains required).
- A lost or stolen device after local key material is protected by the OS credential service.
- Malicious or compromised extension code attempting to obtain sync data or keys.
- Replay, rollback, object substitution, and concurrent writes by stale devices.

### Not automatically solved by E2EE
- Malware or a hostile user with control of an unlocked endpoint.
- A compromised NamiShelf process or compromised OS account.
- Metadata leakage and traffic analysis.
- Availability attacks, deletion by provider, quota exhaustion, or account lockout.
- Malicious data deliberately authored by another already-authorized device.

## 3. Key hierarchy — candidate model

Use a random account-level root secret generated client-side. Derive or wrap separate purpose-specific keys for payload encryption, object authentication, and any future device-management metadata. Do not reuse the backup encryption password or sync provider token as key material.

Each device receives the minimum key material required to decrypt and produce valid sync envelopes. Prefer a design where a device-specific wrapping key protects the account root secret locally, with the wrapping key stored through the OS credential service. The exact hierarchy, KDF, AEAD, nonce construction, and key identifiers require cryptographic review before implementation.

Never derive encryption keys from a GitHub token, WebDAV password, S3 secret, device name, email address, or other low-entropy identifier.

## 4. Device enrollment

Enrollment must be an explicit user-authorized operation. A new device must not gain access merely by obtaining provider credentials.

Candidate enrollment flow:
1. Existing trusted device generates a short-lived enrollment authorization.
2. New device proves possession of an approved transfer/recovery secret or completes an authenticated device-to-device transfer.
3. Account root secret is transferred only inside an authenticated encrypted channel or via an explicit recovery package.
4. New device creates a device-specific local wrapping key and stores it in the OS credential service.
5. Device identity and key version are registered in the encrypted account state.
6. User receives a clear confirmation and can revoke the device later.

Do not transmit the account root secret through ordinary provider objects in plaintext, URLs, logs, or QR payloads without a separately reviewed secure protocol.

## 5. Owner-approved recovery model

The owner selected **both** recovery paths: a user-held recovery key and authenticated transfer from an already-authorized device. They are complementary mechanisms. Provider credentials must never substitute for either path.

### User-held recovery key

The design must generate high-entropy recovery material locally, display it only through a deliberate enrollment/recovery flow, require confirmation that the user has retained it, and explain that losing it together with all trusted devices makes the encrypted remote data unrecoverable. Define a safe encoding and verification procedure; do not log or sync the recovery secret.

### Authorized-device transfer

Transfer must be explicitly approved on a trusted device and authenticated end-to-end to the new device. Use a short-lived, one-time enrollment authorization and a secure local/network channel. Provider storage must not relay plaintext key material.

## 5A. Recovery choices considered

| Model | Benefits | Risks / operational cost |
|---|---|---|
| User-held recovery key | No NamiShelf server; provider-independent | User can lose it; UX and secure storage/printing guidance are critical |
| Existing-device transfer only | Simple mental model; no recovery secret to manage | All devices lost means data may become unrecoverable; requires devices online together |
| Both recovery key and device transfer | Resilient and supports new-device onboarding | More UX, more key lifecycle states, greater test burden |
| Provider-assisted recovery | Convenient | Conflicts with strict E2EE unless provider never learns the secret; requires a separate trust design |

Recovery must not silently downgrade encryption or allow provider credentials alone to decrypt data. If all authorized devices and recovery material are lost, explain whether the remote ciphertext is permanently unrecoverable.

## 6. Owner-approved revocation behavior

Revoking a device triggers key rotation for future synchronized data. The revoked device must not receive the new key version. Rotation must be coordinated through authenticated account state and cannot rely solely on provider ETags or Git history.

This does not retroactively erase plaintext or old ciphertext already copied by the revoked device. Under ADR-024, historical ciphertext must be re-encrypted for remaining authorized devices under the new key version, then deletion of superseded provider objects must be requested where supported. Provider backups/versioning/replicas may retain old ciphertext; deletion is best-effort and must not be represented as guaranteed erasure.

## 7. Rotation and revocation engineering requirements

- Support device revocation and define whether it triggers account-key rotation.
- A revoked device must not decrypt future payloads after the rotation boundary.
- Implement resumable, idempotent historical re-encryption for remaining authorized devices and best-effort provider deletion under ADR-024.
- Define recovery after a device is offline across multiple key rotations.
- Prevent stale devices from overwriting newer key metadata or restoring revoked device state.
- Keep an auditable local device list without exposing private key material.

## 8. Envelope and anti-replay requirements

Each encrypted envelope should authenticate protocol version, payload type, account scope, schema version, key version, object identity, and relevant causal metadata as associated data. Nonces must follow the selected AEAD construction's uniqueness requirements.

Use authenticated sequence/version metadata to detect replay and rollback. Provider ETags or Git commit ancestry may assist transport concurrency but must not be treated as cryptographic freshness proof or the canonical conflict model.

## 9. Credential separation

- Provider access tokens authenticate storage operations only; they do not decrypt payloads.
- E2EE root/key material is separate from provider credentials.
- OS credential storage is mandatory; no plaintext fallback.
- Extensions receive neither provider tokens nor E2EE keys.
- Ordinary backup files exclude sync keys and provider credentials.
- Logs, crash reports, analytics, process arguments, environment variables, and diagnostics must redact secrets.

## 10. Open security decisions

1. Select the key hierarchy and cryptographic suite only after a dedicated threat-model and cryptographic review.
2. Recovery model is owner-approved: both user-held recovery key and authenticated authorized-device transfer (ADR-007). Define the concrete protocol.
3. Device revocation triggers key rotation for future data (ADR-007); historical re-encryption and best-effort deletion are required by ADR-024.
4. Define metadata minimization and whether object sizes/timestamps require padding or indirection.
5. Define account identity and device enrollment without a NamiShelf-operated service.
6. Define export/transfer procedure when changing self-hosted providers.

## 11. Verification gates before implementation

- Independent review of the cryptographic design and protocol state machine.
- Known-answer tests and test vectors for all selected primitives.
- Tests for nonce uniqueness, tampering, replay, rollback, key-version mismatch, and malformed KDF parameters.
- Device enrollment/revocation/recovery tests across offline and concurrent devices.
- Secret-leak tests for logs, crash reports, IPC, process arguments, and ordinary backups.
- Provider-side inspection confirms only ciphertext and approved minimal metadata are stored.
- Loss-of-all-devices behavior is tested and documented.

No cryptographic primitive, library, or key hierarchy is selected by this document. No implementation or security test has been run.

## 12. Decision reconciliation

The following historical decision statements in this document are superseded:
- Recovery supports both a user-held recovery key and authenticated transfer from an already-authorized device (ADR-007); neither is an unresolved owner choice.
- Revocation rotates keys for future data (ADR-007).
- Historical synchronized ciphertext is re-encrypted for remaining authorized devices under the new key version, followed by best-effort provider deletion where supported (ADR-024). Provider backups/versioning/replicas may retain old ciphertext, and a revoked device may already possess copies.

The cryptographic suite/key hierarchy, causal model, metadata minimization, provider conformance and recovery protocol remain engineering/security gates before sync implementation. No cryptographic primitive is selected here.
