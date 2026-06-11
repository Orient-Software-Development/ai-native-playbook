# Task Slicing

> Hand the agent one slice at a time: one deliverable, one test, one commit. A small slice you can verify the moment it lands beats a sprawling diff where one buried mistake taints everything around it — and the approved plan already drew the slice lines for you.

Breadcrumb: [Playbook](../README.md) › Lifecycle

## Story so far

The plan is approved and, if you did it right, it already reads as an ordered list of small pieces. Tempting to just say "build all that" and go make coffee. Don't. A sprawling all-at-once diff is where one buried mistake hides among forty plausible lines and taints everything it touches. So this chapter takes the plan's list and turns it into discipline: one deliverable, one test, one commit — built bottom-up, each slice proven before the next gets to lean on it.

## The principle

Don't ask the agent to build the whole feature in one go. Cut the approved
[plan](02-plan-before-code.md) into **tasks**, and make each task as small as it can
honestly be:

- **One deliverable.** A task produces exactly one nameable thing — *the table exists*,
  *the flag persists*, *the list orders correctly*. If you can't say what single thing a
  task delivers in one short phrase, it's really several tasks wearing one name.
- **One test.** Each deliverable comes with the one check that proves it — and you run
  that check before moving on.
- **One commit.** A verified slice is a checkpoint: a small, self-contained diff you can
  read in one sitting and roll back to if the next slice goes wrong.

And **slice by layer, bottom-up**. Most features stack the same way — data **schema** at
the bottom, then the **commands/logic** that write and read it, then the **API** that
exposes it, then the **UI** on top. Build them in that order, proving each layer before
you build on it, so every slice rests on something already verified rather than on a guess.

The discipline is simple: **verify each slice before you start the next.** A slice isn't
"done" because the agent stopped typing — it's done when its one test passes against real
state.

## Why it works

A bug caught in the slice that introduced it is a one-line fix. The same bug discovered
three layers later — after the API and UI are built on top of it — is a thread you have to
pull through everything stacked above it. Slicing keeps every mistake close to its cause:
when the two-user isolation check fails at the query layer, you already *know* the schema
and the commands beneath it passed their own checks, so there's exactly one place to look
instead of eight.

Small diffs are also the only diffs that actually get reviewed. A wall of plausible
agent-written code invites a skim, and a skim is where ["zombie code"](06-review-and-convergence.md)
survives — code that looks right, compiles, and quietly does the wrong thing. A slice you
can hold in your head is one you can genuinely review, not just wave through.

There's a context benefit too. A focused task keeps the agent working over a small,
relevant slice of the codebase instead of spreading its attention across every layer at
once — the same [attention-budget](../00-foundations/02-context-engineering.md) logic that
makes a short spec beat a sprawling one. Narrow the task and the agent's work gets sharper,
not just easier to check.

And each committed slice is a **checkpoint**. When a slice goes wrong, you revert one
small, self-contained commit and retry — cheap. When a single giant task goes wrong, every
layer is tangled with every other and there's nothing clean to roll back to; the whole
thing is all-or-nothing, and "nothing" is usually what you get.

## How to apply it

- **One deliverable, one test, one commit.** The three-part test for a good slice. If a
  task can't name its single deliverable, or you can't name the one check that proves it,
  split it until you can.
- **Slice by layer, bottom-up.** Schema → commands/logic → API → UI. Each slice sits on a
  layer that's already proven, so a failure points down at *this* slice, not at some
  unverified foundation under it.
- **Verify before you advance.** Run the slice's check against real state before opening
  the next task — see [Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md). Never
  stack a new slice on an unverified one; that's how one bug becomes four.
- **Keep the diff reviewable.** A couple of files, not a sprawl across the codebase. If a
  single task is touching every layer at once, it isn't sliced yet.
- **Let the plan be your slice list.** A good [plan](02-plan-before-code.md) already reads
  as an ordered set of small pieces — you usually don't re-design the slices, you just take
  them one at a time, top of the list down.
- **Commit each verified slice.** The checkpoint is the point. A clean, working commit per
  slice is what makes a wrong turn a one-line revert instead of a salvage operation — and
  it's what lets short-lived branches merge continuously
  ([trunk-based development](../30-delivery/01-trunk-based-development.md)).
- **Don't:** bundle a "while I'm here" cleanup into a feature slice — unrelated changes in
  one commit defeat the checkpoint. Don't let one task span every layer. Don't call a slice
  done because the agent finished, only because its test passed.

## In practice

Take the *per-user deal priority* feature whose plan you approved on the previous page. The
plan already laid out four layers in order: a `deal_priorities` link table, the
flag/unflag commands, the extended list query, and the UI star toggle. That ordered list
*is* the slice list.

**As one big task**, you tell the agent "build per-user deal priority" and it returns a
single diff touching all four layers at once — schema, commands, query, and component
together. Now the cross-user bug from the spec (REQ-002: a flag must stay private) is in
there *somewhere*, but it's buried among migration files and UI tweaks. The schema mistake
only shows when you click through the finished UI, by which point the query and component
already depend on it. One thing is wrong; everything is entangled with it; and there's no
checkpoint to fall back to — the diff is all-or-nothing, so a fix means re-reading the
whole thing. This is exactly the unreviewable wall a busy reviewer skims, and exactly how
the privacy bug ships.

**Sliced**, the same plan becomes four tasks, each proven before the next begins:

```text
Slice 1 — Schema      deliverable: deal_priorities(user_id, deal_id) table + migration
                      test: migration applies cleanly; the (user_id, deal_id) pair is unique
                      commit ✓  →  the table exists and is constrained

Slice 2 — Commands    deliverable: flagDeal / unflagDeal write to the link table
                      test: flagging persists a row for THIS user; unflag removes it
                            (assert real DB state, not "ran without throwing")
                      commit ✓  →  flags actually persist

Slice 3 — Query       deliverable: deals-list query left-joins the current user's
                            priorities, orders flagged-first, keeps existing sub-order
                      test: the two-user test (AC-002) — A's flag never appears for B;
                            ordering is correct and nothing is hidden
                      commit ✓  →  the contract's privacy rule is proven

Slice 4 — UI          deliverable: a star toggle on each row, wired to the commands
                      test/evidence: click-through + screenshot; flag, reorder, unflag
                      commit ✓  →  the feature is visible and usable
```

Each slice rests on a layer already proven. So when the two-user isolation test fails at
**Slice 3**, you don't go hunting: Slices 1 and 2 passed their own checks, so the bug is in
the join you just wrote — one place, one small revert, one retry. The privacy rule that
would have shipped silently in the big-bang version gets caught at the exact slice that
owns it, against a diff small enough that the failing test points straight at the cause.
And every green slice along the way is a commit you can stand on.

## Anti-patterns

- **The big-bang task.** One task spanning every layer, returned as a diff too large to
  truly review — so entangled bugs hide in plausible code and there's no checkpoint to roll
  back to. ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The "while I'm here" bundle.** Unrelated cleanup smuggled into a feature slice, so the
  commit no longer captures one deliverable and stops being a clean checkpoint.
- **Stacked unverified slices.** Building the next layer before the one beneath it is
  proven, so a foundational bug compounds upward and surfaces far from its cause.
- **The untestable slice.** A "task" with no single observable check that proves it — a
  sign it's really several tasks, or that the requirement underneath it is too vague to
  build.

> **Next up — [Code With the Agent](04-code-with-the-agent.md):** the slice is approved and the agent starts typing. This is *not* the moment to step back and wait for the diff — it's the moment to steer in real time, point at concrete references instead of fuzzy analogies, and remember the uncomfortable truth that a rule you stated is not a rule the agent followed.

---
[← Previous: Plan Before Code](02-plan-before-code.md) · [Contents](../README.md) · [Next → Code With the Agent](04-code-with-the-agent.md)

Related: [Plan Before Code](02-plan-before-code.md) · [Code With the Agent](04-code-with-the-agent.md) · [Verify — Proof, Not Vibes](05-verify-proof-not-vibes.md) · [Review and Convergence](06-review-and-convergence.md) · [Trunk-Based Development](../30-delivery/01-trunk-based-development.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
