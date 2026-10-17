# Capstone

The final deliverable: one architecture document as **PDF** plus the
**slides** used to present it. Everything here is assembled from the rest of the
repo — this folder holds the submitted artefacts, not new thinking.

```
capstone/
├── architecture.md   # editable source of the final document
├── architecture.pdf  # exported, submitted
└── slides/           # deck source + exported PDF
```

## Conventions

- `architecture.md` is the source of truth; the PDF is an export and is
  re-generated whenever the Markdown changes, in the same commit.
- Diagrams are **not** redrawn here. They are referenced from
  [../diagrams/](../diagrams/) and embedded at export time, so the capstone and
  the session documents can never disagree about what the architecture is.
- Decisions are **not** re-argued here. State what was chosen and link the
  [ADR](../adr/) that carries the reasoning.
- The document should read standalone for someone who has not seen
  [../docs/](../docs/) — summarise, then link for depth.
- Name exports with the submission date if more than one version is submitted:
  `architecture-2026-12-05.pdf`.

## What the document must cover

Derived from the course deliverables; each row points at where the work already
lives so the capstone is a synthesis, not a rewrite.

| Section | Source |
| ------- | ------ |
| Scenario, users, and business problem | [../README.md](../README.md), [../docs/01-framing.md](../docs/01-framing.md) |
| Assumptions and constraints | [../README.md](../README.md) |
| Requirements, functional and non-functional | [../docs/](../docs/) |
| C4 diagrams, L1–L3 | [../diagrams/](../diagrams/) |
| Key decisions with trade-offs | [../adr/](../adr/) |
| Data flow and retrieval design | [../docs/](../docs/), [../diagrams/](../diagrams/) |
| Security, access control, threat model | [../docs/](../docs/) |
| Cost estimate and the assumptions behind it | [../docs/](../docs/) |
| Evaluation and monitoring plan | [../docs/](../docs/) |
| Risks, open questions, and what I would do next | here |

## Slides

- Keep the deck to the story, not the document: problem → architecture →
  the two or three decisions that mattered → risks.
- One diagram per slide, taken from `../diagrams/exports/` at full size.
- Commit the deck source alongside the exported PDF, same as diagrams.

## Status

| Artefact | State |
| -------- | ----- |
| `architecture.md` | Not started |
| `architecture.pdf` | Not started |
| Slides | Not started |
