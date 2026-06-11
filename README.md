# The AI-Native Harness Playbook

> Humans steer, agents execute. The payoff doesn't come from a smarter model — it comes from the system
> *around* the model: the **guides** that shape what the agent does before it acts, and the **sensors** that
> grade what it produced after. This book is how you build that system for your codebase, and how you run the
> governed lifecycle — spec → plan → code → verify → review → ship — that turns a capable agent into
> production-grade delivery.

A coding agent is a model plus a **harness**. You don't train the model; you choose it. What you *own* is the
harness — and the same model in a sharper harness produces meaningfully better work. These chapters are the
practice of engineering that harness and running the lifecycle on top of it.

Read the foundations first; the rest can be read in order or dipped into by topic.

---

## Repository layout

The playbook ships in two formats, with one chapter per file mirrored across both:

- **[`playbook-md/`](playbook-md/)** — the canonical Markdown source. Read it on GitHub, in an editor, or feed it to an agent. The Contents below link here.
- **[`playbook-html/`](playbook-html/)** — a styled, self-contained HTML edition with bespoke SVG/CSS visualizations. Open [`playbook-html/index.html`](playbook-html/index.html) in a browser, or start from a chapter such as [`playbook-html/00-foundations/01-why-ai-native.html`](playbook-html/00-foundations/01-why-ai-native.html). All pages share [`playbook-html/styles.css`](playbook-html/styles.css).

Both trees carry the same six sections (`00`–`50`), plus a glossary, a scorecard template, and a scoring prompt.

---

## Contents

### 00 · Foundations — *what the whole thing rests on*
- [Why AI-Native Development](playbook-md/00-foundations/01-why-ai-native.md) — autocomplete vs a governed lifecycle, and why the system around the model is the whole game.
- [Context Engineering](playbook-md/00-foundations/02-context-engineering.md) — the context window is a budget, not a backpack; the right context, not the most.
- [Harness Engineering](playbook-md/00-foundations/03-harness-engineering.md) — guides (before) and sensors (after); computational before inferential; grow it, don't design it.

### 10 · Lifecycle — *running one change, end to end*
- [Spec — The Contract](playbook-md/10-lifecycle/01-spec-the-contract.md) — a short, normative, business-legible statement of *what* and how you'll know it's done.
- [Plan Before Code](playbook-md/10-lifecycle/02-plan-before-code.md) — explore → plan → code; catch a wrong approach while it's still a paragraph.
- [Task Slicing](playbook-md/10-lifecycle/03-task-slicing.md) — one deliverable, one test, one commit; bottom-up by layer.
- [Code With the Agent](playbook-md/10-lifecycle/04-code-with-the-agent.md) — steer with the concrete thing, not an analogy; enforcement lives outside the agent.
- [Verify — Proof, Not Vibes](playbook-md/10-lifecycle/05-verify-proof-not-vibes.md) — grade against the spec, by someone other than the author, with evidence attached.
- [Review and Convergence](playbook-md/10-lifecycle/06-review-and-convergence.md) — read the diff deliberately; extract the shared piece before the third copy.

### 20 · Harness — *the controls you build*
- [Guides — Feedforward](playbook-md/20-harness/01-guides-feedforward.md) — instruction files, blueprints, anti-cheat constraints, and the computational guides that barely cost a token.
- [Repo Structure and Legibility](playbook-md/20-harness/02-repo-structure-and-legibility.md) — how the repo is laid out *is* a guide it broadcasts for free.
- [Sensors — Feedback](playbook-md/20-harness/03-sensors-feedback.md) — a layered gate, computational-first, resting on sensor integrity: passing it must require the work.
- [Keep Quality Left](playbook-md/20-harness/04-keep-quality-left.md) — each check at the earliest stage whose speed it can afford, and no further.
- [Behaviour Harness](playbook-md/20-harness/05-behaviour-harness.md) — bind the test to the invariant, not the code; property-based and mutation testing.

### 30 · Delivery — *shipping at agent throughput*
- [Trunk-Based Development](playbook-md/30-delivery/01-trunk-based-development.md) — one shared line, small verified slices, merged behind a green gate.
- [CI and CD](playbook-md/30-delivery/02-ci-and-cd.md) — CI gates the merge, CD gates the release; keep the seam.
- [Observability](playbook-md/30-delivery/03-observability.md) — the sensor that extends past the merge gate into production.
- [Drift and Health Sensors](playbook-md/30-delivery/04-drift-and-health-sensors.md) — controls that run on a clock, to catch the rot no single change causes.

### 40 · Anti-Patterns — *how it goes wrong*
- [Failure Modes](playbook-md/40-anti-patterns/01-failure-modes.md) — one root cause (task-complete, not task-correct) in eight disguises, each paired with its fix.

### 50 · Adoption — *getting there from where you are*
- [The Minimum Viable Harness](playbook-md/50-adoption/01-minimum-viable-harness.md) — the smallest loop that still closes, standable in week one.
- [Growing the Harness](playbook-md/50-adoption/02-growing-the-harness.md) — grow from real failures, prune as the model improves.
- [The Responsible Team and AI Debt](playbook-md/50-adoption/03-responsible-team-and-ai-debt.md) — allocate trust by zone; manage AI debt as a budget; a human owns every merge.
- [Legacy and Brownfield](playbook-md/50-adoption/04-legacy-and-brownfield.md) — earn a foothold one high-risk seam at a time; net before you trapeze.
- [Scoring an Existing Harness](playbook-md/50-adoption/05-scoring-an-existing-harness.md) — a five-layer rubric an agent can score any repo against, with evidence attached to every number — the working answer to "how do you grade the harness?"

### Reference
- [Glossary](playbook-md/glossary.md) — plain-language definitions of every term of art, each linked to the page that treats it most fully.
- [Scorecard Template](playbook-md/templates/harness-scorecard.md) + [Scoring Prompt](playbook-md/templates/harness-scoring-prompt.md) — the fillable report and the agent instruction that drive the scoring chapter above.

---

*Start here:* [Why AI-Native Development →](playbook-md/00-foundations/01-why-ai-native.md)
