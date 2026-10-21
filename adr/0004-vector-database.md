# ADR-0004: Vector database

```
Status:  Accepted
Session: 1 — W1 mini project
Date:    2026-10-21
```

## Context

The Retrieval Service runs Fusion RAG: a keyword search over MongoDB and a dense
vector search, combined with Reciprocal Rank Fusion. The vector half needs an index.

This decision was treated as settled in the first draft of the architecture — Pinecone
simply appeared on the diagram — which is exactly the kind of choice that deserves a
record. It is a vendor outside the trust boundary, it is billable, and it is a
cross-border data flow under PDPL (P3). Three reasons to write it down.

One point governs the privacy analysis. Two different things cross this boundary:

1. **Content vectors**, embedded from agronomist-approved articles and Q&A. These
   contain no farmer data at all — they are, in effect, public agronomic material.
2. **Query vectors**, embedded from the farmer's question at search time. These *are*
   derived from farmer input and can encode PII.

The first is low-risk. The second is the one that constrains the options.

## Options

**A — Pinecone.** Managed, no index operations, mature filtering and namespaces.
Another processor outside the tenancy, another DPA, another region to account for.

**B — pgvector on Azure Database for PostgreSQL.** Vectors sit in the same Azure
region as the rest of the system, so there is no additional cross-border transfer and
no additional vendor. We operate the index — tuning, HNSW parameters, and scaling are
ours. Cheaper at our expected index size.

**C — Azure AI Search.** In-region, and ships hybrid search with RRF built in. Higher
cost floor, and the deepest lock-in of the three.

## Trade-offs

| | Ops burden | Cost at our scale | PDPL exposure | Fit with Fusion RAG |
| --- | --- | --- | --- | --- |
| A | None | Per-pod, higher floor | One more cross-border processor | Good — we do our own RRF |
| B | We tune and scale it | Lowest | None beyond existing Azure | Good — we do our own RRF |
| C | None | Highest floor | None beyond existing Azure | Native RRF we would not use |

C's headline feature is the one we need least: the Retrieval Service already
implements RRF across MongoDB and the vector index, so built-in hybrid search is not
a differentiator — we would be paying for a fusion step we have already written.

## Decision

**Option A, Pinecone**, bounded by two conditions:

1. **Behind a `VectorClient` adapter** (P5), so B remains a migration of one
   component rather than a rewrite of the Retrieval Service and the Ingestion
   Pipeline.
2. **Query vectors are embedded from redacted text only** (P3). The redaction that
   ADR-0001's `LLMClient` already performs before the LLM call applies equally
   before the query embedding. Query vectors are never persisted in the index.

With those conditions, what we store in Pinecone is approved agronomic content, and
what we send at search time carries no PII. That is the argument that makes a
cross-border vector index acceptable here — and it stops holding the moment someone
decides to store farmer conversations as vectors for personalisation.

## Consequences

- Pinecone joins the vendors needing a DPA and a confirmed processing region (P3),
  and the cross-border transfer basis under Law 151/2020 covers it alongside Azure
  OpenAI and the STT provider.
- A third-party outage degrades retrieval to keyword-only. The Retrieval Service must
  return MongoDB results with a lowered confidence rather than failing the turn — and
  a lowered confidence is what triggers P1's "not sure" path.
- Index cost belongs on the cost dashboard per answered question (P4), not as a flat
  monthly line nobody attributes.
- Any proposal to store farmer-derived vectors invalidates this ADR and must come
  back to the ARB.

## Revisit when

The index exceeds roughly 5M vectors, where pgvector becomes clearly cheaper; or PDPL
cross-border authorisation for this vendor proves burdensome; or we need farmer-level
personalisation vectors, which option A's privacy argument does not survive.

## Related

P1, P3, P4, P5 · [ADR-0001](0001-llm-hosting.md) for the adapter pattern and the
redaction this ADR depends on · [ADR-0003](0003-knowledge-sync.md) for what writes
into the index.
