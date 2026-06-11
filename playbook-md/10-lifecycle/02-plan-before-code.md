# Plan Before Code

> Put a planning step between the spec and the first line of code. The spec settled *what* to build; the plan settles *how* — and the cheapest place to catch a wrong approach is in a paragraph you can edit, not a diff you have to throw away.

Breadcrumb: [Playbook](../README.md) › Lifecycle

## Story so far

The spec is signed: *what* to build is now a contract the intent-owner could actually read, challenge, and sign off on. Victory — except "what" isn't "how," and the gap between the two is where good features quietly go to die. Hand an agent an approved spec and a green light and it'll bolt for the first approach that crosses its mind, usually a perfectly reasonable idea aimed at a codebase it *imagined* rather than the one you have. So before a single line of code, we add one cheap step: explore, then plan, then build.

## The principle

Between the [spec](01-spec-the-contract.md) and the code, add one more step: ask the
agent to **explore and plan** before it builds. A **plan** is a short, written
statement of *how* the feature will be built — which files it will touch, the approach
for each, the order of work, the tests that will prove it, and the open questions it hit
along the way. The spec owns the *what*; the plan owns the *how* the spec deliberately
left out.

The shape of the step is **explore → plan → code**:

- **Explore.** Point the agent at the relevant code and let it *read* before it proposes
  anything — the existing helpers, the table it'll extend, how similar features are
  already done. An agent that plans without looking plans against a codebase it imagined.
- **Plan.** It comes back with the approach in prose, not code. Many agents have a
  dedicated **plan mode** — a read-only mode where the agent can investigate and draft a
  plan but *cannot* edit files — which keeps the exploring from quietly turning into
  half-built code.
- **Code.** Only after you've read the plan and approved it does it start writing.

The rule that makes this work: **show the plan before executing.** The plan is a decision
point, not a formality. You read it, you correct it, you approve it — *then* code happens.

Unlike the spec, the plan **doesn't have to be permanent.** The
[spec](01-spec-the-contract.md) is the durable contract — it lives with the code,
version-controlled, audited against the code over time. The plan is *working scaffolding*:
its whole job is to get you to an approved approach before code exists. Once the code is
written and reviewed against it, the plan can be ephemeral — keep it if it helps the next
reader, discard it if it doesn't. The one thing you must not do is confuse the two: never
let a throwaway plan quietly absorb a *requirement* that belongs in the durable spec, or
the contract loses a clause the moment the plan is gone.

## Why it works

An agent will commit to the first plausible approach it thinks of, the same way it fills a
vague spec with a confident guess. Without a planning step it makes every *how* decision
silently, inside the diff — which table, which helper, which order — and you only discover
those decisions after the code exists, when changing them means a rewrite.

Planning pulls those decisions forward into prose, where they're cheap. Correcting "you're
about to add this flag to the shared deals table — use a per-user link instead" is a
one-line comment when it's a sentence in a plan. The same correction after the code is
written is a rebuild: the migration, the query, the tests, all already pointed the wrong
way. A small amount of effort spent planning buys back a large amount of rework you now
never have to do — which is why planning *feels* like a detour and almost never is one.

The explore-first half matters just as much. The most common wrong plan isn't a bad idea —
it's a good idea aimed at a codebase that doesn't exist, because the agent guessed at the
structure instead of reading it. Reading first is what turns "reinvent a helper that's
already there" into "extend the one that's already there."

And the plan does a second job: it's the smallest possible artifact you can fully review.
Reading a five-line approach is faster than reading a two-hundred-line diff, and you catch
the architectural mistake — the wrong boundary, the missed edge case — at the layer where
it's a thought, not buried in code that looks finished.

## How to apply it

- **Explore before you plan.** Tell the agent which files to read first, or let plan mode
  range over them. A plan written without reading the code is a guess in a nicer format.
- **Make the plan show its work.** A useful plan names the files it will touch, the
  approach for each, the **order** (usually bottom-up — schema, then logic, then API, then
  UI), the tests that prove each piece, and any **unknowns** it couldn't resolve. The
  unknowns are the most valuable line: they're where the agent is about to guess.
- **Treat the review as the real decision.** This is where you catch the wrong table, the
  reinvented helper, the edge case the spec implied but didn't spell out. Read it like it
  matters, because correcting it here is free.
- **Use a read-only plan mode** when your agent has one. It stops exploration from sprawling
  into committed code you didn't ask for, and keeps the plan honest — a proposal, not a
  fait accompli.
- **Keep it proportional.** A one-line fix doesn't need a plan; planning it to death is its
  own waste. Reach for the plan step when the *how* has real choices in it — a new data
  shape, more than a couple of files, anything touching more than one layer.
- **Hand the approved plan to task slicing.** A good plan already reads as an ordered list
  of small pieces; the next step ([task slicing](03-task-slicing.md)) builds them one at a
  time, verifying each before the next.
- **Don't:** jump straight from spec to code on anything non-trivial. Don't approve a plan
  you didn't actually read — a rubber-stamped plan is worse than none, because it *feels*
  reviewed. Don't let the plan re-argue the *what*; if the requirements are wrong, fix the
  [spec](01-spec-the-contract.md), don't patch it in the plan.

## In practice

Carry forward the feature from the previous page: the *per-user deal priority* spec is
written and the sales lead has signed off. The *what* is settled — flag a deal, keep the
flag private, show flagged deals first without hiding anything. Now the *how*.

**Without a planning step**, the approved spec goes straight to "build it." The agent
returns a clean, working diff — and it made three *how* decisions on its own, all inside
the code. It added a `priority` column straight onto the shared `deals` table (so the flag
is global — exactly the cross-user leak REQ-002 forbids, now baked into a migration). It
wrote a fresh function to load and sort the list, not noticing the deals list already has a
query it could have extended. And it ordered the list with a filter that drops unflagged
deals instead of sorting them down. Every one of these surfaces at diff review, and every
one is now expensive: the column is in a migration, the duplicate query has its own tests,
the filter is woven through the component. The spec was right; the *how* went wrong in
three places at once, and you're paying for it in code.

**With a planning step**, you ask the agent to explore the deals module and come back with
a plan first. In plan mode it reads the schema, the existing list query, and how flags are
done elsewhere, then proposes:

```markdown
## Plan — per-user deal priority

Approach: a flag is per-user, so it does NOT belong on `deals`. Add a separate
`deal_priorities(user_id, deal_id)` link table — presence = flagged.

Order of work:
1. Schema: new `deal_priorities` link table + migration.
2. Command: flagDeal / unflagDeal (write to the link table).
3. Query: EXTEND the existing deals-list query — left-join the current user's
   priorities, order flagged-first, keep the existing order within each group.
   (Reusing the existing query, not writing a new one.)
4. UI: a star toggle on each row, wired to the commands.

Tests: per-layer; plus a two-user test asserting A's flag never appears for B (AC-002).

Open question: within the flagged group, keep the existing sort order, or
most-recently-flagged first? Spec doesn't say — assuming existing order.
```

You read it in under a minute. The approach is right where it would have been wrong: the
flag lives in a link table, so the leak never gets built; the existing query gets extended
instead of duplicated; ordering is a sort, not a filter. You answer the open question in
the margin ("existing order is fine") and approve. The agent writes code against an
approach you've already agreed to — and the diff review later is about whether the code
matches the plan, not a first encounter with three decisions nobody discussed.

The detour that wasn't: the planning step cost a minute of reading. It saved a migration, a
duplicate query, and a rebuilt component — and it caught the one mistake that would have
violated the contract.

## Anti-patterns

- **The spec-to-code leap.** Skipping straight to building on anything non-trivial, so every
  *how* decision gets made silently inside the diff and reviewed only after it's expensive
  to change. ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The rubber-stamp plan.** Approving a plan you didn't read. It carries the *feeling* of
  review without the substance — worse than no plan, because it lowers your guard at diff
  review too.
- **The plan that re-argues the spec.** Re-litigating *what* to build during planning. If the
  requirements are wrong, the fix is upstream in the [spec](01-spec-the-contract.md); the
  plan is only for the *how*.
- **Planning the one-liner to death.** A heavyweight plan for a trivial change — proportion
  cuts both ways. Plan where the *how* has real choices; just do the obvious thing when it
  doesn't.

> **Next up — [Task Slicing](03-task-slicing.md):** you've got an approved plan that already reads like an ordered to-do list. Resist the urge to hand the whole thing over in one heroic go — next we cut it into slices the agent builds one at a time, each with its own test, so a single buried mistake can't quietly poison everything stacked around it.

---
[← Previous: Spec — The Contract](01-spec-the-contract.md) · [Contents](../README.md) · [Next → Task Slicing](03-task-slicing.md)

Related: [Spec — The Contract](01-spec-the-contract.md) · [Task Slicing](03-task-slicing.md) · [Code With the Agent](04-code-with-the-agent.md) · [Repo Structure and Legibility](../20-harness/02-repo-structure-and-legibility.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
