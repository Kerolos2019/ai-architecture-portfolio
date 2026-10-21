# Governance Pack — Qamhawy Agricultural AI Assistant

**Session 1 mini project · Kerolos · Draft v2 · 2026-10-21**

**System.** Qamhawy is the bilingual (Arabic/English) AI assistant inside the
**Mahaseel** farming platform. Egyptian farmers ask it agronomic questions by text,
voice, or photo; it answers from agronomist-approved content using Fusion RAG, with
Egyptian-dialect voice as the primary interface.

**Goals.** G1 grounded advice · G2 voice-first usability · G3 cost per answered
question ↓30% · G4 quality measurable per release

**Diagram.** [`01-governance-architecture.drawio`](../diagrams/src/01-governance-architecture.drawio)

---

## 1 · Principles

| # | Principle | Business goal | Forces in the design |
| --- | --- | --- | --- |
| **P1** | **Grounded or silent.** No agronomic advice without a retrieved source; otherwise say "not sure" | G1 · a wrong dose costs a harvest | Agent must call retrieval and a grounding check; relevance threshold; an explicit "not sure" path |
| **P2** | **Agronomists stay in the loop.** Only agronomist-approved content enters the knowledge base; low-confidence answers escalate to them | G1 · farmer trust; helpful rating ≥ 75% | Only the Ingestion Pipeline writes to MongoDB knowledge / Pinecone; review queue for flagged answers |
| **P3** | **Farmer data stays protected (PDPL).** Every flow to a vendor declares its PII: none, redacted, or sent under a DPA | Legal compliance; farmer trust | Redaction before the LLM and before tracing; DPA and confirmed region per vendor; audio retention policy |
| **P4** | **Every turn is measured:** traced, costed, evaluable | G3 cost ↓30%; G4 | Langfuse span per stage (tokens, TTS characters, STT minutes); DeepEval gate on the golden set |
| **P5** | **Every vendor sits behind an adapter.** No service calls a vendor directly; the adapter owns retries, redaction, and cost logging | G3 · G4 · keeps P3 and P4 enforceable in one place | `LLMClient`, `STTClient`, `TTSClient`, `VectorClient`, `TraceClient`; vendor credentials held nowhere else |

**P1 has a measurement trap.** Grounded-answer rate ≥ 90% is trivially achievable by
answering less — tighten the relevance threshold far enough and the bot is silent and
perfectly grounded, which fails G2 instead. So P1 is measured by **two** numbers
together: grounded-answer rate **≥ 90%** *and* "not sure" rate **≤ 25%**. Moving one
at the cost of the other is not an improvement.

**P5 was implicit before it was written down.** ADR-0001 and ADR-0002 both decided
"behind an adapter" independently; stating it as a principle is what makes it apply
to the *next* vendor someone adds, rather than being re-argued each time.

## 2 · ADRs

Full records in [`/adr`](../adr/).

| ADR | Options | Key trade-off | Decision · revisit when |
| --- | --- | --- | --- |
| [**0001** LLM hosting](../adr/0001-llm-hosting.md) · Accepted | A) Azure OpenAI direct · B) self-hosted open model · C) Azure OpenAI behind an `LLMClient` adapter with failover | A/C strong Arabic, no ops · B keeps data inside and flattens cost, but GPU ops and unproven dialect quality | **C** — retries, redaction (P3), and cost logging (P4) in one place. *Revisit:* a task's spend exceeds a GPU node **and** an open model is within 5 pts on our golden set |
| [**0002** STT provider](../adr/0002-stt-provider.md) · Proposed | A) global STT API · B) Arabic-specialised API · C) self-hosted open speech model — all behind an `STTClient` adapter | B likely strongest on dialect · C keeps audio inside but needs ops and fine-tuning · none can be settled from vendor claims | **Deferred to spike S1:** 100–200 consented voice notes; lowest WER wins (≤ 20% overall, ≤ 10% on key agri terms, p95 ≤ 1.5 s, DPA signed). *Revisit:* S1 closes, or volume makes C cheaper |
| [**0003** Knowledge sync](../adr/0003-knowledge-sync.md) · Accepted | A) nightly batch · B) real-time on publish · C) real-time + nightly reconciliation | A leaves corrections up to 24 h stale · B is fresh, but a lost event is *silent* drift | **C** — outbreak corrections live in minutes; a nightly id + hash diff repairs drift (P1). *Revisit:* reconciliation keeps finding drift |
| [**0004** Vector database](../adr/0004-vector-database.md) · Accepted | A) Pinecone · B) pgvector on Azure Postgres · C) Azure AI Search | B and C avoid a cross-border processor; C's native RRF duplicates the fusion we already wrote · A is ops-free but adds a PDPL flow | **A**, on two conditions: behind a `VectorClient` adapter (P5), and query vectors embedded from redacted text only and never persisted (P3). *Revisit:* index > ~5M vectors, or farmer-level personalisation vectors are proposed |

## 3 · ARB

**Who.** Architect (chair), the agronomy lead, the platform engineer, and the privacy
owner. **When.** Any PR that changes `architecture.yaml` or adds a vendor, plus a
standing fortnightly slot and a mandatory pass before each production release.

**What passing means.** Every check is Yes / No / N/A with an evidence link. A **No**
requires a written exception with a named owner and a date. Two open exceptions in
the same area block the release.

| Area | Checks |
| --- | --- |
| **Security** | ☐ Threat model covers prompt injection via farmer messages *and* via ingested articles/PDFs — P2's approved-content-only rule is the primary mitigation for the ingestion half ☐ Agent tools least-privilege (read knowledge; write only review flags) ☐ WebSocket auth and rate limits ☐ Secrets in a vault; outbound allow-list to listed vendors only |
| **Privacy (PDPL)** | ☐ Data classified: phone, location, voice notes, photos ☐ Consent captured before any voice note is stored ☐ PII declared on every vendor flow (P3) ☐ No PII in Langfuse traces or DeepEval datasets — weekly scan ☐ Audio retention enforced in Blob ☐ DPA and processing region confirmed for Azure, Pinecone, and the STT vendor |
| **Evals** | ☐ Golden set (≥ 200 Q&A, ≥ 50 safety-sensitive) passes faithfulness ≥ 90% before release ☐ Zero invented product names on the safety set ☐ Coverage: "not sure" rate ≤ 25% on the golden set ☐ STT dialect WER set (ADR-0002) ☐ Agronomist reviews a weekly sample |
| **Cost** | ☐ Cost per answered question (LLM + embeddings + TTS + STT + index) on a dashboard ☐ TTS cache hit rate tracked ☐ Budget alert per vendor |
| **Human oversight** | ☐ Low-confidence and safety-sensitive answers flagged to agronomists (P2) ☐ Named agronomist owner per crop group ☐ Kill switch for voice and photo diagnosis ☐ Audit log of content publishes and edits |

**PDPL specifics.** Egypt's Personal Data Protection Law (**Law No. 151 of 2020**)
requires authorisation for cross-border transfer of personal data — which covers
Azure OpenAI, Pinecone, and the STT vendor, i.e. most of what sits outside our trust
boundary. Its executive regulations have been slow to issue, so confirm the current
procedure with counsel rather than relying on any fixed process described here. Treat
voice notes and geotagged farm photos as the most sensitive categories we hold: both
identify an individual more directly than the text of a question does.

**Runtime triggers.** Every check above is a pre-release gate. These are the
production ones:

| Signal | Threshold | Action | Owner |
| --- | --- | --- | --- |
| Grounded-answer rate | < 90% over 24 h | Page the architect; releases frozen until resolved | Architect |
| "Not sure" rate | > 25% over 24 h | Retrieval tuning review — coverage is failing (P1) | Architect |
| Cost per answered question | over target 3 days running | Cost review; cache and model-tier audit | Platform engineer |
| Safety-sensitive answer disputed by a farmer | any | Agronomist review within 24 h; kill switch if it repeats | Agronomy lead |

