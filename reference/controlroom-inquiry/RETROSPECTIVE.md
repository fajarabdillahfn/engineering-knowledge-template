# Retrospective

Observations from documenting the `controlroom-inquiry` service using the EKS specification. This is the second EKS dogfooding exercise; the first was a fictional URL Shortener. The goal of this retrospective is honest reporting: where EKS worked well, where it was awkward, and where the real system revealed things the fictional reference could not.

The goal is not to validate EKS by force; it is to find friction. Findings and proposed improvements are in `FINDINGS.md`.

## What was different this time

The URL Shortener was greenfield and small. `controlroom-inquiry` is a real, in-production, multi-vendor, multi-protocol service. The differences matter:

- **Real concurrency.** Multiple services, multiple repositories, real production traffic, real ops. The knowledge base has to be useful for someone on call at 3am, not just for a new hire reading on day one.
- **Cross-repository context.** `controlroom-inquiry` is one of 30+ services in the AyoPop payment platform. Some assets in this knowledge base reference services that live in other repositories. The EKS model assumes knowledge lives in one place; reality does not.
- **Domain terms are non-obvious.** "Inquiry", "PLN", "BPJS", "IDPEL", "switch", "vendor" — these are Indonesian payment industry terms that an outsider cannot infer. The glossary has to do real work.
- **Decisions are buried in code.** The URL Shortener had ADRs because the work was new and the choices were deliberate. `controlroom-inquiry` is years old; many of the original decisions are in commit messages, Slack threads, or implicit in the code. Documenting them is archaeology, not journalism.
- **No README, no docs directory.** The URL Shortener had a `README.md` to anchor the knowledge base. `controlroom-inquiry` has neither. The architecture document had to be the entry point from scratch.

## Where EKS felt natural

- **The 7 asset types covered the system well.** Architecture, Service, Flow, Decision, Glossary — all of them earned their place. I did not feel the need to invent a new asset type, just as the URL Shortener retrospective reported.
- **The Service asset worked for the cross-cutting orchestration layer.** `controlroom-inquiry` is not a "service" in the micro-service sense (it has its own database, its own process, etc.) — but it is a service in the EKS sense: a single unit with a clear responsibility. The Service asset captured that cleanly.
- **The Flow asset captured the inquiry path end-to-end.** The orchestration code (`inquiry/service.go`) has 200+ lines of branching and validation. Distilling that into a 50-line flow document was a real win for navigability. A new engineer could read the flow document and understand the system before reading any code.
- **The per-vendor gRPC architecture is exactly what EKS calls a "Decision".** It is a deliberate choice with tradeoffs. The ADR format worked. Without EKS, this choice would live in a commit message and a Slack thread.
- **The Glossary was the highest-leverage asset.** I underestimated this for the URL Shortener. For a domain with non-obvious terms (PLN, IDPEL, switch, vendor, BPJS), the Glossary is what makes the rest of the knowledge base usable. It should be one of the first assets written, not the last.
- **The cross-references between assets earned their keep.** When the agent (or a new engineer) reads the architecture document, the "Why the service is shaped this way" subsection points directly to the ADRs. When the agent reads the flow, the failure handling subsection points to the troubleshooting entries. This is the EKS use case: navigation, not reading.

## Where EKS was awkward

- **Cross-repository assets are not a first-class concept.** `controlroom-inquiry` is one piece of a larger system. The `api-gateway` (upstream) and `controlroom-payment` (sibling) are documented in their own repositories. When this knowledge base references those services, it does so by name and by repository, not by an `assets/<service>/<asset>` link. The EKS model assumes a single knowledge base; this is rarely the case in a real system. See FINDINGS F15.
- **Decisions that are not in a commit message are hard to recover.** Some of the most important choices in this service (per-vendor interfaces, gRPC for vendor comms) are implicit in the code structure. Writing an ADR for them required inferring the reasoning, not recording it. The retrospective could only document "this is what was chosen"; the ADR had to reverse-engineer "this is why". This is acceptable, but it makes the ADR a hypothesis, not a record. See FINDINGS F16.
- **The Flow asset struggles with code that is mostly branching.** The inquiry flow is 200+ lines of `if` statements, mostly validation. The flow document had to abstract this into a sequence of named steps, which is correct but loses fidelity. The flow says "the service applies product-specific rules" — the code says `if product.CategoryCode == "CTAYBP" && vendor.Code != models.VENDOR_BPJSTK { if inquiryRequest.Month < 1 || inquiryRequest.Month > 12 { ... } }`. Both are valid, but the code is the source of truth for the rules. The flow is the source of truth for the order. The relationship between them is implicit.
- **Operational concerns are mostly about vendors, but the vendors live in other processes.** The most common operational failure mode is "the vendor is down". But the vendor is not part of this service; it is a separate process in a separate repository. The troubleshooting entry for "vendor unavailable" has to point outward. EKS can express "this is a relationship to an external system" but the cross-repository aspect is a real friction point.
- **There is no place for "known weak spots in the code itself".** The `inquiry/service.go:validateProductSpecificRules` function has hardcoded category codes ("CTAYBP", "CTAYLS") and vendor codes scattered through it. This is a code smell: the rules should be data, not code. But EKS does not have a concept of "code quality concern" or "known technical debt" at the asset level. The closest is a Decision (explaining why the code is shaped this way) or a Troubleshooting entry (describing what to do when the hardcoded code causes a problem). Neither is quite right.

## What surprised me

- **The most valuable asset to write first was the Glossary, not the Architecture.** I expected the architecture document to be the foundation. In practice, the glossary was the bottleneck: until I had defined "switch", "vendor", "IDPEL", "prepaid vs postpaid", the architecture document could not name the components without defining them inline. I rewrote the architecture twice; each rewrite was blocked by a term I had not yet defined.
- **The vendor adapter file (`providers/jkpln/jkpln.go`) was the best entry point to the code.** The adapter is small, self-contained, and shows the full pattern: build vendor request, call vendor, parse vendor response. Reading one adapter was enough to understand the abstraction. The `inquiry/service.go` file is much longer and harder to navigate. This is the inverse of what I expected: the longest file is the orchestration, but the most useful file for understanding the system is a small leaf.
- **The `common/grpc_client.go` file is a hidden Decision.** The retry policy, the load balancing choice, and the insecure credentials comment are all decisions that deserve an ADR. The URL Shortener had no equivalent — there was no shared infrastructure code to document. Real systems have more shared infrastructure than greenfield examples.
- **Some of the things I would have written a Decision about (e.g., the `VOUCHER` product type that exists in the schema but is not implemented) are not really decisions.** They are gaps — features that were planned but not built. EKS does not have a "this is not implemented" construct. The closest I could come was to mention it in the Glossary ("not implemented in this service") and in the FINDINGS. But there is no first-class way to say "this exists in the schema but the code path is missing".

## Where the test results from the URL Shortener dogfooding held up

- **Per-asset hop counts are similar.** The architecture document is the natural entry point; the glossary is the secondary entry point. For a specific "how do I add a new vendor" question, the answer is in `decisions/ADR-0002-per-vendor-interface.md` plus a pointer to `providers/jkpln/` as an example. The hop count is in the 1-3 range, consistent with the URL Shortener results.
- **The F11 retrofit pattern held up.** Adding a "Common operational questions" section to the architecture document was a small change that significantly improved the discoverability of the troubleshooting entries. I would do this again by default.
- **Honest gap reporting still required.** The probe question (Q10 in the URL Shortener test — "does this system have rate limiting?") was answered honestly as "not specified" by the agent. The equivalent question for `controlroom-inquiry` is "does this service authenticate the caller?" and the answer in the architecture document is "no, the service assumes api-gateway has authenticated the caller". This is documented explicitly because the absence is a deliberate choice, not an oversight.

## What is still open

- **F5 (Policy asset type).** The deferred finding from the URL Shortener dogfooding was about whether policies (PII handling, retention, vendor SLAs) need their own asset type. `controlroom-inquiry` does have vendor-specific policies (rate limits, retry policies, business rules per product code), but they live in the code, not in assets. If F5 is to be revisited, `controlroom-inquiry` would be a strong test case. See FINDINGS F17.
- **F13 (explicit non-decisions).** The deferred finding was about how to express "we explicitly do not do X". `controlroom-inquiry` has several such cases: no authentication at this layer, no rate limiting, no circuit breaker, no VOUCHER product type implementation. I documented these in the architecture document's "Why the service is shaped this way" subsection and in the Glossary. The F13 design proposed `out_of_scope` as a metadata field; in practice, the absence is documented in prose, not as a structured field. See FINDINGS F18.
- **Cross-repository knowledge bases.** The EKS model assumes a single knowledge base. Real systems are usually a federation of knowledge bases, one per service. The cross-references work, but the model does not explicitly address federation. See FINDINGS F15.

## Overall

EKS worked for documenting `controlroom-inquiry`. The friction points are real but they are the friction of a real system, not the friction of a misapplied spec. The most actionable findings are:

1. Cross-repository assets need a first-class concept (F15).
2. Reverse-engineering ADRs from existing code requires a different methodology than writing ADRs for new work (F16).
3. The Flow asset is the weakest fit for code that is mostly branching. A complementary "Rule Index" asset might help (F19).
4. F5 (Policy) and F13 (non-decisions) should be revisited now that a real system is available (F17, F18).

These are observations, not prescriptions. See `FINDINGS.md` for the candidate improvements.
