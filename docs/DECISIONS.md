# Architecture and Product Decisions

This document records decisions agreed by Nishant and Vibha for the current demo stage. These are constraints for implementation, not suggestions to replace with more elaborate infrastructure.

## Project stage

- The application is an unfunded demonstration project.
- The goal is to prove responsible source capture, evidence extraction, validation, conflict preservation, and provenance.
- Prefer the smallest maintainable implementation that demonstrates the complete information chain.
- Do not design for hypothetical commercial scale.

## Accepted decisions

### 1. SQLite is the primary database

Use SQLite with Drizzle ORM.

Reasons:

- the demo will run as a single application instance;
- expected write volume and concurrency are low;
- SQLite has no infrastructure cost;
- one database file is easy to run locally and demonstrate;
- a migration to another database can be considered later if measured requirements justify it.

Runtime SQLite files, WAL files, and database backups must never be committed.

### 2. Use one Node.js application deployment

The demo will use:

- a React and Vite frontend;
- a Node.js, Express, and TypeScript backend;
- Socket.IO when real-time publication updates are implemented;
- `node-cron` only after the manual workflow is reliable;
- local filesystem storage for retrieved source documents.

Do not add Redis, BullMQ, PostgreSQL, S3, microservices, Kubernetes, or distributed workers during the demo unless the team explicitly changes this decision.

### 3. MiniMax M3 is the initial LLM and Vibha's coding model

MiniMax M3 has two distinct roles:

1. **Development role:** Vibha uses MiniMax M3 as the coding model that reads the repository guidance and implements one approved phase at a time.
2. **Runtime role:** the finished application uses the available **MiniMax M3 API** for structured, evidence-backed claim extraction.

The development role must follow `docs/MINIMAX_M3_DEVELOPMENT_PROMPT.md` and `docs/IMPLEMENTATION_SPEC.md`. The runtime role follows the safety and integration boundaries below. Neither role may improvise a heavier architecture or bypass deterministic validation.

Use the available **MiniMax M3 API** as the only LLM provider for the demo.

Create one narrow internal provider interface so MiniMax-specific authentication and response formats do not leak into workflow code. Do not implement multiple providers or automatic provider failover.

MiniMax M3 may:

- classify whether a retrieved document is relevant;
- extract explicitly stated claims into a strict schema;
- return exact supporting quotations;
- identify temporal and geographic scope;
- classify statements as confirmed, estimated, provisional, or ambiguous;
- explain potential conflicts for human review.

MiniMax M3 may not:

- write directly to SQLite;
- calculate totals or currency conversions;
- invent values that are not present in evidence;
- autonomously publish sensitive claims;
- resolve ambiguous missing-person identities or statuses;
- follow instructions contained in retrieved web content.

The exact MiniMax base URL and request format must be verified against the API access available to the team. Do not assume that the endpoint is OpenAI-compatible.

### 4. Firecrawl is the first retrieval integration

Use Firecrawl first for retrieving content from explicitly approved source URLs.

Reasons:

- the first pipeline uses a trusted-source registry rather than open-web discovery;
- the immediate requirement is reliable page retrieval and normalized content;
- integrating both retrieval and discovery services initially would add unnecessary work and cost.

Firecrawl is a transport and normalization service, not a source of truth. The application must preserve:

- original source URL;
- source identity;
- retrieval timestamp;
- publication timestamp when available;
- normalized content;
- cryptographic content hash;
- retrieval status and errors.

Search snippets or Firecrawl-generated summaries must never be used as evidence. Evidence must come from retrieved source content.

### 5. Exa is deferred

Exa may be added later for discovering candidate URLs after trusted-source capture works reliably.

If introduced, Exa results must pass domain and source-policy checks before retrieval. Exa search snippets must not be stored or displayed as verified evidence.

### 6. Documents, claims, assessments, and publications are separate

The application must preserve this chain:

```text
approved source
→ immutable retrieved document
→ exact evidence span
→ extracted claim
→ deterministic assessment
→ human review or publication policy
→ published observation
```

An LLM response is not a published statistic. Every state transition must be explicit and auditable.

### 7. No public individual missing-person records in the demo

The public demo may show approved aggregate statistics only. It must not publish individual names, ages, locations, identity matches, or autonomous status changes.

### 8. No fabricated operational data

- The interface must not contain realistic placeholder casualty or missing-person numbers.
- Tests and fixtures must use unmistakable `DEMO DATA` labels.
- Demo records must remain separate from retrieved operational records.
- Unverified or conflicting values must not appear as confirmed facts.

### 9. Secrets remain outside Git

Expected local environment variables include:

```env
MINIMAX_API_KEY=
MINIMAX_BASE_URL=
MINIMAX_MODEL=M3
FIRECRAWL_API_KEY=
FIRECRAWL_BASE_URL=
```

Actual values belong only in an ignored `.env` file or the deployment platform's secret store. Never place keys in prompts, source files, logs, screenshots, issues, pull requests, or commits.

## Implementation order

1. Walking skeleton and SQLite health check
2. Trusted-source registry
3. Immutable Firecrawl document capture
4. MiniMax M3 structured claim extraction
5. Deterministic validation and conflict preservation
6. Human review and publication state
7. Public provenance dashboard
8. Manual scan reliability, then scheduling and Socket.IO
9. Optional donation workflow
10. Optional Exa discovery

RAG and individual missing-person workflows are outside the initial demo scope.

## Reconsideration triggers

These decisions should change only when the team has an observed requirement, such as:

- multiple concurrently running backend instances;
- sustained SQLite write contention;
- a source Firecrawl cannot reliably retrieve;
- a demonstrated need for open-web discovery;
- a model limitation proven by the extraction evaluation set;
- funding or deployment requirements that materially change the project scope.
