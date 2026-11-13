---
name: onboard-repo
description: Set up a repository for the shared agent workflow. Adds the Agent workflow section to CLAUDE.md, wires the plugins for local use, generates a verify skill that drives the real app (web app, device simulator, Shopify theme, CLI or service), and checks the repo has a CI gate. Use for /kit:onboard-repo, or when a repo has no verify skill.
---

# Onboard a repo

Every other kit skill reads two things from the repo: an `## Agent workflow` section in `CLAUDE.md`, and a `verify-<app>` skill with a feature map. This skill creates both, without disturbing what the repo already has.

## 0. Survey first

Read the repo's `CLAUDE.md`, `.claude/settings.json` (existing hooks, permissions), CI workflows, test setup and planning system. Respect them:
- If the repo routes changes through a process (GSD's `/gsd-quick`, for example), do this onboarding inside it.
- Merge into existing files. Never replace another tool's hooks, permissions or sections.

## 1. Agent workflow section

Add the template at the end of this file to the repo's `CLAUDE.md`, filled in from what you found. Every value comes from the repo:
- check commands: the ones CI and the repo docs actually run;
- the verify skill name (from step 3);
- merge policy: always start at `owner-review`. Only the owner moves it to `low-risk-auto`, and only once CI gates the main branch (step 4);
- never-auto-merge paths: migrations, auth, payments, deploy config, legal or public-surface guards, plus anything the repo's own docs call dangerous;
- production data policy: copy the repo's existing rule if it has one. Otherwise write "read-only, owner decides", and ask the owner.

## 2. Local plugin wiring

Merge these two blocks into `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "agent-kit": { "source": { "source": "github", "repo": "flee720/agent-kit" } },
    "cursor-plugins": { "source": { "source": "github", "repo": "cursor/plugins" } },
    "ponytail": { "source": { "source": "github", "repo": "DietrichGebert/ponytail" } }
  },
  "enabledPlugins": {
    "kit@agent-kit": true,
    "pstack@cursor-plugins": true,
    "ponytail@ponytail": true
  }
}
```

This covers local Claude Code only, and each machine still runs `claude plugin install <plugin>@<marketplace>` once. Cloud sessions never install plugins a repo declares; they get them from the environment's setup script (see the kit README).

## 3. Verify skill

Follow pstack's `create-verification-skill`. In Claude Code, find it with `find ~/.claude/plugins -path '*pstack*/skills/create-verification-skill/SKILL.md' | head -1`, read it, and follow it, writing to `.claude/skills/verify-<app>/` (its `.cursor/skills/` paths map to `.claude/skills/`). Before you start, read [`references/verification-adapters.md`](references/verification-adapters.md) for the project type: it says what "launch" and "drive" mean for a Shopify theme, a device simulator or a web app with a shared database.

The skill is not done until it has run once end to end, as pstack requires.

## 4. CI gate

Auto-merge is only as safe as the checks GitHub enforces. If the repo has no CI workflow running its checks on pull requests, add a minimal one. Then tell the owner the one setting only they can change: branch protection on the main branch, requiring that workflow to pass.

## 5. Report

What was added, the verify skill's first run and its evidence, the merge policy (starting at `owner-review`), and what the owner still has to do.

## Template: Agent workflow section

```markdown
## Agent workflow

Shared kit: github.com/flee720/agent-kit (`/kit:ticket`, `/kit:quality-pass`, `/kit:ship`, `/kit:on-call`).

- **Checks:** `<command>`, `<command>`
- **Verify skill:** `verify-<app>` (`.claude/skills/verify-<app>/`)
- **Merge policy:** owner-review   <!-- owner-review | low-risk-auto -->
- **Never auto-merge:** `<path or glob>`, `<path or glob>`
- **Production data:** <what agents may read, and that they never write>
- **Evidence:** `.agent-evidence/` on the PR branch, removed before merge
```
