# ADR-0001: LLM hosting

```
Status:  Accepted
Session: 1 — W1 mini project
Date:    2026-10-21
```

## Context

Qamhawy answers Egyptian farmers in Arabic and English. The LLM is the largest
single line in cost per answered question (G3) and the component most likely to be
swapped within a year, so *where it runs* and *how it is reached* are both decisions.

Two constraints frame the choice:

- **Arabic quality is not negotiable.** The farmer writes or speaks Egyptian dialect;
  the answer must come back in clear Arabic. A model that handles Modern Standard
  Arabic but stumbles on dialect input fails G1 at the first turn.
- **Farmer messages carry PII** — phone numbers, village names, farm size, crop
  history. Any hosted model is a cross-border flow under PDPL (P3).

## Options

**A — Azure OpenAI, called directly from each service.** Strong Arabic, no GPU
operations, pay per token. Every service that needs the model holds its own client,
its own retry logic, and its own redaction code.

**B — Self-hosted open-weight model on our own GPUs.** Farmer data never leaves our
tenancy. Cost is flat rather than per-token, which eventually beats per-token pricing
at volume. We take on GPU capacity planning, model updates, and an unknown: no
open model has published dialect-Arabic agronomic benchmarks we can trust.

**C — Azure OpenAI behind an `LLMClient` adapter, with a secondary deployment for
failover.** Same model as A, but every call goes through one component that owns
retries, redaction, cost logging, and the choice of deployment.

## Trade-offs

| | Arabic / dialect | Ops burden | Cost shape | Data residency |
| --- | --- | --- | --- | --- |
| A | Strong | None | Per token, scales with use | Cross-border, DPA needed |
| B | Unknown, needs fine-tuning | High — GPU ops, upgrades | Flat, high floor | Stays in tenancy |
| C | Strong | Low | Per token, measurable per stage | Cross-border, DPA needed, redaction centralised |

A and C buy the same model quality. The difference is that A scatters the three
things governance depends on — redaction (P3), cost attribution (P4), and the
ability to switch vendor — across every service that calls the model.

## Decision

**Option C.** Azure OpenAI reached only through an `LLMClient` adapter, with a
secondary deployment configured for failover.

The adapter is what makes the rest of the governance pack enforceable: redaction
happens in one place instead of six, every call emits a costed trace, and swapping
to option B later becomes one component's problem rather than a rewrite. This is the
first instance of what P5 generalises to all vendors.

## Consequences

- One component owns vendor credentials; no service holds an API key directly.
- Cost per stage is attributable, which is what G3's 30% target is measured against.
- A redaction bug is a single-component bug, with a single-component fix.
- The adapter is on the hot path for every answer, so its own latency budget matters;
  it must add no more than a few milliseconds over a direct call.
- We remain a cross-border processor and must keep the DPA and region current (P3).

## Revisit when

A single task's monthly spend exceeds the cost of a dedicated GPU node **and** an
open-weight model lands within 5 points of Azure OpenAI on our golden set.

## Related

P1, P3, P4, P5 · [ADR-0002](0002-stt-provider.md) applies the same adapter pattern to
speech · [ADR-0004](0004-vector-database.md) applies it to the vector index.
