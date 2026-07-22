# Engineering Knowledge Model (EKM)

## Document Metadata

| Field | Value |
| --- | --- |
| Title | Engineering Knowledge Model |
| Specification Version | v0.1.0 Draft |
| Status | Draft |
| Last Updated | 2026-07-22 |
| Compatibility | EKS v0.1 |
| Intended Audience | Authors of EKS implementations, tooling developers, specification reviewers |
| Companion Document | SPECIFICATION.md |

---

## 1. Introduction

### 1.1 Purpose
The Engineering Knowledge Model (EKM) defines the conceptual model underlying the Engineering Knowledge Specification (EKS). It specifies the kinds of knowledge assets an EKS implementation represents, the relationships between them, the lifecycle they pass through, and the concepts of ownership and traceability.

EKM is implementation-independent. It does not prescribe a file format, a storage technology, or a particular set of tools. SPECIFICATION.md describes the rules an EKS-compatible repository must follow; this document describes the conceptual model that those rules implement.

### 1.2 Intended Audience
EKM is written for authors of EKS implementations — file-based repositories, databases, graph stores, APIs — for tooling developers who consume or produce EKS knowledge bases, and for specification reviewers evaluating the conceptual model.

### 1.3 Non-Goals
EKM does not define:
- A specific storage format (Markdown, YAML, JSON, databases, or any other representation).
- A specific metadata schema.
- A specific validation tool or workflow automation.
- Project-specific content.
- The rules an EKS-compatible repository must follow (see SPECIFICATION.md).

### 1.4 Relationship to SPECIFICATION
SPECIFICATION.md is normative for the rules an EKS-compatible repository must follow. EKM is normative for the conceptual model that those rules describe. A repository that does not satisfy EKM is not EKS-compliant, regardless of how it organizes its files.

---

## 2. Normative Language
This document follows the normative language defined in SPECIFICATION §2 (Normative Language), including the keywords MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY.

---

## 3. Knowledge Asset
A **Knowledge Asset** is a single, atomic unit of engineering knowledge. Each asset has a type, a lifecycle state, an owner, and a set of relationships to other assets.

A Knowledge Asset is a logical concept. Markdown files are one possible storage format; databases, APIs, graph stores, and other representations are equally valid. An EKS implementation MAY choose any representation that preserves the properties defined in this document.

### 3.1 Properties
A Knowledge Asset has the following conceptual properties:

- **Type.** One of the seven asset types defined in §4.
- **Lifecycle State.** One of the states defined in §6.
- **Owner.** A team or individual responsible for the asset. See §7.
- **Relationships.** A set of directed connections to other assets. See §5.
- **Body.** The substantive content of the asset, in a representation chosen by the implementation.

### 3.2 Atomicity
A Knowledge Asset MUST represent one concept. An asset that grows past its concept SHOULD be split into multiple assets. The corresponding design principle is defined in SPECIFICATION §3.4 (Single Responsibility).

### 3.3 Cross-references and Intentional Duplication
A Knowledge Asset SHOULD reference other assets where it borrows, depends on, or contextualizes their content.

When the same concept legitimately appears in multiple assets (for example, a design choice mentioned in the Architecture asset, the Decision asset, and a Service asset), the canonical (full) treatment lives in the most specific asset. Other assets mention it briefly and link to the canonical asset. This is intentional duplication, not a violation of atomicity.

An implementation SHOULD support cross-references in a way that allows the relationships to be discovered without requiring human readers to follow the links manually.

---

## 4. Asset Types
EKM defines seven asset types. Each type has a defined purpose, a set of responsibilities, a set of required characteristics, and a normative set of content requirements. The types are orthogonal: a given piece of engineering knowledge SHOULD be expressible as exactly one type.

### 4.1 Architecture
**Purpose.** Describes the system as a whole: its components, boundaries, and how data moves between them.

**Responsibilities.** Capturing the highest-level description of the system; serving as the entry point for understanding the system; representing the system to readers who need a high-level overview; providing operational entry points to the playbooks and troubleshooting entries related to the system.

**Required characteristics.** A single asset per system. The root of the knowledge graph. SHOULD include a section or appendix that cross-references the playbooks and troubleshooting entries related to the system, so that operational questions can be answered from the architecture document.

**MUST represent.** The system's components, boundaries, and primary data flows. Operational entry points (cross-references to playbooks and troubleshooting entries).

**MUST NOT represent.** Implementation details, runbook content, troubleshooting details, or incident history.

**Relationships.** Referenced by Service, Flow, Playbook, and Troubleshooting assets. MAY reference Decision, Playbook, and Troubleshooting assets to provide operational entry points.

### 4.2 Service
**Purpose.** Describes a single deployable or independently operable unit.

**Responsibilities.** Capturing the purpose, dependencies, and operation of a single unit; serving as the authoritative reference for that unit.

**Required characteristics.** Associated with one or more owners. Identified by a stable, descriptive name. Scoped to a single unit.

**MUST represent.** What the service does, what it depends on, and how it is operated.

**MUST NOT represent.** Business rules that belong in the Architecture asset, or operational steps that belong in a Playbook asset.

**Relationships.** A child of the Architecture asset. Referenced by Flow, Playbook, and Troubleshooting assets. MAY reference Decision assets.

### 4.3 Flow
**Purpose.** Describes a coherent end-to-end process. A Flow MAY involve one or more services. Cross-service processes are the most common case; a single-service Flow is permitted when the process is significant enough to warrant its own asset (for example, a request validation pipeline that is central to one service's contract).

**Responsibilities.** Capturing the trigger, steps, outcome, and failure handling of a process.

**Required characteristics.** Identifies the services involved. Specifies the trigger and the outcome.

**MUST represent.** The trigger, the steps, the outcome, and the failure handling. The services involved.

**MUST NOT represent.** The implementation of any single service in isolation. Single-service implementation details belong in the Service asset.

**Relationships.** A child of the Architecture asset. References one or more Service assets. MAY be referenced by Playbook and Troubleshooting assets. MAY reference Decision assets.

### 4.4 Decision
**Purpose.** Records a binding architectural choice, the context in which it was made, and its consequences.

**Responsibilities.** Capturing the rationale for binding choices; preserving institutional knowledge of why the system is shaped as it is.

**Required characteristics.** Stable, sequential identifier. Context, decision, and consequences recorded together.

**MUST represent.** The context, the decision, and the consequences.

**MUST NOT represent.** Transient or trivial choices, or choices that are not yet binding.

**Relationships.** Orthogonal to the asset hierarchy. MAY be referenced by any other asset.

### 4.5 Playbook
**Purpose.** Describes how to handle a recurring operational scenario.

**Responsibilities.** Capturing the preconditions, steps, and rollback for a recurring scenario; serving as a reference for operators.

**Required characteristics.** Identifies the service or services involved. Specifies preconditions and rollback.

**MUST represent.** The preconditions, the ordered steps, and the rollback procedure. The service or services involved.

**MUST NOT represent.** One-off operations that are not expected to recur.

**Relationship with Troubleshooting.** A Playbook describes the *procedure* (what to do); a Troubleshooting asset describes the *issue* (what is wrong). The two are complementary: when both exist for the same situation, the Troubleshooting asset SHOULD reference the Playbook from its resolution section, and the Playbook MAY reference the Troubleshooting asset to indicate which known issue it addresses.

**Relationships.** `applies_to` one or more Service assets. References one or more Flow assets. MAY reference Decision assets. MAY reference Troubleshooting assets when the scenario corresponds to a known issue.

### 4.6 Troubleshooting
**Purpose.** Describes a known issue, its symptoms, root cause, diagnosis, resolution, and prevention.

**Responsibilities.** Capturing the resolution of observed issues; serving as a reference for future occurrences of similar issues.

**Required characteristics.** Associated with the affected service or services. Identifies the issue, symptoms, root cause, and resolution.

**MUST represent.** The issue, the observable symptoms, the root cause, and the resolution.

**MUST NOT represent.** Hypothetical or unresolved issues.

**Relationship with Playbook.** A Troubleshooting asset describes the *issue* (what is wrong); a Playbook describes the *procedure* (what to do about it). The two are complementary: when both exist for the same situation, the Troubleshooting asset SHOULD reference the Playbook from its resolution section, and the Playbook MAY reference the Troubleshooting asset to indicate which known issue it addresses.

**Relationships.** `applies_to` one or more Service assets. References one or more Playbook assets. MAY reference Decision assets. MAY `caused_by` one or more Decision or Architecture assets when the issue exists because of a documented design choice.

### 4.7 Glossary
**Purpose.** Defines terms, acronyms, and domain vocabulary used across other EKS assets.

**Responsibilities.** Capturing non-obvious or project-specific vocabulary; serving as a reference for readers and tools.

**Required characteristics.** Defines each term with sufficient context for a new team member to understand it.

**MUST represent.** Terms that have a non-obvious or project-specific meaning.

**MUST NOT represent.** Terms with standard, widely-known meanings.

**Relationship with other assets.** A Glossary term MAY also be the subject of another asset (for example, "Read-Through Cache" might be defined in the Glossary and be the subject of a Decision asset about the cache strategy). In that case, the Glossary entry is a quick reference; the full treatment lives in the asset. The Glossary is a quick lookup, not the source of truth.

**Relationships.** Orthogonal to the asset hierarchy. Referenced by any asset that uses specialized vocabulary.

---

## 5. Relationships
A **Relationship** is a directed connection from one Knowledge Asset (the source) to another (the target). Relationships enable navigation, impact analysis, and the representation of the knowledge graph.

EKM defines the following relationship types. Implementations MAY introduce additional types, provided the semantics of the defined types are preserved.

### 5.1 references
The source asset mentions the target asset for context.

**Semantics.** A generic connection used when no more specific relationship applies.

**Example.** A Flow asset references the Service assets that participate in it.

### 5.2 documents
The source asset provides detailed documentation of a concept introduced by the target asset.

**Semantics.** The target asset introduces a concept at a high level; the source asset describes it in detail.

**Example.** A Playbook asset documents a procedure that a Flow asset references.

### 5.3 depends_on
The source asset's correctness or validity depends on the target asset.

**Semantics.** If the target asset changes, the source asset SHOULD be revalidated.

**Example.** A Service asset depends on a Decision asset about the message broker it uses.

### 5.4 implements
The source asset realizes the design specified by the target asset.

**Semantics.** The target asset describes a contract or design; the source asset fulfills it.

**Example.** A Service asset implements an API contract defined in a Decision asset.

### 5.5 relates_to
A general, undefined relationship.

**Semantics.** A catch-all used when no other type applies. SHOULD be used sparingly; prefer a more specific type when one is appropriate.

### 5.6 supersedes
The source asset replaces the target asset.

**Semantics.** The target asset is no longer current. The source asset SHOULD be cross-referenced from the target.

**Example.** A Decision asset v2 supersedes a Decision asset v1.

### 5.7 caused_by
The source asset exists as a result of the target asset's design or behavior.

**Semantics.** The target asset is the cause of the source asset's existence. If the target asset changes in a way that affects the cause, the source asset SHOULD be revalidated.

**Example.** A Troubleshooting asset is `caused_by` a Decision asset whose design creates the failure mode the troubleshooting entry addresses. A Troubleshooting asset is `caused_by` an Architecture asset when the system's high-level design is the root cause.

### 5.8 replaced_by
The source asset has been replaced by the target asset. This is the inverse of `supersedes` (§5.6).

**Semantics.** When a `supersedes` relationship is added from a new asset to an old asset, a corresponding `replaced_by` relationship SHOULD be added from the old asset to the new asset. This enables backward navigation from deprecated assets to their replacements.

**Example.** An old Decision asset v1 is `replaced_by` a new Decision asset v2.

### 5.9 applies_to
The source asset describes scenarios, procedures, or issues that manifest in the target asset.

**Semantics.** The source asset is operationally relevant to the target. This is distinct from `references` (generic mention) and `relates_to` (catch-all). Use `applies_to` when the source asset's primary purpose is to describe how to handle, mitigate, or diagnose something that affects the target.

**Example.** A Playbook asset `applies_to` a Service asset. A Troubleshooting asset `applies_to` a Service asset. A Playbook `applies_to` the services involved in the operational scenario it covers.

### 5.10 Implementation
An EKS implementation MAY represent relationships in any format, including file references, graph edges, metadata fields, or links in the asset body. The representation MUST keep the relationships discoverable.

---

## 6. Lifecycle
A Knowledge Asset moves through a series of lifecycle states. The lifecycle captures the asset's maturity and current relevance.

EKM defines the following states. Implementations MAY introduce additional states, but the following MUST be supported.

### 6.1 Draft
The asset is being authored. It has not yet been reviewed.

**Purpose.** Allow authors to develop content without representing it as authoritative.

### 6.2 Review
The asset is under review for adoption. Reviewers have not yet approved it as current.

**Purpose.** Allow a stable version to be examined and accepted or rejected before it becomes authoritative.

### 6.3 Accepted
The asset is approved and current. It is used as a source of truth.

**Purpose.** Indicate that the asset represents the team's current understanding.

### 6.4 Deprecated
The asset is no longer current but is retained for context.

**Purpose.** Preserve historical knowledge while signaling that the asset SHOULD NOT be used as the basis for new work. The asset SHOULD link to a replacement if one exists.

### 6.5 Archived
The asset is no longer relevant to the system. It is retained only for historical reference.

**Purpose.** Remove the asset from active use while preserving it for retrospective analysis.

---

## 7. Ownership
A Knowledge Asset SHOULD have an owner. The owner is the team or individual responsible for keeping the asset current.

Ownership is a logical concept. EKM does not define how ownership is stored; implementations MAY represent it as a metadata field, a separate file, an external system, or any other mechanism.

When an asset changes, the owner SHOULD be notified. When an asset has no owner, it MAY be considered abandoned and SHOULD be reviewed for deprecation or removal.

---

## 8. Traceability
A Knowledge Asset SHOULD reference other relevant assets where appropriate. Traceability supports navigation and impact analysis.

Traceability is a logical concept. EKM does not require any specific linking mechanism; implementations MAY represent references as file paths, hyperlinks, metadata fields, graph edges, or any other mechanism.

The level of traceability is left to the implementation. A minimal implementation MAY omit cross-references entirely; a more thorough implementation SHOULD include them where they aid navigation and impact analysis.

---

## 9. References
- [SPECIFICATION.md](SPECIFICATION.md) — the normative rules for EKS-compatible repositories.
- [HANDBOOK.md](HANDBOOK.md) — informative guidance for adopting EKS in practice.
- The EKS reference repository (this repository) — a file-based implementation of EKM and SPECIFICATION.
