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
      agentName: noop-demo
---

# Noop demo milk run

This example exists only to prove the shared `milk-run` import compiles and runs in this repository.

Do **not** modify repository files. Call **noop** explaining that the demo agent has no real task in this repository.
