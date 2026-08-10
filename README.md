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

`permissions` are **not** merged from the import — the consumer must declare them (see above), including `copilot-requests: write` for org-billed Copilot.

## Auth and org billing

Milk runs use the Copilot engine. In an **organization-owned** repo, prefer billing via the Actions `GITHUB_TOKEN` (no PAT to rotate).

1. An org owner opens **Settings → Copilot → Policies** (e.g. [autotelic Copilot policies](https://github.com/organizations/autotelic/settings/copilot/policies)).
2. Enable **Copilot CLI**.
3. Enable **Allow use of Copilot CLI billed to the organization**.

If Copilot CLI is already enabled, the billing policy is often on by default.

Consumer workflows must include `copilot-requests: write` under `permissions` (already shown in the import example). With that permission set, `COPILOT_GITHUB_TOKEN` is ignored for inference.

**Fallback:** if org billing is unavailable, set repo secret `COPILOT_GITHUB_TOKEN` to a fine-grained PAT with Copilot Requests access, and drop or avoid relying on `copilot-requests: write` for that path.

Without one of these, the agent job fails at Copilot CLI (often a models/`403` or AI-credits pricing error) even though activation and skip-if-match succeed.

## Optional: Copilot BYOK (external inference)

Milk-run stays on the Copilot engine. Consumers can optionally route inference through an external provider via [Copilot BYOK](https://github.github.com/gh-aw/reference/engines/#copilot-bring-your-own-key-byok-mode) — set `COPILOT_PROVIDER_BASE_URL` in `engine.env`. Omit that block to keep normal Copilot org billing.

Example: [OpenCode Go](https://opencode.ai/docs/go/) (OpenAI-compatible chat completions):

1. Create a repo or org Actions secret, e.g. `OPENCODE_GO_API_KEY`, with the key from [opencode.ai/auth](https://opencode.ai/auth).
2. In the consumer workflow, expand `engine` and allowlist the provider host:

```yaml
---
on:
  schedule:
    - cron: "0 9 * * *"
  workflow_dispatch:
engine:
  id: copilot
  env:
    COPILOT_PROVIDER_BASE_URL: https://opencode.ai/zen/go/v1
    COPILOT_MODEL: kimi-k2.7-code
    COPILOT_PROVIDER_API_KEY: ${{ secrets.OPENCODE_GO_API_KEY }}
permissions:
  contents: read
  pull-requests: read
  models: read
  copilot-requests: write
network:
  allowed:
    - defaults
    - opencode.ai
imports:
  - uses: autotelic/milk-run/.github/workflows/shared/milk-run.md@v0.1.0
    with:
      agentName: my-task
---

# My task

Describe the one job this agent should do each run.
```

Use a Go model served on `/chat/completions` (e.g. `kimi-k2.7-code`, `deepseek-v4-flash`, `glm-5.2`). Models on `/messages` need `COPILOT_PROVIDER_TYPE: anthropic`; `gpt-5.6-luna` needs `COPILOT_PROVIDER_WIRE_API: responses`.

A literal `COPILOT_PROVIDER_BASE_URL` also steers threat-detection at that host. Recompile after changing frontmatter (`gh aw compile …`).

## What the wrapper enforces

| Concern | Behavior |
|---------|----------|
| Concurrency | Skips when an open PR title contains `[milk-run/<agentName>]` |
| PR shape | Draft only; label `milk-run`; title prefix `[milk-run/<agentName>] ` |
| Empty work | Agent should `noop`; `if-no-changes: ignore` avoids junk PRs |
| Safety | No merge/force-push/secrets; smallest useful change; one concern per PR |

## Local examples / smoke tests

### Noop (no PR)

[`.github/workflows/milk-run-noop-demo.md`](.github/workflows/milk-run-noop-demo.md) — imports the shared file and must exit without changing the repo.

```bash
gh aw compile .github/workflows/milk-run-noop-demo.md
gh aw run milk-run-noop-demo
```

Expect a green run that calls **noop** and opens no PR.

### Draft PR

[`.github/workflows/milk-run-pr-demo.md`](.github/workflows/milk-run-pr-demo.md) — updates `SMOKE.md` and should open a draft PR.

```bash
gh aw compile .github/workflows/milk-run-pr-demo.md
gh aw run milk-run-pr-demo
```

Expect a **draft** PR titled `[milk-run/pr-demo] …` with label `milk-run`. A second run while that PR is open should skip. Close or merge the PR when finished smoking.

## Versioning

- Tag releases as `v0.1.0`, `v0.2.0`, …
- Consumers should pin `@v0.1.0` (immutable) or a full commit SHA
- Moving majors (`@v0`) are optional convenience; recompile after `gh aw` cache updates when bumping
