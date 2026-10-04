# Schema: `tally_2o3/v1` (voting record)

The audit record of a 2o3 decision (protocol in `references/2o3-protocol.md`).
This is what lets a human — or the next reviewer — see WHO voted, on WHAT
evidence, and WHY the stage passed or failed. Single canonical copy.

```json
{
  "schema": "tally_2o3/v1",
  "task_id": "uuid",
  "stage": "review | verify | release",
  "risk_tier": "MEDIUM | HIGH | CRITICAL",
  "votes": [
    { "voter": "executor (senior-dev | ui-ux)", "verdict": "AGREE | DISAGREE", "evidence_ref": "senior_dev_output / ui_ux_output id" },
    { "voter": "code-reviewer", "verdict": "AGREE | DISAGREE", "evidence_ref": "failure/success payload id" },
    { "voter": "verifier", "verdict": "AGREE | DISAGREE", "evidence_ref": "verification_report/v1 id (HIGH/CRITICAL only)" }
  ],
  "outcome": "PASS | FAIL | ESCALATE",
  "arbiter": {
    "invoked": false,
    "resolved": null,
    "basis": "recorded evidence used for the arbiter pass (null if not invoked)"
  },
  "recorded_at": "iso8601"
}
```

## Rules
- `votes[]` contains ONLY the voters assigned by the risk tier (MEDIUM: executor +
  reviewer; HIGH/CRITICAL: + verifier). Unused voter slots are omitted, not
  guessed.
- Outcome rule: `PASS` when ≥2 assigned voters AGREE; `FAIL` when a
  `BLOCKING_DEFECT`/evidence-FAIL stands; `ESCALATE` when an arbiter pass cannot
  resolve a disagreement.
- Every `DISAGREE` requires an `evidence_ref` — opinions without artifacts are
  invalid votes and are discarded (re-request with evidence).
- At LOW tier no tally is produced (self-check evidence is the record).
- The tally id is carried in the run report and the human gate's
  `artifacts_presented` (HIGH/CRITICAL).
