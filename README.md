# YPASS

A zero-knowledge password manager: a terminal (TUI) client with a local encrypted vault, and a sync server that only ever stores ciphertext.

> **Status:** being rewritten in Rust as a learning project. Nothing works yet; see the [milestones](https://github.com/julian-hemmer/ypass/milestones) for progress.

## Planned features (MVP)

- Encrypted local vault (Argon2id + XChaCha20-Poly1305), unlocked with a master password
- PIN quick-unlock within a session, auto-lock on inactivity
- List, search, add, edit and delete entries in a TUI
- Password generator, TOTP codes
- Clipboard copy with automatic clearing
- Optional sync through a server that can't read your data

## Documentation

- [MVP & milestones](docs/MVP.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Security design](docs/Security.md)
- [Development workflow](docs/WORKFLOW.md)
