# Knowledge

The `knowledge/` directory is the canonical engineering knowledge base for an EKS-compliant project. It is the source of truth for how the system is understood, operated, and evolved.

## What Belongs Here

Place any knowledge asset that the team needs to reference repeatedly when working on the system, and that is stable enough to be useful beyond a single ticket. This includes architecture, glossary, conventions, service references, end-to-end flows, troubleshooting entries, operational playbooks, and architectural decisions.

Knowledge assets that are not stable, that belong to a single change, or that are conversational in nature do not belong in `knowledge/`. Put them in a pull request description, a design asset tied to a ticket, or a chat thread.

## Document Types

Each knowledge asset in `knowledge/` falls into one of the following types. The type determines the starter document to use and the subdirectory the file lives in. (Starter documents in this repository follow the `_template.md` naming convention.)

- **Architecture** — `knowledge/architecture.md`. The system as a whole: components, boundaries, and how data moves between them. One file. Updated when the system shape changes.
- **Service** — `knowledge/services/<name>.md`. A single service: what it does, who owns it, what it depends on, and how it is operated. One file per service.
- **Flow** — `knowledge/flows/<name>.md`. An end-to-end process that crosses service boundaries: trigger, steps, outcomes, and failure handling. One file per flow.
- **Decision** — `knowledge/decisions/ADR-NNNN-<title>.md`. An architectural decision record: context, decision, consequences, and alternatives. One file per decision.
- **Playbook** — `knowledge/playbooks/<scenario>.md`. A recurring operational scenario: preconditions, steps, rollback. One file per scenario.
- **Troubleshooting** — `knowledge/troubleshooting/<issue>.md`. A known issue: symptoms, root cause, diagnosis, resolution, prevention. One file per issue.
- **Glossary** — `knowledge/glossary.md`. Domain terms and acronyms. A single growing asset, organized alphabetically.
- **Conventions** — `knowledge/conventions.md`. Engineering standards adopted by the team. A single growing asset.

## When to Create a New Knowledge Asset

Create a new knowledge asset when:

- The information cannot be added to an existing asset without breaking that asset's scope.
- The topic is referenced from more than one place and warrants a stable link.
- The information is expected to remain valid beyond the current change.

If none of these hold, prefer a comment in the code, a pull request description, or a note in an existing asset.

## When to Update an Existing Knowledge Asset

Update an existing knowledge asset when:

- The behavior it describes has changed.
- A term it defines has been added, removed, or redefined.
- A flow, service, or operational procedure it references has been modified.
- A decision it depends on has been superseded by a new ADR.

When updating, preserve the asset's structure. Add new information at the appropriate section, and revise outdated sections in place rather than appending corrections below them.

## Best Practices

- **One concern per asset.** A knowledge asset that covers multiple concerns should be split, not extended.
- **Lead with metadata.** The first lines of a knowledge asset should declare its identity, owner, and status. Do not bury this in prose.
- **Use the starter documents.** New knowledge assets should be created from the corresponding starter document in the same subdirectory.
- **Link, do not duplicate.** When a knowledge asset references another, use a relative link rather than copying its content.
- **Match the code.** When knowledge assets and code disagree, fix one to match the other and record the change.
- **Keep assets current.** Stale knowledge assets are worse than missing ones. Mark assets that are known to be outdated.
