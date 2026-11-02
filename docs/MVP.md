# ypass MVP

The MVP is a TUI password manager with a local encrypted vault, synced through a zero-knowledge server. It covers milestones **M0–M6**. M7 hardens it for a first release.

Each item below is tracked as a GitHub issue under its milestone: https://github.com/julian-hemmer/ypass/milestones

| Milestone | Delivers | Original MVP items covered |
|---|---|---|
| **M0 Foundations** | Cargo workspace, tooling, CI | – |
| **M1 Crypto core** | Secret types, Argon2id + HKDF, AEAD, key hierarchy, password generator, TOTP | Password generator, TOTP (logic) |
| **M2 Vault & file format** | Entry/vault model, CRUD + search API, encrypted file format, atomic saves | Local encrypted vault |
| **M3 TUI client** | Create/unlock vault, list, search, add, edit, delete | Unlock, list, search, add, edit, delete |
| **M4 Client security** | Clipboard auto-clear, auto-lock, PIN quick-unlock, TOTP view, process hardening | Clipboard support, TOTP (UI) |
| **M5 Server** | API design, accounts, login, sessions, TLS, rate limiting | Registration, authentication, sessions |
| **M6 Sync** | Upload/download encrypted vault, conflict handling, end-to-end test | Store / upload / download encrypted vault |
| **M7 Hardening & release** | Supply-chain checks, fuzzing, security review, release binaries, user docs | – |

## Not in MVP (backlog)

Tracked as issues with the `backlog` label and no milestone:

- OPAQUE authentication (replaces the derived auth key)
- Persistent PIN via OS keyring / TPM
- Import / export (Bitwarden, KeePass)
- Desktop GUI
- Browser extension
- Mobile application
- Password sharing and multiple users per vault
- Passkeys
