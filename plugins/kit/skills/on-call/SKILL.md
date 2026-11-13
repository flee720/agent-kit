---
name: on-call
description: Fix a production error end to end, the way an on-call engineer would. Reproduce it first with the real failing input, write a failing test, fix the root cause, verify by driving the app, then hand to /kit:ship, which merges small fixes and sends big ones to the owner. Use for a Sentry or PostHog issue, an error log, a failing job or URL, for /kit:on-call, or from a scheduled error sweep.
argument-hint: <error link, issue id, log excerpt, or failing URL>
---

# On-call

The rule that makes unattended fixes safe: no fix without a reproduction. If you cannot make the error happen, you cannot prove you fixed it.

## 0. Read the repo's rules

Read the `## Agent workflow` section of the repo's `CLAUDE.md`, especially its production data policy. No policy means: you may read the failing record's inputs, and you never write to production.

## 1. Gather

Get the full error: message, stack trace, release or commit, the request or job input, timestamps, how many users or runs it hit. Use a connected Sentry or PostHog tool or their CLI if the session has one; otherwise work from what you were given and say what is missing.

Before starting, check open PRs and branches for this same error. One PR per root cause.

## 2. Reproduce

Recreate the failure in an isolated environment (this session, a local stack, a preview deploy, the simulator), using the same input that failed in production, as the data policy allows. Real input beats invented test data.

You must observe the same error, or the same wrong output, before you change any code. If you cannot:
- and the stack trace makes the defect certain, a unit test that fails on the current code counts as the reproduction;
- otherwise stop. Report "not reproduced" with what you tried, and do not ship a speculative fix.

## 3. Lock it in

Write a test (or a verify-skill scenario) that fails now for the reason the error happened. Keep it in the PR.

## 4. Fix the cause

Fix the root cause with the smallest change that does it, not the symptom where it surfaced. If the real fix is wide (many callers, a schema change, an architectural flaw), still write it, and expect `/kit:ship` to send it to the owner as high risk.

## 5. Prove it

- The new test passes, and so does the rest of the suite.
- The original reproduction from step 2 now succeeds.
- Drive the touched features with the repo's `verify-*` skill.

## 6. Ship

Hand off to `/kit:ship`. The ticket is: the error (task), what you found (context), and the reproduction plus the test (verification scenarios). Ship decides the risk tier and whether it merges.

Report for the owner: what broke, for whom, the cause in one sentence, the fix, the evidence, and whether it merged or waits for them.
