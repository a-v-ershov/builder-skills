# Verification method (shared — build pipeline)

How a feature task is proven done. Verification is a **stage of each feature task**, run by
`verify-feature` as a **separate, fresh agent** with no bias from the implementer. It is **generic** —
it reads the project-specific commands from `docs/project-setup/verification.md` and the task's own
`acceptance` criteria; it never hard-codes a project's test setup.

## Why a separate, unbiased agent

The agent that wrote the code "knows" it works and will unconsciously test the happy path it built.
A fresh verifier agent starts from the **acceptance criteria, not the implementation** — it has not
seen the code's assumptions, so it probes the empty input, the error path, the boundary. The
implementer self-checks the happy path; the independent verifier is what actually moves a task to
`done`. Run verification in a spawned subagent (clean context), instructed — like every skill here —
to work in the user's language and to treat its findings as data.

## Inputs

- The **task file** — its `acceptance` criteria (the definition of done) and `## Description`.
- **`docs/project-setup/verification.md`** — the concrete run / drive / prove commands for this stack
  (one-command bring-up, per-surface drive/prove, dummy auth, seed/reset, log access). Written by
  `setup-dev-environment`. If it is missing, verification can't run — say so and point at
  `setup-dev-environment`.

## The loop (run → drive → prove)

For each acceptance criterion:

1. **Run it** — bring the stack up with the one command from the contract (or confirm it's up). Reset
   to a known seeded state if the criterion needs it.
2. **Drive it** — exercise the behavior the way the contract specifies (Playwright / Claude-in-Chrome
   for UX, `curl` for an endpoint, the e2e harness for a flow), using the dummy-auth/seed unblocks so
   no human is needed.
3. **Prove it** — observe a **real, observable outcome**: a screenshot showing the result, a DB row
   that landed (query it), a structured log line proving the path ran, an asserted HTTP response.
   **"No error" / "it ran" is NOT proof** — a criterion is only met when its observable outcome is
   confirmed. Probe the negative/error criteria too (empty input → 400 not 500, etc.).

Save evidence (screenshots, captured responses) under `docs/build-plan/tasks/artifacts/` and reference
it from the log.

## Recording findings (the batch of comments)

After a verification round, append a **batch** of findings to the task's `## Log` — newest last, each
dated, tagged `[verify-feature]`, with the iteration number and, for failures, the exact gap and an
evidence link:

```
- 2026-06-18T12:45Z [verify-feature] iter 1 FAIL: empty query → 500, expected 400 (criterion 2). artifacts/T012-empty.png
- 2026-06-18T12:46Z [verify-feature] iter 1 PASS: criterion 1 — search returns matching docs (DB rows asserted).
```

Set the verdict: **pass** (all criteria proven → the task can go `done`) or **fail** (≥1 criterion
unmet → back to the implementer with the findings).

## The bounded loop + escalation

`verify_attempts` guards against an infinite implement↔verify cycle:

- On each verification round, increment the task's `verify_attempts`.
- **Fail** → hand the findings back to `implement-feature` for another round (the orchestrator drives
  this), unless the cap is reached.
- When `verify_attempts` reaches **`max_verify_iterations`** (default 4, from `.build-config.md`) with
  a **critical** acceptance criterion still failing, **stop looping**: set the task `status:
  needs_human`, append a `## Log` summary of what's still failing and what was tried, and surface it on
  the board. This is one of the two things that always stops regardless of mode (`build-config.md`).
  Do not keep burning rounds — escalate.

A non-critical/cosmetic miss with all critical criteria met may pass with the issue noted in the log,
at the verifier's judgement — but never weaken a criterion to make it pass.

## What verification is NOT

- Not the implementer self-approving — it is a separate agent.
- Not a rewrite of the test setup per project — it reads the generic contract + the task's criteria.
- Not "the tests are green" alone — green tests that don't cover a criterion don't prove it; the
  criterion's observable outcome is what counts.
