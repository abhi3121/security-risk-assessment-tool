# Artifact contract — frontmatter, naming, and provenance

Single source of truth for what every artifact this skill writes must look
like. Capabilities reference this file instead of inlining the rules.

---

## 1. Markdown artifact frontmatter (required)

Every `.md` artifact written by this skill MUST start with this YAML block:

```yaml
---
feature_key: <FEATURE_KEY>      # verbatim from status.json.feature_key
stage: <stage>                  # init | design | tests | implement | lint
artifact: <kind>                # see "Artifact kinds" table below
last_updated: YYYY-MM-DD
confidence: <level>             # high | medium | low
generator: skill-spec-driven-dev@<version>
---
```

### Confidence levels

| Level | When to use |
|-------|-------------|
| `high`   | Cross-checked against authoritative sources AND human-reviewed. AI-only verification does NOT count. |
| `medium` | AI-extracted from sources, not yet human-reviewed. The default for newly generated artifacts. |
| `low`    | Inferred from indirect signals (filenames, README, conventions). Mark inline content with ⚠️. |

Inline emoji markers (✅ / ⚠️ / ❌) belong in the body for human readers; the
`confidence:` field is for machines.

---

## 2. Artifact kinds (controlled vocabulary)

| `artifact:` value | File path (under `<workspace>/`) | Created by |
|-------------------|----------------------------------|------------|
| `inputs`            | `00-inputs/inputs.yaml`              | init |
| `context-manifest`  | `00-inputs/context-manifest.md`      | init |
| `pm-requirements`   | `00-inputs/pm-requirements.md`       | init (if supplied) |
| `hld`               | `00-inputs/hld.md`                   | init (if supplied) |
| `jira`              | `00-inputs/jira.md`                  | init (if supplied) |
| `architecture-refs` | `00-inputs/architecture-refs.md`     | init (if supplied) |
| `ui-inputs`         | `00-inputs/ui-prototype.md`, `00-inputs/figma-specs.md`, `00-inputs/screenshots.md` | init (UI scope only) |
| `lld`               | `01-design/lld.md`                   | design |
| `acceptance`        | `01-design/acceptance.md`            | design |
| `plan`              | `01-design/plan.md`                  | design |
| `risks`             | `01-design/risks.md`                 | design |
| `tests`             | `02-tests/tests.md`                  | tests |
| `test-matrix`       | `02-tests/test-matrix.csv`           | tests (no frontmatter — CSV) |
| `impl-log`          | `03-impl/impl-log.md`                | implement |
| `verify-report`     | `03-impl/verify-report.md`           | implement |
| `traceability`      | `traceability.md`                    | every stage (append-only) |
| `clarifications`    | `clarifications.md`                  | any stage |
| `readme`            | `README.md` (inside workspace)       | init / updated by others |
| `lint-report`       | `lint-report-YYYY-MM-DD.md`          | lint |

CSV and JSON artifacts (`test-matrix.csv`, `status.json`) do not carry
frontmatter — they're listed in `status.json.artifacts[]` with a generator
stamp instead.

---

## 3. Provenance header (for files copied / fetched from external sources)

When `init` copies an external document into `00-inputs/`, prepend this
HTML-comment line **above** the YAML frontmatter:

```
<!-- Source: <path|url|mcp:jira:KEY>; fetched: YYYY-MM-DD; by: skill-spec-driven-dev@<version> -->
```

It's a comment so it doesn't interfere with YAML parsing, but it's grep-able
and stays with the file.

---

## 4. ID assignment & authoring rules

- IDs use the `<PREFIX>-<n>` form, with `n` monotonically increasing within
  the feature workspace. Never reset, never renumber.
- An ID is **defined** in exactly one file (e.g. `REQ-*` in `inputs.yaml`,
  `AC-*` in `acceptance.md`, `TC-*` in `tests.md`). It may be **referenced**
  from any other file.
- Append-only: superseded IDs stay with `status: retired` (in the traceability
  matrix) or `status: superseded` (in the defining artifact). Never delete.

Prefix list lives in `rules-core.md` §ID scheme.

---

## 5. Filename conventions

- Lowercase, kebab-case, ASCII-only.
- Phase folders: `00-inputs/`, `01-design/`, `02-tests/`, `03-impl/`. The
  numeric prefix is part of the contract — do not rename.
- Feature workspace lives under `<repo-root>/.specs/<FEATURE_KEY>/` unless the
  user overrides at init time. Record the resolved path in
  `status.json.workspace_path`.

---

## 6. What NOT to write

- No proprietary markdown extensions (Confluence macros, GitLab admonitions,
  GitHub `:::` callouts). Plain CommonMark only.
- No executable code blocks. Use fenced code blocks for samples only.
- No raw secrets / credentials / keys / tokens — flag and prompt before
  persisting.
- No files outside the workspace, except:
  - production source code during `implement` (under the host repo source
    tree), or
  - mirrored LLD into a host-repo formal-doc folder when the user opts in.
