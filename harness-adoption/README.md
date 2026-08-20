# Harness Adoption Starter

A **skill-driven** starter that any team can run to assess its current
engineering practices against the harness-engineering model, then adopt
the smallest set of controls that fits **their** stack, **their** AI
coding tool, and **their** context — greenfield or legacy.

It is deliberately **not** a fixed file tree you copy and prune. A static
template can only fit the stack it was distilled from. This starter
instead ships *skills*: an agent reads your repo, scores it, and
**generates** a fitted harness in your idiom. It is fully self-contained —
everything the skills need (the harness model, the readiness rubric, the
scorecard, the control recipes) lives in this directory.

---

## The three skills

They map directly to the playbook's steering loop
(`../playbook-md/50-adoption/02-growing-the-harness.md`):
assess → adopt → re-assess.

| Skill | What it does | Writes files? |
|-------|--------------|---------------|
| **[assess](skills/assess.md)** | Detects your stack, AI tool, and existing controls. Scores the five harness layers and reports a ranked gap list. | No — read-only |
| **[adopt](skills/adopt.md)** | Takes the gap list, proposes the smallest fitted set of controls for your stack/tool, writes only what you approve, re-scores. | Yes — incrementally, with approval |
| **[audit](skills/audit.md)** | Periodic re-run of `assess` on a repo that already has a harness. Reports drift and proposes the next increment. | A dated scorecard only |

Run them in order the first time. After that, `audit` is the recurring
ritual (every 4–8 weeks).

---

## How a team runs it

The skills are written as **tool-neutral runbooks** — plain markdown an
agent executes. Pick the entry point for your AI tool:

| Your tool | Entry point |
|-----------|-------------|
| Claude Code | `/harness-assess`, `/harness-adopt`, `/harness-audit` (see [adapters/claude/](adapters/claude/) — one thin skill per runbook, so each routes on its own name and description) |
| Copilot / Cursor / Codex / local agents | drop [adapters/AGENTS.md.snippet](adapters/AGENTS.md.snippet) into your `AGENTS.md` and ask the agent to "run the harness assessment" |
| Any chat agent (no repo access) | paste [adapters/paste-prompt.md](adapters/paste-prompt.md) |

Every adapter does the same thing: point the agent at `skills/assess.md`
and let it run. The skill is the source of truth; the adapters are thin.

---

## What it produces

Nothing the starter installs is imported as a library. Everything is
**generated into your repo and owned by you**, in your stack's idiom:

- **A guide file** (`AGENTS.md` by default — the cross-tool standard —
  with an optional `CLAUDE.md` pointer).
- **A single `check` entry point** that runs format + lint + types in
  one command, in your toolchain.
- **A blocking gate** at the earliest stage you can enforce
  (git hook, pre-commit framework, Makefile, or CI), wired to `check`.
- **One real behaviour test** against the product's most important path.
- **Higher-value controls only as assessment shows you need them** —
  suppression budgets, secret scan, architecture fitness, inferential
  review, spec/ADR corpus, behaviour-contract packs, scheduled drift
  & health scans.

The order is the playbook's order: the
[minimum viable harness](../playbook-md/50-adoption/01-minimum-viable-harness.md)
first, then [grow it one real failure at a time](../playbook-md/50-adoption/02-growing-the-harness.md).

---

## Layout

```
harness-adoption/
├── README.md                ← you are here
├── skills/                  ← the three runnable skills (tool-neutral)
│   ├── assess.md
│   ├── adopt.md
│   └── audit.md
├── adapters/                ← thin per-tool entry points
│   ├── claude/              ← one skill dir per runbook
│   │   ├── harness-assess/SKILL.md
│   │   ├── harness-adopt/SKILL.md
│   │   └── harness-audit/SKILL.md
│   ├── AGENTS.md.snippet
│   └── paste-prompt.md
├── patterns/                ← the control library the skills reason from
│   ├── README.md            ← index + control × layer map
│   ├── guide-file.md
│   ├── check-and-gate.md
│   ├── maintainability-sensors.md
│   ├── behaviour-test.md
│   ├── architecture-fitness.md
│   ├── inferential-review.md
│   ├── specs-and-decisions.md
│   ├── ci-and-vcs.md
│   └── drift-and-health.md
└── reference/               ← self-contained theory + templates
    ├── harness-model.md     ← the 5 layers, control types, modes, loop
    ├── readiness-rubric.md  ← harnessability score (0–100)
    ├── scorecard-template.md
    └── spec-and-adr-templates.md
```

Each `patterns/*.md` describes one control as: **intent → what good
looks like → blocking behaviour → how to detect it → recipes per
stack**. The skills read patterns to decide what to propose and how to
write it for the target stack.

---

## Relationship to the playbook

This starter is the **operational arm** of the playbook in `../playbook-md/`.
The playbook explains *why* (the harness model, the lifecycle, the
anti-patterns); this starter is *how* a team runs that thinking on their
own repo. The skills cite the relevant playbook chapters throughout, and
`reference/harness-model.md` is a self-contained condensation of the
foundations so the skills don't depend on anything outside this directory.

The one rule that shapes the whole starter, straight from the playbook:
adopt incrementally — the minimum viable harness first, then grow it one
real failure at a time — never in a single big-bang drop (the
[big-bang anti-pattern](../playbook-md/50-adoption/01-minimum-viable-harness.md#anti-patterns)).

---

## Design principles

1. **Generate, don't dump.** Write controls fitted to the target stack,
   not a fixed tree the team must prune.
2. **Assess before adopt.** Never write a control the assessment didn't
   justify. A harness the team adopts beats a bigger one they route around.
3. **Tool-neutral core, thin adapters.** The skill is markdown any agent
   runs. `AGENTS.md` is canonical; per-tool files point at it.
4. **Concept is portable; implementation is local.** Every control is
   described stack-neutrally, with recipes per stack. The control is the
   contract; the recipe is one way to meet it.
5. **Sensored or aspirational, pick one.** Every rule the adopt skill
   writes into a guide either gets a sensor or is explicitly marked soft.
6. **The harness audits the harness.** `audit` is the steering loop on
   the harness itself.
