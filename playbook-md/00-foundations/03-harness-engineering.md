# Harness Engineering

> A coding agent is a model plus a harness — and the harness is everything that *isn't* the model. It's the **guides** that shape what the agent does before it acts and the **sensors** that grade what it produced after. Engineering that harness — not picking the model — is what makes agent output reliable. 

Breadcrumb: [Playbook](../README.md) › Foundations

## Story so far

We've established that an agent only knows what's in its context window, and that you should treat that window like a tight budget rather than a backpack. Useful — but context is just the *channel*. This chapter is about the thing you actually build and push down it. It has a name, the **harness**, and it's the single highest-leverage thing an AI-native engineer owns. Two moving parts: guides that brief the agent before it acts, and sensors that grade it after. That's the whole machine — let's pop the casing.

Page 01 made one claim above all others: the payoff comes from the system *around* the model, not
the model. Page 02 opened up the one resource that system runs on — context, what the agent can
see. This page is the system itself. It has a name — the **harness** — and building it is the
highest-leverage thing you can do as an AI-native engineer.

Start with the definition, because it's broader than it sounds. The harness is *everything in an
agent system except the model itself*. Some of it the vendor ships — the agent's tooling and
runtime. But the part you own, the part this whole playbook is about, is the harness *your team*
builds for *your* codebase. And it turns out that whole thing reduces to just two kinds of control,
both of which reach the agent through the context channel you just met.

## The harness is the lever you actually control

Here is why this is the page that matters most. You don't train the model — at most you *choose*
it, from a short list everyone else can choose from too. What you fully own is the harness. And
that ownership counts for more than it sounds, because the harness isn't a small adjustment around
a fixed result: **the same model in a better harness produces meaningfully better work.** Hand two
teams the identical model — one wrapping it in sharp guides and honest sensors, the other prompting
it raw — and their output quality diverges hard. The scaffolding around the model is roughly as
decisive as the model itself.

That's the whole reason this playbook bets where it does. Chasing the next model release is the
lever you don't control and everyone shares; engineering the harness is the lever you *do* control
and that a competitor can't copy for free. So when the output disappoints, the reflex to wait for a
smarter model is usually the wrong one — the higher-return move is almost always to tighten the
harness around the model you already have.

## Guides and sensors — before and after

Everything you build to steer an agent is either a **guide** or a **sensor**.

A **guide** works *before* the agent acts. It's feedforward — it raises the odds of a correct
first attempt by putting context, constraints, and conventions in front of the agent up front.
Instruction files, conventions, architecture boundaries, a curated example of the pattern you
want copied: all guides. (This is the channel from the last page in action — a guide is only as
good as your ability to fit it into the budget without crowding out the task.)

A **sensor** works *after* the agent acts. It's feedback — it observes the output so the agent
(or you) can self-correct. Tests, linters, type checkers, architecture fitness functions, an AI
reviewer reading the diff: all sensors.

The trap is thinking you can get away with one. You can't, and the reason is worth holding onto:

- **Guides without sensors** never verify that the rules were followed. The agent reads your
  convention, believes it complied, and merges something that quietly didn't. A rule with nothing
  to enforce it is a *wish*.
- **Sensors without guides** catch mistakes after the fact but never teach the agent what was
  expected — so it re-makes the same mistake every single run. Expensive, and it never converges.

Guides set the expectation; sensors prove it was met. Bugs live in the gap between the two, which
is exactly why you need both halves.

## Reach for the cheap check first

Not all sensors cost the same. They come in two modes, and the cheaper one is usually enough:

| Mode | What it's like | Examples |
|---|---|---|
| **Computational** | deterministic, fast (ms–s), cheap, never falsely confident | tests, linters, type checkers, formatters |
| **Inferential** | semantic, AI-driven, slower, non-deterministic, costs tokens | LLM code reviewers, spec-to-code validators |

The rule of thumb: **encode a rule as a computational check before you reach for an inferential
one.** A type error is caught for free by a type checker; "this variable is misnamed" genuinely
needs an AI to judge. Don't pay for inference where a deterministic check would do — and don't
trust inference where determinism is available, because an AI reviewer can be talked out of a real
finding in a way a failing test cannot.

## Three things you're regulating — hardest last

Guides and sensors get pointed at three kinds of target, and they get harder in order:

1. **Maintainability** — the internal quality a good engineer notices reading a diff: complexity,
   duplication, dead code, style. Computational sensors catch the structural ones reliably.
2. **Architecture fitness** — automated assertions about the system's *shape*: "no cross-module
   imports," "p95 < 200ms," dependency direction. The agent should be *blocked from merging*
   anything that breaks one.
3. **Behaviour** — does the code do what the *business* actually wanted? This is the hard one, and
   still unsolved for high autonomy: a spec is the guide, tests are the sensor — but an agent can
   write tests that match its own *wrong* implementation, so human verification stays essential.
   ([The behaviour harness](../20-harness/05-behaviour-harness.md) goes deep on this.)

## Some codebases take the harness better than others

Before you build a single control, notice the ground you're building on. The same three
targets are far cheaper to regulate in some codebases than others, and that property has a
name: **harnessability**. Four things raise it:

- **Strong static types.** A type checker is a high-signal sensor you get for free — it
  catches a whole class of maintainability and architecture errors the instant the agent
  writes them, with zero tokens and no false confidence.
- **Clear module boundaries.** When the system's shape is explicit, architecture fitness
  functions are easy to write — "this module must not import that one" becomes a one-line
  lint rather than a judgement call.
- **Established frameworks.** Agents have training data on them, so the conventions they
  reach for by default are the ones you actually want — half the guide is written before
  you start.
- **High test coverage.** A real test suite *is* the behaviour harness — the hardest
  category to build from scratch already exists, watching every change.

The payoff is the part worth internalising: **investing in harnessability pays off twice.**
Types, modular architecture, and test coverage were always good engineering — they make the
codebase easier for *humans*. The same investments now also make it easier for *agents*,
because each one turns into a cheaper, more reliable sensor or a more predictable guide. The
work you'd do anyway to keep a codebase healthy is the same work that makes it agent-ready.
The converse is the warning: a sprawling, untyped, untested codebase resists every control
you try to add — and that's exactly when teams reach for an AI reviewer to paper over the
gap, paying inference costs for what a type checker should have caught for free.

Raising harnessability on purpose — repo legibility, module boundaries, coverage that compounds —
is its own practice, picked up in [repo structure as feedforward](../20-harness/02-repo-structure-and-legibility.md)
and [growing the harness](../50-adoption/02-growing-the-harness.md). Here the point is only to read
the ground before you build on it.

## You don't design the harness — you grow it

The last idea is the one that changes how you work. You do **not** sit down and design the
complete harness up front. You can't predict every way an agent will go wrong. Instead you run a
loop:

```
Agent acts → a sensor fires → you look at the failure →
  you add a guide or a sensor → the agent acts again
```

You observe *one real failure*, then encode the fix as a durable control so it can't recur — and
the agent can help build that control (ask it to write the fitness function from a description of
the rule). Each turn makes the next run more reliable. That's how a harness *compounds*, and it's
why a mature one looks designed but was actually grown one failure at a time.

## An open question: how do you grade the harness?

One honest caveat sits under all of this. You can grade your *code* — coverage tells you what ran,
mutation testing tells you whether the tests have teeth. But how do you grade the *harness itself*?
A quiet harness, with sensors that never fire, is ambiguous: it could mean the work is genuinely
clean, or it could mean the harness is blind and catching nothing. There's no settled equivalent of
mutation testing for "are these controls any good?" — *harness coverage* is still an open frontier,
not a solved practice. The working discipline until there is one: treat a harness that never
complains with the same suspicion you'd give a test that always passes, and periodically confirm
each control *can* still fire (the [silent-scan check](../30-delivery/04-drift-and-health-sensors.md)
is one instance of this). Don't mistake silence for safety.

## What it looks like on a real rule

Take an invariant every layered codebase has some version of: **"API routes must not contain
business logic — they call functions in the product package."** Watch each control do its job.

**Guide only.** You write the rule into the instruction file. The agent obeys it most of the time
— but on a rushed task it inlines a database query straight into a route handler. Nothing catches
it; it compiles, its own tests pass, it merges. Weeks later a second route needs the same logic,
diverges, and a bug is born. The rule was stated, never enforced.

**Sensor only.** Now the rule isn't written anywhere, but a sharp reviewer spots the inlined query
and asks for a fix. Good — except nothing told the agent the expectation, so on the *next* route
it inlines logic again. Same catch, same fix, every run. It never converges.

**Both.** The convention lives in the instruction file (guide) *and* a lint rule forbids any route
file from importing the database layer, failing the build if it does (sensor). The rule now applies
everywhere at once: the agent gets a red build the instant it inlines a query and self-corrects
before you see the diff. That's computational-first in action — a deterministic lint, not an AI
reviewer. And notice *how the lint got there*: someone saw the inlined-query failure once, then
encoded a sensor so it couldn't happen again. That's the steering loop — the harness grew a control
in response to a real failure.

The one example carries the whole page: guides and sensors are different controls doing different
jobs, you need both, you reach for the cheap computational one first, and you build the set
iteratively from failures rather than designing it all at once.

One thing the harness can't do is replace you. It *externalises* tacit judgment — the disgust at a
300-line function, the knowledge that this module must not import that one — into explicit guides
and sensors. What it can't externalise (social accountability, taste, organisational context) stays
human. The harness redirects your effort to those calls; it doesn't remove it.

That's the foundations: *why* the governed lifecycle beats autocomplete, *how* the agent perceives
the world through its context, and *what* the harness is made of. The rest of the playbook is the
practice of running that loop — and it starts where every good task starts, with the
[spec as the contract](../10-lifecycle/01-spec-the-contract.md).

> **Next up — [Spec — The Contract](../10-lifecycle/01-spec-the-contract.md):** foundations done, theory over. From here we stop philosophising and run one real change end to end. And every good change starts with the least glamorous artifact in software — a contract that pins down what "done" even means, before the agent gets a single chance to guess.

---
[← Previous: Context Engineering](02-context-engineering.md) · [Contents](../README.md) · [Next → Spec, the Contract](../10-lifecycle/01-spec-the-contract.md)

Related: [Why AI-Native](01-why-ai-native.md) · [Context Engineering](02-context-engineering.md) · [Guides — Feedforward](../20-harness/01-guides-feedforward.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Keep Quality Left](../20-harness/04-keep-quality-left.md)
