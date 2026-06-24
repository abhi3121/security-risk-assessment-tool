# Risk, edge-case, and NFR category checklists

Reference content for `design` (when populating `risks.md` / `lld.md`) and
`tests` (when enumerating edge cases, error paths, and NFR validation). Load
only when actually generating the relevant artifact, and pick the categories
that apply.

---

## 1. Risk categories — for `risks.md`

Consider each of these when generating `RISK-n` entries. Skip any that don't
apply to the feature.

- **Unknowns / underspecified areas** — each linked to a `Q-n`.
- **External dependencies / third-party instability** — version drift,
  upstream API changes, vendor licensing, end-of-life.
- **Performance / scale** — throughput targets, latency tails, memory
  pressure, GC pauses, contention, cold-start.
- **Security / crypto / data handling** — key management, constant-time
  paths, side channels, secrets in logs, injection surfaces, supply chain.
- **Migration / rollback** — irreversible schema changes, data backfill,
  feature-flag interactions, partial-deploy compatibility.
- **Certification / compliance** — FIPS, Common Criteria, SOC 2, ISO 27001,
  PCI-DSS, GDPR / data residency, accessibility (WCAG 2.x), export control.
- **Operational** — observability gaps, alerting absence, runbook absence,
  on-call burden.
- **Concurrency / state** — re-entrancy, dead-locks, race conditions,
  idempotency, ordering guarantees.
- **Configuration / drift** — environment-specific config, secret rotation,
  feature-flag staleness.
- **Build / toolchain** — non-reproducible builds, pinned-version creep,
  cross-platform divergence.

Each `RISK-n` should have likelihood, impact, mitigation, owner, and links to
`LLD-?`, `TASK-?`, `Q-?`.

---

## 2. Edge-case prompts — for `lld.md` §Edge cases and `tests.md`

When enumerating edge cases, walk this list and pick the ones that apply.
File a `Q-n` for any that the spec does not address.

### Input boundaries
- empty input, single-element input, maximum-size input, oversize / overflow,
- whitespace-only, null / nullable, optional vs. required fields,
- unicode (combining chars, RTL, zero-width, normalisation forms NFC/NFD),
- numeric extremes (0, ±1, ±MAX, NaN, Inf, denormals), negative values,
- truncation, malformed encoding (bad UTF-8, BOM, line endings).

### State / lifecycle
- first-use / cold start, repeated invocation, mid-operation cancellation,
- shutdown / SIGTERM during work, crash recovery, partial failure,
- re-entrancy, idempotency on retry, exactly-once vs at-least-once semantics,
- ordering when concurrent.

### Resources
- out of memory, disk full, fd exhaustion, network outage,
- timeout (read / write / connect), slow downstream, downstream 5xx burst,
- rate-limit hit, back-pressure, retry storm.

### Concurrency
- two callers race, reader during writer, writer during reader,
- cancellation during critical section, lock acquisition order,
- deadlock detection, false sharing, ABA.

### Backward / forward compatibility
- old client / new server, new client / old server, mixed-version cluster,
- on-disk format vN reads vN-1 data, schema migrations partially applied,
- deprecated API still in use.

### Security & abuse
- malformed / hostile inputs, injection (SQL, command, LDAP, XSS, XXE),
- path traversal, SSRF, deserialisation gadgets,
- timing leaks, error-message leaks, log leaks of secrets,
- privilege boundary crossing, missing authn / authz, replay attack,
- denial of service (algorithmic complexity attack).

### UI / UX (when applicable)
- empty / loading / error states, very long content, RTL layout,
- offline / flaky network, slow render budget,
- screen-reader / keyboard-only navigation, high-contrast mode,
- breakpoint transitions (mobile ↔ tablet ↔ desktop), zoom 200 %.

---

## 3. NFR families — for `inputs.yaml` (NFR-n) and `tests.md` (performance / security level TCs)

- **Performance** — latency (p50/p95/p99), throughput, memory footprint,
  startup time, build time, binary size.
- **Reliability** — MTBF target, recovery time, durability, retry semantics.
- **Security** — confidentiality, integrity, authentication strength,
  cryptographic agility, audit logging, sandboxing.
- **Compatibility** — OS / arch matrix, browser matrix, mobile OS versions,
  language runtime versions, dependency-version range.
- **Usability** — accessibility (WCAG 2.1 AA minimum if UI), i18n / l10n,
  responsive design breakpoints, keyboard navigability.
- **Observability** — required metrics, log channels, trace propagation,
  alerting hooks, error taxonomy.
- **Maintainability** — code-style, test-coverage threshold, documentation
  requirements.
- **Compliance** — certification (FIPS / SOC2 / ISO / GDPR / PCI-DSS),
  licensing constraints, export-control flags.

Every NFR included in `inputs.yaml` must be measurable — replace "should be
fast" with a number and a method.
