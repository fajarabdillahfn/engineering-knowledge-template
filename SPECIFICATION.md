# Engineering Knowledge Specification (EKS) — Version 0.1

## 1. Introduction

### 1.1 Purpose
The Engineering Knowledge Specification (EKS) defines a file-system-based structure for organizing engineering knowledge in software projects. The structure is intended to be:
- Human-navigable.
- Machine-retrievable.
- Vendor-neutral.
- Tooling-independent.

EKS describes how documents are organized and how they relate to each other. It does not prescribe a runtime, a query language, or any specific implementation.

### 1.2 Goals
EKS exists to:
- Provide a stable, predictable layout for engineering knowledge across projects.
- Allow humans and automated tools, including AI coding agents, to locate relevant knowledge with minimal context.
- Preserve institutional knowledge through scoped, reviewable documents.
- Decouple knowledge organization from any specific tool, framework, platform, or vendor.

### 1.3 Non-Goals
EKS does not define:
- A wiki, CMS, or content management implementation.
- A specific metadata schema, frontmatter format, or validation tool.
- A replacement for design documents tied to a single change.
- A knowledge graph format or query language.
- Project-specific content.

### 1.4 Conformance
An **EKS-compatible repository** is one that follows the Repository Model (§3) and Repository Requirements (§4) of this specification.

A repository MAY extend EKS with additional subdirectories, files, or metadata, provided the core requirements remain satisfied.

### 1.5 Normative Language
The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this document are to be interpreted as described in RFC 2119.

---

## 2. Core Concepts

EKS organizes engineering knowledge into seven document types. Each type has a defined purpose, scope, and relationship to other types.

### 2.1 Architecture
**Purpose.** A document describing the system as a whole: its components, boundaries, and how data moves between them.

**When to create one.** When the system has identifiable components, boundaries, or data flows that benefit from explicit documentation.

**When NOT to create one.** When the system is trivial (a single component with no meaningful internal structure). In that case, a single Service document satisfies the role.

**Relationship.** The Architecture document is the parent of all Service, Flow, Playbook, and Troubleshooting documents. It is the root of the document hierarchy.

### 2.2 Service
**Purpose.** A document describing a single deployable or independently operable unit: what it does, what it depends on, and how it is operated.

**When to create one.** When a component is deployed, scaled, or operated as a unit.

**When NOT to create one.** For internal libraries, modules, or helpers that have no operational concerns. Document those inside an Architecture or another Service.

**Relationship.** A Service is a child of Architecture. A Service may be referenced by zero or more Flows. A Service may have one or more child Playbooks and Troubleshooting entries.

### 2.3 Flow
**Purpose.** A document describing an end-to-end process that crosses one or more service boundaries, including its trigger, steps, outcomes, and failure handling.

**When to create one.** When a process involves more than one service in sequence, or when a single-service process is important enough to warrant an independent document.

**When NOT to create one.** For actions confined to a single service. Document those inside the Service document.

**Relationship.** A Flow is a child of Architecture. A Flow references one or more Services. A Flow may be associated with one or more Playbooks and Troubleshooting entries.

### 2.4 Decision
**Purpose.** An Architectural Decision Record (ADR) capturing a binding choice, the context in which it was made, the consequences, and the alternatives considered.

**When to create one.** When a choice is binding across the system and is expected to remain in force long enough to warrant a record.

**When NOT to create one.** For trivial implementation details, transient decisions, or choices that are not yet binding.

**Relationship.** A Decision is orthogonal to the hierarchy. It MAY be referenced by Architecture, Service, Flow, Playbook, and Troubleshooting documents.

### 2.5 Playbook
**Purpose.** A document describing how to handle a recurring operational scenario, including preconditions, steps, and rollback procedures.

**When to create one.** When a scenario is expected to recur and benefits from a stable, reviewable procedure.

**When NOT to create one.** For one-off operations, ad-hoc investigations, or procedures confined to a single ticket.

**Relationship.** A Playbook is associated with one or more Services. A Playbook references Architecture and MAY reference one or more Decisions.

### 2.6 Troubleshooting
**Purpose.** A document describing a known issue, its observable symptoms, root cause, diagnosis, resolution, and prevention.

**When to create one.** When an issue has been observed, root-caused, and resolved, and the knowledge is expected to be useful again in the future.

**When NOT to create one.** For hypothetical issues or issues still under investigation. Use a ticket or design document until the issue is resolved.

**Relationship.** A Troubleshooting entry is associated with one or more Services. It MAY reference Playbooks and Decisions.

### 2.7 Glossary
**Purpose.** A document defining terms, acronyms, and domain vocabulary used across other EKS documents.

**When to create one.** When a term is used in more than one document and has a non-obvious or project-specific meaning.

**When NOT to create one.** For terms with standard, widely-known meanings.

**Relationship.** The Glossary is referenced by all other document types. It is not part of the hierarchy itself.

---

## 3. Repository Model

### 3.1 Document Hierarchy
EKS organizes documents in the following hierarchy, from most abstract to most concrete:

```
Architecture
   ↓
  Flow
   ↓
Service
   ↓
Playbook
   ↓
Troubleshooting
```

- **Architecture** describes the whole system.
- **Flow** describes cross-service processes.
- **Service** describes individual units.
- **Playbook** describes operational procedures.
- **Troubleshooting** describes known issues.

Each level inherits context from the level above it. The **Decision** and **Glossary** document types are orthogonal to this hierarchy: they MAY be referenced by any document.

### 3.2 Directory Layout
An EKS-compatible repository MUST organize documents into a knowledge base directory with the following canonical layout:

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

The name of the knowledge base directory and its subdirectories MAY vary, but the structure (one file or subdirectory per document type) MUST be preserved. The seven document types defined in §2 MUST each be represented.

### 3.3 Document Naming
- A Service document MUST be named after the service it describes.
- A Flow document MUST be named after the flow it describes.
- A Decision document MUST follow the format `ADR-NNNN-<title>.md`, where `NNNN` is a zero-padded sequence number.
- A Playbook document MUST be named after the operational scenario it describes.
- A Troubleshooting document MUST be named after the issue it describes.
- Document file names SHOULD use lowercase kebab-case.

### 3.4 Cross-References
Documents MAY reference other documents by relative path. A document that references another MUST keep the reference accurate when the referenced document is renamed or moved.

### 3.5 One Concern Per File
Each document MUST address a single concern. A document that grows past its concern MUST be split into multiple documents, not extended.

---

## 4. Repository Requirements

### 4.1 Mandatory
An EKS-compatible repository MUST:
1. Contain a knowledge base directory organized per §3.2.
2. Include an Architecture document, unless the system consists of a single Service, in which case the Service document satisfies the role.
3. Use the seven document types defined in §2 for the corresponding content.
4. Place each document in the subdirectory corresponding to its type.
5. Use stable, descriptive file names per §3.3.
6. Keep one concern per file per §3.5.

### 4.2 Recommended
An EKS-compatible repository SHOULD:
1. Provide a template for each document type, located in the corresponding subdirectory.
2. Cross-reference related documents explicitly rather than duplicating their content.
3. Use the ADR format described in §2.4 for Decision documents.
4. Maintain a `conventions.md` document covering engineering standards.
5. Update documents whenever the behavior they describe changes.

### 4.3 Optional
An EKS-compatible repository MAY:
1. Add metadata frontmatter to documents.
2. Add a `schemas/` directory for document and data schemas.
3. Add an `examples/` directory of completed documents.
4. Add a `prompts/` directory of reusable prompt templates.
5. Add tooling for validation, retrieval, or visualization.
6. Add additional subdirectories for project-specific document types, provided the seven required types remain at their canonical locations.

---

## 5. Future Compatibility

This is version 0.1 of the specification. Future versions MAY introduce:

- **Metadata.** Required frontmatter, document classification, and tagging conventions.
- **Validation.** Linting and structural checks for EKS documents.
- **Knowledge Graph.** Formalized relationships between documents, queryable across repositories.
- **CLI.** Reference tooling for creating, validating, and navigating EKS repositories.
- **AI Integrations.** Reference patterns and recipes for AI coding agents to consume EKS repositories.

These additions MUST NOT invalidate v0.1-compliant repositories. A v0.1-compliant repository MUST continue to satisfy the requirements of §4.1 regardless of which future features it adopts.

A document that opts in to a future feature MUST remain readable by v0.1 tools, except for content placed inside features the v0.1 tool does not understand.

---

## 6. References

- RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels.
- The `README.md`, `AGENTS.md`, and `knowledge/README.md` documents in the EKS reference repository provide a non-normative introduction to v0.1.
