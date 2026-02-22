# PolyStoreIQ PRD

## 1. Product Vision

Build a simple, trusted interface where a user provides a data store connection, asks questions in natural language, and gets:

1. Correct answers grounded in live data.
2. Instant visual representations of results.
3. A consistent experience across relational, NoSQL, graph, vector, and file-based sources.

Long-term goal: become the default "universal data conversation layer" for developers, architects, and non-technical business users.

## 2. Problem Statement

Teams use many storage technologies, each with different query languages, tools, and expertise requirements. This creates:

1. Slow insight generation.
2. Heavy dependence on specialists.
3. Fragmented analytics and inconsistent user experience.

Users need one interface that hides storage complexity while preserving trust, speed, and governance.

## 3. Target Users

1. Developers: quick exploration, debugging, ad hoc querying.
2. Architects/platform engineers: cross-store visibility, schema and data checks.
3. Analysts and business users: plain-language questions with visual outputs.
4. Data leaders: governance, usage tracking, and standardization.

## 4. Product Principles

1. Trust first: show generated query, confidence, and explainability.
2. Safe by default: read-only access for MVP, strict guardrails.
3. Fast time-to-value: connect and ask in minutes.
4. Unified UX: same interaction pattern across stores.
5. Extensible connectors: add sources without redesigning core.

## 5. MVP Scope (V1)

### 5.1 In Scope

1. Authentication: email/password or SSO-lite (Google/Microsoft) with workspace support.
2. Data source onboarding: secure credential entry and connection test.
3. Supported connectors (initial set):
   - Relational: PostgreSQL, MySQL
   - NoSQL: MongoDB
   - Graph: Neo4j
   - Vector: pgvector (Postgres), Qdrant
   - File-based: CSV, JSON, Parquet (upload and S3-compatible bucket)
4. Chat-to-query flow:
   - User asks question in natural language.
   - System generates query/pipeline/cypher/filter request.
   - System executes and returns structured result.
   - User sees and can approve/reject generated query.
   - Every query generated will have metrics attached for review.
5. Visualization:
   - Auto chart recommendation (table, bar, line, pie, scatter, graph view).
   - User can switch chart type.
   - Export chart image and result CSV.
6. Context panel:
   - Selected source, schema preview, recent queries.
7. Observability:
   - Query latency, token usage, error logs, prompt trace IDs.
8. Governance (MVP-level):
   - Read-only roles.
   - Audit logs for connection and query execution.

### 5.2 Out of Scope (MVP)

1. Write-back operations (insert/update/delete).
2. Auto schema migration or data transformation pipelines.
3. Enterprise-grade RBAC/ABAC depth.
4. On-prem deployment automation.
5. Marketplace for third-party connector plugins.

## 6. Core User Stories

1. As a developer, I can connect a Postgres DB and ask "top 10 customers by revenue this month" and see both SQL and chart.
2. As an architect, I can run similar intent across MongoDB and Neo4j and compare outputs.
3. As a business user, I can ask "show weekly signup trend by region" without knowing SQL.
4. As an admin, I can review who connected what source and what queries were executed.

## 7. Functional Requirements

1. Connection management:
   - Create, test, update, disable data source connections.
   - Encrypted secret storage.
2. Metadata discovery:
   - Pull schema/collections/indexes/labels for context grounding.
3. Query generation:
   - Model-based generation per connector type.
   - Deterministic post-processing and validation rules.
4. Query execution:
   - Connector-specific executor service.
   - Timeouts, row limits, and cancellation.
5. Result normalization:
   - Convert heterogeneous results into common tabular/graph payload schema.
6. Visualization recommendation engine:
   - Rule-based mapping from result shape to chart type.
7. Chat session memory:
   - Within-session context retention.
8. Audit and telemetry:
   - User action logs and system metrics.

## 8. Non-Functional Requirements

1. Performance:
   - P95 first response under 5 seconds for moderate queries.
   - P95 chart render under 1.5 seconds for up to 10k points.
2. Reliability:
   - 99.5% uptime target for MVP hosted service.
3. Security:
   - TLS in transit, AES-256 at rest for secrets.
   - Secret rotation support.
4. Compliance readiness:
   - SOC 2-ready controls roadmap (post-MVP).
5. Scalability:
   - Horizontal scaling for chat/query workers.

## 9. High-Level Architecture (Simple and Extensible)

### 9.1 Frontend (TanStack + React)

1. TanStack Router: app routes and nested layouts.
2. TanStack Query: API state management, polling, caching.
3. TanStack Table: result grids.
4. Visualization library: Apache ECharts or Vega-Lite.
5. Chat UI:
   - Conversation thread
   - Query preview/approval panel
   - Chart/result split view

### 9.2 Backend (Spring Boot)

1. API layer:
   - REST endpoints for auth, sources, chat, query, visualization metadata.
2. Orchestration layer:
   - Prompt builder
   - Model provider adapter
   - Guardrail/validation pipeline
3. Connector abstraction:
   - `Connector` interface with adapters per source type.
4. Execution engine:
   - Async query tasks, timeout and retry policy.
5. Metadata service:
   - Schema snapshot and refresh.
6. Audit and telemetry:
   - Structured events, metrics, traces.
7. Storage:
   - PostgreSQL for app metadata.
   - Redis for short-term session/cache.
   - Object storage for uploaded files.

### 9.3 AI Layer

1. LLM provider abstraction (start with one provider, keep pluggable).
2. Prompt templates per data type.
3. Safety filters:
   - Block destructive query intents.
   - Enforce read-only and max result limits.
4. Confidence and explainability:
   - Show model rationale summary and query confidence band.

## 10. Data Flow (MVP)

1. User selects source and enters prompt.
2. Backend fetches relevant schema context.
3. LLM generates source-specific query.
4. Guardrail pipeline validates and constrains query.
5. User review and optional approval.
6. Connector executes query.
7. Results normalized into common payload.
8. Chart recommendation generated and rendered.
9. Chat response, query, chart, and audit event persisted.

## 11. Suggested Tech Stack

1. Frontend:
   - React + TypeScript
   - TanStack Router/Query/Table
   - ECharts/Vega-Lite
   - Tailwind or CSS Modules
2. Backend:
   - Spring Boot 4 (Java 21)
   - Spring Security + OAuth2
   - JDBC + Mongo driver + Neo4j driver + Qdrant client
   - Flyway for migrations
3. Infra:
   - Single JAR build packaged together, can do native images with GraalVM if possible
   - OpenTelemetry + Prometheus + Grafana
   - Managed Postgres + Redis

## 12. MVP Milestones (12-Week Plan)

1. Weeks 1-2: foundation
   - Monorepo setup, auth, workspace model, source registry.
2. Weeks 3-4: core connectors
   - Postgres/MySQL/MongoDB connector + schema discovery.
3. Weeks 5-6: chat-to-query
   - LLM integration, prompt templates, validation and approval flow.
4. Weeks 7-8: results and visuals
   - Normalization layer, table view, auto chart suggestions.
5. Weeks 9-10: advanced connectors
   - Neo4j, pgvector/Qdrant, file upload and parse.
6. Weeks 11-12: hardening
   - Audit logs, observability, load/perf testing, beta release.

## 13. Success Metrics

1. Time to first insight: under 10 minutes from signup.
2. Query success rate: over 85% without manual edits.
3. Weekly active workspaces and retained users (week 4 retention over 35%).
4. Median chart interaction rate over 60% of successful queries.
5. Support tickets per active workspace below defined threshold.

## 14. Risks and Mitigations

1. Hallucinated or invalid queries
   - Mitigation: schema-grounded prompts, validators, human approval.
2. Connector complexity explosion
   - Mitigation: strict connector interface and phased onboarding.
3. Performance bottlenecks on large datasets
   - Mitigation: row limits, sampling, async jobs.
4. Security concerns from enterprise users
   - Mitigation: encrypted secrets, auditability, private network options.

## 15. Post-MVP Roadmap to Commercial SaaS

### Phase 2: Product-Market Fit (Months 4-8)

1. Expand connectors (Snowflake, BigQuery, Elasticsearch, Cassandra, S3 lakehouse).
2. Team collaboration:
   - Shared chats, saved analyses, dashboard sharing.
3. Better semantics:
   - Business glossary, metric definitions, semantic layer mapping.
4. Enterprise auth:
   - SAML, SCIM, advanced workspace controls.
5. Billing and plans:
   - Usage-based + seat-based hybrid.

### Phase 3: Enterprise and Scale (Months 9-18)

1. Deployment options:
   - Multi-tenant cloud, single-tenant, self-hosted.
2. Governance:
   - Fine-grained RBAC/ABAC, data masking, policy engine.
3. Workflow automation:
   - Scheduled insights, alerts, Slack/Teams integration.
4. Marketplace:
   - Connector/plugin SDK and partner ecosystem.
5. Localization:
   - Multi-language UX and prompt handling.

### Phase 4: Global Commercial Platform (18+ Months)

1. AI analyst mode:
   - Multi-step investigation and cross-source reasoning.
2. Natural language to dashboard generation.
3. Industry packs:
   - Prebuilt templates for finance, healthcare, retail, telecom.
4. Compliance suite:
   - SOC 2 Type II, ISO 27001, GDPR tooling, regional residency options.

## 16. Commercialization Plan

1. ICP focus (initial):
   - Mid-size software teams with heterogeneous data stacks.
2. Go-to-market:
   - Product-led growth + technical content + integrations.
3. Pricing model (initial hypothesis):
   - Free tier (limited sources and messages)
   - Pro tier (higher limits and collaboration)
   - Enterprise (governance, SSO, private deployment)
4. Sales motion:
   - Self-serve for SMB, assisted for enterprise.

## 17. Immediate Next Steps (Execution Checklist)

1. Finalize MVP connector list and exact data sources for beta customers.
2. Decide first LLM provider and fallback strategy.
3. Create API contracts for chat, sources, query preview, and visualization payload.
4. Build clickable UX prototype in TanStack frontend.
5. Implement Spring Boot connector abstraction with Postgres first.
6. Run 10 design partner interviews and map top 10 questions users ask.
7. Define beta acceptance criteria and launch plan.

## 18. Open Questions

1. Should query approval be mandatory for all users or role-based?
2. Is file ingestion limited to session context, or persisted with indexing?
3. Which visualization types are mandatory for v1 industry use cases?
4. What is the preferred hosting model for first paying customers?
5. How much cross-source query federation is needed in MVP versus post-MVP?

---

This PRD is intentionally practical for execution while preserving a strong path to enterprise-grade SaaS scale.
