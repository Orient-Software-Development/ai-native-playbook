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

## Contents

### 00 · Foundations — *what the whole thing rests on*
- [Why AI-Native Development](00-foundations/01-why-ai-native.md) — autocomplete vs a governed lifecycle, and why the system around the model is the whole game.
- [Context Engineering](00-foundations/02-context-engineering.md) — the context window is a budget, not a backpack; the right context, not the most.
- [Harness Engineering](00-foundations/03-harness-engineering.md) — guides (before) and sensors (after); computational before inferential; grow it, don't design it.

### 10 · Lifecycle — *running one change, end to end*
- [Spec — The Contract](10-lifecycle/01-spec-the-contract.md) — a short, normative, business-legible statement of *what* and how you'll know it's done.
- [Plan Before Code](10-lifecycle/02-plan-before-code.md) — explore → plan → code; catch a wrong approach while it's still a paragraph.
- [Task Slicing](10-lifecycle/03-task-slicing.md) — one deliverable, one test, one commit; bottom-up by layer.
- [Code With the Agent](10-lifecycle/04-code-with-the-agent.md) — steer with the concrete thing, not an analogy; enforcement lives outside the agent.
- [Verify — Proof, Not Vibes](10-lifecycle/05-verify-proof-not-vibes.md) — grade against the spec, by someone other than the author, with evidence attached.
- [Review and Convergence](10-lifecycle/06-review-and-convergence.md) — read the diff deliberately; extract the shared piece before the third copy.

### 20 · Harness — *the controls you build*
- [Guides — Feedforward](20-harness/01-guides-feedforward.md) — instruction files, blueprints, anti-cheat constraints, and the computational guides that barely cost a token.
- [Repo Structure and Legibility](20-harness/02-repo-structure-and-legibility.md) — how the repo is laid out *is* a guide it broadcasts for free.
- [Sensors — Feedback](20-harness/03-sensors-feedback.md) — a layered gate, computational-first, resting on sensor integrity: passing it must require the work.
- [Keep Quality Left](20-harness/04-keep-quality-left.md) — each check at the earliest stage whose speed it can afford, and no further.
- [Behaviour Harness](20-harness/05-behaviour-harness.md) — bind the test to the invariant, not the code; property-based and mutation testing.

### 30 · Delivery — *shipping at agent throughput*
- [Trunk-Based Development](30-delivery/01-trunk-based-development.md) — one shared line, small verified slices, merged behind a green gate.
- [CI and CD](30-delivery/02-ci-and-cd.md) — CI gates the merge, CD gates the release; keep the seam.
- [Observability](30-delivery/03-observability.md) — the sensor that extends past the merge gate into production.
- [Drift and Health Sensors](30-delivery/04-drift-and-health-sensors.md) — controls that run on a clock, to catch the rot no single change causes.

### 40 · Anti-Patterns — *how it goes wrong*
- [Failure Modes](40-anti-patterns/01-failure-modes.md) — one root cause (task-complete, not task-correct) in eight disguises, each paired with its fix.

### 50 · Adoption — *getting there from where you are*
- [The Minimum Viable Harness](50-adoption/01-minimum-viable-harness.md) — the smallest loop that still closes, standable in week one.
- [Growing the Harness](50-adoption/02-growing-the-harness.md) — grow from real failures, prune as the model improves.
- [The Responsible Team and AI Debt](50-adoption/03-responsible-team-and-ai-debt.md) — allocate trust by zone; manage AI debt as a budget; a human owns every merge.
- [Legacy and Brownfield](50-adoption/04-legacy-and-brownfield.md) — earn a foothold one high-risk seam at a time; net before you trapeze.
- [Scoring an Existing Harness](50-adoption/05-scoring-an-existing-harness.md) — a five-layer rubric an agent can score any repo against, with evidence attached to every number — the working answer to "how do you grade the harness?"

### Reference
- [Glossary](glossary.md) — plain-language definitions of every term of art, each linked to the page that treats it most fully.
- [Scorecard Template](templates/harness-scorecard.md) + [Scoring Prompt](templates/harness-scoring-prompt.md) — the fillable report and the agent instruction that drive the scoring chapter above.

---

*Start here:* [Why AI-Native Development →](00-foundations/01-why-ai-native.md)
