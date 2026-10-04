---
name: database-migration
description: Use when designing or changing database schema — new tables, index changes, data-model revisions — producing migrations that are reviewable, reversible, and applied only to approved environments.
---

# Database Migration

## Goal
Design schema changes and produce safe, reviewable migrations that match the
feature's data requirements without lock surprises or data loss.

## Preconditions
- Data requirements from the PRD / contract.
- The current schema (read the existing migrations, not assumptions).

## Procedure
1. Read the existing schema + recent migrations; identify the minimal delta.
2. Design the change: tables, columns, constraints, indexes; note types and
   nullability against the Interface Contract types.
3. `dev:generate-migration` for the delta; write a matching down-migration where
   the schema allows.
4. Flag risk explicitly: table rebuilds, backfills on large tables, lock
   duration, production impact.
5. `dev:validate-schema` + `dev:run-typecheck` on the data access layer.
6. Hand to `self-check`.

## Guardrails
- No destructive DDL (drop/rename in prod) without explicit human approval at the
  gate.
- No migration that also "cleans up" unrelated schema (scope discipline).
- Backfills must be idempotent and re-runnable.
- Schema that contradicts the Interface Contract → `spec_contradiction`, stop.

## Verification
- Migration validates; up + down migrations exist where possible.
- Type mapping to the contract is explicit.
- Risk notes (locks, backfill size) are recorded for the reviewer.

## Output
- Migration files + schema notes + risk list — the `migrations` block of
  `senior_dev_output/v1` (`references/schemas/senior-dev-output.schema.md`).
