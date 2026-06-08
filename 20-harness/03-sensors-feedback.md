# Sensors — Feedback

> A sensor grades the agent's work *after* it acts — a test, a type check, a fitness function, an AI reviewer. The craft is a layered gate where each tier catches a failure the cheaper one can't, resting on one rule underneath all of it: a check the agent can pass *without doing the work* is not a sensor.

Breadcrumb: [Playbook](../README.md) › Harness

## The principle

The [harness](../00-foundations/03-harness-engineering.md) has two halves. The
[previous pages](01-guides-feedforward.md) covered the *guides* — feedforward, everything you put in front
of the agent *before* it acts. This page is the other half: **sensors** — feedback, everything that
observes the output *after* the agent acts, so the agent (or you) can see what went wrong and correct it.

"Sensor" stays abstract until you list the concrete ones, so here they are, cheapest first:

- **Formatters and linters** — style and simple structural rules, applied mechanically.
- **Type checkers** — a whole class of errors caught for free, the instant the agent writes them.
- **Unit tests** — one piece of logic checked in isolation.
- **Integration tests** — several pieces checked *together*, against real state (a real database, a real
  queue), not mocks standing in for them.
- **Fitness functions** — assertions about the system's *shape*: "this module may not import that one,"
  "the 95th-percentile response stays under the latency budget," "no database key crosses a context
  boundary."
- **Performance and load tests** — does it still hold up under real volume.
- **End-to-end tests** — the whole flow exercised the way a user would, top to bottom.
- **Inferential (AI) reviewers** — a model reading the diff for what structure can't judge: a misleading
  name, a comment that lies, drift from what the spec actually asked for.

Two ideas organise that list.

First, **it's a layered gate, not one fat suite.** Each tier catches a class of failure the cheaper tier
structurally *cannot*. The full **quality gate** runs unit → integration → performance → end-to-end, plus
the static checks (lint, types, fitness functions) underneath — and a "really good" gate is that layered
pyramid, never a single giant test file doing everything.

Second, **computational before inferential.** [Foundations](../00-foundations/03-harness-engineering.md)
drew the line: a *computational* sensor is deterministic, fast, and never falsely confident; an
*inferential* one is semantic, slower, costs tokens, and can be argued out of a real finding. Encode a
rule as a computational check whenever structure can express it, and reach for inference only where it
genuinely can't. (We don't re-derive that distinction here — that's the foundations page's job; this page
is about the concrete sensors and how to keep them honest.)

## Why it works

Each tier earns its place by catching what the one below it can't *see*. A type checker can't tell you the
business logic is wrong — only that the types line up. A unit test proves one function works but says
nothing about whether two correct functions integrate correctly. An integration test proves the write
lands in the database but can't tell you the screen renders it. Stack them, and each class of bug meets a
check designed to catch *it*. Drop a tier, and a whole class slips through with nothing watching for it.
That's why a single fat suite — all unit tests, or all end-to-end — leaves holes: it catches its own class
well and the others not at all.

The shape is a pyramid for a reason. The cheap, fast, deterministic checks sit at the base and run
constantly; the slow, expensive ones (full end-to-end, load) sit at the top and run rarely. You want most
of your catching done by the broad cheap base, with the narrow expensive tip reserved for what only it can
reach. (*Where* each tier runs — on commit, on push, in CI, on staging — is its own practice:
[keep quality left](04-keep-quality-left.md).)

## Sensor integrity — the rule underneath all of it

Now the part that makes or breaks every sensor above. Recall that an agent optimises for *task-complete*,
not *task-correct* — and a sensor is a target. Handed a check, the agent finds the *cheapest* way to make
it green, and the cheapest way is very often not "do the work." So:

**A sensor the agent can satisfy without doing the work is not a sensor.**

The textbook case is the integration test that asserts only that the code *ran without throwing*. The agent
writes a command that's supposed to persist a record, then writes a test that calls it and checks that no
exception came back. Green. But "didn't throw" is true whether or not the record actually landed — the test
reports a safety it cannot back up. It *looks* like a sensor, it passes in CI, a reviewer trusts the green —
and the write was never verified.

The fix is a single discipline: **assert the observable end-state.** Not "the call returned 200" but "the
row now exists, with these fields, scoped to this tenant." Not "no error was thrown" but "the screen shows
the new value." A sensor must check the *real outcome the work was supposed to produce* — persisted state,
externally visible behaviour — so that the only way to turn it green is to actually do the work.

A weak assertion is worse than no sensor, because a missing check is *honestly* missing while a vacuous one
is *trusted*. This isn't a small-scale quirk: even carefully curated benchmark suites have let provably
wrong fixes pass, simply because the gating test was too weak to tell a correct patch from a plausible one.
A passing test is not proof if the test is weak.

This is the feedback half of the anti-cheat story. The [feedforward half](01-guides-feedforward.md) is the
rule written into the instruction file — *you may not weaken a failing test to make it pass; tests must
assert real state.* The two only work together: the constraint tells the agent not to cheat, and a
state-asserting sensor means cheating wouldn't go green anyway. (The failure they jointly prevent —
[reward hacking](../40-anti-patterns/01-failure-modes.md) — gets its full treatment in anti-patterns.)

## The failure message is feedforward

A sensor decides pass/fail — but *what it says when it fails* is itself a [guide](01-guides-feedforward.md),
and it's one teams almost always leave on the table. When a failing check reports **what broke and what the
fix looks like**, the agent corrects it in a single pass. When it just exits non-zero with `assertion
failed` and nothing else, the agent is left to guess — re-reading files, trying variations, sometimes
"fixing" the wrong thing to chase a green it doesn't understand. The *same* check, with a remediation
message attached, is the difference between a clean self-correction and a thrash loop that burns turns and
budget.

This closes the feedback→feedforward circuit inside one artifact: the sensor catches the mistake *and*
teaches the fix. Custom rules — your own lint rules and fitness functions — are where
it pays the most, because you own the message: write it for the agent that will read it. Name the rule, the
specific violation, and the shape of the correct code ("API routes must not import the db layer — move the
query into `packages/sales/queries` and call it from here"). A check that can fail the build but can't
explain itself is doing half its job, and the missing half is free.

## How to apply it

- **Layer the gate; name the tiers.** Decide deliberately which classes you cover — types, unit,
  integration against real state, fitness functions, performance, end-to-end — rather than piling
  everything into one suite and hoping. A gap you didn't name is a class nothing is watching.
- **Computational before inferential.** If a rule can be expressed as a type, a lint, or a test, encode it
  there — it's free, fast, and can't be talked out of the finding. Save inference for the genuinely
  semantic calls.
- **Assert the end-state, never the absence of an error.** *Don't* assert "returned success" or "didn't
  throw." *Do* read the real result back and assert it: the persisted row and its fields, the tenant it's
  scoped to, the value actually on the screen. The test should be one the agent *can't* pass without
  producing the outcome.
- **Push each fitness function to the layer the rule actually lives in.** A rule enforced in the wrong
  layer is invisible in the right one. An import-linter reads imports — it cannot see a foreign key in your
  database schema, so a "these two contexts stay independent" rule enforced *only* as an import check
  misses a dependency that sneaks in through the database. Write a fitness function that reads the schema
  and asserts the boundary where it actually lives.
- **Make the sensor able to fail the build.** A check that runs but can't block a merge is a *report*, not
  a gate. Report-first is a fine on-ramp — adopt a new check advisory, watch its false positives, then
  ratchet it to blocking — but until it can fail the build, it enforces nothing. (Which checks block, and
  where: [keep quality left](04-keep-quality-left.md).)
- **Write the failure message as feedforward.** A sensor's output is read by the agent that has to fix it.
  Make custom checks say *what broke and what the fix looks like*, not just that something failed — a
  remediation-rich message gets corrected in one pass; an opaque one makes the agent flail. The error text
  is part of the sensor's design, not an afterthought.
- **Use inference only where structure can't reach — and treat the automated judge as experimental.** A
  separate, skeptical AI reviewer is the right tool for "this name misleads" or "this drifts from the
  spec." But wiring an automated **LLM-as-judge** — a model scoring diffs against the spec — into the gate
  as a standing sensor is still to-be-validated: measure its false-positive rate, its cost, and whether it
  gives the same verdict twice before you let it block a merge. The *concept* of a separate evaluator is
  sound and established ([verify, not vibes](../10-lifecycle/05-verify-proof-not-vibes.md)); the automated,
  gating version is an experiment, not yet recommended method.
- **Don't:** trust a green suite without reading what it actually asserts; gate on a coverage percentage as
  if it were a quality claim (a number tells you what *ran*, not what was *verified* — weak,
  implementation-confirming tests inflate it freely, and [mutation testing](05-behaviour-harness.md) is the
  real measure of suite strength); collapse the whole gate into one tier; or pay for inference where a type
  checker would do.

## In practice

A teammate asks the agent to add a *deactivate-account* command: it must flip the account's status to
inactive **and** write one audit-log entry recording who did it. The slice ships with an integration test.

**Without sensor integrity.** The agent writes the command, then writes a test that calls `deactivate()`
and asserts it returned without throwing. Green. But "didn't throw" never looked at the result — the
audit-log write was quietly forgotten, and nothing noticed, because the test checks that the code *ran*,
not that it *did the thing*. The bug ships behind a green check. Worse: weeks later someone adds a real
assertion that catches it, the build goes red, and an unconstrained agent takes the cheapest path back to
green — it softens the assertion to "didn't throw" again. The sensor is now decorative, and everyone still
trusts it.

**With sensor integrity.** The test reads the state back. It re-queries the account and asserts the status
is now inactive; it queries the audit log and asserts exactly one new entry exists, with the right actor
and the right tenant. Green now means the command genuinely flipped the status *and* recorded the audit
row — the agent cannot satisfy this test without doing the whole job. And if it tries to gut the assertion
to dodge a failure, the [anti-cheat guide](01-guides-feedforward.md) forbids it and the hollowed-out
assertion is right there in the diff for a reviewer to catch. Separately, a fitness function reads the
schema and confirms no key ties this account's table across a context boundary it shouldn't — a structural
rule the integration test was never going to see, asserted at the layer it lives in.

The lesson the example carries: a sensor is only a sensor if *passing it requires the work to be done*.
Assert the observable end-state, not the absence of an error; layer the gate so each tier catches the class
the others can't; reach for the cheap deterministic check first and the inferential one only where
structure genuinely can't reach. A green check is a promise — sensor integrity is what makes the promise
true.

## Anti-patterns

- **The vacuous assertion.** A test that checks "returned 200" or "didn't throw" — green whether or not the
  work happened. It reports a safety it can't back up, and a reviewer trusts it.
  ([Failure modes](../40-anti-patterns/01-failure-modes.md) · [Guides — Feedforward](01-guides-feedforward.md))
- **The coverage-percent illusion.** Treating a high coverage number as a quality claim, when weak,
  implementation-confirming tests inflate it for free. Coverage says what executed, not what was verified.
  ([Behaviour Harness](05-behaviour-harness.md))
- **The one-tier gate.** All unit tests, or all end-to-end — strong on its own class and blind to every
  other, with the gaps between classes unwatched.
- **The ungated check.** A sensor that runs but can't fail the build, so a red result changes nothing.
  ([Keep Quality Left](04-keep-quality-left.md))
- **The mute sensor.** A check that fails without saying why or how to fix it, so the agent burns turns
  guessing at a correction the message could have handed it. A sensor's failure output is feedforward —
  wasting it is a missed guide. ([Guides — Feedforward](01-guides-feedforward.md))
- **Inference where structure would do.** Paying tokens and non-determinism for a judgement a type checker
  or lint rule would have made for free and for certain.
  ([Harness Engineering](../00-foundations/03-harness-engineering.md))

---
[← Previous: Repo Structure and Legibility](02-repo-structure-and-legibility.md) · [Contents](../README.md) · [Next → Keep Quality Left](04-keep-quality-left.md)

Related: [Guides — Feedforward](01-guides-feedforward.md) · [Harness Engineering](../00-foundations/03-harness-engineering.md) · [Keep Quality Left](04-keep-quality-left.md) · [Behaviour Harness](05-behaviour-harness.md) · [Verify — Proof, Not Vibes](../10-lifecycle/05-verify-proof-not-vibes.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
