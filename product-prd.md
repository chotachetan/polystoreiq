# PolyStoreIQ PRD (Local-First Desktop)

## 1. Product Vision

Build a local-first desktop app where users connect any data store, ask natural language questions, and get trusted answers plus visualizations without needing a hosted backend server.

Long-term goal: make cross-database insight accessible to developers, architects, and non-technical users through one secure desktop interface.

## 2. Problem Statement

Teams use relational, NoSQL, graph, vector, and file-based stores with different tools and query languages. This causes:

1. Slow insight cycles.
2. Heavy specialist dependency.
3. Fragmented tooling and inconsistent UX.

Users need one interface that hides database complexity while keeping control, privacy, and explainability.

## 3. Product Principles

1. Local first: core functionality runs on-device.
2. Trust first: generated queries are visible and user-approved.
3. Safe by default: read-only execution and strict guardrails.
4. Unified UX: same workflow across storage types.
5. Optional cloud: cloud services are add-ons, not dependencies.

## 4. MVP Scope (Desktop V1)

### 4.1 In Scope

1. Desktop app with Electron + TanStack (macOS and Windows).
2. Local profile and workspace model (on-device).
3. Data source onboarding:
   - Secure credential entry.
   - Local connection testing and credential storage.
4. Supported connectors:
   - Relational: PostgreSQL, MySQL
   - NoSQL: MongoDB
   - Graph: Neo4j
   - Vector: pgvector (Postgres), Qdrant
   - File-based: CSV, JSON, Parquet
5. Chat-to-query:
   - Natural language prompt.
   - Query generation per source type.
   - Mandatory query approval before execution.
   - Ability to execute query and apply NLP conversion before showing results.
6. Visualization:
   - Auto chart recommendation (table, line, bar, pie, scatter, graph view).
   - Manual chart switching.
   - CSV and chart image export.
7. Local observability:
   - Query latency.
   - LLM latency/token usage.
   - Error logs and trace IDs.
8. Security:
   - OS keychain for tokens/secrets.
   - Local encrypted app database.
   - Strict Electron hardening and IPC allowlist.

### 4.2 Out of Scope (MVP)

1. Required hosted backend.
2. Multi-user real-time collaboration.
3. Write-back operations (insert/update/delete).
4. Marketplace for third-party connector plugins.
5. Full offline LLM quality parity for all workloads.

## 5. Core User Stories

1. As a developer, I connect Postgres and ask for top customers by revenue and see SQL plus chart.
2. As an architect, I run equivalent intent across MongoDB and Neo4j with one UX.
3. As a business user, I ask for trends without writing SQL or Cypher.
4. As a security-conscious team member, I keep secrets local and avoid sending database credentials to a central server.

## 6. Functional Requirements

1. Source management:
   - Create, test, edit, disable local data sources.
2. Metadata discovery:
   - Pull schema, collections, labels, indexes for grounding.
3. Query generation:
   - Connector-specific generation using pluggable model adapters.
4. Query guardrails:
   - Block destructive commands and enforce limits.
5. Query execution:
   - Connector execution engine with timeouts and cancellation.
6. Result normalization:
   - Convert heterogeneous results into a shared schema.
7. Visualization recommendation:
   - Rule-based chart suggestion from result shape.
8. Session memory:
   - Multi-turn context in local chat history.
9. Local audit trail:
   - Append-only logs for critical actions.
10. Ability for user to provide feedback:
   - Change queries based on feedback from Users.
   - Cache the query changes for future use on dataset
   

## 7. Non-Functional Requirements

1. Performance:
   - P95 chat response under 5 seconds for moderate queries.
   - P95 chart render under 1.5 seconds for 10k points.
2. Reliability:
   - App crash-free sessions over 99.5%.
3. Security:
   - Credentials stored in OS keychain.
   - Sensitive local data encrypted at rest.
   - Hardened Electron config.
4. Scalability:
   - Worker-based execution to keep UI responsive under heavy workloads.

## 8. High-Level Architecture (No Required Server)

### 8.1 Desktop Client (TanStack + React)

1. TanStack Router for app routes and layouts.
2. TanStack Query for local async state and cache.
3. TanStack Table for result rendering.
4. Charting with ECharts or Vega-Lite.
5. Chat, query preview, and chart split-view workflow.

### 8.2 Electron Runtime (Main + Preload + Workers)

1. Main process:
   - Secure IPC broker.
   - Lifecycle, updates, native dialogs.
2. Preload bridge:
   - Typed allowlisted APIs exposed to renderer.
3. Worker pool:
   - Connector operations.
   - Long-running query execution.
4. Security defaults:
   - `contextIsolation=true`
   - `nodeIntegration=false`
   - navigation blocking + strict CSP.

### 8.3 Local Data and Secrets Layer

1. Local metadata store:
   - SQLite (prefer SQLCipher for encryption).
2. OS keychain:
   - Credentials, API keys, refresh tokens.
3. Local file cache:
   - Query/export artifacts with retention policy.

### 8.4 AI Layer (Pluggable)

1. Provider abstraction:
   - Cloud LLM provider via user-provided key.
   - Local model endpoint option (for example Ollama).
2. Prompt templates by connector type.
3. Guardrail policy engine before execution.
4. Confidence and explanation metadata per generated query.

## 9. Data Flow (Desktop MVP)

1. User selects source and enters a question in desktop app.
2. Local runtime fetches metadata context from connector.
3. Model adapter generates source-specific query.
4. Guardrail engine validates query and injects limits.
5. User approves or rejects generated query.
6. Connector executes approved query.
7. Result mapper normalizes output.
8. UI renders table + recommended chart.
9. Local audit and telemetry events are persisted.

## 10. Suggested Tech Stack

1. App shell:
   - Electron
   - electron-builder
   - electron-updater
2. Frontend:
   - React + TypeScript
   - TanStack Router/Query/Table
   - ECharts or Vega-Lite
3. Local runtime:
   - Node.js in Electron main/workers
   - Connector libraries (JDBC alternative Node drivers, Mongo, Neo4j, Qdrant clients)
4. Local storage/security:
   - SQLite/SQLCipher
   - OS keychain integration
5. Observability and quality:
   - OpenTelemetry local tracing
   - Playwright Electron E2E

## 11. MVP Milestones (14 Weeks)

1. Weeks 1-2: Desktop foundation
   - Monorepo setup, Electron shell, local storage, security baseline.
2. Weeks 3-4: Core connectors
   - Postgres/MySQL/Mongo connectors + metadata explorer.
3. Weeks 5-6: Chat-to-query core
   - Model adapters, prompts, guardrails, approval workflow.
4. Weeks 7-8: Visualization and exports
   - Unified results, chart recommendations, export workflows.
5. Weeks 9-10: Advanced connectors and file support
   - Neo4j, pgvector, Qdrant, CSV/JSON/Parquet ingestion.
6. Weeks 11-12: Hardening
   - Audit, security test suite, performance tuning, signing pipeline.
7. Weeks 13-14: Distribution readiness
   - Installer certification, auto-update validation, closed beta.

## 12. Success Metrics

1. Time to first insight under 10 minutes from install.
2. Query success rate above 85% without manual edits.
3. Median chart interaction above 60% on successful queries.
4. Desktop install success rate above 98%.
5. Crash-free sessions above 99.5%.

## 13. Risks and Mitigations

1. TanStack is not a backend runtime.
   - Mitigation: keep TanStack for UI/state; run data and connector logic in Electron main/workers.
2. Local machine variability impacts reliability.
   - Mitigation: strict supported OS matrix and diagnostics bundles.
3. Model quality variability across local/cloud options.
   - Mitigation: provider abstraction and configurable fallback strategy.
4. Desktop security surface (IPC, local files, key storage).
   - Mitigation: hardened Electron, typed IPC allowlist, keychain usage, security tests in CI.

## 14. Post-MVP Roadmap

### Phase 2

1. Optional cloud sync for shared chats and team workspaces.
2. Enterprise policy packs and SSO.
3. Managed connector bundles and observability dashboard.

### Phase 3

1. Linux desktop support.
2. Team collaboration with optional hosted control plane.
3. Plugin SDK and template marketplace.

## 15. Immediate Next Steps

1. Confirm local-first architecture as official V1 strategy.
2. Choose model strategy for MVP:
   - Cloud LLM with user key.
   - Local LLM endpoint.
   - Hybrid fallback.
3. Define typed IPC contract for source, chat, query, and export operations.
4. Build desktop proof of concept with one connector (Postgres).
5. Finalize packaging/signing approach for macOS and Windows.

## 16. Open Questions

1. Do we require internet only for model calls or also for updates/telemetry?
2. Is local encrypted SQLite enough, or do we need pluggable encrypted stores?
3. Which user personas require strict air-gapped mode in V1?
4. Should BYOK for LLM be mandatory in MVP?

---

This PRD intentionally removes mandatory server dependency for MVP while preserving a path for optional cloud capabilities later.
