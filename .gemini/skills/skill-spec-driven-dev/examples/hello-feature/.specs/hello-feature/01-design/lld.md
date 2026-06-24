---
feature_key: hello-feature
stage: design
artifact: lld
last_updated: 2026-05-06
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# Low-Level Design — hello-feature

## Overview & scope

Add a `hello` subcommand that prints `Hello, <name>!` and returns exit code
0. Does not change existing commands; does not persist state.

## Affected components

| Component       | Role                                     | Source path       |
|-----------------|------------------------------------------|-------------------|
| CLI dispatcher  | Routes `hello` to the new handler        | `src/cli/index.ts`|
| Hello module    | Implements the greeting function + handler | `src/cli/hello.ts`|

## Interfaces

### LLD-1 Public function — `hello(name: string): string`
- **traces:** REQ-1
- **Signature:** `export function hello(name: string): string`
- **Returns:** `` `Hello, ${name}!` ``
- **Preconditions:** `name` is a non-empty string (caller's responsibility).
- **Errors:** none (pure function).
- **Thread-safety:** pure; safe for concurrent use.

### LLD-2 CLI handler — `handleHello(argv: string[])`
- **traces:** REQ-1, REQ-2
- **Signature:** `export function handleHello(argv: string[]): number`
- **Returns:** process exit code (`0` on success, `2` on usage error).
- **Behaviour:**
  - If `argv` has a name: print `hello(name)` to stdout followed by newline,
    return 0.
  - Else: print `usage: hello <name>` to stderr, return 2.

## Data model

None (stateless).

## Dependencies

| Dependency | Kind | Notes |
|------------|------|-------|
| node:process | internal | `process.stdout`, `process.stderr`, exit code |

## Error handling & observability

- Usage errors → stderr, exit code 2.
- No logging infrastructure at this scope.

## Configuration

None.

## Edge cases

| Edge case                  | Covered by |
|----------------------------|------------|
| No arguments               | AC-2       |
| Whitespace-only name       | AC-3       |
| 1024-character name        | AC-4, AC-5 (perf) |
| Unicode name (emoji)       | AC-6       |

## Constraints

- **Performance (NFR-1):** `hello()` must return in <10 ms for names ≤ 1024
  chars. Measured by vitest bench. Expected actual: <1 ms (plenty of margin).

## Open questions

- Q-1 (advisory): Should a name longer than 1024 chars be rejected, truncated,
  or accepted silently? Assumption: accepted silently (matches "any UTF-8
  string is acceptable" non-goal). ⚠️
