# Commands (deterministic operations)

Commands do one deterministic thing and return machine-readable results. Agents
reference commands; they never re-describe them.
Conventions: exit 0 = success; structured output (JSON or report file); idempotent
where possible. Commands never call a model — **commands compute, skills judge**.

## dev
- `dev:run-lint <paths?>` — lint changed files (or all); violations with `file:line`.
- `dev:run-typecheck <paths?>` — strict typecheck; errors with `file:line`.
- `dev:run-tests <filter?>` — targeted test run; per-test results.
- `dev:git-diff <base?>` — unified diff of current work.
- `dev:generate-migration <name>` — scaffold a migration from a schema description.
- `dev:db-migrate <dir>` — apply migrations to target DB. **DESTRUCTIVE:** dev/
  staging only without the gate.
- `dev:validate-schema <file>` — validate OpenAPI v3 / TS contract files.

## ui
- `ui:check-types <paths>` — verify client types match the contract
  (`types_match` + deviations).
- `ui:check-a11y <paths>` — accessibility audit (`wcag_check` + issues).

## reviewer
- `reviewer:static-analyze <diff>` — linters + SAST over the diff.
- `reviewer:owasp-scan <diff>` — OWASP Top 10 checks.
- `reviewer:arch-check <diff>` — architectural guidelines + strict typing.
- `reviewer:contract-check <diff> <contract>` — conformance table + deviations.

## qa
- `qa:run-suite <manifest>` — execute the test suite in isolation; per-test results.
- `qa:capture-trace <suite-result>` — extract assertion diffs + stack traces.
- `qa:coverage <suite-result> <criteria>` — coverage of acceptance criteria.

## devops
- `devops:build-container <feature>` — containerization files (multi-stage, non-root).
- `devops:write-pipeline <repo>` — CI/CD + IaC.
- `devops:deploy <task_id> --requires human_approval` — gated deployment.
- `devops:watch <task_id>` — post-deploy SLO monitor.
- `devops:rollback <task_id> <release>` — revert to last good.

## pm
- `pm:validate-contract <file>` — OpenAPI/TS syntax + internal consistency.
- `pm:validate-gherkin <file>` — parse Given/When/Then.

## orch
- `orch:status <task_id>` — state dump (stage, retries, active agent, risk_tier).
- `orch:step <task_id>` — advance one transition (supervised runs).
- `orch:approve <task_id>` — record a Human Review Gate approval.
- `orch:abort <task_id> <reason>` — force-halt a run.

## Safety
- Destructive commands (`dev:db-migrate` in prod, `devops:deploy`,
  `devops:rollback`) require the orchestrator's gate check; all others may run
  freely.
