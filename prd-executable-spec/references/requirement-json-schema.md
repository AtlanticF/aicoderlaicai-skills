# Spec JSON Schema Reference

This document describes the shape of `prd_spec.json`. A machine-validatable JSON
Schema lives at `schemas/prd_spec.schema.json` — use it in CI to enforce the gate
automatically instead of eyeballing statuses.

## Top-Level Document

```json
{
  "spec_version": "1.2.0",
  "source_document": "string (PRD title or link)",
  "source_hash": "string (hash/etag of the PRD revision this spec was built from)",
  "generated_at": "string (date-time)",
  "glossary": [
    { "term": "agent", "definition": "string", "canonical_name": "agent", "aliases": ["distributor"] }
  ],
  "requirements": [ /* array of Requirement Point objects, see below */ ],
  "assumptions": [ /* array of Assumption objects, see below */ ],
  "change_log": [ /* array of Change objects, populated in Delta mode */ ]
}
```

## Requirement Point — Full Field Schema

```json
{
  "id": "M01-F01-R01",
  "module": "string",
  "feature": "string",
  "requirement": "string",
  "original_text": "string",
  "intent": "string (the business goal / metric this requirement moves — the WHY, not the WHAT)",
  "29148_category": "functional | interface | performance | constraint | quality",
  "complexity": "S | M | L | XL",
  "risk": "low | medium | high",
  "acceptance_owner": "string (who signs off on this point)",

  "gherkin": "string (complete Feature + Scenario text)",

  "logic": {
    "pseudocode": "string | null",
    "formula": "string | null",
    "state_machine": "string | null",
    "conditions": ["string"]
  },

  "data_schema": "object | null (JSON Schema draft 2020-12)",
  "entity_impact": {
    "new_entities": ["string"],
    "modified_entities": ["string"],
    "migration_notes": "string | null"
  },

  "api_spec": {
    "path": "string",
    "method": "GET | POST | PUT | DELETE | PATCH",
    "parameters": [],
    "request_body": "object | null",
    "responses": {}
  },

  "error_contract": {
    "errors": [
      { "code": "string", "http_status": 400, "when": "string", "retriable": false }
    ],
    "idempotent": true,
    "idempotency_key": "string | null",
    "retry_policy": "string | null"
  },

  "observability": {
    "logs": ["string (key events that must be logged)"],
    "metrics": ["string (counters/gauges/histograms to emit)"],
    "alerts": ["string (alert conditions)"]
  },

  "nfr": [ /* array of NFR objects, see references/nfr-verification.md */ ],

  "dependencies": ["string (other requirement point IDs)"],

  "issues": [
    {
      "type": "undefined | logic_flaw | contradiction | ambiguity | missing_boundary | missing_edge_case",
      "severity": "blocker | major | minor",
      "location": "gherkin | pseudocode | formula | state_machine | data_schema | api_spec | error_contract | observability | nfr | cross_module",
      "original_text": "string",
      "description": "string (engineering language)",
      "proposed_fix": "string (pseudocode / formula / schema fragment)",
      "question": "string (specific question for the product owner)"
    }
  ],

  "spec_status": "draft | needs_clarification | confirmed"
}
```

## Assumption Object

When an issue is non-blocking, do NOT stall the loop. Record a default decision
and proceed. Product can veto later; silence is treated as acceptance.

```json
{
  "id": "A-M03-01",
  "requirement_id": "M03-F01-R01",
  "statement": "Leaderboard ties are broken by earliest completion timestamp.",
  "default_choice": "earliest_timestamp_wins",
  "alternatives": ["highest_user_id_wins", "random"],
  "rationale": "Most common convention; deterministic and auditable.",
  "needs_veto_by": "product owner",
  "status": "assumed | vetoed | ratified"
}
```

## Change Object (Delta mode)

Populated when re-running against a new PRD revision. Drives stale-artifact
detection along the dependency graph.

```json
{
  "requirement_id": "M03-F02-R01",
  "change_type": "added | modified | removed",
  "summary": "string (what changed in engineering terms)",
  "stale_artifacts": ["m03-leaderboard.feature", "m03-leaderboard-trigger.dmn"],
  "impacted_downstream": ["M04-F01-R02 (depends on M03-F02-R01)"]
}
```

## spec_status Lifecycle

| Status | Meaning |
|--------|---------|
| `draft` | Initial conversion done, not yet confirmed |
| `needs_clarification` | Has at least one open `blocker` or `major` issue |
| `confirmed` | No open `blocker`/`major` issues; remaining `minor` issues are covered by assumptions |

> A point with only `minor` issues that are all backed by an assumption may be
> marked `confirmed`. A point with any open `blocker` or `major` issue must be
> `needs_clarification`.

## Issue Severity — Gate Semantics

| Severity | Meaning | Blocks handoff? |
|----------|---------|-----------------|
| `blocker` | Ambiguity that changes architecture, data model, or contracts; cannot start implementation | Yes — blocks the whole affected module |
| `major` | Wrong behavior likely if unresolved, but implementation shape is known | Yes — blocks the affected requirement point |
| `minor` | Cosmetic / low-impact; safe to proceed under a stated assumption | No — proceed with an Assumption |

## Five-Layer Quick Reference

| Layer | Tool | When to Use | Output Format |
|-------|------|-------------|---------------|
| Macro | ISO/IEC/IEEE 29148 | Phase 1 completeness scan | Checklist (✅/❌) |
| Meso — Behavior | BDD Gherkin | All functional requirement points | Feature + Scenario |
| Meso — Decision | DMN | Conditional rules / decision tables | `.dmn` XML |
| Micro | Pseudocode + Math | Computation / rules / state machines | FUNCTION / formula / STATE_MACHINE |
| Foundation | JSON Schema + OpenAPI | Data entities and interfaces | JSON Schema / YAML |
| Non-Functional | Verifiable NFR | Performance / security / availability targets | Measurable assertion + verification method (see `references/nfr-verification.md`) |

## Issue Type → Engineering Expression

| Type | How to express the problem |
|------|---------------------------|
| `undefined` | List missing Gherkin Scenarios or Schema fields |
| `logic_flaw` | Use pseudocode/formula to mark the contradictory execution path |
| `contradiction` | Show two code blocks side by side, mark the conflict point |
| `ambiguity` | Provide 2–3 formal interpretations (A/B/C choice) |
| `missing_boundary` | List Schema fields missing min/max/pattern/enum |
| `missing_edge_case` | List missing failure-path Scenarios |
