# Spec and ADR templates

Minimal, stack-neutral templates the [adopt](../skills/adopt.md) skill
drops into a repo when closing the specs/decisions gap
(`../patterns/specs-and-decisions.md`). Keep them light — they grow with
the product, not on day one.

---

## Specification template

Save as `specs/<area>/NNN-<slug>.md`.

```markdown
---
id: SPEC-NNN
title: <short title>
status: draft        # draft → approved → implemented → superseded
owner: <name>
updated: <YYYY-MM-DD>
---

## Problem
<what need this addresses, and for whom>

## Contract
- **Inputs:** <data / preconditions>
- **Outputs / effects:** <observable result — the persisted row, the
  returned value, the rendered state>
- **Invariants:** <what must always hold, in every case>

## Out of scope
<what this deliberately does not cover>

## Acceptance
- [ ] <observable, testable criterion>
- [ ] <observable, testable criterion>

## Open questions
<TBD items — mark them, don't let the agent infer answers>
```

Rule: the spec asserts **observable end-states**, never "it runs". The
behaviour test (`../patterns/behaviour-test.md`) proves the acceptance
criteria.

---

## ADR template

Save as `docs/decisions/ADR-NNN-<slug>.md`. One file per decision,
numbered, **never edited after acceptance** — supersede with a new ADR
instead.

```markdown
---
id: ADR-NNN
title: <the decision, as a noun phrase>
status: proposed     # proposed → accepted → superseded by ADR-MMM
date: <YYYY-MM-DD>
---

## Context
<the forces at play — what makes this a decision with trade-offs>

## Options considered
1. <option> — pros / cons
2. <option> — pros / cons

## Decision
<what we chose>

## Consequences
<what becomes easier, what becomes harder, what we now must live with>
```

The first ADR a team writes is usually the harness adoption itself — a
real decision with consequences worth recording.

---

## When to reach for the heavyweight option

For most teams the two templates above are enough. A domain with **many
commands sharing invariants and a real cost to silent divergence**
(billing, tenancy, access control) may warrant a **behaviour-contract
pack**: pin invariants, enumerate approved scenarios, build a coverage
matrix, and add sensors that fail the build when spec ↔ test ↔
implementation drift apart. Apply it to *one* active domain, never
retroactively across the codebase. See
`../patterns/specs-and-decisions.md` for when this is justified — it is
the most specialised control and assessment should not rank it highly by
default.
