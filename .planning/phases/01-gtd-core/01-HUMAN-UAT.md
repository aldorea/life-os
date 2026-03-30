---
status: partial
phase: 01-gtd-core
source: [01-VERIFICATION.md]
started: 2026-03-29T00:00:00Z
updated: 2026-03-29T00:00:00Z
---

## Current Test

[awaiting human testing]

## Tests

### 1. Actual Vault Migration Execution
expected: Copy config.example.yaml to config.yaml, configure vault path, run /migrate-vault. Git commit before changes, migration plan displayed, user confirms, folders moved, frontmatter added, config updated, second git commit.
result: [pending]

### 2. End-to-End Daily Loop
expected: Run /morning on real vault with Backlog and goals.yaml. Morning chains calendar + inbox + daily note. Daily note has goal-aligned focus. Run /close — captures reflection, updates only "Cierre del dia" section.
result: [pending]

### 3. Weekly Review Interactivity
expected: Run /weekly-review on vault used for at least one week. 7 steps present data and wait for input. "saltar" skips correctly. Goal history[] entries appended after goals step. Weekly note generated at end.
result: [pending]

### 4. /goal Weight Validation
expected: Define two quarterly goals with weights not summing to 1.0, run /goal list. Warning displayed: "Los pesos de objetivos quarterly suman X, deberían sumar 1.0."
result: [pending]

## Summary

total: 4
passed: 0
issues: 0
pending: 4
skipped: 0
blocked: 0

## Gaps
