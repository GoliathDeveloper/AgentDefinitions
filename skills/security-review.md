---
name: security-review
description: Use when reviewing a diff for security — OWASP Top 10, injection, secrets exposure, authn/authz flaws, crypto misuse — typically as part of the code-reviewer's HIGH/CRITICAL pass.
---

# Security Review

## Goal
Find demonstrable security defects in the merged diff and classify each under
the review-scope rule.

## Preconditions
- Merged diff + Interface Contract + repository conventions.

## Procedure
1. `reviewer:static-analyze` on the diff.
2. `reviewer:owasp-scan`: injection (SQL/command/XSS), authn/authz bypass,
   broken object-level authorization, secrets exposure, insecure deserialization,
   crypto misuse.
3. Cross-check contract error states: do 4xx/5xx responses leak internals
   (stack traces, SQL, paths)?
4. Check new dependencies: known-vulnerable packages; unpinned image digests in
   infra are flagged (infra itself is devops' lane).
5. For each finding: file, line, evidence, severity (`critical|major|minor`),
   classification (`BLOCKING_DEFECT | FOLLOW_UP | OUT_OF_SCOPE`), concrete
   suggested fix.

## Guardrails
- Evidence-only: "could be a problem" without a demonstrated path is `FOLLOW_UP`,
  never blocking.
- No scope expansion into unrelated legacy code (review-scope rule).
- No secrets in the report — reference by location.

## Verification
- Every OWASP category checked or explicitly `NOT APPLICABLE` with reason.
- Findings are reproducible from the cited evidence.

## Output
- Findings appended to the code-reviewer's `agent_failure_payload/v1`
  (`category=security`) + advisory list.
