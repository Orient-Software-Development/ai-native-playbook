# Paste-prompt adapter

For agents that **can't read this repo's files** (a plain chat assistant,
a tool without filesystem access). Copy the prompt below into the chat,
then paste the outputs it asks for. It runs the assessment inline without
the skill files.

For any agent that *can* read files, prefer the real skills — point it at
`harness-adoption/skills/assess.md` instead. This fallback is lossy: it
can't run detection itself, so you feed it the evidence.

---

```
You are running a harness-engineering assessment of my repository. A
"harness" has five layers — feedforward (guides the agent reads),
maintainability (lint/format/types), architecture fitness (boundaries),
behaviour (tests that prove the product works), and inferential (LLM
review). Each layer needs the loop to CLOSE: a guide shapes the work, a
sensor grades it, a GATE enforces the verdict. A check that only warns is
not a gate.

I'll give you evidence about my repo. For each layer, tell me whether the
loop is CLOSED (guide+sensor+gate), a HALF-LOOP (guide or sensor but no
gate), or OPEN (nothing). Then give me a gap list ranked by value, using
these rules: (1) readiness blockers first — missing lockfile/test runner;
(2) the minimum viable harness before anything advanced — a guide file,
one `check` command running format+lint+types, a blocking gate, and one
real behaviour test that asserts an observable end-state; (3) finishing
half-loops before opening new ones; (4) match severity to any incident I
mention. Demote LLM review and elaborate behaviour-contract systems to
"later" unless I clearly need them. End with the single next step.

Here is my evidence:
- Stack / language / package manager / lockfile:
- Guide file present (AGENTS.md / CLAUDE.md / Cursor rules / none)? Is it
  concrete or vague?
- Lint / formatter / type-check configured? Do they BLOCK or just warn?
- Is there a gate (git hook / pre-commit / CI), and does red actually
  fail the push/merge?
- Tests: is there one that proves the product's most important path end
  to end, asserting a real end-state (not just "didn't throw")?
- Architecture boundaries enforced by a machine, by review, or not at all?
- Any LLM review in CI?
- Greenfield or legacy? Team size? Any recent incident the harness should
  have caught?
```

---

Once it returns the ranked gap list, take the **single top item** to an
agent that *can* edit your repo and ask it to run the adopt step for that
control (`harness-adoption/skills/adopt.md`). Adopt one control, watch it
work, then come back for the next.
