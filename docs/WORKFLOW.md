# Workflow

ypass is a learning project: the goal is to learn Rust **and** how to work well with Claude Code. The user writes the feature code; Claude mentors and reviews (rules in `CLAUDE.md`).

## Lifecycle of an issue

```
pick issue ─▶ /start-issue <n> ─▶ code + ask questions ─▶ fmt/clippy/test ─▶ PR ─▶ /code-review ─▶ fix ─▶ merge
```

1. **Pick** the next open issue in the current milestone (lowest number first; they're ordered).
2. **`/start-issue <n>`** in Claude Code. Claude reads the issue, creates the branch `<n>-<slug>`, explains the Rust concepts and Book chapters involved, checks your understanding, and proposes a step order. Issues labelled `refine-first` get their acceptance criteria rewritten first.
3. **Code it yourself.** When stuck, ask Claude. It hints first, and gives a full solution only if you ask. Good questions:
   - "Explain this compiler error. Which concept am I missing?"
   - "Is this idiomatic? What would an experienced Rustacean change?"
   - "Which test cases should I add for this function?"
4. **Check locally:**
   ```sh
   cargo fmt --all
   cargo clippy --workspace --all-targets -- -D warnings
   cargo test --workspace
   ```
5. **Open a PR** (`gh pr create`, with `Closes #<n>` in the body). Claude can draft the description.
6. **Review** with `/code-review`, fix the findings, and ask *why* for anything you don't understand.
7. **Merge** once CI is green (from M0 #3 on).

Commit style: `add: …`, `update: …`, `fix: …`, `remove: …`, `docs: …`.

## Labels

| Label | Meaning |
|---|---|
| `area:core` / `area:crypto` / `area:tui` / `area:server` / `area:proto` / `area:infra` | Which part of the code |
| `type:design` | Produces a design doc or decision, little or no code |
| `security` | Security-sensitive: review extra carefully |
| `refine-first` | Draft issue: refine its acceptance criteria before starting |
| `good first issue` | Small, good entry point |
| `backlog` | Not planned for the MVP |

## Learning Claude Code alongside the milestones

Each milestone introduces one or two Claude Code features, so the tooling grows with the project.

| Milestone | Claude Code feature to practise |
|---|---|
| M0 | `CLAUDE.md` (project instructions), **plan mode** (Shift+Tab) before non-trivial work, the `/start-issue` skill, memory ("remember that…") |
| M1 | **Learning** output style (`/config` → Output style), asking Claude to explain compiler errors, `/code-review` on every PR |
| M2 | Ask Claude for test cases while you write the implementation; ask it to research crate options (it can use subagents for that) |
| M3 | **Hooks** via `/update-config` (e.g. run clippy whenever Claude finishes a turn), `/simplify` on a finished PR |
| M4–M5 | `/security-review` on security-sensitive PRs; write your own second skill with the skill-creator |
| M6–M7 | Git worktrees for working on two branches in parallel; optionally the Claude GitHub Action for automatic PR review |

Rule of thumb for custom skills: when you catch yourself typing the same multi-step prompt a third time, turn it into a skill in `.claude/skills/<name>/SKILL.md`.
