# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a custom skills repository for aicoderlaicai's Claude Code installation. Each subdirectory is a standalone skill that Claude Code can invoke via a `/skill-name` slash command.

## Skill Structure

Each skill lives in its own directory and must contain a `SKILL.md` file as its entry point:

```
<skill-name>/
  SKILL.md           # Required: frontmatter + full skill instructions
  references/        # Optional: supporting reference docs the skill may read
```

### SKILL.md Frontmatter

Every `SKILL.md` must begin with YAML frontmatter:

```yaml
---
name: skill-name           # matches the directory name and slash command
description: ...           # one-paragraph description; Claude uses this to decide when to trigger the skill
---
```

The `description` field is critical — it determines when Claude auto-triggers the skill. It should list explicit trigger phrases and URL patterns the skill handles.

## Current Skills

- **prd-executable-spec** — Transforms PRDs into engineering-grade Executable Specs using a 5-layer precision model (ISO/IEC/IEEE 29148 completeness, BDD Gherkin behavior, DMN decision tables, pseudocode/math logic, JSON Schema/OpenAPI data). Outputs `prd_spec_report.md`, `prd_spec.json`, `.feature` files, and `.dmn` files. Enforces a hard gate: Phase 4 output is blocked until all requirement points reach `spec_status: "confirmed"`.

### prd-executable-spec key files
- `prd-executable-spec/SKILL.md` — Full workflow, phase definitions, output formats, and key principles
- `prd-executable-spec/references/requirement-json-schema.md` — Canonical JSON schema for requirement point objects and quick-reference tables for the 4-layer precision model

## Adding a New Skill

1. Create a new directory named after the skill (use kebab-case).
2. Add `SKILL.md` with the required frontmatter and full instructions.
3. Add a `references/` subdirectory for any supporting documents the skill will `Read` at runtime.

The `skill-creator` skill (`/skill-creator`) can guide the creation process if needed.
