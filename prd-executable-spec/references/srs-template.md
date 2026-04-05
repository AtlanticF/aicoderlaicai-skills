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

**Open Items** (if any):
- [ ] Issue description — suggested confirmation: ...

[Repeat above structure for all requirement points]

## 4. Non-Functional Requirements
### 4.1 Performance
### 4.2 Security
### 4.3 Availability

## 5. Cross-Module Constraints
[Data consistency, ordering constraints, and shared invariants across modules]

## 6. Issue Tracker
| ID | Module | Type | Description | Status |
|----|--------|------|-------------|--------|

## Appendix
### A. Full JSON Schema
### B. Full OpenAPI Spec
### C. State Machine Summary
```
