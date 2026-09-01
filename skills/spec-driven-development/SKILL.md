---
name: spec-driven-development
description: Creates specs before coding, and keeps a multi-spec project in sync as it grows. Use when starting a new project, feature, or significant change and no specification exists yet. Use when requirements are unclear, ambiguous, or only exist as a vague idea. Use when a single requirement spans several independently testable capabilities and needs decomposing into a nexus spec of modules before specifying. Use when a module spec changes and dependent specs, cross-references, or a nexus status table need to stay accurate.
---

# Spec-Driven Development

## Overview

Write a structured specification before writing any code. The spec is the shared source of truth between you and the human engineer — it defines what we're building, why, and how we'll know it's done. Code without a spec is guessing.

## When to Use

- Starting a new project or feature
- Requirements are ambiguous or incomplete
- The change touches multiple files or modules
- You're about to make an architectural decision
- The task would take more than 30 minutes to implement

**When NOT to use:** Single-line fixes, typo corrections, or changes where requirements are unambiguous and self-contained.

## The Gated Workflow

Spec-driven development has four phases, preceded by a scope check (Phase 0) that activates only when one request bundles several independently testable capabilities. Do not advance to the next phase until the current one is validated.

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 Human      Human    Human      Human
 reviews    reviews  reviews    reviews
```

### Phase 0: Scope Check

Most requests describe one capability. If this one does, skip this phase and go straight to Specify — Phase 0 exists for the exception, not the rule, and it puts no hierarchy on single-capability features.

**Detection.** Decompose before specifying when a single requirement bundles several independently testable capabilities:

- The requirement names distinct capabilities with their own consumers or data (e.g. identity, billing, notifications, reporting) — or distinct screens, views, or menus in a UI-heavy project
- Acceptance criteria cluster into groups that could ship and be verified separately
- One capability could be cut or replaced without rewriting the others' requirements

**Propose a nexus spec before writing any module spec.** One project has exactly one nexus spec, `SPEC-NEXUS.md`, at the project root. Unlike a module spec it is never "done" — it is a living index, reviewed and updated for as long as the project has more than one spec. Small and reviewable at proposal time — a module table plus a build order, not a project plan:

```markdown
# Nexus: [Initiative Name]

## Tech Foundations
[Stack, language/runtime versions, key dependencies, and cross-cutting design
patterns (e.g. error handling, auth, state management) that every module spec
inherits instead of re-declaring. A module spec overrides this only when it
has a documented reason to diverge.]

## Modules

| Module id | Spec file | Responsibility | Depends on | Status | Blocked by |
|---|---|---|---|---|---|
| identity | SPEC-identity.md | Accounts, sessions, SSO | — | ready | — |
| billing | SPEC-billing.md | Plans, invoices, payments | identity | blocked | identity |
| notifications | SPEC-notifications.md | Email and webhook fan-out | identity | blocked | identity |
| reporting | SPEC-reporting.md | Usage dashboards | billing, notifications | draft | — |

Build order: identity → billing, notifications → reporting

## Change Log
[Append-only. One line per sync: date, module id, what changed, what was propagated.]
```

- **Stable module ids.** Kebab-case, chosen once, never renamed mid-initiative. Specs, plans, downstream commands, and cross-spec citations select work by these ids instead of guessing which spec is active.
- **Dependency direction, no cycles.** Arrows point one way. If two modules each need the other, they are one module.
- **Interfaces live at the boundary.** The table records that `billing` depends on `identity`; the contract between them belongs in the provider module's spec, cited from the nexus by id and heading (see Citation Convention below), never copied into the nexus.
- **Status is not aspirational.** `draft` (spec not yet written or not yet reviewed), `ready` (spec approved, no unresolved blockers, safe to start Plan/Tasks/Implement), `blocked` (waiting on another module's interface or decision — name it in `Blocked by`), `in-progress`, `done`. A module moves to `ready` only when everything in its `Blocked by` column is `done`.
- **The nexus is gated like every phase.** The human reviews module boundaries, dependency direction, tech foundations, and build order before any module spec is written. Getting the map wrong is expensive; reviewing ten lines is not.

**Then recurse per module.** Run Specify → Plan → Tasks → Implement for each module in dependency order. Each module gets its own spec, scoped to that module's objective, boundaries, and success criteria, and inheriting Tech Foundations from the nexus instead of restating them. Save the nexus at the project root and each module's spec alongside it, named by module id (`SPEC-identity.md`, `SPEC-billing.md`) — the nexus, not filename guessing, is the index of what exists and what state it's in.

### Citation Convention

Specs cite each other by **module id + section heading**, never by line number or paragraph position — line numbers shift on every edit and rot silently. Use a stable anchor form: `SPEC-billing.md#pricing-rules` (the module id from the nexus table, plus the target heading slug), not `SPEC-billing.md:42`. If a heading is renamed, the citing spec breaks loudly at the next sync pass (Phase 0 heading no longer resolves) instead of silently pointing at the wrong paragraph.

### Keeping a Multi-Spec Project in Sync

A nexus spec rots the same way a single spec does, faster: every module edit can invalidate another module's citations, the nexus status table, or the build order. Run this **sync protocol** every time a module spec changes — not just at the end of a phase:

1. **Update the module's own entry.** Status, and Blocked by if the change resolves or introduces a blocker.
2. **Find citers.** Search the project for the module id (`grep -rl '<module-id>'` across `SPEC-*.md`) to find every spec and the nexus itself that references it.
3. **Re-resolve every citation found.** For each `SPEC-<id>.md#<heading>` reference to the changed spec, confirm the heading still exists and still means what the citer assumed. Fix or flag drift — don't leave a citation pointing at a heading that moved or a contract that changed meaning.
4. **Propagate contract changes.** If the change alters a public interface at a module boundary (per Tech Foundations or the module's own spec), update every dependent module's spec (per the nexus dependency column) or flip it to `blocked` with this module named in `Blocked by` if it can't be updated immediately.
5. **Recompute readiness.** Any module whose `Blocked by` list is now empty and whose dependencies are `done` moves to `ready`.
6. **Append one line to the Change Log.** Date, module id, what changed, what was propagated — so the next sync pass (human or agent) doesn't have to re-derive what already happened.

This protocol is what `/spec-sync` automates end to end; running the steps manually after any module edit has the same effect.

### Phase 1: Specify

Start with a high-level vision. Ask the human clarifying questions until requirements are concrete.

**Surface assumptions immediately.** Before writing any spec content, list what you're assuming:

```
ASSUMPTIONS I'M MAKING:
1. This is a web application (not native mobile)
2. Authentication uses session-based cookies (not JWT)
3. The database is PostgreSQL (based on existing Prisma schema)
4. We're targeting modern browsers only (no IE11)
→ Correct me now or I'll proceed with these.
```

Don't silently fill in ambiguous requirements. The spec's entire purpose is to surface misunderstandings *before* code gets written — assumptions are the most dangerous form of misunderstanding.

**Write a spec document covering these six core areas:**

1. **Objective** — What are we building and why? Who is the user? What does success look like?

2. **Commands** — Full executable commands with flags, not just tool names.
   ```
   Build: npm run build
   Test: npm test -- --coverage
   Lint: npm run lint --fix
   Dev: npm run dev
   ```

3. **Project Structure** — Where source code lives, where tests go, where docs belong.
   ```
   src/           → Application source code
   src/components → React components
   src/lib        → Shared utilities
   tests/         → Unit and integration tests
   e2e/           → End-to-end tests
   docs/          → Documentation
   ```

4. **Code Style** — One real code snippet showing your style beats three paragraphs describing it. Include naming conventions, formatting rules, and examples of good output.

5. **Testing Strategy** — What framework, where tests live, coverage expectations, which test levels for which concerns.

6. **Boundaries** — Three-tier system:
   - **Always do:** Run tests before commits, follow naming conventions, validate inputs
   - **Ask first:** Database schema changes, adding dependencies, changing CI config
   - **Never do:** Commit secrets, edit vendor directories, remove failing tests without approval

**Spec template:**

```markdown
# Spec: [Project/Feature Name]

<!-- If this module is part of a multi-spec project, link the nexus and drop
     the sections below that the nexus already covers instead of restating them:
     Nexus: SPEC-NEXUS.md | Module id: <id> -->

## Objective
[What we're building and why. User stories or acceptance criteria.]

## Tech Stack
[Framework, language, key dependencies with versions — omit if inherited from SPEC-NEXUS.md#tech-foundations]

## Commands
[Build, test, lint, dev — full commands]

## Project Structure
[Directory layout with descriptions]

## Code Style
[Example snippet + key conventions]

## Testing Strategy
[Framework, test locations, coverage requirements, test levels]

## Boundaries
- Always: [...]
- Ask first: [...]
- Never: [...]

## Success Criteria
[How we'll know this is done — specific, testable conditions]

## Open Questions
[Anything unresolved that needs human input]
```

**Reframe instructions as success criteria.** When receiving vague requirements, translate them into concrete conditions:

```
REQUIREMENT: "Make the dashboard faster"

REFRAMED SUCCESS CRITERIA:
- Dashboard LCP < 2.5s on 4G connection
- Initial data load completes in < 500ms
- No layout shift during load (CLS < 0.1)
→ Are these the right targets?
```

This lets you loop, retry, and problem-solve toward a clear goal rather than guessing what "faster" means.

### Phase 2: Plan

With the validated spec, generate a technical implementation plan:

1. Identify the major components and their dependencies
2. Determine the implementation order (what must be built first)
3. Note risks and mitigation strategies
4. Identify what can be built in parallel vs. what must be sequential
5. Define verification checkpoints between phases

> **Output convention:** Save the plan to `tasks/plan.md` and record the task list in `tasks/todo.md` (projects may designate an external tracker instead). Create `tasks/` if it does not exist.

The plan should be reviewable: the human should be able to read it and say "yes, that's the right approach" or "no, change X."

### Phase 3: Tasks

Break the plan into discrete, implementable tasks:

- Each task should be completable in a single focused session
- Each task has explicit acceptance criteria
- Each task includes a verification step (test, build, manual check)
- Tasks are ordered by dependency, not by perceived importance
- No task should require changing more than ~5 files

**Task template:**
```markdown
- [ ] Task: [Description]
  - Acceptance: [What must be true when done]
  - Verify: [How to confirm — test command, build, manual check]
  - Files: [Which files will be touched]
```

### Phase 4: Implement

Execute tasks one at a time. Verify each task against its acceptance criteria before moving to the next; don't batch several tasks before checking any of them.

## Keeping the Spec Alive

The spec is a living document, not a one-time artifact:

- **Update when decisions change** — If you discover the data model needs to change, update the spec first, then implement.
- **Update when scope changes** — Features added or cut should be reflected in the spec.
- **Commit the spec** — The spec belongs in version control alongside the code.
- **Reference the spec in PRs** — Link back to the spec section that each PR implements.

If the project has more than one spec, this is not enough on its own — a module spec editing itself into staleness with no propagation is exactly how nexus projects rot. Run the **sync protocol** (see Phase 0, "Keeping a Multi-Spec Project in Sync") after every module edit, not just this checklist.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is simple, I don't need a spec" | Simple tasks don't need *long* specs, but they still need acceptance criteria. A two-line spec is fine. |
| "I'll write the spec after I code it" | That's documentation, not specification. The spec's value is in forcing clarity *before* code. |
| "The spec will slow us down" | A 15-minute spec prevents hours of rework. Waterfall in 15 minutes beats debugging in 15 hours. |
| "Requirements will change anyway" | That's why the spec is a living document. An outdated spec is still better than no spec. |
| "The user knows what they want" | Even clear requests have implicit assumptions. The spec surfaces those assumptions. |
| "It's one big feature; splitting it is overhead" | If acceptance criteria cluster into independently testable groups, a monolithic spec forces every downstream task to reason over the whole contract. A ten-line nexus spec is the cheap alternative. |
| "I'll decompose during planning" | Planning slices tasks within a spec. By then the oversized artifact already exists — module boundaries and dependency direction must be decided before the spec is written, not after. |
| "I only changed one module, the others are fine" | Other specs may cite the one you changed. A citation to a heading that moved or a contract that changed meaning is now silently wrong — run the sync protocol before assuming the blast radius is one file. |
| "I'll fix the cross-references at the end" | "The end" of a continuously developed project never arrives. Stale citations compound; fix them at the edit that caused them, when the diff is one module, not months later across a dozen. |
| "Line numbers are precise, that's better than a heading" | Precise and stable are different things. A line number is invalidated by the next unrelated edit above it; a heading survives until someone deliberately renames it — which is exactly the case you want a sync pass to catch. |

## Red Flags

- Starting to write code without any written requirements
- Asking "should I just start building?" before clarifying what "done" means
- Implementing features not mentioned in any spec or task list
- Making architectural decisions without documenting them
- Skipping the spec because "it's obvious what to build"
- One spec whose requirements span several independently testable capabilities
- Module boundaries or build order decided implicitly during implementation because no nexus spec was approved up front
- A module spec edited without checking who cites it
- A nexus status table that says `blocked` for a dependency that's actually `done`, or `ready` for a module whose blocker never resolved
- Cross-spec references by line number instead of module id + heading
- A nexus Change Log with gaps — edits happened but no sync pass recorded what propagated

## Verification

Before proceeding to implementation, confirm:

- [ ] The spec covers all six core areas
- [ ] The human has reviewed and approved the spec
- [ ] Success criteria are specific and testable
- [ ] Boundaries (Always/Ask First/Never) are defined
- [ ] The spec is saved to a file in the repository
- [ ] If the request bundles several independently testable capabilities, a nexus spec (module ids, dependency direction, build order, tech foundations) was approved before any module spec was written
- [ ] Every module spec traces to a module id in the nexus, and cites other modules by id + heading, never by line number
- [ ] After any module spec edit in a multi-spec project, the sync protocol ran: the module's status/blockers are current, citers were checked, propagated changes and the sync itself are logged in the nexus Change Log
