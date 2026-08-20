# Harness scorecard — {{DATE}}

The output artefact of [assess](../skills/assess.md) (optional to save)
and [audit](../skills/audit.md) (saved to `docs/audits/harness-{{DATE}}.md`).

| Layer | Score | Loop state | Trend vs last | Headline |
|-------|-------|-----------|---------------|----------|
| **Feedforward (guides)** | {{F_SCORE}} / 10 | {{F_LOOP}} | {{F_TREND}} | {{F_HEADLINE}} |
| **Maintainability** | {{M_SCORE}} / 10 | {{M_LOOP}} | {{M_TREND}} | {{M_HEADLINE}} |
| **Architecture fitness** | {{A_SCORE}} / 10 | {{A_LOOP}} | {{A_TREND}} | {{A_HEADLINE}} |
| **Behaviour** | {{B_SCORE}} / 10 | {{B_LOOP}} | {{B_TREND}} | {{B_HEADLINE}} |
| **Inferential** | {{I_SCORE}} / 10 | {{I_LOOP}} | {{I_TREND}} | {{I_HEADLINE}} |

**Overall:** {{OVERALL_SCORE}} / 50.  **Readiness:** {{READINESS}} / 100.

Loop legend: `closed` (guide+sensor+gate) · `half` (guide or sensor, no gate) · `open` (none).
Trend legend: `↑` improved · `→` flat · `↓` regressed · `—` first assessment.

---

## Context

- Stack / tool / VCS-CI: {{CONTEXT_LINE}}
- Greenfield or legacy: {{LIFECYCLE}}
- Last assessment: {{LAST_DATE}} · This assessment: {{THIS_DATE}}
- Commits/PRs in period: {{PR_COUNT}}
- Incidents in period: {{INCIDENT_COUNT}} (attributable to a harness miss: {{HARNESS_MISS_COUNT}})
- Gate bypasses (`--no-verify` etc.) in period: {{BYPASS_COUNT}}
- Scheduled drift sensors running: {{DRIFT_SENSORS}} (deps / dead code / spec-drift / mutation trend, or "none")
- Autonomy rungs per zone (if the team runs zoned autonomy): {{AUTONOMY_RUNGS}}
  (propose-only · merge-with-review · merge-on-green · self-merge; demote a rung on any escaped defect)

---

## Ranked gaps / next actions

Priority 1 (now):
1. {{P1_1}} — closes {{LOOP_1}} — recipe `{{PATTERN_1}}` — effort {{EFFORT_1}}
2. {{P1_2}} — closes {{LOOP_2}} — recipe `{{PATTERN_2}}` — effort {{EFFORT_2}}

Later:
- {{LATER_1}}
- {{LATER_2}}

**Next single increment:** {{RECOMMENDATION}}
