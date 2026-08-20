---
name: harness-audit
description: Periodic re-assessment of a repository that already has a harness. Re-runs the layer scoring, compares to the last audit, reports drift (decayed guides, dangling references, noisy or bypassed sensors, stale specs), and proposes the next single increment. Use when the user asks to "audit the harness", "is our harness still working", "run the harness review", or invokes /harness-audit. Run every 4–8 weeks or after an incident the harness should have caught.
---

# Harness Audit

The steering loop on the harness *itself*. A harness silently degrades:
guides go stale, sensors get bypassed, specs drift from code. The audit
is the recurring ritual that catches the decay and steers the next
increment. Run every **4–8 weeks**, or after any incident a working
harness should have caught.

> This skill **is** the audit playbook — tool-neutral and stack-agnostic.
> It derives the checks to run from the controls the repo actually has,
> rather than assuming any particular command names.

---

## Step 0 — Establish the baseline

- Find the last audit (`docs/audits/harness-*.md`) if any. First audit →
  trend is `—` for every layer.
- Snapshot recent activity in parallel:

```
git log --oneline -50
git log --since="<last audit date>" --oneline | wc -l   # PRs/commits in period
```

- Ask the user (briefly): how many `--no-verify` / gate bypasses did the
  team resort to this period? Any production incidents — and were any
  attributable to a harness miss? These are the highest-signal inputs.

---

## Step 1 — Re-score the five layers

Run the [assess](assess.md) layer scoring again. Then check for the decay
patterns the first assessment can't see (they only appear over time):

### Feedforward decay
- **Dangling references** — every "MUST read" path in the guide file
  still resolves to an existing file? A broken reference erodes trust in
  the whole guide.
- **Stub sections** — any "TODO / add as decided" left in a guide? An
  agent infers a non-rule from a stub.
- **Staleness** — when was the guide / brain doc last updated relative
  to feature activity?
- **Unloaded guides** — if a run failed in territory a guide rule or
  skill covers, check whether the rule was ever *read* before diagnosing
  its content. A rule that exists but wasn't loaded has a discovery
  problem: fix the name, description, or index first, not the body.

### Maintainability / architecture / behaviour decay
- **Budget trend** — are suppression and TODO budgets stable, shrinking,
  or quietly climbing? Run the repo's budget checks in `--report` mode.
- **Bypass rate** — is the gate being skipped (`--no-verify`, disabled
  CI checks, `// eslint-disable` clusters)? A bypassed gate is no gate.
- **Noisy sensors** — any sensor producing false positives the team has
  learned to ignore? A noisy sensor is worse than none — tune or remove it.
- **Silent sensors** — any sensor that hasn't fired all period? It's
  mastered (a prune candidate) or broken — plant a violation and prove
  it still *can* fire before trusting its silence. Failures trending
  down across sensors is the good sign: guides absorbing what feedback
  used to catch.
- **Unreasoned suppressions** — are suppressions accumulating without
  stated justifications? The escape hatch has become a bypass.
- **Spec rot** — any spec marked `implemented` that no longer matches the
  code? Worse than no spec. If specs carry AC ids, run the traceability
  check: every criterion maps to a test, every scenario-tagged test back
  to a criterion (`../patterns/drift-and-health.md`).

---

## Step 2 — Produce the scorecard

Fill `../reference/scorecard-template.md` into a dated
file `docs/audits/harness-YYYY-MM-DD.md`:

- Per-layer score (/10) **and trend** vs last audit (`↑ → ↓ —`).
- Period stats: commits/PRs, incidents, harness-miss count, bypass count.
- The **top three actions** for the next period (this week / this month),
  each with an owner and a due date if the user can give one.

Write this file (the audit's output *is* a committed artefact, unlike
assess). Confirm before committing.

---

## Step 3 — Steer

Translate the decay findings into concrete steering moves — the same four
the model names (`../reference/harness-model.md` §5):

- A regression escaped the gate → tighten the gate, or consciously accept
  the class belongs to a later stage (CI, not pre-commit).
- A guide rule is repeatedly ignored → rewrite it more concretely, encode
  it as a sensor, or — if it's an invariant — move it to a hook.
- A sensor produces false positives → tune or remove it.
- A new failure class appeared twice → add it to the next increment.

For any incident, **trace the chain upstream** before fixing the
instance: the bug came from a gap in the spec; the gap survived review;
review was overloaded — where did the ambiguity *enter* (elicitation,
spec, review, verification)? Fix that layer, or the same class of bug
walks back in through the same door.

If the team runs zoned autonomy, apply the ladder rules: an **escaped
defect demotes the zone a rung** (propose-only ← merge-with-review ←
merge-on-green ← self-merge), and it stays demoted until the sensor that
should have caught the defect exists. Promotion needs evidence — the
zone's sensors have actually caught the failure class — never
confidence.

Hand the single highest-value move to [adopt](adopt.md). One increment,
then stop — same discipline as adoption.

---

## Step 4 — Close the loop on the ritual

Remind the user when the next audit is due (4–8 weeks out) and offer to
schedule it. If the trend is flat across layers and no incidents
occurred, the harness is healthy — say so plainly; not every audit needs
an action.
