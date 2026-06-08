# Repo Structure and Legibility

> How your repo is laid out *is* a guide. A predictable, conventional structure — contracts in one place, tests beside the code they guard, boundaries written down where the agent can read them — is feedforward the repo broadcasts for free, on every task, without spending a token to re-explain itself.

Breadcrumb: [Playbook](../README.md) › Harness

## The principle

The previous page listed the guides you *write* — instruction files, blueprints, constraints. This
one is about a guide you don't write so much as *arrange*: the shape of the repository itself.

The agent perceives your project entirely through its [context window](../00-foundations/02-context-engineering.md).
Before it reads a single guide you authored, it reads the repo — the folder tree, the file names, where
things sit relative to each other. That layout is the first feedforward the agent ever meets, and it's
the cheapest you'll ever own: you arrange the structure once, and every future task benefits without you
re-spending a token. A **legible** repo — one a newcomer could navigate and reason about on its own — is
a map. An illegible one is a maze the agent has to grope through file by file, guessing as it goes.

Legibility isn't a vibe; it's a small set of concrete arrangements:

- **A conventional, industry-standard layout.** Lay the project out the way its framework and ecosystem
  expect. Agents carry strong priors from training data about where things *should* live; when your
  structure matches those priors, the agent's defaults are already right and half the map is drawn before
  you start. A bespoke layout throws all of that away.
- **A dedicated home for the contracts.** Keep specs in one obvious, named place (a `specs/` folder),
  not scattered through wikis, tickets, and comments. The contract the agent builds against — and that
  [the spec page](../10-lifecycle/01-spec-the-contract.md) makes the heart of the lifecycle — has to be
  *findable* to be useful.
- **Sensors co-located with the code they guard.** Put each test beside the module it covers, not in a
  far-off mirror tree. When the agent opens a module it sees the test that proves it, so it extends the
  suite instead of leaving the new path uncovered — and a reviewer sees the gap at a glance.
- **Clear package and module boundaries.** Make it obvious what lives where and which direction
  dependencies may flow, so the agent extends the structure instead of reaching across a line it
  shouldn't.
- **Consistent naming.** When the same kind of thing is named the same way everywhere, the agent can
  *predict* a file's name from its job and find it without a full-tree search.

The throughline: a legible repo turns "the agent can't find it" into "the agent navigates it." Everything
below is in service of that one shift.

## Why it works

The agent fills every gap it meets with its most probable guess (the same mechanism behind every guide —
see [feedforward](01-guides-feedforward.md)). A legible repo simply leaves fewer gaps to fill. When the
layout matches convention, the agent's default placement is correct. When the test sits beside the code,
the agent finds it and extends it rather than inventing a parallel one. When the boundary is explicit, the
agent doesn't reach across it, because it can *see* the line.

There's a budget reason too. In an illegible repo the agent burns its [attention budget](../00-foundations/02-context-engineering.md)
just *locating* things — grepping the tree, opening wrong files, reconstructing a structure that was never
written down. Every one of those tokens is spent on navigation instead of on the task, and the agent reasons
worse for it. A legible structure is feedforward that hands the agent the map up front, so its budget goes
to the work.

This is the [harnessability](../00-foundations/03-harness-engineering.md) idea made concrete, and it pays
off the same way: twice. Clear boundaries, conventional layout, and tests beside their code were always
good engineering — they make the repo easier for *humans*. The same arrangements now make it easier for
*agents*, because each one turns into a guide the structure broadcasts on its own. You're not doing new
work to be agent-ready; you're doing the work you'd do anyway, and collecting a second payoff.

## How to apply it

- **Follow the conventional layout for your stack.** Use the directory structure your framework and
  package manager expect, named the way the ecosystem names it. Save your creativity for the product, not
  the scaffolding — every deviation from convention is a guide you now have to write by hand to make up for
  it.
- **Give the contracts one obvious home.** A top-level `specs/` folder, one file per spec, named
  predictably. The agent should never have to ask where the spec for a feature lives.
- **Put sensors next to the code.** Co-locate tests with the modules they cover. Proximity is itself a
  signal: it tells the agent (and the reviewer) that *this* code is the thing *this* test is responsible
  for.
- **Make boundaries a readable map, not just a buried config.** This is the one teams get wrong most
  often. The rule "package A may not import package B" usually *does* exist — encoded deep in a
  hundreds-of-lines lint config. But that config is a [sensor](03-sensors-feedback.md): it enforces the
  boundary *after* the agent crosses it, and no agent reads a giant config to learn the shape *first*.
  Give the agent a short, human-readable map — the packages, what each owns, and the allowed dependency
  directions — *alongside* the config that enforces them. The map is the guide; the config is the check;
  you want both. (Same point, from the guides side: [keep boundaries explicit](01-guides-feedforward.md).)
- **Name things predictably.** One naming pattern per kind of thing, applied everywhere, so a file's name
  is derivable from its role.
- **For a large codebase, give the agent a structural index it can query.** Beyond a tidy tree, you can
  hand the agent a *code graph* — a machine-generated index of the symbols and how they connect (files →
  functions → classes → call chains) that it queries directly instead of grepping the tree file by file.
  It's feedforward the repo generates about *itself*: a concise, navigable reference that cuts the tokens
  an agent spends locating the right prior context and sharply improves how it navigates a big codebase.
  Treat this as a general practice — *give the agent a structural index it can query* — not an
  endorsement of any one indexer; the recommendation has to survive swapping the tool underneath it.
- **Don't:** invent a bespoke layout the agent has no priors for; scatter conventions across a wiki, three
  docs, and a config; let a giant lint config be the *only* record of your architecture; or let the
  structure drift until the layout no longer reflects how the code actually works — a stale map misleads
  with the full authority of being read first.

## In practice

A teammate asks the agent to add a feature that spans two packages of a monorepo — say, a reporting view
that needs a number computed in the finance package. The repo enforces a strict rule: packages are
bounded contexts, and finance may not depend on reporting. That rule is real, and it's enforced — by a
long, generated lint config that fails the build on a forbidden import.

**Without legibility.** Nowhere in the repo is there a plain-language map of which package owns what or
which way dependencies may flow; the only record is the lint config, which no agent reads front-to-back to
*learn* the architecture. So the agent guesses. It puts the new computation in the reporting package and
imports finance the wrong way — plausible, and exactly backwards. The build goes red on a rule the agent
never saw stated. It tries again, moves a file, trips a *different* line of the same config, and thrashes —
each attempt a fresh guess against an architecture it can only infer from failures. The boundary was
*enforced* but never *legible*, so the agent learned it the most expensive way possible.

**With legibility.** A short map file sits beside the config: here are the packages, here's what each owns,
here are the arrows for which may depend on which. The agent reads it first — it's small, it's findable,
it's written for a reader — and places the computation in the finance package, exposed through the one edge
reporting is allowed to use. The import direction is right on the first attempt. The lint config still
stands behind the map as the sensor that would have caught a violation; it just never has to fire, because
the guide got there first.

The lesson the example carries: enforcing a boundary and making it *legible* are two different jobs. A
config that fails the build is a sensor — it tells the agent it was wrong, after the fact, one red build at
a time. A readable map of the same boundary is a guide — it tells the agent what's right, before it acts.
Structure that's only enforced makes the agent learn by colliding with it; structure that's also legible
lets the agent navigate it. Arrange the repo so the shape is *readable*, and pair every boundary the config
guards with a map the agent can actually read.

## Anti-patterns

- **The buried boundary.** The architecture lives only inside a sprawling lint config — enforced, but
  never stated anywhere a reader (human or agent) learns it first. The agent discovers each rule by
  tripping it. ([Guides — Feedforward](01-guides-feedforward.md) · [Sensors — Feedback](03-sensors-feedback.md))
- **The bespoke layout.** An idiosyncratic structure the agent has no training-data priors for, so every
  placement is a guess and every convention has to be hand-written to compensate.
- **Scattered conventions.** The rules for how the repo is organised spread across a wiki, several docs,
  and a config, so "where does X go?" has no single answer — and nothing the agent can read up front.
- **The stale map.** A layout or boundary map that no longer matches how the code actually works, teaching
  the wrong structure with the authority of being read first. ([Growing the Harness](../50-adoption/02-growing-the-harness.md))
- **Grep-the-whole-tree.** On a large codebase, leaving the agent to locate every symbol by scanning files
  one at a time — burning budget on navigation that a structural index would hand it for free.

---
[← Previous: Guides — Feedforward](01-guides-feedforward.md) · [Contents](../README.md) · [Next → Sensors — Feedback](03-sensors-feedback.md)

Related: [Guides — Feedforward](01-guides-feedforward.md) · [Harness Engineering](../00-foundations/03-harness-engineering.md) · [Context Engineering](../00-foundations/02-context-engineering.md) · [Spec — The Contract](../10-lifecycle/01-spec-the-contract.md) · [Sensors — Feedback](03-sensors-feedback.md) · [Growing the Harness](../50-adoption/02-growing-the-harness.md)
