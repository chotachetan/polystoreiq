# PolyStoreIQ

PolyStoreIQ is a desktop-first product that lets users connect heterogeneous data sources, ask questions in natural language, preview the generated query, approve execution, and get normalized tabular + visual results.

The goal is a trusted "universal data conversation layer" across relational, NoSQL, and file-based stores, while keeping execution safe, explainable, and fast.

## Product Vision

- One interface for developers, architects, analysts, and business users.
- Correct answers grounded in live data.
- Consistent UX across different storage technologies.
- Trust-first execution with query preview, approval, and auditability.

## Locked MVP Scope (Desktop)

- Platform: Electron desktop app, macOS-first beta.
- Workspace model: single-user local workspace.
- Access policy: read-only query execution only.
- Initial connectors:
  - PostgreSQL
  - MySQL
  - MongoDB
  - CSV/JSON/Parquet (via DuckDB)
- Deferred connectors: Neo4j, pgvector, Qdrant.

## Core User Flow

1. Add and test a source connection.
2. Ask a natural language question.
3. Generate source-specific query draft.
4. Validate via guardrails (read-only, limits, risk checks).
5. User approves with short-lived token.
6. Execute query through connector.
7. Normalize result payload.
8. Render table + recommended chart.
9. Persist chat, query history, telemetry, and audit events.

## Architecture Summary

### Runtime

- Renderer: React + TanStack (Router/Query/Table)
- Desktop shell: Electron main + preload bridge
- Engine: local Node.js orchestration and connector execution
- Local data plane:
  - SQLite (`better-sqlite3`) for app state
  - DuckDB for file analytics only
  - OS keychain for credentials

### Planned Monorepo Layout

- `apps/desktop` (Electron main + preload)
- `apps/renderer` (UI)
- `packages/engine` (orchestration, guardrails, execution)
- `packages/connectors` (source adapters)
- `packages/shared-types` (IPC and domain contracts)

## Trust, Security, and Governance

- Renderer isolation (`contextIsolation: true`, `nodeIntegration: false`)
- Strict typed IPC contracts and payload validation
- Mandatory approval before execution
- Read-only enforcement with destructive query blocking
- Row limits, timeouts, and cancellation
- No plaintext credentials in SQLite or logs
- Audit events for connection and query actions

## Performance Targets (MVP)

- P95 first response: under 5 seconds (moderate queries)
- P95 chart render: under 1.5 seconds (up to 10k points)
- Stage metrics: p50/p95 for generate, validate, execute, normalize, render

## Delivery Roadmap (12 Weeks)

1. Sprint 1: desktop foundation, SQLite baseline, secure IPC, source registry, CI bootstrap
2. Sprint 2: Postgres/MySQL/Mongo connectors + metadata discovery
3. Sprint 3: chat-to-query generation, guardrails, approval workflow, telemetry
4. Sprint 4: normalization, table rendering, chart recommendation + export
5. Sprint 5: file ingestion and DuckDB analytics connector
6. Sprint 6: hardening, audit pipeline, resilience, release packaging, UAT

## Success Metrics

- Time to first insight: under 10 minutes from signup/onboarding
- Query success rate: over 85% without manual edits
- Week-4 workspace retention: over 35%
- Chart interaction rate: over 60% of successful queries

## Current Repository State

This repository currently contains product and engineering planning documents (not the full implementation codebase yet).

Primary references:

- `polystoreiq-prd.md`
- `polystoreiq-architecture-guide.md`
- `sprint-plan.md`

## Notes on Scope Evolution

The PRD includes an earlier high-level cloud/service framing. The architecture and sprint plan lock the MVP implementation direction to a local-first Electron desktop product. This README reflects that locked MVP baseline.

## License

Licensed under the terms in `LICENSE`.
