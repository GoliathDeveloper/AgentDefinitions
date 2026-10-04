# AgentDefinitions — Orchestrated AI Software Squad

A four-layer, model-agnostic agent system for autonomous software development:
one orchestrator routes capability-based work to thin specialists, who execute
through reusable skills and deterministic commands, under a fixed set of global
rules — with machine-checked contracts between every hop.

## Read in this order

1. **`AGENTS.md`** — global rules that apply to everything (always-on context)
2. **`orchestrator/orchestrator.md`** — the conductor: routing, state, budgets, gates
3. **`references/routing-map.md`** — capability → specialist → skill routing
4. **`flowchart.md`** — the pipeline at a glance (tiers, 2o3, gates, lanes)
5. **`agents/` + `skills/` + `commands.md`** — the specialists and their tools
6. **`references/schemas/`** — load on demand, only the contract you need

## Structure

```text
AgentDefinitions/
├── AGENTS.md                  Layer 1 — global instructions (always loaded)
├── orchestrator/
│   └── orchestrator.md        Layer 2 — lead agent (routing / state / 2o3 / gates)
├── agents/                    Layer 3 — thin specialists (50–59 lines each)
│   ├── pm.md                  triage, PRD + Interface Contract, defect briefs
│   ├── senior-dev.md          backend executor (build / fix / bug_fix)
│   ├── ui-ux.md               frontend executor (build / fix / bug_fix)
│   ├── code-reviewer.md       reviewer voter — security + contract, scope-bounded
│   ├── verifier.md            verifier voter — evidence-only (HIGH/CRITICAL)
│   └── devops.md              infra, gated deploy, SLO + rollback
├── skills/                    Layer 4 — reusable procedures
│   ├── self-check.md          mandatory pre-report verification (PASS/FAIL/NOT RUN/N-A)
│   ├── debug-investigate.md   read-only root cause before any patch
│   ├── targeted-fix.md        minimal patch, never a rewrite
│   ├── backend-implementation.md
│   ├── database-migration.md
│   ├── frontend-component.md
│   ├── security-review.md
│   ├── contract-conformance.md
│   ├── test-generation.md     Gherkin → executable suite
│   └── release-validation.md  SLO watch + rollback
├── commands.md                Layer 4 — deterministic ops (commands compute, skills judge)
├── references/                on-demand layer
│   ├── 2o3-protocol.md        risk tiers, voting, review scope (anti-scope-creep)
│   ├── routing-map.md         capability-based routing
│   ├── model-config.md        GLM 5.3-flash settings per agent (est. VRAM)
│   └── schemas/               14 canonical machine contracts (single copies)
└── flowchart.md               pipeline: intake gate → lanes → review → 2o3 → gate → deploy
```

## The four layers

| Layer | What lives here | Loaded |
|---|---|---|
| 1 — Global rules | quality, safety, failure taxonomy, state model | always |
| 2 — Orchestrator | routing, risk tier, retries, 2o3, human gate | always |
| 3 — Specialists | one thin agent per domain; decisions only | per task |
| 4 — Skills & commands | procedures (judgment) + deterministic ops | per task |
| On-demand | schemas, protocol, routing, model config | when needed |

## How work flows

Two intake lanes share one validation tail — a bug fix is a PR:

- **Feature:** `human ask → pm (PRD + contract) → senior-dev ∥ ui-ux → merge → review → 2o3 → human gate → devops → LIVE`
- **Bug fix:** `Jira ticket → intake gate → pm (defect brief) → owning executor (investigate → targeted fix) → merge → review → 2o3 → human gate → devops → LIVE`
- **Trivial (LOW):** executor + targeted self-check, no reviewer.

## Safety guarantees

1. **Nothing deploys without a recorded human `APPROVE`** (`human_gate/v1`; devops cites it).
2. **No infinite loops** — `max_retries = 3` per (item, stage); the 3rd failure halts to a human.
3. **No guessing** — missing ticket fields block as `insufficient_input` (a Jira comment asks the reporter; nothing is dispatched).
4. **Reviewers stay in scope** — only `BLOCKING_DEFECT` returns work to an executor; `OUT_OF_SCOPE`/`FOLLOW_UP` never block.
5. **Spec defects don't burn retries** — `spec_contradiction` / `environment_failure` halt immediately.
6. **Honest verification** — a check not run is `NOT RUN`, never `PASS`.

## Contracts (who talks to whom)

| Direction | Contract |
|---|---|
| Jira → squad | `jira_ticket/v1` (+ proceed rules) |
| squad → Jira | `jira_update/v1` (evidence-backed; orchestrator-only writer) |
| orchestrator → agent | `handoff/v1` |
| spec work | `pm_output/v1` (features), `defect_brief/v1` (bugs) |
| executors | `senior_dev_output/v1`, `ui_ux_output/v1` |
| infra / deploy | `devops_output/v1` (cites the gate record) |
| verdicts | `agent_failure_payload/v1`, `verification_report/v1`, `tally_2o3/v1` |
| coordination | `orchestrator_state/v1`, `human_gate/v1` |

Full registry with producer/acceptor per contract: `plan.md` §4.1.

## Model

GLM 5.3-flash (per-agent mode/temperature/quantization in `references/model-config.md`).
The architecture is model-agnostic — swap the model, update that one file, re-validate.

## Open items

- VRAM figures are estimates — verify against the GLM 5.3-flash model card.
- The orchestrator assumes a harness that persists `task_id` state across stage calls.
