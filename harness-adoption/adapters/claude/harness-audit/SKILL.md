---
name: harness-audit
description: Periodic re-assessment of a repository that already has a harness. Re-runs the layer scoring, compares to the last audit, reports drift (decayed guides, silent or bypassed sensors, stale specs), and proposes the next single increment. Use when the user asks to "audit the harness", "is our harness still working", "run the harness review", or invokes /harness-audit. Run every 4–8 weeks or after an incident the harness should have caught.
---

# Harness Audit (Claude entry point)

Thin Claude Code adapter for the **Harness Adoption Starter**. The skill
logic lives in the tool-neutral runbook — read it in full and execute it
step by step:

`harness-adoption/skills/audit.md`

The runbook references the control library
(`harness-adoption/patterns/`) and shared theory and templates
(`harness-adoption/reference/`).

## Notes for Claude specifically

- The audit's output is a committed artefact
  (`docs/audits/harness-YYYY-MM-DD.md`) — confirm before committing.
- Hand the single highest-value steering move to the `harness-adopt`
  skill; one increment, then stop.
