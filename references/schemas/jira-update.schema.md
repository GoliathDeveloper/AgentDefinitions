# Schema: `jira_update/v1` (external output contract)

The squad's ONLY Jira write-back shape. Posted by the **orchestrator only** —
agents never contact Jira. Every update carries evidence refs so a human (or the
next reviewer) can validate the work without re-deriving it.

```json
{
  "schema": "jira_update/v1",
  "task_id": "uuid",
  "ticket": "JIRA-1234",
  "event": "comment | status_transition | label | link",
  "posted_by": "orchestrator",
  "stage": "intake | triage | build | review | verify | human_gate | deploy | escalation",
  "body": "human-readable summary of what happened at this stage",
  "evidence": [
    { "type": "defect_brief | prd | patch_ref | failure_payload | verification_report | tally_2o3 | human_gate | run_report", "ref": "artifact id or path" }
  ],
  "status_to": "In Review (status_transition only)",
  "labels_add": ["agent-reviewed"],
  "links_add": [ { "type": "relates to", "ticket": "JIRA-1200" } ]
}
```

## Rules
- One update per stage transition — no spam; a failure posts one comment carrying
  the failure-payload ref.
- `evidence[]` is REQUIRED on every comment: a stage claim without artifacts is
  not postable.
- Never post secrets, credentials inside stack traces, or raw customer data.
- Suggested status mapping: intake→To Do · triage→In Triage · build→In Progress ·
  review/verify→In Review · human_gate→Pending Approval · deploy→Deploying ·
  completed→Done · escalation→Blocked.
- **Human validation path:** walk `evidence[]` top-to-bottom — brief/PRD → patch →
  review verdict → verification report → 2o3 tally → gate record → deploy result.
  Each artifact is independently checkable; nothing relies on "trust the agent".
