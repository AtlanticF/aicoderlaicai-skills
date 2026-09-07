---
name: prd-executable-spec
description: Transforms Product Requirement Documents (PRDs) into engineering-grade Executable Specs. Uses ISO/IEC/IEEE 29148 for completeness, BDD Gherkin for behavior alignment, DMN for decisions, pseudocode/math for logic precision, JSON Schema/OpenAPI to eliminate data ambiguity, and verifiable NFRs for non-functional targets. Trigger when the user mentions "review requirements", "review PRD", "requirement analysis", "spec generation", "convert PRD to spec", pastes or references product requirement content, or provides a Lark document link for requirement analysis. Also trigger when the user wants to check, formalize, or re-analyze a changed requirements document, even if they don't use the word "PRD".
---

# PRD → Executable Spec

Transform ambiguous natural-language PRDs into quantifiable, verifiable, engineering-ready specifications.

## Core Concept

Natural language is inherently ambiguous. "Support bulk import" — how large is bulk? "A reasonable timeout" — how many seconds? These expressions inevitably cause misalignment during development.

This skill uses a five-layer precision model to translate each requirement from natural language into engineering language:

| Layer | Tool | Problem Solved |
|-------|------|----------------|
| Macro — Completeness | ISO/IEC/IEEE 29148 | Ensures no sections or dimensions are missing |
| Meso — Behavior | BDD Gherkin (`.feature`) | Aligns product and engineering on expected behavior via Given/When/Then |
| Meso — Decisions | DMN (`.dmn`) | Formalizes conditional rules and decision tables into executable models |
| Micro — Logic | Pseudocode + Math | Precisely defines computation rules, conditions, state transitions |
| Foundation — Data | JSON Schema / OpenAPI | Eliminates ambiguity in field types, formats, and constraints |
| Non-Functional — Verifiable NFR | Measurable target + verification method | Turns performance/security/availability from prose into testable assertions |

See `references/layer-examples.md` for annotated examples of each layer, and `references/nfr-verification.md` for the NFR layer.

The output is not a "list of issues" — it is a **complete Executable Spec** ready to hand off to engineering, including standalone `.feature` and `.dmn` files, and (optionally) failing test stubs.

## What this skill optimizes for (engineering reality)

Precision on a single requirement is necessary but not sufficient. This skill also targets the pains engineers actually hit:

- **Don't stall on trivia** — issues carry a `severity`; only `blocker`/`major` block handoff. `minor` items proceed under a recorded **Assumption** (default-and-proceed), turning synchronous Q&A into asynchronous veto.
- **Partial handoff** — a module whose points are all `confirmed` can ship its artifacts without waiting for the whole PRD.
- **Change survives** — a **Delta mode** diffs a new PRD revision against the last spec and flags stale downstream artifacts along the dependency graph.
- **Contracts engineers argue about** — every point can carry an `error_contract` (codes, idempotency, retries), `observability` (logs/metrics/alerts), and `entity_impact` (new vs. modified entities + migration).
- **Prioritization signal** — every point carries `complexity`, `risk`, and `intent` (the business "why").
- **Machine-checkable** — `prd_spec.json` validates against `schemas/prd_spec.schema.json`; a cross-artifact linter (`references/consistency-lint.md`) catches field/enum/DMN/glossary drift before handoff.

## Workflow Loop

```
Input PRD  ──(if a prior prd_spec.json exists)── Delta mode: diff + stale-artifact scan
    ↓
Phase 1: Structure scan → Requirement point index (+ complexity/risk)
    ↓ (wait for user confirmation)
Phase 2: Progressive spec — one point at a time (behavior, logic, data, contracts, NFR)
    ↓
Phase 3: Issues checklist (by severity) → clarify blockers/majors; record assumptions for minors
    ↓ (wait for user feedback on blockers/majors)
    ↓ ← re-scan after each feedback round for new issues
    ↓ (repeat until no open blocker/major issues remain)
    ↓ Run consistency linter → fix drift
Phase 4: Output .feature + .dmn + NFR + assumptions + prd_spec_report.md + prd_spec.json (+ test stubs)
```

**Tiered gate**: Phase 4 (for a given module) is blocked while any of its requirement points has an open `blocker` or `major` issue. `minor` issues do **not** block, provided each is covered by an Assumption. After each feedback round, re-scan the updated spec for newly introduced issues, then run the consistency linter, before declaring a module ready.

## Delta Mode (re-analyzing a changed PRD)

If a prior `prd_spec.json` exists (user provides it, or it is on disk), run Delta mode first:

1. Compute/compare `source_hash` against the new PRD revision.
2. For each requirement point, classify as `added | modified | removed` and write a `change_log` entry.
3. Walk the `dependencies` graph: any point that depends on a changed point has its artifacts (`.feature`, `.dmn`, schema) marked **stale** and must be re-derived.
4. Reset `spec_status` to `draft` for changed and stale points; keep untouched points `confirmed`.
5. Present the change summary (added/modified/removed + stale downstream) before regenerating.

This makes the spec a living document instead of a one-shot artifact.

## Input

1. **Lark document link** — fetch content using WebFetch
2. **Pasted Markdown** — user pastes directly into the conversation
3. **Local file** — user specifies a `.md` / `.txt` path, read using the Read tool
4. **Images** — PRDs may include diagrams or screenshots; use the Read tool to load image files and incorporate visual content
5. **Prior `prd_spec.json`** — if present, triggers Delta mode

## Phases

### Phase 1: Structure Scan — ISO/IEC/IEEE 29148 Completeness Check

Read the full document (text and images). Map content against ISO/IEC/IEEE 29148 dimensions using `references/29148-checklist.md`.

Output to user:

```markdown
## ISO/IEC/IEEE 29148 Completeness Scan

### Covered
- ✅ Functional requirements: 3 modules, 12 feature points
- ✅ Terms defined: "agent", "commission period", and 3 others

### Missing
- ❌ Performance requirements: No throughput, response time, or concurrency targets defined
- ❌ Operational requirements: No timezone basis or trigger time window specified
- ❌ External interfaces: Integration protocol with payment system not described

### Requirement Point Index (with dependencies, complexity, risk)
- M01 — User module (3 feature points) — complexity S, risk low
- M02 — Transaction module (5 feature points) → depends on M01 — complexity M, risk medium
- M03 — Settlement module (4 feature points) → depends on M01, M02 — complexity L, risk high
```

**After Phase 1: output the requirement point index and wait for user confirmation before proceeding to Phase 2. Do not expand all requirement points at once.**

Assign `complexity` (S/M/L/XL) and `risk` (low/medium/high) to each point so the user can prioritize. Process high-risk / high-complexity points earlier.

If an unfamiliar business concept appears, ask the user to clarify — do not guess.

---

### Phase 2: Progressive Spec Generation — One Requirement Point at a Time

**Core rule: process one requirement point per round.**

After completing each point: output the spec, list issues (with severity), record assumptions for `minor` items, wait for acknowledgment before moving to the next. Process in dependency order (upstream first), highest risk first within a tier.

Capture `intent` (the business goal / metric this point moves) and `acceptance_owner` for every point. See `references/layer-examples.md` for full annotated examples of the layers below.

#### 2.1 BDD Gherkin Scenarios (Meso — Behavior Layer)

Translate each requirement point into Gherkin scenarios. Cover both happy paths and failure paths. Example:

```gherkin
Feature: M03-F01 Leaderboard Calculation
  Scenario: Daily leaderboard — triggered after scoring task
    Given the scoring task for the current day has completed
    And the current time in the user's local timezone is before 02:00
    When the system triggers leaderboard calculation
    Then the leaderboard is computed based on completed scoring results
```

#### 2.2 DMN Decision Tables (Meso — Decision Layer)

Use DMN when multiple input conditions map to a discrete output (tier → rate, role → permissions, trigger conditions). Save each as `{module}-{decision-name}.dmn`. See `references/layer-examples.md` for full XML.

#### 2.3 Pseudocode + Math (Micro — Logic Layer)

For computation, rule engines, and state machines. Example:

```
FUNCTION calculate_commission(trade):
  rate = CASE user.tier OF
    "bronze" → 0.10  |  "silver" → 0.20  |  "gold" → 0.30
    DEFAULT  → ERROR("unknown tier: " + user.tier)
  commission = ROUND_DOWN(trade.volume × trade.fee_rate × rate, 8)
  ASSERT commission ≥ 0
  RETURN commission
```

See `references/layer-examples.md` for formula and state machine patterns.

#### 2.4 JSON Schema / OpenAPI + Entity Impact (Foundation — Data Layer)

Define all data entities and interfaces. Additionally record `entity_impact`: which entities are **new** vs. **modified**, and any migration implications — the spec must not pretend the system is greenfield. See `references/layer-examples.md`.

#### 2.5 Error, Idempotency & Observability Contract

For any behavior that writes state or exposes an interface, define:

- `error_contract` — error codes, HTTP status, when they fire, whether retriable; plus `idempotent` + `idempotency_key` + `retry_policy`.
- `observability` — key logs, metrics, and alert conditions.

These are where interfaces and on-call debugging break down; make them explicit. See `references/layer-examples.md`.

#### 2.6 Verifiable NFR (Non-Functional Layer)

For each relevant non-functional dimension flagged in Phase 1, produce a measurable assertion with a verification method (e.g. `p99 < 200ms @ 1000 rps → load_test`). Do not leave NFRs as prose. See `references/nfr-verification.md`.

#### 2.7 Output structure per requirement point

Each requirement point becomes one entry in `prd_spec.json`. See `references/requirement-json-schema.md` for the full field schema and `spec_status` lifecycle (`draft` → `needs_clarification` → `confirmed`), and `schemas/prd_spec.schema.json` for the machine-validatable version.

**ID scheme**: `M{module#}-F{feature#}-R{requirement#}`

---

### Phase 3: Issue Clarification — Severity-Ranked Q&A and Iteration

Group issues by module, then by severity. Questions must use engineering language, not natural language.

**Issue types**:

| Type | Engineering expression |
|------|----------------------|
| `undefined` | List missing Gherkin Scenarios or Schema fields |
| `logic_flaw` | Use pseudocode/formula to mark the contradictory execution path |
| `contradiction` | Show two code blocks side by side, mark the conflict |
| `ambiguity` | Provide 2–3 formal interpretations (A/B/C), ask product to choose |
| `missing_boundary` | List Schema fields missing min/max/pattern/enum |
| `missing_edge_case` | List missing failure-path Scenarios |

**Severity drives handling**:

| Severity | Action |
|----------|--------|
| `blocker` | Must be answered; blocks the whole affected module. Do not proceed to Phase 4 for that module. |
| `major` | Must be answered; blocks the affected requirement point. |
| `minor` | Do **not** stall. Record an **Assumption** (default choice + rationale + who can veto), mark the point `confirmed`, and continue. |

**Default-and-proceed**: for every `minor` issue, write an Assumption object (see `references/requirement-json-schema.md`). Silence is treated as acceptance; the product owner can veto later, which reopens the point. Present the running assumption register each round so nothing is silently decided.

After user replies: update `prd_spec.json`, advance `spec_status`, and if a reply surfaces new questions, append new issues and continue the loop. Before declaring a module ready, run the consistency linter (`references/consistency-lint.md`).

---

### Phase 4: Spec Document Output

**Gate check (per module)**: Count requirement points in the module with an open `blocker` or `major` issue. If count > 0, return to Phase 3 for that module — do not proceed. A module with only `minor` issues (each backed by an Assumption) may proceed. Modules that pass may be handed off independently (partial handoff).

**Pre-output linter**: run `references/consistency-lint.md`. Fix any field/enum/DMN-input/glossary drift before emitting files.

Generate `prd_spec_report.md` following the SRS structure in `references/srs-template.md`.

**Output files**:

| File | Content |
|------|---------|
| `prd_spec_report.md` | Full human-readable Executable Spec |
| `prd_spec.json` | Machine-readable structured spec (validate against `schemas/prd_spec.schema.json`) |
| `assumptions.md` | Human-readable assumption register (mirror of `assumptions[]`) |
| `{module}-{feature}.feature` | One Gherkin `.feature` file per Feature block |
| `{module}-{decision}.dmn` | One DMN `.dmn` file per decision table |
| `{module}-{feature}.steps.<ext>` | (Optional) failing test-step stubs generated from the `.feature`, so the spec is literally runnable |

**Naming**: `m01-user-registration.feature`, `m03-leaderboard-trigger.dmn`

Each `.feature` file contains one complete `Feature:` block. Each `.dmn` file contains one `<decision>` element. Do not bundle multiple features or decisions into a single file.

**Optional CI integration**: validate the emitted `prd_spec.json` with `check-jsonschema --schemafile schemas/prd_spec.schema.json prd_spec.json` (or any draft 2020-12 validator). This automates the gate — a `confirmed` point carrying an open `blocker`/`major` issue fails validation.

---

## Key Principles

1. **Eliminate natural language**: Every requirement point must have at least one engineering-language expression. If it can only be expressed in natural language, mark it `needs_clarification`.
2. **Cite source text**: Every translation retains `original_text` pointing back to the PRD for traceability, and `intent` for the business "why".
3. **Report all issues, but rank them**: List every issue found; do not filter by perceived importance. Assign a `severity` so blockers surface first and trivia never stalls delivery.
4. **Default-and-proceed on minors**: Never block the loop on a `minor` issue — record an Assumption and continue. Silence is acceptance; vetoes reopen the point.
5. **Enforce the tiered gate**: Phase 4 is hard-gated on `blocker`/`major` issues, per module. Partial handoff of clean modules is allowed.
6. **Keep it consistent**: Run the cross-artifact linter before handoff — no field used in Gherkin/DMN that is missing from the schema, no enum drift, no glossary drift.
7. **Design for change**: On re-analysis, run Delta mode and propagate staleness along dependencies.
8. **Ground in the real system**: Record `entity_impact` (new vs. modified + migration), `error_contract`, and `observability` — not just greenfield happy paths.
9. **Persist outputs**: Save `prd_spec.json`, `prd_spec_report.md`, and `assumptions.md` to disk after each phase.
