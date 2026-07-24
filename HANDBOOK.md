# Engineering Knowledge Specification — Handbook

## About this Document

This handbook is informative, not normative. It explains how engineering teams adopt and use the Engineering Knowledge Specification (EKS) in practice.

The normative rules are defined in [SPECIFICATION.md](SPECIFICATION.md). This document describes the workflow, philosophy, and recommendations that make EKS work well in real projects. Where the two documents overlap, the specification takes precedence.

Throughout this document, "knowledge asset" refers to a single document of one of the seven types defined in the specification. "Knowledge base" refers to the collection of all such assets in a repository.

---

## 1. Why EKS Exists

Most engineering knowledge is captured in long, monolithic documents that mix architecture, operations, and incident history. This works against the reader: humans lose the thread, and automated tools — including AI coding agents — cannot reliably retrieve the relevant subset.

EKS addresses this by:

- Defining a small set of knowledge asset types, each scoped to one concern.
- Specifying the relationships between them as a graph, not a hierarchy.
- Treating the knowledge base as machine-retrievable, not just human-readable.

The result is a body of knowledge that can be maintained by humans and consumed by humans and tools with equal reliability.

---

## 2. Adoption Strategy

The right adoption strategy depends on whether the project is greenfield or brownfield.

### Greenfield projects
For new projects, start with the seven knowledge asset types defined in the specification. Create an empty knowledge base with one starter document per type, then fill them in as decisions are made. The Architecture and Service assets usually come first; the rest follow as the system grows.

### Brownfield projects
For existing projects, do not attempt a full migration at once. Pick one knowledge asset — usually the Architecture asset or one Service asset — and write it. Migrate additional assets as the need arises.

### Common starting points
- A new system that has never been documented.
- A system undergoing a major refactor.
- A system where AI coding agents are producing work that contradicts team practice.

The signal that adoption is working: contributors reach for the knowledge base before making a change, and AI agents cite it correctly in their output.

---

## 3. Incremental Migration

A gradual approach works better than a rewrite. The recommended sequence:

1. Identify the highest-value knowledge asset — usually Architecture or the most-involved Service.
2. Write that asset using the corresponding starter document in the reference implementation.
3. Add Decision assets for choices that are binding but undocumented.
4. As flows emerge or incidents occur, add Flow, Playbook, and Troubleshooting assets.
5. Expand the Glossary as new terms appear.

The goal is to have a useful subset of the knowledge base, not a complete one. A knowledge base that is 60% complete and current is more valuable than one that is 100% complete and stale.

---

## 4. Recommended Workflow

The recommended workflow for maintaining an EKS knowledge base:

1. **Read first.** When working on a system change, read the relevant Service, Flow, and Decision assets before opening an editor.
2. **Update as part of the change.** When a change affects documented behavior, update the corresponding knowledge asset in the same change.
3. **Review for drift.** Periodically review each knowledge asset against the current system. Mark outdated assets as deprecated or update them.
4. **Deprecate, do not delete.** When a knowledge asset is no longer accurate, mark it as deprecated and link to its replacement. Historical context is valuable.

This workflow treats the knowledge base as part of the change, not as a separate artifact that drifts from the code.

---

## 5. Writing Philosophy

EKS knowledge assets should be:

- **Small.** A knowledge asset that grows past its concern is split. The specification's Design Principles provide the foundational rules.
- **Scoped.** Each asset represents one concept. Architectural, operational, and historical content are not mixed.
- **Linked.** Cross-references are explicit. The reader can navigate from any asset to the related ones.
- **Updated, not appended.** Corrections are made in place, not added at the bottom. Stale content is removed or marked.
- **Anchored in the system.** Knowledge assets describe the system, not the code. Code locations are referenced by path and identifier, not by copy.

This philosophy is a practical application of the Design Principles in the specification. When in doubt, return to the principles.

---

## 6. Repository Maintenance

A knowledge base is a living artifact. Recommended maintenance practices:

- **Assign owners.** Each knowledge asset has an owner (a team or individual) responsible for keeping it current.
- **Review on change.** Code changes that affect documented behavior update the relevant knowledge assets in the same change.
- **Periodic audits.** Quarterly or per-release, review each knowledge asset against the system. Mark outdated ones as deprecated.
- **Track deprecation.** Deprecated assets remain in the knowledge base with a clear marker and a link to the replacement.
- **Do not let the knowledge base grow stale.** Stale knowledge assets are worse than missing ones. If a knowledge asset cannot be maintained, remove it.

Maintenance is not optional. A knowledge base without a maintenance practice decays quickly.

---

## 7. Asset Lifecycle

Each knowledge asset moves through these states:

- **Draft.** Initial creation. Not yet reviewed.
- **Active.** Reviewed and current. Used as a source of truth.
- **Deprecated.** No longer current but retained for context. Linked to a replacement if one exists.
- **Archived.** No longer relevant to the system. Retained only for historical reference.

Move assets between states explicitly. Do not delete deprecated or archived assets without good reason — they often contain context that later decisions rely on.

---

## 8. When to Create New Knowledge

Create a new knowledge asset when:

- A new component is deployed or operated as a unit (Service).
- A new process crosses service boundaries (Flow).
- A binding choice is made that should be recorded (Decision).
- A new operational scenario is expected to recur (Playbook).
- A new issue has been observed, root-caused, and resolved (Troubleshooting).
- A new term appears that has a non-obvious or project-specific meaning (Glossary).

If none of these conditions hold, prefer a code comment, a pull request description, or a ticket. Not every piece of information belongs in the knowledge base.

---

## 9. When to Update Existing Knowledge

Update an existing knowledge asset when:

- The behavior it describes has changed.
- A term it defines has been added, removed, or redefined.
- A flow, service, or procedure it references has been modified.
- A decision it depends on has been superseded by a new Decision asset.

When updating, preserve the asset's structure. Add new information at the appropriate section, and revise outdated sections in place rather than appending corrections below them.

---

## 10. Best Practices

Recommended practices, in addition to the normative rules in the specification:

- **Lead with metadata.** The first lines of a knowledge asset should declare its identity, owner, and status. Do not bury this in prose.
- **Use the starter documents.** New assets should be created from the corresponding starter document in the reference implementation. Starter documents use the `_template.md` naming convention.
- **Link, do not duplicate.** When an asset references another, use a relative link rather than copying the content.
- **Match the system.** When knowledge assets and the system disagree, fix one to match the other and record the change.
- **Keep assets current.** Stale knowledge assets are worse than missing ones. Mark assets that are known to be outdated.
- **Cross-reference generously.** When an asset is relevant to another, link them. The graph of references is part of the value.

---

## 11. Common Mistakes

Patterns that reduce the value of an EKS knowledge base:

- **Starting with starter documents but never filling them in.** Starter documents are starting points, not deliverables. An empty starter document provides no value.
- **Writing only when there is an incident.** Knowledge assets written in response to incidents capture knowledge that is already stale. Document as decisions are made.
- **Copying code into knowledge assets.** Code embedded in knowledge assets drifts. Reference code by path and identifier.
- **Letting one knowledge asset grow without bound.** When an asset grows past its concern, split it. Do not append new sections to an asset that has lost its focus.
- **Skipping the review process.** Knowledge assets that are not reviewed become unreliable. Establish a review cadence.
- **Mixing knowledge layers.** Architecture, operations, and incident history are different layers. Keep them in different asset types.

---

## 12. Reference Implementation

The repository that contains this handbook is itself a reference implementation of EKS. It demonstrates one possible way to organize a knowledge base. The folders `knowledge/`, `prompts/`, `examples/`, and `.github/` are the reference implementation; the documents at the repository root (README, SPECIFICATION, HANDBOOK, AGENTS) are the specification and methodology that the reference implementation illustrates.

A repository adopting EKS is NOT required to use this layout. The specification defines concepts and requirements; the reference implementation shows one way to satisfy them. Other layouts that satisfy the Conformance requirements (§6 of the specification) are equally valid.

When in doubt about whether a layout choice is acceptable, check the specification's Conformance and Repository Structure sections rather than mirroring this repository.

---

## 13. Cross-Repository Navigation

Real systems are usually federations: a single service is one piece of a larger platform, with sibling services in other repositories, upstream gateways, and downstream vendors. EKS knowledge bases do not need to document the whole federation — each knowledge base is for one service. But the architecture document of each service SHOULD acknowledge its place in the larger system and point to the related knowledge bases.

When writing an architecture document for a service that has sibling or upstream services in other repositories:

- **Acknowledge the service's place in the larger system.** The first paragraph of the architecture document SHOULD mention the upstream caller (e.g., `api-gateway`), the sibling services (e.g., `controlroom-payment`), and the downstream dependencies (e.g., vendor gRPC services).
- **Reference other services by name and by repository.** The cross-reference is a placeholder; an EKS implementation MAY resolve it to navigate to the related knowledge base. The spec does not require a global index or registry.
- **Do not attempt to document the entire federation in your service's knowledge base.** The risk is duplication, drift, and the loss of a single source of truth. Each service is the source of truth for itself; cross-references point to the other sources of truth.
- **For services that are not yet documented in EKS,** the cross-reference is a TODO. Note it as such and move on. A placeholder is better than no mention.

The cost of not cross-referencing is that the architecture document becomes myopic — useful only to engineers working on the service in isolation, not to engineers trying to understand how the system fits together.

See FINDINGS F15 (in the controlroom-inquiry dogfooding) for the originating finding and the agent-readability implications.

---

## 14. ADR Confidence Levels

ADRs can be written at three different points in a project's life. The point at which an ADR is written affects the epistemic strength of its "rationale" sections, and the difference should be acknowledged in the ADR itself.

- **Recorded.** The ADR is written at or near the time the decision is made. The "Context" section reflects the actual considerations, the "Decision" section is the actual choice, and the "Consequences" section captures the trade-offs as understood at the time. **Confidence: high.**
- **Inferred.** The ADR is written after the fact, by reading existing code and reconstructing the reasoning. The "Context" and "Consequences" sections are the doc-writer's hypothesis about why the code is shaped this way, not a verbatim record of what was decided. **Confidence: medium.** The "Decision" section may be accurate (the code shows what was chosen) but the rationale is reconstructed.
- **Unverified.** The ADR is written for a hypothetical or aspirational decision — code that does not exist yet, or code that is about to change. **Confidence: low.** The ADR is a proposal, not a record.

An ADR SHOULD declare its confidence level explicitly. A simple frontmatter field (`confidence: recorded | inferred | unverified`) is sufficient. The reader can then weight the ADR's "rationale" sections accordingly.

When writing an inferred ADR, the doc-writer SHOULD:

- Note the inference in the body of the ADR (for example, an HTML comment: `<!-- This rationale is inferred from the code, not recorded from a prior decision. -->`).
- Cross-reference the source of the inference (which code files or commit messages were read).
- Mark the ADR for follow-up: if the original decision-maker is available, ask them to confirm or correct the rationale.

When reading an ADR, the reader SHOULD:

- Note the confidence level before weighing the rationale.
- Treat inferred and unverified ADRs as hypotheses, not records.
- Not let an inferred ADR be used to defend a design choice that the original author never actually defended.

The reverse-engineering methodology is sometimes the only available option for legacy systems, but it is not equivalent to recording. The distinction is the difference between a hypothesis to be tested and a fact to be trusted.

See FINDINGS F16 (in the controlroom-inquiry dogfooding) for the originating finding and the example of an inferred ADR (the per-vendor-interface decision).
