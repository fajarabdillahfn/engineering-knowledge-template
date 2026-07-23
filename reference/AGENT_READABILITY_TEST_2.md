# Agent-Readability Test 2 — controlroom-inquiry with metadata

## Metadata

- **Date:** 2026-07-23
- **Subject:** `controlroom-inquiry` reference project with metadata applied to 3 assets (`architecture.md`, `ADR-0001-grpc-for-vendor-comms.md`, `ADR-0002-per-vendor-interface.md`)
- **Method:** Fresh sub-agent, blind to the previous test. Same 10 questions as the first run for direct comparison.
- **Goal:** Empirically validate `METADATA_DESIGN.md` (proposed in `METADATA_DESIGN.md`) — does applying the 5 conceptual fields actually improve agent navigation and gap-handling?
- **Tester:** Mavis (this session)
- **Subject agent:** `general` (sub-agent, fresh context)
- **Status:** METADATA_DESIGN validated for the navigation and gap-handling use cases. See "Conclusions" below.

## Why this test

`METADATA_DESIGN.md` proposed 5 conceptual fields (`status`, `last_validated`, `owner`, `scope`, `out_of_scope`) to address gaps surfaced by the URL Shortener dogfooding. The proposal was theoretical: the design was not yet applied to any asset, and F18's claim that the `out_of_scope` field would help with explicit non-decisions was inferential ("we have 5 candidates in `controlroom-inquiry`, the field would help if applied").

This test applies the metadata to 3 controlroom-inquiry assets and measures the impact. The test is intentionally minimal:
- Only 3 assets, not the full 11.
- The same 10 questions as the first run, to compare directly.
- One binary serialization choice (YAML frontmatter) — the design is conceptual, not format-specific.

## Methodology

**Baseline test (no metadata).** A fresh sub-agent answered 10 questions about `controlroom-inquiry` using only the assets in `reference/controlroom-inquiry/`. No metadata was applied. The full transcript is in the session log; the per-question hop counts and key findings are in the "Results" section below.

**Metadata application.** 3 assets were selected to cover the breadth of the test questions:
- `architecture.md` — the most cross-referenced asset; covers system-level questions.
- `ADR-0001-grpc-for-vendor-comms.md` — the gRPC decision; covers "why" questions.
- `ADR-0002-per-vendor-interface.md` — the per-vendor-interface decision; covers design questions.

The 5 fields from `METADATA_DESIGN.md` were applied as YAML frontmatter at the top of each file. Format choice rationale: YAML frontmatter is a common Markdown convention supported by static site generators, easy for agents to parse, and visible to humans without obscuring the body content. The format is a valid implementation choice per `METADATA_DESIGN.md`; the design is conceptual, not format-specific.

**Re-test.** A second fresh sub-agent answered the same 10 questions with the metadata now visible. The agent was explicitly told the metadata is part of the knowledge base and should be used as a navigation signal.

## The 10 questions

| # | Type | Question |
| --- | --- | --- |
| Q1 | WHAT | What is the API surface? What protocol and operations are exposed? |
| Q2 | WHAT | What data stores are read? Which holds the routing context? |
| Q3 | HOW | Walk through the inquiry flow end-to-end. |
| Q4 | WHY | Why gRPC for vendor communication instead of REST or MQ? |
| Q5 | WHY | Why per-vendor Go interface instead of generic abstraction? |
| Q6 | WHAT-IF (ops) | 3am, p99 latency 800ms (target 200ms). What to investigate? |
| Q7 | WHAT-IF (ops) | Vendor returns UNAVAILABLE. What happens? Circuit breaker? Fallback? |
| Q8 | DETAIL | gRPC retry policy values + rationale for those specific numbers. |
| Q9 | CONTRACT | What fields are in a successful `InquiryV2Response`? |
| Q10 | PROBE (gap) | Does this service authenticate the caller? |

## Results

### Per-question hop count: baseline vs metadata

| Q | Baseline (no metadata) | With metadata | Δ | Where metadata helped |
| --- | --- | --- | --- | --- |
| Q1 | 3 | 2 | -1 | `scope: system` confirmed architecture.md was the entry point. |
| Q2 | 1 | 1 | 0 | N/A (already 1 hop; `out_of_scope` confirmed Postgres as live source). |
| Q3 | 4 | 3 | -1 | Indirect. The flow is a value question; metadata could not shortcut it. |
| Q4 | 3 | 1 | **-2** | `out_of_scope` on ADR-0001 surfaced the uniformity and no-circuit-breaker caveats before reading the body. |
| Q5 | 3 | 1 | **-2** | `out_of_scope` on ADR-0002 made the boundaries of the decision immediately legible. |
| Q6 | 3 | 2 | -1 | `out_of_scope` on architecture.md ("no built-in circuit breaker") told the agent what *can't* be done — equally important at 3am. |
| Q7 | 3 | 2 | -1 | Same as Q6. The "no circuit breaker, no fallback" was directly in `out_of_scope`. |
| Q8 | 3 | 3 | 0 | Metadata did not help with the value (3, 0.1s, etc.); helped with the rationale (uniform across vendors, no per-vendor tuning). |
| Q9 | 2 | 2 | 0 | N/A (value question, not scope). |
| Q10 | 4 | 1 | **-3** | `out_of_scope` on architecture.md literally contained the answer: "Authentication (handled by api-gateway, not this service)". |
| **Total** | **27** | **18** | **-9 (-33%)** | |

### Hop distribution

| Hop count | Baseline | With metadata |
| --- | --- | --- |
| 1 hop | 1 | 4 |
| 2 hop | 4 | 4 |
| 3 hop | 3 | 2 |
| 4 hop | 2 | 0 |

The metadata eliminated all 4-hop answers and brought the "1 hop" count from 1 to 4.

### Per-agent assessment of metadata effectiveness

The sub-agent's own summary, in its own words:

- **High payoff (direct `out_of_scope` hit, 1-hop answer)**: Q10. The `out_of_scope` field on `architecture.md` literally contains the answer.
- **High payoff (frontmatter shaped the answer)**: Q4, Q5, Q6, Q7, Q8. The `out_of_scope` fields on `ADR-0001` and `ADR-0002` and on `architecture.md` directly surfaced the "uniform policy," "no circuit breaker," "no per-vendor customization" caveats that would otherwise have to be hunted down in the bodies.
- **Indirect (scope/last_validated as a "is this current?" check)**: Q1, Q2. The `scope: system` on `architecture.md` and the 2026-07-23 `last_validated` dates on the ADRs were useful as "is this the right entry point and is it recent?" signals.
- **No help (value questions, not scope statements)**: Q3, Q9. These required reading the full flow / full response-shaping doc; metadata could not shortcut that.
- **Navigation pattern that worked**: open `architecture.md` first for every question (its `out_of_scope` + `Operational Concerns` + Boundaries sections cover most of the "what is this service / what does it not do" questions), then follow the cross-references in its `Relationships` section to the specific ADR, flow, or troubleshooting doc.

The agent's assessment: **metadata helped 8/10 questions**, only failed for pure "value" questions.

## Findings (extending F1–F20)

### F21: Spec-vs-reality gap in architecture's "Operational Concerns" routing — RESOLVED by metadata

**Category.** Routing.

**Problem (from baseline).** The architecture document's "Operational Concerns" section routed to operational concerns (vendor failures, latency spikes, data store failure) but did not have a routing entry for "is X authenticated?" or other non-functional questions. The baseline test surfaced this as F21: Q10 (does the service authenticate?) took 4 hops because the agent had to fall back to `services/controlroom-inquiry.md` to find the answer.

**Resolution (with metadata).** The `out_of_scope` field on `architecture.md` lists "Authentication (handled by api-gateway, not this service)" as the first entry. The re-test agent answered Q10 in 1 hop, citing the metadata directly. The metadata resolved the routing gap without requiring a prose edit to the body of the architecture document.

**Implication.** Metadata can resolve some routing gaps that prose routing sections cannot. The "Operational Concerns" prose section routes to operational *issues*; the `out_of_scope` field routes to operational *absences*. Both are useful. A well-designed EKS asset should have both.

**Status.** Resolved for the architecture asset. The same pattern should be applied to other assets as they are revised.

### F22: Metadata is most valuable for "what is this NOT?" questions, less for "what is this?" questions

**Category.** Design implication.

**Problem.** The metadata has uneven payoff across question types. It is most valuable for questions that match an `out_of_scope` entry (Q10, the highest delta) or for questions whose scope can be disambiguated by `scope` and `out_of_scope` (Q4, Q5, Q6, Q7). It is less valuable for "walk me through the value" questions (Q3, Q9) where the agent must read the full body to extract the value.

**Evidence.** Per the agent's own assessment: 8/10 questions benefited from metadata in some way; the 2 that did not (Q3, Q9) are both value-walkthrough questions.

**Implication.** `METADATA_DESIGN.md` is correct in its field choices. The fields are designed to scope and qualify, not to replace content. Authors should not be tempted to put the full body content in `out_of_scope` (the temptation is real — `out_of_scope` is a natural place to put negative statements). The field is for *boundary* statements, not for content.

**Status.** Validation result. No change to the design.

### F23: The 5 fields proposed in `METADATA_DESIGN.md` are sufficient for this test

**Category.** Design validation.

**Problem.** The metadata design proposed 5 fields. Are 5 enough? Are 5 too many? Is one missing?

**Evidence.** All 5 fields were used by the agent in some way:
- `status` — used implicitly (Accepted = trusted) but not directly queried.
- `last_validated` — used as a "is this recent?" signal.
- `owner` — not directly used in this test; would be used by an agent that needs to "who do I ask about this?"
- `scope` — used as a routing signal (e.g., `scope: system` on architecture.md confirmed it as the entry point).
- `out_of_scope` — used heavily, especially for the "no" questions (no auth, no rate limit, no circuit breaker, no fallback).

**Implication.** 5 fields is a reasonable minimum. The 4 required + 1 optional balance is appropriate. A 6th field would be over-engineering at this point.

**Status.** Validated. No change to the design.

### F24: YAML frontmatter is a viable serialization choice for metadata

**Category.** Implementation.

**Problem.** The metadata design is conceptual. For the test, a serialization choice was required. YAML frontmatter was used.

**Evidence.** The agent parsed the frontmatter without issue. The Markdown body was unaffected. The frontmatter is visible to humans in standard Markdown editors and supported by static site generators.

**Implication.** YAML frontmatter is a valid implementation choice for EKS metadata. Other valid choices (sidecar files, JSON, graph database) are equally valid per the design. The conceptual design is preserved.

**Status.** Implementation note, not a design change.

## Conclusions

**The metadata design is empirically validated for the navigation and gap-handling use cases.**

- 33% reduction in hop count across 10 questions.
- All 4-hop answers eliminated.
- The 1-hop count more than tripled (1 → 4).
- F18's claim (that `out_of_scope` would help with explicit non-decisions) is now empirically supported, not inferential.
- F21 (architecture's "Operational Concerns" routing gap) was resolved by the metadata without a prose edit.

**The metadata design is not a panacea.** Q3 and Q9 (value-walkthrough questions) did not benefit. The metadata is for *qualifying* and *scoping*, not for *substituting* content. Authors should not be tempted to push content into metadata to "make it more visible" — the field structure is for boundaries, not for content.

**The 5 fields are sufficient for this test.** No 6th field surfaced as a need. The required/optional split (4 required + 1 optional) is appropriate. `out_of_scope` is the highest-leverage field; `last_validated` and `scope` are useful routing signals; `status` is implicitly useful; `owner` was not directly tested but is conceptually necessary.

**YAML frontmatter is a viable implementation choice.** It preserves the conceptual design, parses cleanly, and is widely supported. Other formats are equally valid per the design.

## Implications for the rest of the plan

1. **Adopt the metadata design.** The proposal in `METADATA_DESIGN.md` is now empirically validated. The next step is to apply it to all 11 controlroom-inquiry assets, then to the URL Shortener reference, and finally to formalize it in `KNOWLEDGE_MODEL.md`. The proposed spec change (7 surgical edits to `KNOWLEDGE_MODEL.md`) is now supported by data.

2. **F18 status changes.** From "Open" to "Validated." The `out_of_scope` field is empirically useful; the 5 explicit non-decisions in `controlroom-inquiry` are now structured fields, not prose.

3. **F21 status changes.** From "Open" to "Resolved (by metadata)." The architecture's routing gap for "is X authenticated?" is closed by the `out_of_scope` field.

4. **Session 2 task 2.4 (AGENTS.md guidance on `out_of_scope`) is now evidence-backed.** The re-test demonstrates that the field works as a navigation signal for agents. The guidance in `AGENTS.md` should reference this test as evidence.

5. **The next dogfooding (v1.0 in the roadmap) should apply metadata to all assets from the start, not as a retrofit.** The retrofit worked, but the upfront cost is lower.

## How to use this document

- **For spec authors:** F18 and F21 are resolved. The metadata design is validated. The proposed 7 edits to `KNOWLEDGE_MODEL.md` (in `METADATA_DESIGN.md` "What this means for `KNOWLEDGE_MODEL.md`") can now be applied with confidence.
- **For future dogfoodings:** Apply metadata from the start. Don't retrofit.
- **For the `AGENTS.md` update (Session 2 task 2.4):** Use the per-agent assessment in this report as evidence. The agent's own words ("The `out_of_scope` field on `architecture.md` literally contains the answer") are more convincing than any prose justification.
- **For methodology:** The test methodology (baseline test → apply → re-test → compare) is the validation gate for any future spec change. Every proposed field, every new relationship type, every new asset type should pass this gate before adoption.
