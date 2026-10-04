# Routing Map (capability-based)

The orchestrator routes by **requested capability**, not by agent name — so the
system stays portable if agents are renamed or reorganized.
Format: capability → specialist(s) → skill(s) → default tier.

| Requested capability | Specialist | Skill | Default tier |
|---|---|---|---|
| Triage / requirements / scoping | `pm` | (intrinsic; `pm:validate-contract`, `pm:validate-gherkin`) | — |
| Author interface contract | `pm` | (intrinsic spec work) | — |
| Implement backend / API / business logic | `senior-dev` | `backend-implementation` | HIGH |
| Design / apply database schema + migration | `senior-dev` | `database-migration` | HIGH (prod: CRITICAL) |
| Implement frontend / components | `ui-ux` | `frontend-component` | HIGH |
| Root-cause a failing test / defect | owning executor | `debug-investigate` | per defect |
| Fix cited failure / defect (minimal patch) | owning executor | `targeted-fix` | per defect |
| Security assessment of a diff | `code-reviewer` | `security-review` | CRITICAL |
| Contract conformance check | `code-reviewer` | `contract-conformance` | MEDIUM |
| Architecture decision | `pm` + `senior-dev` (consult) | — | CRITICAL (human decides) |
| Generate tests from Gherkin | `verifier` | `test-generation` | MEDIUM |
| Run / judge test evidence | `verifier` | `self-check` | per task |
| SLO / release validation | `devops` | `release-validation` | HIGH |

## Lanes
- **Feature:** `pm` (PRD + contract) → (`senior-dev` ∥ `ui-ux`) → merge →
  `code-reviewer` → `verifier` (per tier) → self-check → 2o3 tally →
  Human Review Gate → `devops` → deploy.
- **Bug fix:** `pm` triage → defect brief → owning executor
  (`debug-investigate` → `targeted-fix`) → merge → `code-reviewer` →
  `verifier` (per tier) → self-check → 2o3 tally → Human Review Gate →
  `devops` → deploy.
- **Trivial** (rename, typo, one-line fix): owning executor direct + targeted
  self-check; no reviewer; tier LOW.

## Notes
- "Owning executor" = `target_agent` in the defect brief.
- A capability may involve multiple specialists; the orchestrator sequences them
  with structured handoffs.
- No dedicated `architect` agent is instantiated: CRITICAL architecture decisions
  route to `pm` (spec level) or `senior-dev` (system design) and escalate to the
  human for the final call. Do not invent agents for gaps.
