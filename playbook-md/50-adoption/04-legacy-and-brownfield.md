# Legacy and Brownfield

> The harness is hardest to build exactly where it's needed most: an old, untested codebase with no types, no boundaries, and no tests to lean on. You don't fix that with a big rewrite — you earn a foothold. Pin the current behaviour with characterization tests, draw one real boundary at a time, and spend your first effort on the highest-risk seams. The minimum viable harness assumed an empty repo; this is the version for a repo that's been around for ten years.

Breadcrumb: [Playbook](../README.md) › Adoption

## Story so far

The minimum-viable-harness page made one quiet, very convenient assumption: an empty repo, where every harnessability investment is nearly free. Most teams don't live there. They live in a *brownfield* — years old, sparsely tested, untyped in the corners that matter, with boundaries that exist only in someone's memory and possibly their resignation letter. And here's the cruel joke at the centre of this page: the harness is hardest to build exactly where it would help the most. The way in isn't a heroic rewrite. It's earning one foothold, on the seam that matters most, and widening from there.

## The principle

The [minimum viable harness](01-minimum-viable-harness.md) page made one quiet assumption: an empty
repo. Day one, no code to protect, every harnessability investment nearly free because there's nothing
to retrofit. That's the *easy* case — and it's not the case most teams are in. Most teams have a
**brownfield** codebase: years old, large, sparsely tested, untyped in the corners that matter,
boundaries that exist only in someone's memory. And here's the cruel part — **the harness is hardest
to build precisely where it would help the most.** The codebase that most needs guarding against a fast
agent is the one that gives you the least to guard it *with*.

You cannot close that gap by fiat. Pointing an agent at a 200,000-line untested codebase with "add
types and tests everywhere" is a rewrite in disguise, and it will fail the way rewrites fail. The
brownfield move is the opposite of big-bang: **earn a foothold, then widen it.** Three practices do
that work:

- **Characterization tests pin what the code does *now*.** Before you can safely change a tangled
  function, you need a test that captures its *current* behaviour — bugs and all — so that when the
  agent refactors it, you can tell whether behaviour *changed*. It's not a test of what the code
  *should* do (you may not even know yet); it's a snapshot of what it *does*, so change becomes visible
  instead of silent. This is the legacy-code equivalent of the [behaviour
  test](../20-harness/05-behaviour-harness.md): a net under the trapeze before anyone starts swinging.
- **Boundaries get drawn incrementally, one real seam at a time.** You don't reorganise the whole
  codebase. You find one place where two responsibilities are tangled, draw a single clean
  [boundary](../20-harness/02-repo-structure-and-legibility.md) there — an interface, a module edge, a
  typed contract — and make *that* seam legible and checkable. Then the next one. Each boundary you draw
  is a place the agent can now work safely; the untouched tangle around it can wait.
- **High-risk seams come first.** You will never harness all of it, so don't try. Spend your scarce
  early effort where a mistake is most expensive: the payment path, the auth check, the data migration,
  the [flop zone](03-responsible-team-and-ai-debt.md) where the agent is least trustworthy and the blast
  radius is largest. Harness the dangerous seams; let the sleepy, low-risk corners stay un-harnessed
  until they actually change.

The throughline: in a greenfield repo you bank [harnessability](../00-foundations/03-harness-engineering.md)
for free up front; in a brownfield repo you *buy* it, deliberately, seam by seam, starting with the
seams that are worth the most.

## Why it works

**A characterization test makes change visible — which is the precondition for any safe change.** The
thing that makes legacy code terrifying to touch is that you can't tell what your change broke, because
nothing told you what "working" looked like before. A characterization test fixes exactly that: it
records the current outputs, so the moment a refactor alters them, a test goes red and points at the
difference. You're not asserting the behaviour is *correct* — you're asserting it *didn't change
without you noticing*. That single property is what lets an agent loose on a gnarly function at all:
without it, every agent refactor is a coin flip you can't even see the result of; with it, an
unintended behaviour change [can't merge silently](../20-harness/03-sensors-feedback.md).

**Incremental boundaries beat the rewrite because the rewrite never lands.** The instinct in a messy
codebase is to fix it all at once — and that instinct has killed more projects than almost anything
else, because a big-bang rewrite is a long-lived divergence from a moving target (the same reason
[long-lived branches rot](../30-delivery/01-trunk-based-development.md), at the scale of a whole codebase). Drawing
one boundary at a time keeps every step small, shippable, and verified against the characterization
net. The codebase improves *while staying alive*, and each boundary immediately pays off: it's one more
seam where the agent can work inside a real contract instead of guessing at an implicit one.

**Triaging by risk is the only honest budget.** You have finite effort and an effectively infinite
codebase, so "harness all of it" isn't a plan, it's a wish. Risk-first is the allocation that survives
contact with reality: the auth seam, where a mistake is a breach, earns a characterization test and a
typed boundary today; the rarely-touched reporting helper earns nothing until the day someone changes
it. This is the same logic as [keeping quality left](../20-harness/04-keep-quality-left.md) — spend the
control where it pays — applied to *where in the codebase* you invest, not just *when in the pipeline*.

**And the agent is the lever that makes it affordable.** Characterization tests are tedious to write by
hand — which is exactly why they often don't get written. But "read this function and write tests that
capture its current behaviour" is a task an agent is genuinely good at, because it's archaeology, not
invention. [Using the agent to build the harness](02-growing-the-harness.md) turns the costliest part
of brownfield adoption — generating the safety net — into reviewable agent work, which is what makes the
whole approach practical instead of aspirational.

## How to apply it

- **Net before you trapeze.** Before changing any tangled code, have the agent write characterization
  tests that capture its *current* behaviour — feed in representative inputs, record the actual outputs,
  assert they don't change. You're freezing behaviour so a refactor's effects become visible, not
  certifying the behaviour is right. The net goes up *first*.
- **Pick the seam by blast radius, not by how annoying it is.** List the places a mistake hurts most —
  money, auth, data integrity, the [zones where the agent reliably
  flops](03-responsible-team-and-ai-debt.md) — and harness those first. The ugliest file isn't the
  priority; the *most dangerous* one is.
- **Draw one boundary, make it checkable, move on.** Take a single tangled seam, extract a clean
  interface or typed contract, and add the [fitness function](../20-harness/03-sensors-feedback.md) that
  keeps it from re-tangling. Resist widening the scope mid-seam — one boundary, landed and gated, beats
  five started. The [steering loop](02-growing-the-harness.md) applies here too: each boundary is a
  control you add in response to a real risk.
- **Let the un-dangerous tangle wait.** Code that's low-risk and rarely changes does not need a harness
  yet — un-harnessed is fine for what nobody's touching. Harness it the day it becomes a flop zone or a
  change target, not before. Spreading thin over everything buys you nothing where it counts.
- **Use the agent for the archaeology.** Generating characterization tests, mapping which modules call
  which, drafting the first boundary interface from observed usage — these are agent tasks. Describe the
  seam, have it produce the tests or the interface, and [review the
  output](../10-lifecycle/06-review-and-convergence.md). The expensive part of brownfield gets cheap.
- **Bank each win as permanent.** A seam you've characterized and bounded doesn't slide back — it's a
  standing increase in [harnessability](../00-foundations/03-harness-engineering.md) that makes the next
  change there cheaper forever. Brownfield adoption isn't a phase you finish; it's a ratchet you turn,
  highest-risk seam first.
- **Don't:** try to type or test the whole codebase at once (that's a rewrite, and it won't land); start
  refactoring before the characterization net exists; spend early effort on low-risk corners because
  they're easier; or treat "we'll harness it eventually" as covering the payment path *today*.

## In practice

A team inherits a six-year-old order-processing service: 80,000 lines, a 4% test suite that mostly
checks getters, no types on the core pricing logic, and a single 600-line function that calculates
what a customer owes. They want to start using an agent on it.

**The big-bang attempt.** The first instinct is to clean it up first: they task the agent with "add
type annotations and unit tests across the service." Three weeks later there's a 4,000-line branch that
won't merge — it's diverged from a codebase six other people kept shipping into, half the new tests
encode the *current bugs* as if they were intended (because nothing said otherwise), and nobody can
review a change that large with any confidence. The branch is quietly abandoned. The codebase is
exactly as un-harnessed as before, minus three weeks.

**The brownfield approach.** They start with one question: *where does a mistake hurt most?* The
answer is the 600-line pricing function — get it wrong and customers are overcharged. So that seam goes
first. The agent is pointed at the function with a narrow task: *write characterization tests that
capture what this returns today, for a spread of representative orders.* It produces a few dozen tests
pinning the current outputs — bugs included — and the team reviews and merges them. Now there's a net.
*Then* they let the agent refactor: extract the tangled pricing rules into a typed module with a clean
boundary, with the characterization suite proving on every change that the numbers didn't move. When
one refactor *does* shift a result, a test goes red and points right at it — and it turns out to be a
real bug the old code had, which they now fix *deliberately*, updating the test to the corrected value.
One seam is now typed, bounded, and covered. The reporting helper nobody touches stays untouched. Next
sprint, the auth check gets the same treatment. The service gets safer one dangerous seam at a time,
while staying alive the whole way.

The lesson the example carries: you don't harness a legacy codebase by fixing it all — you earn a
foothold on the seam that matters most, pin its behaviour so change is visible, draw one real boundary,
and turn the ratchet again. The rewrite tries to buy all the harnessability at once and lands none of
it; the brownfield approach buys it seam by seam, risk-first, and every increment is permanent.

## Anti-patterns

- **The harness-it-all rewrite.** Trying to add types and tests across the whole codebase in one
  push — a [big-bang](01-minimum-viable-harness.md) at codebase scale, which diverges from a moving
  target and never merges. Earn one seam at a time instead.
- **Refactoring without a net.** Changing tangled legacy code before any characterization test pins its
  current behaviour, so you can't tell what your change broke — every refactor a coin flip you can't see
  the result of. ([Sensors — Feedback](../20-harness/03-sensors-feedback.md))
- **Characterizing the bug as the spec.** Writing characterization tests and then mistaking them for
  *correctness* tests — they record what the code *does*, not what it *should*; when one goes red, ask
  whether the behaviour was right before, don't reflexively restore it.
- **Easy-corner-first.** Spending the scarce early harnessing effort on the low-risk, easy-to-test
  modules because they're pleasant, while the payment and auth seams — the ones a fast agent could
  actually break expensively — stay [un-harnessed](03-responsible-team-and-ai-debt.md).
- **The eventual harness.** Treating "we'll get to it" as if it covers the dangerous seams, so the
  highest-risk code keeps getting agent changes with no net under it indefinitely. Risk-first means the
  payment path is harnessed *today*, not eventually.

> **Next up — [Scoring an Existing Harness](05-scoring-an-existing-harness.md):** greenfield or brownfield, one fair question keeps surfacing — how good is the harness you've actually got? The foundations chapter left that one wide open. The final page closes it with a working answer: a five-layer rubric an agent can score any repo against, with evidence stapled to every number.

---
[← Previous: The Responsible Team and AI Debt](03-responsible-team-and-ai-debt.md) · [Contents](../README.md) · [Next → Scoring an Existing Harness](05-scoring-an-existing-harness.md)

Related: [The Minimum Viable Harness](01-minimum-viable-harness.md) · [Growing the Harness](02-growing-the-harness.md) · [The Responsible Team and AI Debt](03-responsible-team-and-ai-debt.md) · [Repo Structure and Legibility](../20-harness/02-repo-structure-and-legibility.md) · [Behaviour Harness](../20-harness/05-behaviour-harness.md) · [Harness Engineering](../00-foundations/03-harness-engineering.md)
