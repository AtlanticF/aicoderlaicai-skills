# Spec JSON Schema Reference

## Requirement Point — Full Field Schema

```json
{
  "id": "M01-F01-R01",
  "module": "string",
  "feature": "string",
  "requirement": "string",
  "original_text": "string",
  "29148_category": "functional | interface | performance | constraint | quality",
  "gherkin": "string (complete Feature + Scenario text)",
  "logic": {
    "pseudocode": "string | null",
    "formula": "string | null",
    "state_machine": "string | null",
    "conditions": ["string"]
  },
  "data_schema": "object | null (JSON Schema draft 2020-12)",
  "api_spec": {
    "path": "string",
    "method": "GET | POST | PUT | DELETE | PATCH",
    "parameters": [],
    "request_body": "object | null",
    "responses": {}
  },
  "dependencies": ["string (other requirement point IDs)"],
  "issues": [
    {
      "type": "undefined | logic_flaw | contradiction | ambiguity | missing_boundary | missing_edge_case",
      "location": "gherkin | pseudocode | formula | state_machine | data_schema | api_spec | cross_module",
      "original_text": "string",
      "description": "string (engineering language)",
      "proposed_fix": "string (pseudocode / formula / schema fragment)",
      "question": "string (specific question for the product owner)"
    }
  ],
  "spec_status": "draft | needs_clarification | confirmed"
}
```

## spec_status Lifecycle

| Status | Meaning |
|--------|---------|
| `draft` | Initial conversion done, not yet confirmed |
| `needs_clarification` | Has open issues requiring product confirmation |
| `confirmed` | All issues resolved, spec is ready for handoff |

## Five-Layer Quick Reference

| Layer | Tool | When to Use | Output Format |
|-------|------|-------------|---------------|
| Macro | ISO/IEC/IEEE 29148 | Phase 1 completeness scan | Checklist (✅/❌) |
| Meso — Behavior | BDD Gherkin | All functional requirement points | Feature + Scenario |
| Meso — Decision | DMN | Conditional rules / decision tables | `.dmn` XML |
| Micro | Pseudocode + Math | Computation / rules / state machines | FUNCTION / formula / STATE_MACHINE |
| Foundation | JSON Schema + OpenAPI | Data entities and interfaces | JSON Schema / YAML |

## Issue Type → Engineering Expression

| Type | How to express the problem |
|------|---------------------------|
| `undefined` | List missing Gherkin Scenarios or Schema fields |
| `logic_flaw` | Use pseudocode/formula to mark the contradictory execution path |
| `contradiction` | Show two code blocks side by side, mark the conflict point |
| `ambiguity` | Provide 2–3 formal interpretations (A/B/C choice) |
| `missing_boundary` | List Schema fields missing min/max/pattern/enum |
| `missing_edge_case` | List missing failure-path Scenarios |
