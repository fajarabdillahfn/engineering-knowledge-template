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
| Companion Document | KNOWLEDGE_MODEL.md |

---

## 1. Introduction

### 1.1 Purpose
The Engineering Knowledge Specification (EKS) defines the rules an EKS-compatible repository MUST follow. It is normative for compliance, not for the conceptual model itself: the conceptual model — what a Knowledge Asset is, what types exist, and how they relate — is defined in [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md).

An EKS-compatible repository is one that satisfies the Conformance section (§4) of this document and represents the conceptual entities defined in [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md).

### 1.2 Intended Audience
EKS is written for engineering teams adopting a knowledge organization practice, authors of tools that consume or produce EKS-compliant repositories, and reviewers evaluating the specification itself.

### 1.3 Non-Goals
EKS does not define:
- The conceptual model of engineering knowledge (see [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md)).
- A specific metadata schema, frontmatter format, or validation tool.
- A knowledge graph format or query language.
- A replacement for design documents tied to a single change.
- Project-specific content.

---

## 2. Normative Language

This specification follows the terminology of RFC 2119. The following keywords appear in this document and are to be interpreted exactly as defined here.

- **MUST** — the requirement is absolute. Compliant knowledge bases and repositories MUST satisfy it.
- **MUST NOT** — the prohibition is absolute.
- **SHOULD** — the requirement is recommended. Valid reasons to ignore it exist, but the implications MUST be understood.
- **SHOULD NOT** — the prohibition is recommended. Valid reasons to ignore it exist, but the implications MUST be understood.
- **MAY** — the item is truly optional. Knowledge bases and repositories MAY include or omit it at their discretion.

When a requirement is not marked with one of these keywords, it is descriptive rather than normative.

---

## 3. Design Principles

EKS is built on the following principles. Each principle states a position, the reason it exists, and the practical implications for compliant knowledge bases.

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

## 4. Conformance

An **EKS-compliant repository** is one that satisfies the requirements of this section. The conceptual entities it represents are defined in [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md); the rules for representing them are defined here.

### 4.1 Mandatory
An EKS-compliant repository MUST:
1. Represent the seven knowledge asset types defined in [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) §4 (Asset Types).
2. Provide an Architecture asset, or, if the system consists of a single Service, a Service asset that satisfies the same role.
3. Place each knowledge asset in the location appropriate for its type, as specified in §5 (Repository Structure).
4. Use stable, descriptive identifiers per §5.2 (Naming).
5. Keep one concern per asset per §5.3 (One Concern Per Asset).
6. Keep the relationships between assets discoverable per §5.4 (Cross-References).

### 4.2 Recommended
An EKS-compliant repository SHOULD:
1. Provide a starter document for each knowledge asset type, located in the corresponding location.
2. Cross-reference related assets rather than duplicating content.
3. Maintain engineering standards knowledge assets.
4. Update knowledge assets when the behavior they describe changes.
5. Record ownership of each knowledge asset per [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) §7 (Ownership).
6. Capture lifecycle state per [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) §6 (Lifecycle).

### 4.3 Optional
An EKS-compliant repository MAY:
1. Add additional subdirectories for project-specific knowledge asset types, provided the seven required types remain at their canonical locations.
2. Add tooling for validation, retrieval, or visualization.
3. Add additional files and directories not defined by this specification, provided the mandatory rules remain satisfied.
4. Organize files in a layout that differs from §5.1 (Canonical Layout), as long as the seven required types remain present and discoverable.

---

## 5. Repository Structure

This section describes how a knowledge base MAY be organized in a repository. EKS does not mandate a particular file-system layout; it requires only that the seven knowledge asset types defined in [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) be present and locatable.

### 5.1 Canonical Layout
A reference implementation of EKS MAY use the following canonical layout. Other layouts that satisfy §4 (Conformance) are permitted.

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

### 5.2 Naming
- A Service asset MUST be named after the service it describes.
- A Flow asset MUST be named after the flow it describes.
- A Decision asset MUST follow the format `ADR-NNNN-<title>.md`, where `NNNN` is a zero-padded sequence number.
- A Playbook asset MUST be named after the operational scenario it describes.
- A Troubleshooting asset MUST be named after the issue it describes.
- Knowledge asset file names SHOULD use lowercase kebab-case.

### 5.3 One Concern Per Asset
Each knowledge asset MUST address a single concern. An asset that grows past its concern MUST be split into multiple assets, not extended.

### 5.4 Cross-References
Knowledge assets MAY reference other assets by relative path. An asset that references another MUST keep the reference accurate when the referenced asset is renamed or moved.

---

## 6. Versioning

EKS follows semantic versioning.

- **Major versions** introduce changes that invalidate assets written for the previous major version. A repository MUST be revalidated after a major upgrade.
- **Minor versions** introduce backward-compatible additions, such as new asset types or new SHOULD requirements that existing assets already satisfy.
- **Patch versions** introduce clarifications, typo fixes, and other changes with no semantic effect on compliant assets.

Versions numbered `0.x`, including this document, are pre-release. They MAY contain breaking changes between minor versions. Projects adopting a `0.x` version SHOULD pin a specific minor version and review the changelog on upgrade.

---

## 7. Future Extensions

Future versions of EKS MAY introduce the following as optional extensions. They are NOT part of EKS v0.1 and MUST NOT be required for v0.1 conformance.

- **Metadata.** Asset frontmatter, classification, and tagging conventions.
- **Validation.** Linting and structural checks for EKS assets.
- **Schemas.** Machine-readable definitions of asset structure.
- **Knowledge Graph.** Formalized, queryable relationships between assets, building on the relationship types defined in [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) §5.
- **Search.** Reference indexing and search tooling.
- **CLI.** Reference tooling for creating and navigating EKS repositories.
- **AI Integrations.** Reference patterns for AI coding agents to consume EKS repositories.

Adoption of any future extension MUST NOT invalidate v0.1-compliant assets. A v0.1-compliant repository MUST continue to satisfy §4.1 regardless of which future extensions it adopts.

---

## 8. References

- RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels.
- [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) — the conceptual model underlying this specification.
- The `README.md`, `HANDBOOK.md`, `AGENTS.md`, and `knowledge/README.md` documents in the EKS reference repository are informative companions to this specification.
