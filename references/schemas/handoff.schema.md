# Schema: structured handoff

Every inter-agent handoff uses this compact envelope. Include only what the
receiving agent needs — never the entire conversation.

```json
{
  "schema": "handoff/v1",
  "task_id": "uuid",
  "from": "orchestrator | senior-dev | ui-ux | code-reviewer | verifier | devops | pm",
  "to": "specialist name or orchestrator",
  "task": "one-line statement of the assignment",
  "objective": "what 'done' looks like",
  "context": "minimum necessary background",
  "evidence": ["stack trace", "failing assertion", "SLO reading", "screenshot"],
  "constraints": ["do not change the contract", "bug-fix: minimal patch only", "risk_tier=HIGH"],
  "files": ["src/auth/login.ts", "migrations/001_auth.sql"],
  "acceptance_criteria": ["Given ... When ... Then ..."],
  "previous_work": "what was already attempted and its outcome (null for first pass)",
  "verification_status": "PENDING | PASS | FAIL | NOT RUN"
}
```

Rules:
- `evidence` items must be concrete artifacts, not summaries of summaries.
- `constraints` always include the risk tier and the review-scope rule for
  reviewer handoffs.
- The orchestrator fills `task_id` and appends `previous_work` on every retry, so
  the executor sees exactly what was already tried.
