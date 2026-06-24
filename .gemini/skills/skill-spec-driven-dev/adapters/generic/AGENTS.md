# AGENTS.md

> Generic entry point for coding agents that honour `AGENTS.md` at the repo
> root (Aider, Codex CLI, OpenAI Agents SDK, custom runners, and any agent
> without a dedicated rules file).

## Spec-Driven Development skill

This repo uses the `skill-spec-driven-dev` agent skill for design, test-first
development, and implementation.

- Skill root: `<SKILL_PATH>`
- Entry point: `<SKILL_PATH>/SKILL.md`
- Always-on rules: `<SKILL_PATH>/rules-core.md`

When the user asks to design, spec, create tests for, or implement a feature:

1. Read `<SKILL_PATH>/SKILL.md` and `<SKILL_PATH>/rules-core.md`.
2. Pick the matching capability from `<SKILL_PATH>/capabilities/`:
   - `init`      — gather inputs, pick workspace, detect toolchain
   - `design`    — LLD + acceptance criteria + implementation plan
   - `tests`     — TDD test spec + failing test skeletons
   - `implement` — iterate tasks until tests go green
   - `lint`      — traceability / coverage check
   - `pipeline`  — run init → design → tests → implement with checkpoints
3. Follow the capability file exactly.
4. Before advancing any stage, read the feature workspace's `status.json`
   (default location: `.specs/<FEATURE_KEY>/status.json`).
5. Never auto-commit. Leave the working tree dirty for human review.

## Aider-specific hint

At session start, run:

```
/read <SKILL_PATH>/SKILL.md
/read <SKILL_PATH>/rules-core.md
```

Then run the capability file you want, e.g.:

```
/read <SKILL_PATH>/capabilities/design.md
"Apply the design capability to feature LUNA-1234."
```
