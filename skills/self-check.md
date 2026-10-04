---
name: self-check
description: Use at the end of any implementation or fix to verify the diff before claiming completion — the squad's mandatory pre-report verification step, producing a verification_report/v1.
---

# Self-Check

## Goal
Produce an honest verification report for the current change: what was run, what
passed, what was not run — with cited evidence per check.

## Preconditions
- A diff exists (`dev:git-diff`).
- The task's acceptance criteria or failure items are known.

## Procedure
1. `dev:git-diff` — list changed files.
2. Run targeted checks on changed files: `dev:run-lint`, `dev:run-typecheck`
   (backend) or `ui:check-types`, `ui:check-a11y` (frontend).
3. Run the targeted test suite: `dev:run-tests` / `qa:run-suite` for the changed
   area.
4. Run the broader suite when the change crosses module boundaries.
5. Verify contract conformance: `reviewer:contract-check` against the
   Interface Contract.
6. Verify regression coverage: confirm the fix's `regression_note` is covered by a
   test (`qa:coverage`).
7. Record every check as `PASS | FAIL | NOT RUN | NOT APPLICABLE` with evidence
   (exit code, output snippet).

## Guardrails
- Never report `PASS` for a check that was not actually executed.
- `NOT RUN` is acceptable with a reason; it fails a HIGH/CRITICAL gate unless the
  orchestrator records the accepted risk in `open_risks`.
- Do NOT fix inside self-check. Report; fixing is the executor's job (prevents
  self-approval loops).

## Verification
- The report is a valid `verification_report/v1` with all statuses explicit and
  evidence cited per check.

## Output
- `verification_report/v1` (`references/schemas/verification-report.schema.md`)
  with summary + open risks.
