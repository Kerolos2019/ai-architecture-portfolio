# ADR-0002: Speech-to-text provider

```
Status:  Proposed — blocked on spike S1
Session: 1 — W1 mini project
Date:    2026-10-21
```

## Context

Voice is the primary interface for a large share of Qamhawy's users (G2). Many
farmers will send a voice note rather than type, and a transcription error at the
top of the pipeline poisons everything downstream: retrieval searches for the wrong
term, and the grounding check (P1) happily grounds the wrong question.

The hard part is not Arabic, it is **Egyptian dialect plus agricultural vocabulary**.
Crop varieties, pesticide brand names, and dosages are exactly the tokens a general
Arabic model has never seen, and exactly the tokens where an error does damage. A
provider with excellent headline Arabic WER can still be unusable here.

Voice notes are also the most sensitive data we handle — a recording of a farmer's
own voice, which PDPL treats more strictly than text (P3).

## Options

**A — Global STT API.** Broad language coverage, mature tooling, well-understood
pricing. Dialect and domain vocabulary are a known weak spot.

**B — Arabic-specialised API.** Likely the strongest on dialect. Smaller vendor,
which means more diligence on the DPA and on processing region, and more
concentration risk.

**C — Self-hosted open speech model.** Audio never leaves our tenancy, which
materially simplifies P3. We would need to fine-tune on agricultural vocabulary and
run the inference ourselves.

All three sit behind an `STTClient` adapter (P5), so this decision is reversible by
construction — that is the point of deciding the adapter before deciding the vendor.

## Trade-offs

| | Dialect accuracy | Agri vocabulary | Ops burden | Audio residency |
| --- | --- | --- | --- | --- |
| A | Moderate | Weak without adaptation | None | Cross-border |
| B | Likely strongest | Unproven on our terms | None | Cross-border, smaller vendor |
| C | Unknown before tuning | Tunable — the only option we control | High | Stays in tenancy |

None of these can be settled from vendor marketing. Published Arabic WER figures are
measured on broadcast speech, not on a farmer recording outdoors near machinery.

## Decision

**Deferred, pending spike S1.** We will not pick a provider on reputation.

**Spike S1.** Collect 100–200 consented voice notes from real users, covering the
main crop groups and a spread of governorates. Transcribe with all three options and
pick the lowest error against these gates:

| Gate | Threshold |
| --- | --- |
| Overall WER | ≤ 20% |
| WER on key agri terms (crops, pesticides, dosages) | ≤ 10% |
| p95 latency | ≤ 1.5 s |
| DPA signed and processing region confirmed | required — disqualifying if absent |

The key-term gate is deliberately stricter than the overall gate: a 20% error on
filler words costs nothing, a 10% error on a pesticide name costs a harvest.

## Consequences

- Voice remains behind a feature flag until S1 closes. The kill switch named in the
  ARB checklist exists partly for this.
- Consent must be collected before any voice note is stored or sent (P3), and S1
  itself cannot start until that consent flow ships.
- Whichever option wins, the `STTClient` adapter and its traces (P4) are already
  specified, so S1 measures the vendor and nothing else.
- If no option clears the key-term gate, the fallback is to ship voice input with an
  on-screen transcript the farmer confirms before the question is answered.

## Revisit when

S1 completes (this ADR then moves to Accepted with the chosen vendor), or volume
grows enough that self-hosting in option C becomes the cheaper path.

## Related

P2, P3, P4, P5 · [ADR-0001](0001-llm-hosting.md) established the adapter pattern.
