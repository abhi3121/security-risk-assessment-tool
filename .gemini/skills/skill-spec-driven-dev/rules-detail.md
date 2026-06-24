# Spec-Driven Development — Detailed Rules (load on demand)

Detail and reference material that the always-on `rules-core.md` points at.
Do **not** load this file by default — only when a capability step
explicitly references one of its sections.

---

## §traceability — column rules & status enum

`traceability.md` columns (stable order, do not rearrange):

```markdown
| REQ/NFR | AC | LLD | TASK | TC | Commits | Status |
|---------|----|-----|------|----|---------|--------|
| REQ-1   | AC-1 | LLD-3 | TASK-2 | TC-4, TC-5 | <sha> | implemented |
```

Rules:
- Multiple IDs in one cell are comma-separated.
- Empty cell = `—`, never blank.
- Rows append only. A changed relationship gets a new row; the prior row is
  marked `status: superseded`.

`Status` enum (valid values):
- `needs-design` — REQ/NFR captured but no AC yet.
- `designed`     — AC + LLD exist; no TC yet.
- `tested`       — TC exists; not yet implemented.
- `implemented`  — TC green and source change linked.
- `pending`      — commit SHA placeholder (after `implement` but before user commits).
- `superseded`   — relationship replaced by a newer row.
- `retired`      — element removed from scope (keep ID; mark here).

The `lint` capability enforces these invariants.

---

## §clarifications — entry template, anchoring, rollover

Entry template:

```markdown
## Q-1 (blocker | advisory | resolved)
- Raised by: init | design | tests | implement | lint
- Date: YYYY-MM-DD
- Source: inputs.yaml#REQ-3   (or other artifact anchor)
- Question: <clear question>
- Impact: <what is blocked if unresolved>
- Proposed assumption: <what the skill will assume if unresolved, or "none">
- Resolution: <YYYY-MM-DD — who resolved it and how>  (fill when resolved)
```

Source-anchor conventions:
- `<workspace-relative path>#<anchor>` — e.g. `inputs.yaml#REQ-3`,
  `01-design/lld.md#LLD-7`.
- `mcp:jira:<KEY>#description` for ticket-derived sources.
- For free-form sources, cite as plainly as possible.

Rollover:
- Resolved Q-n stay in place (do not delete or renumber).
- After a capability run, regenerate the `open_clarifications` count in
  `status.json` to reflect non-resolved entries.

---

## §host-repo — probing matrix (used by `init`)

`init` probes the host repo and records findings in
`<workspace>/00-inputs/context-manifest.md`. Categories:

- **Knowledge base** — `.wiki/`, `wiki/`, `knowledge/`, `memory-bank/`. If a
  schema/index exists (`schema.md`, `README.md`), record its path.
- **Design-doc folder** — `docs/`, `design/`, `specs/`, per-module `docs/`.
- **Ticketing** — MCP server providing Jira / Confluence / GitHub / GitLab.
- **Build system** — see `references/frameworks.md` §1.
- **Test framework** — see `references/frameworks.md` §2.
- **Agent rules files** — `CLAUDE.md`, `AGENTS.md`, `.clinerules/`,
  `.cursor/`, `.github/copilot-instructions.md`.
- **CI** — `.gitlab-ci.yml`, `.github/workflows/`, `azure-pipelines.yml`,
  `Jenkinsfile`, `.circleci/`, `bitbucket-pipelines.yml`.

The skill MUST work when none of these are present. Missing integrations
become advisory `Q-n` entries or explicit `_not detected_` notes.

---

## §knowledge-base — optional integration

When a knowledge base is detected (e.g. an `.wiki/` built from a schema-driven
llm-wiki pattern):

- Read the knowledge base's `schema.md` / `README.md` first to learn its
  conventions.
- Use its index to identify pages within the feature's scope (target modules,
  concepts, patterns). Read only those pages.
- When `implement` changes behaviour documented in a wiki page, follow the
  wiki's own maintenance rules (update fields, mark confidence, append
  changelog). If the wiki has no schema, just update touched pages and leave
  new content `confidence: medium`.

If no knowledge base is present, skip this entire section. It is NEVER a hard
dependency.

---

## §portability — additional notes

- Capabilities are written so they can be executed by any coding agent. Use
  generic verbs only.
- Do not rely on in-memory state between capability runs. All state is on
  disk under the workspace.
- Plain CommonMark + YAML frontmatter only.

---

## §changelog

| Date       | Version | Change |
|------------|---------|--------|
| 2026-05-06 | 0.1.0   | Initial ruleset. |
| 2026-05-10 | 0.2.0   | Split `rules.md` into always-on (`rules-core.md`) + load-on-demand (`rules-detail.md`); extracted framework tables, self-validation protocol, risk/edge-case checklists, artifact frontmatter contract, and `status.json` schema into `references/`. No behavioural change — pure structural refactor for token economy. |
