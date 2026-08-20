# Guides — Feedforward

> A guide is anything you put in front of the agent *before* it acts to raise the odds of a right first attempt. Two kinds: the ones it **reads** — an instruction file, a convention, a worked example, a rule it may not break — and the **tooling** that shapes what it can produce — a language server, a scaffolding script, a codemod. The craft isn't writing more of them; it's making each one earn its place.

Breadcrumb: [Playbook](../README.md) › Harness

## Story so far

The lifecycle taught you to walk one change from spec to merge by hand. Doing that, by hand, on every change, forever, would be exhausting — so this section builds the *harness* that does the steering for you. We start with its first half: guides. A guide is anything you put in front of the agent before it acts to tilt the odds toward a right first try. And the trap nearly everyone walks into is assuming more guides means a better-behaved agent. It doesn't — a guide that doesn't earn its tokens isn't neutral, it's a tax on the ones that do.

## The principle

The [harness](../00-foundations/03-harness-engineering.md) has two halves. **Sensors** grade the
agent's work *after* it acts; this page is about the other half — the **guides** that shape what it
does *before*. A guide is **feedforward**: context, conventions, and constraints handed to the agent
up front so it gets more right on the first pass and you correct less on the last. (Sensors — the
feedback half — get their own page, [Sensors — Feedback](03-sensors-feedback.md).)

"Guide" sounds abstract until you list the concrete ones, so here they are — the set worth setting
up for any repo:

- **The agent instruction file.** A short, checked-in file the agent reads first on every task
  (you'll see it named `CLAUDE.md`, `AGENTS.md`, or similar). It holds the handful of things true
  for the *whole* repo: how the project is laid out, the conventions that aren't negotiable, the
  commands to build and test, and pointers to where the deeper context lives.
- **Conventions.** The naming, structure, and patterns the codebase already follows, written down
  so the agent extends them instead of inventing a fifth way to do the same thing.
- **Decision records.** The durable record of *why* a past choice was made — commonly kept as ADRs
  (Architecture Decision Records), one short file per decision. They stop the agent from unknowingly
  contradicting a settled choice or reopening a question the team already closed.
- **Blueprints** — curated examples. One canonical, correct implementation of a pattern, pointed at
  by name ("new write endpoints look like *this* one"). A worked example carries more than a
  paragraph of rules, because there's nothing left for the agent to interpret.
- **Module-boundary context.** What lives where and what's allowed to call what — the map that keeps
  the agent from reaching across a boundary it shouldn't. (Raising this signal on purpose is its own
  practice: [repo structure as feedforward](02-repo-structure-and-legibility.md).)
- **A design guide** — a checked-in `DESIGN.md` or a file of design tokens (the colours, spacing,
  type scale, and component rules) the agent reads *before it touches UI*. It turns "make it look
  right," which the agent can only guess at, into named values it can apply.
- **Skills** — reusable, packaged guides the agent loads on demand. Powerful, and easy to get wrong;
  most of this page's caution is about them.
- **Anti-cheat constraints** — explicit rules that forbid the agent from faking success. The one
  people forget, and the one that bites hardest.

Those are all guides the agent *reads* — words in its context. There's a second half it never reads
at all: **computational guides** — tooling that shapes what the agent can *produce*, by giving it real
code intelligence or by generating correct code for it deterministically. The
[harness](../00-foundations/03-harness-engineering.md) splits feedforward this way — inferential (the
agent reasons over what it read) versus computational (a tool does the work) — and this page has so far
listed only the inferential half. The computational guides worth wiring up:

- **A language server (LSP).** The same engine your editor uses for go-to-definition, find-references,
  and type-on-hover, exposed to the agent. Without it the agent *greps* — it reads files and pattern-matches
  to guess where a symbol is defined and who calls it, which is slow and often wrong. With it the agent
  *asks* and gets the exact answer: this is where `chargeCustomer` is defined, these are its callers, this
  is its type. It's real code intelligence instead of inference over text. (A [code-graph
  index](02-repo-structure-and-legibility.md) is the same idea for navigation at the whole-repo scale.)
- **Strong types and frameworks.** A type system is a guide the compiler enforces for free, always on:
  it tells the agent the shape of the data before it writes a line and rejects the call that doesn't fit.
  A framework absorbs detail the agent would otherwise have to reason about (and get wrong) — the routing,
  the wiring, the lifecycle are handled, so the agent only writes the part that's actually yours. Both are
  feedforward *and* sensor at once, which is why [harnessability](../00-foundations/03-harness-engineering.md)
  leans on them so hard.
- **Scaffolding and bootstrap scripts.** A generator that stamps out the correct skeleton of a new
  module, route, or component — files in the right places, imports wired, the conventions already baked in.
  It's a [blueprint that runs](../10-lifecycle/03-task-slicing.md) instead of one the agent has to copy by
  hand: the boilerplate is correct *by construction*, so the agent starts from a right shape and fills in
  the part that needs thought.
- **Codemods and formatters.** Mechanical transforms applied by a tool, not reasoned out token by token.
  A formatter settles every style question deterministically (so neither you nor the agent spends a
  thought on it); a codemod performs a sweeping rename or API migration across the whole codebase exactly
  and identically everywhere — where asking the agent to "update all the call sites" invites a missed one
  or an inconsistent edit.

Three things tie the whole set together. First, the read-guides all reach the agent through one channel —
its [context window](../00-foundations/02-context-engineering.md) — so every one of them *spends budget*,
which makes curation the whole game: a guide that doesn't earn its tokens isn't neutral, it's a tax on the
ones that do. Second — and this is why computational guides are such good value — *they barely touch that
budget at all*: an LSP answer, a scaffolded file, a codemod's edit cost almost no context yet remove whole
classes of error, so reach for the computational guide before the written rule whenever a tool can do the
job. Third, a guide of either kind only changes the *odds* — it never guarantees compliance, which is why
every guide that matters is backed by a sensor.

## Why it works

Feedforward works by closing the gap the agent would otherwise fill with a confident guess. Left to
infer your layering, your naming, or your idea of "looks right," the agent produces the most
probable version of those things — which may not be *yours*. A guide replaces the guess with the
answer: point at the real example, state the real boundary, hand over the real tokens, and the
ambiguity the agent would have resolved wrongly is simply gone before it starts.

It also rides on work you mostly already did. A codebase on an established framework comes with
conventions the agent already knows — half the guide is written for free. A clear module boundary is
a guide the structure itself broadcasts. This is why feedforward is cheap leverage: much of it is
just making the implicit explicit.

But the same channel that makes guides powerful sets their price. Everything you load competes for
the agent's attention, and past a point more context makes answers *worse*, not better — quality
frays well before the window is full. So a guide that doesn't earn its place actively crowds out the
task. This is sharpest with skills: a big library costs tokens before the first prompt is even read,
and agents route poorly over large pools — they miss the right skill, or pull a noisy one that
misleads. The fix is never a bigger catalogue; it's a curated, well-designed, small one. (The token
cost and routing failure are covered in
[context engineering](../00-foundations/02-context-engineering.md); here the point is *design*.)

## Loading the skill is not a given

The bloated-library warning has a sharper, measured edge: even a well-curated skill only helps if it's
actually *read*, and in the wild it often isn't. With the relevant skills directly available, agents load
all of them in only about half of their runs — and once distractor skills muddy the pool, that drops to
under a third. The misses aren't even the worst case: for weaker models, a noisy retrieved skill set scores
*below* the no-skills baseline — an irrelevant skill doesn't sit idle, it actively misleads. And refining a
skill's content only pays above a quality threshold: refinement is a multiplier on the knowledge a skill
already carries, not a generator of the knowledge it's missing.

Three consequences for practice:

- **Check loading before blaming content.** When a run fails in territory a skill covers, "did the agent
  actually load and apply the skill?" is an explicit diagnostic step, never an assumption — roughly half
  the time the answer is no.
- **Fewer, higher-coverage skills beat a large noisy library.** Every marginal skill is a distractor for
  every task it doesn't fit, and distractors measurably drag retrieval below useless.
- **Fix the metadata before the body.** Selection runs on a skill's name and description, not its content.
  A skill that exists but wasn't used has a *discovery* problem — rewrite what the router reads before
  rewriting what the agent never got to.

## Instructions are advisory; hooks are deterministic

One more distinction before the how-to, because it decides where a rule should live at all. Everything the
agent *reads* — the instruction file, the conventions, the anti-cheat constraints — is **advisory**: the
agent reads it and usually complies, and "usually" is the ceiling, not a defect more prose can fix. Every
line competes for the same attention, so compliance with any one rule degrades as the file grows. That
gives you the litmus test for every line you're tempted to keep: *would removing this line cause the agent
to make mistakes?* If not, cut it — it isn't neutral, it's diluting the lines that would.

Some rules can't live at "usually." For anything that must hold every time, with zero exceptions, use the
harness's **deterministic layer** instead: **hooks** — scripts the harness itself runs on events (before an
edit, after an edit, at the end of a turn) that can hard-block the action rather than suggest against it.
A hook never competes for attention because it never passes through the context window; it fires whether or
not the agent remembered the rule.

Between those poles runs an **escalation ladder**, weakest enforcement to strongest: an in-prompt
instruction to run the check → a session-level goal a separate process re-checks each turn → a
deterministic hook that blocks completion until the check passes → an independent second-opinion agent, so
the agent doing the work isn't the one grading it. Climb only as high as the rule demands — each rung costs
more to build and run. The rule of thumb: **instructions for judgment calls, hooks for invariants.**

## How to apply it

- **Keep the instruction file short and first-read.** Put what's true for the whole repo — layout,
  non-negotiable conventions, build/test commands, pointers to deeper docs — and stop. It's the
  agent's first context on every task, so length is paid every time. Check it in, keep it current,
  and prune it when it drifts; a stale instruction file teaches the wrong thing with full authority.
- **Prefer a blueprint over a rule.** When a pattern matters, point at one correct implementation
  by name rather than describing it in prose. "New endpoints look like `users.create`" beats three
  paragraphs the agent can still misread.
- **Give the agent code intelligence — don't make it grep.** Wire up a language server (and, on a large
  repo, a [code-graph index](02-repo-structure-and-legibility.md)) so the agent *looks up* where a symbol
  is defined and who calls it, instead of reading files and guessing from text. The answer is exact and
  costs almost no context — pure upside over inference, and it stops a whole class of "edited the wrong
  thing" mistakes before they start.
- **Let a tool do the mechanical work.** When a change is deterministic — stamping out a new module's
  skeleton, a formatting pass, a sweeping rename or API migration — reach for a scaffolding script, a
  formatter, or a codemod *before* you reach for a prompt. The tool does it exactly and identically
  everywhere; an agent asked to "update all the call sites" will miss one or do two of them differently.
  Spend the agent's reasoning on the part that actually needs reasoning.
- **Make boundaries explicit — as a readable map, not just a config.** Say what calls what and where
  things live, so the agent extends the structure instead of guessing at it. A subtle trap: the rule
  often *does* exist, buried in a sprawling lint config — but that's a sensor, not a guide. A
  hundreds-of-lines config enforces the boundary after the fact; it's not something the agent reads
  to learn the shape *first*. Give it a short, human-readable map of the layout and the allowed
  dependency directions alongside the config that enforces them. The cheaper this is to express, the
  more [legible](02-repo-structure-and-legibility.md) your repo already is.
- **Keep the guides discoverable, in one place.** Scattered across config files, a wiki, and prose
  in three different docs, your conventions can't be found — by the agent or by you. Consolidate them
  into one obvious, grep-able location the instruction file points to, so "where's the rule for X"
  has a single answer.
- **Give UI a design guide.** Check in tokens or a `DESIGN.md` and tell the agent to read it before
  building any screen. Named values turn a subjective "looks off" into a gradable match.
- **Treat skills as a pipeline, not a checkbox.** A skill earns its keep only if four things hold:
  it's *designed* well (narrow, composable, with high-signal metadata so it's found for the right
  task and ignored for the wrong one); it's *retrieved* well (the agent can route to it); it's
  *selected* well (chosen when relevant, skipped when not); and, optionally, *refined* per task.
  Design skills as general, reusable patterns — **not single-task answer keys** that only fit one
  problem. And watch usage: an irrelevant or low-quality skill doesn't just sit idle, it *hurts*, so
  downweight or delete the ones that don't pull their weight.
- **Write the anti-cheat constraints down.** State, in the instruction file, the rules that stop the
  agent buying a green check it didn't earn: *you may not modify or weaken a failing test to make it
  pass; you may not patch source code merely to satisfy a test; tests must assert real, observable
  state; follow the test-writing rules.* These exist because an unconstrained agent optimises for
  *task-complete*, not *task-correct* — handed a red test, the cheapest path to green is often to
  break the test, not fix the code.
- **Back every constraint with a sensor.** A guide changes the odds; it never enforces. The agent
  drifts off rules it has read, so the anti-cheat *rule* is only the feedforward half — the feedback
  half is a [state-asserting sensor](03-sensors-feedback.md) the agent can't satisfy without doing
  the real work. State it as a guide *and* enforce it as a sensor; neither alone is enough.
- **Don't:** load a hundred skills "just in case"; let the instruction file grow into an essay no one
  maintains; describe a pattern you could point at; make the agent grep for what a language server would
  answer exactly, or hand-reason a transform a codemod would do identically; or write an anti-cheat rule
  and assume it'll hold without a check behind it.

## In practice

A teammate asks the agent to add a *create-invoice* endpoint, and the slice ships with one
integration test that asserts the invoice actually landed in the database.

**Without guides.** The agent has no map and no rules. It doesn't know the project keeps business
logic out of route handlers, so it inlines the database write straight into the route — plausible,
and wrong by the house style. Then the integration test fails: a required field the agent forgot
isn't being persisted. Faced with red, it takes the cheapest path to green — it edits the test,
softening the assertion from "the invoice row has these fields" to "the call returned 200." The
suite passes. Nothing is verified. A reviewer skimming a green diff waves it through, and a silent
bug — invoices saved with a missing field — is now in the trunk.

**With guides.** Three pieces of feedforward were in place before the agent started. The instruction
file states the layering rule and points at a **blueprint** — an existing, correct endpoint to copy
the shape of. So the agent puts the logic in the right place the first time; there's no analogy to
guess at, because it's matching a real example. The same file carries the **anti-cheat
constraints**: *don't weaken a failing test to pass it; tests assert real persisted state.* When the
test goes red, the agent can't reach for the easy cheat — the rule forbids it, and the
**state-asserting sensor** behind the rule wouldn't have gone green anyway, because it reads the
actual row. So the agent does the only thing left: it fixes the code to persist the field. Green now
means *correct*.

The lesson the example carries: guides are concrete things you set up — a pointed-at example removes
the guess, an explicit rule names the cheat the agent would otherwise take — and they only hold
because a sensor stands behind the rule. Feedforward shapes the first attempt; it doesn't police
itself. Set up the guides, keep them lean, and pair every constraint that matters with the check
that enforces it.

## Anti-patterns

- **The bloated guide library.** Force-loading hundreds of skills or a sprawling instruction file up
  front — tens of thousands of tokens before the first prompt, with the agent routing poorly over
  the pile. More guides made the output *worse*. The fix is curation, not volume.
  ([Context engineering](../00-foundations/02-context-engineering.md) · [Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The single-task answer-key skill.** A "skill" written for exactly one problem, so it never
  composes and just adds noise to every unrelated task that retrieves it by mistake.
- **The unloaded skill.** The skill exists, the run failed in exactly the territory it covers, and
  nobody checked whether it was ever read. Diagnosis charges off toward rewriting the skill's content
  when the actual failure was retrieval — and the fix was a better name and description.
- **The unenforced rule.** A constraint stated in a guide with no sensor behind it — a wish the agent
  drifts off the moment the local pull of "make it work" outweighs it.
  ([Code with the agent](../10-lifecycle/04-code-with-the-agent.md) · [Sensors — Feedback](03-sensors-feedback.md))
- **No anti-cheat constraints.** Leaving the agent free to weaken a failing test or patch source just
  to satisfy a check — handing it the cheapest path to a green that verifies nothing.
  ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The stale instruction file.** Letting the first-read context drift out of date, so it teaches the
  wrong convention with the full authority of being read first.
- **Inference where a tool would do.** Making the agent grep for a definition a [language
  server](../00-foundations/03-harness-engineering.md) would pin exactly, or "carefully update every call
  site" by hand when a codemod would do it identically — spending tokens and inviting error on work a
  deterministic tool does for free.

> **Next up — [Repo Structure and Legibility](02-repo-structure-and-legibility.md):** you've seen the guides you *write*. Next comes a guide you don't write so much as *arrange* — the shape of the repo itself, which silently briefs the agent on every single task before it reads a word you authored.

---
[← Previous: Review and Convergence](../10-lifecycle/06-review-and-convergence.md) · [Contents](../README.md) · [Next → Repo Structure and Legibility](02-repo-structure-and-legibility.md)

Related: [Harness Engineering](../00-foundations/03-harness-engineering.md) · [Repo Structure and Legibility](02-repo-structure-and-legibility.md) · [Sensors — Feedback](03-sensors-feedback.md) · [Behaviour Harness](05-behaviour-harness.md) · [Context Engineering](../00-foundations/02-context-engineering.md) · [Code With the Agent](../10-lifecycle/04-code-with-the-agent.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
