# spector

This is Spector, a trimmed fork of the agent-skills project, kept down to the skills that generate or maintain specs.

> **Scope:** This file configures agents working on the [`HarounDominique/spector`](https://github.com/HarounDominique/spector) repository itself, not other projects. Don't copy it into another project or a global agent configuration; the reusable assets are the skills in `skills/`.

## Project Structure

```
skills/               → spec-driven-development, documentation-and-adrs (SKILL.md per directory)
.claude/commands/     → /spec, /spec-sync (Claude Code)
.gemini/commands/     → /spec, /spec-sync (Gemini CLI)
commands/             → /spec, /spec-sync (Antigravity CLI)
```

## Skills

- **spec-driven-development** — writes a structured spec before code; for multi-spec projects, also owns the nexus spec (`SPEC-NEXUS.md`: tech foundations, module status, blockers) and the sync protocol that keeps cross-spec citations and status current as modules change
- **documentation-and-adrs** — records decisions and documentation, keeps the spec's context alive after it ships

## Conventions

- Every skill lives in `skills/<name>/SKILL.md`
- YAML frontmatter with `name` and `description` fields
- Description starts with what the skill does (third person), followed by trigger conditions ("Use when...")
- Every skill has: Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification
- Supporting files only created when content exceeds 100 lines

## Contributing

Only add a skill here if it generates or maintains specs/decision records. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Boundaries

- Always: Follow the same anatomy (Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification) for any skill edit
- Never: Add skills that are vague advice instead of actionable processes
- Never: Reintroduce content from the full upstream pack (other skills, agent personas, reference checklists, hooks, eval tooling) without a deliberate decision to do so — this fork exists to stay narrow
- Never: Duplicate content between skills — reference other skills instead
