---
name: ticket
description: Turn a feature request or bug report into a three-part ticket (task, context, verification scenarios) before any code is written. Use at the start of non-trivial work, for /kit:ticket, or when a request is vague about what "done" means.
argument-hint: <what to build or fix>
---

# Ticket

Front-load the engineering. A ticket that names the files, the constraints and the checks that prove "done" lets the implementer finish in one pass. A vague one burns tokens, guesses at intent, and leaves nobody sure what was built.

## The three sections

### 1. Task
One short paragraph, from the user's point of view.
- Bug: what happens now, what should happen, and the exact steps or input that trigger it.
- Feature: the before state and the after state.

### 2. Context
Everything the implementer would otherwise have to go and find:
- Files and symbols involved, with paths. Found by reading the repo, never guessed.
- Docs that govern this area: the repo's `CLAUDE.md` sections, planning docs, design notes, related PRs or commits.
- Constraints: conventions the repo enforces, tests or guards that will trip, behavior that must not change.
- Data: the real input that reproduces the bug or exercises the feature, and where it lives.

### 3. Verification scenarios
A numbered list. Every item must pass before the work counts as done, and each is observable:
- the exact user path to drive, through the repo's `verify-*` skill when it has one;
- the expected end state, including side effects (rows written, files produced, messages sent), not only what is on screen;
- failure paths and edge cases, not only the happy path;
- the repo's own checks (tests, typecheck, lint, guards) as listed in its `CLAUDE.md`.

If a scenario cannot be driven by an agent (it needs hardware, a paid account or a human), say so in the scenario and name what would prove it instead. Never drop it silently.

## Procedure

1. Read before asking. Answer every context question you can from the code, docs and git history.
2. Ask the user only what you cannot observe: product decisions, data you cannot reach. One batched round.
3. Present the ticket. Then implement against it if the user approves or the repo's instructions say to proceed on your own. Section 3 is the definition of done.
4. Keep the ticket where the repo keeps plans (for example `.planning/` in a GSD repo). Otherwise put it in the PR description, which `/kit:ship` reads.
