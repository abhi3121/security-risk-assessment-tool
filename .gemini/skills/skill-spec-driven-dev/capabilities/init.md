# Capability: init — Normalise inputs into a feature workspace

Run this first for every new feature. Outcome: a fully-populated `00-inputs/`
folder, an initial `status.json`, and a `context-manifest.md` describing what
this host repo offers.

## Preconditions

- None. `init` always runs from a clean slate (or re-runs to refresh inputs).

## Inputs (what to look for, in order)

1. **Feature key** — Jira ticket id preferred (verbatim). Fallback: a
   user-supplied kebab slug. If not provided, prompt:

   > "Do you have a Jira (or equivalent) ticket id for this feature?
   > Enter the id (e.g. `LUNA-1234`), or leave blank to use a custom slug."

   If blank, prompt:

   > "Enter a short kebab-case name for the feature (e.g.
   > `multi-part-envelope-import`):"

2. **Workspace path** — prompt:

   > "Where should the spec artifacts be stored?
   > Default: `.specs/<FEATURE_KEY>/` at the repo root. Press enter to accept,
   > or provide an absolute/relative path."

   Resolve, create directories, record in `status.json.workspace_path`.

3. **Product / feature inputs** — ask which are available and accept any of:
   a file path, a URL, an MCP reference, or "paste":

   - Product / feature requirements (PM brief)
   - High-level design (HLD)
   - Feature ticket (Jira / GitHub issue / GitLab issue / …)
   - Architecture design documents
   - Any additional context
   - **UI development inputs** (only if UI is in scope — see step 4a)

   Save each under `00-inputs/<slug>.md` using the canonical names
   (`pm-requirements.md`, `hld.md`, `jira.md`, `architecture-refs.md`, plus
   freeform `extra-<slug>.md`). Frontmatter / provenance header rules:
   **`references/artifacts.md`**.

## Steps

### 1. Resolve feature key + workspace

- Prompt for feature key + workspace path as above.
- Create `<workspace>/{00-inputs,01-design,02-tests,03-impl}` plus empty
  `traceability.md`, `clarifications.md`, `README.md`, `status.json`.

### 2. Detect host repo context → `00-inputs/context-manifest.md`

Probe the host repo for each category and record findings (one section per
category; bullet list of detected items, or the literal line `_not detected_`):

- **Knowledge base**: `.wiki/`, `wiki/`, `knowledge/`, `memory-bank/`. If a
  schema/index file exists (`schema.md`, `README.md`), record its path.
- **Design-doc folders**: `docs/`, `design/`, `specs/`, per-module `docs/`.
- **Ticketing integration**: try available MCP servers for Jira / Confluence
  / GitHub / GitLab. Record which one responded.
- **Build system & test framework**: use the detection signals in
  **`references/frameworks.md`** §1 and §2. List every match. If detection
  is ambiguous, list candidates and ask the user to confirm.
- **Agent rules files**: `CLAUDE.md`, `AGENTS.md`, `.clinerules/`, `.cursor/`,
  `.github/copilot-instructions.md`.
- **CI**: `.gitlab-ci.yml`, `.github/workflows/`, `azure-pipelines.yml`,
  `Jenkinsfile`, `.circleci/`, `bitbucket-pipelines.yml`.

### 3. Resolve toolchain (interactive)

Present the candidate build + test commands and confirm or override:

> "Detected build system: `<x>`. Detected test framework: `<y>`.
> Suggested commands:
>   build: `<build cmd>`
>   test:  `<test cmd>`
> Use these (y), override (o), or skip for now (s)?"

Record confirmed values under `status.json.toolchain`.

### 4. Fetch or collect inputs

For each input the user said exists:

- **Jira / ticketing via MCP** — try the first available ticketing MCP. Save
  the raw response under `00-inputs/jira.md`. Include ticket key, title,
  status, reporter, description, acceptance criteria (if any), comments
  relevant to scope, and attachments list. On failure, prompt the user to
  paste.
- **Local files** — copy (not move) into `00-inputs/` with a normalised name.
- **URLs** — fetch if a tool is available; otherwise prompt to paste.
- **Paste mode** — accept a block of text; save verbatim.

Every saved file must start with the provenance header defined in
**`references/artifacts.md`** §3, above the YAML frontmatter.

### 4a. UI development inputs (conditional)

Ask:

> "Does this feature involve UI development? (y/N)"

If **no** (or feature is clearly back-end only): set
`ui_inputs.applicable: false` in `inputs.yaml`. Do NOT create empty UI
artifact files — keep the workspace clean.

If **yes**, prompt for each in turn (any may be left blank; all independently
optional):

1. **UI prototype URL(s)** — hosted prototype, Storybook, deployed preview,
   design-system playground. Multiple URLs accepted (one per line).
2. **Figma specs** — file URL(s) and/or specific frame / node URLs.
3. **Screenshots / mockups** — for each: local file path, URL, or "paste"
   (agent attaches inline image).

Save artifacts under `00-inputs/`:

- `00-inputs/ui-prototype.md` — markdown table of every prototype URL with
  description, captured timestamp, and (if applicable) login / navigation
  hints.
- `00-inputs/figma-specs.md` — markdown table of every Figma URL: file link,
  frame name, frame URL, description, captured timestamp. Include screen
  flow if the user mentions one.
- `00-inputs/screenshots/` — directory holding image files. **Local image
  files MUST be COPIED here** (not moved, not symlinked) so the workspace
  remains self-contained. For URL-only images, record in
  `00-inputs/screenshots.md` (URL, description, captured timestamp).

Each markdown file uses frontmatter with `artifact: ui-inputs`. Update
`inputs.yaml.ui_inputs` block (see `../templates/inputs.yaml`) and set
`ui_inputs.applicable: true`.

If any UI input contains visible PII or secrets, flag inline and ask before
persisting.

### 5. Extract REQ / NFR statements

Read all collected inputs. Write candidates into `00-inputs/inputs.yaml` (see
`../templates/inputs.yaml`):

- One `REQ-n` per functional requirement.
- One `NFR-n` per non-functional requirement (see NFR families in
  **`references/risk-categories.md`** §3).
- Each entry MUST have:
  - `source:` — `<file>#<anchor>` or `mcp:jira:<KEY>#description`
  - `statement:` — text, verbatim or lightly normalised
  - `category:` — functional | performance | security | reliability |
    compatibility | usability | observability | other
  - `derived:` — `true` if paraphrased/inferred, `false` if verbatim
- For any gap or ambiguity, file a `Q-n` (blocker if it prevents design,
  else advisory).

### 6. Detect contradictions

Cross-compare inputs. If sources disagree, file a `Q-n` with severity
`blocker` quoting both sources. Do NOT pick a side.

### 7. Write / update `status.json`

Update per the protocol in **`references/status-schema.md`** §3:

- `stage`: `inputs-ready` if no blocker Q-n, else `inputs-blocked`.
- `stages.init.done` = `true` (if unblocked) with ISO-8601 timestamp.
- `counters.REQ`, `counters.NFR`, `counters.Q` reflect counts.
- `host_integrations` and `toolchain` from steps 2–3.
- `artifacts[]` lists every file written this run.

### 8. Seed `traceability.md` and `README.md`

- `traceability.md` — header + table header row + one row per `REQ-*`/`NFR-*`
  with `—` cells in later columns. Status `needs-design`.
- `README.md` (workspace) — one-page overview: feature key, title (from
  ticket or user), summary, links to every artifact file, current stage, and
  open `Q-n` list.

### 9. Self-validation

Run the protocol in **`references/self-validation.md`** using these
questions:

1. "What is the title and feature key of this feature, and where is the
   workspace?" → answer from `README.md` + `status.json`.
2. "List every REQ and NFR with its source citation." → answer from
   `inputs.yaml`.
3. "Which host-repo integrations are available (knowledge base, ticketing,
   build, tests)?" → answer from `context-manifest.md` + `status.json`.

### 10. Report to the user

Print a short summary:
- Feature key, workspace path.
- Counts of REQ, NFR, Q (blocker / advisory breakdown).
- Toolchain: build command, test command.
- Detected integrations (knowledge base, ticketing, formal-docs folder, CI).
- If blockers exist: list them and STOP. Tell the user to resolve them
  (edit `clarifications.md`, then re-run init or proceed once resolved).
- If no blockers: invite the user to run `design` next.

## Outputs (files this capability writes)

- `<workspace>/00-inputs/inputs.yaml`
- `<workspace>/00-inputs/context-manifest.md`
- `<workspace>/00-inputs/<source>.md` for each supplied input
- UI artifacts (only when `ui_inputs.applicable: true`)
- `<workspace>/traceability.md` (seeded)
- `<workspace>/clarifications.md` (may contain Q-n entries)
- `<workspace>/status.json`
- `<workspace>/README.md`

## Gate (must pass to advance `stage` to `inputs-ready`)

- `inputs.yaml` parses and contains ≥1 REQ or NFR.
- Every REQ/NFR has a non-empty `source` and `statement`.
- `context-manifest.md` exists.
- No `Q-n` of severity `blocker` is unresolved.

If any gate fails, set `stage: inputs-blocked` and stop.
