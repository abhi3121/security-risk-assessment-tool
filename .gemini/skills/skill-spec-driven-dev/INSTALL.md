# Installing the `skill-spec-driven-dev` skill on each agent

The skill is a self-contained directory (this one). Installation per agent is
just: (1) make the skill reachable from the host repo, and (2) drop in a tiny
rules/commands shim that tells the agent to use it.

Throughout this document, **`<SKILL_PATH>`** is the absolute or repo-relative
path to this skill's root folder, for example:

- Absolute: `C:/Users/<you>/src/ai-powered-engineering/ai-toolkit/skills/skill-spec-driven-dev`
- Linked into the repo: `tools/skills/skill-spec-driven-dev` (a git submodule
  or a plain copy)
- Symlinked into the repo: `.skills/skill-spec-driven-dev`

All the adapter files under `adapters/` use the literal token `<SKILL_PATH>`
— replace with your chosen path when you copy them.

> **Always-on payload:** Agents only need to keep `SKILL.md` and
> `rules-core.md` loaded throughout a session. Capability files
> (`capabilities/<name>.md`) and `references/*.md` are pulled on demand —
> never preload them.

---

## Cline (VS Code)

Cline auto-loads anything in `.clinerules/`.

1. Copy `adapters/cline/spec-driven-rules.md` into the repo at
   `.clinerules/spec-driven-rules.md`. Replace `<SKILL_PATH>`.
2. *(Optional)* Copy `adapters/cline/workflows/*` into
   `.clinerules/workflows/`. Each file is a 2–3-line slash-command shim
   (`/spec-init`, `/spec-design`, `/spec-tests`, `/spec-implement`,
   `/spec-lint`, `/spec-pipeline`). Replace `<SKILL_PATH>`.

Usage:

```
/spec-init
/spec-design
/spec-tests
/spec-implement
```

Or natural language: *"Use the skill-spec-driven-dev skill to implement LUNA-1234."*

---

## Claude Code (terminal)

Claude Code reads `CLAUDE.md` at repo root and slash commands from
`.claude/commands/`. It also has native support for skills under
`.claude/skills/`.

Option A — native skill (preferred):

1. Place this skill directory under `.claude/skills/skill-spec-driven-dev/` in your
   repo (or symlink it there). Claude Code auto-discovers skills in that
   folder.

Option B — rules-based (works anywhere):

1. Append the snippet in `adapters/claude-code/CLAUDE.snippet.md` to the
   repo's `CLAUDE.md`. Replace `<SKILL_PATH>`.
2. Copy the files under `adapters/claude-code/commands/` to
   `.claude/commands/`. Replace `<SKILL_PATH>`.

Usage:

```
/spec-init
/spec-design
/spec-tests
/spec-implement
```

---

## Cursor

1. Copy `adapters/cursor/spec-driven.mdc` to `.cursor/rules/spec-driven.mdc` in the repo.
   Replace `<SKILL_PATH>`.

Usage: natural language. *"Run the skill-spec-driven-dev `design` capability for
LUNA-1234."*

---

## Copilot Chat (GitHub / VS Code)

1. Append the snippet in `adapters/copilot/copilot-instructions.snippet.md`
   to `.github/copilot-instructions.md` (create it if missing). Replace
   `<SKILL_PATH>`.

Usage: natural language. Copilot will follow the pointer and use the skill.

---

## Aider / Codex CLI / OpenAI Agents SDK / generic agents

These agents honour a top-level `AGENTS.md` file.

1. Copy `adapters/generic/AGENTS.md` to the repo root. If an `AGENTS.md`
   already exists, merge the `## Spec-Driven Development` section into it.
   Replace `<SKILL_PATH>`.

Usage: natural language. *"Follow `<SKILL_PATH>/capabilities/design.md` for
LUNA-1234."*

For Aider specifically, pin the skill files with `/read <SKILL_PATH>/SKILL.md`
and `/read <SKILL_PATH>/rules-core.md` at the start of your session, then ask
for the capability you want.

---

## Plain chat (ChatGPT / Claude web / Gemini / …)

When you don't have an agent integrated with the filesystem:

1. Open a new chat.
2. Paste `SKILL.md` and `rules-core.md`, plus the
   `capabilities/<stage>.md` file you want to run. Only add a `references/*`
   file when the capability explicitly points at it (this keeps the prompt
   small).
3. Also paste the relevant inputs (Jira ticket text, PM doc, HLD) inline.
4. Ask the model to produce the artifacts as code blocks; copy them into
   your repo under `.specs/<FEATURE_KEY>/`.

This is less automated but produces the same artifact layout and IDs as the
tool-driven path — so you can switch back to a full agent later without
redoing anything.

---

## Keeping adapters in sync

All adapter files under `adapters/` are 3–6 lines of shim pointing at
`<SKILL_PATH>`. The actual prompts live in `SKILL.md`, `rules-core.md`,
`rules-detail.md`, the `capabilities/*.md` files, and `references/*.md`. If
you edit the skill, no adapter changes are needed (the shims just reference
paths).

## Verifying the install

Ask the agent:

> "What skill-spec-driven-dev capabilities are available? What does `status.json`
> contain before any feature is started?"

A healthy install produces:
- The list of six capabilities (`init`, `design`, `tests`, `implement`,
  `lint`, `pipeline`).
- The `status.json` schema from `references/status-schema.md` or a reference
  to it.

If the agent instead invents its own SDD process, the shim isn't wired up —
check paths in the adapter file you copied.
