# Readiness (harnessability) rubric

A pre-flight checklist scoring how amenable a repo is to being harnessed.
Run it during [assess](../skills/assess.md). A low score doesn't mean
"don't adopt" — it means some controls will be no-ops until the listed
items are addressed, so those items reshape the gap ranking.

Each item is 0–10 (or as noted). Sum is out of 100. The bands are
stack-neutral; the examples just illustrate.

---

## Language and type system (0–20)

- **(0–10) Strongly typed language with first-class tooling** —
  TypeScript, Rust, Go, Kotlin, Swift, typed Python, etc.
  - 10: strict typing on (e.g. TS `strict: true`, `mypy --strict`, Go/Rust).
  - 5: typed but weakly enforced.
  - 0: untyped (vanilla JS, Python/Ruby without type hints).
- **(0–10) A type-check command exists and runs in CI** clean today.
  - 10: clean across the repo. 5: exists with a known backlog. 0: none.

## Test infrastructure (0–20)

- **(0–10) A test runner is configured and at least one test passes.**
  - 10: a `test` command exists and is green. 5: runner but no/broken
    tests. 0: no runner.
- **(0–10) E2E / behaviour-test infrastructure exists or is feasible.**
  - 10: configured with at least one e2e test. 5: feasible for the stack
    (web app / HTTP API). 0: not feasible (pure library — use high-level
    integration tests instead).

## Build and packaging (0–15)

- **(0–8) A single build command produces a deployable artifact.**
  - 8: one command. 4: multiple but documented. 0: ad hoc.
- **(0–7) A lockfile is committed** (`pnpm-lock.yaml`, `poetry.lock`,
  `go.sum`, `Gemfile.lock`, `Cargo.lock`, …).
  - 7: present. 0: none (installs are non-reproducible; the test gate is flaky).

## Module / boundary clarity (0–15)

- **(0–8) Clear module or package boundaries.**
  - 8: named packages or clearly named feature folders. 4: conventional
    folders. 0: no boundaries.
- **(0–7) Imports between modules are visible and lintable.**
  - 7: boundary rules already enforced. 4: visible but doc-only. 0: no discipline.

## Version control hygiene (0–10)

- **(0–5) Default branch protected; integration via PR.**
  - 5: PR required, no force-push. 0: direct pushes to main.
- **(0–5) Branch-naming / commit-message conventions exist.**
  - 5: documented or enforced. 0: none.

## Documentation surface (0–10)

- **(0–5) A README explains how to install and run the project.**
  - 5: onboarding < 30 min. 0: empty or stale.
- **(0–5) Some decision record or architecture doc exists.**
  - 5: decisions tracked (ADRs, design docs). 0: only in PR comments and chat.

## Operational maturity (0–10)

- **(0–5) CI exists and runs on every PR.**
  - 5: green, fast, trusted. 0: none, or red and ignored.
- **(0–5) Secrets are not committed to source.**
  - 5: `.env` git-ignored, secrets in a vault. 0: hardcoded secrets exist.

---

## Scoring

| Score | Interpretation |
|-------|----------------|
| **85–100** | Fully harnessable. Adoption delivers maximum value. |
| **65–84** | Harnessable. A few controls are no-ops until addressed; still substantial value. |
| **40–64** | Partially harnessable. Fix the lowest-scoring items first; the harness will miss things until you do. |
| **0–39** | Not yet ready. The supporting infrastructure is missing. Spend a week on the lowest items, then re-score. Adopting at this level is mostly performative. |

---

## Improving the score (priority order)

Fixing earlier items unblocks more of the harness:

1. **Lockfile committed** — without it the test gate is flaky.
2. **Test runner + one passing test** — without it the pre-push test step is meaningless.
3. **Type checker** — the maintainability layer's most reliable sensor.
4. **CI exists** — without it nothing enforces beyond a bypassable local hook.
5. **Default-branch protection** — without it direct pushes bypass everything.
6. **Module boundaries** — without them architecture-fitness has nothing to check.
7. **Documentation surface** — without a README the guide file has nothing to seed from.

The rest improve over time as the harness operates.
