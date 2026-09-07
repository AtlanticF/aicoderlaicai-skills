# AI Coder LaiCai Skills

Custom skills for the AI Coder LaiCai workflow.

This repository is intended to host reusable, task-focused skills that can be installed and invoked by an agent runtime. Each skill lives in its own directory and uses a `SKILL.md` file as the entry point.

## Install

Use `npx skills add` to install skills from this repository:

```bash
npx skills add https://github.com/AtlanticF/aicoderlaicai-skills <skillname>
```

## Included Skills

### `prd-executable-spec`

Transforms PRDs into engineering-grade executable specifications. It is designed for requirement review, ambiguity reduction, and structured spec generation using standards and formal artifacts such as:

- ISO/IEC/IEEE 29148 completeness checks
- BDD Gherkin scenarios
- DMN decision tables
- pseudocode and formula-level logic
- JSON Schema / OpenAPI data definitions
- verifiable NFRs (measurable target + verification method)

It also targets real engineering pains beyond single-requirement precision:

- **Severity-tiered gate + partial handoff** — only `blocker`/`major` issues block a module; clean modules ship without waiting for the whole PRD.
- **Assumption register (default-and-proceed)** — `minor` ambiguities get a recorded default instead of stalling the loop; silence = acceptance, product can veto later.
- **Delta mode** — re-analyze a changed PRD, diff it, and flag stale downstream artifacts along the dependency graph.
- **Engineering contracts** — per-point `error_contract` (codes/idempotency/retries), `observability` (logs/metrics/alerts), and `entity_impact` (new vs. modified entities + migration).
- **Prioritization signal** — `complexity`, `risk`, and `intent` on every point.
- **Machine-checkable** — `prd_spec.json` validates against `prd-executable-spec/schemas/prd_spec.schema.json`; a cross-artifact linter catches field/enum/DMN/glossary drift.
- **Runnable output** — optional failing test-step stubs generated from each `.feature`.

Path: `prd-executable-spec/`

### `3d-character-generation`

Produces a Pixar/"Up"-style Q-version 3D animated character as a **transparent-background (alpha channel) video** ready to drop into an app, website, or product UI. Runs the full pipeline:

- pick a character concept (example: an orange);
- generate a Pixar/Up-style, Q-style image on a white background (1:1, 1K);
- animate it into a short idle "looking around + gentle hand swing" clip;
- key out the white background and export an alpha-channel video (ProRes 4444 / WebM VP9 alpha / PNG sequence);
- compress the transparent video with `ffmpeg` while preserving alpha.

Includes prompt templates and a tested transparency + compression reference (correct VP9-alpha command and how to actually verify alpha survived).

Path: `3d-character-generation/`

## Repository Structure

```text
.
├── README.md
├── CLAUDE.md
└── <skill-name>/
    ├── SKILL.md
    ├── references/
    └── schemas/        # optional: machine-validatable schemas + example fixtures
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

MIT. See `LICENSE`.
