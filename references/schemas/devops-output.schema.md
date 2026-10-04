# Schema: `devops_output/v1`

Emitted by `devops` for infra artifacts (build) and deployment results (deploy).
Single canonical copy — reference this file; never duplicate the body.

```json
{
  "schema": "devops_output/v1",
  "task_id": "uuid",
  "lane": "build | deploy",
  "status": "BUILD_OK | DEPLOY_OK | ROLLED_BACK",
  "files": [ { "path": "Dockerfile", "content": "..." } ],
  "gate_record": "human_gate/v1 id (deploy lane; REQUIRED)",
  "slo_metrics": {
    "error_rate_5xx": 0.002,
    "p99_ms": 410,
    "restarts": 0,
    "health_status": 200,
    "window": "5m"
  },
  "action_taken": "none | rollback",
  "verification_ref": "verification_report/v1 id"
}
```

## Field rules
- **deploy** lane requires `gate_record` referencing a `human_gate/v1` with
  `decision: APPROVED` — the orchestrator blocks dispatch otherwise.
- `status: DEPLOY_OK` requires `slo_metrics` from an actual `devops:watch` window
  (never asserted) and `action_taken: none`.
- `status: ROLLED_BACK` requires `action_taken: rollback` AND a
  `agent_failure_payload/v1` (`source_stage=devops`, `category=infra`,
  `target_agent=senior-dev`) with the SLO evidence.
- Exact readings only — "looks healthy" is not a value.
