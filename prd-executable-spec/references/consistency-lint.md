# Cross-Artifact Consistency Linter

The five layers are generated separately, so they can drift apart. These are the
inconsistencies engineers hit *during implementation* — catch them before handoff.
Run this checklist in Phase 3 (before declaring a module ready) and again in
Phase 4 (before emitting files).

## Structural check (automatable)

Validate `prd_spec.json` against `schemas/prd_spec.schema.json`:

```bash
check-jsonschema --schemafile prd-executable-spec/schemas/prd_spec.schema.json prd_spec.json
```

This enforces required fields, enums, ID format, and the gate rule (a `confirmed`
point may not carry an open `blocker`/`major` issue).

## Semantic checks (reason over the content)

### 1. Field coherence
- Every field referenced in a Gherkin `Then`/`Given` step exists in the point's
  `data_schema` (or an upstream dependency's schema).
- Every DMN `<input>` maps to a defined data field.
- Every field in the API request/response (`api_spec`) is defined in a schema.

### 2. Enum coherence
- Any literal value used in Gherkin / DMN / pseudocode that represents a state,
  tier, currency, status, etc. must appear in the corresponding schema `enum`.
- Example drift: Gherkin says status `released`, schema `enum` only lists
  `pending, frozen, cancelled` → **fail**.

### 3. Decision coherence
- Every DMN rule's output value is a valid value for the field it sets.
- DMN hit policy is stated and the rule set is complete (no uncovered input
  combination unless a default rule exists).

### 4. State-machine coherence
- Every state named in Gherkin exists in the `state_machine` STATES list.
- Every transition trigger in Gherkin exists as a transition in the state machine.

### 5. Contract coherence
- Every failure-path Gherkin scenario maps to an `error_contract` entry (matching
  code / condition).
- If `idempotent: true`, an `idempotency_key` is defined.
- Every retriable error has a `retry_policy`.

### 6. Glossary / naming coherence
- Every domain concept uses its `canonical_name` from `glossary` across Gherkin,
  schema, DMN, and pseudocode. Aliases appear only in `glossary.aliases`, never in
  generated artifacts.
- Example drift: PRD/Gherkin mixes "agent", "distributor", "referrer" for one
  concept → collapse to the canonical term.

### 7. Dependency coherence
- Every ID in `dependencies` exists in `requirements`.
- No dependency cycles.
- A point does not reference a schema/entity owned by a point it does not depend on.

### 8. NFR coherence
- Every NFR has `metric` + `target` + `condition` + `verification_method`
  (see `references/nfr-verification.md`). Prose-only NFR → raise an `undefined`
  issue.

## Output of a lint run

Report drift as issues on the affected points (do not silently fix schema-vs-behavior
conflicts — they are often real ambiguities):

```markdown
## Consistency Lint — M03 Settlement
- ❌ enum drift (M03-F02-R01): Gherkin uses status "released", schema enum missing it → severity: major
- ❌ glossary drift (M03-F01-R01): "distributor" used in Gherkin; canonical is "agent" → severity: minor (auto-fixable)
- ✅ field coherence: OK
- ✅ dependency graph: acyclic
```

Auto-fixable drift (e.g. naming) can be corrected and noted. Semantic conflicts
(e.g. an enum value that implies a missing state) must become issues with an
appropriate `severity`.
