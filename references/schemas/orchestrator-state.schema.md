# Schema: `orchestrator_state/v1` (state event, escalation, run report)

The orchestrator's three machine outputs. Single canonical copy.

## 1. State event (one per transition)
```json
{
  "schema": "orchestrator_state/v1",
  "task_id": "uuid",
  "work_type": "feature | bug_fix",
  "transition": "pm -> senior-dev,ui-ux",
  "from_stage": "pm",
  "to_stage": ["senior-dev", "ui-ux"],
  "active_agent": "senior-dev",
  "active_skill": "targeted-fix",
  "risk_tier": "LOW | MEDIUM | HIGH | CRITICAL",
  "retry_count": { "review": 0, "qa": 0 },
  "action": "dispatch | await | escalate | halt | approve_deploy",
  "payload_ref": "id/path of the payload being routed",
  "timestamp": "iso8601"
}
```

## 2. Escalation record (terminal)
```json
{
  "schema": "orchestrator_state/v1",
  "task_id": "uuid",
  "work_type": "feature | bug_fix",
  "transition": "qa -> human_escalation",
  "action": "halt",
  "escalation": {
    "reason": "retry_budget_exhausted | spec_contradiction | environment_failure | deploy_failure | 2o3_disagreement | insufficient_input",
    "stage": "qa",
    "retry_count": 3,
    "missing_fields": ["repro_steps", "expected (insufficient_input only)"],
    "summary_for_human": "plain-language summary of what happened and why the loop stopped",
    "evidence_refs": ["failure payload / tally / SLO artifact refs"]
  }
}
```

## 3. Final run report (returned to the human)
```json
{
  "schema": "orchestrator_state/v1",
  "task_id": "uuid",
  "work_type": "feature | bug_fix",
  "risk_tier": "MEDIUM",
  "outcome": "COMPLETED | ESCALATED | ABORTED",
  "stages_completed": ["intake", "triage", "build", "review", "verify", "gate", "deploy"],
  "artifacts": {
    "spec": "prd.json / defect_brief ref",
    "patch_ref": "diff or PR id",
    "review": "failure/success payload id",
    "verification": "verification_report/v1 id",
    "tally": "tally_2o3/v1 id (HIGH/CRITICAL)",
    "gate": "human_gate/v1 id",
    "deploy": "devops_output/v1 id"
  },
  "verification_summary": "PASS (with per-check statuses)",
  "open_risks": [ { "risk": "...", "accepted_by": "human id | null" } ],
  "jira_refs": ["jira_update/v1 ids posted for this run"]
}
```

## Rules
- `missing_fields` is populated only for `reason: insufficient_input`
  (`references/schemas/jira-ticket.schema.md`).
- The run report is the human's table of contents for the whole run; every
  artifact ref must resolve.
