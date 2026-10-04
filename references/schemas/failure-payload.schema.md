# Schema: `agent_failure_payload/v1` (failure payload)

Canonical copy. All producers emit this exact shape; all consumers accept it.
Do not edit fields here without an OLD/NEW/WHY/MIGRATION-IMPACT record in `plan.md`.

## Producer → orchestrator
```json
{
  "schema": "agent_failure_payload/v1",
  "task_id": "uuid of the current squad run",
  "source_stage": "code_review | qa | devops | human_review",
  "failure_class": "AGENT_FAILURE | VERIFICATION_FAILURE | SPEC_CONTRADICTION | ENVIRONMENT_FAILURE",
  "status": "FAILED",
  "feature": "feature name or ticket id",
  "retry_count": 1,
  "max_retries": 3,
  "retry_exhausted": false,
  "escalate_to_human": false,
  "spec_contradiction": false,
  "contradiction_detail": null,
  "failed_items": [
    {
      "item_id": "unique id for this failure",
      "target_agent": "senior-dev | ui-ux",
      "file": "src/auth/login.ts",
      "line": 42,
      "end_line": 58,
      "category": "security | lint | type | contract | logic | visual | e2e | style | infra",
      "severity": "critical | major | minor",
      "classification": "BLOCKING_DEFECT | FOLLOW_UP | OUT_OF_SCOPE (review lane only)",
      "title": "short human-readable summary",
      "description": "what is wrong and why",
      "failing_assertion": "expected X, got Y (or null)",
      "stack_trace": "raw trace, line numbers preserved (or null)",
      "suggested_fix": "concrete patch guidance (or null)",
      "contract_ref": "InterfaceContract#/api/v1/auth/login (or null)",
      "code_context": "the offending source snippet, verbatim"
    }
  ]
}
```

## Field rules
- `retry_count` is owned by the **orchestrator**; producers report the count they
  were told. Producers never invent their own counter.
- `escalate_to_human` is `true` only when `retry_count >= max_retries`, or
  `failure_class` is `SPEC_CONTRADICTION` / `ENVIRONMENT_FAILURE`.
- Every `failed_item` carries **line-level** context (`line`, `code_context`,
  `stack_trace`) so the executor patches specific lines instead of rewriting files.
- `category` drives **routing** (see Routing Matrix). `severity` drives **policy**:
  `critical`/`major` block; `minor` is advisory unless it accumulates past
  threshold (default 10).
- Review-lane items MUST carry `classification` (2o3 review-scope rule);
  only `BLOCKING_DEFECT` may fail a stage.

## Routing matrix (category → owning executor)
| `category` | `target_agent` |
|---|---|
| `security`, `lint`, `type`, `contract`, `logic`, `infra` | `senior-dev` |
| `visual`, `e2e`, `style` | `ui-ux` |

A merged PR contains both frontend and backend code. One failure maps to exactly
one `target_agent` via `category`; a defect spanning both emits **two** items
sharing an `item_id` prefix.

## Success payload
```json
{
  "schema": "agent_failure_payload/v1",
  "task_id": "uuid",
  "source_stage": "code_review | qa | devops | human_review",
  "status": "PASSED",
  "feature": "feature name or ticket id",
  "artifacts": ["path or type of produced artifact"],
  "metrics": { "checks_run": 128, "checks_passed": 128, "duration_ms": 4200 }
}
```

## Changelog
- `v1`: additive change vs. the Updated/ original — added `failure_class`,
  `classification`, `task_id` (renamed from `run_id` at the envelope level;
  see plan.md §Contracts). Item fields unchanged.
