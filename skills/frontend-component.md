---
name: frontend-component
description: Use when building frontend components from the PM Interface Contract and wireframes — React/Tailwind, typed API integration, loading/empty/error states, accessibility and responsive layout.
---

# Frontend Component

## Goal
Translate the approved Interface Contract + wireframes into production-ready
React/Tailwind components with zero type drift.

## Preconditions
- `prd.json` `interface_contract` (TypeScript types + OpenAPI) and wireframes if
  available.

## Procedure
1. Map every endpoint to a typed fetch handler; import types from the contract
   (never redefine them locally).
2. Build components: modular, reusable, fully typed props.
3. Implement loading / empty / error / 4xx / 5xx states in every data view.
4. Local state + client-side validation mirroring the OpenAPI validation rules.
5. Responsive layout across viewports; WCAG: labels, contrast, focus order,
   keyboard paths.
6. `ui:check-types` + `ui:check-a11y`.
7. Hand to `self-check`.

## Guardrails
- No new API types: consume the contract exactly; a type mismatch is a defect,
  not a local "fix".
- No inline secrets; no hardcoded endpoints outside the configured base URL.
- No visual scope creep beyond the wireframe/contract.

## Verification
- `ui:check-types`: `types_match=true`. `ui:check-a11y`: `wcag_check=pass`.
- Self-check exercises the component in each data state.

## Output
- `ui_ux_output/v1` (`references/schemas/ui-ux-output.schema.md`): `files`,
  `accessibility`, `contract_conformance` (`types_match=true`, `deviations: []`).
