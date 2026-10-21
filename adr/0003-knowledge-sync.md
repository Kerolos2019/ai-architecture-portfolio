# ADR-0003: Knowledge base sync

```
Status:  Accepted
Session: 1 — W1 mini project
Date:    2026-10-21
```

## Context

Agronomists publish and correct content through the Content Admin tool. That content
has to reach two places before a farmer can be answered from it: MongoDB, which holds
the article text and Q&A pairs, and the Pinecone index, which holds their embeddings.
The Retrieval Service fuses results from both (Fusion RAG with RRF), so the two
stores being out of step is not a cosmetic problem — it produces answers grounded in
text that has since been corrected, which is precisely the P1 failure mode.

The deciding scenario is a **correction**, not a new article. An agronomist discovers
that a recommended dosage is wrong, or that a pest outbreak has changed the advice for
a region. The time between their edit and the farmer seeing the new advice is the
number that matters.

## Options

**A — Nightly batch.** One job re-chunks, re-embeds, and upserts everything changed
that day. Simple, cheap, easy to reason about. A correction can be up to 24 hours
stale, and during those hours the system confidently gives advice the agronomist has
already retracted.

**B — Real-time on publish.** The Content Admin tool emits an event; the Ingestion
Pipeline chunks, embeds, and upserts immediately. Corrections land in minutes. But a
lost or failed event is **silent** — nothing in the system notices that MongoDB and
Pinecone have diverged, and the divergence persists indefinitely.

**C — Real-time on publish, plus a nightly reconciliation pass.** B's latency with a
job that walks both stores by document id and content hash, repairing anything that
does not match.

## Trade-offs

| | Correction latency | Failure mode | Cost | Complexity |
| --- | --- | --- | --- | --- |
| A | Up to 24 h | Visible — the job fails loudly | Low, batched embeddings | Low |
| B | Minutes | Silent drift | Low | Moderate — event delivery |
| C | Minutes | Drift repaired within a day, and reported | Low + one nightly pass | Moderate |

## Decision

**Option C.** Real-time upsert on publish, with a nightly reconciliation that diffs
MongoDB and Pinecone by document id and content hash.

An outbreak correction reaching farmers in minutes is the entire benefit of B, and
the nightly pass removes B's one serious objection. The reconciliation also doubles
as a monitor: the count of repairs it makes is a health signal, not just a repair.

## Consequences

- The Ingestion Pipeline remains the only writer to either knowledge store (P2). The
  reconciliation job runs inside it, not as a separate service with its own
  credentials.
- Content hashes must be stored alongside each chunk for the diff to be cheap.
- A non-zero repair count on consecutive nights means event delivery is broken and
  should page someone, rather than being quietly fixed every night forever.
- Re-embedding on every publish costs tokens; the cost per publish is small but it
  belongs on the cost dashboard (P4) alongside answer costs.

## Revisit when

The nightly reconciliation keeps finding drift — at that point the event path is not
trustworthy and the choice is between fixing delivery and falling back to A.

## Related

P1, P2, P4 · [ADR-0004](0004-vector-database.md) determines what the sync writes into.
