# AI Coder LaiCai Skills

Custom skills for the AI Coder LaiCai workflow.

This repository is intended to host reusable, task-focused skills that can be installed and invoked by an agent runtime. Each skill lives in its own directory and uses a `SKILL.md` file as the entry point.

## Install

Use `npx add skills` to install skills from this repository:

```bash
npx add skills
```

## Included Skills

### `prd-executable-spec`

Transforms PRDs into engineering-grade executable specifications. It is designed for requirement review, ambiguity reduction, and structured spec generation using standards and formal artifacts such as:

- ISO/IEC/IEEE 29148 completeness checks
- BDD Gherkin scenarios
- DMN decision tables
- pseudocode and formula-level logic
- JSON Schema / OpenAPI data definitions

Path: `prd-executable-spec/`

## Repository Structure

```text
.
├── README.md
├── CLAUDE.md
└── <skill-name>/
    ├── SKILL.md
    └── references/
```

## Skill Authoring Conventions

- Use one directory per skill and keep the directory name in kebab-case.
- Add a `SKILL.md` file with YAML frontmatter at the top.
- Keep the `name` field aligned with the directory name.
- Use the `description` field to describe trigger phrases and supported scenarios clearly.
- Put reusable supporting materials under `references/` when needed.

## Example Frontmatter

```yaml
---
name: my-skill
description: Briefly describe what the skill does, when it should trigger, and what inputs it supports.
---
```

## Development Notes

- Keep each skill focused on a single job to improve discoverability and invocation quality.
- Prefer explicit trigger phrases over vague descriptions.
- Treat `SKILL.md` as the source of truth for workflow, scope, and output expectations.

## License

Add a license if you plan to distribute this repository publicly.
