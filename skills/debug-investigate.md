---
name: debug-investigate
description: Use BEFORE modifying code when a defect (failure payload, defect brief, flaky test, SLO breach) needs a root cause — a read-only investigation that produces a localized diagnosis with a minimal patch boundary.
---

# Debug & Investigate

## Goal
Produce a localized root-cause diagnosis — files, lines, mechanism, evidence —
without modifying any code.

## Preconditions
- A defect source: `agent_failure_payload/v1`, `defect_brief/v1`, failing test
  output, or SLO/rollback data.

## Procedure
1. Read the defect source; extract repro steps, expected vs actual, cited
   file/lines, stack traces.
2. Read the implicated code plus its direct callers/dependents (targeted reads,
   not whole-repo reads).
3. Reproduce when possible: run the cited test (`dev:run-tests --filter`) or the
   failing endpoint with the repro input.
4. Trace the mechanism: which branch/line produces the actual behavior and why
   (data, timing, contract mismatch, environment).
5. Classify the root cause: implementation defect | contract/spec defect |
   test defect | environment defect.
6. Record the **minimal patch boundary**: exactly which files/lines must change.

## Guardrails
- Read-only: no edits during investigation.
- No scope creep: diagnose the cited defect, not "while I'm here" issues.
- Root cause in the spec/contract → stop and mark `spec_contradiction`; do not
  diagnose deeper.
- Not reproducible after honest attempts → report `NOT REPRODUCED` with what was
  tried (candidate `ENVIRONMENT_FAILURE`).

## Verification
- Diagnosis cites exact `file:line` for each mechanism step.
- The reproduction attempt is recorded (command + result).
- The classification matches one of the four classes.

## Output
- Handoff addendum: `root_cause`, mechanism, minimal patch boundary, reproduction
  evidence, classification. Feeds `targeted-fix`.
