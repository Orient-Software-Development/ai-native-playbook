# Spec — The Contract

> The spec is the contract between what you meant and what the agent builds. Write it short, write it concrete, and write it so the person who owns the intent can read it — because a vague spec doesn't slow the agent down, it makes it confidently wrong.

Breadcrumb: [Playbook](../README.md) › Lifecycle

## Story so far

Foundations are behind us: you know *why* the governed loop beats autocomplete, *how* the agent sees the world through context, and *what* a harness is made of. Theory bank: full. Now we spend it on a single real change and follow that change all the way through — spec → plan → code → verify → review → ship. First stop is the spec, the least glamorous artifact in software and the one that quietly decides everything else. Because here's the thing about a vague spec: it doesn't slow an agent down. It just makes it confidently, fluently, plausibly wrong.

## The principle

Before any code, write a **specification** — a short, concrete statement of what the
feature must do and how you'll know it's done. The spec is the *contract*: the agent
builds against it, and you check the result against it. Everything downstream — the
plan, the code, the tests — is only as good as this document.

Two things make a spec a contract rather than a wish:

- **It's normative.** A *normative* requirement is an obligation stated as a clear
  pass/fail rule — "the export **must** include every account for the current tenant,"
  not "it should probably handle all the accounts." Each requirement is atomic enough
  to verify on its own, and paired with **acceptance criteria**: the observable checks
  that decide "done" without anyone having to interpret. Write these in business
  language, not implementation detail.
- **It separates the *what* from the *how*.** The spec owns the *what* — the
  behaviour and the rules the business cares about. The *how* — which function, which
  table, which library — is deferred to the [plan](02-plan-before-code.md). Mixing them
  buries the contract under technical noise and locks in design decisions before anyone
  has reasoned about them.

The spec is also **durable**. Unlike the [plan](02-plan-before-code.md) — which can be
ephemeral, thrown away once the code is written — the spec **lives with the code**:
committed to the repository, version-controlled, reviewed alongside the change it
describes. It's the
[system of record](../00-foundations/02-context-engineering.md) for *what the feature is
meant to do* — the one place a future reader, or a future agent, recovers the intent the
code itself can't explain. And because code drifts while the spec sits still, the contract
only stays a contract if the two are **regularly reconciled** — see "audit the two for
drift" below.

For UI features, go one step further: attach a **clickable prototype** — a throwaway
HTML mockup of the screen. Look-and-feel is the part prose is worst at describing and
the part stakeholders are quickest to react to. A mockup turns "is this what you meant?"
from a sprint-long round trip into a two-minute conversation, *before* a line of real
code exists.

## Why it works

An agent only acts on what's in front of it. Where a human fills a gap in the
requirements with a question or a sensible guess from experience, an agent fills it with
the *most probable* completion — which is often wrong in a way that looks completely
plausible. Vagueness doesn't make the agent hesitate; it makes it invent. A concrete
spec removes the gaps it would otherwise paper over.

Normative, business-legible requirements do a second job: they let the person who owns
the intent — a product owner, a sales lead, the client — actually read the contract,
challenge it, and sign off. Stripping the jargon isn't dumbing down. The people who know
what the feature is *for* can only catch a misunderstanding if they can understand the
document, and catching it here costs a sentence. Catching it after the code is written
costs a rebuild.

Keeping the spec *short* matters as much as keeping it concrete. A long spec degrades the
agent's own work: it reads the front carefully and skims the tail as its attention
spreads thin, so the requirements you buried at the bottom quietly don't get built. A
tight contract that links out to related specs beats one sprawling document that tries to
say everything.

## How to apply it

- **State requirements as obligations.** Use *must* / *must not* for binding rules.
  One rule per line, each one checkable. "Closed deals are hidden by default" — not "the
  list should feel clean."
- **Pair every requirement with an acceptance criterion.** If you can't name the
  observable check that proves a requirement, the requirement is too vague to build.
- **Write the *what*, defer the *how*.** Resist naming tables, endpoints, or components
  in the spec. If a technical constraint is genuinely part of the contract (a regulatory
  rule, an external API shape), state it as a requirement, not as a design.
- **Keep it short and linked.** One feature, one focused spec. Point to related specs by
  reference rather than restating them — and tell the agent which ones to read, so it
  doesn't reconstruct the context (wrongly) from the code.
- **Keep the spec with the code, and audit the two for drift.** Commit the spec to the
  repository next to what it describes, so it's versioned and reviewed with the change —
  not parked in a wiki the code can silently outgrow. Then run a *regular audit* that
  re-reads each spec against the code that's supposed to satisfy it, and treat any gap as a
  defect in one of the two: either the code regressed away from the contract, or the spec
  went stale and needs rewriting to match a deliberate change. A spec nobody re-checks
  quietly stops being the contract. (The standing, re-run "audit specs" in
  [Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md) are one way to make this
  automatic.)
- **For UI, ship a prototype with it.** A static HTML mockup the stakeholder can click
  through. It validates intent pre-code and later becomes the visual contract the
  [behaviour harness](../20-harness/05-behaviour-harness.md) and design guide check
  against.
- **Draft it with the agent, then own it.** You don't have to write the spec from a blank
  page — have the agent draft one from your rough description, then edit it until every line
  is what you meant. The discipline is unchanged: you review it, you sign it off, it's *your*
  contract. Lowering the cost of the first draft is not the same as outsourcing the intent —
  the agent proposes the words; the obligation they encode is still yours to confirm.
- **Don't:** open with "build a system that…" and a page of prose. Don't let the spec
  carry implementation detail the plan should own. Don't write a spec only an engineer
  can read for a feature a non-engineer owns.

## In practice

A sales lead asks for what sounds like a one-liner: *"let people mark a deal as a
priority and see priorities first."*

**Without a spec**, that sentence goes straight to the agent. It returns a clean,
plausible diff — and three decisions nobody made surface later. The priority flag is
*global*, so when one rep stars a deal it jumps to the top for the whole team (the lead
meant *per-user*). "See priorities first" became a hard filter that *hides* everything
else, not a sort (the lead meant the list still shows everything, just reordered). And
the star sits in a column the reps don't look at. None of this is a bug the build catches
— it compiles, it runs, the agent's tests pass. It's wrong against an intent that was
never written down, and now it's a rebuild.

**With a spec**, the same request becomes a contract the lead can read and sign off in
two minutes. Here's the whole thing — short, normative, business-legible, no
implementation in sight:

```markdown
---
title: Per-user deal priority
status: draft
type: specification
depends_on: []
related: [priority-mockup.html]   # clickable prototype — the visual half of the contract
---

# Per-user deal priority

## Purpose
Let each salesperson flag the deals they care about and see them first,
without changing what anyone else sees.

## Scope
In scope: flagging/unflagging a deal; ordering of the deals list.
Out of scope: notifications, priority on any other screen, team-wide priorities.

## Problem Statement
Salespeople track which deals matter in their heads or a side spreadsheet.
The list shows everyone's deals in one fixed order, so the few that need
attention get lost among the rest. There is no per-user way to surface them.

## Normative Contract
- REQ-001  A user MUST be able to flag a deal as a priority, and clear that flag.
- REQ-002  A flag MUST be private to the user who set it — one user's flag
           MUST NOT change another user's view.
- REQ-003  The deals list MUST show that user's flagged deals first, then all
           remaining deals in the existing order. No deal is hidden.

## Interfaces and Data Contracts
No new externally consumed API. A per-user priority flag is associated with a
deal; the storage shape and query are deferred to the plan. No existing
response consumed by another client changes.

## Acceptance Criteria
- AC-001  User A flags a deal. A's list shows it on top; every other deal is
          still present below it.
- AC-002  With users A and B logged in, A flagging a deal does not change
          B's list at all.
- AC-003  A clears the flag; the deal returns to its normal position for A.

## Risks and Mitigations
| Risk | Mitigation |
|------|------------|
| A flag leaks across users | REQ-002 + AC-002 assert isolation explicitly; covered by a two-user test |

## Open Questions
- None blocking. (Assumed: flags persist across sessions.)
```

Notice what the contract *doesn't* say: no table, no join, no sort key, no component
name — even the Interfaces section pushes the data shape down to the plan. Every line is
something the sales lead can confirm is what they meant, and every `REQ` has an `AC` that
decides "done" without interpretation. The `MUST`/`MUST NOT` wording is what makes it a
contract rather than a wish — each one is a check a
[sensor](../20-harness/03-sensors-feedback.md) can later enforce. The clickable prototype
referenced alongside it carries the look-and-feel the prose can't.

The lead clicks through the linked mockup and immediately says "actually, put the star at
the row's left edge" — a correction that would otherwise have arrived after the feature
was built, as a rebuild instead of a sentence. The *what* is now unambiguous and agreed; the
*how* (a per-user join, a sort key, where the button mounts) is left for the plan. The
agent builds against a contract instead of guessing at a sentence — and "done" is a thing
you can check, not a thing you hope for.

## Anti-patterns

- **The vague spec.** A loose narrative the agent fills with confident guesses — the
  failure this whole page exists to prevent. ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The novel spec.** So long the agent builds the front and silently skips the tail as
  its attention thins out.
- **The how-not-what spec.** A pile of implementation detail with no clear obligations,
  so neither the stakeholder nor a sensor can tell whether it was satisfied.
- **The unreadable spec.** Written in jargon the intent-owner can't follow, so the one
  person who could catch the misunderstanding never gets the chance.
- **The orphaned spec.** Written once, committed, and then never reconciled with the code,
  so it drifts into fiction — the code says one thing, the contract another, and nobody
  knows which is right. Prevented by keeping the spec with the code and auditing the two on
  a regular cadence.

> **Next up — [Plan Before Code](02-plan-before-code.md):** the contract settles *what* to build. But the agent will happily commit to the first *how* it dreams up and bury that decision deep inside a diff — so next we slip a planning step in between, and catch a wrong approach while it's still a paragraph you can edit, not a rewrite you have to mourn.

---
[← Previous: Harness Engineering](../00-foundations/03-harness-engineering.md) · [Contents](../README.md) · [Next → Plan Before Code](02-plan-before-code.md)

Related: [Plan Before Code](02-plan-before-code.md) · [Task Slicing](03-task-slicing.md) · [Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md) · [Guides — Feedforward](../20-harness/01-guides-feedforward.md) · [Behaviour Harness](../20-harness/05-behaviour-harness.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
