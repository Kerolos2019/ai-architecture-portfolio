# Architecture Decision Records

`adr/` — one file per decision: `0001-record-architecture-decisions.md`, …

Each record covers one architecturally significant decision: the context that
forced it, the options weighed, what was chosen, and what that costs.

A decision earns an ADR if reversing it later would be expensive — model hosting,
retrieval strategy, where data lives, whether an agent is used at all, a boundary
between components, a constraint everything downstream inherits. Local,
reversible choices do not need one.

## Conventions

- **Filename**: `NNNN-short-title.md` in kebab-case, e.g. `0002-model-hosting.md`
- **Numbering**: sequential, never reused, never renumbered. Gaps are fine.
- **Immutable once accepted**: to change course, write a new ADR and set the old
  one's status to `Superseded by ADR-NNNN`. The old file stays.
- **Status**: `Proposed` → `Accepted` | `Rejected`, later possibly `Deprecated`
  or `Superseded`.
- Start every ADR from [0000-template.md](0000-template.md).

## Index

| ADR | Title | Status | Date |
| --- | ----- | ------ | ---- |
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted | 2026-10-17 |

## Decisions this course will likely force

A backlog, not a commitment — numbers get assigned when each one is written.

- Model hosting: managed API vs self-hosted, and the evidence behind it
- Model selection and fallback behaviour when the primary model is unavailable
- Chunking and embedding strategy for the real document set
- Vector store choice and how access control is enforced at retrieval time
- Arabic/English handling: one index or separate, and where language is detected
- Whether an agent is justified, and if so its tool surface and approval points
- Human-in-the-loop boundaries: what the system may never do unattended
- Data residency and retention under the applicable regional law
- Prompt and context logging: what is stored, for how long, and who can read it
- Evaluation gates that must pass before a release ships
- Rollback trigger and mechanism
