# Agent Instructions

Instructions for AI coding agents operating in an EKS-compliant repository.

## Purpose

This file describes how an AI agent should navigate and contribute to an EKS knowledge base. It assumes the agent can read files, search the repository, and propose edits, but cannot rely on prior knowledge of the project.

## Reading Order

When entering an unfamiliar EKS repository, read the following documents in this order. Stop after each step if it is sufficient to answer the current question; otherwise continue.

1. `README.md` — the project homepage and entry point.
2. `SPECIFICATION.md` — the normative rules. Read this whenever a conformance question arises.
3. `HANDBOOK.md` — the methodology and recommended workflow. Read this for guidance on how to adopt and maintain EKS.
4. `knowledge/README.md` — the structure and intent of the reference knowledge base.
5. `knowledge/architecture.md` — the system's components, boundaries, and data flow.
6. `knowledge/glossary.md` — the project's vocabulary and acronym definitions.
7. `knowledge/flows/` — end-to-end processes that cross service boundaries.
8. `knowledge/services/` — per-service references for any service involved in the task.
9. `knowledge/playbooks/` — operational procedures when the task is operational rather than developmental.

Read additional assets (`conventions.md`, `troubleshooting/`, `decisions/`) on demand, when the task is clearly about conventions, known issues, or historical decisions.

## Principles

- **Prefer existing knowledge over assumptions.** If a knowledge asset answers the question, use it. Do not regenerate information that already lives in the knowledge base.
- **Surface conflicts.** If a knowledge asset contradicts observed code or behavior, flag the conflict explicitly rather than silently picking a side. Cite the asset and the location in code.
- **Never invent architecture.** Do not describe components, services, or flows that are not documented in the knowledge base or directly visible in the code. If something is missing, say so.
- **Prefer updating existing assets over creating duplicates.** Search the knowledge base for a relevant asset before creating a new one. A new asset is only justified when no existing asset covers the concern.
- **Keep assets small and focused.** An asset that grows past a single concern should be split. Do not append unrelated material to an existing asset.
- **Respect scope.** `prompts/`, `schemas/`, and `examples/` have their own conventions. Read them before modifying.

## Output Expectations

- **Explain reasoning.** When proposing a change, summarize what was found, what was inferred, and what remains uncertain.
- **Reference assets.** When a claim depends on a knowledge asset, cite it by relative path and section. When a claim depends on code, cite file and line.
- **Distinguish asset from code.** If a knowledge asset and implementation disagree, name both. Do not silently rewrite one to match the other.
- **Propose, do not silently change.** Suggestions to `knowledge/` should be presented as proposed edits with rationale, not applied without review.
