---
description: Start spec-driven development — write a structured specification before writing code
---

Invoke the spector:spec-driven-development skill.

Begin by understanding what the user wants to build. Ask clarifying questions about:
1. The objective and target users
2. Core features and acceptance criteria
3. Tech stack preferences and constraints
4. Known boundaries (what to always do, ask first about, and never do)

Then generate a structured spec covering all six core areas: objective, commands, project structure, code style, testing strategy, and boundaries.

If the request bundles several independently testable capabilities — or several distinct screens, views, or menus — first propose a nexus spec (module ids, dependency direction, build order, shared tech foundations) per the skill's Phase 0 and get it approved, then spec each module in dependency order, citing other modules by id + heading, never by line number.

Save a single-capability spec as SPEC.md in the project root. Save a multi-spec project as SPEC-NEXUS.md at the root plus one SPEC-<module-id>.md per module. Confirm with the user before proceeding. Once more than one spec exists, use `/spec-sync` after every module edit to keep the set consistent.
