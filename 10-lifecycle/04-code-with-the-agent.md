# Code With the Agent

> Steer with the concrete thing, not an analogy — "match this function" beats "do it like the other page," because the agent fills "like" with its most probable guess. And never assume a rule you stated is a rule the agent followed: it drifts off clear instructions, so the enforcement has to live *outside* it, in sensors and review.

Breadcrumb: [Playbook](../README.md) › Lifecycle

## The principle

The [slice](03-task-slicing.md) is approved and the agent starts writing. Your job now
isn't to step back and wait for the diff — it's to *steer in the moment*, and to not trust
what you can't check. Two halves.

**Steer with concrete reference, not analogy.** "Make it work like the accounts list"
forces the agent to guess what *like* means — and it guesses the most probable thing, which
may not be the thing you meant. Instead quote the concrete behaviour: point it at the exact
file and lines, or paste the rule you want followed. Be specific about the task itself — the
inputs, the outputs, the files to touch, the files to read first. A scoped, concrete
instruction is to this in-the-moment loop what a concrete [spec](01-spec-the-contract.md) is
to the whole feature: it removes the gap the agent would otherwise fill with a confident
wrong guess. Then *watch it work and course-correct early* — the moment it heads the wrong
way, redirect. A nudge at the first wrong line is a sentence; the same correction ten files
later is a rebuild.

**Don't trust the agent to police itself.** Here's the part that catches people out:
putting a rule in front of the agent does not mean the agent will follow it. Not out of
defiance — it simply can't reliably hold every instruction in attention while solving the
problem in front of it, so it drifts off rules you stated plainly. Guides and prompts
*lower the rate* of violations; they never drive it to zero. So never rely on the agent
enforcing its own constraints. The enforcement lives outside the agent: **sensors** —
computational checks like type checkers, linters, and tests that fail loudly the instant a
rule is broken — and **review** by a separate evaluator. Instructions are *feedforward* (what
you tell the agent before it acts); sensors and review are *feedback* (what catches it after).
You need both, because the first is unreliable on its own.

## Why it works

"Like X" relies on the agent reconstructing your intent from an analogy, and it doesn't
have your mental model of how X works — it has the most probable completion of the phrase
"like X," which can be subtly or wildly off. Hand it the actual reference and there's
nothing left to reconstruct: the gap where the wrong guess would go is simply closed.

Early correction is cheap for the same reason [slicing](03-task-slicing.md) and
[plan review](02-plan-before-code.md) are: a wrong direction *compounds*. Caught at its
source it's a one-line redirect; caught after the agent has built three things on top of it,
it's a thread you have to pull through all of them. Watching and nudging keeps every wrong
turn small enough to fix in a sentence.

And the self-follow limit is *structural*, not a tuning problem you can prompt your way out
of. The agent produces the next most probable step given everything in front of it; a rule
you stated is one signal among many competing for its attention, and the local pull of
"make this work" can simply outweigh it. That's why "but I told it not to" is neither a
defence nor a fix — the only reliable enforcement is a check the agent cannot talk its way
past. It's the same reason [verification](05-verify-proof-not-vibes.md) can't be self-graded
and [review](06-review-and-convergence.md) can't be done by the author: the agent's own word
about its own compliance is the least reliable signal you have.

## How to apply it

- **Quote, don't gesture.** "Match the validation in `handleSubmit` at `orders.ts:40-58`"
  beats "validate like the orders form." Paste the reference behaviour or the rule straight
  into the task.
- **Be specific and scoped.** Name the deliverable, the inputs and outputs, the files to
  touch, and the files to read *first*. A scoped task is one the agent can hold whole — and
  one you can actually review.
- **Course-correct early and often.** Watch the work as it happens and redirect at the first
  wrong turn, not at the finished diff. You can **stop the agent mid-action** to redirect — that
  interrupt is a normal, expected part of steering, not a disruption, and it's the cheapest
  correction you have. Interrupting isn't rude; it's the cheap moment, and waiting for the
  finished diff to raise something you saw coming ten steps ago is the expensive one.
- **Send investigation to a side channel.** For "go figure out how X works," use a
  **subagent** — a separate, scoped agent run — so the answer comes back without flooding
  your working thread with the whole search. Keep the main thread's
  [attention budget](../00-foundations/02-context-engineering.md) for the task itself.
- **Keep the agent on a leash you can see.** Prefer a scoped, reversible working mode where
  consequential actions are approved deliberately, and keep diffs small enough to review.
  Don't let it range unsupervised across the repo on things you can't undo.
- **Put rules where a sensor can enforce them.** If a constraint matters, back it with a
  check — a type, a lint rule, a test — not just a sentence in a [guide](../20-harness/01-guides-feedforward.md).
  State it as feedforward *and* enforce it as [feedback](../20-harness/03-sensors-feedback.md).
- **Don't:** say "like X does it" and hope; let a wrong direction run because interrupting
  feels premature; assume a rule in the prompt is a rule obeyed; or let the agent act
  unsupervised on anything you can't reverse.

## In practice

Take the *per-user deal priority* feature at **slice 3** — extend the existing deals-list
query to join the current user's priorities and order flagged-first.

**Vague and trusting**, you tell the agent: *"extend the deals list to show flagged deals
first, like how we sort other lists, and remember flags are per-user."* Two things go
wrong. *"Like how we sort other lists"* — the agent picks a different list that happens to
sort by a global column and mirrors *that* shape, which doesn't fit per-user data at all.
And *"remember flags are per-user"* — a rule, stated clearly — doesn't hold: deep in writing
the join, the agent drops the `user_id` filter because the simpler query still "works"
(it returns rows), and the local pull of make-it-run beat the rule you stated. It hands you
a green diff that leaks flags across users. You told it the rule; it didn't follow the rule.

**Concrete and checked**, you point at the exact thing instead: *"extend `getDealsList` in
`deals/queries.ts:120-145`; left-join `deal_priorities` on `(deal_id, current_user_id)`;
order flagged-first, keep the existing sub-order. Read the `deal_priorities` schema first."*
Now there's no analogy to reconstruct. And you don't *rely* on "remember it's per-user" —
the privacy rule is backed by the two-user test from the slice's plan (a sensor) and a
reviewer who isn't the author. Mid-stream you watch the agent begin the join without the
user filter and redirect it in one sentence, before it's buried under anything. The rule is
honoured — not because you asked nicely, but because a check would have caught the violation
if you hadn't, and because the concrete reference meant it mostly didn't need catching.

The lesson the example carries: *specificity removes the guess, and external checks remove
the dependence on the agent's self-discipline.* Steer concretely, verify always, and never
confuse "I instructed it" with "it complied."

## Anti-patterns

- **The "like X does it" prompt.** Steering by analogy, so the agent reconstructs your
  intent from a guess and builds against the wrong reference.
  ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **Trusting the agent to self-enforce.** Assuming a rule stated in the prompt or a guide is
  a rule obeyed. The agent drifts off clear instructions, so an unenforced rule is only a
  suggestion — the fix is [sensors](../20-harness/03-sensors-feedback.md) and
  [review](06-review-and-convergence.md), not a firmer tone.
- **Late course-correction.** Letting a wrong direction run all the way to the diff because
  interrupting felt premature — turning a one-line redirect into a rebuild.
- **The unsupervised agent.** Letting it act across the repo on irreversible things with no
  scoped mode, no approval step, and no small diff to review.

---
[← Previous: Task Slicing](03-task-slicing.md) · [Contents](../README.md) · [Next → Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md)

Related: [Task Slicing](03-task-slicing.md) · [Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md) · [Review and Convergence](06-review-and-convergence.md) · [Guides — Feedforward](../20-harness/01-guides-feedforward.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Context Engineering](../00-foundations/02-context-engineering.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
