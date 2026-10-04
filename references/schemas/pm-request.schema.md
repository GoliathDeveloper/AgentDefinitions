# Schema: `pm_request/v1` (intake)

Single entry-point shape for everything the squad receives: free-form human asks
and Jira tickets.

```json
{
  "schema": "pm_request/v1",
  "task_id": "uuid",
  "source": "human | jira",
  "raw_request": "the human's goal in words (null for a ticket)",
  "ticket": {
    "id": "JIRA-1234",
    "title": "...",
    "description": "...",
    "repro_steps": ["...", "..."],
    "expected": "...",
    "actual": "...",
    "severity": "critical | major | minor",
    "attachments": ["screenshot.png"]
  },
  "context": ["constraints", "prior feedback"]
}
```

Rules:
- `ticket` is null for free-form feature asks; `raw_request` is null for tickets.
- `task_id` is assigned by the orchestrator at intake and travels with every
  downstream payload.
