# Diagrams

`diagrams/` — exported PNG/SVG plus the source files they were rendered from.

C4 diagrams for the architecture. Sources are committed alongside exports so
diagrams can be edited and diffed, not just viewed.

```
diagrams/
├── src/      # editable source: .mmd, .drawio, .py (diagrams-as-code)
└── exports/  # rendered .png / .svg, referenced from the docs
```

## Conventions

- **Filename**: `lN-subject.ext`, where `N` is the C4 level.
  - `l1-system-context.mmd` → `exports/l1-system-context.png`
  - `l2-containers.mmd`, `l3-retrieval.mmd`
- An export keeps the same basename as its source. Re-export whenever the source
  changes, in the same commit — a stale PNG is worse than no PNG.
- Prefer **SVG** for anything with small text; add PNG as well if the submission
  form needs a raster image.
- Docs link to the export and name the source beside it. Diagrams are never
  pasted inline into a Markdown file.

## The four C4 levels, and how far to go

| Level | Shows | Audience | Draw it? |
| ----- | ----- | -------- | -------- |
| L1 Context | The system as one box, plus users and external systems | Everyone, including non-technical | Always |
| L2 Container | Deployable/runnable units and the data stores between them | Engineers, reviewers | Always |
| L3 Component | Inside one container | Engineers building that container | Only where the internals are genuinely contested — typically retrieval or orchestration |
| L4 Code | Classes | — | Skip. It rots faster than it informs |

Supplementary diagrams worth having when they say something the C4 set cannot:
a **sequence diagram** for the main request path, a **data flow diagram**
annotated with trust boundaries for the threat model, and a **deployment
diagram** when data residency is a constraint.

## Rules that keep a diagram readable

- Every box has a name **and** a one-line responsibility. An unlabelled box is
  an unanswered question.
- Every arrow states what crosses it and in which direction — "query + user's
  group claims", not a bare line.
- Mark trust boundaries explicitly, and say where data comes to rest.
- One diagram answers one question. If it needs a paragraph of explanation to be
  understood, it is two diagrams.
- Include a legend when any shape, colour, or line style carries meaning.
- Check it in greyscale: colour alone must never be the only thing
  distinguishing two kinds of element.

## Tooling

Any of these is fine; the only requirement is that the source is committed.

- **Mermaid** (`.mmd`) — text, diffs well, renders natively on GitHub. Default
  choice for L1/L2.
- **draw.io / diagrams.net** (`.drawio`) — more control over layout; export
  PNG/SVG manually.
- **Structurizr DSL** — purpose-built for C4; one model generates all levels.
- **python `diagrams`** — code-generated, good when cloud provider icons help.

[src/l1-system-context.mmd](src/l1-system-context.mmd) is a starting skeleton.
