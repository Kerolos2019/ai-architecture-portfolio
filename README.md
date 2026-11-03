# AI Architecture Portfolio

Coursework and capstone for the AI Architecture Design course.

| Folder | Contents |
| ------ | -------- |
| [docs/](docs/) | One document per session |
| [diagrams/](diagrams/) | Architecture diagrams |
| [adr/](adr/) | Architecture decision records |
| [capstone/](capstone/) | Final document and slides |

## Session 1 — Governance pack

The system used throughout the course is **Qamhawy**, the bilingual (Arabic/English)
AI assistant inside the Mahaseel farming platform: Egyptian farmers ask agronomic
questions by text, voice, or photo and get answers grounded in agronomist-approved
content.

The assignment was to put governance around that system before building it — the
rules it must obey, the decisions already made, and the checks that enforce both.
What was produced:

| Deliverable | Where | What it contains |
| --- | --- | --- |
| Governance pack | [docs/01-governance-pack.md](docs/01-governance-pack.md) | Five design principles (P1–P5) tied to business goals, a summary of the ADRs, the ARB review checklist with production triggers, and the design for a CI guardrail script |
| Architecture diagram | [diagrams/src/01-governance-architecture.drawio](diagrams/src/01-governance-architecture.drawio) ([PNG](diagrams/src/01-governance-architecture.drawio.png)) | The system with its trust boundary, showing which flows leave it and what PII each carries |
| 4 decision records | [adr/](adr/) | LLM hosting, STT provider, knowledge sync, and vector database — each with the options considered, the trade-off, and when to revisit |

The pack is the entry point; the diagram and the ADRs are what it refers to. Of the
four decisions, three are accepted and ADR-0002 (STT provider) is deliberately left
open pending a measurement spike, since vendor claims about Egyptian-dialect accuracy
can't settle it.

## Session 2 — Use-case flow

One use case drawn end to end: a farmer adds an input to an order and doesn't finish it,
and two hours later gets one WhatsApp reminder with two suitable in-stock alternatives
and a link that resumes the order.

| Deliverable | Where | What it contains |
| --- | --- | --- |
| Flow notes | [docs/02-use-case-flow.md](docs/02-use-case-flow.md) | The two options compared for step 11, the decision, the over-engineering check, and the P95 latency budget |
| Flow diagram | [diagrams/src/02-order-nudge-flow.drawio](diagrams/src/02-order-nudge-flow.drawio) ([PNG](diagrams/src/02-order-nudge-flow.drawio.png)) | A sequence view, one column per system, with the real-time path and the background ingestion and batch path kept apart |

The decision the use case turns on: the two alternatives come from a **real-time vector
database query** rather than a nightly precomputed top-K. On an asynchronous path behind
a 2-hour timer, latency isn't the constraint — freshness and cost are, and a precomputed
list goes stale enough to suggest fertiliser that is out of stock.
