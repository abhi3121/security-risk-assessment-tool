---
name: skill-spec-driven-dev
slug: spec-driven
version: 0.2.0
description: >
  Spec-driven development (SDD) skill. Turns PM docs, HLD, feature tickets
  (Jira/etc.), architecture documents, and any in-repo knowledge base into a
  low-level design, acceptance criteria, an implementation plan, a TDD test
  suite, and finally production code — while maintaining requirement
  traceability throughout. Invoke when the user asks to design, spec, plan,
  test-first, or implement a feature.
invocation_hints:
  - "design a feature"
  - "create an LLD for ..."
  - "write tests for this feature"
  - "implement LUNA-* / JIRA-*"
  - "run spec-driven development for ..."
  - "/spec-init /spec-design /spec-tests /spec-implement /spec-lint /spec-pipeline"
entrypoint_rules: rules-core.md
capabilities:
  - id: init
    file: capabilities/init.md
    when: "start a new feature: gather inputs, pick workspace, detect toolchain"
  - id: design
    file: capabilities/design.md
    when: "inputs are ready; produce LLD + acceptance criteria + implementation plan"
  - id: tests
    file: capabilities/tests.md
    when: "design is ready; produce TDD test spec and failing test skeletons"
  - id: implement
    file: capabilities/implement.md
    when: "design + tests are ready; implement code task-by-task until green"
  - id: lint
    file: capabilities/lint.md
    when: "verify REQ->AC->TC->TASK traceability and coverage gates"
  - id: pipeline
    file: capabilities/pipeline.md
    when: "run init->design->tests->implement sequentially, pausing at gates"
references:
  rules_detail: rules-detail.md
  frameworks:   references/frameworks.md
  artifacts:    references/artifacts.md
  status_schema: references/status-schema.md
  self_validation: references/self-validation.md
  risk_categories: references/risk-categories.md
state_file_relative_to_workspace: status.json
id_scheme:
  REQ:  "Product / feature requirement"
  NFR:  "Non-functional requirement"
  AC:   "Acceptance criterion"
  LLD:  "Low-level design section"
  TASK: "Implementation plan task"
  RISK: "Risk, ambiguity, or open assumption"
  TC:   "Test case"
  Q:    "Clarification question / blocker"
---

# Spec-Driven Development Skill — router

This file is the entry point. Its frontmatter is the machine-readable
catalogue; the prose below is only the routing rules.

## Routing rules

1. **Always read `rules-core.md` first** (small, always-on). Treat it as part
   of your system prompt for the duration of any spec-driven-dev work.
2. **Identify the requested capability:**
   - Explicit slash command (`/spec-<cap>` or `spec-<cap>`) → that capability.
   - Natural language → pick the capability whose `when:` clause best matches
     the user's intent. If ambiguous, ask.
3. **Check preconditions** by reading the feature workspace's `status.json`
   (if it exists). A capability may only run if its preconditions (declared
   at the top of each capability file) are satisfied. If not, stop and tell
   the user which earlier capability to run first.
4. **Load only the capability file you need.** Do not preload all
   capabilities. Pull `references/*.md` and `rules-detail.md` only when a
   step explicitly points at them.
5. **After any capability finishes**, update `status.json` and append to
   `traceability.md` if IDs were added or changed.

## Workspace layout

Each feature has its own workspace at `<repo-root>/.specs/<FEATURE_KEY>/`
(user-overridable at `init` time). The exact layout is defined in
`rules-core.md` §3.

## Hard guarantees (summary — full text in `rules-core.md`)

- **Specs first, code last** — no production code before LLD + AC + failing tests.
- **Traceability is non-negotiable** — REQ→AC→TC→TASK→commit, append-only.
- **Graceful degradation** — missing inputs become `Q-n`, never silent invention.
- **TDD enforced mechanically** — `implement` refuses without `red_state_confirmed: true`.
- **No auto-commit** — `implement` leaves the working tree dirty for review.

Proceed to `rules-core.md`, then to the requested capability file under
`capabilities/`.
