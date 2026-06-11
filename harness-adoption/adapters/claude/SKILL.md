---
name: harness-assess
description: Assess a repository's engineering practices against the harness-engineering model, then adopt fitted controls incrementally. Use when the user asks to "assess our harness", "score our practices", "set up the harness", "close our top gap", "audit the harness", or invokes /harness-assess, /harness-adopt, or /harness-audit. Stack-agnostic and tool-agnostic — generates controls fitted to the detected stack rather than copying a fixed template.
---

# Harness Adoption (Claude entry point)

This is the thin Claude Code adapter for the **Harness Adoption Starter**.
The skill logic lives in the tool-neutral runbooks under
`harness-adoption/skills/`. This file just routes you there.

## Routing

| User intent | Run the runbook |
|-------------|-----------------|
| "assess", "score", "where are our gaps", `/harness-assess` | `harness-adoption/skills/assess.md` |
| "adopt", "set up the gate/test/guide", "close the top gap", `/harness-adopt` | `harness-adoption/skills/adopt.md` |
| "audit", "is the harness still working", `/harness-audit` | `harness-adoption/skills/audit.md` |

Read the matching runbook in full and execute it step by step. The
runbooks reference:

- The control library: `harness-adoption/patterns/`
- Shared theory and templates: `harness-adoption/reference/`
  (harness model, readiness rubric, scorecard, spec/ADR templates).

## Notes for Claude specifically

- Default the generated guide file to `AGENTS.md` (cross-tool). When the
  team uses Claude, also write a one-line `CLAUDE.md` pointing at it
  rather than duplicating content.
- `assess` is read-only — do not write to the repo unless the user asks
  to save the report.
- `adopt` writes one control at a time and commits only on explicit
  instruction. Never chain the whole gap list in one pass.
- Use `AskUserQuestion` for the ≤5 assessment questions in `assess.md`
  Step 2 (group them, max four per call).
