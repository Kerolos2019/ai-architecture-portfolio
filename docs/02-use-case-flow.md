# Use-Case Flow — Qamhawy Incomplete-Order Nudge on WhatsApp

**Session 2 mini project · Kerolos · 2026-10-30**

**Use case.** A farmer adds an input (e.g. NPK fertilizer, often suggested in a Qamhawy
chat) to an order but doesn't finish it. After 2 hours, send one WhatsApp reminder with
2 suitable in-stock alternatives (matching their crop and governorate).

**Diagram.** [`02-order-nudge-flow.drawio`](../diagrams/src/02-order-nudge-flow.drawio)
([PNG](../diagrams/src/02-order-nudge-flow.drawio.png))

**Two paths.** Real-time (A): order event → CDC → Kafka → Flink timer → recommendation
engine → WhatsApp → farmer resumes the order. Background (B): events flow to the CDP;
Airflow builds profiles and features nightly, embeds the product catalog into the vector
DB, and retrains the ranker weekly.

## Options for choosing the alternatives (step 11)

| | A · Precomputed top-K in CDP / recommender DB | B · Real-time vector DB query (chosen) |
| --- | --- | --- |
| **Latency** | ≈ 5 ms read | ≈ 60 ms query, irrelevant on an async path with a 2 h timer |
| **Cost** | Nightly top-K for every farmer × product, mostly never used | Computed only for abandoned orders (a small share of farmers per day) |
| **Freshness** | Up to 24 h stale, so it may suggest out-of-stock fertilizer | Uses live stock from the feature store (fed by CDC → Flink) |
| **Operational DB load** | None | None: stock comes from the event stream, not DB queries |

**Decision: B.** The nudge path is asynchronous, so latency isn't the constraint.
Freshness and cost are, and B wins on both. An LLM writing each message was rejected:
WhatsApp needs pre-approved templates, and fixed templates keep advice grounded
(principle P1).

## Over-engineering check

**Needed from the high-level view:** channels + gateway (resume link), order service + DB
(source of truth), CDC → Kafka → Flink (detect inactivity without polling the DB),
Airflow + CDP (profiles, embeddings), vector DB (reuse Pinecone, add a product
namespace), recommendation engine, WhatsApp.

**Possibly over-engineered:** a full feature store (Feast) at today's volume. Start with
CDP profiles cached in Redis plus a stock stream, and adopt Feast when features are
shared by 2+ models.

**Not needed in this flow:** the Qamhawy LLM agent, TTS/STT, guardrail models (no free
text is generated).

**Missing from the high-level view:** a consent/opt-out service (PDPL; WhatsApp marketing
needs opt-in), a WhatsApp template registry, a frequency-cap store (1 nudge per order,
quiet hours 22:00–07:00), and attribution (`decision_id` → `order_completed` → CDP).

## Latency budget (P95)

| Path | Hops | Total | Target |
| --- | --- | --- | --- |
| Nudge (after the timer fires) | Kafka → engine 50 · features 10 · vector DB 60 · rules 20 · WhatsApp API 600 | ≈ 740 ms | ≤ 2 s |
| Farmer resumes order | gateway 60 · order read 40 · saved alternatives 10 · render 40 | ≈ 150 ms | ≤ 500 ms (course SLO) |

**Assumptions:** 2 h timer, latency figures and WhatsApp as the channel are illustrative.
Confirm them with Mahaseel's product team and in a load test.
