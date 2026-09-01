---
description: Propagate a module spec change across a multi-spec project — update the nexus, fix stale cross-references, recompute readiness
---

Invoke the spector:spec-driven-development skill's sync protocol (Phase 0, "Keeping a Multi-Spec Project in Sync").

If the user named a module, use that. Otherwise ask which module spec just changed (or detect it from the most recently modified `SPEC-*.md` other than `SPEC-NEXUS.md`).

Run the protocol in order:
1. Update that module's row in `SPEC-NEXUS.md` (status, `Blocked by`) to match its current spec.
2. Search the project for the module id across every `SPEC-*.md` to find every spec — and the nexus — that cites it.
3. Re-resolve each citation found (`SPEC-<id>.md#<heading>`); flag or fix any that now point at a renamed or removed heading, or a contract that changed meaning.
4. If the change alters a public interface at a module boundary, propagate it into every dependent module's spec, or mark that dependent `blocked` on this module if it can't be updated now.
5. Recompute status for any module whose blockers just cleared.
6. Append one line to the nexus Change Log: date, module id, what changed, what was propagated.

Report what was updated and flag anything that needs human review before confirming the sync is done.
