# `status.json` — full schema and worked example

`status.json` lives at the root of every feature workspace and is the
machine-readable state machine of the feature. Every capability updates it
atomically at the end of its run.

`rules-core.md` covers the always-on invariants (monotonic `stage`,
append-only counters, etc.). This file is the full schema reference and is
loaded only when a capability is actually writing or repairing `status.json`.

---

## 1. Field-by-field

```jsonc
{
  // Identity
  "feature_key":     "<string, verbatim>",      // e.g. "LUNA-1234" or "multi-part-import"
  "workspace_path":  "<string, repo-relative>", // e.g. ".specs/LUNA-1234"
  "title":           "<string, human title>",

  // Current stage of the state machine.
  // One of: inputs-blocked | inputs-ready | design-ready | tests-ready | implement-done
  "stage": "design-ready",

  // Per-stage completion + gates
  "stages": {
    "init":      { "done": true,  "at": "<ISO-8601>" },
    "design":    { "done": true,  "at": "<ISO-8601>",
                   "gates": {
                     "lld_complete": true,
                     "acceptance_covers_all_reqs": true,
                     "plan_ordered_and_sized": true,
                     "risks_registered": true
                   } },
    "tests":     { "done": false,
                   "gates": { "red_state_confirmed": false,
                              "red_state_notes": "<optional explanation>" } },
    "implement": { "done": false,
                   "gates": { "green_state_confirmed": false } },
    "lint":      { "done": false }  // optional; populated after a lint run
  },

  // Counters — must match actual ID counts in the workspace (lint enforces).
  "counters": {
    "REQ": 0, "NFR": 0, "AC": 0, "LLD": 0,
    "TASK": 0, "TC": 0, "RISK": 0, "Q": 0
  },

  // Quick-glance counts derived from clarifications.md / risks.md.
  "open_clarifications": 0,
  "open_risks": 0,

  // Toolchain — established at init, may be updated by tests.
  "toolchain": {
    "build_system":    "cmake",          // see references/frameworks.md §1
    "test_framework":  "gtest+ctest",    // see references/frameworks.md §2
    "build_command":   "<string>",
    "test_command":    "<string>",
    "languages":       ["c", "c++"],
    "targets":         ["amd64-release", "arm64-release"]  // optional
  },

  // Host-repo integrations detected at init.
  "host_integrations": {
    "knowledge_base":     "<path|null>",   // e.g. ".wiki"
    "formal_docs_root":   "<path|null>",   // e.g. "docs/"
    "jira_mcp":           true,
    "github_mcp":         false,
    "gitlab_mcp":         false,
    "ci":                 "<gitlab-ci|github-actions|azure-pipelines|null>"
  },

  // Catalogue of every file the skill has written to this workspace.
  "artifacts": [
    { "path": "00-inputs/inputs.yaml", "generator": "skill-spec-driven-dev@<version>" }
  ]
}
```

---

## 2. `stage` transitions

| From               | Trigger             | To                |
|--------------------|---------------------|-------------------|
| *(no file)*        | `init` starts       | (file created)    |
| *(any)*            | `init` finishes w/o blockers | `inputs-ready`     |
| *(any)*            | `init` finishes w/ blocker `Q-n` | `inputs-blocked` |
| `inputs-ready`     | `design` passes all gates | `design-ready` |
| `design-ready`     | `tests` confirms RED state | `tests-ready`  |
| `tests-ready`      | `implement` confirms GREEN state | `implement-done` |
| *(any)*            | user re-runs an earlier capability | reverts to that capability's "ready" value |

`stage` is monotone unless a capability is intentionally re-run; the `lint`
capability flags backward jumps not caused by an explicit re-run.

---

## 3. Update protocol (every capability)

1. Read `status.json` (treat as absent → start fresh).
2. Do the capability's work.
3. Just before exiting:
   - Update `stages.<cap>.done` and `stages.<cap>.at`.
   - Update `stages.<cap>.gates.*` with the result of each gate.
   - Update `counters` to reflect the actual ID counts on disk.
   - Append any new entries to `artifacts[]`.
   - Update `open_clarifications` and `open_risks`.
   - Update `stage` to the new value iff all gates passed.
4. Write `status.json` atomically (write-then-rename if the agent supports it;
   otherwise a single write is acceptable — the file is small).

---

## 4. Worked example

```json
{
  "feature_key": "LUNA-1234",
  "workspace_path": ".specs/LUNA-1234",
  "title": "Add ML-KEM-1024 support to CRY_MOD",
  "stage": "design-ready",
  "stages": {
    "init":      { "done": true, "at": "2026-05-06T14:00:00Z" },
    "design":    { "done": true, "at": "2026-05-06T15:40:00Z",
                   "gates": { "lld_complete": true,
                              "acceptance_covers_all_reqs": true,
                              "plan_ordered_and_sized": true,
                              "risks_registered": true } },
    "tests":     { "done": false, "gates": { "red_state_confirmed": false } },
    "implement": { "done": false, "gates": { "green_state_confirmed": false } }
  },
  "counters": { "REQ": 7, "NFR": 2, "AC": 9, "LLD": 14, "TASK": 11,
                "TC": 0,  "RISK": 3, "Q": 2 },
  "open_clarifications": 2,
  "open_risks": 3,
  "toolchain": {
    "build_system": "cmake",
    "test_framework": "gtest+ctest",
    "test_command": "ctest --preset amd64-release",
    "languages": ["c"]
  },
  "host_integrations": {
    "knowledge_base": ".wiki",
    "formal_docs_root": "firmware_vob/SOURCE/LUNA2",
    "jira_mcp": true
  },
  "artifacts": [
    { "path": "00-inputs/inputs.yaml", "generator": "skill-spec-driven-dev@0.2.0" }
  ]
}
```
