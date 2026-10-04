```mermaid
flowchart TD
    Entry([Human ask / Jira ticket]) --> Fields{Required input present?}

    Fields -->|no| Blocked[Insufficient input - halt + Jira comment]
    Fields -->|yes| Triage[PM: triage feature vs bug_fix]
    Blocked -->|reporter completes ticket| Fields

    Triage -->|feature: prd + contract| F[Feature lane]
    Triage -->|bug_fix: defect brief| B[Owning executor]

    F -->|HIGH: prd + contract + risk note| P2[PM adds risk note]
    F -->|LOW/MEDIUM| Build
    P2 --> Build
    subgraph Build[Development - executors]
        D[Senior Dev]
        U[UI/UX]
    end
    B --> D
    B --> U

    D --> M[Merged work]
    U --> M

    Budget{Retry budget under 3?}

    M --> Rev[Code Reviewer - in scope only]
    Rev -->|PASS| Tier{Risk tier?}
    Rev -->|BLOCKING_DEFECT| Budget

    Tier -->|LOW| SC[Self-check]
    Tier -->|MEDIUM| SC
    Tier -->|HIGH| V[Verifier - evidence only]
    Tier -->|CRITICAL| V
    V -->|evidence PASS| SC
    V -->|evidence FAIL| Budget

    SC --> Gate2{2o3 tally - agree?}
    Gate2 -->|agree| HG[Human Review Gate]
    Gate2 -->|disagree| Arb[Arbiter on recorded evidence]
    Arb -->|resolved| HG
    Arb -->|unresolved| ESC[Human Escalation - halt]

    Budget -->|yes| M
    Budget -->|no| ESC

    HG -->|APPROVE| Ops[DevOps]
    HG -->|REJECT| Triage

    Ops -->|Deploy OK| Live([LIVE])
    Ops -->|SLO breach| RB[Rollback]
    RB -->|fix root cause| B
    RB -->|repeated failure| ESC

    Live --> Done([Complete])
    ESC --> Done
```

## Reading the diagram
- **Intake gate:** a Jira ticket missing required fields (`repro_steps`,
  `expected`/`actual`, `severity` — see `references/schemas/jira-ticket.schema.md`)
  is **blocked** as `insufficient_input`: the orchestrator halts, posts a
  `jira_update/v1` comment asking the reporter to complete the ticket, and
  dispatches nothing. No agent guesses. Work resumes automatically when the
  completed ticket re-enters the check. (A free-form human ask passes trivially.)
- **Two intake lanes:** a free-form feature ask becomes a PRD + Interface
  Contract; a Jira ticket becomes a Defect Brief routed to the single owning
  executor. Both converge at **Merged work** and share the same validation tail —
  a bug fix is a PR.
- **Reviewer stays in scope:** only `BLOCKING_DEFECT` findings return work to an
  executor; `OUT_OF_SCOPE`/`FOLLOW_UP` never block (review-scope rule, see
  `references/2o3-protocol.md`).
- **2o3 is a protocol, not three permanent agents:** the reviewer always runs for
  non-trivial work; the **verifier** (evidence-only voter) joins at HIGH/CRITICAL.
  A disagreement goes to the **arbiter** on recorded evidence; an unresolved
  disagreement escalates to a human.
- **Human Review Gate:** nothing deploys without a recorded `APPROVE`.
- **Retry budget:** each `BLOCKING_DEFECT`/evidence-FAIL passes the "under 3?"
  check; at 3 the run halts at **Human Escalation** instead of looping.
- **Rollback:** an SLO breach rolls back to the last good release, routes to the
  owning executor for a root-cause fix, and escalates if it repeats.
