# Engineering Knowledge Specification (EKS)

A specification for organizing engineering knowledge so that it is usable by humans and by AI coding agents.

## What is EKS

EKS defines a small set of knowledge asset types — Architecture, Service, Flow, Decision, Playbook, Troubleshooting, and Glossary — and the relationships between them. Each asset is scoped to a single concern, making it easier to maintain and easier for automated tools to retrieve.

This repository is the reference implementation of EKS. The specification is normative; the reference implementation demonstrates one possible way to satisfy it.

## Problem Statement

Most engineering knowledge is captured in long, monolithic documents. This makes it hard for AI coding agents to retrieve the relevant subset, and hard for humans to keep the knowledge current. EKS addresses this by:

- Defining a small, fixed set of knowledge asset types.
- Scoping each asset to a single concern.
- Specifying the relationships between assets as a graph, not a hierarchy.

## Repository Layout

This repository is organized in three layers:

- **Specification.** [SPECIFICATION.md](SPECIFICATION.md) defines the rules an EKS-compliant repository must follow; [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) defines the conceptual model those rules implement. Both are normative.
- **Methodology.** [HANDBOOK.md](HANDBOOK.md) explains how to adopt and use EKS in practice. [AGENTS.md](AGENTS.md) gives AI coding agents specific guidance. Both are informative.
- **Reference implementation.** The `knowledge/`, `prompts/`, `examples/`, and `.github/` directories show one possible layout. Repositories MAY organize files differently while remaining EKS-compliant.

## Getting Started

1. Read [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) to understand the conceptual model.
2. Read [SPECIFICATION.md](SPECIFICATION.md) to understand the rules.
3. Read [HANDBOOK.md](HANDBOOK.md) to learn the recommended workflow.
4. If you are an AI coding agent, also read [AGENTS.md](AGENTS.md).
5. Adopt EKS in your project incrementally — see the Handbook's adoption strategy.

## Where to Read Next

- [KNOWLEDGE_MODEL.md](KNOWLEDGE_MODEL.md) — the conceptual model.
- [SPECIFICATION.md](SPECIFICATION.md) — the normative specification.
- [HANDBOOK.md](HANDBOOK.md) — methodology and best practices.
- [AGENTS.md](AGENTS.md) — guidance for AI coding agents.
- [knowledge/](knowledge/) — the reference knowledge base.

## Contributing

Feedback and proposals are welcome through issues and pull requests. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Roadmap

- **v0.1 — Foundation.** Repository layout, asset types, and starter documents. *Current.*
- **v0.2 — Metadata Spec.** Frontmatter and asset classification.
- **v0.3 — Validation.** Structural checks for EKS assets.
- **v1.0 — AI Agent Ecosystem.** Reference agents and tooling.
