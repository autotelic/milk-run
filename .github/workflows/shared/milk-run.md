---
description: Milk-run wrapper — draft PRs, skip if open, one small change
import-schema:
  agentName:
    type: string
    required: true
    description: Short agent id used in PR title prefix and skip query (e.g. openapi-tcl)
on:
  skip-if-match: >-
    is:pr is:open in:title "[milk-run/${{ github.aw.import-inputs.agentName }}]"
permissions:
  contents: read
  pull-requests: read
  issues: write
  models: read
  copilot-requests: write
network: defaults
tools:
  github:
    toolsets: [repos, pull_requests, issues]
  edit:
safe-outputs:
  noop:
  create-pull-request:
    title-prefix: "[milk-run/${{ github.aw.import-inputs.agentName }}] "
    labels: [milk-run]
    draft: true
    max: 1
    fallback-as-issue: false
    if-no-changes: ignore
---

# Milk-run guardrails

You are running a **milk run**: a routine, low-drama pass that delivers at most one small, useful change as a **draft pull request**.

Agent id: `${{ github.aw.import-inputs.agentName }}`

## Hard rules

1. Make the **smallest useful change** that advances this agent's stated task. One concern only.
2. If there is nothing useful to do, call **noop** and stop. Do **not** open an empty PR.
3. Never force-push. Never merge. Never touch secrets, credentials, Doppler, or `.env` files.
4. Do not expand scope beyond the task instructions that follow these guardrails.
5. PR title (after the fixed prefix) must briefly describe the change.
6. PR body must include: what changed, how to verify, and any evidence (commands, coverage notes, links) when relevant.

Follow the task-specific instructions in the importing workflow after this section.
