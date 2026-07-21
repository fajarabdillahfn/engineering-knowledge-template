# Knowledge

The `knowledge/` directory is the canonical engineering knowledge base for an EKS-compliant project. It is the source of truth for how the system is understood, operated, and evolved.

## What Belongs Here

Place any document that the team needs to reference repeatedly when working on the system, and that is stable enough to be useful beyond a single ticket. This includes architecture, glossary, conventions, service references, end-to-end flows, troubleshooting guides, operational playbooks, and architectural decisions.

Documents that are not stable, that belong to a single change, or that are conversational in nature do not belong in `knowledge/`. Put them in a pull request description, a design document tied to a ticket, or a chat thread.

## Document Types

Each document in `knowledge/` falls into one of the following types. The type determines the template to start from and the subdirectory the file lives in.

- **Architecture** — `knowledge/architecture.md`. The system as a whole: components, boundaries, and how data moves between them. One file. Updated when the system shape changes.
- **Service** — `knowledge/services/<name>.md`. A single service: what it does, who owns it, what it depends on, and how it is operated. One file per service.
- **Flow** — `knowledge/flows/<name>.md`. An end-to-end process that crosses service boundaries: trigger, steps, outcomes, and failure handling. One file per flow.
- **Decision** — `knowledge/decisions/ADR-NNNN-<title>.md`. An architectural decision record: context, decision, consequences, and alternatives. One file per decision.
- **Playbook** — `knowledge/playbooks/<scenario>.md`. A recurring operational scenario: preconditions, steps, rollback. One file per scenario.
- **Troubleshooting** — `knowledge/troubleshooting/<issue>.md`. A known issue: symptoms, root cause, diagnosis, resolution, prevention. One file per issue.
- **Glossary** — `knowledge/glossary.md`. Domain terms and acronyms. A single growing document, organized alphabetically.
- **Conventions** — `knowledge/conventions.md`. Engineering standards adopted by the team. A single growing document.

## When to Create a New Document

Create a new document when:

- The information cannot be added to an existing document without breaking that document's scope.
- The topic is referenced from more than one place and warrants a stable link.
- The information is expected to remain valid beyond the current change.

If none of these hold, prefer a comment in the code, a pull request description, or a note in an existing document.

## When to Update an Existing Document

Update an existing document when:

- The behavior it describes has changed.
- A term it defines has been added, removed, or redefined.
- A flow, service, or operational procedure it references has been modified.
- A decision it depends on has been superseded by a new ADR.

When updating, preserve the document's structure. Add new information at the appropriate section, and revise outdated sections in place rather than appending corrections below them.

## Best Practices

- **One concern per file.** A document that covers multiple concerns should be split, not extended.
- **Lead with metadata.** The first lines of a document should declare its identity, owner, and status. Do not bury this in prose.
- **Use the templates.** New documents should be created from the corresponding template in the same subdirectory.
- **Link, do not duplicate.** When a document references another, use a relative link rather than copying its content.
- **Match the code.** When documentation and code disagree, fix one to match the other and record the change.
- **Keep documents current.** Stale documentation is worse than missing documentation. Mark documents that are known to be outdated.
