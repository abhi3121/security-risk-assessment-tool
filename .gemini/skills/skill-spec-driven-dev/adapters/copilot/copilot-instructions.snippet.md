<!-- Append this section to .github/copilot-instructions.md -->

## Spec-Driven Development skill

This repo uses the `skill-spec-driven-dev` agent skill.

- Skill root: `<SKILL_PATH>`
- Entry point: `<SKILL_PATH>/SKILL.md`
- Always-on rules: `<SKILL_PATH>/rules-core.md`

When the user asks to design, spec, write tests for, or implement a feature:

1. Read `<SKILL_PATH>/SKILL.md` and `<SKILL_PATH>/rules-core.md`.
2. Pick the right capability from `<SKILL_PATH>/capabilities/`.
3. Follow the capability file exactly and update artifacts under
   `.specs/<FEATURE_KEY>/`.
4. Never auto-commit.
