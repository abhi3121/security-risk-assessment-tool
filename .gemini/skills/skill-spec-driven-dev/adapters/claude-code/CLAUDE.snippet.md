<!-- Append this section to your repo's CLAUDE.md -->

## Spec-Driven Development skill

This repo uses the `skill-spec-driven-dev` agent skill for feature design,
TDD test generation, and implementation.

- Skill root: `<SKILL_PATH>`
- Entry point: `<SKILL_PATH>/SKILL.md`
- Always-on rules: `<SKILL_PATH>/rules-core.md`
- Feature workspaces live under `.specs/<FEATURE_KEY>/` by default.

Before any spec-driven work:

1. Read `<SKILL_PATH>/SKILL.md` and `<SKILL_PATH>/rules-core.md`.
2. Identify the capability (from `/spec-*` slash command or user intent).
3. Load `<SKILL_PATH>/capabilities/<capability>.md` and follow it exactly.
4. Read `status.json` in the feature workspace before advancing any stage.
5. Never auto-commit; leave the working tree dirty for human review.
