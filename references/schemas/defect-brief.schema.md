# Schema: `defect_brief/v1`

PM-authored brief for the bug-fix lane. Consumed by the owning executor and the
orchestrator (routing). Single canonical copy.

```json
{
  "schema": "defect_brief/v1",
  "task_id": "uuid",
  "work_type": "bug_fix",
  "ticket": "JIRA-1234",
  "title": "login returns 500 when email contains a plus sign",
  "repro_steps": ["...", "..."],
  "expected": "200 OK with token",
  "actual": "500 internal error",
  "severity": "critical | major | minor",
  "suspected_area": "backend | frontend | both",
  "target_agent": "senior-dev | ui-ux",
  "contract_ref": "/api/v1/auth/login (or null)",
  "regression_criteria": "Given ... When ... Then ... (what QA must verify after the fix)",
  "attachments": ["screenshot.png"],
  "spec_contradiction": false
}
```

## Field rules
- `target_agent` is **required** — it is the routing key; a brief without it halts
  at the orchestrator.
- `regression_criteria` is **required**: the verifier turns it into a regression
  test before the fix is accepted.
- `severity` drives the default risk tier (critical/major → HIGH, minor → MEDIUM;
  see `references/2o3-protocol.md`).
- No full PRD is produced for a bug fix. If analysis shows the "defect" is actually
  missing/broken spec, set `spec_contradiction: true` and stop.
