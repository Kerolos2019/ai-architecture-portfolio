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
