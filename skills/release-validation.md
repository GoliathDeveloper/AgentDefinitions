---
name: release-validation
description: Use post-deployment to validate production health against SLOs and decide/execute rollback on breach — devops' gated release step after a human-approved deploy.
---

# Release Validation

## Goal
Confirm a deployment is healthy against SLOs — or roll back to the last good
release with recorded evidence.

## Preconditions
- Deployment completed (`devops:deploy` returned).
- Last-good release recorded.

## Procedure
1. `devops:watch` over the SLO windows: 5xx rate (>1% / 5 min), p99 latency
   (>800 ms / 5 min), restarts (>3 / 2 min), health endpoint (non-200 × 3).
2. Sample logs for exceptions outside the known error budget.
3. All SLOs in window → record readings; report deploy OK.
4. Any breach → `devops:rollback` to the last good release; confirm the
   last-good version is serving.
5. Report exact readings — never "looks healthy".

## Guardrails
- No rollback without recorded SLO evidence.
- Never extend the watch window to "wait and see" — the window is the budget.
- A failed rollback is an immediate escalation, not a retry.
- No unapproved manual changes to the running system during validation.

## Verification
- Report includes every SLO value, threshold, and window.
- Rollback confirms the last-good version + post-revert health.

## Output
- `devops_output/v1` (`references/schemas/devops-output.schema.md`): deploy
  result with `slo_metrics`, `action_taken` (`none | rollback`), `verification_ref`.
- On breach: `agent_failure_payload/v1` (`category=infra`,
  `target_agent=senior-dev`).
