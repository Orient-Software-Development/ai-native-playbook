# Review and Convergence

> Agent code is fluent, and fluency is exactly what hides the bug — plausible code that does the wrong thing sails past a skim. Read the diff like you mean it, and as the same pattern lands a second and third time, pull the shared piece out *before* the copies drift. Review keeps wrong code out; convergence keeps the right code from rotting.

Breadcrumb: [Playbook](../README.md) › Lifecycle

## The principle

Two jobs happen after a change is written and [verified](05-verify-proof-not-vibes.md), and
both exist to stop quality eroding as agents produce *volume*.

**1. Deliberate diff review.** Read the diff like it matters. Agent-written code is fluent —
it compiles, it reads well, it looks like something a careful engineer wrote — and that
plausibility is precisely what hides **zombie code**: code that looks right and does the
wrong thing. A dead branch that's never reached, a condition that's always true, a handler
that silently swallows the case it was meant to handle, a path nobody asked for. A skim
waves it through because it *reads* fine; only a deliberate read — tracing what the code
actually does, not what it appears to do — catches it.

**2. Convergence.** A feature rarely lands in one shot; it lands and then gets polished — a
fix, a tweak, a second near-copy of almost-the-same thing. Treat that follow-up work as
*convergence*: each pass should make the codebase **more coherent**, not just bolt more on.
The signal to watch is duplication, and the timing rule is the **rule of three** — the
first time you write a pattern, fine; the second, note it; **before the third copy, extract
the shared piece** into one named place. Pull it out while it's still three small similar
blocks, not after it's a dozen drifting ones.

They share a page because they share a cause: at agent throughput, code accumulates faster
than ever, and so does **entropy** — the slow drift toward a messier, more duplicated,
harder-to-change codebase. Review is the gate that keeps wrong code *out*; convergence is
the garbage-collection that keeps the right code from rotting into a pile of near-duplicates.
Volume makes both non-optional.

## Why it works

The agent's strength — producing fluent, plausible code — is the reviewer's trap. A
human's bug usually *looks* like a bug: a typo, an obvious gap. An agent's bug looks like
working code, because the model is optimised to produce text that reads as correct. So
"looks fine" is the *least* reliable signal you have on agent output, and the review has to
be deliberate: trace the logic, not the prose. And it can't be the author who reviews — for
the same reason a feature can't [self-verify](05-verify-proof-not-vibes.md), the process
that wrote the plausible code is biased to read it as correct.

Convergence works because duplication isn't expensive when you *create* it — it's expensive
later, every time the copied thing has to change and you must find and fix every copy, and
miss one. The rule of three is a timing rule with a reason on each side. Wait too long and
the copies drift — each picks up its own special case — until there's no clean shared piece
left to extract; the duplication is baked in. Extract too *early*, at the first copy, and
you're guessing at the shared shape from a single example and will abstract the wrong thing.
Three is the sweet spot: enough examples to see what's genuinely common, few enough that
they haven't diverged yet.

Both compound at scale, which is why they earn their own step. One unreviewed zombie is a
bug; a *habit* of skimming is a codebase you can't trust. One duplicated block is nothing; a
codebase of near-duplicates is one neither human nor agent can change safely, because every
edit ripples in ways no one can predict. The discipline is cheap per change and its absence
is ruinous in aggregate.

## How to apply it

**Reviewing:**
- **Read the diff deliberately** — trace the logic, don't skim the prose. For each change
  ask: what does this actually do on the inputs the spec cares about, including the edge
  ones?
- **Hunt for zombies:** dead branches, always-true conditions, swallowed errors, handlers
  for cases that can't occur, plausible code that's unreachable or inert. The test isn't
  "does it look reasonable" — it's "where in the spec is this, and does it do that."
- **Someone other than the author reviews** — a teammate, or a fresh agent with the spec —
  the same generator/evaluator split that makes verification trustworthy.
- **Lean on small slices.** A reviewable diff is the entire payoff of
  [task slicing](03-task-slicing.md); a wall of code can't be read deliberately, so don't
  produce one.

**Converging:**
- **Watch duplication and apply the rule of three.** First copy: fine. Second: notice.
  Third: extract the shared piece into one named place *before* you write it.
- **Don't extract too early.** One copy isn't a pattern. Abstracting from a single example
  builds a leaky abstraction everything else has to bend around.
- **Treat polish passes as convergence, not accretion** — each follow-up should leave the
  code more coherent than it found it. Garbage-collect: delete the dead, fold the
  duplicated, before the entropy compounds.
- **Don't:** rubber-stamp a green diff; trust "looks right" on agent code; or copy-paste a
  fourth time because it's faster in the moment.

## In practice

Take the *per-user deal priority* feature — verified and ready to merge. Two things surface.

**A zombie.** The deliberate read of the query slice hits a branch nobody mentioned:

```text
if (user.role === 'admin') {
  // show all teams' flagged deals
  return allFlagsAcrossTeams(...)
}
```

Nothing in the spec asked for this — REQ-002 says flags are private, full stop. The agent
invented a plausible-sounding admin path; it compiles, the tests never hit it, and it
quietly breaks the privacy contract for one class of user. A skim sails right past it
("looks like reasonable admin handling"). The deliberate read asks the one question that
matters — *where in the spec is this?* — finds the answer is *nowhere*, and deletes it.
That's a zombie: plausible, inert-looking, and wrong.

**Convergence.** A sprint later, two more features want the same shape of thing: a
*follow-up* flag on deals and a *watched* marker on accounts. Each arrives as a near-copy of
the deal-priority pattern — a per-user link table, a flag/unflag command pair, a join in the
list query. The first copy (deal priority) was fine. On the second (follow-ups) you *notice*
the repeated shape. The third (watched accounts) is the trigger: before writing it as a
third copy, you extract a small **per-user toggle** helper that all three call, while they're
still three similar blocks. Wait until the fifth and they'd each have drifted — one adds a
timestamp, one a note — and there'd be no clean thing left to pull out. Extracting at three
is a small refactor; extracting at twelve is a project nobody schedules, so it never happens
and the drift just grows.

## Anti-patterns

- **The rubber-stamp review.** Approving a green diff without really reading it — plausibility
  waved through, zombie code shipped. The headline failure this page prevents.
  ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **"Looks right" on agent code.** Trusting fluency as correctness — the single least
  reliable signal on generated code, because the model is built to make wrong code read as
  right.
- **Premature extraction.** Abstracting at the first copy, guessing the shared shape wrong,
  and leaving a leaky abstraction everything has to bend around.
- **Copy-paste creep.** The opposite failure — never extracting, letting near-duplicates pile
  up until the codebase can't be changed safely.

---
[← Previous: Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md) · [Contents](../README.md) · [Next → Guides — Feedforward](../20-harness/01-guides-feedforward.md)

Related: [Task Slicing](03-task-slicing.md) · [Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md) · [Guides — Feedforward](../20-harness/01-guides-feedforward.md) · [Trunk-Based Development](../30-delivery/01-trunk-based-development.md) · [Growing the Harness](../50-adoption/02-growing-the-harness.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
