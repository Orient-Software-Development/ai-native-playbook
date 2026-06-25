# The AI-Native Harness Playbook

*Start here. Run three skills on your own repo, then read the topics that matter to you.*

---

## Start with a story

Imagine you hand a capable new engineer a ticket: *"Add password reset to our login page."*

They're fast. They're tireless. They never get bored. But they've also never seen your codebase, they don't know your conventions, they can't tell which tests actually matter, and when they say "done," they genuinely believe it — even when the feature silently breaks in production.

That new engineer is a **coding agent**. And the difference between it being a liability and being the best thing that ever happened to your team is **not** a smarter model. It's the **system you build around the model**: the instructions it reads before it acts, and the checks that grade its work after.

This book calls that system the **harness**. The fastest way to start is not to read the whole book — it's to point an agent at your own repo and let it tell you where your harness is thin. That's what the three skills below do.

> **The one idea, if you remember nothing else:** Humans steer, agents execute. The payoff comes from the harness *around* the model — the **guides** that shape what the agent does before it acts, and the **sensors** that grade what it produced after.

---

## Get started: run the three skills

The [`harness-adoption/`](harness-adoption/) kit ships three **skills** — tool-neutral runbooks an agent executes on your repo. They map to the playbook's steering loop: **assess → adopt → audit**. Run them in order the first time; after that, `audit` is the recurring ritual.

You don't need to read any code. You point your agent at a skill and let it run.

### Step 1 · Assess — *where is my harness today?*
Read-only. The agent detects your stack, your AI coding tool, and the controls you already have, then scores the five harness layers and hands you a **ranked gap list**.

- Skill: [`harness-adoption/skills/assess.md`](harness-adoption/skills/assess.md)
- Produces a scorecard — nothing is written to your repo.

### Step 2 · Adopt — *install the smallest harness that fits*
The agent takes the gap list and proposes the **smallest fitted set of controls** for your stack and tool — a guide file, a single `check` command, a blocking gate, one real behaviour test — and writes only what you approve, then re-scores.

- Skill: [`harness-adoption/skills/adopt.md`](harness-adoption/skills/adopt.md)
- Everything it writes is **generated into your repo and owned by you**, in your stack's idiom — never imported as a library.

### Step 3 · Audit — *keep it honest over time*
Every 4–8 weeks, re-run the assessment on a repo that already has a harness. The agent reports **drift** and proposes the next increment.

- Skill: [`harness-adoption/skills/audit.md`](harness-adoption/skills/audit.md)

### How to launch them (pick your tool)

| Your tool | Entry point |
|-----------|-------------|
| **Claude Code** | `/harness-assess` — see [`adapters/claude/SKILL.md`](harness-adoption/adapters/claude/SKILL.md) |
| **Copilot / Cursor / Codex / local agents** | drop [`adapters/AGENTS.md.snippet`](harness-adoption/adapters/AGENTS.md.snippet) into your `AGENTS.md`, then ask the agent to *"run the harness assessment"* |
| **Any chat agent (no repo access)** | paste [`adapters/paste-prompt.md`](harness-adoption/adapters/paste-prompt.md) |

The full kit — including the control library it reasons from and the self-contained theory — is documented in [`harness-adoption/README.md`](harness-adoption/README.md).

**That's the loop.** Assess, adopt the minimum, ship real changes through it, audit on a clock. Everything below makes that loop sharper.

---

## Then read the topics that matter to you

Once the skills have shown you where you stand, dive into the chapters relevant to your gaps and interests. The playbook is six sections; here's where to go for what.

### "I want to understand *why* before I do anything." → Foundations
- [Why AI-Native Development](playbook-md/00-foundations/01-why-ai-native.md) — autocomplete vs. a governed lifecycle, and why the system around the model is the whole game.
- [Context Engineering](playbook-md/00-foundations/02-context-engineering.md) — the context window is a budget, not a backpack; give the right context, not the most.
- [Harness Engineering](playbook-md/00-foundations/03-harness-engineering.md) — guides (before) and sensors (after); computational before inferential; grow it, don't design it.

### "Show me how one change moves end to end." → Lifecycle
- [Spec — The Contract](playbook-md/10-lifecycle/01-spec-the-contract.md) — a short, business-legible statement of *what* and how you'll know it's done.
- [Plan Before Code](playbook-md/10-lifecycle/02-plan-before-code.md) — explore → plan → code; catch a wrong approach while it's still a paragraph.
- [Task Slicing](playbook-md/10-lifecycle/03-task-slicing.md) — one deliverable, one test, one commit.
- [Code With the Agent](playbook-md/10-lifecycle/04-code-with-the-agent.md) — steer with the concrete thing, not an analogy.
- [Verify — Proof, Not Vibes](playbook-md/10-lifecycle/05-verify-proof-not-vibes.md) — grade against the spec, with evidence attached.
- [Review and Convergence](playbook-md/10-lifecycle/06-review-and-convergence.md) — read the diff deliberately; extract the shared piece before the third copy.

### "My `adopt` run said I need a control — how do I build it well?" → Harness
- [Guides — Feedforward](playbook-md/20-harness/01-guides-feedforward.md) — instruction files, blueprints, anti-cheat constraints, and computational guides that barely cost a token.
- [Repo Structure and Legibility](playbook-md/20-harness/02-repo-structure-and-legibility.md) — how the repo is laid out *is* a guide it broadcasts for free.
- [Sensors — Feedback](playbook-md/20-harness/03-sensors-feedback.md) — a layered gate, computational-first, resting on sensor integrity.
- [Keep Quality Left](playbook-md/20-harness/04-keep-quality-left.md) — each check at the earliest stage whose speed it can afford.
- [Behaviour Harness](playbook-md/20-harness/05-behaviour-harness.md) — bind the test to the invariant, not the code; property-based and mutation testing.

### "The agent produces good changes — now I need to ship them fast and safely." → Delivery
- [Trunk-Based Development](playbook-md/30-delivery/01-trunk-based-development.md) — one shared line, small verified slices, merged behind a green gate.
- [CI and CD](playbook-md/30-delivery/02-ci-and-cd.md) — CI gates the merge, CD gates the release; keep the seam.
- [Observability](playbook-md/30-delivery/03-observability.md) — the sensor that extends past the merge gate into production.
- [Drift and Health Sensors](playbook-md/30-delivery/04-drift-and-health-sensors.md) — controls that run on a clock, to catch the rot no single change causes.

### "It's going wrong and I want to recognize the pattern." → Anti-Patterns
- [Failure Modes](playbook-md/40-anti-patterns/01-failure-modes.md) — one root cause (task-complete, not task-correct) in eight disguises, each paired with its fix.

### "I'm not greenfield — meet me where I actually am." → Adoption
- [The Minimum Viable Harness](playbook-md/50-adoption/01-minimum-viable-harness.md) — the smallest loop that still closes, standable in week one.
- [Growing the Harness](playbook-md/50-adoption/02-growing-the-harness.md) — grow from real failures, prune as the model improves. *(This is the loop the three skills run.)*
- [The Responsible Team and AI Debt](playbook-md/50-adoption/03-responsible-team-and-ai-debt.md) — allocate trust by zone; manage AI debt as a budget; a human owns every merge.
- [Legacy and Brownfield](playbook-md/50-adoption/04-legacy-and-brownfield.md) — earn a foothold one high-risk seam at a time.
- [Scoring an Existing Harness](playbook-md/50-adoption/05-scoring-an-existing-harness.md) — the five-layer rubric the `assess` skill scores against, with evidence attached to every number.

---

## Who this is for

- **Non-technical readers** — product owners, founders, managers. You don't need to read code. Run the skills with your team's agent, read the Foundations, and use the Adoption section to steer.
- **Technical readers** — engineers, leads, architects. Run the skills, then read Lifecycle and Harness as your working manual.
- **Coding agents** — if a human pointed you here, start at [`harness-adoption/skills/assess.md`](harness-adoption/skills/assess.md), follow the lifecycle (spec → plan → code → verify → review → ship), and cite the playbook chapters you rely on.

---

## How this playbook is organized

The book ships in two formats, one chapter per file, mirrored across both:

- **[`playbook-md/`](playbook-md/)** — the canonical Markdown source. Read it on GitHub, in an editor, or feed it to an agent. Every chapter link above points here.
- **[`playbook-html/`](playbook-html/)** — a styled, self-contained HTML edition with bespoke SVG/CSS visualizations. Open [`playbook-html/index.html`](playbook-html/index.html) in a browser.

Plus the operational kit and reference material:

- **[`harness-adoption/`](harness-adoption/)** — the three skills (`assess`, `adopt`, `audit`), the control library they reason from, and self-contained theory. The operational arm of the playbook.
- [Glossary](playbook-md/glossary.md) — plain-language definitions of every term of art.
- [Scorecard Template](playbook-md/templates/harness-scorecard.md) + [Scoring Prompt](playbook-md/templates/harness-scoring-prompt.md) — the fillable report and the agent instruction behind the scoring chapter.
- [References](playbook-md/references.md) — the outside research behind the playbook.

---

*Ready? Point your agent at the first skill:* [**Assess your repo →**](harness-adoption/skills/assess.md)
