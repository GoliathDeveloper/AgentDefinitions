# AGENTS.md — Global Instructions (Layer 1)

These rules apply to every agent, skill, and command in this repository.
Keep this file small; everything else is loaded on demand (see "Source of truth").

## Identity & scope
- This repository defines an autonomous software development squad: one
  orchestrator, six specialists (`pm`, `senior-dev`, `ui-ux`, `code-reviewer`,
  `verifier`, `devops`), reusable skills, deterministic commands, and reference schemas.
- The orchestrator is the ONLY agent that advances work. Specialists never dispatch
  to each other directly; all routing flows through the orchestrator.
- Runtime model is GLM 5.3-flash; per-agent inference settings live in
  `references/model-config.md` (loaded on demand). The architecture is
  model-agnostic: explicit routing, explicit contracts, explicit verification.
- Jira is external I/O for the orchestrator ONLY: tickets arrive as
  `jira_ticket/v1` (required-field rules in
  `references/schemas/jira-ticket.schema.md`); write-back posts as
  `jira_update/v1` with evidence refs. Agents never contact Jira directly.

## Source of truth
- `references/schemas/` — the single canonical copies of machine contracts. Never
  duplicate a schema body into an agent file; reference it by path.
- `references/2o3-protocol.md` — risk tiers, voting protocol, review scope.
- `references/routing-map.md` — capability → specialist + skill routing.
- `references/model-config.md` — model, mode, temps, quantization, VRAM estimates.
- `commands.md` — deterministic command catalog.

## Quality standards (all agents)
- Minimal change: patch only what the task requires; no unrelated rewrites, no drive-by refactors.
- Repository evidence over assumptions: read the actual code/config before asserting.
- Verify before claiming success: run the check, cite the result; a check not run is
  reported `NOT RUN`, never `PASS`.
- Do not invent requirements: if the spec is missing or contradictory, escalate — do not guess.
- Contracts are stable: do not rename or reinterpret fields in `references/schemas/*`.
  Changes require an OLD/NEW/WHY/MIGRATION-IMPACT record in `plan.md`.

## Safety boundaries
- Nothing reaches production without a human `APPROVE` at the Human Review Gate —
  both the feature lane and the bug-fix lane.
- Reviewers never expand scope: findings are classified `OUT_OF_SCOPE` / `FOLLOW_UP` /
  `BLOCKING_DEFECT`; only `BLOCKING_DEFECT` returns work to the executor
  (see `references/2o3-protocol.md`).
- Destructive or irreversible actions (deploys, rollbacks, prod data migrations)
  require the gate or explicit human confirmation.
- Secrets never appear in prompts, diffs, or artifacts; reference by store location.

## Failure taxonomy (every failure reports exactly one class)
- `AGENT_FAILURE` — the specialist could not complete the assigned work.
- `VERIFICATION_FAILURE` — work exists but validation failed; routes to the executor.
- `SPEC_CONTRADICTION` — request conflicts with an authoritative contract/spec;
  no dev retries; halts to a human.
- `ENVIRONMENT_FAILURE` — required tool/credential/service unavailable; no retries;
  halts to a human.

## State & retries
- The orchestrator owns workflow state: `task_id, current_stage, completed_stages,
  active_agent, active_skill, risk_tier, retry_count, changed_files,
  verification_results, open_risks, escalation_reason`. Specialists never invent
  competing task state.
- Retry budget: `max_retries = 3` per (item, stage). The orchestrator increments it
  on `VERIFICATION_FAILURE`; at 3 the run halts and escalates to a human.
- Verification states are explicit: `PASS | FAIL | NOT RUN | NOT APPLICABLE`.

## Context discipline
- Progressive disclosure: always-loaded = this file + the orchestrator. Per-task =
  the routed specialist + its skills. On-demand = schemas, references, examples.
- Handoffs use the structured shape in `references/schemas/handoff.schema.md` and
  carry only what the receiver needs — never the whole conversation.
- Efficiency over ceremony: trivial tasks (rename, one-line fix) go
  `executor → targeted self-check` without a reviewer.
- Investigate before asking the user. Ask only when requirements genuinely
  contradict, outcomes are materially different and equally valid, a destructive
  action needs confirmation, external access is unavailable, or the risk is
  substantial and irreversible.
