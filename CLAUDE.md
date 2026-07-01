# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Claude Code plugin** (`management-consultant`, see `.claude-plugin/plugin.json`), not an application. It bundles 22 markdown-defined skills that give Claude an MBB-style (McKinsey/BCG/Bain) management-consulting operating mode. There is no build, lint, or test tooling — the entire "codebase" is a set of `SKILL.md` prompt files under `skills/`.

## Repository Structure

- `.claude-plugin/plugin.json` — plugin manifest (name, version, description, keywords). Bump `version` when skills are added/changed meaningfully.
- `skills/<skill-name>/SKILL.md` — one skill per directory, each a self-contained markdown file with YAML frontmatter (`name`, `description`) followed by the skill's instructions. There are no other files per skill (no scripts, no assets).

## Architecture: Orchestrator + Specialists

`skills/strategy-engagement/SKILL.md` is the **orchestrator**. It is the entry point for any fuzzy business/strategy question and is responsible for:
1. Framing the real decision (not just answering the literal question).
2. Routing to the minimal set of specialist skills via the routing map in that file, then invoking them with the `Skill` tool rather than reimplementing their logic inline.
3. Synthesizing sub-skill outputs into one answer-first recommendation instead of stapling outputs together.

The other 21 skills are **specialists**, each owning one MBB method and grouped into five phases (mirrored in the orchestrator's routing map):
- **Diagnose & Frame**: `situation-assessment`, `growth-barriers`, `assumption-audit`
- **Market & Competitive Intelligence**: `market-mapping`, `customer-segmentation`, `competitive-intel`, `profit-pool-analysis`
- **Strategic Choice & Economics**: `strategic-options`, `business-case-builder`, `pricing-strategy`, `portfolio-review`
- **Operating Model & Execution**: `operating-model-design`, `initiative-prioritizer`, `transformation-roadmap`
- **Risk, Performance & Value Governance**: `risk-and-mitigation`, `war-gaming`, `kpi-architect`, `value-realization`
- **Align & Communicate**: `narrative-builder`, `stakeholder-alignment`, `decision-memo`

When editing the routing map or adding a new specialist skill, keep `strategy-engagement/SKILL.md`'s routing map and "Common Engagement Patterns" section in sync with the actual skill set.

## Specialist Skill Format

Every specialist skill (all except `strategy-engagement`) follows the same internal shape — preserve this when adding or editing a skill:

```markdown
---
name: <kebab-case-skill-name>
description: <one paragraph: what it does + "Use when/for ..." trigger phrases>
---

# <Skill Title>

## When To Use
## McKinsey-Style Approach
## Workflow
## Output Format   (a fenced ```markdown``` template the skill should fill in)
## Quality Bar      (bullet list of non-negotiables, e.g. "lead with the recommendation")
```

- `description` frontmatter is what Claude uses to decide when to auto-invoke the skill — write it as trigger phrases a user would actually say, not a generic summary.
- `Output Format` should be a literal markdown template with the section headers the skill's output must have, not prose describing the output.
- `Quality Bar` items are terse imperatives, not explanations.

## Working in This Repository

- There is no runtime to build, lint, or test — changes are validated by reading the skill prompts for MECE structure, correct routing, and consistency with the format above.
- Keep skill names and directory names identical and kebab-case; the `name` frontmatter field must match the directory name under `skills/`.
