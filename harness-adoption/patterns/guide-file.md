# Pattern: the guide file (feedforward)

## Intent

A single checked-in file holding what the agent must know **before** it
acts: where the code lives, the non-negotiable conventions, and the exact
build/test/check commands. This is feedforward — the work you do up front
to raise the odds of a right first attempt. See
`../../playbook-md/20-harness/01-guides-feedforward.md`.

## What good looks like

- **Concrete.** Every rule is specific enough to act on without guessing
  ("import data-layer code only through `repositories/`", not "be careful
  with imports"). Vague rules are noise the agent pays a context tax to read.
- **Every line earns its place.** Everything the agent reads is advisory —
  each line competes with every other for attention, so compliance with any
  one rule degrades as the file grows. Litmus test per line: *would removing
  it cause the agent to make mistakes?* If not, cut it.
- **Stable.** Volatile state (current sprint, who's on call) lives
  elsewhere; the guide changes maybe monthly.
- **Discoverable.** Every "MUST read" path resolves to a real file.
- **Short.** If it grows past ~400 lines the agent attends to the wrong
  things — extract sections (UI conventions, runbooks) into linked files.
  This is **progressive disclosure**: the guide is the index the agent
  always reads; detail loads only when a task makes it relevant. The
  index and each linked file's one-line description matter more than the
  bodies — they're what the agent routes on.
- **Enforced to match its weight.** Each rule sits on one of three rungs:
  **(soft)** — a judgment call, explicitly marked, no sensor; **sensored**
  — a check in the gate verifies it; or **hook-enforced** — for invariants
  that must hold every time, a script the harness runs on the event
  (before an edit, end of turn) hard-blocks the violation instead of
  relying on the agent having read the rule. Prose is advisory; a hook
  never passes through the context window. Rule of thumb: instructions
  for judgment calls, hooks for invariants. Aspirational rules with no
  enforcement decay and erode trust in the whole file.

## Blocking behaviour

The guide itself doesn't block — it's feedforward. But it must **name the
gate**: point at the one `check` command, so the guide and the gate agree
on what "passing" means. A guide whose commands don't match the gate is
worse than none.

## Assessment signal

- Present? Look for `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`/`.cursorrules`,
  `.github/copilot-instructions.md`.
- Concrete or vague? Sample 3–5 rules; can the agent act on each without
  interpretation?
- Discoverable? Do referenced paths resolve? (Dangling refs = decay.)
- Aligned? Does it name the same `check` command the gate runs?

## The cross-tool standard: `AGENTS.md`

Default to **`AGENTS.md`**. It's the convention Copilot, Cursor, Codex,
and most local agents read. For tools that look for their own file, add a
**one-line pointer**, never a copy:

- `CLAUDE.md` → `See AGENTS.md for the project guide.`
- `.cursor/rules/harness.md` → same pointer.
- `.github/copilot-instructions.md` → same pointer.

One source of truth, thin redirects. Duplicated guide content drifts.

## Recipe (all stacks — content is stack-shaped, structure is not)

A skeleton the adopt skill fills from the repo + assessment:

```markdown
# <Project> — agent guide

## Layout
- app code: <where>
- tests: <where>
- specs / contracts: <where, or "none yet">

## Conventions (non-negotiable)
- <the 3–5 rules you will not bend — each one the agent would get wrong
  without being told, each backed by a sensor or marked (soft)>

## Commands
- build:  <command>
- test:   <command>
- check:  <the one command the gate runs: format + lint + types (+ test)>

## Workflow
- branch from <integration branch>; never commit to <main>
- run `check` before pushing; the gate enforces it
- commit / open PR only on explicit instruction
```

Stack only changes the *commands* and *conventions*, not the structure.

## How adopt writes it

1. Infer layout, commands, and existing conventions from the repo — don't
   ask what you can read.
2. Write `AGENTS.md` at repo root. Keep it under ~400 lines; extract if
   needed.
3. Add one-line pointer files for whichever AI tools the team uses.
4. For each convention written, confirm a sensor exists or mark it
   `(soft)`. Flag to the user any rule that ought to have a sensor —
   that's a candidate for the next adopt increment. For a rule that must
   hold *every* time (never commit to main, never edit generated files),
   propose a hook in the team's agent tool rather than another line of
   prose.
5. Verify every referenced path exists before finishing.
