# SUBMISSIONS

| Report | State | Anomalies | Notes |
|---|---|---|---|
| R-01 ga mise auto-trust | READY, not filed | A-1 | VALID-HIGH 7.8 after self-triage. Filing is the operator's action. |

## Pre-written triage answers (R-01)
- "User chose to work on the repo": ga only creates a branch; mise refused config on entry and on plain worktree (control 1 and 2). #11336 precedent.
- "Upstream mise": mise behaves as documented; Omarchy's `mise trust` call overrides it. Scope carve-out: "how Omarchy specifically uses or integrates the dependency".
- "Dupe of #11336": different mechanism and file; fix a73bcbfc did not touch default/bash/fns/worktrees.
