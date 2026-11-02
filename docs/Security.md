# Security

ypass is **zero-knowledge**: secrets are encrypted and decrypted only on the client. The server and the database only ever hold ciphertext.

## Goals

- **Server**: knows nothing it could use to read a vault. No plaintext entries, no master password, no vault key.
- **Database**: never stores an unencrypted blob or a password in plain text.
- **Network**: every connection uses TLS. An attacker who intercepts traffic sees only TLS-wrapped ciphertext.
- **Client**:
  - The local vault file is always encrypted, including the copy fetched from the server.
  - The vault is decrypted only after the master password (or, inside a live session, the PIN) is given.
  - Unlock attempts are slowed down so they can't be brute-forced quickly.
  - Inactivity locks the vault automatically.
  - A copied password is cleared from the clipboard after a timeout.
  - No feature leaks secrets into shell history: passwords are never command-line arguments.

## Threat model

| Attacker | Has | Must not get | Mitigation |
|---|---|---|---|
| Network eavesdropper | Traffic | Anything readable | TLS (rustls); payloads are already encrypted |
| Compromised server / DB dump | Encrypted vaults, auth-key hashes, usernames | Vault contents | Client-side encryption; auth key is independent of the vault keys (HKDF); Argon2id makes offline guessing expensive |
| Thief of the vault file | The file | Vault contents | Argon2id(master password) with strong parameters; AEAD detects tampering |
| Someone at an unlocked, unattended machine | Physical access | Long-lived access | Auto-lock on inactivity; PIN limited to 3 attempts, then master password |
| Shoulder surfer / shell history | Screen, history | Passwords | Hidden input, masked display, no secrets in argv |

**Out of scope**: malware running as the user (keyloggers, memory dumps of a live process), a compromised OS, and hardware attacks. If an attacker controls the running client, they can read what it reads.

**By design, a forgotten master password cannot be recovered.** No one can decrypt the vault without it, including the server operator.

## Key hierarchy

```
master password ──Argon2id(salt, params)──▶ master key (32 B)
                                               │
                     ┌──── HKDF-SHA256 "kek" ──┴── HKDF-SHA256 "auth" ────┐
                     ▼                                                     ▼
                    KEK                                                auth key
                     │ wraps (AEAD)                                        │ sent at login (over TLS)
                     ▼                                                     ▼
          vault key (random 32 B) ──AEAD──▶ vault payload     server stores Argon2id(auth key)
```

- **Master key**: derived from the master password with Argon2id. The salt and parameters are stored in the vault header (and on the server for new devices), so they can be raised later.
- **KEK and auth key**: independent HKDF subkeys. Knowing the auth key (or its hash) reveals nothing about the KEK.
- **Vault key**: random, generated once per vault, and stored only wrapped by the KEK. Changing the master password just re-wraps it; entries aren't re-encrypted.
- **Server-side hashing**: the server runs Argon2id over the auth key again, so a DB dump can't be replayed as a login.

## PIN quick-unlock

The original design used a PIN to unlock the vault. A short PIN can't protect a file that's been copied off the machine: an attacker can try every PIN offline, however slow the KDF is. So:

- The **master password** protects the vault at rest and is required at every start of the app.
- After unlocking, the user may set a **session PIN**. On auto-lock, the vault key is wrapped with `Argon2id(PIN, random session salt)` and kept **in RAM only**. The plaintext vault key and decrypted entries are zeroized.
- Entering the PIN unwraps the vault key. **After 3 wrong PINs** the wrapped key is wiped and the master password is required.
- Quitting the app wipes everything. Nothing PIN-related is ever written to disk.
- Limit: an attacker who can read the live process memory can brute-force the PIN offline. That attacker is already out of scope (see above).

Persisting a PIN across restarts (OS keyring / TPM) is in the backlog.

## Crypto choices

| Purpose | Choice | Notes |
|---|---|---|
| Password → key | Argon2id | Parameters calibrated to about 0.5–1 s on the user's machine and stored in the header |
| Subkeys | HKDF-SHA256 | Distinct `info` strings per purpose |
| Encryption | XChaCha20-Poly1305 | 192-bit random nonces are safe; AES-256-GCM is the alternative, decided in the AEAD issue |
| Randomness | OS CSPRNG (`getrandom` via `rand`) | Never a seeded or userspace RNG for secrets |
| Transport | TLS 1.3 via rustls | |
| Header integrity | Header bytes are passed as AEAD associated data | Tampering with KDF params or version breaks decryption |

Implementation rules (also enforced in `CLAUDE.md`):

- Use audited RustCrypto implementations; never hand-roll primitives.
- Keep secrets in zeroizing types; no `Debug`/`Display`/logging of secrets.
- No `unsafe` code (workspace lint).

## What the server knows

| Data | Stored as |
|---|---|
| Username | Plain (needed for lookup) |
| KDF salt + parameters | Plain (needed by new devices, not secret) |
| Auth key | Argon2id hash |
| Vault | Encrypted blob + revision number |
| Session tokens | Hash + expiry |
| Metadata | Account creation time, last sync time, blob size |

Known leak to address in the API design (issue #29): an unauthenticated prelogin endpoint can reveal whether a username exists.
