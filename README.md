# agent-kit

One agent workflow shared by every repo: web apps, device panels and the Shopify theme. Agents prove their changes by using the real app, keep the code lean, get reviewed by other models, and merge on their own only when a change is small and low risk. Everything else waits for the owner.

Based on YourAverageTechBro's "How to become a 10x AI-native engineer" series, adapted to Claude Code.

## What's in it

| Skill | Use it for | Series |
|---|---|---|
| `/kit:ticket` | Turn a request into task, context and verification scenarios before writing code | Ep 6 |
| `/kit:ship` | Checks, then drive the app for evidence, quality pass, two independent reviews and a risk tier, then a PR that merges or waits for the owner | Ep 1, 2 |
| `/kit:on-call` | Production error: reproduce with the real input, failing test, root-cause fix, verify, ship | Ep 4 |
| `/kit:quality-pass` | Cut bloat from a branch without changing behavior (Ponytail, thermo-nuclear, pstack unslop / no-comments) | Ep 5 |
| `/kit:onboard-repo` | Set a repo up: `Agent workflow` section in `CLAUDE.md`, a `verify-<app>` skill, a CI gate | — |
| `/kit:thermo-nuclear-review` | Strict maintainability review (vendored from Cursor, MIT) | Ep 5 |

The kit relies on two third-party plugins:
- **[pstack](https://github.com/cursor/plugins/tree/main/pstack)** by Lauren Tan. Its `create-verification-skill` and `maintain-verification-skill` build and maintain each repo's verify skill.
- **[Ponytail](https://github.com/DietrichGebert/ponytail)** by Dietrich Gebert: restraint rules and `ponytail-review`.

The kit works without them, and skips their steps with a note.

## Install

### Cloud sessions (claude.ai/code)

Cloud sessions do not install plugins that a repo declares in `.claude/settings.json`. Install them from the cloud environment's setup script instead, so every session in that environment has them:

```bash
claude plugin marketplace add cursor/plugins && claude plugin install pstack@cursor-plugins || true
claude plugin marketplace add DietrichGebert/ponytail && claude plugin install ponytail@ponytail || true
claude plugin marketplace add flee720/agent-kit && claude plugin install kit@agent-kit || true
```

A setup script cannot fetch a private repository, so the last line only works while this repo is public. Each `|| true` keeps a failed install from stopping the session.

### Local Claude Code (terminal, desktop)

Once per machine:

```bash
claude plugin marketplace add flee720/agent-kit && claude plugin install kit@agent-kit
```

Run the same for `cursor/plugins` / `pstack@cursor-plugins` and `DietrichGebert/ponytail` / `ponytail@ponytail`. `/kit:onboard-repo` also records all three in the repo's `.claude/settings.json`, so the repo documents what it expects.

## Using it in a repo

1. Run `/kit:onboard-repo` once. It writes the `Agent workflow` section, builds the verify skill and checks for CI.
2. For each piece of work: `/kit:ticket`, build it, then `/kit:ship`.
3. For production errors: `/kit:on-call <issue>`, by hand or from a scheduled sweep.
4. Weekly: pstack's `maintain-verification-skill`, so the feature map keeps matching the app.

## Safety defaults

- Every repo starts on merge policy `owner-review`. Only the owner switches it to `low-risk-auto`, after branch protection makes CI required on the main branch.
- Auto-merge uses GitHub's own auto-merge, so nothing merges until the required checks pass.
- Low risk means a small, proven change that touches no migration, auth, payment, secret, deploy, dependency or legal or public-surface path, with both reviewers approving.
- Production data is read-only unless the repo's `CLAUDE.md` says otherwise.

## Third-party code

See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
