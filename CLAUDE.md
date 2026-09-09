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

- **prd-executable-spec** — Transforms PRDs into engineering-grade Executable Specs using a 6-layer precision model (ISO/IEC/IEEE 29148 completeness, BDD Gherkin behavior, DMN decision tables, pseudocode/math logic, JSON Schema/OpenAPI data, verifiable NFRs). Outputs `prd_spec_report.md`, `prd_spec.json`, `assumptions.md`, `.feature` files, `.dmn` files, and optional test-step stubs. Enforces a **tiered gate**: a module's Phase 4 output is blocked only while it has open `blocker`/`major` issues; `minor` issues proceed under a recorded Assumption (default-and-proceed), enabling partial handoff. Supports a **Delta mode** for re-analyzing changed PRDs and flagging stale downstream artifacts. `prd_spec.json` is validatable against a real JSON Schema for CI enforcement.

### prd-executable-spec key files
- `prd-executable-spec/SKILL.md` — Full workflow, phase definitions, tiered gate, Delta mode, output formats, and key principles
- `prd-executable-spec/references/requirement-json-schema.md` — Canonical field schema for the top-level doc, requirement points, assumptions, and change log
- `prd-executable-spec/references/nfr-verification.md` — Verifiable NFR layer (measurable target + verification method)
- `prd-executable-spec/references/consistency-lint.md` — Cross-artifact consistency linter (field/enum/DMN/glossary/dependency coherence)
- `prd-executable-spec/schemas/prd_spec.schema.json` — Machine-validatable JSON Schema for `prd_spec.json` (CI gate enforcement)
- `prd-executable-spec/schemas/examples/prd_spec.example.json` — Valid example instance / fixture

- **3d-character-generation** — Produces a Pixar/"Up"-style Q-version 3D animated character as a transparent-background (alpha channel) video asset. Five-step pipeline: choose concept → generate a 1:1 1K white-background image → animate to a short idle clip → key out the white background and export an alpha video → compress with `ffmpeg` preserving alpha. Emphasizes that only alpha-capable formats (ProRes 4444, WebM VP9 alpha, HEVC alpha, PNG sequence) keep transparency, and documents the tested VP9-alpha command plus the correct way to verify alpha survived (WebM: `alpha_mode=1` tag + browser preview, not `pix_fmt`).

### 3d-character-generation key files
- `3d-character-generation/SKILL.md` — Full 5-step pipeline, prompts, and key principles
- `3d-character-generation/references/prompts.md` — Image + image-to-video prompt templates
- `3d-character-generation/references/transparency-and-compression.md` — Keying (DaVinci/CapCut), alpha export formats, `ffmpeg` compression commands, and format-specific alpha verification
- `3d-character-generation/examples/README.md` — Orange mascot example run + how to add the binary asset (Git LFS)

## Adding a New Skill

1. Create a new directory named after the skill (use kebab-case).
2. Add `SKILL.md` with the required frontmatter and full instructions.
3. Add a `references/` subdirectory for any supporting documents the skill will `Read` at runtime.

The `skill-creator` skill (`/skill-creator`) can guide the creation process if needed.
