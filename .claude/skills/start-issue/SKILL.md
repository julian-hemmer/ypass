---
name: start-issue
description: Start work on a ypass GitHub issue in mentor mode. Reads the issue, creates the branch, explains the Rust concepts involved and plans the steps with the user, without writing the implementation.
argument-hint: <issue-number>
disable-model-invocation: true
---

# Start issue #$ARGUMENTS

The user is learning Rust and writes the feature code themselves (see CLAUDE.md, "Mentor mode"). Your job here is to set them up to succeed, **not** to implement the issue.

## Steps

1. **Read the issue**: `gh issue view $ARGUMENTS --comments`. If it doesn't exist or is closed, say so and stop.

2. **Refine first if needed.** If the issue has the `refine-first` label, its acceptance criteria are a draft. Discuss them with the user, propose a rewritten body (same sections as `.github/ISSUE_TEMPLATE/task.md`), and only run `gh issue edit $ARGUMENTS --body-file …` after the user confirms. Remove the label once refined (`gh issue edit $ARGUMENTS --remove-label refine-first`).

3. **Prepare the branch.**
   - `git status --porcelain`: if the tree isn't clean, stop and ask the user what to do with their changes.
   - `git switch main && git pull --ff-only`
   - `git switch -c <number>-<short-slug>` (slug: 2–4 words from the title, kebab-case).

4. **Restate the goal** in 2–3 sentences in your own words, then list the acceptance criteria as a checklist.

5. **Concepts to learn.** For each Rust concept the issue exercises, give:
   - one sentence on what it is and why it matters here
   - the Rust Book chapter (by number and title) or the relevant docs.rs page for crates

   Also point to the existing code the user will build on (paths), if any.

6. **Check understanding.** Ask 1–3 short questions about design choices or concepts (e.g. "Should `Key` implement `Clone`? Why or why not?"). Wait for the answers and discuss them before moving on.

7. **Propose a step order**, smallest compiling step first (types → functions → tests → integration). Each step should end with `cargo check` or `cargo test` passing. Describe *what* to do in each step, not the code.

8. Close with: "Ask me whenever you're stuck. I'll hint first. When you're done, run fmt/clippy/test and we'll open the PR and run `/code-review`."

## Don't

- Don't write the implementation, or a code skeleton of it, unless the user explicitly asks.
- Don't commit or push.
- Don't change the issue text without confirmation.
