# Behaviour Harness

> The hardest harness to build is the one that proves the code does what the *business* wanted — not just that it's clean or well-shaped. The lever is the **invariant**: state what must always hold, bind a test to that rule rather than to the code the agent wrote, and you escape the agent's favourite failure — tests that merely confirm the implementation.

Breadcrumb: [Playbook](../README.md) › Harness

## The principle

[Foundations](../00-foundations/03-harness-engineering.md) named three things a harness regulates, hardest
last: **maintainability** (is the code clean?), **architecture fitness** (is the system the right shape?),
and **behaviour** (does it do what the business actually wanted?). The first two pages of concrete sensors
mostly served the first two categories. This page is the third — and it's the hard one, still unsolved for
high-autonomy work, the reason a human stays in the loop.

Here's what makes it hard. Maintainability and shape can be checked against the *code itself* — a linter
reads the code, a fitness function reads the dependency graph. But "does this do what the business wanted?"
can't be answered from the code, because the code *is* the agent's answer to that question. And here the
agent has a specific, predictable failure — driven not by *who* wrote the test but by *what was in front of
it when the test was written*. Handed the job of testing its own work *with only that code in context*, it
reverse-engineers a test from the code it just wrote, so the test proves the code does what it does — never
what the business *needed*. A green suite built that way is a mirror, not a check. The cure is not to
distrust every agent-written test — it's to change the **source the test is derived from**: give the agent
the spec or the invariant, not just the code, and it writes a test that can actually disagree with the
implementation. That switch of source is the whole of this page. (The failure gets its own catalogue entry
in [anti-patterns](../40-anti-patterns/01-failure-modes.md); here we build the defence.)

The defence is to derive the sensor from the *intent*, not the code. And the tool for that is the
**invariant**: a statement of what must *always* be true for a scenario, no matter the input or the
implementation. "After any transfer between accounts, the total money across all accounts is unchanged."
"A posted invoice can never have a negative line total." "Sorting a list and sorting it again gives the
same list." An invariant is a property of the *problem*, so a test built from it is bound to the business
rule — it can't silently encode the code's current bug, because it was never derived from the code.

The practice that operationalises this names each scenario and pins its invariant:

- **Name the scenario.** Give each behaviour contract a stable identifier — an **SCN** (a *scenario id*,
  e.g. `SCN-LEDGER-014`). The name is the anchor: the test cites the scenario, not the function.
- **State its invariant precisely**, in business language, and have the person who owns the *intent* (a
  domain owner, not the implementer) approve it. That approved description is the contract the behaviour
  test proves — the [spec-as-contract](../10-lifecycle/01-spec-the-contract.md) idea, made executable.
- **Trace it both ways.** Every approved scenario must have a test; every test must cite a real scenario.
  Then a dropped contract is *visible* — a scenario with no test stands out — instead of silently going
  uncovered.

For UI, "behaviour" includes "does it look right," which is subjective and exactly what the agent can't
self-grade. Two guides from earlier turn it gradable: a [design guide](01-guides-feedforward.md) (checked-in
tokens) replaces "looks right" with named values, and captured
[evidence images](../10-lifecycle/05-verify-proof-not-vibes.md) let a reviewer *see* the rendered result
rather than trust that the DOM held some value.

## Why it works

A test is only as honest as the thing it's derived from. Derive it from the code and it inherits the code's
bugs; derive it from the invariant and it stands *outside* the code, able to disagree with it. That's the
whole move — and two techniques sharpen it, because each attacks the confirms-the-implementation failure
from a different side.

**Property-based testing** changes how the test gets its inputs. A normal test picks an example by hand —
and when the agent picks it, it picks one that exercises the path it just wrote. A property-based test
instead asserts the *invariant* and lets a tool **generate** the inputs — hundreds of them, including the
weird edges no one thought to type. Because the property comes from the spec's meaning — grounded in the
docstring, the types, the names, *what the function is supposed to do* — it's independent of the function
body, so it catches the case the agent's hand-picked example skipped. You assert "the total is conserved"
and the tool hunts for any input that breaks it; the overdraw case the agent never tested is found *for*
you.

**Mutation testing** tests the tests themselves. It injects small deliberate bugs into your source — flips a
`<` to `<=`, a `+` to `-`, deletes a line — to produce a "mutant," then reruns the suite. If a test fails,
the mutant is *killed* — good, the suite noticed. If the suite still passes, the mutant **survived** — and a
surviving mutant is proof, in one concrete example, that your tests would *not* have caught that bug. This
is the empirical answer to the [coverage illusion](03-sensors-feedback.md): coverage tells you a line ran,
but a survived mutant tells you that line ran *and nothing checked the result*. A green, high-coverage suite
with surviving mutants is toothless, and mutation testing is what makes the toothlessness visible.

Put them in a loop and they compound: **property-based tests** write spec-derived assertions → **mutation
score** grades how strong those assertions really are → the **surviving mutants** become a worklist you hand
back to the agent. And the agent is genuinely good at *this* last step — given a concrete surviving mutant
("when I changed this line, no test complained"), writing the assertion that kills it is a well-specified
task, not an open-ended one.

## How to apply it

- **Start from the invariant, not the example.** For each scenario, ask "what must be true here *no matter
  the input*?" and write that down. The example test comes second, if at all — the invariant is the
  contract.
- **Name and pin each scenario; bind the test to the name.** Give every behaviour contract a stable id, cite
  it from the test, and keep the trace bidirectional so a missing test is loud, not silent.
- **Let the person who owns the intent approve the contract.** The domain owner signs off on the invariant,
  independently of whoever (or whatever) implements it — so the test proves the *business's* need, not the
  implementer's guess at it.
- **Use property-based tests for invariants you can state.** Assert the property; let the tool generate the
  inputs; ground the property in the spec's semantics (docstring, types, names), never in the function body.
- **Grade the suite with mutation testing, not coverage.** Run it on the high-value modules, treat a high
  coverage number with surviving mutants as the warning it is, and feed the survivors back to the agent to
  kill.
- **Make UI quality gradable.** Check in design tokens so "looks right" becomes a named-value match, and
  capture an evidence image on the passing run so a reviewer sees the real rendered screen — not a green
  check that only proves a locator resolved.
- **Keep the human on the question of intent.** A behaviour harness can prove the code satisfies the
  invariant; it can't prove the *invariant itself* is what the business meant. That call, plus the
  taste and organisational-context judgements an agent has no [feel for](../00-foundations/03-harness-engineering.md),
  stays with a person — and a separate, skeptical reviewer grades the result against the spec rather than
  letting the author [self-certify](../10-lifecycle/05-verify-proof-not-vibes.md).
- **Don't:** let the agent write the behaviour test from the code it just wrote; treat a coverage percentage
  as proof of behaviour; or treat a green structure-only UI test as proof the screen is actually right.

## In practice

A teammate asks the agent to build `transfer(from, to, amount)` — move money between two accounts.

**Without the behaviour harness.** The agent writes the function, then writes its own test: transfer $100
from A to B, assert A dropped by $100 and B rose by $100. Green. But that test was reverse-engineered from
the happy path the agent just coded — it asserts exactly what the code does. And the code has a bug: on a
transfer that would overdraw `from`, it deducts the full amount from A but silently caps the credit to B,
so the difference simply vanishes. The clean-$100 example never touches that branch, so the suite stays
green and money leaks in production behind a passing test.

**With the behaviour harness.** The scenario is named and its invariant is approved by the domain owner
before code is written: *after any transfer, the sum of all account balances is unchanged* — money is
neither created nor destroyed. A **property-based test** asserts exactly that and generates hundreds of
transfers, including the overdraw case nobody hand-wrote; the conservation property fails on it, and the bug
surfaces *before merge*, from a test derived from the rule rather than the code. Then **mutation testing**
runs on the module: it flips the sign on the deduction, the property test catches the mutant — confirming
the suite has real teeth — while a *surviving* mutant in a rounding helper the property didn't cover is
handed to the agent, which writes the assertion that kills it. And because the balances also render on a
screen, a captured **evidence image** on the green run lets the reviewer see the figures are right, not just
trust that the DOM had numbers in it.

The lesson the example carries: the behaviour harness works by binding the check to the business *rule* —
the invariant, named as a scenario and owned by the domain — instead of to the code the agent wrote, which
is the only way out of tests that just confirm the implementation. Property-based testing derives the test
from the rule and generates the inputs that break it; mutation testing proves the suite can catch a bug at
all; evidence images make subjective UI gradable; and a human still owns the one thing none of it can
settle — whether the rule is what the business actually meant.

## Anti-patterns

- **The confirms-the-implementation test.** A test reverse-engineered from the code it's meant to check, so
  it proves the code does what it does — not what was needed. The default *when the code, not the spec, is
  the source* the test is derived from — and the failure this whole page exists to prevent.
  ([Sensors — Feedback](03-sensors-feedback.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md))
- **The coverage mirage.** High coverage, surviving mutants — a suite that *executes* everything and
  *verifies* nothing. Coverage measured the wrong thing; mutation score measures the right one.
  ([Sensors — Feedback](03-sensors-feedback.md))
- **The unowned contract.** A behaviour test with no domain-approved invariant behind it, so nobody ever
  confirmed the rule it checks is the business's actual intent.
  ([Spec — The Contract](../10-lifecycle/01-spec-the-contract.md))
- **The DOM-only "looks right."** Treating a passing structure-only UI test as proof the screen renders
  correctly — no evidence image, no design-token grading — so a visually broken but DOM-intact page ships
  green. ([Verify — Proof, Not Vibes](../10-lifecycle/05-verify-proof-not-vibes.md) · [Guides — Feedforward](01-guides-feedforward.md))
- **The closed loop on intent.** Letting the harness convince you the behaviour question is settled, with no
  separate human check that the result is what the business meant.
  ([Responsible Team and AI Debt](../50-adoption/03-responsible-team-and-ai-debt.md))

---
[← Previous: Keep Quality Left](04-keep-quality-left.md) · [Contents](../README.md) · [Next → Trunk-Based Development](../30-delivery/01-trunk-based-development.md)

Related: [Harness Engineering](../00-foundations/03-harness-engineering.md) · [Guides — Feedforward](01-guides-feedforward.md) · [Sensors — Feedback](03-sensors-feedback.md) · [Spec — The Contract](../10-lifecycle/01-spec-the-contract.md) · [Verify — Proof, Not Vibes](../10-lifecycle/05-verify-proof-not-vibes.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md) · [Responsible Team and AI Debt](../50-adoption/03-responsible-team-and-ai-debt.md)
