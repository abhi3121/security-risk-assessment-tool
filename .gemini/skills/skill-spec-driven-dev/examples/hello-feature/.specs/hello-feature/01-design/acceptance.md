---
feature_key: hello-feature
stage: design
artifact: acceptance
last_updated: 2026-05-06
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# Acceptance Criteria — hello-feature

## AC-1  (REQ-1)
- **kind:** automatable
- **Given** the CLI built from `main`
- **When**  the user runs `hello Alice`
- **Then**  stdout is exactly `Hello, Alice!\n` and the process exits 0.

## AC-2  (REQ-2)
- **kind:** automatable
- **Given** the CLI
- **When**  the user runs `hello` with no arguments
- **Then**  stderr is exactly `usage: hello <name>\n` and the process exits 2.

## AC-3  (REQ-1)
- **kind:** automatable
- **Given** the CLI
- **When**  the user runs `hello "   "` (whitespace-only name)
- **Then**  stdout is `Hello,    !\n` and the process exits 0.

## AC-4  (REQ-1)
- **kind:** automatable
- **Given** the CLI
- **When**  the user runs `hello <1024-char ASCII name>`
- **Then**  stdout is `Hello, <name>!\n` and the process exits 0.

## AC-5  (NFR-1)
- **kind:** performance
- **Given** the `hello()` function
- **When**  called with names of length 1, 64, 1024
- **Then**  each call completes in <10 ms (p95 over 1000 iterations on node20).

## AC-6  (REQ-1)
- **kind:** automatable
- **Given** the CLI
- **When**  the user runs `hello 🙂`
- **Then**  stdout is `Hello, 🙂!\n` and the process exits 0.
