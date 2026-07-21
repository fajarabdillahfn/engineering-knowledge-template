# Agent Instructions

Instructions for AI coding agents operating in an EKS-compliant repository.

## Purpose

This file describes how an AI agent should navigate and contribute to an EKS knowledge base. It assumes the agent can read files, search the repository, and propose edits, but cannot rely on prior knowledge of the project.

## Reading Order

When entering an unfamiliar EKS repository, read the following documents in this order. Stop after each step if it is sufficient to answer the current question; otherwise continue.

1. `README.md` — the specification overview and the project's adopted conventions.
2. `knowledge/README.md` — the structure and intent of the knowledge base.
3. `knowledge/architecture.md` — the system's components, boundaries, and data flow.
4. `knowledge/glossary.md` — the project's vocabulary and acronym definitions.
5. `knowledge/flows/` — end-to-end processes that cross service boundaries.
6. `knowledge/services/` — per-service references for any service involved in the task.
7. `knowledge/playbooks/` — operational procedures when the task is operational rather than developmental.

Read additional documents (`conventions.md`, `troubleshooting/`, `decisions/`) on demand, when the task is clearly about conventions, known issues, or historical decisions.

## Guidelines

- **Prefer existing knowledge over assumptions.** If a document answers the question, use it. Do not regenerate information that already lives in the repository.
- **Surface conflicts.** If a document contradicts observed code or behavior, flag the conflict explicitly rather than silently picking a side. Cite the document and the location in code.
- **Never invent architecture.** Do not describe components, services, or flows that are not documented or directly visible in the code. If something is missing, say so.
- **Prefer updating existing documents over creating duplicates.** Search the repository for a relevant file before creating a new one. A new document is only justified when no existing document covers the concern.
- **Keep documents small and focused.** A document that grows past a single concern should be split. Do not append unrelated material to an existing file.
- **Respect scope.** `prompts/`, `schemas/`, and `examples/` have their own conventions. Read them before modifying.

## Output Expectations

- **Explain reasoning.** When proposing a change, summarize what was found, what was inferred, and what remains uncertain.
- **Reference documents.** When a claim depends on a document, cite it by relative path and section. When a claim depends on code, cite file and line.
- **Distinguish document from code.** If documentation and implementation disagree, name both. Do not silently rewrite one to match the other.
- **Propose, do not silently change.** Suggestions to `knowledge/` should be presented as proposed edits with rationale, not applied without review.
