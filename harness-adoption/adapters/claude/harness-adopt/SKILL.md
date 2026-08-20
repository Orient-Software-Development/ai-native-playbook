---
name: harness-adopt
description: Generate and install a fitted harness control into a repository, in the target stack's idiom. Takes the ranked gap list from harness-assess and writes only the controls the user approves, smallest first, then re-scores. Use when the user asks to "adopt the harness", "set up the gate", "add the behaviour test", "close the top gap", or invokes /harness-adopt. Writes files, one increment at a time, with explicit approval.
---

# Harness Adopt (Claude entry point)

Thin Claude Code adapter for the **Harness Adoption Starter**. The skill
logic lives in the tool-neutral runbook — read it in full and execute it
step by step:

`harness-adoption/skills/adopt.md`

The runbook references the control library
(`harness-adoption/patterns/`) and shared theory and templates
(`harness-adoption/reference/`).

## Notes for Claude specifically

- Default the generated guide file to `AGENTS.md` (cross-tool). When the
  team uses Claude, also write a one-line `CLAUDE.md` pointing at it
  rather than duplicating content.
- Write one control at a time; commit only on explicit instruction.
  Never chain the whole gap list in one pass.
- If no assessment ran this session, run the `harness-assess` skill (or
  its detection + scoring steps inline) first — never adopt a control
  the assessment didn't justify.
