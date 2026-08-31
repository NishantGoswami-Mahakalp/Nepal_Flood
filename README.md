# Nepal Flood Autonomous Relief & Intelligence Dashboard

A lightweight demonstration of an AI-assisted disaster-information pipeline for Nepal flood reporting.

The project is being developed by **Nishant Goswami** and **Vibha** to explore how trusted-source collection, structured extraction, provenance, deterministic validation, conflict preservation, and human review can support a responsible public dashboard.

> [!IMPORTANT]
> This repository is currently a **demonstration project**, not an official emergency-information service. It must not present fabricated or demo statistics as real disaster information.

## Intended workflow

```text
Approved sources
      ↓
Scheduled collection
      ↓
Immutable source documents
      ↓
Structured claim extraction
      ↓
Deterministic validation
      ↓
Conflict assessment / human review
      ↓
SQLite
      ↓
Express + Socket.IO
      ↓
React dashboard
```

AI may extract and interpret evidence, but it must not write arbitrary records directly to the database or autonomously publish sensitive information. Every displayed statistic must be traceable to an exact source and evidence quotation.

## Proposed demo stack

- React, Vite, TypeScript and Tailwind CSS
- Node.js, Express and Socket.IO
- SQLite with Drizzle ORM
- Zod validation
- `node-cron` for a single-process scheduler
- Local filesystem storage for retrieved source documents
- A provider-adapter interface for structured LLM extraction

The demo intentionally avoids Redis, distributed queues, microservices and paid infrastructure until real requirements justify them.

## Current status

The repository has been initialized with project documentation only. Application implementation will proceed incrementally after the initial architecture and milestone plan are approved.

## Safety principles

- Never fabricate disaster statistics.
- Clearly label all mock records as `DEMO DATA`.
- Store source URL, retrieval time and exact evidence for every claim.
- Preserve conflicting reports rather than silently choosing a value.
- Keep extracted claims separate from published observations.
- Require human review for sensitive or ambiguous changes.
- Do not expose individual missing-person records in the public demo.
- Treat fetched web content as untrusted input.

## Local data

Runtime data will be stored under `data/` and is ignored by Git:

```text
data/
├── app.db
└── documents/
```

Only placeholder files may be committed inside that directory.

## Documentation

- [Lean architecture](docs/ARCHITECTURE.md)
- [Contribution and agent rules](AGENTS.md)

## License

No license has been selected yet. Until one is added, normal copyright protections apply.
