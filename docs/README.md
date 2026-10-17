# Session documents

`docs/` — one file per session: `01-framing.md`, `02-building-blocks.md`, …

Each document is the written architecture work for that session's assignment,
and together they build up to the [capstone](../capstone/).

## Conventions

- **Filename**: `NN-topic.md`, numbered by session: `01-framing.md`,
  `02-building-blocks.md`, …
- Start from [00-session-template.md](00-session-template.md).
- A document describes **what the architecture is**. The reasoning behind a
  choice goes in an [ADR](../adr/) and is linked from here — do not duplicate the
  argument in both places.
- Diagrams are not pasted inline; they live in [diagrams/](../diagrams/) and are
  referenced by relative link so there is one source of truth.
- Mark anything unresolved as `TBD` or `OPEN QUESTION:` rather than writing a
  confident guess. Open questions are a legitimate architecture output.

## Index

| Session | Document | Status |
| ------- | -------- | ------ |
| 01 | [Framing the problem](01-framing.md) | Skeleton |
| 02 | [Building blocks](02-building-blocks.md) | Skeleton |

Rename and extend as the real syllabus lands. Topics the course covers, which
will each want a document: model selection (API vs self-hosted), RAG pipeline
design, access control in retrieval, agents and guardrails, threat modelling,
regional compliance, cost, evaluation and monitoring.
