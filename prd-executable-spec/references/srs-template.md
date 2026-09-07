# SRS Document Template

Use this structure when generating `prd_spec_report.md` in Phase 4.

```markdown
# Software Requirements Specification (SRS)

**Source document**: [PRD title]
**Version**: 1.0
**Date**: [date]
**Status**: Draft / Review / Approved

## 1. Introduction
### 1.1 Purpose
### 1.2 Scope
### 1.3 Terms & Definitions
| Term | Definition | Constraints |
|------|------------|-------------|

## 2. System Overview
### 2.1 System Context
### 2.2 Module Dependency Map
[ASCII or Mermaid diagram of module dependencies]

## 3. Functional Specification (by module)

### 3.1 [Module Name]

#### 3.1.1 [Feature Name]

**Requirement M01-F01-R01: [Name]**

**Intent**: [the business goal / metric this requirement moves]
**Complexity**: S/M/L/XL   **Risk**: low/medium/high   **Acceptance owner**: [name]

**Behavior Spec (BDD)**:
​```gherkin
Feature: ...
  Scenario: ...
​```

**Computation / Rule Logic**:
​```
FUNCTION ...
​```

**Math Expression**:
​```
formula = ...
​```

**Data Definition**:
​```json
{ "$schema": "...", "properties": {} }
​```

**Interface Definition** (if applicable):
​```yaml
/api/v1/...:
  get: ...
​```

**Entity Impact**:
- New: [entities] | Modified: [entities] | Migration: [notes]

**Error / Idempotency / Observability Contract**:
- Errors: [code → when → retriable] | Idempotent: yes/no (key: ...) | Retry: ...
- Logs / Metrics / Alerts: ...

**Verifiable NFRs** (if applicable):
| Metric | Target | Condition | Verification |
|--------|--------|-----------|--------------|

**Open Items** (if any):
- [ ] [severity] Issue description — suggested confirmation: ...

[Repeat above structure for all requirement points]

## 4. Non-Functional Requirements (verifiable)
Every entry is measurable: metric + target + condition + verification method.
See `references/nfr-verification.md`.
| Category | Metric | Target | Condition | Verification |
|----------|--------|--------|-----------|--------------|

## 5. Cross-Module Constraints
[Data consistency, ordering constraints, and shared invariants across modules]

## 6. Assumption Register
Default decisions taken for `minor` issues. Silence = acceptance; product may veto.
| ID | Requirement | Assumption | Default | Needs veto by | Status |
|----|-------------|-----------|---------|---------------|--------|

## 7. Issue Tracker
| ID | Module | Type | Severity | Description | Status |
|----|--------|------|----------|-------------|--------|

## 8. Change Log (Delta mode)
Populated when re-analyzing a changed PRD. Lists what changed and which
downstream artifacts went stale.
| Requirement | Change | Summary | Stale artifacts | Impacted downstream |
|-------------|--------|---------|-----------------|---------------------|

## Appendix
### A. Full JSON Schema
### B. Full OpenAPI Spec
### C. State Machine Summary
### D. Glossary (canonical terms + aliases)
```
