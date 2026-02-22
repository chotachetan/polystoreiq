# PolyStoreIQ Sprint Plan (Electron Desktop MVP Engineering Backlog)

## 1. Planning Guardrails

- Sprint cadence: 6 sprints, 2 weeks each (12 weeks total).
- Team assumption: 3 full-stack JavaScript/TypeScript engineers, 1 QA, 1 DevOps/SRE.
- Task sizing:
  - `XS`: 0.25 to 0.5 day
  - `S`: 0.5 to 1 day
  - `M`: 1 to 2 days
- Quality rule: no task is complete without passing acceptance criteria and tests.
- MVP scope boundary:
  - Desktop-first (Electron), macOS-first release.
  - Single-user local workspace.
  - Read-only data access.
  - Connectors in MVP: Postgres, MySQL, MongoDB, CSV/JSON/Parquet.

## 1.1 Locked Framework and Runtime Baseline

- Frontend runtime: Electron + React + TypeScript.
- Desktop architecture: `main` process + `preload` bridge + `renderer` UI + local Node engine process.
- Data stores:
  - SQLite (`better-sqlite3`) for app state (profiles, sources, chats, audit, settings).
  - DuckDB for file analytics only (CSV/JSON/Parquet ingestion and query execution).
- Connector scope for MVP: PostgreSQL, MySQL, MongoDB, file sources.
- LLM architecture: provider abstraction with hosted default adapter and optional local adapter.
- IPC/API contract strategy: schema-validated contracts (`zod`) shared across renderer and engine.
- Non-goals for MVP: Spring/Java backend, Redis dependency, Neo4j/pgvector/Qdrant, cloud multi-user collaboration.
- Optimization priority: time-to-market first, then runtime performance, then connector breadth.

## 2. Global Definition of Done (Applies to Every Task)

- [ ] Code merged to main branch.
- [ ] Unit tests added or updated and passing in CI.
- [ ] Integration/E2E tests added where relevant.
- [ ] Logs and error handling implemented (no silent failures).
- [ ] Docs updated for behavior, API, UI, or architecture changes.
- [ ] Security checks pass for changed area.

## 3. Sprint 1 (Weeks 1-2): Desktop Foundation and Security Baseline

### Sprint Goal
Stand up a runnable Electron app with secure local profile, source registry, and local persistence baseline.

### Backlog Items

#### S1-001 Monorepo skeleton and tooling
- [ ] Size: `S`
- [ ] Dependencies: None
- Implementation:
  1. Create `apps/desktop` (Electron main + preload).
  2. Create `apps/renderer` (React + TanStack + TypeScript).
  3. Create `packages/engine`, `packages/connectors`, and `packages/shared-types`.
  4. Add root scripts (`build`, `test`, `lint`, `dev`).
- Acceptance Criteria:
  1. Single command boots desktop shell, renderer, and engine.
  2. Root `build` and `test` run successfully.
- Test Cases:
  1. Run root `build` and confirm zero failures.
  2. Run root `test` and confirm all package suites execute.

#### S1-002 Local persistence baseline (SQLite)
- [ ] Size: `M`
- [ ] Dependencies: S1-001
- Implementation:
  1. Add SQLite (`better-sqlite3`) with WAL mode.
  2. Create schema for `profiles`, `sources`, `chat_sessions`, `chat_messages`, `audit_events`, `telemetry_runs`.
  3. Add migration bootstrap for first launch and upgrades.
- Acceptance Criteria:
  1. Fresh app initializes schema automatically.
  2. Existing local DB can roll forward migrations.
- Test Cases:
  1. Integration test clean DB initialization.
  2. Migration test old schema to latest.

#### S1-003 Secure secret storage
- [ ] Size: `S`
- [ ] Dependencies: S1-002
- Implementation:
  1. Store credentials using OS keychain (`keytar` or equivalent).
  2. Persist key references only in SQLite.
- Acceptance Criteria:
  1. No plaintext secrets in DB records.
  2. Secrets are retrievable only through engine runtime path.
- Test Cases:
  1. Unit test store/get/delete lifecycle.
  2. Integration test DB never stores plaintext credentials.

#### S1-004 Electron security baseline
- [ ] Size: `S`
- [ ] Dependencies: S1-001
- Implementation:
  1. Enforce `contextIsolation: true` and `nodeIntegration: false`.
  2. Implement minimal typed preload APIs.
  3. Add strict IPC validation and rejection paths.
- Acceptance Criteria:
  1. Renderer cannot call Node APIs directly.
  2. Invalid IPC payloads are rejected with typed errors.
- Test Cases:
  1. Security test for blocked unsafe IPC access.
  2. Contract tests for IPC schema validation.

#### S1-005 Local profile auth and app shell
- [ ] Size: `S`
- [ ] Dependencies: S1-002, S1-004
- Implementation:
  1. Add local profile lock (PIN or passphrase).
  2. Add protected routes for Sources, Chat, Results, and Settings.
- Acceptance Criteria:
  1. Locked profile gates app usage.
  2. Route guards survive app restart.
- Test Cases:
  1. E2E lock and unlock flow.
  2. E2E unauthorized route access blocked.

#### S1-006 Source registry service
- [ ] Size: `M`
- [ ] Dependencies: S1-003
- Implementation:
  1. Add source CRUD and status tracking (`CONNECTED`, `FAILED`, `DISABLED`).
  2. Add capability flags per source type.
- Acceptance Criteria:
  1. Source lifecycle persists locally.
  2. Disabled sources are blocked from execution.
- Test Cases:
  1. Integration test source create/list/disable lifecycle.
  2. Integration test disabled-source execution pre-check.

#### S1-007 CI and desktop packaging bootstrap
- [ ] Size: `S`
- [ ] Dependencies: S1-001
- Implementation:
  1. Add CI stages for lint, unit, integration, and E2E smoke tests.
  2. Add Electron build pipeline for macOS artifact generation.
- Acceptance Criteria:
  1. PR merge is blocked when required tests fail.
  2. CI publishes desktop build artifacts.
- Test Cases:
  1. Simulate failing tests and verify merge block.
  2. Verify artifact generation and upload.

### Sprint 1 Exit Criteria
- [ ] Desktop app launches with secure local profile, local persistence, and source registry.
- [ ] CI enforces quality gates.

## 4. Sprint 2 (Weeks 3-4): Core Connectors and Metadata Discovery

### Sprint Goal
Enable reliable read-only connection testing and metadata browsing for Postgres, MySQL, and MongoDB.

### Backlog Items

#### S2-001 Connector interface and execution contract
- [ ] Size: `S`
- [ ] Dependencies: S1-006
- Implementation:
  1. Define connector interface (`testConnection`, `fetchMetadata`, `executeReadOnly`).
  2. Add typed capability flags and unsupported-operation errors.
- Acceptance Criteria:
  1. All connector implementations compile against same interface.
  2. Unsupported capabilities return typed errors.
- Test Cases:
  1. Unit test interface contract with fake connector.
  2. Unit test capability gating behavior.

#### S2-002 Postgres connector (read-only)
- [ ] Size: `M`
- [ ] Dependencies: S2-001
- Implementation:
  1. Implement connection test and schema discovery.
  2. Enforce read-only query policy and row limits.
- Acceptance Criteria:
  1. Valid credentials pass connection test.
  2. Write-intent queries are rejected pre-execution.
- Test Cases:
  1. Integration test seeded Postgres connection and metadata.
  2. Integration test blocked write query.

#### S2-003 MySQL connector (read-only)
- [ ] Size: `M`
- [ ] Dependencies: S2-001
- Implementation:
  1. Implement connection test and metadata fetch.
  2. Enforce read-only query policy.
- Acceptance Criteria:
  1. Metadata returns tables and columns.
  2. Write-intent queries are rejected.
- Test Cases:
  1. Integration test seeded MySQL metadata.
  2. Integration test write query rejection.

#### S2-004 MongoDB connector (find/aggregate read-only)
- [ ] Size: `M`
- [ ] Dependencies: S2-001
- Implementation:
  1. Implement connection test and collection metadata.
  2. Allow read-only operations; block inserts, updates, deletes.
- Acceptance Criteria:
  1. Collection list and sampled field metadata are returned.
  2. Write operations are blocked.
- Test Cases:
  1. Integration test seeded Mongo metadata.
  2. Integration test write-command rejection.

#### S2-005 Source onboarding UI with test connection flow
- [ ] Size: `M`
- [ ] Dependencies: S2-002, S2-003, S2-004
- Implementation:
  1. Build source form for credentials and source type.
  2. Add connection test button with status feedback.
- Acceptance Criteria:
  1. User can save and test source from UI.
  2. Error feedback is actionable and non-sensitive.
- Test Cases:
  1. E2E create and test valid source.
  2. E2E invalid credentials feedback path.

#### S2-006 Metadata API and cache layer
- [ ] Size: `S`
- [ ] Dependencies: S2-002, S2-003, S2-004
- Implementation:
  1. Add unified metadata response schema.
  2. Cache metadata snapshots with configurable TTL.
- Acceptance Criteria:
  1. Metadata responses follow common schema.
  2. Repeat request within TTL uses cache.
- Test Cases:
  1. Integration test response schema validation.
  2. Cache hit and miss behavior tests.

### Sprint 2 Exit Criteria
- [ ] User can add sources, test connections, and view metadata for Postgres, MySQL, and MongoDB.

## 5. Sprint 3 (Weeks 5-6): Chat-to-Query Core and Guardrails

### Sprint Goal
Ship safe generate-preview-approve-execute flow for core connectors.

### Backlog Items

#### S3-001 Chat session and message persistence
- [ ] Size: `S`
- [ ] Dependencies: S1-002
- Implementation:
  1. Add session and message persistence in SQLite.
  2. Add APIs for create session, list sessions, and fetch messages.
- Acceptance Criteria:
  1. Messages are ordered and immutable.
  2. Session data can be resumed across app restarts.
- Test Cases:
  1. Integration test create and fetch session flow.
  2. Integration test message ordering and immutability.

#### S3-002 LLM provider abstraction and adapters
- [ ] Size: `M`
- [ ] Dependencies: S3-001
- Implementation:
  1. Define provider interface with normalized request and response shape.
  2. Implement hosted provider adapter and optional local adapter.
- Acceptance Criteria:
  1. Provider is switchable by config without UI changes.
  2. Timeouts and failures map to typed internal errors.
- Test Cases:
  1. Unit test adapter output normalization.
  2. Integration test timeout and fallback behavior.

#### S3-003 Prompt builder with schema grounding
- [ ] Size: `M`
- [ ] Dependencies: S2-006, S3-002
- Implementation:
  1. Add templates by connector type.
  2. Inject schema context and policy limits.
- Acceptance Criteria:
  1. Prompt includes allowed operations and limits.
  2. Prompt excludes credentials and sensitive fields.
- Test Cases:
  1. Unit snapshot tests for prompt composition.
  2. Security test preventing secret leakage.

#### S3-004 Guardrail validation and approval token workflow
- [ ] Size: `M`
- [ ] Dependencies: S3-003
- Implementation:
  1. Detect and block write operations and risky patterns.
  2. Require short-lived approval token before execution.
- Acceptance Criteria:
  1. Unsafe query is blocked with reason code.
  2. Query is never auto-executed without explicit approval.
- Test Cases:
  1. Injection and jailbreak regression tests.
  2. Expired and replayed approval token rejection tests.

#### S3-005 Query execution orchestration for core connectors
- [ ] Size: `S`
- [ ] Dependencies: S2-002, S2-003, S2-004, S3-004
- Implementation:
  1. Route approved query to correct connector.
  2. Persist execution latency, row count, and status.
- Acceptance Criteria:
  1. Source type routes to correct connector.
  2. Failures return typed, safe errors.
- Test Cases:
  1. Integration test source routing matrix.
  2. Integration test timeout and connector failure handling.

#### S3-006 Chat UI with query preview and approve/deny controls
- [ ] Size: `M`
- [ ] Dependencies: S3-004, S3-005
- Implementation:
  1. Build chat thread and composer UI.
  2. Add query preview panel with approve and deny actions.
- Acceptance Criteria:
  1. User can inspect generated query before run.
  2. Denied query never calls executor.
- Test Cases:
  1. E2E ask question and preview generated query.
  2. E2E deny flow prevents execution.

#### S3-007 Telemetry for model and query paths
- [ ] Size: `S`
- [ ] Dependencies: S3-002, S3-005
- Implementation:
  1. Track tokens, model latency, query latency, and trace IDs.
  2. Persist events in local telemetry tables.
- Acceptance Criteria:
  1. Every generation and execution has traceable metrics.
  2. Telemetry supports p95 calculations per stage.
- Test Cases:
  1. Unit test telemetry schema.
  2. Integration test trace ID propagation.

### Sprint 3 Exit Criteria
- [ ] User can ask, preview, approve, and execute safe read-only queries on core connectors.

## 6. Sprint 4 (Weeks 7-8): Result Normalization and Visualization

### Sprint Goal
Deliver normalized results, table-first rendering, and chart recommendations.

### Backlog Items

#### S4-001 Unified result schema and mapper library
- [ ] Size: `M`
- [ ] Dependencies: S3-005
- Implementation:
  1. Define common payload for tabular and graph-like results.
  2. Implement mappers for Postgres, MySQL, and Mongo outputs.
- Acceptance Criteria:
  1. Same UI components render all core connector results.
  2. Nulls and types are normalized consistently.
- Test Cases:
  1. Unit tests for type mapping matrix.
  2. Integration tests for schema validity.

#### S4-002 TanStack Table results component
- [ ] Size: `S`
- [ ] Dependencies: S4-001
- Implementation:
  1. Render rows and columns with sorting and pagination.
  2. Use virtualization for large result sets.
- Acceptance Criteria:
  1. Smooth scroll and interaction at configured row limit.
  2. Empty results show clear no-data state.
- Test Cases:
  1. Component tests for sorting and pagination.
  2. E2E empty-state handling.

#### S4-003 Chart recommendation rules engine
- [ ] Size: `S`
- [ ] Dependencies: S4-001
- Implementation:
  1. Rule-based mapping from result shape to chart types.
  2. Return top recommendation and alternates.
- Acceptance Criteria:
  1. Time series defaults to line chart.
  2. Categorical aggregates default to bar chart.
- Test Cases:
  1. Unit tests for recommendation matrix.
  2. Regression tests for ambiguous shapes.

#### S4-004 Chart rendering and switching
- [ ] Size: `M`
- [ ] Dependencies: S4-003
- Implementation:
  1. Integrate ECharts for line, bar, pie, and scatter views.
  2. Add chart switcher without re-executing query.
- Acceptance Criteria:
  1. Recommended chart renders by default.
  2. Chart switch preserves current dataset.
- Test Cases:
  1. E2E default chart verification.
  2. E2E chart switch behavior.

#### S4-005 Export results and chart artifacts
- [ ] Size: `S`
- [ ] Dependencies: S4-002, S4-004
- Implementation:
  1. Export table as CSV.
  2. Export chart as PNG or SVG.
- Acceptance Criteria:
  1. CSV contains displayed dataset.
  2. Exported image matches current chart state.
- Test Cases:
  1. E2E CSV validation (header and row count).
  2. E2E image export non-empty check.

#### S4-006 Session memory for follow-up questions
- [ ] Size: `M`
- [ ] Dependencies: S3-001, S3-003
- Implementation:
  1. Build context window from prior messages and query outputs.
  2. Add configurable depth and truncation.
- Acceptance Criteria:
  1. Follow-up prompts resolve prior references.
  2. Truncation preserves recent relevant turns.
- Test Cases:
  1. Integration test multi-turn resolution.
  2. Unit test truncation edge cases.

#### S4-007 Performance instrumentation for response and render SLAs
- [ ] Size: `S`
- [ ] Dependencies: S4-002, S4-004, S3-007
- Implementation:
  1. Track p95 query response and chart render timings.
  2. Add local diagnostics panel.
- Acceptance Criteria:
  1. Metrics visible for SLA thresholds.
  2. Slow path traces can be inspected by run ID.
- Test Cases:
  1. Synthetic latency test produces measurable p95.
  2. Diagnostics data integrity test.

### Sprint 4 Exit Criteria
- [ ] User receives table and auto-chart visualization for successful approved queries.

## 7. Sprint 5 (Weeks 9-10): File Data and DuckDB Analytics

### Sprint Goal
Add high-performance file analytics while preserving the same chat, approval, and visualization UX.

### Backlog Items

#### S5-001 File import workflow (CSV/JSON/Parquet)
- [ ] Size: `M`
- [ ] Dependencies: S1-002
- Implementation:
  1. Add local file picker and validation (type and size).
  2. Store file metadata and provenance in SQLite.
- Acceptance Criteria:
  1. Supported file types import successfully.
  2. Invalid files fail with actionable errors.
- Test Cases:
  1. Integration tests for each supported file type.
  2. Integration tests for invalid type and size rejection.

#### S5-002 DuckDB execution layer
- [ ] Size: `M`
- [ ] Dependencies: S5-001
- Implementation:
  1. Embed DuckDB and register imported datasets.
  2. Support schema inference and read-only execution.
- Acceptance Criteria:
  1. File datasets query successfully through common pipeline.
  2. Inferred schema appears in metadata explorer.
- Test Cases:
  1. Integration tests with representative datasets.
  2. Schema inference edge-case tests.

#### S5-003 File connector adapter and result normalization
- [ ] Size: `S`
- [ ] Dependencies: S5-002, S4-001
- Implementation:
  1. Implement `files-duckdb` connector with shared interface.
  2. Map results into unified schema.
- Acceptance Criteria:
  1. Query-to-chart flow works for file sources.
  2. Type mapping matches normalization rules.
- Test Cases:
  1. E2E file query and visualization flow.
  2. Mapper regression tests.

#### S5-004 Source capability matrix and UX hints
- [ ] Size: `S`
- [ ] Dependencies: S5-003
- Implementation:
  1. Expose capability metadata per source.
  2. Show limits and disable unsupported actions in UI.
- Acceptance Criteria:
  1. Users see source capabilities before query execution.
  2. Unsupported actions blocked in UI and engine.
- Test Cases:
  1. E2E capability rendering tests.
  2. Integration tests for policy enforcement.

### Sprint 5 Exit Criteria
- [ ] User can chat against file sources through DuckDB with the same approval and visualization flow.

## 8. Sprint 6 (Weeks 11-12): Hardening, Packaging, and Beta Readiness

### Sprint Goal
Meet security, reliability, and operability standards for desktop beta release.

### Backlog Items

#### S6-001 Immutable audit log pipeline
- [ ] Size: `S`
- [ ] Dependencies: S1-002, S3-005
- Implementation:
  1. Log source changes, query approvals, and query executions.
  2. Add append-only constraints and retention policy.
- Acceptance Criteria:
  1. Critical actions emit actor and timestamp metadata.
  2. Audit list supports filtering by source, session, and time range.
- Test Cases:
  1. Integration test action-to-audit-event mapping.
  2. Integration test append-only behavior.

#### S6-002 Role-lite access model (`owner`, `viewer`)
- [ ] Size: `S`
- [ ] Dependencies: S1-005
- Implementation:
  1. Add local permission matrix for admin and query actions.
  2. Enforce role checks in UI and engine.
- Acceptance Criteria:
  1. Viewer cannot perform owner-only operations.
  2. Unauthorized actions are hidden or blocked.
- Test Cases:
  1. Integration test role matrix.
  2. E2E role behavior checks.

#### S6-003 Rate limiting and abuse protection
- [ ] Size: `S`
- [ ] Dependencies: S3-002, S3-005
- Implementation:
  1. Add local request throttling and queue backpressure.
  2. Return typed retry guidance on limit exceed.
- Acceptance Criteria:
  1. Burst requests are throttled without app freeze.
  2. Counters reset according to policy window.
- Test Cases:
  1. Load test for throttle behavior.
  2. Integration test counter reset behavior.

#### S6-004 Resilience and failure recovery paths
- [ ] Size: `M`
- [ ] Dependencies: S3-005
- Implementation:
  1. Add retry policy for transient connector failures.
  2. Add circuit breaker for unstable sources.
- Acceptance Criteria:
  1. Transient failures retry with bounded attempts.
  2. Persistent failures trip breaker with clear user error.
- Test Cases:
  1. Integration test transient failure recovery.
  2. Integration test breaker open and recovery transitions.

#### S6-005 Desktop release pipeline (macOS)
- [ ] Size: `M`
- [ ] Dependencies: S1-007
- Implementation:
  1. Configure signing and notarization for release builds.
  2. Configure auto-update channel and rollback.
- Acceptance Criteria:
  1. Signed build installs and launches on clean machine.
  2. Update and rollback procedure is documented and tested.
- Test Cases:
  1. Packaging and install smoke test.
  2. Update and rollback dry run.

#### S6-006 Security test suite and release drill
- [ ] Size: `S`
- [ ] Dependencies: S1-003, S3-004
- Implementation:
  1. Add automated tests for IPC abuse, injection, and secret leaks.
  2. Run release security drill on candidate build.
- Acceptance Criteria:
  1. No critical security findings in release candidate.
  2. Credential handling and redaction checks pass.
- Test Cases:
  1. Security regression suite run.
  2. Manual verification checklist for secret handling.

#### S6-007 UAT and bug bash closure
- [ ] Size: `S`
- [ ] Dependencies: All prior sprint exit criteria
- Implementation:
  1. Build UAT scripts for developer, architect, and business user personas.
  2. Run bug bash and triage.
- Acceptance Criteria:
  1. All critical UAT flows pass.
  2. P0 and P1 bugs are closed or explicitly waived.
- Test Cases:
  1. UAT checklist execution with sign-off.
  2. Regression rerun after bug fixes.

### Sprint 6 Exit Criteria
- [ ] Closed desktop beta release approved.
- [ ] Reliability, security, and performance baselines meet MVP thresholds.

## 9. Cross-Sprint Test Strategy

### 9.1 Automated Test Layers
- Unit tests: business logic, validators, prompt builder, mappers, capability matrix.
- Integration tests: connectors, SQLite migrations, DuckDB file flows, orchestration.
- E2E tests: user flow from source onboarding to chart export.
- Security tests: IPC abuse, injection attempts, secret leak checks.
- Performance tests: query latency and chart render latency thresholds.

### 9.2 Mandatory Regression Pack Before Any Sprint Close
- [ ] Source onboarding tests pass.
- [ ] Read-only connector guardrail tests pass.
- [ ] Chat generate and approve and execute tests pass.
- [ ] File analytics tests (DuckDB path) pass.
- [ ] Visualization render and export tests pass.
- [ ] Audit and telemetry event schema tests pass.

## 10. Acceptance Gates by Milestone

### Gate A (End of Sprint 2)
- [ ] Source onboarding complete for Postgres, MySQL, and MongoDB.
- [ ] Metadata explorer works for all core connectors.

### Gate B (End of Sprint 4)
- [ ] Chat-to-query with approval and visualization works end-to-end for core connectors.
- [ ] Query safety guardrails verified with negative test pack.

### Gate C (End of Sprint 6)
- [ ] DuckDB-backed file flow is validated.
- [ ] Security, performance, and UAT sign-off complete.
- [ ] Signed macOS beta release candidate is deployed.

## 11. Completion Tracking Template (Use for Every Item)

Copy this block under each task in your tracker tool:

```md
- [ ] Code merged
- [ ] Unit tests passed
- [ ] Integration tests passed
- [ ] E2E tests passed (if applicable)
- [ ] Acceptance criteria verified by QA
- [ ] Docs/runbook updated
- [ ] Product owner sign-off
```

## 12. Backlog Hygiene Rules

- Split any task larger than 2 days into smaller tasks before sprint start.
- No task enters sprint without explicit acceptance criteria and test cases.
- No sprint closes with failing regression pack.
- Unplanned work over 10% capacity requires sprint re-baseline.
