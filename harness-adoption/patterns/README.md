# Control patterns

The library the skills reason from. Each file describes **one control**
in a portable way and gives a recipe per stack. The control is the
*contract* (intent + blocking behaviour); the recipe is one way to meet
it in a given toolchain. When a stack isn't listed, implement the
control's intent with that stack's native tooling.

Every pattern follows the same shape:

- **Intent** — what the control regulates and why.
- **What good looks like** — the closed-loop version.
- **Blocking behaviour** — what makes it a gate, not a dashboard.
- **Assessment signal** — how [assess](../skills/assess.md) detects
  whether it's present.
- **Recipes by stack** — generic + JS/TS, Python, Go, JVM (Java/Kotlin),
  .NET, Ruby. Each recipe names the stack's native tooling and describes
  the control's algorithm in prose, so it can be implemented from scratch
  with no dependency on this starter.
- **How adopt writes it** — what [adopt](../skills/adopt.md) generates.

---

## Control × layer map

| Pattern | Harness layer | In the minimum viable harness? |
|---------|---------------|--------------------------------|
| [guide-file](guide-file.md) | Feedforward | ✅ yes |
| [check-and-gate](check-and-gate.md) | (wiring for all sensors) | ✅ yes |
| [maintainability-sensors](maintainability-sensors.md) | Maintainability | format+lint+types ✅; budgets/secret-scan later |
| [behaviour-test](behaviour-test.md) | Behaviour | ✅ yes (one test) |
| [architecture-fitness](architecture-fitness.md) | Architecture fitness | later |
| [inferential-review](inferential-review.md) | Inferential | later |
| [specs-and-decisions](specs-and-decisions.md) | Feedforward | later (specs grow with the product) |
| [ci-and-vcs](ci-and-vcs.md) | (wiring + enforcement of last resort) | later (the local gate is enough week one) |

The four ✅ rows are the
[minimum viable harness](../../playbook-md/50-adoption/01-minimum-viable-harness.md):
the smallest set that still closes the loop. Adopt those first; grow the
rest by watching what fails.

---

## Reading order for the skills

`assess` reads the **Assessment signal** of every pattern.
`adopt` reads the **Recipes by stack** and **How adopt writes it** of the
one pattern it's installing.
`audit` reads the decay notes scattered through the patterns (budget
trend, bypass, noise, rot).
