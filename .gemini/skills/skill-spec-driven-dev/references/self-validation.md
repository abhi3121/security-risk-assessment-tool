# Self-validation protocol

Every capability runs this protocol before declaring itself done. It's a
guardrail: if a capability cannot answer its own self-validation questions
using only the artifacts it just wrote, it has a gap to fix before exiting.

## Protocol

1. **Pick the 2–3 self-validation questions** declared by the capability you
   just ran (each capability file lists its own questions in a "self-validation
   questions" section — keep them short, verifiable, and tied to artifacts).
2. **Answer each question using only the files this capability wrote (or
   updated)** during this run. Do not consult external files or memory.
3. **If any answer requires content from a file the capability did not write or
   update**, that is a gap:
   - Either extend the capability's output so the answer is locally
     reachable, or
   - File a `Q-n` (advisory) noting the cross-file dependency, then proceed.
4. **Record the result** by updating `status.json`:
   - `stages.<cap>.self_validation_passed: true | false`
   - On `false`, set `stages.<cap>.self_validation_notes: <one-line reason>`.

The protocol is intentionally light: it costs ~30 seconds and catches the
most common "wrote the artifact but forgot to update traceability" class of
bug. Each capability MUST list its own questions; this file just defines the
how, not the what.

## Why questions, not assertions?

Because the questions force the agent to read the artifact it just produced
through the eyes of a downstream consumer. A YAML-style assertion ("REQ count
> 0") is satisfied by trivial output; "list every REQ with its source
citation" only succeeds when the output is actually useful.

## Gate interaction

Self-validation is **not** a hard gate by itself — the per-capability gates
(declared in each capability file) are what advance `status.json.stage`. A
failed self-validation is a warning that something is incomplete; resolve it
before running the next capability.
