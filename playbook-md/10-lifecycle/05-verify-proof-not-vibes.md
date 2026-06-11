# Verify — Proof, Not Vibes

> "It runs" is not "it works," and "all tests passing" is not proof when the agent wrote the tests from its own code. A change is done when someone other than the author has checked it against the spec and attached the evidence — proof a reviewer can see, not a claim they have to trust.

Breadcrumb: [Playbook](../README.md) › Lifecycle

## Story so far

You steered the agent through the code, nudging it off the wrong turns in real time. It now beams up at you: *"Done! All tests passing."* Resist the urge to high-five. The agent wrote those tests, from its own code, with finishing as its goal — which is a bit like letting a student grade their own exam and act surprised everyone got an A. This chapter is where "it runs" stops counting as "it works." We trade vibes for proof: graded against the spec, by someone other than the author, with the evidence stapled on.

## The principle

Verification is not the optional last step — it's the step that decides whether anything
before it counts. And it has a specific shape, because the easy version of it doesn't work
on agent output. Three rules:

- **Grade against the spec, not the implementation.** The question is never "does the code
  do what the code does" — it always does. The question is "does it do what the
  [contract](01-spec-the-contract.md) said." This is exactly where AI-generated tests fall
  down: a test written *from* the code mirrors the code, so it passes whether the behaviour
  is right or wrong. A green suite like that confirms the implementation; it says nothing
  about the business need.
- **A separate, skeptical evaluator grades it — never the author.** Ask the agent that
  wrote the code whether the code is good and it will confidently say yes; finishing was
  its target, so it reads its own work as finished *and* correct. This is the
  **self-evaluation failure**, and the only fix is to split the roles: a different human,
  or a fresh agent with no stake in the code and the spec in hand, does the grading.
- **Proof is an artifact, not an adjective.** "Verified" attached to nothing is a vibe.
  Attach the **evidence**: for UI, captured screenshots of the real screen in the real
  states; for backend, saved log excerpts and state dumps showing the actual persisted
  result. A reviewer should be able to *see* the proof, not take your word for it.

One distinction keeps the second rule from being misread: **self-checking and self-gating
are not the same act.** Have the agent verify its own work *hard* — read the logs, query
the state, walk the edge cases, capture the evidence — because that first pass catches most
mechanical defects at the lowest cost there is, before anyone else spends a minute on it.
Demanding that self-check is right; it is *not* the thing the rule forbids. What the agent's
self-check can never be is the **final word** — the verdict that the change matches the
contract goes to a separate evaluator with the spec in hand. So the shape is two passes, not
zero: the agent self-verifies as the cheap first line, and an independent grader owns the
gate. "Necessary but not sufficient" — never "don't bother."

In practice that means doing three concrete things: **read the logs and check the state
yourself** (did the row actually land? did the log show the path you expected, or a
swallowed error?); **walk the spec's edge cases** — the boundaries the happy path skips
(empty, the second user, the second click, the cleared flag), using the acceptance
criteria as your checklist; and keep an **audit spec** for anything that must keep working
— a standing end-to-end walk-through you re-run later to catch *drift*, so a feature that
worked at merge but broke three changes later surfaces on purpose instead of in production.

None of that is possible without something to observe. Evidence has a precondition: a
**runnable environment the agent can actually drive** — a bootable app, a queryable
datastore, a browser the agent can click through (for example a Playwright driver wired into
the agent). To *capture* a screenshot of the real screen in the real state, or to read
persisted state back out, the agent has to run the thing and reach into it. Where that
substrate is missing, "attach the proof" is an instruction the agent physically cannot
follow, and *proof not vibes* quietly degrades back to vibes. Provisioning the runnable
environment is what makes agent self-verification possible at all — it's the substrate the
whole page stands on.

> **Experimental — the LLM-as-judge.** A local agent that reads the diff and the spec and
> scores the match is a promising way to *scale* the skeptical-evaluator idea. But wiring it
> into the gate as trusted method is not yet proven: measure its false-positive rate, its
> cost, and how consistent it is run-to-run before you rely on it, and keep a human signing
> off until it earns its place. Treat it as an experiment to validate, not a recommendation.

## Why it works

Agents optimise for *task-complete*, not *task-correct* — so self-grading inherits the
bias of the thing it's grading. The same process that produced the code produces the
praise, and "I'm done" quietly becomes "it's good." Only an evaluator with no stake and the
spec in front of them asks the question that actually matters — *does this match the
contract?* — instead of the one the author already answered for themselves: *did I finish?*

The implementation-mirroring problem is structural, not a matter of effort. A test derived
from the code encodes the code's current behaviour as "expected." If the code has a bug,
the test enshrines the bug and goes green. So the suite proves the code does what it does —
which you already knew — and is silent on whether that's what was needed. The only test
that can catch a wrong behaviour is one derived from the *spec*. (The structural way to make
tests resist this — [property-based and mutation testing](../20-harness/05-behaviour-harness.md)
— lives in the harness chapter; here the lever is simply *grade against the contract, by
someone other than the author*.)

And proof-as-artifact changes *who* can check, and *when*. A claim ("I verified it") can
only be trusted. A screenshot of the two-user case, or a state dump showing one user's flag
absent from another user's query, can be *checked* — by a reviewer today, by a teammate at
merge, by you next week when something regresses. Evidence travels and outlasts the
conversation; trust does neither.

## How to apply it

- **Grade against the spec, by someone other than the author.** A fresh reviewer or a fresh
  agent, acceptance criteria in hand. The author never self-certifies — the same separation
  that makes [review](06-review-and-convergence.md) work.
- **Read logs, check real state.** Don't accept "ran without throwing." Confirm the write
  landed, the value is right, and the error path isn't silently swallowing the case it was
  meant to handle.
- **Walk the edge cases.** Empty, boundary, second-actor, undo, repeat. The spec's
  acceptance criteria are the floor; add the boundaries they imply.
- **Attach the evidence to the change.** Screenshots for UI; log excerpts and state dumps
  for backend. Make the proof reviewable, not recountable.
- **Give the agent a runnable environment to verify in.** Evidence requires a substrate — a
  bootable app, a queryable store, a browser driver. Without one the agent can't produce the
  screenshot or read the state back, and verification falls back to a claim. The environment
  is the precondition for proof, not an afterthought.
- **Keep an audit spec for anything that must keep working.** A standing walk-through you
  re-run to catch drift — the runtime sibling of the spec-vs-code reconciliation in
  [Spec — The Contract](01-spec-the-contract.md). Past merge, this extends into
  [observability](../30-delivery/03-observability.md): logs and metrics are verification
  that keeps running in production.
- **If you trial an LLM-as-judge, measure it before you trust it.** False positives, cost,
  run-to-run consistency — and keep a human in the loop until the numbers earn the gate.
- **Don't:** let the author grade their own work; accept "tests pass" as proof when the
  tests were written from the code; call anything verified with nothing attached.

## In practice

Take the *per-user deal priority* feature, sliced and built across the previous pages. Now
prove it.

**Vibe verification** looks like this. The agent reports: *"Done — implemented per-user
deal priority, all tests passing."* The tests it wrote: `flagDeal` inserts a row (passes);
the list query returns deals (passes). Green across the board. But it self-graded, and its
tests mirror its code — *not one of them logs in as a second user*. REQ-002, the privacy
rule that was the whole risk of this feature, was never actually exercised. Ship it on that
vibe and the first time two reps use the app, one sees the other's flags.

**Proof** looks different. A separate reviewer takes the spec's acceptance-criteria list —
not the code's tests — and verifies each, capturing an artifact for each:

```text
AC-002  (the one that matters — flags stay private)
  Open two sessions. User A flags deal #314. Read user B's list straight from
  the query + DB — not the UI.
  → state dump attached:  B's result set for deal #314 → { flagged_for_B: false }
    A's flag is absent from B's view. PASS, and the dump is the evidence.

AC-001 / AC-003  (flag surfaces, unflag restores)
  A flags #314 → screenshot: it jumps to the top, every other deal still below it.
  A unflags    → screenshot: it returns to its normal position.

Edge case the spec implied but didn't spell out
  A deal nobody has flagged → still appears, in normal order. Checked.
```

Now "verified" means something a reviewer can *see*: the two-user state dump shows the
privacy rule holding, instead of "all tests passing" asking them to trust it. And that
two-user walk-through doesn't get thrown away — it becomes an **audit spec**, re-run after
later changes, so if someone refactors the query next month and breaks isolation, it's
caught on purpose rather than by a confused customer.

(The team also trials a local agent that reads the diff against the spec and flags
mismatches. It's promising — but until they've measured how often it cries wolf and what it
costs, a human still signs off.)

## Anti-patterns

- **Vibe verification.** "It runs, ship it." Proof by feeling, with nothing checked and
  nothing attached — the failure this whole page exists to prevent.
  ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **Author self-certification.** The agent (or engineer) grading its own work. The
  self-evaluation bias all but guarantees a yes, so the grade is worthless.
- **Implementation-mirroring tests.** AI tests written from the code, so green means "the
  code does what the code does," not "the code does what the spec said." The structural fix
  is in [the behaviour harness](../20-harness/05-behaviour-harness.md).
- **The unattached claim.** "Verified," "tested," "looks good" — attached to no screenshot,
  log, or state dump. A reviewer can't check an adjective.

> **Next up — [Review and Convergence](06-review-and-convergence.md):** verification proved the change is correct. Now the last lifecycle step guards two slower threats — the zombie code that *reads* perfectly while doing the wrong thing, and the quiet pile-up of near-duplicates that turns a tidy codebase into one nobody dares touch.

---
[← Previous: Code With the Agent](04-code-with-the-agent.md) · [Contents](../README.md) · [Next → Review and Convergence](06-review-and-convergence.md)

Related: [Spec — The Contract](01-spec-the-contract.md) · [Task Slicing](03-task-slicing.md) · [Review and Convergence](06-review-and-convergence.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Behaviour Harness](../20-harness/05-behaviour-harness.md) · [Observability](../30-delivery/03-observability.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
