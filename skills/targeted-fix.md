---
name: targeted-fix
description: Use after debug-investigate (or when a failure payload already cites exact lines) to apply the minimal patch that resolves the cited defect — never a rewrite, never drive-by improvements.
---

# Targeted Fix

## Goal
Produce the minimal unified diff that resolves every cited item and nothing else.

## Preconditions
- A root-cause diagnosis (from `debug-investigate`) or a failure payload with
  file/line + failing assertion.
- The Interface Contract (to verify conformance before and after).

## Procedure
1. Re-state each cited item and its minimal patch boundary.
2. Patch ONLY the cited files/lines; preserve surrounding style, types, naming.
3. If the fix changes observable behavior (response shape, status code), check
   the Interface Contract still holds. If it does not, stop: contract changes
   require `pm` approval (`spec_contradiction` path).
4. Add/extend a test covering the exact repro where the suite allows; otherwise
   record it in `regression_note`.
5. `dev:git-diff` — confirm the diff contains only intended changes.
6. Hand to `self-check`.

## Guardrails
- No unrelated rewrites, no drive-by refactors, no "improvements" found en route.
- Never modify the Interface Contract without explicit approval in the payload.
- One defect class per hunk; cross-cutting defects get separate items.
- If the fix cannot fit in a bounded diff (it is actually a redesign), stop and
  escalate as a spec/plan defect — do not force it.

## Verification
- The diff touches only files in the patch boundary.
- Lint + typecheck clean on changed files.
- `regression_note` names the behavior the verifier must cover.

## Output
- `senior_dev_output/v1` (`references/schemas/senior-dev-output.schema.md`) or
  `ui_ux_output/v1` (`references/schemas/ui-ux-output.schema.md`):
  `status=FIX_OK`, `patch`, `root_cause`, `regression_note`, `addressed_items`.
