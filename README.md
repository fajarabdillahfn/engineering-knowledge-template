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

**Completed work** (captured here for context, not a release target):

- **Foundation.** Repository layout, asset types, and starter documents.
- **First dogfooding** (URL Shortener, fictional). Surfaced 10 findings (F1–F10); the high-priority ones (`caused_by`, `replaced_by`, `applies_to`, etc.) are now in the spec.
- **Agent-readability validation.** A fresh sub-agent navigates the URL Shortener reference using only its asset graph. The initial test (Q6: 4 hops, Q7: 3 hops) revealed a spec-vs-reality gap (F11). F11 retrofit dropped both to 2 hops. The test is in `reference/url-shortener/AGENT_READABILITY_TEST.md`.
- **Second dogfooding** (`controlroom-inquiry`, real Go+gRPC bill payment service). Surfaced 6 findings (F15–F20). Validates the F13 `out_of_scope` construct (5 explicit non-decisions in one service) and surfaces the F15 federation gap.
- **Metadata design proposal.** 5 conceptual fields proposed in `METADATA_DESIGN.md`. Adoption deferred until the second dogfooding validates the design — which it did.

**Forward-looking work:**

- **v0.2 — Metadata Spec.** Adopt the conceptual metadata fields from `METADATA_DESIGN.md` into `KNOWLEDGE_MODEL.md`. Apply to existing reference projects.
- **v0.3 — Validation Gate.** The agent-readability test (`reference/url-shortener/AGENT_READABILITY_TEST.md`) becomes the reference validation procedure. Every spec change SHOULD pass this test before adoption. The URL Shortener and `controlroom-inquiry` references are the test fixtures.
- **v1.0 — Reference Implementations.** Additional dogfooding projects (real and fictional) exercise the spec under different conditions. New asset types, relationship types, and conceptual refinements emerge from the findings. CLI and tooling MAY be added if they serve the validation discipline; they are not primary deliverables.
