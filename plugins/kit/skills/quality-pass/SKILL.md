---
name: quality-pass
description: Cut bloat from the current branch's changes without changing behavior. Runs the restraint (Ponytail), structure (thermo-nuclear) and prose (pstack unslop, no-comments) reviews in parallel, applies the findings that keep behavior identical, and re-runs the repo's checks. Use before opening a PR, for /kit:quality-pass, or when asked to simplify or debloat a change.
---

# Quality pass

Agents that write and merge a lot of code drift toward bloat: extra layers, defensive branches nobody asked for, narrating comments, files that only grow. This pass makes each change as small and plain as it can be while doing exactly the same thing.

## 1. Scope

- Base: the branch the PR targets (default `main`). Diff: `git diff $(git merge-base HEAD origin/<base>)...HEAD`, plus uncommitted changes.
- Read the repo's `CLAUDE.md`. Its conventions outrank every finding below.
- No diff, no pass. Say so and stop.

## 2. Review in parallel (read-only)

Start these as separate subagents in one message. Each gets the diff, the changed files in full, and the repo's `CLAUDE.md`. Each returns findings as text and edits nothing.

| Lens | Source | If unavailable |
|---|---|---|
| Restraint: unrequested abstractions, dependencies where the stdlib or platform does it, code that need not exist | The `ponytail:ponytail-review` skill (Ponytail plugin) | Skip, and say "Ponytail not installed" in the report |
| Structure: code-judo restructurings, files pushed past 1k lines, spaghetti conditionals | This plugin's `thermo-nuclear-review` skill | Always available |
| Prose: comments that narrate the code, AI tells in docs, PR text and comments | pstack's `no-comments` and `unslop` skills | Skip, and say "pstack not installed" |

The pstack skills cannot be called through the Skill tool. Find their files with `find ~/.claude/plugins -path '*pstack*/skills/<name>/SKILL.md' 2>/dev/null | head -1` and have the subagent read and follow that file.

## 3. Reconcile and apply

- Keep a finding only if behavior stays identical. A finding that reveals a bug is not a cleanup: list it separately for the user (or for `/kit:ship`) and leave the code as is.
- Drop findings that contradict the repo's conventions, and findings that conflict with each other's stronger option.
- Prefer deletion over rearrangement. Two findings that touch the same lines: take the one that leaves less code.
- Apply what survives, in small commits, or one commit if the change is small.

## 4. Prove nothing broke

- Re-run the repo's checks from its `CLAUDE.md` (tests, typecheck, lint, guards).
- If a cleanup touched code on a user path, re-drive that feature with the repo's `verify-*` skill.
- A check that fails after cleanup: revert that cleanup, do not "fix forward".

## 5. Report

Short and plain: lines added and removed by the pass, what was cut and why in a line each, findings declined and why, bugs found (not fixed here), and any lens that was skipped.
