---
name: backend-implementation
description: Use when building backend functionality from an approved Interface Contract — API handlers, business logic, and data access layers, including all contract error states.
---

# Backend Implementation

## Goal
Implement the backend of a feature strictly to the approved Interface Contract.

## Preconditions
- `prd.json` with `interface_contract` (TypeScript + OpenAPI v3), including 4xx/5xx
  error states.
- Data-model requirements from the PRD.

## Procedure
1. Read the contract; enumerate endpoints, types, status codes, error shapes.
2. Design schema + migrations first (`database-migration`).
3. Implement handlers: modular, strictly typed, documented; one handler per
   endpoint; shared validation mirroring the OpenAPI schemas.
4. Implement every contract error state (400/401/404/409/429/500 as specified).
5. `dev:generate-migration` + `dev:validate-schema` for the schema.
6. `dev:run-lint` + `dev:run-typecheck` + `dev:run-tests` (unit level).
7. Hand to `self-check`.

## Guardrails
- No endpoint, field, or status code that is not in the contract; none missing.
- No secrets in code — reference the secret store.
- Parameterized queries only; never string-concatenated SQL.
- Ambiguous/contradictory contract → stop → `spec_contradiction`; do not guess.

## Verification
- Every contract endpoint implemented with exact paths/types/statuses.
- Lint + typecheck + unit tests green.
- Conformance spot-check via `reviewer:contract-check` is clean.

## Output
- `senior_dev_output/v1` (`references/schemas/senior-dev-output.schema.md`):
  `files`, `migrations`, `contract_conformance` (`endpoints_implemented`,
  `deviations: []`).
