# The Responsible Team and AI Debt

> Responsibility doesn't transfer to the model — the human who merges owns the code, full stop. So the team does two things the agent can't: it maps the zones where the agent reliably flops and decides what stays human, and it treats the debt the agent generates as a managed budget — visible, and paid down before it compounds.

Breadcrumb: [Playbook](../README.md) › Adoption

## The principle

Everything in this playbook makes the agent more capable. None of it makes the agent *responsible*. The
agent has no stake in the outcome — no career on the line, no teammate who'll think less of it, no recoil
at a 300-line function. It optimises for the task in front of it and moves on. That means it can never be
the party accountable for what ships. **The human who merges the change is the author of record** — not the
model that wrote it.

This isn't a moral nicety; it's load-bearing. The "humans steer, agents execute" division of labour only
works if a human is genuinely steering — owning the call on what's good enough to ship. Drop that, and you
get the failure mode where [debt compounds invisibly](../40-anti-patterns/01-failure-modes.md) precisely
because no one feels accountable for code they didn't write.

Two responsibilities follow, and they're the subject of this page:

1. **Allocate trust by zone, not uniformly.** The agent is not equally reliable everywhere. Name the zones
   where it reliably flops, and decide for each one what to automate, what to guard with a sensor, and what
   stays human.
2. **Manage AI debt as a budget.** Plausible-but-mediocre code accumulates faster than anyone reviews it.
   Make that debt visible and pay it down continuously, the way you'd service a high-interest loan — not in
   occasional painful bursts.

## Why it works

**Uniform trust over-trusts the agent exactly where it's weakest.** The agent is genuinely strong in
well-harnessed, verifiable zones — typed CRUD against a real test suite — and genuinely weak in others.
Treat it as uniformly trustworthy and you apply the *same* light review to a routine list endpoint and a
change to how money is calculated. The routine change didn't need you; the money change needed you most,
and got the same glance. Mapping the **flop zones** — the places the agent reliably gets wrong — is how you
put human attention where it has the most leverage, which is the whole point of a harness: it
[redirects your effort to the calls that matter](../00-foundations/03-harness-engineering.md), it doesn't
remove it.

There are four flop zones worth naming because they recur across codebases:

- **Subjective frontend and design.** The agent is weakest where it's most confident — it declares a screen
  "looks right" without being able to see it, and gravitates to bland, generic layouts. This is the
  [self-verify blind spot](../10-lifecycle/05-verify-proof-not-vibes.md).
- **Cross-cutting refactors.** The agent replicates the patterns already in the repo — *including* the bad
  ones — so a sweeping change spreads existing flaws and drifts the architecture rather than cleaning it up.
- **Ambiguous business rules.** A vague spec doesn't make the agent cautious, it makes it
  [confidently wrong](../10-lifecycle/01-spec-the-contract.md): it fills the gap with a plausible guess and
  writes tests that confirm the guess.
- **Security-sensitive code.** Auth, permissions, data exposure — the stakes are high and the failure is
  plausible-looking, the worst combination for a skim review.

**AI debt compounds invisibly, so it needs a process, not goodwill.** Two forces stack. The agent produces
plausible code faster than humans can carefully read it, so unreviewed shortcuts pile up. And because no one
*wrote* the code, no one feels the ownership that normally makes an engineer go back and clean it up — the
debt has no one to pay it. Left alone it compounds until the codebase is one neither human nor agent can
change safely. The fix is to make the debt a tracked, scheduled thing instead of a feeling someone might
act on.

**Debt is a high-interest loan — pay it continuously.** It's almost always cheaper to pay debt down in
small, regular increments than to let it accumulate and tackle it in a painful burst. The "we'll spend a
day a quarter cleaning up the slop" approach doesn't scale; by the quarter mark the slop has already spread
into everything built on top of it. Capture the standard once, enforce it continuously.

## How to apply it

- **Write the flop zones down, with a harness plan for each.** Don't keep "the agent is bad at design" in
  your head — name the zone and decide its controls. For the four above:
  - *Frontend/design* → a [design guide and evidence images](../20-harness/05-behaviour-harness.md), graded
    by a [separate evaluator](../10-lifecycle/05-verify-proof-not-vibes.md), never the agent that built it.
  - *Cross-cutting refactors* → a human owns the scope and reviews the sweep; lean on
    [fitness functions](../20-harness/03-sensors-feedback.md) to catch drift the diff hides.
  - *Ambiguous business rules* → pin the rule in a [normative spec](../10-lifecycle/01-spec-the-contract.md)
    before any code, and require human sign-off that the spec captured what was meant.
  - *Security-sensitive code* → mandatory human review, never auto-merge — the one zone where the gate is a
    person.
- **For each zone, decide: automate, guard, or keep human.** Automate the verifiable; guard the regressable
  with a sensor; keep human the subjective and the high-stakes. Make the call deliberately — the default of
  "trust it everywhere" is a decision too, just an unexamined one.
- **Earn autonomy zone by zone.** Tie this to [growing the harness](02-growing-the-harness.md): a zone earns
  more agent autonomy as its controls prove they catch mistakes. A flop zone keeps a human in the loop until
  it's harnessed well enough to let go — and some zones (security, money) may stay human indefinitely, on
  purpose.
- **Make AI debt visible — in the repo, not in someone's head.** Keep a tracked list of the known
  shortcuts, the stubbed corners, the areas flagged weak. A debt that lives in the
  [system of record](../00-foundations/02-context-engineering.md) can be scheduled; a debt that lives in a
  Slack thread is invisible the moment the thread scrolls away — and invisible to the agent entirely.
- **Schedule paydown on a continuous cadence.** Pay down a little every cycle rather than saving it for a
  cleanup sprint that never comes. Cleanup is itself good agent work — point the agent at the tracked debt
  on a regular rhythm and review the small diffs.
- **Keep a named human accountable for every merge.** "Who owns this if it breaks?" must always have an
  answer that is a person. Diffuse accountability is how silent debt is born.
- **Don't:** auto-merge on green across the board as if the agent were uniformly reliable; treat
  responsibility as something the model now carries; let debt accumulate for a someday cleanup; hand a
  security or business-rule change to the agent end-to-end with no human gate.

## In practice

A team has a fast, productive agent workflow and a habit they're proud of: if CI is green, it merges. It's
served them well on dozens of routine features. Then the agent picks up a change to billing — how a
subscription is prorated when a customer upgrades mid-cycle.

**Treating the agent as uniformly trustworthy.** The billing spec was a loose paragraph; it never pinned
down how proration should round, or what happens on a same-day upgrade. The agent filled the gap with a
plausible guess, and — because it also wrote the tests — those tests assert the guess. CI is green. Under
the merge-on-green habit, the change ships with the same glance a list endpoint would have got. The guess
was wrong, and customers are mischarged on upgrade. In the post-mortem the hardest question is *who owned
this?* — and the honest answer is no one did. The agent wrote it; the human waved it through on a green
check; the spec never said. The miss had no owner because responsibility had quietly been left with the
model.

**Staying accountable.** A team that has done the work of this page treats billing as what it is: an
ambiguous business rule *and* high-stakes money code — two flop zones at once. So it has a standing rule
that money logic is never auto-merged, and that its spec must state the rounding and the edge cases
normatively before any code is written. The vague paragraph gets sent back and pinned down first; a human
signs off that the pinned spec matches what the business actually wants; and the change gets a real human
review, not a glance. The wrong proration assumption surfaces in that review, while it's a paragraph and a
diff, not a billing run. The same team keeps the small shortcuts the agent took elsewhere in a tracked debt
list and pays them down a little each week, so the codebase stays one they can still change.

The lesson the example carries: the agent is not uniformly trustworthy, and pretending it is means
spending your least attention exactly where you needed the most. The defence isn't to trust the agent less
everywhere — it's to decide, on purpose and zone by zone, what's automated, what's guarded, and what stays
human. And whatever the agent writes, a person still owns the merge. Responsibility is the one thing the
harness can't externalise.

## Anti-patterns

- **Uniform auto-trust.** Merging on green everywhere, as if a billing change and a list endpoint carried
  the same risk. The agent's reliability is not uniform, so neither should your trust be.
- **"The model is responsible."** Treating accountability as transferred to the agent. A merge with no human
  owner is a miss with no one to catch it and a debt with no one to pay it.
- **The invisible debt.** Shortcuts and known-weak areas that live in people's heads instead of a tracked
  list. What isn't written down can't be scheduled, and [compounds silently](../40-anti-patterns/01-failure-modes.md)
  until the codebase is unchangeable.
- **The big cleanup day.** Letting debt batch up for a quarterly "slop sprint" instead of paying it down
  continuously. By the time the sprint arrives, the debt has spread into everything built on top of it.
- **The un-gated flop zone.** Handing the agent a security or business-rule change end-to-end with no human
  in the loop — full autonomy granted to exactly the zone that hasn't earned it.

---
[← Previous: Growing the Harness](02-growing-the-harness.md) · [Contents](../README.md) · [Next → Legacy and Brownfield](04-legacy-and-brownfield.md)

Related: [Legacy and Brownfield](04-legacy-and-brownfield.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md) · [Growing the Harness](02-growing-the-harness.md) · [Behaviour Harness](../20-harness/05-behaviour-harness.md) · [Verify — Proof, Not Vibes](../10-lifecycle/05-verify-proof-not-vibes.md) · [Spec — The Contract](../10-lifecycle/01-spec-the-contract.md) · [Why AI-Native](../00-foundations/01-why-ai-native.md)
