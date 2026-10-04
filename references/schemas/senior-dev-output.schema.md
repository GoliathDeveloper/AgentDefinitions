# Schema: `senior_dev_output/v1`

Emitted by `senior-dev` on every handoff completion (build / fix / bug_fix lanes).
Single canonical copy — reference this file; never duplicate the body.

```json
{
  "schema": "senior_dev_output/v1",
  "task_id": "uuid",
  "lane": "build | fix | bug_fix",
  "status": "BUILD_OK | FIX_OK",
  "ticket": "JIRA-1234 (bug_fix lane only; null otherwise)",
  "addressed_items": ["item_id (fix lane)"],
  "files": [ { "path": "src/auth/login.ts", "content": "..." } ],
  "migrations": [ { "path": "migrations/001_auth.sql", "content": "..." } ],
  "patch": "unified diff (fix/bug_fix lanes; null on build)",
  "root_cause": "mechanism + cited evidence (fix/bug_fix lanes; null on build)",
  "regression_note": "behavior self-check/verifier must cover (fix/bug_fix lanes)",
  "contract_conformance": { "endpoints_implemented": ["/api/v1/auth/login"], "deviations": [] }
}
```

## Field rules
- **build** lane: `files` required (`migrations` when schema changes); `patch` null.
- **fix / bug_fix** lanes: `patch` + `root_cause` + `regression_note` required;
  `files` optional (content behind the patch).
- `contract_conformance.deviations` MUST be `[]` — a non-empty deviation means the
  self-check fails (or `spec_contradiction` if the contract itself is wrong).
- Emitted to the orchestrator as the handoff payload; never posted to Jira
  directly (the orchestrator writes `jira_update/v1`).
