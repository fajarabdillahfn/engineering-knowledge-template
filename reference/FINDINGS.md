# Findings

Concrete candidates for improving EKS, based on the URL Shortener reference project. Each finding is backed by an example from `RETROSPECTIVE.md`.

Findings were collected during the dogfooding sprint. The high- and medium-priority findings have been addressed in a follow-up sprint (see Resolution fields below). The low-priority findings are deferred.

## Summary

| ID | Category | Title | Priority | Status |
| --- | --- | --- | --- | --- |
| F1 | Relationship types | Add `caused_by` | High | Addressed |
| F2 | Relationship types | Add `replaced_by` (inverse of `supersedes`) | High | Addressed |
| F3 | Asset boundaries | Clarify the difference between Troubleshooting and Playbook | High | Addressed |
| F4 | Asset boundaries | Clarify the boundary between Flow and Service | Medium | Addressed |
| F5 | Asset types | No asset type for non-architectural policies | Low | Deferred |
| F6 | Guidance | No guidance on intentional duplication between assets | Medium | Addressed |
| F7 | Guidance | No guidance on the relationship between Glossary and other assets | Low | Addressed |
| F8 | Terminology | No standard term for "the team's knowledge base as a whole" | Low | Deferred |
| F9 | Architecture | No guidance on cross-referencing playbooks and troubleshooting from architecture | Medium | Addressed |
| F10 | Relationship types | No relationship type for "this asset is operationally relevant to that asset" | High | Addressed |

---

## F1: Add a `caused_by` relationship

**Category.** Relationship types.

**Problem.** A Troubleshooting asset often describes an issue that exists because of a design choice. The current relationship vocabulary has no way to express this.

**Example.** `troubleshooting/elevated-redirect-latency.md` describes elevated latency that is `caused_by` a cache miss storm, which in turn exists because of the cache strategy in `decisions/ADR-0002-read-through-cache.md`. The closest existing type is `depends_on` (used to point to the playbook), but `depends_on` is about correctness, not causality. `relates_to` is generic. Neither expresses "this issue exists because of this design choice".

**Proposed addition.** Add `caused_by`: the source asset exists as a result of the target asset's design or behavior.

**Semantics.** If the target asset changes in a way that affects the cause, the source asset SHOULD be revalidated.

**Example usage.**
- `troubleshooting/elevated-redirect-latency.md` `caused_by` `decisions/ADR-0002-read-through-cache.md`.
- `troubleshooting/elevated-redirect-latency.md` `caused_by` `playbooks/cache-stampede-mitigation.md` (the failure mode).

**Priority.** High. The relationship type is missing, and the absence is felt in real examples.

**Resolution.** Added to KNOWLEDGE_MODEL §5.7. The new `caused_by` relationship enables expressing causality between an issue and the design choice that creates it.

---

## F2: Add a `replaced_by` relationship (inverse of `supersedes`)

**Category.** Relationship types.

**Problem.** `supersedes` lets the new asset link to the old one. But a reader of the old asset has no way to find the new one without already knowing about the new one.

**Example.** A future `ADR-0003` might supersede `ADR-0002` if the cache strategy changes. With `supersedes` only, `ADR-0002` does not point to `ADR-0003`. A reader of `ADR-0002` would not know it is no longer current unless they actively searched.

**Proposed addition.** Add `replaced_by`: the source asset has been replaced by the target asset.

**Semantics.** When a `supersedes` relationship is added from new to old, a corresponding `replaced_by` relationship SHOULD be added from old to new.

**Example usage.**
- `ADR-0002-read-through-cache.md` `replaced_by` `ADR-0003-new-cache-strategy.md` (when the latter is created).

**Priority.** High. Without it, deprecated assets are dead ends in the knowledge graph.

**Resolution.** Added to KNOWLEDGE_MODEL §5.8. The new `replaced_by` relationship is the inverse of `supersedes`, and the spec now requires that adding a `supersedes` relationship from a new asset to an old asset should be accompanied by a `replaced_by` from the old asset to the new.

---

## F3: Clarify the difference between Troubleshooting and Playbook

**Category.** Asset boundaries.

**Problem.** Both can describe a known operational situation. The spec says Troubleshooting is for "known issues" and Playbook is for "recurring operational scenarios", but the same situation can be both.

**Example.** The cache stampede scenario. As a Playbook, it is a recurring operational scenario with steps. As a Troubleshooting entry, it is a known issue with symptoms, diagnosis, and resolution. I ended up with both, but a reader could be confused about which to read first.

**Proposed clarification.** Define Troubleshooting as the *issue* (what is wrong) and Playbook as the *procedure* (what to do about it), making them complementary rather than overlapping. A Troubleshooting asset MAY reference a Playbook for the resolution.

**Alternative clarification.** Allow a Troubleshooting asset to embed a reference to a Playbook, making the pattern of "issue + procedure" explicit in the spec.

**Priority.** High. The current ambiguity led to an extra round of writing in this project.

**Resolution.** Addressed in KNOWLEDGE_MODEL §4.5 (Playbook) and §4.6 (Troubleshooting). Each type now has a "Relationship with [the other type]" subsection that clarifies the complementary roles: Troubleshooting describes the issue (what is wrong), Playbook describes the procedure (what to do about it). When both exist for the same situation, the Troubleshooting asset should reference the Playbook from its resolution section, and the Playbook may reference the Troubleshooting asset.

---

## F4: Clarify the boundary between Flow and Service

**Category.** Asset boundaries.

**Problem.** The spec says Flow is for cross-service processes. But a single-service process (like URL shortening in api-service) is still useful to document.

**Example.** `flows/url-shortening.md` is contained within `api-service`. It is not cross-service. But it is a useful artifact for new engineers and for code reviewers.

**Proposed clarification.** Either:
- Allow Flows to be contained within a single Service, with the cross-service case being a specialization.
- Or rename "Flow" to make the multi-service emphasis clearer (e.g., "Process" or "End-to-End Process").

**Priority.** Medium. The current ambiguity did not cause problems in this project, but it could in larger systems.

**Resolution.** Addressed in KNOWLEDGE_MODEL §4.3 (Flow). The purpose is now: "A Flow MAY involve one or more services. Cross-service processes are the most common case; a single-service Flow is permitted when the process is significant enough to warrant its own asset (for example, a request validation pipeline that is central to one service's contract)." The MUST NOT for Flow was also clarified: single-service implementation details belong in the Service asset, not the Flow.

---

## F5: No asset type for non-architectural policies

**Category.** Asset types.

**Problem.** The system has policies (e.g., "all services must use structured logging") that are not Architecture, not Service, not Flow, not Decision, not Playbook, not Troubleshooting, and not Glossary.

**Example.** A "logging policy" might be expressed as a Decision, but it is not really an architectural decision. It is an operational policy.

**Proposed addition.** Consider a new asset type for policies (e.g., `Policy`), or clarify that policies belong in the Conventions asset (which is mentioned in the canonical layout but is not one of the seven types).

**Priority.** Low. Policies can be expressed as Decision assets for now, even if it is a stretch.

**Resolution.** Deferred. A dedicated Policy asset type would require broader design work and was not justified by the dogfooding. The URL Shortener project had no policies that did not fit as Decisions. Revisit if a future project surfaces a clear need.

---

## F6: No guidance on intentional duplication between assets

**Category.** Guidance.

**Problem.** The spec says "one concern per asset" and "link, do not duplicate". But what about cases where the same concept legitimately needs to appear in multiple places (e.g., the cache strategy in architecture, ADR, and service)?

**Example.** The cache strategy appears in `architecture.md`, `decisions/ADR-0002-read-through-cache.md`, and `services/redirect-service.md`. Each mention is intentional, but the spec does not say so.

**Proposed addition.** Add guidance: when the same concept appears in multiple assets, the canonical (full) treatment is in the most specific asset (e.g., the ADR for a decision). Other assets mention it briefly and link to the canonical asset.

**Priority.** Medium. The current ambiguity led to uncertainty about how much to repeat.

**Resolution.** Added to KNOWLEDGE_MODEL §3.3 (Cross-references and Intentional Duplication). The new subsection states: when the same concept legitimately appears in multiple assets, the canonical (full) treatment lives in the most specific asset, and other assets mention it briefly and link to the canonical asset. Intentional duplication is not a violation of atomicity.

---

## F7: No guidance on the relationship between Glossary and other assets

**Category.** Guidance.

**Problem.** A Glossary term may also be the subject of a Decision or a Service. The spec does not say how to handle this overlap.

**Example.** "Read-Through Cache" is defined in `glossary.md` and is the subject of `ADR-0002-read-through-cache.md`. The glossary entry is short; the ADR is the full story.

**Proposed addition.** Add guidance: define the term in the Glossary for quick reference, and link to the full treatment in the corresponding asset. The Glossary is a quick lookup, not the source of truth.

**Priority.** Low. The pattern is intuitive, but a sentence in the spec would help.

**Resolution.** Addressed in KNOWLEDGE_MODEL §4.7 (Glossary). A new "Relationship with other assets" subsection clarifies: when a Glossary term is also the subject of another asset, the Glossary entry is a quick reference; the full treatment lives in the asset. The Glossary is a quick lookup, not the source of truth.

---

## F8: No standard term for "the team's knowledge base as a whole"

**Category.** Terminology.

**Problem.** I used "the system" and "the knowledge base" interchangeably. The Glossary defines specific terms but does not define what the collection itself is called.

**Example.** When describing the URL Shortener project, I said "the system" sometimes and "the knowledge base" other times. Both are correct, but the inconsistency is noticeable.

**Proposed addition.** A Glossary entry for "knowledge base" might help. The spec already uses the term informally; making it a defined term would tighten the language.

**Priority.** Low. Cosmetic.

**Resolution.** Deferred. Adding a "knowledge base" Glossary entry is a one-line change but was not justified by the dogfooding. The term is used informally in both the spec and the model without confusion.

---

## F9: No guidance on cross-referencing playbooks and troubleshooting from architecture

**Category.** Architecture.

**Problem.** The architecture document is the natural entry point for many questions, but it does not currently point to the playbooks and troubleshooting entries.

**Example.** The question "How do I mitigate a cache stampede?" is not directly answerable from `architecture.md`. A reader has to know to look in `playbooks/` or `troubleshooting/`.

**Proposed addition.** Architecture documents SHOULD include a section or appendix that cross-references the playbooks and troubleshooting entries. This is not a metadata requirement; it is a content guidance.

**Priority.** Medium. Improves discoverability without requiring new metadata.

**Resolution.** Addressed in KNOWLEDGE_MODEL §4.1 (Architecture). The Architecture asset's required characteristics now state: "SHOULD include a section or appendix that cross-references the playbooks and troubleshooting entries related to the system, so that operational questions can be answered from the architecture document." The MUST represent and Relationships sections were also updated to reflect this.

---

## F10: No relationship type for "this asset is operationally relevant to that asset"

**Category.** Relationship types.

**Problem.** During the dogfooding, I caught myself wanting to use a relationship that does not exist in the spec. I wanted to say "this playbook is operationally relevant to redirect-service" and "this troubleshooting entry is operationally relevant to redirect-service". The spec's vocabulary (`references`, `documents`, `depends_on`, `implements`, `relates_to`, `supersedes`) does not have a natural fit:

- `references` is "mentions for context" — too generic.
- `documents` is "source provides detailed docs of target" — wrong direction (a playbook does not document a service).
- `depends_on` is "correctness" — a playbook does not depend on a service in the correctness sense.
- `implements` is "realizes design" — wrong direction.
- `relates_to` is the catch-all — usable but loses semantic meaning.
- `supersedes` is "replaces" — wrong.

I first wrote `associated_with`, which is not in the spec. I then corrected it to `references` (the closest spec-defined type), but `references` does not capture the "operationally relevant to" semantics.

**Example.** In the URL Shortener project, the cache stampede playbook and the elevated latency troubleshooting entry are both "about" redirect-service, in the sense that they describe scenarios that manifest in that service. This is neither a dependency nor a documentation relationship.

**Proposed addition.** Add `applies_to` (or `operationally_relevant_to`): the source asset describes scenarios, procedures, or issues that manifest in the target asset. This is distinct from `references` (generic mention) and `relates_to` (catch-all).

**Why I caught this.** This finding is unusual in that I caught it by reviewing my own writing after the fact. The fact that I made this slip while trying to follow the spec is itself evidence that the vocabulary has a gap.

**Priority.** High. The gap is felt in real examples, and the slip shows it is not obvious how to express the relationship using only the spec's vocabulary.

**Resolution.** Added to KNOWLEDGE_MODEL §5.9. The new `applies_to` relationship expresses "this asset describes scenarios, procedures, or issues that manifest in the target asset" — distinct from `references` (generic mention) and `relates_to` (catch-all). The Playbook and Troubleshooting asset type definitions now use `applies_to` to express operational relevance.

---

## Out of scope for this sprint

The following were considered and explicitly NOT included in the findings:

- **Metadata schema.** Out of scope. Will be informed by findings.
- **Validation.** Out of scope. Will be informed by findings.
- **Templates.** Out of scope. The existing `_template.md` files in `knowledge/` are reference templates; new template work is future.
- **CLI.** Out of scope.
- **AI retrieval.** Out of scope.
- **Graph databases.** Out of scope.

These are explicitly the next sprints, not findings.

---

## How to use this document

Each finding has:
- A category (Relationship types, Asset boundaries, etc.).
- A problem statement.
- An example from the URL Shortener project.
- A proposed addition or clarification.
- A priority.

Findings are not commitments. The spec should not be modified without discussion. Use this document to drive that discussion.
