# The harness model

The conceptual model the skills reason from, self-contained. Read once;
reread when deciding where a new control belongs. It distils the
playbook's foundations (`../../playbook-md/00-foundations/03-harness-engineering.md`)
into the working vocabulary the assess/adopt/audit skills use.

---

## 1. Two control types — both required

| Type | What it does | Examples |
|------|--------------|----------|
| **Guide (feedforward)** | Lowers the chance of a bad output by injecting context **before** the agent acts | guide file (`AGENTS.md`), specs, ADRs, invariants, UI conventions |
| **Sensor (feedback)** | Detects a bad output **after** it happens, blocking or surfacing it | hooks, lint, type check, build, tests, budgets, CI checks, LLM review |

Feedforward-only leaves no verification the rules were followed.
Feedback-only forces the agent to learn by repeated failure. A harness
ships both — and the loop only **closes** when a third thing is present:
a **gate** that enforces the sensor's verdict. Guide + sensor + gate =
closed loop. Miss any one and you don't have a small harness, you have
no harness.

---

## 2. Two execution modes

| Mode | Speed | Cost | Determinism | Examples |
|------|-------|------|-------------|----------|
| **Computational** | ms–s | ~zero | deterministic | lint, types, build, tests, budgets, secret scan |
| **Inferential** | s–min | per-call | non-deterministic | LLM diff review |

Rule: **prefer computational for structural problems; reserve
inferential for semantic ones.** Never gate fast feedback (commit, push)
on an inferential sensor.

---

## 3. The five layers

1. **Feedforward (guides)** — what context the agent has before acting.
   Guide file, specs, ADRs, invariants, UI conventions, definition of done.
2. **Maintainability** — internal code quality a reviewer cares about.
   Format, lint, types, suppression budget, TODO budget, secret scan.
3. **Architecture fitness** — structural properties: module boundaries,
   layer order, banned dependencies, no cycles.
4. **Behaviour** — does the code do what the business wants. Unit +
   integration + e2e tests; behaviour-contract packs for complex domains.
5. **Inferential** — LLM-based semantic review of the diff (CI only).

Assess scores each 0–10 and records which of {guide, sensor, gate} is
present per layer.

---

## 4. Keep quality left

Distribute checks by cost and speed:

```
pre-commit   →   pre-push   →   CI   →   staging   →   production
[ ms–5s ]      [ 5–90 s ]    [ 1–10 min ] [ minutes ]  [ continuous ]
fast/cheap                                              slow/expensive
```

- Fast computational checks (lint touched files, secret scan) → pre-commit
- Heavier computational (full build, tests, budgets) → pre-push
- Expensive (e2e, dependency graph, bundle budget) → CI
- Inferential (LLM review) → CI on PR
- Runtime fitness assertions → production

A slow pre-commit trains developers to use `--no-verify`. The single
`check` command + gate (`../patterns/check-and-gate.md`) is what enforces
this distribution.

---

## 5. The steering loop

Humans iteratively improve the harness in response to observed failures:

```
Agent acts → Sensor fires (or fails to) → Human reviews failure →
   Human updates a guide, adds a sensor, or tightens a rule → repeat
```

- A regression escaped the gate → tighten the gate, or accept the class
  belongs to a later stage (CI, not pre-commit).
- A guide rule is ignored → rewrite it more concretely, or sensor it.
- A sensor produces false positives → tune or remove it (noise is worse
  than nothing).
- A failure class appeared twice → add it to the next increment.

The [audit](../skills/audit.md) skill is the formal cadence (every 4–8
weeks) for this loop, applied to the harness itself.

---

## 6. The human role

A good harness externalises tacit expertise into explicit guides and
sensors, but cannot replace judgement: it tells you when a decision
violates an encoded rule, never when a decision is *wrong*. New rules
require human discovery. **Effective harnesses redirect human effort to
the decisions that matter — spec authoring, ADR debate, audit and tuning
— rather than eliminating human input.**

---

## 7. Anti-patterns

- **Aspirational rules without enforcement.** A guide rule with no sensor
  is wish-thinking. Sensor it or mark it soft.
- **Dangling guide references.** A guide pointing at a file that doesn't
  exist breaks trust in the whole guide. Audit for this.
- **Stub sections.** "_Add as decided._" is dead content; the agent
  infers a non-rule. Mark undecided things `TBD — defer to PR review`.
- **Sensors that soft-skip on environmental failure.** Exiting 0 when a
  dependency is unreachable gives no signal. Fail loudly instead.
- **Pre-commit checks > 5 s.** They train `--no-verify`. Move them right.
- **Inferential gates on the fast path.** They destroy the local loop and
  cost more than they add. CI only.
- **One giant guide file.** Past ~400 lines the agent attends to the
  wrong things. Extract and link.
- **Spec rot.** A spec marked `implemented` that no longer matches the
  code is worse than no spec. Keep current or supersede.
- **The big-bang harness.** Wiring every control at once. It stalls and
  over-engineers a repo with no shape yet. Ship the minimum viable
  harness, grow the rest by watching what fails.
- **The half-loop.** Guides with no gate (suggestion box) or sensors with
  no gate (dashboard nobody reads). All three, or it isn't a harness.
