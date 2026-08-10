# milk-run

Scheduled **milk runs** for GitHub Agentic Workflows: Copilot does one small, well-scoped repo task and opens a **draft PR** for humans. Skip if that agent already has an open PR.

This repository publishes the reusable wrapper. Task agents and skills live in consuming repos.

## Install tooling

```bash
gh extension install github/gh-aw
```

## Import the shared component

In a consumer workflow under `.github/workflows/`:

```yaml
---
on:
  schedule:
    - cron: "0 9 * * *"
  workflow_dispatch:
engine: copilot
permissions:
  contents: read
  pull-requests: read
  models: read
  copilot-requests: write
imports:
  - uses: autotelic/milk-run/.github/workflows/shared/milk-run.md@v0.1.0
    with:
      agentName: my-task
---

# My task

Describe the one job this agent should do each run.
```

Prefer a tag or commit SHA over a moving branch.

`permissions` are **not** merged from the import — the consumer must declare them (see above). Org Copilot CLI billing requires the org policy “Allow use of Copilot CLI billed to the organization.” Otherwise set repo secret `COPILOT_GITHUB_TOKEN`.

## What the wrapper enforces

| Concern | Behavior |
|---------|----------|
| Concurrency | Skips when an open PR title contains `[milk-run/<agentName>]` |
| PR shape | Draft only; label `milk-run`; title prefix `[milk-run/<agentName>] ` |
| Empty work | Agent should `noop`; `if-no-changes: ignore` avoids junk PRs |
| Safety | No merge/force-push/secrets; smallest useful change; one concern per PR |

## Local example / smoke test

[`.github/workflows/milk-run-noop-demo.md`](.github/workflows/milk-run-noop-demo.md) imports the shared file in-repo and compiles to a lockfile GitHub Actions can dispatch.

```bash
gh aw compile .github/workflows/milk-run-noop-demo.md
gh aw run milk-run-noop-demo
```

Expect a green run that calls **noop** and opens no PR.

## Versioning

- Tag releases as `v0.1.0`, `v0.2.0`, …
- Consumers should pin `@v0.1.0` (immutable) or a full commit SHA
- Moving majors (`@v0`) are optional convenience; recompile after `gh aw` cache updates when bumping
