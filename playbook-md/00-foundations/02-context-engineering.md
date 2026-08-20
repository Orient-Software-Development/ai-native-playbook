# Context Engineering

> The context window is a budget, not a backpack. What you load — and what you keep *out* — decides whether the agent reasons over the right information or drowns in noise. Anything it can't see effectively doesn't exist. 

Breadcrumb: [Playbook](../README.md) › Foundations

## Story so far

Last chapter we agreed on the deal: humans steer, agents execute. Tidy. The catch nobody mentioned at the signing ceremony is that an agent only knows what it can *see* — and seeing, it turns out, is the whole job. So before we build a single guide or sensor, we have to talk about the one resource the entire harness runs on: context. Think of it less as a hard drive and more as a desk. A small desk. That you keep piling paper onto.

A model only knows what's in its context window *at the instant it acts*. That window is a
**budget**, not a backpack you keep stuffing. Every token spent on a stale instruction or an
irrelevant file is a token *not* spent on the task. And this isn't just about hitting the limit:
a model reasons most reliably over a focused input, and as the input grows the thread frays — a
specific fact gets buried, and earlier framing quietly biases later steps — so quality degrades
*well before* the window fills. That fraying-with-length is called **context rot**. More context
is not more understanding — past a point it's dilution.

The instinct to pile *more* in — a longer spec, a bigger instruction file, one endless chat — is the
bug, not the fix. But the answer isn't simply *less* either: too little context and the agent guesses
at what it can't see. The discipline is **the right context — not too much, not too little, just what
the task needs.**

It helps to know that context lives in three layers, each with a different job:

- **Working memory** — what's in the window right now. Ephemeral and budgeted; it vanishes on a
  reset. This is the layer everything above is about.
- **The memory store** — a semi-durable scratchpad the agent can retrieve from across turns. Handy,
  but an *accelerator*, never the source of truth.
- **The repository** — code, specs, decisions, and notes under version control: the durable
  **system of record**, reviewable and legible to every future run. When something matters, it belongs
  here.

The practical rule that falls out: the repo is the canon, the store is a convenience, the window is
scratch space. So when a session gets bloated, the move is *situational* — **reset** (a clean slate
carrying only a durable note) when you switch tasks, or before any independent-judgement step where
the accumulated history would bias the verdict; **continue** when you're deep in one coherent task
and the context is still signal, not noise. Either way, write the durable note first: the repo
remembers reliably; a long chat doesn't. (When to reset rather than compact is the next bullet — and
the distinction matters enough that it has its own treatment below.)

## The habits that get you there

- **Point, don't dump.** Name the specific spec, file, or decision the task needs. An agent told
  where the source of truth *is* reasons from it; an agent handed everything reconstructs a
  plausible — often wrong — version. Give it a map, not a 1,000-page manual.

- **Write it down where the agent can see it.** Anything living in a chat thread, a doc tool, or
  someone's head is *invisible* to the agent. Only repo-local, versioned artifacts — code, specs,
  schemas, instruction files — are reliably legible across runs. If a decision exists only in Microsoft Teams, it may as well not exist.

- **Clear context when you switch topics — or when history would bias the agent.** A long session
  accumulates noise, and worse, *bias*. The classic antipattern: one marathon conversation that runs
  from brainstorm → plan → code → tests. By the time the agent writes the tests, it has spent the
  whole thread convincing itself the design is right — so it writes tests that confirm its own code,
  not the spec, and its "all green" verdict isn't objective. Start a fresh context for each distinct
  job, carrying only a small handoff of what matters. A clean slate is what keeps verification
  honest. *(This is a* reset *— a genuine clean start — not* compaction*, which just summarises the
  same biased history in place.)*

- **More skills and guides isn't better — it's a tax.** Loading a big library of reusable skills or
  instruction files up front can burn tens of thousands of tokens *before you even ask your first
  question*, and agents are bad at picking the right one from a large pile — they miss the good skill
  and a noisy, irrelevant one can actively mislead them. Curate a small, high-signal set and load the
  rest only when the task calls for it. (How to *design* skills so they retrieve well is
  [Guides — Feedforward](../20-harness/01-guides-feedforward.md); the failure when you don't is in
  [Failure Modes](../40-anti-patterns/01-failure-modes.md).)

- **Own your prompts — don't outsource prompt engineering to a framework.** Context is your biggest
  lever over the agent's output, so keep your hands on it. A framework that "handles context for you"
  is making the most important decision — *what the agent sees* — where you can't inspect or tune it.
  You don't need to see the model's weights, but you must see its prompt. A thin helper that does
  what you told it is fine; a framework that decides for you is not.

## Techniques that stretch the budget

Beyond the habits, a few named techniques do most of the heavy lifting:

- **Progressive disclosure.** Structure information so the agent loads it in stages: lightweight
  identifiers first — file paths, skill names and one-line descriptions, an index — full content only
  when judged relevant, appendix detail only when actually needed. A well-organized manual works the
  same way: table of contents, then the chapter, then the appendix. This is why a skill's metadata and
  an instruction file's index matter more than their bodies: the agent decides *whether* to read from
  the cheap layer, and only then pays for the expensive one.

- **Just-in-time retrieval over up-front dumping.** Keep *references* in context — paths, queries,
  links — and fetch the content at the moment of need, rather than pre-loading everything that might
  turn out relevant. The same move applies to large outputs: interrogate a big test report with small
  query tools — head, tail, a summary command — instead of loading the whole artifact into the window
  to answer one question about it.

- **Structured note-taking as agentic memory.** The agent writes durable notes outside the window — a
  progress file, a decision log — and pulls them back in after a reset. This is what makes the
  reset-over-compaction guidance workable on long tasks: the reset stops being amnesia because the
  note, not the chat history, carries the state. It's the memory-store layer from above doing real
  work — still a convenience, not the canon, but the bridge that lets working memory be cleared
  ruthlessly.

- **Delegate the investigation, keep the summary.** A subagent's value isn't parallelism — it's
  context isolation. It burns *its own* window on the search trail and returns only the condensed
  answer: a thousand-token summary instead of the hundred-thousand-token exhaust it took to find it.
  The main thread stays focused on the task; the noise stays in the subagent.

> **Next up — [Harness Engineering](03-harness-engineering.md):** you've met the channel the agent perceives the world through. Now we name the whole apparatus you build around it — guides before, sensors after — and see why the same model in a sharp harness quietly runs rings around itself in a sloppy one.

---
[← Previous: Why AI-Native](01-why-ai-native.md) · [Contents](../README.md) · [Next → Harness Engineering](03-harness-engineering.md)

Related: [Why AI-Native](01-why-ai-native.md) · [Harness Engineering](03-harness-engineering.md) · [Guides — Feedforward](../20-harness/01-guides-feedforward.md) · [Spec — The Contract](../10-lifecycle/01-spec-the-contract.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
