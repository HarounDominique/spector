# Contributing to Spector

This is Spector, a trimmed fork kept down to the two skills that generate or maintain specs: `spec-driven-development` and `documentation-and-adrs`.

## Scope

Only propose a new skill, or significant rework of an existing one, if it generates a spec or keeps one (or its decision records) accurate over time. Anything else belongs in the full upstream pack, not here.

### Creating a skill

1. Create a directory under `skills/` with a kebab-case name
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`)
3. The `description` starts with what the skill does (third person), then one or more `Use when` trigger conditions
4. Follow the standard anatomy: Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification

### Skill Quality Bar

Skills should be:

- **Specific** — Actionable steps, not vague advice
- **Verifiable** — Clear exit criteria with evidence requirements
- **Battle-tested** — Based on real engineering workflows, not theoretical ideals
- **Minimal** — Only the content needed to guide the agent correctly

### What Not to Do

- Don't duplicate content between skills — reference other skills instead
- Don't add skills that are vague advice instead of actionable processes
- Don't create supporting files unless content exceeds 100 lines

## Modifying Existing Skills

- Keep changes focused and minimal
- Preserve the existing structure and tone
- Test that YAML frontmatter remains valid after edits

## Repo-scoped files

`AGENTS.md` and `CLAUDE.md` at the repo root configure agents working on this repository itself. Don't instruct users to copy them into their own projects or a global agent configuration; the reusable assets are the skills in `skills/`.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
