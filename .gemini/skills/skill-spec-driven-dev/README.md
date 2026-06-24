---
name: skill-spec-driven-dev
type: skill
phase: development
step: code-generation
product: all
description: A drop-in agentic skill for spec-driven development that transforms specification inputs—such as product requirements, design docs, and feature tickets—into a traceable workflow that generates low-level designs, acceptance criteria, implementation plans, TDD test specs, and production-ready code with any coding agent.
version: 0.2.0
status: stable
installer_managed: true
agent_compat:
  - claude-code
  - cline
  - copilot
  - cursor
  - generic
platform_compat:
  - linux
  - macos
  - windows
requires: []
slug: spec-driven
---

# skill-spec-driven-dev — Agent Skill

A drop-in agentic skill for **spec-driven development** with any coding agent
(Cline, Claude Code, Cursor, Aider, Codex CLI, Copilot, plain chat, …).

The skill turns whatever specification inputs you have — product requirements,
high-level design, feature tickets (Jira / GitHub / GitLab), architecture
docs, an optional in-repo knowledge base — into:

1. a **low-level design** (LLD),
2. **acceptance criteria**,
3. an **implementation plan** with ordered tasks,
4. a **TDD test specification** with failing test skeletons (RED state), and
5. a **production-ready implementation** (tests go GREEN; nothing is
   auto-committed).

Requirements → acceptance criteria → tests → tasks → code are all linked by a
**single, append-only traceability matrix**.

---

## Contents

```
skill-spec-driven-dev/
├── SKILL.md                           ← entry point (router; ~400 tokens)
├── rules-core.md                      ← always-on rules (~800 tokens)
├── rules-detail.md                    ← detailed rules, load on demand
├── README.md                          ← you are here (human-only)
├── INSTALL.md                         ← per-agent install (human-only)
├── capabilities/                      ← one file per capability (stage)
│   ├── init.md
│   ├── design.md
│   ├── tests.md
│   ├── implement.md
│   ├── lint.md
│   └── pipeline.md
├── references/                        ← load-on-demand reference material
│   ├── frameworks.md                  ←   build/test framework detection + skeletons
│   ├── artifacts.md                   ←   frontmatter & artifact contract
│   ├── status-schema.md               ←   full status.json schema + example
│   ├── self-validation.md             ←   self-validation protocol
│   └── risk-categories.md             ←   risk / edge-case / NFR checklists
├── templates/                         ← artifact templates
│   ├── inputs.yaml
│   ├── status.json
│   ├── lld.md
│   ├── acceptance.md
│   ├── plan.md
│   ├── risks.md
│   ├── tests.md
│   ├── test-matrix.csv
│   ├── traceability.md
│   ├── clarifications.md
│   ├── context-manifest.md
│   ├── impl-log.md
│   ├── verify-report.md
│   └── feature-README.md
├── adapters/                          ← ready-to-copy per-agent shims
│   ├── cline/
│   ├── claude-code/
│   ├── cursor/
│   ├── copilot/
│   └── generic/AGENTS.md
└── examples/
    └── hello-feature/                 ← minimal worked example
```

> **Token economy:** as of `v0.2.0`, the always-on payload (SKILL.md +
> rules-core.md) is ~1.3 k tokens; per-capability calls add ~1.5–2 k. Reference
> files under `references/` are pulled only when a capability step explicitly
> needs a row — they are NOT part of the always-on prompt.

---

## What problem does it solve?

Coding agents are great at writing code, but on real teams that's only ~20%
of the job. The other 80% is:

- extracting real requirements from scattered sources (PM docs, Jira, Slack),
- converting them into a concrete design that fits an existing architecture,
- producing test cases that actually check the intent,
- implementing without regressions,
- keeping a paper trail from requirement to commit.

This skill encodes the 80% as six **capabilities** and a **filesystem state
machine**. The agent just follows it.

---

## Key design properties

- **One skill, many capabilities** — single mental model for users, but the
  agent only loads the capability file it needs (progressive disclosure).
- **Layered prompts** — `rules-core.md` (always-on) → capability file (per
  call) → `references/*.md` (per step that needs it). Keeps the always-on
  payload small and lets the skill grow capabilities cheaply.
- **Agent-neutral** — prompts use generic verbs and on-disk state so any
  agent (or a human) can pick up where another left off.
- **Stable IDs** — `REQ-n`, `NFR-n`, `AC-n`, `LLD-n`, `TASK-n`, `TC-n`,
  `RISK-n`, `Q-n`. Append-only. Never renumbered.
- **Graceful degradation** — missing inputs never cause silent invention;
  they become `Q-n` clarifications, and blockers halt the pipeline.
- **TDD enforced mechanically** — `implement` refuses to run without a
  confirmed RED state recorded in `status.json`.
- **Machine-readable outputs** — every artifact has YAML frontmatter or a
  stable CSV/JSON schema, so downstream agents, CI, dashboards, or scripts
  can consume them without parsing prose.
- **No auto-commit** — the skill always leaves the working tree dirty for
  human review.

---

## Capabilities at a glance

| Capability | Inputs                                    | Outputs                                                       |
|------------|-------------------------------------------|---------------------------------------------------------------|
| `init`     | PM docs, HLD, Jira, architecture docs, wiki (optional) | `inputs.yaml`, `context-manifest.md`, seeded `status.json`, workspace scaffold |
| `design`   | `inputs.yaml`, knowledge base (optional)  | `lld.md`, `acceptance.md`, `plan.md`, `risks.md`, traceability |
| `tests`    | Design artifacts, detected test framework | `tests.md`, `test-matrix.csv`, failing test skeletons (RED)    |
| `implement`| All of the above                          | Source-code changes, `impl-log.md`, `verify-report.md` (GREEN) |
| `lint`     | Any stage ≥ `inputs-ready`                | `lint-report-YYYY-MM-DD.md`                                    |
| `pipeline` | —                                         | Runs `init → design → tests → implement` with human checkpoints |

See `SKILL.md` for the capability router, `rules-core.md` for the always-on
rules, `rules-detail.md` for the deep-dive, and each file under
`capabilities/` for the full per-stage prompt.

---

## Where does the work live?

Each feature has its own workspace. The `init` capability prompts for the
feature key (Jira ticket id preferred, kebab slug fallback) and the workspace
path (default: `<repo-root>/.specs/<FEATURE_KEY>/`).

```
<workspace>/
├── 00-inputs/
├── 01-design/
├── 02-tests/
├── 03-impl/
├── traceability.md
├── clarifications.md
├── status.json
└── README.md
```

---

## Installation

See [`INSTALL.md`](INSTALL.md) for one-page instructions per agent:

- **Claude Code** — `.claude/skills/` or `CLAUDE.md` reference
- **Cline** — `.clinerules/` + optional slash-command shims
- **Cursor** — `.cursor/rules/spec-driven.mdc`
- **Aider / Codex CLI / generic agents** — `AGENTS.md`
- **Copilot Chat** — `.github/copilot-instructions.md`
- **Plain chat (ChatGPT / Claude web)** — attach `SKILL.md` + `rules-core.md`
  + the capability file you need

All shims live under `adapters/` and are 3–6 lines each.

---

## Host-repo integrations (all optional)

The skill works on any repo. If any of the following are present, it uses
them:

- A **knowledge base** (e.g. `.wiki/`, `wiki/`, `knowledge/`, `memory-bank/`)
  with or without a schema.
- A **formal-docs** folder (`docs/`, `design/`, per-module `docs/`).
- A **ticketing MCP** (Jira / Confluence / GitHub / GitLab).
- Any build system or test framework (full detection table in
  `references/frameworks.md`).

If none are present, the skill falls back to reading source code directly and
prompting the user for anything it cannot infer.

---

## Quick start

On a repo set up for the skill, just ask the agent:

> "Use the `skill-spec-driven-dev` skill to design and implement `LUNA-1234`."

or, with an agent that supports slash commands:

```
/spec-init
/spec-design
/spec-tests
/spec-implement
/spec-lint
```

---

## License / status

Version `0.2.0` — structural refactor for token economy (no behavioural
change). Changelog in `rules-detail.md` §changelog. MIT-style licence; reuse
freely inside your organisation.
