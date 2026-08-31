# Application Implementation Specification

This is the implementation contract for the Nepal Flood demo. Coding models and contributors must follow it together with `AGENTS.md`, `docs/ARCHITECTURE.md`, and `docs/DECISIONS.md`.

When this document conflicts with an older aspirational product prompt, this document controls the demo implementation. Any change to these boundaries must be recorded as a reviewed decision before code changes.

## 1. Product proof

The demo must prove one trustworthy end-to-end chain:

```text
approved source
→ retrieved source document
→ immutable local snapshot and content hash
→ exact evidence quotation
→ structured candidate claim
→ deterministic validation and assessment
→ human publication decision
→ public dashboard observation
```

The dashboard is not the primary technical achievement. Traceability through this chain is.

## 2. Current scope

### Included

- approved source registry;
- Firecrawl retrieval from approved URLs;
- immutable source-document metadata and local content files;
- MiniMax M3 structured extraction;
- exact evidence validation;
- deterministic claim validation;
- duplicate and conflict preservation;
- local human review;
- published aggregate statistics;
- provenance-first React dashboard;
- historical changes;
- scan and audit activity;
- manual scans before scheduled scans;
- Socket.IO updates after publication changes.

### Excluded

- public individual missing-person records;
- autonomous death or identity-status decisions;
- RAG;
- chatbot interface;
- open-web crawling;
- Exa integration until explicitly approved;
- autonomous donation totals in the core demonstration;
- Redis, BullMQ, PostgreSQL, S3, microservices, or distributed workers;
- multiple LLM provider implementations;
- multi-instance deployment;
- unreviewed public deployment of local admin endpoints.

## 3. Technology contract

Use:

- Node.js and npm workspaces;
- TypeScript with strict mode;
- React and Vite;
- Express;
- SQLite;
- Drizzle ORM and generated SQL migrations;
- Zod at environment, HTTP, external API, persisted JSON, and LLM boundaries;
- Vitest;
- Supertest for API integration tests;
- Testing Library for React behavior tests;
- Socket.IO only in the real-time milestone;
- `node-cron` only after manual scans are proven reliable;
- native `fetch` unless a dependency demonstrates immediate value;
- local filesystem storage under ignored `data/documents/`.

Do not add a dependency when a small standard-library implementation is clearer. Every new dependency must be used in the same milestone in which it is introduced.

## 4. Repository structure

Create directories only when their milestone begins. The intended completed structure is:

```text
Nepal_Flood/
├── client/
│   ├── src/
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   ├── schemas.ts
│   │   │   └── *.test.ts
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   ├── provenance/
│   │   │   ├── sources/
│   │   │   ├── statistics/
│   │   │   └── system/
│   │   ├── pages/
│   │   │   ├── OverviewPage.tsx
│   │   │   ├── ReviewPage.tsx
│   │   │   ├── SourcesPage.tsx
│   │   │   └── StatisticsPage.tsx
│   │   ├── test/
│   │   │   └── setup.ts
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── styles.css
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── vitest.config.ts
├── server/
│   ├── drizzle/
│   ├── src/
│   │   ├── config/
│   │   │   └── env.ts
│   │   ├── db/
│   │   │   ├── client.ts
│   │   │   ├── migrate.ts
│   │   │   └── schema.ts
│   │   ├── domain/
│   │   │   ├── assessment.ts
│   │   │   ├── claim.ts
│   │   │   ├── document.ts
│   │   │   ├── publication.ts
│   │   │   ├── scan.ts
│   │   │   └── source.ts
│   │   ├── integrations/
│   │   │   ├── firecrawl/
│   │   │   │   ├── client.ts
│   │   │   │   ├── schemas.ts
│   │   │   │   └── types.ts
│   │   │   └── minimax/
│   │   │       ├── client.ts
│   │   │       ├── prompts.ts
│   │   │       ├── schemas.ts
│   │   │       └── types.ts
│   │   ├── middleware/
│   │   │   ├── errorHandler.ts
│   │   │   ├── requestId.ts
│   │   │   └── validate.ts
│   │   ├── repositories/
│   │   │   ├── assessmentRepository.ts
│   │   │   ├── auditRepository.ts
│   │   │   ├── claimRepository.ts
│   │   │   ├── documentRepository.ts
│   │   │   ├── publicationRepository.ts
│   │   │   ├── scanRepository.ts
│   │   │   └── sourceRepository.ts
│   │   ├── routes/
│   │   │   ├── activity.ts
│   │   │   ├── dashboard.ts
│   │   │   ├── documents.ts
│   │   │   ├── health.ts
│   │   │   ├── review.ts
│   │   │   ├── scans.ts
│   │   │   ├── sources.ts
│   │   │   └── statistics.ts
│   │   ├── services/
│   │   │   ├── assessmentService.ts
│   │   │   ├── documentStorageService.ts
│   │   │   ├── extractionService.ts
│   │   │   ├── publicationService.ts
│   │   │   ├── retrievalService.ts
│   │   │   ├── sourcePolicyService.ts
│   │   │   └── validationService.ts
│   │   ├── workflows/
│   │   │   └── runScan.ts
│   │   ├── app.ts
│   │   └── server.ts
│   ├── package.json
│   ├── tsconfig.json
│   └── vitest.config.ts
├── data/
│   └── documents/
├── docs/
├── .env.example
├── .gitignore
├── AGENTS.md
├── package-lock.json
├── package.json
└── README.md
```

### Layer rules

- `routes/` translates HTTP input/output only. Routes do not call Drizzle directly.
- `services/` implements application rules and coordinates repositories/integrations.
- `repositories/` is the only normal location for database queries.
- `integrations/` owns external API details and never publishes data.
- `domain/` owns enums, Zod schemas, and pure rules with no network or database access.
- `workflows/` coordinates complete scans through services.
- React components call typed API functions, not raw URLs scattered through components.
- Tests live beside the behavior they test using `*.test.ts` or `*.test.tsx`.

## 5. Root scripts

The completed root `package.json` should expose stable commands:

```text
npm run dev              start client and server development processes
npm test                 run all deterministic tests without live paid APIs
npm run typecheck        type-check all workspaces
npm run build            build all workspaces
npm run db:generate      generate Drizzle migrations
npm run db:migrate       apply migrations to the configured SQLite database
npm run scan             run one manual scan
npm run verify           run tests, type checks, builds, and diff checks
```

Live Firecrawl or MiniMax smoke tests must use separately named commands and must never run under `npm test`.

## 6. Configuration contract

All environment input is parsed once in `server/src/config/env.ts`. Application modules receive typed configuration rather than reading `process.env` directly.

Expected variables:

```env
NODE_ENV=development
PORT=3000
CLIENT_ORIGIN=http://localhost:5173
DATABASE_URL=file:./data/app.db
DEMO_MODE=true
AUTO_PUBLISH_ENABLED=false
ADMIN_API_ENABLED=true
SCHEDULER_ENABLED=false
SCAN_INTERVAL_MINUTES=30
SCAN_MAX_URLS=10
FETCH_TIMEOUT_MS=30000
FETCH_MAX_CONTENT_BYTES=5000000
MINIMAX_API_KEY=
MINIMAX_BASE_URL=
MINIMAX_MODEL=M3
FIRECRAWL_API_KEY=
FIRECRAWL_BASE_URL=
EXA_API_KEY=
```

Rules:

- empty optional keys become `undefined`;
- booleans are parsed explicitly, not with truthiness;
- URLs use Zod URL validation;
- numeric limits have safe minimums and maximums;
- production startup fails if unsafe local admin functionality is enabled without approved protection;
- configuration errors identify variable names but never print secret values.

## 7. SQLite conventions

- Use text UUIDs for entity IDs.
- Store timestamps as UTC ISO-8601 text.
- Store booleans as SQLite integers through Drizzle.
- Store decimal values as normalized decimal strings where precision matters; do not use floating point for money.
- Persist JSON only as text validated by a corresponding Zod schema when read and written.
- Enable foreign keys for every connection.
- Use transactions for multi-table state transitions.
- Never edit an already-applied migration; add a new migration.
- Never overwrite historical source documents, claims, assessments, or publication decisions.

## 8. Database schema contract

Names may be adjusted only if the model documents why and receives approval. Required semantics may not be removed.

### `system_metadata`

```text
key                 text primary key
value               text not null
updated_at          text not null
```

### `sources`

```text
id                  text primary key
name                text not null
canonical_base_url  text not null unique
source_type         text not null
trust_tier          text not null
authority_scope     text not null
geographic_scope    text
active              integer not null default 1
created_at          text not null
updated_at          text not null
last_checked_at     text
```

Allowed `source_type` values:

```text
government
international_organization
ngo
news
other
```

`trust_tier` does not prove a claim is true. It describes source-policy priority.

### `source_targets`

```text
id                  text primary key
source_id           text not null references sources(id)
url                 text not null
canonical_url       text not null unique
target_type         text not null
active              integer not null default 1
created_at          text not null
updated_at          text not null
last_checked_at     text
```

Allowed target types:

```text
page
feed
report_index
```

A source record approves a publisher. A source-target record approves a concrete retrieval URL. Firecrawl may retrieve only active targets or explicitly reviewed child URLs discovered from an active report index. Same-domain membership alone is not authorization to crawl arbitrary pages.

### `disaster_events`

```text
id                  text primary key
name                text not null
country_code        text not null default NP
event_type          text not null
status              text not null
start_date          text not null
end_date            text
description         text
created_at          text not null
updated_at          text not null
```

Every claim belongs to one explicit disaster event. Do not compare claims from different events or silently treat the entire monsoon season as one event.

### `scan_runs`

```text
id                  text primary key
disaster_event_id   text not null references disaster_events(id)
trigger_type        text not null
status              text not null
started_at          text not null
completed_at        text
sources_attempted   integer not null default 0
sources_succeeded   integer not null default 0
new_documents       integer not null default 0
claims_extracted    integer not null default 0
error_summary_json  text
```

Allowed status values:

```text
queued
running
completed
completed_with_errors
failed
cancelled
```

### `source_documents`

```text
id                  text primary key
source_id           text not null references sources(id)
scan_run_id         text references scan_runs(id)
original_url        text not null
canonical_url       text not null
final_url           text not null
source_title        text
published_at        text
retrieved_at        text not null
content_hash        text not null
content_path        text not null
content_type        text
language            text
parser_version      text not null
retrieval_status    text not null
retrieval_error     text
```

Required uniqueness prevents duplicate storage of the same source, canonical URL, and content hash.

### `claims`

```text
id                    text primary key
disaster_event_id     text not null references disaster_events(id)
document_id           text not null references source_documents(id)
metric                 text not null
value_text             text not null
value_number            integer
unit                    text not null
qualifier               text not null
geographic_scope_json   text not null
temporal_scope_json     text not null
reported_at             text
status                  text not null
extractor_model         text not null
prompt_version          text not null
schema_version          text not null
extraction_key          text not null unique
created_at              text not null
```

`value_text` preserves exactly what the source stated. `value_number` is nullable and exists only when deterministic parsing succeeds.

Allowed claim statuses:

```text
candidate
invalid
validated
assessed
pending_review
publishable
published
superseded
retracted
rejected
```

### `evidence_spans`

```text
id                  text primary key
claim_id            text not null references claims(id)
document_id         text not null references source_documents(id)
exact_quote         text not null
start_offset        integer
end_offset          integer
page_number         integer
section_heading     text
created_at          text not null
```

The exact quotation must be found verbatim in normalized document content before the evidence is accepted.

### `assessments`

```text
id                      text primary key
claim_id                text not null references claims(id)
source_authority         text not null
evidence_match           text not null
freshness_status         text not null
corroboration_count      integer not null default 0
independence_status      text not null
conflict_status          text not null
assessment_status        text not null
reason                   text not null
rules_version            text not null
created_at               text not null
```

Do not store model-invented decimal confidence as proof. Store explainable assessment factors.

### `conflict_sets`

```text
id                  text primary key
metric              text not null
scope_key           text not null
status              text not null
created_at          text not null
resolved_at         text
```

### `conflict_members`

```text
conflict_set_id     text not null references conflict_sets(id)
claim_id            text not null references claims(id)
primary key         (conflict_set_id, claim_id)
```

### `publication_decisions`

```text
id                  text primary key
claim_id            text not null references claims(id)
decision            text not null
decision_type       text not null
reviewer_name       text
reason              text
created_at          text not null
```

Allowed decisions:

```text
publish
reject
supersede
retract
```

Allowed decision types:

```text
human
automatic_policy
```

`automatic_policy` remains disabled during the initial demo.

### `published_statistics`

```text
id                  text primary key
disaster_event_id   text not null references disaster_events(id)
claim_id            text not null references claims(id)
metric              text not null
value_text          text not null
value_number        integer
unit                text not null
scope_key           text not null
published_at        text not null
superseded_at       text
retracted_at        text
current             integer not null default 1
```

Only the publication service may write this table, and only within the same transaction as a valid publication decision and audit event.

### `audit_logs`

```text
id                  text primary key
actor_type          text not null
actor_name          text not null
action              text not null
entity_type         text not null
entity_id           text
status              text not null
details_json        text
created_at          text not null
```

Audit details must not contain API keys, complete prompts, private data, or raw exception stacks.

## 9. Metric registry and canonical scopes

Metrics are controlled domain identifiers, not arbitrary model strings. The initial registry is:

```text
deaths                people, sensitive, human review required
missing_persons       people, sensitive, human review required
rescued_people        people, aggregate
people_affected       people, aggregate
households_affected   households, aggregate
affected_districts    districts, aggregate
shelters_operational  shelters, aggregate
```

Each registry entry defines canonical unit, whether integer parsing is allowed, minimum value, sensitivity, publication requirements, and compatible qualifiers. MiniMax may propose only a registered metric. Adding a metric requires a reviewed registry and test change; the model must not create metric names dynamically from source wording.

A claim is not identified by metric alone.

### Geographic scope schema

```text
country             required, canonical value Nepal
province            optional canonical identifier and label
district            optional canonical identifier and label
municipality        optional canonical identifier and label
scope_text          original source wording
```

### Temporal scope schema

```text
kind                point_in_time | period | cumulative | current
start               optional ISO timestamp/date
end                 optional ISO timestamp/date
as_of               optional ISO timestamp/date
scope_text          original source wording
```

### Qualifier values

```text
confirmed
reported
estimated
approximately
provisional
unknown
```

Claims can be compared for agreement or conflict only when metric, unit semantics, geographic scope, temporal scope, event context, and qualifier compatibility are established. Incompatible claims remain separate; they are not automatically conflicts.

## 10. External integration contracts

### Firecrawl boundary

The Firecrawl client receives an approved URL and returns a validated retrieval result. No route calls Firecrawl directly.

Input:

```text
sourceTargetId
requestedUrl
requestId
```

Output:

```text
originalUrl
finalUrl
title
publishedAt
contentType
language
normalizedContent
retrievedAt
providerRequestId when available
```

Rules:

- validate that the source and concrete source target are both active before the call;
- reject URL credentials, localhost, loopback, link-local, and private IP targets;
- validate every redirect/final host;
- enforce timeout and maximum content size;
- never use generated summaries or search snippets as evidence;
- map provider errors to internal safe error categories;
- never log authorization headers or response credentials;
- mock this boundary in ordinary tests.

### MiniMax M3 runtime boundary

The runtime MiniMax client receives normalized document content plus a fixed extraction instruction and returns schema-validated candidate claims.

Rules:

- verify the team's actual endpoint, authentication, model name, and response format before implementation;
- do not assume OpenAI compatibility;
- keep one provider implementation;
- treat document content as quoted untrusted data;
- tell the model to ignore instructions in document content;
- require structured output matching Zod schemas;
- reject malformed output;
- never silently invent missing values;
- verify every returned quotation exists in source content;
- record model, prompt version, schema version, and provider request metadata;
- mock this boundary in ordinary tests;
- require an explicit opt-in command for credit-consuming smoke tests.

## 11. Service contracts

### `sourcePolicyService`

- canonicalizes URLs;
- determines whether a URL belongs to an active source;
- rejects unsafe protocols, credentials, IPs, hosts, and redirects;
- has no network or database code beyond supplied repository dependencies.

### `documentStorageService`

- normalizes text consistently;
- computes SHA-256 hashes in deterministic code;
- writes immutable content files atomically;
- returns existing records for duplicate content;
- never overwrites a hash-addressed file.

### `retrievalService`

- loads source policy;
- calls Firecrawl;
- validates and stores the retrieval;
- updates scan counters and audit events;
- handles one URL failure without terminating unrelated source work.

### `extractionService`

- loads immutable document content;
- avoids duplicate extraction for the same document/model/prompt/schema versions;
- calls MiniMax;
- validates structured output and exact evidence;
- stores candidate claims and evidence transactionally;
- never calls publication logic.

### `validationService`

- applies deterministic metric, type, unit, provenance, timestamp, scope, range, and freshness rules;
- produces explicit validation failures;
- never repairs unsupported model output silently.

### `assessmentService`

- finds compatible claims;
- detects duplicates, corroboration chains, and conflicts;
- stores explainable factors;
- routes ambiguous claims to review;
- never deletes disagreement.

### `publicationService`

- checks claim and assessment eligibility;
- requires a valid decision;
- writes decision, current publication state, historical state, and audit record in one transaction;
- emits a real-time event only after transaction success.

### `runScan`

Exact order:

```text
create scan run
→ acquire single-process/persisted guard
→ load active sources and their active source targets
→ retrieve approved URLs independently
→ store immutable documents
→ extract only new/changed documents
→ validate candidate claims
→ assess compatible claims
→ leave sensitive/ambiguous claims pending review
→ complete scan counters/status
→ release guard
```

One source failure must not abort successful sources. Re-running unchanged content must not duplicate documents or claims.

## 12. API contract

All responses use JSON. Errors have a stable safe shape:

```json
{
  "error": {
    "code": "STABLE_CODE",
    "message": "Safe explanation",
    "requestId": "..."
  }
}
```

Do not return stack traces, database paths, provider payloads, or secrets.

### Public routes

```text
GET /api/health
GET /api/events
GET /api/dashboard
GET /api/statistics
GET /api/statistics/history
GET /api/updates
GET /api/sources
GET /api/activity
GET /api/documents/:id/provenance
```

Public document provenance returns safe metadata and exact evidence used by published claims, not arbitrary raw files.

### Local demo admin routes

```text
POST  /api/admin/events
PATCH /api/admin/events/:id
POST  /api/admin/sources
PATCH /api/admin/sources/:id
POST  /api/admin/sources/:id/activate
POST  /api/admin/sources/:id/deactivate
POST  /api/admin/source-targets
PATCH /api/admin/source-targets/:id
POST  /api/admin/source-targets/:id/activate
POST  /api/admin/source-targets/:id/deactivate
POST  /api/admin/scans
GET   /api/admin/scans/:id
GET   /api/admin/review
POST  /api/admin/review/:claimId/publish
POST  /api/admin/review/:claimId/reject
POST  /api/admin/review/:claimId/supersede
POST  /api/admin/review/:claimId/retract
```

Admin routes are local-demo functionality until authentication is explicitly implemented. The server must not pretend they are production-safe.

## 13. HTTP and security rules

- Validate params, query strings, and bodies with Zod.
- Apply strict JSON body-size limits.
- Configure CORS from validated environment configuration.
- Add secure headers before public deployment.
- Add rate limits before public deployment.
- Generate a request ID for every request.
- Sanitize or safely render source evidence; never inject source HTML.
- External links use safe `rel` attributes.
- Avoid logging request bodies on admin/provider routes.
- Secrets must never appear in serialized errors.
- Retrieved content is untrusted regardless of source trust tier.

## 14. UI information architecture

### Initial walking skeleton

- product title;
- prominent `DEMO` badge;
- not-an-official-service warning;
- API status;
- SQLite status;
- no operational figures.

### Provenance dashboard

Navigation:

```text
Overview
Statistics
Sources
Latest Updates
Review (local admin only)
AI Activity
```

Every published statistic displays:

- metric label;
- value and unit;
- qualifier;
- geographic scope;
- temporal/as-of scope;
- source name and link;
- source reporting time;
- retrieval time;
- publication/assessment wording;
- stale, conflict, superseded, or retracted state when applicable;
- access to exact supporting evidence.

Do not display `LIVE` for periodic polling. Prefer `Last scan`, `Data current through`, and `Last publication`.

### Required UI states

Every data view must handle:

```text
loading
empty
success
stale
degraded/error
```

The empty state must not insert example operational figures.

## 15. Test contract

### Tests that never use paid APIs

The default suite must mock Firecrawl and MiniMax and cover:

- environment parsing;
- URL normalization and source allowlisting;
- private-network and unsafe redirect rejection;
- content hashing and duplicate retrieval;
- extraction schema validation;
- exact evidence verification;
- prompt-injection document fixtures;
- duplicate extraction idempotency;
- compatible versus incompatible claim scopes;
- conflict preservation;
- publication eligibility;
- rejection, supersession, and retraction history;
- API success and safe error responses;
- React loading, empty, stale, success, and error states;
- scheduler overlap and stale guard recovery when scheduling is introduced.

### Fixtures

- Every artificial document begins with `DEMO DATA`.
- Use obviously synthetic names and values.
- Do not reuse realistic numbers from product-planning examples.
- Maintain fixtures under a test-only directory.
- Never mix fixture records with an operational database.

### Live smoke tests

Live API tests:

- are never part of `npm test`;
- require an explicit environment flag;
- run against one approved non-sensitive test URL/document;
- print cost-safe status, not complete provider responses;
- skip cleanly when credentials are absent.

## 16. Logging and audit rules

Operational logs should be structured and contain:

```text
level
timestamp
requestId or scanRunId
event name
safe entity identifiers
duration
status
safe error category
```

Never log:

- API keys or authorization headers;
- complete `.env` values;
- full MiniMax prompts/responses in normal logs;
- raw private data;
- database contents;
- arbitrary retrieved HTML;
- stack traces in public responses.

## 17. Development behavior

For every phase:

1. Read controlling documents.
2. Inspect current files, dependencies, tests, and Git status.
3. Propose a narrow milestone and exact file list.
4. Wait for approval.
5. Write a failing test for new behavior.
6. Run it and confirm the expected failure.
7. Implement the minimum behavior.
8. Run the focused test.
9. Refactor only while green.
10. Run all tests, type checks, and builds.
11. Inspect the diff and staged files.
12. Stop at the phase boundary.

Do not claim success without actual command output. Do not substitute a stub, TODO, or plausible output for a working artifact.

## 18. Definition of done for the demo

The demo is done when:

- a new developer can follow the README to start it;
- the SQLite schema initializes from migrations;
- an approved URL is captured through Firecrawl;
- source content is stored immutably with a SHA-256 hash;
- MiniMax M3 extracts a strict candidate claim with exact evidence;
- deterministic validation rejects malformed or unsupported output;
- conflicts remain visible;
- a human can publish an eligible aggregate claim;
- the public dashboard displays only published data;
- the displayed value links back to exact evidence and source metadata;
- repeated scans are idempotent;
- tests, type checks, and builds pass;
- no credentials, runtime databases, retrieved documents, or logs are tracked;
- documentation explains setup, limitations, reset, backup, and the non-official demo status.
