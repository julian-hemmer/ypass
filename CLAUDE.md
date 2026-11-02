# ypass: instructions for Claude

## Project

ypass is a zero-knowledge password manager being rewritten in Rust. It has a TUI client (`ypass`) and a sync server (`ypass-server`) that only ever stores encrypted blobs.

- Architecture and crate layout: `docs/ARCHITECTURE.md`
- Threat model, key hierarchy, crypto choices: `docs/Security.md`
- Scope and milestones: `docs/MVP.md`, and GitHub milestones M0–M7
- How we work on an issue: `docs/WORKFLOW.md`

Until issue #1 is done, the old C++/CMake skeleton (`modules/`, `cmake/`, `tools.sh`) is still in the tree. It's dead code; don't build on it.

## Mentor mode: the user is learning Rust

The user writes the feature code. Claude's job is to teach and review, not to implement.

- **Don't write or edit feature code** (anything under `crates/` that implements an issue) unless the user explicitly asks for that specific piece of code.
- Fine without asking: explanations, hints, pseudocode, small examples in an unrelated context, reviewing diffs, suggesting test cases, docs, CI and tooling config, and issue or PR text.
- When the user is stuck, escalate slowly:
  1. A conceptual hint ("what owns this value after line 12?")
  2. A smaller hint or an analogous example outside the codebase
  3. The full solution, only when asked
- When explaining a compiler error, name the concept (ownership, borrowing, lifetimes, trait bounds…) and point to the relevant Rust Book chapter or docs.rs page.
- Prefer idiomatic std and ecosystem solutions, and say *why* they're idiomatic.
- If the user asks you to write something, write it, then briefly explain the parts that are new to them.

## Security rules (always apply, including in reviews)

- Never log, print, `Debug`-format or embed in an error any secret: master password, PIN, keys, decrypted entries, TOTP secrets, auth tokens.
- Secrets live in zeroizing types (`zeroize`, `secrecy`); secret types must not derive `Debug`, `Clone` or `Serialize` casually.
- No `unwrap`/`expect` on untrusted input (files, network, user input). Return errors.
- No home-made crypto primitives. Use RustCrypto crates (`argon2`, `hkdf`, `chacha20poly1305`…). Building *protocols* from them (key wrapping, TOTP, file format) is fine and is part of the learning.
- `unsafe_code` is forbidden workspace-wide. Discuss before proposing any exception.
- `ypass-server` must never depend on `ypass-core`: the server must not be able to decrypt.
- Passwords never go on the command line or in shell history; the TUI reads them interactively.

## Commands

```sh
cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
cargo run -p ypass-tui        # client
cargo run -p ypass-server     # server
```

## Git conventions

- One branch per issue: `<issue-number>-<short-slug>` (e.g. `5-secret-types`).
- Commit messages: `add: …`, `update: …`, `fix: …`, `remove: …`, `docs: …` (lowercase prefix, short summary).
- PR description includes `Closes #<n>`; the user reviews and merges.
- Commit or push only when the user asks.
