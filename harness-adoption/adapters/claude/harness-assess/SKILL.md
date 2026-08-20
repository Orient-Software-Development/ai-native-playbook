---
name: harness-assess
description: Read-only assessment of a repository's engineering practices against the harness-engineering model. Detects the stack, AI tool, and existing controls; scores the five harness layers; reports a ranked gap list. Use when the user asks to "assess our harness", "score our practices", "how harness-ready is this repo", "where are our gaps", or invokes /harness-assess. Writes nothing unless asked to save the report.
---

# Harness Assess (Claude entry point)

Thin Claude Code adapter for the **Harness Adoption Starter**. The skill
logic lives in the tool-neutral runbook — read it in full and execute it
step by step:

`harness-adoption/skills/assess.md`

The runbook references the control library
(`harness-adoption/patterns/`) and shared theory and templates
(`harness-adoption/reference/`).

## Notes for Claude specifically

- `assess` is read-only — do not write to the repo unless the user asks
  to save the report.
- Use `AskUserQuestion` for the ≤5 assessment questions in Step 2
  (group them, max four per call).
- When the user then wants to act on a gap, hand off to the
  `harness-adopt` skill — do not start writing controls from here.
