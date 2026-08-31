# Lean Demo Architecture

## Context

This project is currently an unfunded demonstration. Its architecture should prove the central safety and provenance ideas without introducing infrastructure intended for a commercial, distributed service.

## Architectural principles

1. **Documents, claims and published observations are separate layers.**
2. **AI proposes structured interpretations; deterministic code validates them.**
3. **Every claim points to exact evidence in an immutable retrieved document.**
4. **Conflicts are stored and shown rather than hidden.**
5. **Sensitive publication decisions require human review.**
6. **The demo runs as one application deployment with one SQLite database.**

## Accepted external services

- **MiniMax M3** is the only initial LLM and is used only for structured, evidence-backed claim extraction.
- **Firecrawl** is the first retrieval API and may fetch only explicitly approved source URLs.
- **Exa** is deferred until the team demonstrates a need for open-web candidate discovery.
- Exact API base URLs and request formats must be verified from the access available to the team rather than assumed.

External services never write directly to SQLite. Firecrawl output is archived and hashed before extraction; MiniMax output is schema-validated and remains a candidate until deterministic checks and any required human review are complete.

## Proposed runtime

```text
One Node.js application
├── Express REST API
├── Socket.IO server
├── scheduled scan workflow
├── static React production build
├── SQLite database
└── local retrieved-document directory
```

This design avoids separate workers and queues. A database scan record and an in-process guard will prevent overlapping scans.

## Proposed repository structure

```text
Nepal_Flood/
├── client/
│   └── src/
├── server/
│   └── src/
│       ├── ai/
│       ├── db/
│       ├── routes/
│       ├── services/
│       └── workflows/
├── data/
│   └── documents/
├── docs/
├── .env.example
├── .gitignore
├── AGENTS.md
└── README.md
```

The implementation directories should be created by the relevant milestone rather than as empty scaffolding.

## Initial data entities

- `sources`: approved publishers and collection configuration
- `source_documents`: immutable versions of retrieved pages and reports
- `claims`: structured statements extracted from documents
- `evidence`: exact quotations supporting claims
- `assessments`: validation, corroboration and conflict results
- `published_statistics`: dashboard-safe observations
- `scan_runs`: scheduled workflow status and failures
- `audit_logs`: important automated and administrative actions

## Claim lifecycle

```text
discovered
→ extracted
→ validated
→ assessed
→ pending_review or publishable
→ published
→ superseded or retracted
```

No stage may be skipped when writing published data.

## Demo boundaries

The first vertical slice should support one aggregate metric from a small set of explicitly approved sources. It is complete only when a displayed value can be traced through:

```text
published observation
→ claim
→ exact evidence quotation
→ immutable source document
→ original URL
```

The initial demo will not include:

- public individual missing-person records;
- autonomous death or identity-status decisions;
- web-wide autonomous crawling;
- donation totals across incompatible contribution states;
- RAG;
- distributed job queues;
- multi-instance deployment.

## Future migration triggers

SQLite should be reconsidered only when observed requirements demand it, such as multiple concurrent application instances, sustained write contention, operational replication requirements, or substantially more complex geographic querying.
