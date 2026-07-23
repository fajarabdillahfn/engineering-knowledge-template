# Findings

Concrete candidates for improving EKS, based on the `controlroom-inquiry` dogfooding. Each finding is backed by an example from `RETROSPECTIVE.md`. Findings are numbered starting at F15 to continue from the URL Shortener dogfooding (F1–F10) and the agent-readability test (F11–F14).

Findings were collected during the second dogfooding sprint. The high-priority findings address gaps in EKS that the URL Shortener could not surface. The medium- and low-priority findings are observations that may inform future iterations.

## Summary

| ID | Category | Title | Priority | Status |
| --- | --- | --- | --- | --- |
| F15 | Federation | Cross-repository assets need a first-class concept | High | Open |
| F16 | Methodology | Reverse-engineering ADRs from existing code requires a different methodology | High | Open |
| F17 | Asset types | F5 (Policy) — `controlroom-inquiry` is the right test case | Medium | Open |
| F18 | Architecture | No circuit breaker, no rate limit, no auth — explicit non-decisions | Medium | Open |
| F19 | Asset types | A "Rule Index" asset for code that is mostly branching | Low | Open |
| F20 | Data model | Routing table is the real source of truth — needs a first-class concept | Low | Open |

---

## F15: Cross-repository assets need a first-class concept

**Category.** Federation.

**Problem.** `controlroom-inquiry` is one of 30+ services in the AyoPop payment platform. Its knowledge base references services that live in other repositories (`api-gateway`, `controlroom-payment`, the vendor gRPC services). EKS does not have a first-class concept for cross-repository assets. The current workaround is to reference services by name and by repository, which is imprecise and unverified.

**Example.** The architecture document of `controlroom-inquiry` references `api-gateway` and `controlroom-payment`. These are sibling services with their own EKS knowledge bases (in their own repositories). When a new engineer reads the `controlroom-inquiry` architecture, they see "upstream api-gateway service" and "parallel controlroom-payment service" but cannot navigate to those services' knowledge bases. They have to know the repository URLs.

**Proposed addition.** Add a "Federation" concept to EKS. An asset MAY reference assets in other knowledge bases by including a structured cross-reference. The cross-reference includes the repository, the asset path, and the relationship (e.g., `upstream`, `sibling`, `downstream`). An implementation MAY resolve these cross-references by reading the referenced knowledge base. The spec does not require a registry or a global index; it just requires the cross-reference to be explicit.

**Priority.** High. Cross-repository assets are the norm in any non-trivial system, not the exception. Without a federation concept, EKS cannot describe the most common kind of system accurately.

**Resolution.** Open. Requires a spec change to `KNOWLEDGE_MODEL.md` and possibly `SPECIFICATION.md`. The change is small (a new section on federation) but the implications for tooling are larger (an EKS implementation needs to know how to follow cross-references).

---

## F16: Reverse-engineering ADRs from existing code requires a different methodology

**Category.** Methodology.

**Problem.** The URL Shortener dogfooding wrote ADRs as part of the work, because the work was new and the choices were deliberate. `controlroom-inquiry` is years old; many of the most important choices (per-vendor interfaces, gRPC for vendor comms) are implicit in the code structure. Writing ADRs for these choices required inferring the reasoning, not recording it.

**Example.** `decisions/ADR-0002-per-vendor-interface.md` documents the choice to use per-vendor Go interfaces. The reasoning ("vendor quirks are explicit in the type signature; adding a new vendor is a bounded, well-defined task") is a hypothesis, not a record. The original developer may have had different reasons, or may not have had a conscious reason at all. The ADR is a guess dressed as a fact.

**Proposed addition.** Add a methodology note to `HANDBOOK.md` (or a new `METHODOLOGY.md`) about reverse-engineering ADRs. The note should cover:

- How to distinguish a "deliberate choice" from an "emergent pattern" in the code.
- How to mark an ADR as "inferred" vs "recorded" (e.g., a "Confidence" field with values like "recorded from commit message", "inferred from code", "unverified").
- When to write an inferred ADR at all (when the choice has ongoing impact and is non-obvious) vs when to skip it (when the choice is local and self-documenting).

**Priority.** High. Without this methodology, every EKS dogfooding on a real system will produce ADRs that mix fact and inference without acknowledging the difference.

**Resolution.** Open. Requires a new section in `HANDBOOK.md` (or a new methodology document). The change is purely additive; it does not affect the conceptual model.

---

## F17: F5 (Policy asset type) — `controlroom-inquiry` is the right test case

**Category.** Asset types.

**Problem.** F5 from the URL Shortener dogfooding was deferred because the URL Shortener was too clean to surface policies. `controlroom-inquiry` has multiple policies:

- Vendor rate limits (each vendor has different throughput limits; some vendors enforce strict per-second caps).
- Retry policies (currently uniform; should be per-vendor).
- Vendor-specific business rules (BPJS month, PLN IDPEL length, KAI enrollment) — these are in the code as `if` statements.
- PII handling (the account number is PII; the customer name is PII).
- Vendor contract terms (SLA, payment terms, etc.) — not currently in the code, but exist in vendor contracts.

**Example.** The account number prefix logic (`inquiry/service.go:case PREPAID`) is a policy, not a code-level decision. It encodes a business rule that should be data-driven. The F13 (out-of-scope) construct is not quite right for it because the rule exists and is implemented; it just should be in a more reviewable place.

**Proposed re-evaluation.** Revisit F5. The candidate asset types are:

- **Policy** — a documented business rule that affects how the system behaves.
- **Rule** — a code-level condition that encodes a policy (current state: hardcoded in `if` statements).
- **Convention** — already partially exists in `knowledge/conventions.md` of the EKS template, but it is not a first-class asset type.

The minimum useful change might be: add "Policy" as a first-class asset type, and migrate the vendor-specific business rules (BPJS month, PLN IDPEL) into Policy assets. The code would then read the rule from the routing table or from a Policy asset instead of hardcoding it.

**Priority.** Medium. The current state works; the change would be a quality-of-life improvement for code reviewers and for new engineers.

**Resolution.** Open. Requires deciding on the asset type (Policy vs Rule vs Convention) and adding it to `KNOWLEDGE_MODEL.md`.

---

## F18: No circuit breaker, no rate limit, no auth — explicit non-decisions

**Category.** Architecture (and the F13 "out-of-scope" construct).

**Problem.** `controlroom-inquiry` has several things it explicitly does NOT do:

- **No authentication.** The service assumes `api-gateway` has authenticated the caller. There is no auth check in `inquiry/handler.go`.
- **No rate limiting.** The service does not enforce per-caller or per-product rate limits. The caller is expected to do this.
- **No circuit breaker.** The gRPC client retries 3 times on `UNAVAILABLE` but does not back off after repeated failures. A slow vendor can be hammered with retries from many concurrent callers.
- **No caching of the routing table.** The service queries Postgres on every inquiry for the routing context.
- **`VOUCHER` product type is not implemented.** The product type exists in the database schema and the switch statement recognizes it, but the handler returns an error.

These are all explicit non-decisions. They are documented in the architecture document and the glossary, but the documentation is prose, not a structured field. The F13 design proposed `out_of_scope` as a metadata field for this. The `controlroom-inquiry` dogfooding surfaced 5 such cases in a single service.

**Proposed addition.** Adopt the F13 metadata design (from `METADATA_DESIGN.md`) and apply it to the `controlroom-inquiry` assets. The 5 cases above would each be a value in the `out_of_scope` field. The metadata would make these non-decisions searchable and reviewable in a way that prose cannot.

**Priority.** Medium. The current state works (prose documentation is honest). The change would be a quality-of-life improvement for code reviewers and for new engineers trying to understand "why is this not implemented?"

**Resolution.** Open. Requires adopting the metadata schema (Task 2 of the original plan) and applying it to the assets.

---

## F19: A "Rule Index" asset for code that is mostly branching

**Category.** Asset types.

**Problem.** The inquiry flow is 200+ lines of `if` statements, mostly validation. The flow document had to abstract this into a sequence of named steps, which is correct but loses fidelity. There is no asset that captures the "rules" in a structured, reviewable way.

**Example.** `inquiry/service.go:validateProductSpecificRules` contains:

- `if product.CategoryCode == "CTAYBP" && vendor.Code != models.VENDOR_BPJSTK { ... }` — BPJS requires a month parameter.
- `if product.CategoryCode == "CTAYLS" && (vendor.Code == models.VENDOR_JL || vendor.Code == models.VENDOR_JLB) { ... }` — PLN IDPEL length depends on prepaid vs postpaid.

These rules are scattered through the code. A new engineer reading the code has to find them by searching for category codes. A "Rule Index" asset would list all the rules in one place, with their source (file and line) and their scope (which product/vendor combinations they apply to).

**Proposed addition.** Add a "Rule Index" asset type. The asset lists the rules that govern the system's behavior, each with:

- The rule statement (what is checked).
- The source (file and line, or asset reference if data-driven).
- The scope (product, vendor, or environment).
- The error code returned when the rule fails.

**Priority.** Low. The current state works (the rules are in the code, where they are executed). The change would be a quality-of-life improvement for code reviewers and for new engineers.

**Resolution.** Open. Requires deciding on the asset type and its relationship to existing types (Policy, Decision, Service).

---

## F20: Routing table is the real source of truth — needs a first-class concept

**Category.** Data model.

**Problem.** The routing table in Postgres (`unified_inquiry_info` or equivalent) is the single source of truth for which product maps to which switch and which vendor. It is queried on every inquiry. It is the most important data structure in the service, but EKS has no concept for it.

**Example.** `inquiry/service.go:inquiryRegistry.GetUnifiedInquiryInfo` is the only call that matters for routing. The result (`UnifiedInquiryInfo`) holds the product, the switch, and the vendor. Changes to this table (adding a product, deactivating a vendor, changing a switch mapping) have immediate and global impact on the service. But the routing table is not an asset; it is just a database table.

**Proposed addition.** Add a "Data Asset" or "Configuration Source" concept to EKS. The concept would describe a data structure that the system depends on, with its source (Postgres table, config file, environment variable), its update mechanism (manual, automated, scheduled), and its consumers (which services read it). The routing table would be the canonical example.

**Priority.** Low. The current state works (the routing table is queryable in Postgres). The change would make the routing table reviewable as part of the knowledge base, not just as a database schema.

**Resolution.** Open. Requires deciding on the asset type and its scope. This is the most abstract of the findings; the others are more concrete.

---

## How to use this document

Each finding has:
- A category (Federation, Methodology, Asset types, etc.).
- A problem statement with an example from `controlroom-inquiry`.
- A proposed addition or clarification.
- A priority.

Findings F15 and F16 are the most actionable. F17 and F18 are the most likely to be adopted. F19 and F20 are the most speculative and may turn out to be wrong.

Findings are not commitments. The spec should not be modified without discussion. Use this document to drive that discussion.
