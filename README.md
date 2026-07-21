# Engineering Knowledge Specification (EKS)

A specification for organizing engineering knowledge so that it is usable by humans and by AI coding agents.

## Problem Statement

Most engineering documentation is written for human readers in long, monolithic documents. This creates problems for AI coding agents and, over time, for humans as well:

- Long documents exceed the context window of language models and force lossy truncation.
- Multiple concerns are mixed in a single file: architecture, runbooks, and history are not separated.
- Documents lack consistent metadata, so retrieval depends on fragile keyword matching.
- Information drifts from implementation, and there is no signal when a document is stale.
- Implicit context is left to the reader, which a model cannot infer reliably.

The result is that AI agents either ignore the documentation, hallucinate structure, or produce work that contradicts the team's actual practices. Humans face the same retrieval problem, but compensate through experience.

## Goals

EKS is designed to be useful for three audiences simultaneously:

- **Humans.** Clear, navigable, and easy to keep up to date.
- **AI Coding Agents.** Structured, retrievable, and unambiguous at the document level.
- **Future Engineering Teams.** Institutional knowledge that survives turnover and is not locked inside any one person's head.

## Core Principles

- **Documentation is retrievable.** Documents are scoped so they can be located by topic, not by guessing.
- **Knowledge is modular.** Each document covers one concern. Concerns are not interleaved.
- **Small files over large documents.** A document that grows past its scope is split, not extended.
- **Metadata before prose.** A document's identity and scope are declared at the top, not buried in text.
- **Architecture before implementation.** The system is described before any code is referenced.
- **Business knowledge is first-class documentation.** Domain context is documented with the same discipline as technical context.

## Repository Structure

- `knowledge/` — The engineering knowledge base: architecture, glossary, conventions, services, flows, troubleshooting, playbooks, and decisions.
- `prompts/` — Reusable prompt templates for common engineering workflows: investigate, implement, review, and refactor.
- `schemas/` — Reserved for document and metadata schemas. Empty in v0.1.
- `examples/` — Reserved for worked examples of compliant documents. Empty in v0.1.
- `.github/` — Issue and pull request templates for the project that hosts EKS.

## Quick Start

To adopt EKS in a new project:

1. Copy this repository (or at minimum the `knowledge/`, `prompts/`, and `.github/` directories) into the target project.
2. Replace the placeholder content in `knowledge/` with project-specific documentation, one file per concern.
3. Keep the directory structure intact. Add subdirectories only when a genuinely new document type emerges.
4. Use the templates in `knowledge/services/`, `knowledge/flows/`, and so on as the starting point for new documents.
5. Treat `knowledge/` as the source of truth: changes to behavior are followed by changes to the relevant documents.

## Roadmap

- **v0.1 — Foundation.** Repository layout, document types, and templates. *Current.*
- **v0.2 — Metadata Spec.** Required frontmatter and document classification rules.
- **v0.3 — Validation.** Linting and structural checks for EKS documents.
- **v1.0 — AI Agent Ecosystem.** Reference agents, retrieval recipes, and tooling integrations.

## Contribution

EKS is expected to evolve. Feedback, corrections, and proposals are welcome through issues and pull requests. Changes to the specification are discussed openly and recorded as ADRs in `knowledge/decisions/`.
