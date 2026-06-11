# Pattern: the behaviour test

## Intent

Prove the code does what the **product** needs, not just what the code
does. One end-to-end test of the single most important path — a user can
log in, an order saves and reads back, the core calculation returns the
right number. This is the seed of the behaviour harness, and the piece
teams skip first and regret first. See
`../../playbook-md/20-harness/05-behaviour-harness.md`.

## What good looks like

- **Against the spec, not the code.** It asserts what the product must
  do, so it can catch the code doing the wrong thing correctly.
- **Observable end-state.** It asserts the row that persisted, the value
  the user sees, the response body — never "it ran without throwing".
  A test that only checks for absence of exceptions is a
  [vacuous sensor](../../playbook-md/40-anti-patterns/01-failure-modes.md).
- **One real path first.** Not 100 tests of trivia. One test of the thing
  that would page someone at 2am if it broke.
- **In the gate.** Added to `check` (or pre-push/CI if slow) so red blocks.

## Blocking behaviour

The behaviour test runs in the gate and **fails the push/merge when the
behaviour breaks**. If it's fast, fold into `check`; if it needs a
running service or a browser, run it pre-push or in CI — keep it off the
ms-budget pre-commit path.

## Assessment signal

- Is there a test runner configured and at least one passing test?
- Is there a test that exercises the product's main path end to end, or
  only unit tests of internals?
- Do tests assert end-states, or just "no error"? (Sample a few.)
- Is e2e feasible for this stack (web app / HTTP API = yes; pure library
  = the "behaviour" test is a high-level integration test instead)?

## Recipes by stack

### The runner / e2e tool

| Stack | Unit/integration | End-to-end |
|-------|------------------|------------|
| **JS/TS** | Vitest / Jest | Playwright / Cypress |
| **Python** | pytest | Playwright-python / requests against a live app / Django test client |
| **Go** | `go test` | `httptest` + table tests; `rod`/`chromedp` for browser |
| **JVM** | JUnit | Testcontainers + REST-assured; Selenium/Playwright-java |
| **.NET** | xUnit/NUnit | `WebApplicationFactory` integration tests; Playwright-dotnet |
| **Ruby** | RSpec/Minitest | Capybara (system specs); request specs |

### Shape of the one test (any stack)

```
arrange:  set up the minimal real state (a user, a product, a tenant)
act:      drive the most important path the way a user/caller would
assert:   the OBSERVABLE end-state — the persisted row, the returned
          value, the rendered text — equals what the product requires
```

For a non-UI library: the "behaviour test" is the highest-level public
API call that proves the core promise, asserting the returned result.

## How adopt writes it

1. Ask the user (or infer from the README/specs) the single most
   important thing the product must do.
2. Pick the runner already in the repo; only add an e2e tool if none
   exists and the stack supports it.
3. Write **one** test against that path, asserting an observable
   end-state. Resist writing a suite — one real test now beats a backlog
   later.
4. Add it to `check` (fast) or pre-push/CI (slow).
5. **Smoke-check it bites:** break the behaviour (or assert the wrong
   value) and confirm the test goes red; restore. A behaviour test you
   never watched fail is not yet trusted.

## Decay notes (for audit)

- Coverage of *critical paths*, not coverage percentage, is the metric.
- Watch for tests weakened over time into "did not throw" assertions —
  that's a behaviour test rotting into a vacuous one.
