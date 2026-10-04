# Schema: `verification_report/v1`

Emitted by any agent running `self-check` (or a deterministic verification
command). Explicit states are mandatory: a check that was not run is `NOT RUN`,
never `PASS`.

```json
{
  "schema": "verification_report/v1",
  "task_id": "uuid",
  "stage": "self_check | review | verify | release",
  "agent": "agent that produced this report",
  "checks": [
    {
      "check": "dev:run-lint",
      "status": "PASS | FAIL | NOT RUN | NOT APPLICABLE",
      "evidence": "exit 0, 0 errors",
      "duration_ms": 320
    },
    {
      "check": "dev:run-typecheck",
      "status": "PASS",
      "evidence": "exit 0",
      "duration_ms": 1400
    }
  ],
  "summary": "PASS | FAIL",
  "open_risks": [ { "risk": "...", "accepted_by": "orchestrator (null if not yet)" } ]
}
```

Rules:
- `evidence` must be a real observation (exit code, output snippet, SLO value) —
  not an assertion.
- `NOT APPLICABLE` requires a reason in `evidence`.
- `NOT RUN` is acceptable only when the orchestrator records the accepted risk in
  `open_risks`; otherwise it fails a HIGH/CRITICAL gate.
- `summary` is `PASS` only when every check is `PASS` or `NOT APPLICABLE` and no
  unaccepted `open_risks` remain.
