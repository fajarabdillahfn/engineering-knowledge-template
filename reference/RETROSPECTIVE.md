# Retrospective

Observations from creating the URL Shortener reference project. This document captures friction encountered while using the EKS specification to document a realistic engineering system.

The goal is honest reporting. Where the specification worked well, that is noted. Where it was awkward or forced, that is also noted. Findings and proposed improvements are in `FINDINGS.md`.

## Where the specification felt natural

- **Asset boundaries held up.** Each asset type had a clear role. Architecture captured the system overview without bleeding into per-service detail. Services stayed focused on a single unit. Flows captured the end-to-end process without re-stating what services do in isolation.
- **Relationship vocabulary was expressive.** The six defined relationship types (references, documents, depends_on, implements, relates_to, supersedes) covered most of the connections I wanted to express. The natural English phrasing "the playbook references the flow" mapped cleanly onto the formal relationship.
- **The seven types were enough.** I did not feel the need to invent a new type. The system fit naturally into Architecture, Service, Flow, Decision, Playbook, Troubleshooting, and Glossary.
- **Lifecycle states made sense.** Draft, Review, Accepted, Deprecated, Archived was a comfortable sequence. I did not feel any state was missing for the assets I created.
- **The "one concern per file" rule produced tight, scannable documents.** Each asset is short enough to read in one sitting. None exceeded 300 lines.

## Where documentation overlapped

- **The cache strategy appears in three places.** It is mentioned in `architecture.md` (as part of the data flow), in `decisions/ADR-0002-read-through-cache.md` (as a binding choice), and in `services/redirect-service.md` (as a service responsibility). Each mention has a different audience and a different purpose, so the overlap is justified — but the duplication was noticeable when writing.
- **The url-redirection flow repeats steps that are also in `redirect-service.md`.** The flow describes the steps end-to-end; the service describes its responsibilities. The overlap is small but exists. Both are useful.
- **The Glossary overlaps with content in other assets.** For example, "Read-Through Cache" is defined in `glossary.md` and is also the subject of ADR-0002. The glossary entry is shorter and intended as a quick reference; the ADR is the full story. This is acceptable, but worth noting in the spec.

## Which asset type was hardest to write

**Troubleshooting.** It was difficult to choose between putting an issue in a Troubleshooting asset vs. capturing the response in a Playbook. For example, the cache stampede scenario: is it an issue (Troubleshooting) or an operational scenario (Playbook)?

I ended up putting the playbook first (mitigation steps) and writing a Troubleshooting asset that pointed to the playbook for resolution. This worked, but the line between "known issue" and "recurring operational scenario" is fuzzy. The spec says Troubleshooting is for "known issues" and Playbook is for "recurring operational scenarios", but in practice the same situation can be both.

## Were any relationship types missing

- **I wanted a `caused_by` relationship.** A Troubleshooting asset describes an issue; I wanted to say "this issue is `caused_by` the cache strategy in ADR-0002". The closest existing type is `depends_on` (used in the troubleshooting entry to point to the playbook), but that is about correctness, not causality. `relates_to` is generic. Neither expresses "this issue exists because of this design choice".
- **I wanted a `replaced_by` relationship (inverse of `supersedes`).** The spec defines `supersedes` (source replaces target) but not the inverse. When an asset is replaced, the new asset links to the old one with `supersedes`. It would also be useful for the old asset to link to the new one with `replaced_by`, so a reader of the old asset can find its replacement.
- **I caught myself using a non-spec relationship type during self-review.** While writing the playbook and troubleshooting entries, I instinctively wrote `associated_with` to mean "this asset is operationally relevant to that asset". I then noticed this is not in the spec's vocabulary and changed it to `references`, but `references` does not capture the "operationally relevant to" semantics. This slip is itself evidence of a gap. See FINDINGS F10.

## Were any concepts ambiguous

- **What is a "Service"?** The spec says it is a single deployable or independently operable unit. In the URL Shortener, api-service and redirect-service are clearly separate (different binaries, different scaling profiles). But what about analytics-service, which has multiple worker types? Is each worker a Service, or is the whole analytics layer a Service? I treated the analytics layer as a single Service, but this is a judgment call. A future addition of the `composed_of` relationship might help.
- **What is the boundary between Flow and Service?** Both describe behavior. The spec says Flow is for cross-service processes and Service is for the unit. In the URL Shortener, the url-shortening flow is contained within api-service. Strictly, it is not cross-service. But it is still a useful artifact to document, especially for new engineers.
- **What is the difference between a Flow and a Playbook?** A Flow describes a normal path; a Playbook describes an operational procedure. But the cache stampede mitigation could be either. I treated the cache stampede as a Playbook because it is invoked when something goes wrong, not as part of normal operation.

## Did any information feel duplicated

- The cache strategy explanation appears in three assets (architecture, ADR-0002, redirect-service). This is intentional but feels like duplication.
- The list of services is repeated in architecture.md and in the services/ directory. This is structural and unavoidable.
- The dependencies of redirect-service (Postgres, Redis, message queue) are mentioned in architecture.md and again in redirect-service.md. This is also intentional.

## What surprised me

- **The asset writing was the easiest part.** I expected it to be hard. It was not. The structure of the spec made each asset easy to start: copy the structure, fill in the project-specific content.
- **The retrospective and findings were the hardest part.** Identifying weaknesses requires stepping back from the work. The spec is good; the work is good; finding what is missing is harder than finding what is there.
- **The relationship vocabulary was sufficient 90% of the time.** I rarely wanted a relationship I could not express. The cases where I wanted more (caused_by, replaced_by) were the exception, not the rule.
- **Decisions were the most valuable asset type for capturing "why".** When I came to write the cache stampede mitigation playbook and the elevated latency troubleshooting entry, the ADR was the source of truth for why the system is shaped as it is. Without the ADRs, the playbook and troubleshooting entry would have had to re-explain the design rationale.

## Asset boundary review

For each asset, asked: "Could this information belong somewhere else? Did any asset become too large? Was the single-responsibility principle maintained?"

- **architecture.md.** Stays at the system level. Does not contain per-service detail (that is in services/). Does not contain operational procedures (those are in playbooks/). The cache strategy is mentioned at a high level, with the full story in ADR-0002. Single concern: system overview. **OK.**
- **services/api-service.md.** Stays focused on api-service. Does not include the URL shortening flow (that is in flows/). Does not include why base62 was chosen (that is in ADR-0001). Single concern: api-service as a unit. **OK.**
- **services/redirect-service.md.** Similar to api-service. The cache strategy is mentioned because it is a core responsibility of the service; the full rationale is in ADR-0002. The cache stampede mitigation is referenced but not detailed (that is in the playbook). Single concern: redirect-service. **OK, but the cache strategy mention could be trimmed to reduce overlap with ADR-0002.**
- **services/analytics-service.md.** Stays focused on analytics-service. The click event contract is referenced from redirect-service, not re-documented here. Single concern: analytics-service. **OK.**
- **flows/url-shortening.md.** Captures the write path end-to-end. Does not include the rationale for base62 (that is in ADR-0001). Does not detail the api-service internals (that is in services/api-service.md). Single concern: the URL shortening process. **OK.**
- **flows/url-redirection.md.** Same shape. References ADR-0002, the cache stampede playbook, and the elevated latency troubleshooting entry without duplicating their content. Single concern: the URL redirection process. **OK.**
- **decisions/ADR-0001-base62-short-codes.md.** Single concern: the short code generation choice. **OK.**
- **decisions/ADR-0002-read-through-cache.md.** Single concern: the cache strategy. The cross-references to the playbook and troubleshooting entry are appropriate; the consequences section references them but does not duplicate their content. **OK.**
- **playbooks/cache-stampede-mitigation.md.** Single concern: the cache stampede mitigation procedure. References the related decisions, services, and flows without re-explaining them. Includes time estimates and rollback. **OK.**
- **troubleshooting/elevated-redirect-latency.md.** Single concern: the elevated latency issue. The resolution section references the playbook rather than re-stating it. **OK.**

## Navigation evaluation

Starting from `architecture.md`, can a new engineer answer the standard questions?

- **"How is a URL shortened?"** Read `architecture.md` → see the write path → follow the reference to `flows/url-shortening.md` → see the full procedure. **Easy: 2 hops.**
- **"Which service handles redirects?"** Read `architecture.md` → see the read path mentions redirect-service → open `services/redirect-service.md` → understand its responsibilities. **Easy: 2 hops.**
- **"Why was base62 chosen for short codes?"** Read `architecture.md` → see the architecture mentions ADR-0001 → open `decisions/ADR-0001-base62-short-codes.md` → read the full rationale. **Easy: 2 hops.**
- **"Why do we use a read-through cache for redirects?"** Same pattern: architecture → ADR-0002 → read the rationale. **Easy: 2 hops.**
- **"How do I mitigate a cache stampede?"** Read `architecture.md` → see the cache mention → no direct pointer. **Have to know that cache stampede is a known issue.** Open `troubleshooting/elevated-redirect-latency.md` → it references `playbooks/cache-stampede-mitigation.md`. **3 hops, but the entry point is not obvious from architecture.md.**
- **"What happens if Postgres is down?"** Read `architecture.md` → no direct pointer. **Have to know to look at troubleshooting or playbooks.** Open `troubleshooting/elevated-redirect-latency.md` or `flows/url-redirection.md` → the failure handling section mentions it. **3-4 hops. Could be more direct.**

The architecture document serves as a strong entry point for the high-level "what" and "why" questions. The "how" and "what if" questions require some knowledge of which asset to start from. This is acceptable but could be improved with a "Common questions" section in architecture.md or a more explicit cross-reference from architecture.md to the playbooks and troubleshooting entries.

## Overall

EKS worked well for this system. The friction points are real but small. The most actionable observations are:

1. The relationship vocabulary could use a `caused_by` type.
2. The relationship vocabulary could use a `replaced_by` inverse of `supersedes`.
3. The boundary between Troubleshooting and Playbook is fuzzy for some scenarios.
4. The Glossary's overlap with other assets is intentional but should be acknowledged in the spec.
5. Architecture could cross-reference playbooks and troubleshooting entries more explicitly for common operational questions.

These are observations, not prescriptions. See `FINDINGS.md` for the candidate improvements.
