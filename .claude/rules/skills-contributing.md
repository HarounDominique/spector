---
description: Anti-duplication guardrail for adding or changing skills
paths:
  - "skills/**"
---

# Adding or changing a skill

This fork is kept down to skills that generate or maintain specs: `spec-driven-development` and `documentation-and-adrs`. Before creating a new `skills/<name>/` directory or significantly reworking an existing one:

- Confirm the idea generates a spec or keeps one (or its decision records) accurate over time. Anything else belongs in the full upstream pack, not here.
- Prefer extending an existing skill over adding a near-duplicate.
- Follow the standard anatomy (Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification), and never duplicate content between skills, reference the other skill instead.

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for the full workflow.
