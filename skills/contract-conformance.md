---
name: contract-conformance
description: Use when verifying that implemented code matches the approved Interface Contract exactly — endpoints, types, status codes, error shapes — before the review verdict is issued.
---

# Contract Conformance

## Goal
Prove (or disprove) that the merged diff implements the Interface Contract with
zero drift.

## Preconditions
- The approved Interface Contract (TypeScript + OpenAPI v3) and the merged diff.

## Procedure
1. Enumerate contract endpoints, request/response schemas, required status codes.
2. For each: locate the implementation and compare path, method, field
   names/types, required vs optional, status codes, error shapes.
3. Flag: missing endpoint, extra endpoint not in the contract, renamed field,
   drifted type, missing error state.
4. Frontend lane: verify client fetch types against the TS contract
   (`ui:check-types`).
5. Produce the conformance table + `deviations[]` with `file:line` per deviation.

## Guardrails
- An extra endpoint not in the contract is a `BLOCKING_DEFECT` (scope drift) —
  never silently accepted.
- The contract being internally inconsistent is `spec_contradiction`, not a
  conformance failure.

## Verification
- 100% of contract endpoints checked; each has an explicit status.
- The deviations list is complete with `file:line`.

## Output
- `reviewer:contract-check` result: conformance table + `deviations[]`, appended
  to the review payload.
