# Schema: `jira_ticket/v1` (external input contract)

The squad's ONLY Jira input shape. The orchestrator is the only Jira reader; it
wraps the ticket as `pm_request/v1` and `pm` triages it. This contract defines
what must be present for the squad to proceed.

```json
{
  "schema": "jira_ticket/v1",
  "id": "JIRA-1234",
  "issue_type": "Bug | Story | Task | Spike",
  "title": "login returns 500 when email contains a plus sign",
  "description": "...",
  "repro_steps": ["open login", "enter a+b@example.com", "submit"],
  "expected": "200 OK with token",
  "actual": "500 internal error",
  "severity": "blocker | critical | major | minor | trivial",
  "priority": "highest | high | medium | low | lowest",
  "labels": ["frontend"],
  "components": ["auth"],
  "affected_version": "1.4.2",
  "environment": "prod | staging | dev",
  "acceptance_criteria": ["Given ... When ... Then ... (optional)"],
  "attachments": ["screenshot.png", "stacktrace.log"],
  "comments": [ { "author": "user id", "body": "..." } ],
  "links": [ { "type": "relates to", "ticket": "JIRA-1200" } ],
  "reporter": "user id",
  "status": "Jira workflow state (read-only to the squad)"
}
```

## Proceed rules (halt rather than guess)
- **Required for ALL tickets:** `id`, `title`, `description`.
- **Required for Bug work:** `repro_steps`, `expected`, `actual`, and `severity`
  (or `priority`, mapped below).
- Everything else is optional.

If a required field is missing, the orchestrator:
1. emits an escalation record with `reason: insufficient_input` and
   `missing_fields: [...]`,
2. posts a `jira_update/v1` comment asking the reporter to complete the fields,
3. dispatches **no** executor.

No agent fills gaps by guessing (AGENTS.md: "do not invent requirements").

## Field mapping
| Jira value | Squad value |
|---|---|
| `blocker`, `critical` | severity `critical` |
| `major`, `high` | severity `major` |
| `minor`, `medium`, `low` | severity `minor` |
| `trivial`, `lowest` | severity `trivial` (treated as minor; tier LOW only if a one-line fix) |
| `issue_type: Bug` | work_type `bug_fix` (pm still triages and may reclassify) |
| `issue_type: Story / Task` | work_type `feature` (pm still triages) |

Severity maps to the default risk tier per `references/2o3-protocol.md`.
`acceptance_criteria` (if the reporter wrote any) is treated as input to pm's
Gherkin authoring, not as authoritative scope.
