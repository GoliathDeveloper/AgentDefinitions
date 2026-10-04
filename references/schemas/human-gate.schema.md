# Schema: `human_gate/v1` (production sign-off record)

The audit record for the Human Review Gate. Nothing deploys without one of these
with `decision: APPROVED`. Single canonical copy.

```json
{
  "schema": "human_gate/v1",
  "task_id": "uuid",
  "work_type": "feature | bug_fix",
  "requested_by": "orchestrator",
  "artifacts_presented": [
    { "type": "prd | defect_brief | patch_ref | review_payload | verification_report | tally_2o3", "ref": "artifact id/path" }
  ],
  "decision": "APPROVED | REJECTED",
  "decided_by": "human user id",
  "timestamp": "iso8601",
  "conditions": ["deploy to staging only", "null unless set"],
  "notes": "reviewer comments",
  "rejection_target": "pm | senior-dev | ui-ux (REJECTED only)",
  "rejection_reason": "spec gap | code defect (REJECTED only)"
}
```

## Rules
- `artifacts_presented` must include the review payload and the verification
  report (both verdicts on record) for HIGH/CRITICAL runs.
- On `REJECTED`, the orchestrator routes to `rejection_target`
  (`pm` for spec gaps, executor for code defects) and does NOT re-present until
  the targeted work completes.
- A deploy dispatched by `devops` must cite this record's id in
  `devops_output/v1.gate_record` — an untraceable approval is treated as absent.
