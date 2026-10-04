---
name: test-generation
description: Use when converting PM Gherkin acceptance criteria into executable unit/integration/E2E test suites (Jest, PyTest, Playwright) — including edge cases and defect-brief regression criteria.
---

# Test Generation

## Goal
Produce an executable test suite that maps 1:1 to the acceptance criteria, plus
regression tests for the bug-fix lane.

## Preconditions
- Gherkin acceptance criteria (`prd.json`) or a `defect_brief/v1`
  `regression_criteria`.
- The merged diff under test.

## Procedure
1. For each Given/When/Then: one test — unit where possible; integration for
   endpoint behavior; E2E (Playwright) only for user journeys.
2. Add edge cases the criteria imply: empty/invalid inputs, timeouts, 4xx/5xx
   paths, boundary values.
3. Bug lane: write the regression test from the brief's `regression_criteria`
   BEFORE verifying the fix (it should fail on the pre-fix diff).
4. Determinism: fixed seeds for randomized/fuzz inputs; no wall-clock or real
   network dependencies (mock clocks, stubbed network).
5. `qa:run-suite` in isolation; record per-test results.

## Guardrails
- No test asserts behavior that is not in the criteria (scope discipline applies
  to tests too).
- No `xfail`/`skip` without a recorded reason + `open_risks` entry.
- Tests never modify application code.

## Verification
- Criteria coverage: every criterion maps to at least one test
  (`qa:coverage` cross-check).
- The suite is green on a known-good reference, or failures are correctly
  attributed to the diff.

## Output
- Test files + suite result JSON; feeds the verifier's success/failure payload.
