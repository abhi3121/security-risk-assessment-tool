# Example — `hello-feature`

A minimal worked example of the `skill-spec-driven-dev` skill, designed to show the
shape of every artifact without drowning you in detail.

Scenario: add a tiny `hello(name)` function to an imaginary CLI tool. Two
requirements:

- **REQ-1** The CLI shall print `Hello, <name>!` when invoked as
  `hello <name>`.
- **NFR-1** The function shall run in under 10 ms for any `name` up to 1024
  characters.

What you'll find in `.specs/hello-feature/`:

- `00-inputs/inputs.yaml` — normalised requirements.
- `00-inputs/context-manifest.md` — a repo with no knowledge base, no Jira.
- `01-design/lld.md`, `acceptance.md`, `plan.md`, `risks.md` — design stage output.
- `02-tests/tests.md` + `test-matrix.csv` — three test cases; kept abstract
  since no real test framework is wired up.
- `03-impl/impl-log.md` + `verify-report.md` — pretend-implementation report.
- `traceability.md` — the matrix filled all the way through.
- `clarifications.md` — one advisory Q still open.
- `status.json` — final state at `implement-done`.

Use this example as a reference for the artifact shapes, YAML frontmatter,
and ID conventions. **Do not** use the numbers here as defaults for real
features — they're deliberately small.
