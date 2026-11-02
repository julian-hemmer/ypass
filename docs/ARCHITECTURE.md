# Architecture

ypass is a Cargo workspace with four crates. The split exists so that **the compiler enforces the security boundaries**: the server can't decrypt because it doesn't even link the crypto code.

> Status: this is the target layout. The workspace itself is created in issue #1.

## Workspace layout

```
ypass/
├── Cargo.toml              # [workspace]: members, shared [workspace.dependencies], [workspace.lints]
├── rust-toolchain.toml     # pinned stable toolchain + rustfmt/clippy components
├── rustfmt.toml
├── crates/
│   ├── ypass-core/         # lib
│   ├── ypass-proto/        # lib
│   ├── ypass-tui/          # bin: `ypass`
│   └── ypass-server/       # bin: `ypass-server`
├── docs/
├── .github/                # CI workflows, issue & PR templates
├── .claude/                # Claude Code project skills
└── CLAUDE.md
```

| Crate | Kind | Responsibility | Must not |
|---|---|---|---|
| `ypass-core` | lib | Secret types, key derivation, AEAD, key hierarchy, vault data model, vault file format, password generator, TOTP | Touch the terminal or network |
| `ypass-proto` | lib | API request/response types (plain `serde` structs) shared by client and server | Contain logic or crypto |
| `ypass-tui` | bin | ratatui UI, config, clipboard, auto-lock, PIN unlock, HTTP sync client | Implement crypto itself (it calls `ypass-core`) |
| `ypass-server` | bin | Accounts, sessions, storing and serving encrypted vault blobs | Depend on `ypass-core` |

## Dependency rules

```
ypass-tui ──▶ ypass-core
    │
    └──────▶ ypass-proto ◀────── ypass-server
```

- `ypass-server` **never** depends on `ypass-core`. It only ever sees opaque bytes. It uses `argon2` directly to hash the client's auth key.
- `ypass-core` has no async runtime and no I/O beyond reading and writing the vault file. That keeps it fast to test.
- `ypass-proto` depends only on `serde` (plus `uuid`/`time` if needed).

## Data flow

**Local use (M1–M4)**
1. The TUI asks for the master password, reads the vault file and calls `ypass-core` to derive the keys and decrypt.
2. The decrypted `Vault` lives in memory inside zeroizing types while unlocked.
3. On every change the TUI asks `ypass-core` to re-encrypt and atomically rewrite the file.
4. On inactivity it zeroizes the decrypted vault. A PIN can re-unlock it during the same session (see `Security.md`).

**Sync (M5–M6)**
1. Prelogin: the client fetches the user's KDF parameters and salt from the server.
2. The client derives the master key, then the **auth key** (HKDF), and logs in with it. The server checks it against its stored Argon2id hash and returns a session token.
3. The client uploads or downloads the encrypted vault blob with a revision number. A stale revision gets `409 Conflict`, and the user picks which side to keep.

## Proposed crate choices

Each one is confirmed, or replaced, in the issue that first needs it.

| Need | Crate(s) |
|---|---|
| Password → key | `argon2` (Argon2id) |
| Subkeys | `hkdf` + `sha2` |
| Encryption | `chacha20poly1305` (XChaCha20-Poly1305); `aes-gcm` as the alternative |
| Secret memory | `zeroize`, `secrecy` |
| Randomness | `rand` (OS RNG via `getrandom`) |
| TOTP | `hmac` + `sha1` (TOTP written by hand, verified against RFC 6238 vectors) |
| Serialization | `serde`, `serde_json` (encrypted payload); hand-written binary header |
| IDs / time | `uuid`, `time` |
| Errors | `thiserror` (libs), `anyhow` (bins) |
| TUI | `ratatui`, `crossterm` |
| Clipboard | `arboard` (+ WSL fallback, see issue #24) |
| Config | `directories`, `toml` |
| Server | `tokio`, `axum`, `tower`/`tower-http`, `sqlx` (SQLite), `rustls`, `tracing` |
| HTTP client | `reqwest` with `rustls` |
| Testing | built-in tests, `proptest`, `cargo-fuzz` |
| Supply chain | `cargo-deny`, `cargo-audit` |
