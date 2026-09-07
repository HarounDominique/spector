# Spector

**Spec-driven development skills for AI coding agents.**

A trimmed fork of [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) kept down to the skills that generate and maintain specs: writing a structured spec before code, recording the decisions and documentation that keep it accurate afterward, and auditing the corpus for drift between what it claims and what's actually true.

## Commands

| What you're doing | Command |
|-------------------|---------|
| Write a spec before coding | `/spec` |
| Keep a multi-spec project in sync after editing one module's spec | `/spec-sync` |

Skills also activate automatically based on what you're doing — starting a project with no spec triggers `spec-driven-development`, making an architectural decision or shipping a feature triggers `documentation-and-adrs`, and asking for an audit of the spec corpus (or finishing an edit that moved specs or decision records) triggers `spec-corpus-hygiene`.

## Multi-Spec Projects

A single `/spec` run produces one spec for one iteration. Large or continuously developed projects — several screens, views, menus, or modules — need more than that: a **nexus spec** (`SPEC-NEXUS.md`) that holds the shared tech stack, design patterns, and versions, plus a status table tracking which module specs are `ready`, `blocked`, `in-progress`, or `done`, and what blocks each one.

Because these specs cite each other, they rot as the project evolves — headings move, contracts change, a blocker clears without anyone updating the table. `/spec-sync` runs the propagation protocol after any module spec changes: it updates the nexus, re-resolves every cross-spec citation (module id + heading, never a line number — line numbers rot on the next unrelated edit), propagates interface changes to dependents, and recomputes readiness. See [spec-driven-development §Phase 0](skills/spec-driven-development/SKILL.md) for the full protocol.

## Quick Start

**Claude Code:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

Skills are plain Markdown — they work with any agent that accepts system prompts or instruction files. Copy `skills/<name>/SKILL.md` into whatever mechanism your agent uses to load instructions.

## The Skills

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Write a structured spec covering objectives, commands, structure, code style, testing, and boundaries before any code. Four gated phases (Specify → Plan → Tasks → Implement), a nexus-spec step for requests that bundle several independently testable capabilities, and a sync protocol that keeps a multi-spec project's cross-references and status table accurate as it evolves. | Starting a new project, feature, or significant change with no spec yet; keeping a multi-spec project in sync after a module changes |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | Architecture Decision Records, API docs, inline documentation standards, README structure, changelog maintenance — document the *why*, not just the *what* | Making an architectural decision, changing a public API, or shipping a feature |
| [spec-corpus-hygiene](skills/spec-corpus-hygiene/SKILL.md) | Audits a spec corpus for internal truth: citations that still resolve, derived figures that agree with their source, and claims about built state that match the code. Mechanical checks first, then parallel read-only audit lanes, a fix-or-register triage, and a severity threshold so the loop converges instead of chaining forever. | Asked to audit or sanity-check the spec corpus; after any edit that moved specs, decision records, or a nexus status table |

## How Skills Work

Every skill follows a consistent anatomy:

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: lowercase-hyphen-name               │  │
│  │ description: Guides agents through [task].│  │
│  │              Use when…                    │  │
│  └───────────────────────────────────────────┘  │
│  Overview         → What this skill does        │
│  When to Use      → Triggering conditions       │
│  Process          → Step-by-step workflow       │
│  Rationalizations → Excuses + rebuttals         │
│  Red Flags        → Signs something's wrong     │
│  Verification     → Evidence requirements       │
└─────────────────────────────────────────────────┘
```

## Project Structure

```
agent-skills/
├── skills/
│   ├── spec-driven-development/   # Write the spec
│   ├── documentation-and-adrs/    # Keep it (and decisions) documented
│   └── spec-corpus-hygiene/       # Audit the corpus for drift from the truth
├── .claude/commands/               # /spec, /spec-sync (Claude Code)
├── .gemini/commands/               # /spec, /spec-sync (Gemini CLI)
├── commands/                       # /spec, /spec-sync (Antigravity CLI)
└── plugin.json                    # Plugin manifest
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT
