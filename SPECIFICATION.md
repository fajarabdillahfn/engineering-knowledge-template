# Engineering Knowledge Specification (EKS)

## Document Metadata

| Field | Value |
| --- | --- |
| Title | Engineering Knowledge Specification |
| Specification Version | v0.1.0 Draft |
| Status | Draft |
| Last Updated | 2026-07-21 |
| Compatibility | EKS v0.1 |
| Intended Audience | Engineering teams adopting EKS, authors of EKS-compatible tools, specification reviewers |

---

## 1. Introduction

### 1.1 Purpose
The Engineering Knowledge Specification (EKS) defines a structure for organizing engineering knowledge in software projects. The structure is intended to be human-maintainable and machine-retrievable, independent of any specific tool, framework, or vendor.

EKS describes how knowledge assets are organized and how they relate to each other. It does not prescribe a runtime, a query language, or any specific implementation.

### 1.2 Intended Audience
EKS is written for engineering teams adopting a knowledge organization practice, authors of tools that consume or produce EKS-compliant repositories, and reviewers evaluating the specification itself.

### 1.3 Non-Goals
EKS does not define:
- A wiki, CMS, or content management implementation.
- A specific metadata schema, frontmatter format, or validation tool.
- A knowledge graph format or query language.
- A replacement for design documents tied to a single change.
- Project-specific content.

---

## 2. Normative Language

This specification follows the terminology of RFC 2119. The following keywords appear in this document and are to be interpreted exactly as defined here.

- **MUST** — the requirement is absolute. Compliant knowledge assets and repositories MUST satisfy it.
- **MUST NOT** — the prohibition is absolute.
- **SHOULD** — the requirement is recommended. Valid reasons to ignore it exist, but the implications MUST be understood.
- **SHOULD NOT** — the prohibition is recommended. Valid reasons to ignore it exist, but the implications MUST be understood.
- **MAY** — the item is truly optional. Knowledge assets and repositories MAY include or omit it at their discretion.

When a requirement is not marked with one of these keywords, it is descriptive rather than normative.

---

## 3. Design Principles

EKS is built on the following principles. Each principle states a position, the reason it exists, and the practical implications for compliant knowledge assets.

### 3.1 Human Maintainable
Engineering knowledge MUST remain easy to write, review, and maintain by humans.

**Why.** Knowledge that is hard to maintain becomes stale, and stale knowledge is worse than missing knowledge.

**Implications.** Knowledge assets MUST be short. Asset structure MUST be predictable. Updates MUST be a small, reviewable change.

### 3.2 AI Retrievable
Knowledge SHOULD be organized so that automated tools, including AI coding agents, can retrieve only the relevant subset for a given task.

**Why.** Large, monolithic documents exceed the context window of language models and force lossy truncation. Retrieval is the primary mode of consumption for automated tools.

**Implications.** Each knowledge asset MUST address a single concern. Assets MUST carry enough identity to be located by topic. Cross-references MUST be explicit.

### 3.3 Modular Knowledge
Small, focused knowledge assets are preferred over large documents.

**Why.** Modular assets are easier to read, easier to update, and easier to retrieve individually.

**Implications.** A knowledge asset that grows past its concern MUST be split. A new concern SHOULD be expressed as a new asset rather than an addition to an existing one.

### 3.4 Single Responsibility
Each knowledge asset MUST represent one concept.

**Why.** An asset that covers multiple concepts makes the relationships between them implicit, and the asset becomes ambiguous to retrieve.

**Implications.** A knowledge asset's title MUST name a single concept. An asset MUST NOT mix architectural, operational, and historical content.

### 3.5 Architecture Before Implementation
Business and architecture knowledge SHOULD exist before implementation details are recorded.

**Why.** The system is shaped by the business and architectural decisions that preceded the code. Recording implementation without that context loses the rationale.

**Implications.** New projects SHOULD populate the Architecture asset before populating Service assets. Service assets SHOULD be written from the perspective of the system, not the code.

### 3.6 Knowledge Evolves
The knowledge base is expected to evolve with the system it describes.

**Why.** A static specification applied to a living system drifts, and the drift introduces incorrect information.

**Implications.** Knowledge assets MUST be updated when the behavior they describe changes. Deprecated assets MUST be marked as such. Superseded decisions MUST be cross-referenced to their replacements.

### 3.7 Vendor Neutrality
EKS MUST NOT depend on any specific AI model, editor, framework, or tool.

**Why.** A specification tied to a vendor becomes obsolete when the vendor's offerings change.

**Implications.** Examples in this specification are abstract. Concrete tool names, product names, and vendor identifiers MUST NOT appear in this specification or in knowledge assets that comply with it.

---

## 4. Core Concepts

EKS organizes engineering knowledge into seven knowledge asset types. Each type has a defined purpose, a normative set of requirements, a relationship to the other types, and abstract examples.

### 4.1 Architecture
**Purpose.** Describes the system as a whole: its components, boundaries, and how data moves between them.

**MUST.** Capture the system's components, boundaries, and primary data flows. Be the root of the repository's knowledge graph.

**SHOULD.** Be a single asset. Be updated whenever the system shape changes.

**MUST NOT.** Mix architectural content with runbook, troubleshooting, or implementation details.

**Relationship.** Referenced by Flows, Services, Playbooks, and Troubleshooting assets. References Decisions.

**Examples.** A multi-component system overview; a description of boundaries between subsystems.

### 4.2 Service
**Purpose.** Describes a single deployable or independently operable unit.

**MUST.** Identify what the service does, what it depends on, and how it is operated. Be associated with one or more owners.

**SHOULD.** Document interfaces, scaling behavior, and observability expectations.

**MUST NOT.** Contain business rules that belong in the Architecture asset, or operational steps that belong in a Playbook.

**Relationship.** A child of Architecture. Referenced by Flows, Playbooks, and Troubleshooting assets. MAY reference Decisions.

**Examples.** An authentication service; a payment processor; a notification worker.

### 4.3 Flow
**Purpose.** Describes an end-to-end process that crosses one or more service boundaries.

**MUST.** Specify the trigger, the steps, the outcome, and the failure handling. Identify the services involved.

**SHOULD.** Document timing expectations and observability signals.

**MUST NOT.** Document the implementation of any single service in isolation.

**Relationship.** A child of Architecture. References one or more Services. MAY be referenced by Playbooks and Troubleshooting assets. MAY reference Decisions.

**Examples.** A user onboarding sequence; a payment settlement flow; a data import pipeline.

### 4.4 Decision
**Purpose.** Records a binding architectural choice, the context in which it was made, and its consequences.

**MUST.** State the context, the decision, and the consequences. Use a stable, sequential identifier.

**SHOULD.** List the alternatives considered and the reasons they were rejected.

**MUST NOT.** Record transient or trivial choices.

**Relationship.** Orthogonal to the asset hierarchy. MAY be referenced by any other knowledge asset.

**Examples.** Choice of message broker; choice of authentication strategy; choice of data storage.

### 4.5 Playbook
**Purpose.** Describes how to handle a recurring operational scenario.

**MUST.** Specify the preconditions, the ordered steps, and the rollback procedure. Identify the service or services involved.

**SHOULD.** Estimate the time required and the impact of execution.

**MUST NOT.** Document a one-off operation that is not expected to recur.

**Relationship.** Associated with one or more Services. References one or more Flows. MAY reference Decisions.

**Examples.** Database failover procedure; certificate rotation; on-call handoff.

### 4.6 Troubleshooting
**Purpose.** Describes a known issue, its symptoms, root cause, diagnosis, resolution, and prevention.

**MUST.** Identify the issue, the observable symptoms, the root cause, and the resolution. Be associated with the affected service or services.

**SHOULD.** Describe the diagnostic steps and the preventive measures.

**MUST NOT.** Document hypothetical or unresolved issues.

**Relationship.** Associated with one or more Services. References one or more Playbooks. MAY reference Decisions.

**Examples.** Recurring timeout in a specific service; intermittent data loss; authentication failure after a specific deployment.

### 4.7 Glossary
**Purpose.** Defines terms, acronyms, and domain vocabulary used across other EKS knowledge assets.

**MUST.** Define each term with sufficient context for a new team member to understand it.

**SHOULD.** Organize entries alphabetically and link to the assets that use each term.

**MUST NOT.** Define terms with standard, widely-known meanings.

**Relationship.** Orthogonal to the asset hierarchy. Referenced by any asset that uses specialized vocabulary.

**Examples.** Domain-specific terms; project-specific acronyms; non-obvious naming conventions.

---

## 5. Knowledge Graph

EKS does not impose a strict hierarchy. The seven knowledge asset types form a directed graph of relationships:

- Architecture references Flows and Services.
- Flows involve one or more Services.
- Services reference Decisions and link to Playbooks and Troubleshooting assets.
- Playbooks reference Flows.
- Troubleshooting references Services and Playbooks.
- Decisions and Glossary are orthogonal: any asset MAY reference them.

A repository MAY represent this graph using any combination of folders, file names, in-asset links, metadata, or future tooling. The representation MUST keep the relationships discoverable.

---

## 6. Conformance

An **EKS-compliant repository** is one that satisfies the requirements of this section.

### 6.1 Mandatory
An EKS-compliant repository MUST:
1. Use the seven knowledge asset types defined in §4 for the corresponding content.
2. Provide an Architecture asset, or, if the system consists of a single Service, a Service asset that satisfies the same role.
3. Place each knowledge asset in the location appropriate for its type, as specified in §7 (Repository Structure).
4. Use stable, descriptive identifiers per §7.2 (Naming).
5. Keep one concern per asset per §7.3 (One Concern Per Asset).
6. Keep the relationships between assets discoverable per §5 (Knowledge Graph).

### 6.2 Recommended
An EKS-compliant repository SHOULD:
1. Provide a starter document for each knowledge asset type, located in the corresponding location.
2. Cross-reference related assets rather than duplicating content.
3. Maintain engineering standards knowledge assets.
4. Update knowledge assets when the behavior they describe changes.

### 6.3 Optional
An EKS-compliant repository MAY:
1. Add additional subdirectories for project-specific knowledge asset types, provided the seven required types remain at their canonical locations.
2. Add tooling for validation, retrieval, or visualization.
3. Add additional files and directories not defined by this specification, provided the mandatory rules remain satisfied.
4. Organize files in a layout that differs from §7.1 (Canonical Layout), as long as the seven required types remain present and discoverable.

---

## 7. Repository Structure

This section describes how a knowledge base MAY be organized in a repository. EKS does not mandate a particular file-system layout; it requires only that the seven knowledge asset types be present and locatable.

### 7.1 Canonical Layout
A reference implementation of EKS MAY use the following canonical layout. Other layouts that satisfy §6 (Conformance) are permitted.

```
knowledge/
├── architecture.md
├── glossary.md
├── conventions.md
├── decisions/
├── flows/
├── playbooks/
├── services/
└── troubleshooting/
```

The name of the knowledge base directory and its subdirectories MAY vary, but the structure (one file or subdirectory per asset type) MUST be preserved.

### 7.2 Naming
- A Service asset MUST be named after the service it describes.
- A Flow asset MUST be named after the flow it describes.
- A Decision asset MUST follow the format `ADR-NNNN-<title>.md`, where `NNNN` is a zero-padded sequence number.
- A Playbook asset MUST be named after the operational scenario it describes.
- A Troubleshooting asset MUST be named after the issue it describes.
- Knowledge asset file names SHOULD use lowercase kebab-case.

### 7.3 One Concern Per Asset
Each knowledge asset MUST address a single concern. An asset that grows past its concern MUST be split into multiple assets, not extended.

### 7.4 Cross-References
Knowledge assets MAY reference other assets by relative path. An asset that references another MUST keep the reference accurate when the referenced asset is renamed or moved.

---

## 8. Versioning

EKS follows semantic versioning.

- **Major versions** introduce changes that invalidate assets written for the previous major version. A repository MUST be revalidated after a major upgrade.
- **Minor versions** introduce backward-compatible additions, such as new asset types or new SHOULD requirements that existing assets already satisfy.
- **Patch versions** introduce clarifications, typo fixes, and other changes with no semantic effect on compliant assets.

Versions numbered `0.x`, including this document, are pre-release. They MAY contain breaking changes between minor versions. Projects adopting a `0.x` version SHOULD pin a specific minor version and review the changelog on upgrade.

---

## 9. Future Extensions

Future versions of EKS MAY introduce the following as optional extensions. They are NOT part of EKS v0.1 and MUST NOT be required for v0.1 conformance.

- **Metadata.** Asset frontmatter, classification, and tagging conventions.
- **Validation.** Linting and structural checks for EKS assets.
- **Schemas.** Machine-readable definitions of asset structure.
- **Knowledge Graph.** Formalized, queryable relationships between assets.
- **Search.** Reference indexing and search tooling.
- **CLI.** Reference tooling for creating and navigating EKS repositories.
- **AI Integrations.** Reference patterns for AI coding agents to consume EKS repositories.

Adoption of any future extension MUST NOT invalidate v0.1-compliant assets. A v0.1-compliant repository MUST continue to satisfy §6.1 regardless of which future extensions it adopts.

---

## 10. References

- RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels.
- The `README.md`, `HANDBOOK.md`, `AGENTS.md`, and `knowledge/README.md` documents in the EKS reference repository are informative companions to this specification.
