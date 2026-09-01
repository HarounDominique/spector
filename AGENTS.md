# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, Antigravity, etc.) when working with code in this repository.

> **Scope:** This file configures agents working on the [`HarounDominique/spector`](https://github.com/HarounDominique/spector) repository itself. It is not meant to be copied into other projects or into a global agent configuration; the reusable assets are the skills in `skills/`, not this file.

## Repository Overview

This is Spector, a trimmed fork kept down to the skills that generate or maintain specs: `spec-driven-development` and `documentation-and-adrs`. No other skills, agent personas, reference checklists, hooks, or eval tooling remain.

## Intent → Skill Mapping

- Starting a project or feature with no spec yet → `spec-driven-development` (`/spec`)
- A request bundles several independently testable capabilities (or several screens/views/menus) → `spec-driven-development` Phase 0, propose a nexus spec (`SPEC-NEXUS.md`) before writing any module spec
- A module spec just changed in a multi-spec project → `spec-driven-development`'s sync protocol (`/spec-sync`): update the nexus, re-resolve cross-spec citations, propagate contract changes, recompute readiness
- Making an architectural decision, changing a public API, or shipping a feature → `documentation-and-adrs`

Anything outside spec generation or spec/decision maintenance is out of scope for this repo — implement it directly rather than reaching for a skill that no longer exists here.

## Creating a New Skill

Only add a skill here if it generates or maintains specs/decision records; anything else belongs in a fuller fork. Follow the standard anatomy: `skills/<kebab-case-name>/SKILL.md` with YAML frontmatter (`name`, `description`) and sections Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification. See [CONTRIBUTING.md](CONTRIBUTING.md).
