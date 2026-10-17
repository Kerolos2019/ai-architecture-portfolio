# AI Architecture Portfolio

Coursework and capstone for **AI Architecture Design** — designing production-grade
AI systems (LLMs, RAG, agents, security, evaluation) from whiteboard to enterprise.

One architecture is built up across the whole course and becomes the capstone.

## My scenario

> **Not chosen yet.** Decided in Session 1 (framing). Fill this in with two or
> three sentences: the organisation, the business problem, and who the users are.

- **Domain**: TBD — telecom / e-commerce / government / own approved domain
- **System in one line**: TBD
- **Primary users**: TBD
- **Languages**: TBD (Arabic, English, or both)

## Assumptions

Listed here so every decision in `adr/` can be read against them. Updated as the
course goes, not rewritten silently.

| # | Assumption | Why it matters | Confidence |
| - | ---------- | -------------- | ---------- |
| A1 | TBD | | |

## How to read this repo

```
ai-architecture-portfolio/
├── README.md        # this file: scenario, assumptions, how to read the repo
├── docs/            # one file per session: 01-framing.md, 02-building-blocks.md ...
├── diagrams/        # exported PNG/SVG + source files
├── adr/             # one file per decision: 0001-model-hosting.md ...
└── capstone/        # final architecture PDF + slides
```

| Path | Contents |
| ---- | -------- |
| [docs/](docs/) | One document per session — the written architecture work, numbered `NN-topic.md` |
| [diagrams/](diagrams/) | C4 diagrams: editable sources in `src/`, rendered PNG/SVG in `exports/` |
| [adr/](adr/) | Architecture Decision Records, one file per decision, numbered `NNNN-short-title.md` |
| [capstone/](capstone/) | The submitted deliverables: final architecture PDF and presentation slides |

Suggested order for a reviewer: this README → [docs/01-framing.md](docs/01-framing.md)
→ the C4 context diagram → the ADR index → the capstone document.

Each folder has a README with the conventions used inside it.

## Status

| Deliverable | State |
| ----------- | ----- |
| Scenario and framing | Not started |
| C4 diagrams (L1–L3) | Not started |
| ADRs | 1 (process only) |
| Cost estimate | Not started |
| Threat model | Not started |
| Evaluation and monitoring plan | Not started |
| Capstone document | Not started |
| Proof of concept (bonus) | Not started |
