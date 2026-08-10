---
on:
  workflow_dispatch:
engine: copilot
permissions:
  contents: read
  pull-requests: read
  models: read
  copilot-requests: write
imports:
  - uses: shared/milk-run.md
    with:
      agentName: pr-demo
---

# PR demo milk run

Prove the shared `milk-run` wrapper can open a **draft PR**.

## Task

Make exactly one small change:

1. Create or update `SMOKE.md` at the repository root.
2. Set its contents to a short note that this file is maintained by the milk-run `pr-demo` agent, plus the ISO date of this run (UTC is fine).
3. Do not modify any other files.
4. Open the draft PR via the milk-run safe-outputs (do not noop unless `SMOKE.md` already matches that requirement with today's date).
