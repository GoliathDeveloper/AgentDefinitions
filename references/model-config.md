# Model Config (GLM 5.3-flash)

Current runtime model: **GLM 5.3-flash**. The architecture is model-agnostic — if
the model changes, update this file and re-validate the gates. This file is loaded
on demand (never embedded in agent prompts).

| Agent | Mode | Temperature | Top-p | Reasoning effort | Quant | Est. VRAM |
|---|---|---|---|---|---|---|
| orchestrator | Instruct | 0.1 | 0.90 | Medium | Q4_K_M | ~8–12 GB |
| pm | Thinking | 0.6 | 0.95 | High | Q4_K_M | ~8–12 GB |
| senior-dev | Thinking/Instruct hybrid | 0.2 | 0.95 | High | Q8_0 | ~16–22 GB |
| ui-ux | Instruct | 0.3 | 0.95 | Medium | Q4_K_M | ~8–12 GB |
| code-reviewer | Thinking | 0.1 | 0.90 | High | Q8_0 | ~16–22 GB |
| verifier | Instruct | 0.1 | 0.90 | Medium | Q4_K_M | ~8–12 GB |
| devops | Instruct | 0.1 | 0.90 | Low | Q4_K_M | ~8–12 GB |

## Rationale
- Code-producing agents run low temperature (0.1–0.3): deterministic, reproducible
  output; code generation must not be "creative".
- `code-reviewer` and `senior-dev` run Q8_0: security verdicts and code
  correctness are the most quantization-sensitive.
- Reviewer at 0.1 keeps verdicts reproducible: same input → same PASS/REJECT.
- The verifier uses fixed seeds for any randomized/fuzz inputs.

## Open items
- VRAM figures are estimates for a GLM 5.3-flash-class model — verify against the
  official model card and update this table.
- 2o3 voters (`code-reviewer`, `verifier`) run in isolated contexts: share only
  diff + requirements + contract + evidence, never the executor's reasoning trace.
