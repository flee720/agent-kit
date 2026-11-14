---
name: ship
description: Take a finished change to a PR with proof it works, then merge or hand it to the owner by risk. Runs the repo's checks, drives the changed features with the repo's verify skill, updates the feature map, runs the quality pass, gets two independent reviews on different models, and classifies the risk. Use when work is done and ready for a PR, for /kit:ship, or at the end of an on-call fix.
---

# Ship

A change merges on evidence, not on someone reading every line: the agent proves the behavior by using the app, other models review it, and only small, low-risk changes may merge without the owner.

## 0. Read the repo's rules

Find the `## Agent workflow` section in the repo's `CLAUDE.md`. It names the check commands, the verify skill, the merge policy and the paths that never auto-merge. No section means: run whatever checks the repo documents, and the merge policy is `owner-review`.

The repo's own process wins over this skill (for example a GSD repo routes work through its `/gsd-*` commands). Run these steps inside it, not around it.

## 1. Checks

Run every check command the repo lists. Fix failures and re-run until green. A check you cannot run here (missing credentials, hardware) is reported as not run, with the reason. It is never reported as passing.

## 2. Verify by using the app

- Run the repo's `verify-*` skill on every feature the diff touches, using its feature map to find them. No verify skill: say so, drive what you can by hand with the same standard, and recommend `/kit:onboard-repo`.
- Prove the ticket's verification scenarios one by one. For a bug, capture the broken "before" as well as the fixed "after".
- Evidence is what a reviewer can check without rerunning anything: screenshots, terminal transcripts, response bodies, rows written. Note where each piece came from.
- If the change adds or alters a user-facing feature, update that feature's file in the verify skill's feature map (or add one) in this same PR, and drive the new journey once.

## 3. Quality pass

Run `/kit:quality-pass`. Then repeat step 1 if it changed anything.

## 4. Independent review

Spawn two reviewers in one message, as `general-purpose` subagents on two different models (for example `model: "opus"` and `model: "sonnet"`; prefer models other than your own). Each one gets [`references/reviewer-brief.md`](references/reviewer-brief.md) followed by the base branch, the diff, the ticket, and the evidence.

If a `codex` CLI is on the PATH and the repo uses it, one reviewer may be Codex instead (`codex exec` with the same brief). A reviewer from a different model family catches different mistakes.

Fix every blocker and every finding you agree with. Re-run checks and evidence for anything that changed, then send the new diff back to the reviewer who raised it. After two rounds without both approving, stop and hand it to the owner with the open disagreement stated plainly.

## 5. Risk tier

Low risk only if all of these hold, otherwise high:
- a bug fix or small feature with a clear before and after, proven in step 2;
- small: a few files and roughly 200 changed lines or fewer;
- touches no path the repo lists under never auto-merge, and none of: database migrations or schema, auth or permissions, payments, secrets or env config, CI or deploy config, dependency changes, anything the repo calls a legal or public-surface boundary;
- both reviewers approved.

A wide or architectural fix is high risk even when it is correct.

## 6. PR

Open the PR with:
- the ticket's task, in two or three sentences;
- each verification scenario with pass / fail / not run, and its evidence;
- the reviewers, their models and their verdicts;
- the risk tier and the reason for it.

Screenshots: commit them under `.agent-evidence/` on the PR branch and link them by commit-SHA permalink, then delete the folder in a final commit so a squash merge keeps them out of the main branch. Use the repo's own evidence convention instead if its verify skill names one.

## 7. Merge or hand off

- Merge policy `low-risk-auto`, tier low, both reviewers approved, all checks green: turn on GitHub auto-merge (squash), so GitHub merges only after the repo's required CI checks pass. If this session's own rules do not let you merge, stop at a ready PR and say so.
- Anything else: leave the PR open, request the owner's review, and say in three lines what they should look at first.

Report in plain words: what shipped or is waiting, the evidence, and what is still unproven.
