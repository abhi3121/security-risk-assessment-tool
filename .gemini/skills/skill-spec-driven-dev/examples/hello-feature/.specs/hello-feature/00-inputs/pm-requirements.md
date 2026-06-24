<!-- Source: paste by user; fetched: 2026-05-06; by: skill-spec-driven-dev@0.1.0 -->
---
feature_key: hello-feature
stage: init
artifact: inputs
last_updated: 2026-05-06
confidence: high
generator: skill-spec-driven-dev@0.1.0
---

# PM requirements — hello-feature

## Requirements

- The CLI tool shall support a `hello <name>` subcommand.
- Running `hello <name>` shall print `Hello, <name>!` to stdout, followed by
  a newline.
- Exit status shall be `0` on success.

## Error handling

- When invoked without a name, the CLI shall print `usage: hello <name>` to
  stderr and exit with status `2`.

## Performance

- The `hello` function shall complete in under 10 ms for any name up to 1024
  characters.

## Non-goals

- Localisation / translation.
- Input validation of name contents (any UTF-8 string is acceptable).
