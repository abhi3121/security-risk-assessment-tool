# Spec-Driven Development (Cline rule)

This repo uses the **`skill-spec-driven-dev`** agent skill for spec-driven,
test-first development.

- Skill root: `<SKILL_PATH>`
- Entry point: `<SKILL_PATH>/SKILL.md`
- Always-on rules: `<SKILL_PATH>/rules-core.md`

At the start of any task that involves designing, specing, testing, or
implementing a feature, you MUST:

1. Read `<SKILL_PATH>/SKILL.md`.
2. Read `<SKILL_PATH>/rules-core.md`.
3. Identify the requested capability (from a slash command such as
   `/spec-design`, or from the user's natural-language request).
4. Load the matching `<SKILL_PATH>/capabilities/<capability>.md` and follow
   it exactly.
5. Before acting on an existing feature, read its
   `<workspace>/status.json` (default workspace: `.specs/<FEATURE_KEY>/`).
6. Never advance `status.json.stage` unless the capability's gates passed.
7. Never auto-commit. Leave the working tree dirty for human review.

This rule supplements any other `.clinerules/*` files (for example a wiki
rule); it does not replace them.
