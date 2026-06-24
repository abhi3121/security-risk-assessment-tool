# `references/` — load-on-demand reference material

Files in this directory are **not** part of the always-on skill prompt. They
are loaded only when a capability step explicitly points at them, and even then
typically only a single row / section is needed.

Keeping this material out of the always-on path is what allows the core skill
to fit in a small token budget while still covering many frameworks, ID
conventions, and host-repo patterns.

| File | Purpose | Loaded by |
|------|---------|-----------|
| `frameworks.md` | Build-system + test-framework detection signals, conventional test-file locations, failing-test skeleton snippets (one row per framework). | `init` step "detect host repo context", `tests` step "generate skeletons" |
| `self-validation.md` | The standard self-validation protocol every capability runs at the end. | Every capability's "self-validation" step |
| `risk-categories.md` | Checklists of risk categories, edge cases, and NFR families to consider when populating `risks.md` / `lld.md` / `tests.md`. | `design` (risks), `tests` (edge/error/NFR cases) |
| `artifacts.md` | The frontmatter contract for every artifact this skill writes (one source of truth, replaces inlined repetition). | Any capability that writes a markdown artifact |
| `status-schema.md` | The full `status.json` shape with field-by-field semantics and a worked example. Referenced from `rules-core.md` §status. | Any capability that writes `status.json` (effectively all of them, but the file is small) |

## Convention for capabilities

When a capability step needs reference content, it should say something like:

> "Look up the detected framework in `references/frameworks.md` and copy only
> the matching row's skeleton pattern."

Capabilities MUST NOT inline the reference content. If a reference is missing
a row you need, add the row in `references/` rather than embedding it in the
capability.
