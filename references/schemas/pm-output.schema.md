# Schema: `pm_output/v1` — `prd.json` (feature lane)

Feature-lane output from `pm`. The `interface_contract` block is what `senior-dev`
and `ui-ux` implement against. Single canonical copy.

```json
{
  "schema": "pm_output/v1",
  "task_id": "uuid",
  "work_type": "feature",
  "feature_name": "User Authentication",
  "problem_statement": "...",
  "user_stories": [ { "id": "US-1", "as_a": "...", "i_want": "...", "so_that": "..." } ],
  "acceptance_criteria": [
    "Feature: Login\n  Scenario: Valid credentials\n    Given ... When ... Then ..."
  ],
  "edge_cases": ["empty input", "expired token", "rate limit"],
  "interface_contract": {
    "typescript": "export interface LoginRequest { email: string; password: string } ...",
    "openapi": {
      "openapi": "3.0.0",
      "paths": {
        "/api/v1/auth/login": {
          "post": {
            "requestBody": { "content": { "application/json": { "schema": { "$ref": "#/components/schemas/LoginRequest" } } } },
            "responses": {
              "200": { "description": "OK", "content": { "application/json": { "schema": { "$ref": "#/components/schemas/LoginResponse" } } } },
              "400": { "description": "Validation error" },
              "401": { "description": "Invalid credentials" },
              "429": { "description": "Rate limited" },
              "500": { "description": "Server error" }
            }
          }
        }
      },
      "components": { "schemas": {
        "LoginRequest": { "type": "object", "properties": { "email": { "type": "string", "format": "email" }, "password": { "type": "string" } }, "required": ["email", "password"] },
        "LoginResponse": { "type": "object", "properties": { "token": { "type": "string" }, "expires_at": { "type": "number" } }, "required": ["token", "expires_at"] }
      } }
    }
  },
  "spec_contradiction": null
}
```

Rules:
- `acceptance_criteria` are Gherkin (Given/When/Then) and executable verbatim by
  the verifier.
- `interface_contract` MUST cover 4xx/5xx error states for every endpoint.
- Bug-fix lane does NOT use this schema — it uses `defect_brief/v1`
  (`references/schemas/defect-brief.schema.md`).
