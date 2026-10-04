# Schema: `ui_ux_output/v1`

Emitted by `ui-ux` on every handoff completion (build / fix / bug_fix lanes).
Single canonical copy — reference this file; never duplicate the body.

```json
{
  "schema": "ui_ux_output/v1",
  "task_id": "uuid",
  "lane": "build | fix | bug_fix",
  "status": "BUILD_OK | FIX_OK",
  "ticket": "JIRA-1234 (bug_fix lane only; null otherwise)",
  "addressed_items": ["item_id (fix lane)"],
  "files": [ { "path": "src/components/LoginForm.tsx", "content": "..." } ],
  "patch": "unified diff (fix/bug_fix lanes; null on build)",
  "root_cause": "reproduction + mechanism (fix/bug_fix lanes; null on build)",
  "regression_note": "behavior self-check/verifier must cover (fix/bug_fix lanes)",
  "accessibility": { "wcag_check": "pass | fail", "issues": [] },
  "contract_conformance": { "types_match": true, "deviations": [] }
}
```

## Field rules
- **build** lane: `files` required; `patch` null.
- **fix / bug_fix** lanes: `patch` + `root_cause` + `regression_note` required.
- `accessibility.wcag_check` must be `pass` to report completion (`fail` fails
  self-check).
- `contract_conformance.types_match` must be `true`; `deviations` MUST be `[]` —
  a type mismatch is a defect, never a local "fix".
- Emitted to the orchestrator as the handoff payload; never posted to Jira
  directly (the orchestrator writes `jira_update/v1`).
