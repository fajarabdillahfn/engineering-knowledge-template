# Agent-Readability Test

## Metadata

- **Date:** 2026-07-22 (initial) / 2026-07-23 (re-test after F11 retrofit)
- **Subject:** URL Shortener reference project (`reference/`)
- **Method:** Fresh sub-agent with access to `reference/` only. No spec, no handbook, no AGENTS.md.
- **Goal:** Validate the central EKS claim — that AI agents can navigate a knowledge base by following asset relationships, rather than reading monolithic documents.
- **Tester:** Mavis (this session)
- **Subject agent:** `general` (sub-agent, fresh context)
- **Status:** Central claim validated after F11 retrofit. See "Re-test" section below.

## Why this test

Both external reviews (Claude, Qwen) accepted EKS's conceptual model but flagged the same gap: **"AI Retrievable" is asserted but not demonstrated.** The URL Shortener `RETROSPECTIVE.md` counts hops, but the author wrote the assets themselves, so the test is contaminated. This test uses a fresh agent that has never seen the system to get an honest signal.

## Methodology

The sub-agent received:

- Hard constraint: only the `reference/` directory is in scope. No spec, no knowledge model, no handbook, no other repo files.
- 10 questions spanning 4 categories (what / how / why / what-if) plus one probe for honest gap-reporting.
- For each question: state the answer, list files opened in order, call out caveats.

The sub-agent did not know this was a test of EKS — it was framed as a new engineer asking questions. This keeps the test realistic and the agent unbiased.

## The 10 questions

| # | Type | Question |
| --- | --- | --- |
| Q1 | WHAT (easy) | What are the three services, and which one handles creating short URLs? |
| Q2 | WHAT (easy) | What data store holds the source of truth? What is used for caching? |
| Q3 | HOW | Walk through the URL shortening flow. |
| Q4 | WHY | Why base62? What alternatives were rejected? |
| Q5 | WHY | Why read-through cache over other strategies? |
| Q6 | WHAT-IF (ops) | 3am. Redirect p99 is 800ms. What do I do? |
| Q7 | WHAT-IF (ops) | Redis was just flushed. What happens? How do we recover? |
| Q8 | DETAIL | What is the cache TTL, and what is the rationale for that specific duration? |
| Q9 | CONTRACT | What status code if a short code is not in cache or primary store? |
| Q10 | PROBE (gap) | Is there rate limiting on the write path? |

## Initial results (2026-07-22)

### Per-question hop count and correctness

| Q | Min hops to answer | Agent's hop count | Correctness | Notes |
| --- | --- | --- | --- | --- |
| Q1 | 1 (`architecture.md`) | 4 (over-searched) | ✓ Correct | Opened all 3 service files unnecessarily. |
| Q2 | 1 (`architecture.md`) | 2 | ✓ Correct | Cross-checked against ADR-0002. |
| Q3 | 1 (`flows/url-shortening.md`) | 4 (comprehensive) | ✓ Correct | Read flow + service + ADR + architecture. |
| Q4 | 1 (`ADR-0001`) | 3 | ✓ Correct | All 3 alternatives and rejection rationale captured. |
| Q5 | 1 (`ADR-0002`) | 2 | ✓ Correct | All 3 rejected alternatives captured. |
| Q6 | 2 (with F9 fix) / 3 (current) | **4** | ✓ Correct | Same navigation cost RETROSPECTIVE flagged as weakest. |
| Q7 | 1 (playbook) or 2 (via troubleshooting) | **3** | ✓ Correct | Correctly identified data is safe, only performance impacted. |
| Q8 | 1 (ADR-0002) | 3 | ✓ Correct TTL; ⚠ rationale gap | **Agent correctly noted the 1-hour figure is stated but the specific number is not justified.** |
| Q9 | 1 (`flows/url-redirection.md`) | 2 | ✓ Correct | Status code found in two places. |
| Q10 | 1 (any write-path doc, for absence) | 3 | ✓ Correctly reported absent | **Agent honestly said "not specified."** |

### Hop distribution (initial)

- **1 hop (theoretical minimum):** achievable for Q1, Q2, Q4, Q5, Q7, Q8, Q9, Q10 — i.e. **8/10 questions**.
- **2+ hops required:** Q3 (flow) and Q6 (operational). Q3 is fundamentally multi-hop (flow + service + ADR for full picture). Q6 was the open question.
- **3+ hops in practice:** Q6 and Q7, both operational. This matched the original `RETROSPECTIVE.md` finding.

### Initial scorecard

**What worked:**

- **Architecture.md is a strong entry point.** The agent opened it for almost every question, found the right sub-sections, and followed references.
- **ADRs are self-contained.** A single ADR-0001 or ADR-0002 read was enough to answer the corresponding "why" question completely.
- **The agent used cross-references for verification, not just navigation.** It re-opened ADRs from glossary entries to double-check facts. This is exactly the EKS use case.
- **The agent was honest about gaps.** Q8: "If a precise model was behind the number, it isn't in the reference docs." Q10: "rate limiting on the write path is not specified in the available documentation." This is the most important behavior for the EKS use case.
- **Operational scenarios are answered correctly, even when navigation is slow.** Q6 and Q7 produced detailed, correct answers despite the 3-4 hop cost.

**What didn't work (initially):**

- **Operational questions still take 3-4 hops.** This matched `RETROSPECTIVE.md` exactly. The F9 fix (cross-reference playbooks/troubleshooting from architecture) was added to `KNOWLEDGE_MODEL.md` §4.1, but **not retrofitted into this reference's `architecture.md`**. The spec and the reference implementation had diverged.
- **The directory did not distinguish system knowledge from process knowledge.** The sub-agent had to figure out that `FINDINGS.md` and `RETROSPECTIVE.md` are about the documentation process, not the URL shortener.
- **Content gaps surfaced that navigation cannot fix.** Q8's TTL rationale and Q10's rate-limiting silence are not EKS defects — they are content gaps in the URL shortener reference itself.

## Findings (extending F1–F10)

### F11: The reference implementation does not implement the F9 fix — RESOLVED

**Category:** Spec-vs-reality gap.

**Problem.** F9 said architecture documents SHOULD cross-reference playbooks and troubleshooting entries. The fix is in `KNOWLEDGE_MODEL.md` §4.1. But the URL Shortener's `reference/url-shortener/architecture.md` did not include this cross-reference. Q6 and Q7 cost 3-4 hops.

**Evidence (initial test).** Sub-agent's Q6 navigation: `troubleshooting/elevated-redirect-latency.md` → `playbooks/cache-stampede-mitigation.md` → `ADR-0002` → `architecture.md`. A direct pointer from `architecture.md` to the troubleshooting entry would have made this 2 hops.

**Resolution (2026-07-23).** Added a new "Operational Concerns" section to `reference/url-shortener/architecture.md` with three subsections (latency/error spikes, data store failure, why-the-system-is-shaped-this-way). Each entry points to the asset that has the full answer. Also updated the Relationships section to add `references` to the operational assets. Re-test confirmed the hop count reduction. See "Re-test results" below.

### F12: Directory structure conflates system knowledge with process knowledge

**Category.** Layout.

**Problem.** `reference/` contains 12 system assets and 2 process assets (`FINDINGS.md`, `RETROSPECTIVE.md`). The two kinds are not visually or structurally distinguished. An agent (or a new engineer) cannot tell from the directory that one set describes the URL shortener and the other describes the documentation effort.

**Evidence.** Sub-agent noted the discrepancy itself and had to skim the process files to confirm they were not system knowledge. This worked, but only because the agent was careful.

**Priority.** Low. Cosmetic in the reference implementation, but a real consideration for the canonical layout in `knowledge/`.

**Resolution.** Defer to second dogfooding. If `controlroom-inquiry` surfaces a similar ambiguity, address in the canonical layout.

### F13: EKS has no construct for "explicit non-decisions"

**Category.** Asset type / content guidance.

**Problem.** Q10 revealed that rate limiting is not implemented *and* not discussed. EKS has Decision assets for choices that were made. It has no way to express "we considered X and explicitly chose not to do it" (anti-decision) or "X is intentionally not in scope" (non-decision). This means the absence of a feature looks identical to a documentation gap, even when it's a deliberate choice.

**Evidence.** Sub-agent's Q10 answer: "If a rate limiter exists as an undocumented cross-cutting concern (e.g., at the load balancer or API gateway), it is not represented in the knowledge base." The agent had to acknowledge it could not distinguish three states: (1) not implemented and not discussed, (2) not implemented and deliberately out of scope, (3) implemented but not documented.

**Priority.** Low for fictional reference; potentially high for real systems where "we chose not to do X" is a real and recurring need.

**Resolution.** Defer. If `controlroom-inquiry` dogfooding surfaces more cases of "explicitly not done," revisit. Could be expressed as a Decision with a "Rejected" status, or as a new "Non-Decision" asset type. Not enough evidence yet.

### F14: Justification depth varies across ADRs (content quality, not EKS issue)

**Category.** Reference content.

**Problem.** ADR-0001 (base62) and ADR-0002 (read-through cache) both have a "Context / Decision / Consequences / Alternatives" structure. But the *depth* of justification varies. ADR-0001 doesn't quantify why 7 characters specifically (vs. 6 or 8). ADR-0002 doesn't quantify why 1 hour specifically (vs. 5 minutes or 24 hours). These are decisions that probably had numbers behind them, but the ADRs present only qualitative tradeoffs.

**Evidence.** Sub-agent's Q8 caveats: "The docs don't quantify *why* 1 hour specifically ... If a precise model was behind the number, it isn't in the reference docs."

**Priority.** Low. This is a writing-quality issue, not a structural one. The ADR template is fine.

**Resolution.** Out of scope for the spec. Reference authors should be reminded to include the quantitative basis when one exists. Could be added to `HANDBOOK.md` as a writing tip.

## Re-test results (2026-07-23)

After applying the F11 fix (added "Operational Concerns" section to `reference/url-shortener/architecture.md`), all 10 questions were re-run with a fresh sub-agent.

### Q6 hop count: 4 → 2

**Before:** Agent navigated `troubleshooting/elevated-redirect-latency.md` → `playbooks/cache-stampede-mitigation.md` → `ADR-0002` → `architecture.md`. 4 files, entry point not obvious.

**After:** Agent navigated `architecture.md` (Operational Concerns) → `troubleshooting/elevated-redirect-latency.md`. 2 files. Agent's own comment: "The 'start here' pointer for operational questions is in `architecture.md` itself ... the effective hop count is 1-2 from there."

### Q7 hop count: 3 → 2

**Before:** Agent navigated `playbooks/cache-stampede-mitigation.md` → `architecture.md` → `troubleshooting/elevated-redirect-latency.md`. 3 files.

**After:** Agent navigated `architecture.md` (Operational Concerns, "Redis unavailable" entry) → `playbooks/cache-stampede-mitigation.md`. 2 files. Agent's own comment: "the architecture file frames Redis unavailability as 'functionally a cache stampede' and points straight at the playbook — that's the key 1-hop shortcut."

### Regression check (Q1–Q5, Q8–Q10)

All other questions answered correctly with the same or fewer files than before. No regressions.

- Q1: 1 file (down from 4 — agent no longer over-opens service files)
- Q2: 1 file (same)
- Q3: 4 files (same — comprehensive answer)
- Q4: 1 file (same)
- Q5: 1 file (same)
- Q8: 1 file for the value, with cross-references (same)
- Q9: 1 file (same)
- Q10: 3 files checked, "not specified" answer (same honest answer)

### F11 status: Resolved

The retrofit produced the expected hop reduction. The "Operational Concerns" section works as a routing layer for operational questions. The spec and reference implementation are now in sync on this point.

## Conclusion: what this means for EKS

**The central claim is validated for fictional references.**

After the F11 retrofit:

- For "what" and "why" questions, the asset graph delivers on the 2-hop promise. ADRs are well-anchored, `architecture.md` is a good entry point, and cross-references enable verification.
- For "what-if" operational questions, navigation now costs 1-2 hops. `architecture.md`'s "Operational Concerns" section explicitly routes latency-spike questions to the troubleshooting entry, and Redis-flush questions to the playbook. **Q6 dropped from 4 hops to 2. Q7 dropped from 3 hops to 2.**
- The agent behaved well: it was honest about gaps, it cross-verified, it used the cross-references for context. EKS's asset boundaries made the agent's job easier, not harder.

**Net assessment:** EKS's structure is sound for a fictional reference. The "AI Retrievable" claim now has empirical support from this test, with the caveat that the structural gap (F11) had to be closed first. Real-system validation (the second dogfooding on `controlroom-inquiry`) is the harder and more important test. Fictional references can demonstrate the structure; only a real system can demonstrate the discipline.

## Implications for the rest of the plan

1. **F11 retrofit is now done.** The fix to `reference/url-shortener/architecture.md` was small and effective. Re-test confirmed hop count dropped from 4 to 2 for Q6 and from 3 to 2 for Q7.

2. **F12 and F13 are low-priority but real.** The directory layout issue and the "explicit non-decision" gap should be noted for the second dogfooding. If `controlroom-inquiry` surfaces more cases of "we explicitly do not do X" or "where is the system-knowledge/process-knowledge boundary," they should be addressed then.

3. **F14 belongs in `HANDBOOK.md`, not the spec.** Writing-quality issues don't justify spec changes. The spec should stay clean.

4. **The metadata schema work (Task 2 in the plan) is now better scoped.** The test surfaced that the agent needed to know: which assets are still current, which are "we explicitly don't do," and which are "out of scope." These map to metadata fields: `status`, `scope`, `out_of_scope`. The schema can be designed with these in mind.

5. **The second dogfooding on `controlroom-inquiry` is even more justified.** Real systems have more operational mess, more "explicit non-decisions" (compliance, security, vendor constraints), and more cross-cutting concerns. A clean fictional reference validates the structure; a real system validates the *content discipline*.

6. **"AI Retrieval" is now a validation gate, not a milestone.** With this test as evidence, the spec can claim empirical support for the structure. Every future change to `KNOWLEDGE_MODEL.md` should ideally be re-tested — not because the test is expensive, but because the cost of structural regression is hidden until an agent misbehaves.

## How to use this document

- **For spec authors:** F11 is resolved. F12, F13, F14 remain open at low priority.
- **For the next dogfooding:** F13 (explicit non-decisions) is the pre-seeded question for the `controlroom-inquiry` test. F12 (directory layout) should be tested by re-running this procedure against the new project's directory.
- **For the metadata schema (Task 2):** use the gap-revealing questions (Q8, Q10) as test cases — does the proposed metadata help the agent answer "is this still true?" and "is this in scope?" more reliably?
- **For future re-tests:** keep this report's question set and methodology as a template. Re-running the same 10 questions against a modified knowledge base gives a comparable hop-count signal.
