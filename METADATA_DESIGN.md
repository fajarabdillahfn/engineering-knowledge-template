# Asset Metadata Design

## Metadata

- **Date:** 2026-07-23
- **Status:** Design proposal. Not yet adopted into `KNOWLEDGE_MODEL.md`.
- **Scope:** Defines the conceptual metadata fields an EKS implementation MUST or SHOULD support. Does NOT define serialization format.
- **Driver:** `reference/AGENT_READABILITY_TEST.md` — the test surfaced gaps that current EKS concepts do not address (specifically: freshness, scope, and explicit non-decisions).
- **Relationship to spec:** Proposes additions to `KNOWLEDGE_MODEL.md` sections 6–8. Three of the five fields below are NEW concepts; two formalize existing concepts.

## Context

`KNOWLEDGE_MODEL.md` §1.1 explicitly defers a "specific metadata schema":

> EKM does not define:
> - A specific storage format ...
> - **A specific metadata schema.**
> - A specific validation tool or workflow automation.

The 2026-07-22 agent-readability test on the URL shortener reference showed this deferral has a cost. Three categories of agent behavior were unanswerable from current EKS concepts:

1. **"Is this asset still trustworthy?"** — no freshness concept. The agent's Q8 caveats ("why 1 hour specifically is not in the docs") and Q10 ("rate limiting not specified") were the agent's honest way of saying "I can't tell if this is current."
2. **"What part of the system does this describe?"** — no scope concept. The agent had to infer from the asset's body.
3. **"What was explicitly NOT done?"** — no construct for non-decisions. F13.

This design proposes the **minimum set of conceptual fields** that close these gaps without committing to a serialization format.

## Goals

1. Enable an AI agent to assess, at a glance, whether an asset is current and trustworthy.
2. Enable an agent to route questions to the right asset by knowing its scope.
3. Address F13: express "this was explicitly not done" without inventing a new asset type.
4. Stay **conceptual only** — the spec describes fields and semantics; implementations choose how to represent them.

## Non-goals

1. **No serialization format.** YAML frontmatter, JSON, sidecar files, graph databases, MCP tool calls — all are valid implementation choices. The spec describes fields, not their representation.
2. **No tooling requirements.** No validators, no CLIs, no CI checks. These may come later, informed by usage.
3. **No new asset types.** F5 (Policy) and F13 (Non-Decision) remain separate questions. This design uses metadata to express what F13 needs without adding a new asset type.
4. **No mandatory content templates.** Metadata fields are about the asset, not the content within it. A 50-line asset and a 500-line asset both need the same metadata.

## The proposed schema — 5 fields

### Summary table

| Field | Required? | Type | Purpose |
| --- | --- | --- | --- |
| `status` | Required | Enum (5 values) | Asset's maturity and current relevance. |
| `last_validated` | Required | Date | When this asset was last confirmed to reflect the system accurately. |
| `owner` | Required | String | Team or individual accountable for keeping this asset current. |
| `scope` | Required | String | What part of the system this asset describes. |
| `out_of_scope` | Optional | List of strings | What was explicitly considered and excluded from this asset or system. |

### Field 1: `status` (Required)

**Already in spec:** `KNOWLEDGE_MODEL.md` §6 — Draft, Review, Accepted, Deprecated, Archived.

**Promote to first-class metadata field.** The spec defines the lifecycle states conceptually; this design requires every asset to declare which state it is in.

**Semantics.** "Is this asset considered a source of truth right now?"

**Per-type conventions.** All asset types use the same five states. The transition rules are already in §6.

**Example (URL shortener).**

| Asset | Status |
| --- | --- |
| `architecture.md` | Accepted |
| `decisions/ADR-0001-base62-short-codes.md` | Accepted |
| `decisions/ADR-0002-read-through-cache.md` | Accepted (would become Deprecated if a v3 ADR exists) |
| A new `decisions/ADR-0003-foo.md` being drafted | Draft |

### Field 2: `last_validated` (Required)

**NEW concept. Not in `KNOWLEDGE_MODEL.md` yet.**

**Semantics.** "When was this asset last confirmed to accurately reflect the system?"

This is distinct from `status`:
- `status` says *whether* the asset is considered current.
- `last_validated` says *how recently* someone checked.

A 5-year-old `Accepted` asset might still be marked Accepted, but its `last_validated` field tells the agent "no one has verified this in 5 years" — which is very different information.

**Per-type conventions.** Same semantics for all types. May be the same date as the asset's creation if it was reviewed on creation. The spec does NOT prescribe a date format; implementations may use ISO 8601, RFC 3339, "YYYY-MM-DD", or any unambiguous date representation.

**Optional companion.** An implementation MAY also record *who* performed the validation. The spec leaves this to implementations — a "validated_by" field is reasonable but not required.

**Example (URL shortener).**

| Asset | Last validated |
| --- | --- |
| `architecture.md` | 2026-07-15 |
| `decisions/ADR-0001-base62-short-codes.md` | 2026-06-01 |
| `troubleshooting/elevated-redirect-latency.md` | 2026-05-20 |

### Field 3: `owner` (Required)

**Already in spec:** `KNOWLEDGE_MODEL.md` §7.

**Promote to first-class metadata field.** The spec defines ownership conceptually; this design requires every asset to declare an owner (or explicitly state that it has none).

**Semantics.** "Who is accountable for keeping this asset current?"

**Values:**
- A team name (e.g., "platform-team")
- An individual (e.g., "alice@example.com")
- `unowned` (explicitly no owner; SHOULD be reviewed for deprecation per §7)

**Per-type conventions.** Same for all types. A Service asset typically has one owner (the team that runs the service). A Decision asset might have a different owner from a Service asset (e.g., the team that made the decision vs. the team that operates the system).

**Example (URL shortener).**

| Asset | Owner |
| --- | --- |
| `services/api-service.md` | platform-team |
| `services/redirect-service.md` | platform-team |
| `services/analytics-service.md` | data-team |
| `decisions/ADR-0001-base62-short-codes.md` | platform-team |
| `troubleshooting/elevated-redirect-latency.md` | platform-team |

### Field 4: `scope` (Required)

**NEW concept. Not in `KNOWLEDGE_MODEL.md` yet.**

**Semantics.** "What part of the system does this asset describe?"

This complements the `applies_to` relationship (which is a *relationship* between assets, e.g., "this playbook applies to redirect-service"). `scope` is an *attribute* of the asset itself: "this asset is about X."

**Per-type conventions.** Each asset type defines a scope convention:

| Asset type | Scope convention |
| --- | --- |
| Architecture | `system` (always) |
| Service | `service:<service-name>` (e.g., `service:api-service`) |
| Flow | `flow:<flow-name>` (e.g., `flow:url-shortening`) or `flow:cross-service` for multi-service flows |
| Decision | `decision:<area>` (e.g., `decision:caching`, `decision:short-code-generation`) |
| Playbook | `service:<service-name>` of the primary service, or `flow:<flow-name>` if cross-service |
| Troubleshooting | `service:<service-name>` of the affected service, or `flow:<flow-name>` if cross-service |
| Glossary | `system` (always) |

The spec recommends these conventions but does not require them. An implementation MAY use different scope formats as long as the semantics are preserved (an asset's scope identifies the part of the system it describes).

**Example (URL shortener).**

| Asset | Scope |
| --- | --- |
| `architecture.md` | `system` |
| `services/api-service.md` | `service:api-service` |
| `flows/url-shortening.md` | `flow:url-shortening` (contained in api-service, but documented as a flow) |
| `flows/url-redirection.md` | `flow:url-redirection` (cross-service: redirect-service + analytics-service) |
| `decisions/ADR-0002-read-through-cache.md` | `decision:caching` |
| `troubleshooting/elevated-redirect-latency.md` | `service:redirect-service` |

### Field 5: `out_of_scope` (Optional)

**NEW concept. Addresses F13.**

**Semantics.** "What was explicitly considered and excluded from this asset or system?"

This is the metadata construct for explicit non-decisions. It enables the agent to answer "is rate limiting in this system?" with "no, and it's explicitly out of scope" — not just "no, and I can't tell if that's deliberate."

**Per-type conventions.** All asset types MAY use this. Most common uses:
- For Service assets: list features or concerns that the service does NOT handle (and why).
- For Architecture assets: list cross-cutting concerns that the system explicitly does not address.
- For Decision assets: list alternatives that were considered and rejected (the ADR's "Alternatives considered" section captures this in the body; `out_of_scope` is the metadata-level equivalent).

**Values.** A list of strings. Each string describes one thing that is explicitly out of scope. The spec does NOT prescribe a string format, but recommends each entry be human-readable: "Rate limiting on POST /shorten" rather than "rate-limit".

**Example (URL shortener).**

For `architecture.md`:

- "Multi-region deployment (single-region only; failover is manual)"
- "Custom short-code prefixes (e.g., /brand/...) — not supported by design"
- "End-user analytics dashboards (only operational dashboards are exposed)"

For `services/api-service.md`:

- "Rate limiting on POST /shorten (not implemented; relies on client-side API key discipline)"
- "URL content scanning or blocklist beyond the static blocklist (no malware/phishtank integration)"

For `decisions/ADR-0001-base62-short-codes.md`:

- "Auto-incrementing codes (rejected for usage-disclosure reasons; see Context)"
- "Hash-based codes (rejected for collision frequency; see Context)"

## What this means for `KNOWLEDGE_MODEL.md`

If accepted, this design proposes the following changes:

1. **Modify §1.1** — remove the "EKM does not define a specific metadata schema" line, or replace it with "EKM defines the conceptual metadata fields; the representation is left to implementations." (The latter is more accurate post-change.)

2. **Modify §3** — update the sentence "Each asset has a type, a lifecycle state, an owner, and a set of relationships to other assets" to: "Each asset has a type, a lifecycle state, an owner, a last-validated date, a scope, and a set of relationships to other assets. An asset MAY also declare items that are out of scope."

3. **Modify §6** — add a note that `status` is a metadata field that implementations MUST support.

4. **Modify §7** — add a note that `owner` is a metadata field that implementations MUST support, with explicit `unowned` value when no owner exists.

5. **Add §8.5: Freshness** — define `last_validated` as a new concept.

6. **Add §8.6: Scope** — define `scope` as a new concept with per-type conventions.

7. **Add §8.7: Out of Scope** — define `out_of_scope` as a new concept addressing F13.

These changes do NOT introduce any serialization format. The spec remains implementation-independent.

## Open questions

These were considered but not resolved in this design. They should be revisited after the second dogfooding (`controlroom-inquiry`).

1. **Should `scope` be an enum (limited to the per-type conventions) or a free-form string?** Free-form is more flexible; enum is more verifiable. The current design uses free-form with recommended conventions.

2. **Should `out_of_scope` be allowed to reference specific assets (e.g., "we explicitly do not use Decision #5")?** Currently designed as plain strings. Could be modeled as references to Decision assets with a "Rejected" status. Not enough evidence yet.

3. **Is there a need for a `supersedes_metadata` field, or is `status: Deprecated` + the existing `replaced_by` relationship enough?** Currently: enough. F2 already added `replaced_by`.

4. **How does `last_validated` interact with automation?** In a system where assets can be auto-validated (e.g., a CI check that confirms the API contract still matches the doc), should the field be auto-updated? Spec says nothing; this is an implementation concern.

## Why these five fields (and not more)

The test surfaced three concrete agent behaviors that EKS concepts did not support: trust assessment, routing by scope, and explicit non-decisions. The five fields above cover all three.

Considered and rejected for this minimal design:

- **`created_date` and `last_modified`** — useful for audit trails but not for agent navigation. The agent does not navigate by recency of edits; it navigates by *validation recency* (`last_validated`).
- **`tags` or `keywords`** — useful for search but redundant with the relationship graph. The cross-references already group related assets.
- **`confidentiality` or `visibility`** — not surfaced by the test. Most engineering knowledge is internal by default; if a system needs access control, it is a separate concern, not a metadata field.
- **`author`** — overlaps with `owner`. A document may have been authored by Alice but is now owned by Bob (e.g., after a team transfer). `owner` captures the current accountability; `author` adds noise.

If a future dogfooding surfaces another need, add a field. Until then, five is the minimum that works.

## Next steps

1. **Review this design.** If accepted, propose the changes to `KNOWLEDGE_MODEL.md` listed in the "What this means for `KNOWLEDGE_MODEL.md`" section above.
2. **Re-run the agent-readability test with metadata applied.** The URL shortener reference should add these 5 fields to each of its 12 system assets, then re-run the 10 questions to see if the agent's gap-handling improves (especially Q8 and Q10).
3. **Then proceed to Task 3 (controlroom-inquiry dogfooding).** Real systems will exercise these fields more thoroughly. The controlroom-inquiry repo will likely surface:
   - Whether `scope` per-type conventions are sufficient or need refinement.
   - Whether `out_of_scope` is used heavily enough to justify it as a first-class field.
   - Whether `last_validated` becomes the dominant trust signal or if it gets ignored.

If the design holds up under the second dogfooding, it gets promoted into the spec. If it breaks, the test data tells us why.

## How to use this document

- **For reviewers:** the "The proposed schema" section is the actual proposal. Everything else is rationale.
- **For implementation:** the "Per-type conventions" subsections give concrete examples; the spec change list at "What this means for `KNOWLEDGE_MODEL.md`" is the path to adoption.
- **For the next dogfooding:** "Next steps" item 3 lists the specific questions `controlroom-inquiry` should answer.
