---
name: prd-executable-spec
description: Transforms Product Requirement Documents (PRDs) into engineering-grade Executable Specs. Uses ISO/IEC/IEEE 29148 for completeness, BDD Gherkin for behavior alignment, pseudocode/math for logic precision, and JSON Schema/OpenAPI to eliminate data ambiguity. Trigger when the user mentions "review requirements", "review PRD", "requirement analysis", "spec generation", "convert PRD to spec", pastes or references product requirement content, or provides a Lark document link for requirement analysis. Also trigger when the user wants to check or formalize any requirements document, even if they don't use the word "PRD".
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

See `references/layer-examples.md` for annotated examples of each layer.

The output is not a "list of issues" — it is a **complete Executable Spec** ready to hand off to engineering, including standalone `.feature` and `.dmn` files.

## Workflow Loop

```
Input PRD
    ↓
Phase 1: Structure scan → Requirement point index
    ↓ (wait for user confirmation)
Phase 2: Progressive spec — one point at a time
    ↓
Phase 3: Output issues checklist
    ↓ (wait for user feedback)
    ↓ ← re-scan after each feedback round for new issues
    ↓ (repeat until checklist is empty)
    ↓ ALL spec_status = "confirmed"
Phase 4: Output .feature + .dmn + prd_spec_report.md + prd_spec.json
```

**Hard gate**: Phase 4 is blocked as long as any requirement point has `spec_status: needs_clarification`. After each round of user feedback, re-scan the updated spec for newly introduced issues before declaring the checklist empty.

## Input

1. **Lark document link** — fetch content using WebFetch
2. **Pasted Markdown** — user pastes directly into the conversation
3. **Local file** — user specifies a `.md` / `.txt` path, read using the Read tool
4. **Images** — PRDs may include diagrams or screenshots; use the Read tool to load image files and incorporate visual content

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

### Requirement Point Index (with dependency relationships)
- M01 — User module (3 feature points)
- M02 — Transaction module (5 feature points) → depends on M01
- M03 — Settlement module (4 feature points) → depends on M01, M02
```

**After Phase 1: output the requirement point index and wait for user confirmation before proceeding to Phase 2. Do not expand all requirement points at once.**

If an unfamiliar business concept appears, ask the user to clarify — do not guess.

---

### Phase 2: Progressive Spec Generation — One Requirement Point at a Time

**Core rule: process one requirement point per round.**

After completing each point: output the spec, list issues, wait for acknowledgment before moving to the next. Process in dependency order (upstream first).

See `references/layer-examples.md` for full annotated examples of all layers below.

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

#### 2.4 JSON Schema / OpenAPI (Foundation — Data Layer)

Define all data entities and interfaces. See `references/layer-examples.md` for complete examples.

#### 2.5 Output structure per requirement point

Each requirement point becomes one entry in `prd_spec.json`. See `references/requirement-json-schema.md` for the full field schema and `spec_status` lifecycle (`draft` → `needs_clarification` → `confirmed`).

**ID scheme**: `M{module#}-F{feature#}-R{requirement#}`

---

### Phase 3: Issue Clarification — Structured Q&A and Iteration

Group issues by module. Questions must use engineering language, not natural language.

**Issue types**:

| Type | Engineering expression |
|------|----------------------|
| `undefined` | List missing Gherkin Scenarios or Schema fields |
| `logic_flaw` | Use pseudocode/formula to mark the contradictory execution path |
| `contradiction` | Show two code blocks side by side, mark the conflict |
| `ambiguity` | Provide 2–3 formal interpretations (A/B/C), ask product to choose |
| `missing_boundary` | List Schema fields missing min/max/pattern/enum |
| `missing_edge_case` | List missing failure-path Scenarios |

After user replies: update `prd_spec.json`, advance `spec_status` to `confirmed`. If the reply surfaces new questions, append new issues and continue the loop.

---

### Phase 4: Spec Document Output

**Gate check**: Count entries in `prd_spec.json` where `spec_status != "confirmed"`. If count > 0, return to Phase 3 — do not proceed.

Generate `prd_spec_report.md` following the SRS structure in `references/srs-template.md`.

**Output files**:

| File | Content |
|------|---------|
| `prd_spec_report.md` | Full human-readable Executable Spec |
| `prd_spec.json` | Machine-readable structured spec |
| `{module}-{feature}.feature` | One Gherkin `.feature` file per Feature block |
| `{module}-{decision}.dmn` | One DMN `.dmn` file per decision table |

**Naming**: `m01-user-registration.feature`, `m03-leaderboard-trigger.dmn`

Each `.feature` file contains one complete `Feature:` block. Each `.dmn` file contains one `<decision>` element. Do not bundle multiple features or decisions into a single file.

---

## Key Principles

1. **Eliminate natural language**: Every requirement point must have at least one engineering-language expression. If it can only be expressed in natural language, mark it `needs_clarification`.
2. **Cite source text**: Every translation retains `original_text` pointing back to the PRD for traceability.
3. **Report all issues**: List every issue found; do not filter by perceived importance.
4. **Enforce the loop**: Phase 4 is hard-gated. After each feedback round, re-scan for newly introduced issues before declaring the checklist empty.
5. **Persist outputs**: Save `prd_spec.json` and `prd_spec_report.md` to disk after each phase.
