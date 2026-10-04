# 2o3 Adaptive Voting & Review Protocol

2o3 (two-out-of-three) is a **conditional protocol, not three permanent agents**.
The orchestrator wakes only the voters the risk tier requires — every extra agent
adds context/token overhead.

## Voters and their orthogonal questions
| Voter | Squad role | Question it answers |
|---|---|---|
| Executor | `senior-dev` or `ui-ux` (the owning agent) | Did we implement the requested behavior correctly? |
| Reviewer | `code-reviewer` | Does the implementation satisfy the approved requirements and avoid regressions — in scope? |
| Verifier | `verifier` | Does the **evidence** demonstrate that it works? |

They vote on **different evidence**, not taste. A second reasoning agent can easily
reproduce the first's mistake; the verifier has a different mandate:

> "Don't decide whether the implementation sounds correct. Determine whether the
> evidence demonstrates that it is correct."

## Risk tiers
The orchestrator assigns the tier at intake from: defect severity, feature scope,
production impact, security surface.

- **LOW** — `executor → self-check`. No reviewer.
- **MEDIUM** — `executor → code-reviewer → self-check`.
- **HIGH** — `executor → code-reviewer → verifier`; disagreement → arbiter pass on
  recorded evidence, then human-gate notice.
- **CRITICAL** — HIGH + mandatory human escalation after the gate. Security and
  data-integrity work defaults to CRITICAL.

Defaults: new feature = HIGH. Bug fix: severity `critical`/`major` = HIGH,
`minor` = MEDIUM. Isolated style/doc change = LOW.

## The 2o3 decision
- A stage passes when **≥ 2 of the tier's assigned voters agree**.
- Disagreements must be recorded **with evidence** (failing assertions, diffs, SLO
  data) — not opinions.
- A reviewer's disagreement is only a valid veto as a `BLOCKING_DEFECT` (below).

## Review scope (anti-scope-creep — binding on code-reviewer)
Review against:
1. approved requirements
2. approved plan / contract
3. changed files
4. relevant existing contracts
5. demonstrable regression / security risk

**Do NOT expand scope because you identify an unrelated improvement.** Classify
every finding:
- `OUT_OF_SCOPE` — noted, no action, never counted against the executor.
- `FOLLOW_UP` — separate future task; not a blocker.
- `BLOCKING_DEFECT` — the **only** class that returns work to the executor.

This prevents the documented failure where reviewer findings become unbounded
blocking work (out-of-scope redesign requests looping back through the executor).

## Where votes happen
- LOW: self-check evidence only (run commands).
- MEDIUM: executor claim + reviewer judgment.
- HIGH/CRITICAL: executor claim + reviewer judgment + verifier evidence; the
  orchestrator tallies.
- The verifier runs in an **isolated context**: it shares only the diff,
  requirements, contract, and executor evidence — never the executor's
  reasoning trace.

- The tally is recorded as `tally_2o3/v1`
  (`references/schemas/2o3-tally.schema.md`) — who voted, on what evidence, and
  why the stage passed or failed. Every `DISAGREE` cites an evidence artifact;
  opinion-only votes are discarded.

## Interaction with retries
- A 2o3 rejection counts as one `VERIFICATION_FAILURE` against the owning stage.
- The budget still applies: 3 per (item, stage). Persistent deadlock → escalate to
  a human with the recorded disagreement (do not loop indefinitely).
