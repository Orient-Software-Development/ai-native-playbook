# The Minimum Viable Harness

> Don't wait for the perfect harness — stand up the smallest one that already closes the loop. One guide file, a formatter and linter, a type check, a gate that blocks on red, and one real behaviour test. You can build it in week one, and it beats a grand plan you never finish.

Breadcrumb: [Playbook](../README.md) › Adoption

## The principle

A **minimum viable harness** is the smallest set of controls that still does the whole job: shape the
work before the agent acts, grade it after, and refuse to let red merge. Everything earlier in this
playbook describes a *full* harness — layered sensors, design guides, fitness functions, behaviour
contracts. That's the destination, not the on-ramp. This page is the on-ramp: what you put in place on
day one of a new project, before the codebase even has a shape, so the very first agent-written change
lands inside a loop instead of outside one.

Five pieces, one of each kind that matters:

1. **One guide file** — a single checked-in [agent instruction file](../20-harness/01-guides-feedforward.md)
   (`CLAUDE.md`, `AGENTS.md`, whatever your tool reads first) holding the handful of things the agent
   must know before it touches anything: where the code lives, the non-negotiable conventions, and the
   exact commands to build and test. This is your [feedforward](../00-foundations/03-harness-engineering.md) —
   the work you do *before* the agent acts to raise the odds of a right first attempt.
2. **A formatter and a linter** — the cheapest [sensors](../20-harness/03-sensors-feedback.md) there are.
   They catch style drift and a whole class of obvious mistakes deterministically, in milliseconds, with
   no judgement call.
3. **A type check** — if your language has one, this is the single highest-signal free sensor you will
   ever get. A type checker is a proof, run on every change, that whole categories of bug *cannot* be in
   the diff. If your language is untyped, skip this line and lean harder on the test.
4. **A gate that blocks on red** — a [pre-push hook](../20-harness/04-keep-quality-left.md) (or the
   equivalent earliest stage you can enforce) that runs those checks and *refuses the push* when any of
   them fail. A check that only warns is not a gate. The gate is what turns "we have a linter" into "red
   does not get in."
5. **One real behaviour test** — a single end-to-end test that proves the app's one most important thing
   actually works: a user can log in, an order saves to the database and reads back, the core calculation
   returns the right number. Not a test of what the code *does* — a test of what the product *needs*. This
   is the seed of your [behaviour harness](../20-harness/05-behaviour-harness.md), and it's the piece teams
   skip first and regret first.

That's the whole list. Notice what's *not* on it: no fitness functions, no mutation testing, no AI
reviewer, no design guide, no CI matrix. Those are all real, and they all come later (the
[next page](02-growing-the-harness.md) is about adding them). Week one is about getting *one of each
control type* working end to end, not getting any of them complete.

## Why it works

A harness is only a harness when the loop closes — feedforward shapes the attempt, a sensor grades it,
and a gate enforces the verdict. Miss any one of the three and you don't have a small harness, you have
no harness:

- **Guides with no gate** is a suggestion box. The agent reads your conventions, ignores half of them
  under pressure (it will — [it doesn't reliably self-follow rules](../10-lifecycle/04-code-with-the-agent.md)),
  and nothing stops the result.
- **Sensors with no gate** is a dashboard nobody reads. The linter is red, the build is broken, and the
  change merges anyway because failing was never *blocking*.
- **A gate over nothing** blocks on green checks that prove nothing — the [vacuous sensor](../40-anti-patterns/01-failure-modes.md)
  that passes whether or not the work happened.

The minimum viable harness is the smallest configuration where all three are present at once. It's
weak — one behaviour test covers one path, the linter misses semantic bugs — but it is *whole*. And a
whole-but-weak loop is the thing you can actually improve, because the [steering loop](02-growing-the-harness.md)
that grows a harness needs a harness to grow: you watch it miss something, you add the guide or sensor
that would have caught it, you re-run. You can't steer a loop that doesn't exist yet.

There's a second reason to start small, and it's the same lesson as
[keeping a hook fast enough that nobody skips it](../20-harness/04-keep-quality-left.md): **a harness the
team actually adopts beats a bigger one they route around.** A five-line guide file gets read and kept
current; a five-page one goes stale and gets ignored. A pre-push gate that runs in seconds gets left on;
one that takes minutes gets bypassed with `--no-verify` by the end of the first week. Minimum viable isn't
a compromise you'll grow out of — it's the only version that survives contact with a real team under a real
deadline.

And there's a timing argument: **an empty new repo is the cheapest moment you will ever have to do this.**
Choosing a typed language, drawing module boundaries, wiring the first gate — all of that is nearly free
before there's code, and expensive to retrofit after. [Harnessability](../00-foundations/03-harness-engineering.md) —
how readily a codebase accepts guides and sensors — is something you bank early or pay for forever. Week
one is when the down payment is smallest.

## How to apply it

Work the checklist top to bottom. Each item is one sitting, and the whole thing is a day, not a sprint.

- **Write the guide file first, and keep it short.** Repo layout in a few lines, the conventions you
  refuse to compromise on, the literal build and test commands, and a pointer to where deeper context
  lives. The test for every line: *would the agent get this wrong without it?* If not, cut the line — a
  bloated guide file is a [tax paid on every task](../00-foundations/02-context-engineering.md). A skeleton:

  ```markdown
  # <Project> — agent guide

  ## Layout
  - app code: <where>
  - tests: <where>
  - specs/contracts: <where>

  ## Conventions (non-negotiable)
  - <the 3–5 rules you will not bend>

  ## Commands
  - build:  <command>
  - test:   <command>
  - check:  <lint + types + test, the one command the gate runs>
  ```

- **Wire the formatter, linter, and type check as one command.** Make a single entry point — call it
  `check` — that runs all three and exits non-zero if any fail. One command is what the guide file points
  at, what the gate runs, and what a human runs by hand. Don't make the agent (or yourself) remember three.

- **Put the gate at the earliest stage you can enforce, and make it block.** A pre-push hook is the usual
  sweet spot: fast enough to run on every push, enforced before code leaves the machine. The non-negotiable
  property is that **a failing check fails the push** — warn-only is not a gate. (When you add a shared CI
  server later, the *authoritative* gate moves there, where it [can't be bypassed with a local flag](../20-harness/04-keep-quality-left.md);
  the hook stays as the fast local courtesy. Week one, the hook is all you need.)

- **Write one behaviour test against the spec, not the code.** Pick the single most important thing the
  product must do and prove it end to end — and assert the [observable end-state](../20-harness/03-sensors-feedback.md),
  the row that persisted or the value the user sees, never just "it ran without throwing." This one test is
  worth more than a hundred that confirm what the code already does. Add it to the `check` command so the
  gate runs it too.

- **Do:** keep every piece small enough to finish in week one; make the gate block, not warn; point the
  guide file at the same `check` command the gate runs, so there's one source of truth for "passing."
- **Don't:** wait for the "real" harness before shipping the first feature — the first ungated agent PR is
  exactly what you're trying to prevent; pile every check you can think of onto day one (you don't know yet
  what this repo needs — you'll learn that by [watching it fail](02-growing-the-harness.md)); skip the
  behaviour test because lint and types are "probably enough" — they check that the code is *clean and
  well-typed*, never that it does the *right thing*.

## In practice

Two teams start the same new service in the same week.

**Team A ships first, harnesses later.** The reasoning is reasonable: the harness is overhead, and there's
no code to protect yet, so why slow down? They point the agent at an empty repo with a one-line prompt and
start building features. It's fast and it feels great. By the end of week two there are forty files, no
guide file, no gate, and a linter someone installed but nobody runs. The agent has been quietly inventing
its own conventions — three different ways to talk to the database, two naming schemes — because nothing
told it which was right and nothing rejected the ones that weren't. The first real bug is a feature that
returns the wrong total; it passes review because it *looks* right and there's no test that would have
caught it. Now the team faces the expensive version of every week-one task — retrofitting types onto
forty files, reconciling three conventions, writing the gate that should have been there before the mess
existed. The harness arrives as cleanup instead of scaffolding.

**Team B spends day one on the minimum viable harness.** Before the first feature, they write a short guide
file (layout, five conventions, the `check` command), wire format-lint-type into that one command, add a
pre-push hook that blocks on red, and write a single end-to-end test that proves a record saves and reads
back. *Then* they start building. The agent reads the conventions on every task and the gate rejects the
pushes that ignore them, so the database is talked to one way from the first commit. When the agent writes
a feature that returns the wrong total in week two, the behaviour test — extended a little as the product
grew — goes red before the push lands, and it's fixed while it's one small diff. Team B isn't slower for
the day they spent; they're faster by week two, because every change since day one landed inside a loop
that caught its own mistakes.

The lesson the example carries: the harness is not overhead you add once there's something to protect — it
*is* the thing that keeps there from being a mess to protect against. The cost of standing up the minimum
viable harness is a day at the start. The cost of skipping it is paid back, with interest, as cleanup —
and some of it (the conventions that diverged, the bug that shipped) you don't get to pay back at all.

## Anti-patterns

- **The someday harness.** Waiting for the perfect, complete setup before standing up *any* of it — so the
  perfect harness never ships and the first agent PRs land completely ungated. The fix is this page: small
  and whole beats big and someday.
- **The big-bang harness.** The opposite mistake — trying to wire fitness functions, mutation testing, an
  AI reviewer, and a full CI matrix on day one. It stalls (none of it lands in week one) and it's
  over-engineered for a repo that has no shape yet to protect. Ship one of each control type; [grow the
  rest](02-growing-the-harness.md) by watching what actually fails.
- **The half-loop.** Guides with no gate (a suggestion box), or sensors with no gate (a dashboard nobody
  reads). Either way the loop doesn't close and nothing is enforced. All three — feedforward, sensor, gate —
  or it isn't a harness.
- **The decorative gate.** A hook or check that runs and *warns* but doesn't block, so red still merges.
  A gate that can't say no is set dressing.
- **The lint-only start.** Format, lint, and types but no behaviour test — so the harness proves the code
  is clean and well-typed and never once proves it does what the product needs. The [behaviour test is the
  piece](../20-harness/05-behaviour-harness.md) that's hardest to add later and easiest to skip now; skip it
  and you've built a harness that's blind to the only failure that matters to a user.

---
[← Previous: Failure Modes](../40-anti-patterns/01-failure-modes.md) · [Contents](../README.md) · [Next → Growing the Harness](02-growing-the-harness.md)

Related: [Guides — Feedforward](../20-harness/01-guides-feedforward.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Keep Quality Left](../20-harness/04-keep-quality-left.md) · [Behaviour Harness](../20-harness/05-behaviour-harness.md) · [Harness Engineering](../00-foundations/03-harness-engineering.md) · [Growing the Harness](02-growing-the-harness.md)
