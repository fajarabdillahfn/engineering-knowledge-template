---
status: Accepted
last_validated: 2026-07-23
owner: platform-team
scope: decision:vendor-interface
out_of_scope:
  - Generic vendor abstraction (rejected in this ADR)
  - Per-vendor field sharing (each adapter owns its own request/response types)
  - Runtime vendor selection (selection is deterministic per product code)
  - Compile-time shape guarantee across vendors (enforced by convention and tests, not by the type system)
---

<!--
NOTE: This ADR was reverse-engineered from existing code, not recorded from a prior decision.
See FINDINGS.md F16. The "rationale" sections are the doc-writer's hypothesis about why the
code is shaped this way, not a verbatim record. A confidence field for ADRs is a future
enhancement per the F16 proposal in HANDBOOK.md.
-->

# ADR-0002: Per-vendor interface instead of a generic vendor abstraction

## Status

Accepted.

## Context

The service integrates with 30+ vendors, each with its own request/response schema. The question is how to express the vendor abstraction in Go: a single generic interface that takes a vendor-agnostic request and returns a vendor-agnostic response, or a per-vendor interface that is shaped to the vendor's schema.

Options considered:

- **Generic interface with `map[string]interface{}` or a single canonical struct.** Each vendor adapter would implement the same Go interface. Requests would be passed as a generic structure; responses would be parsed from a generic structure into the canonical shape. The advantage is that the orchestration code (`inquiryService.Inquiry`) does not need to know which vendor is being called; it can iterate over a slice of vendor implementations.
- **Per-vendor interface with a vendor-specific request and response type.** Each vendor defines its own Go interface (e.g., `IJKPLNService`) with vendor-specific types (e.g., `JKPLNInquiryRequest`, `JKPLNResponse`). The orchestration code calls the appropriate vendor's `DoInquiry` method, and the vendor adapter handles the type conversion to and from the internal `models.InquiryResponse` shape. The advantage is that the vendor's quirks are explicit in the type signature; the disadvantage is that the orchestration code must know which vendor to call.

## Decision

Use per-vendor interfaces. Each vendor in `providers/<vendor>/` defines its own `I<Vendor>Service` interface with a `DoInquiry(ctx, ...) (InquiryResponse, error)` method. The vendor-specific request and response types are generated from the vendor's `.proto` file and live alongside the interface. The orchestration code in `inquiryService.Inquiry` and the domain services (`prepaidService`, `postpaidService`) explicitly select the vendor based on the routing context and call the vendor's `DoInquiry` method.

The current set of vendor interfaces follows a common shape:

```go
type I<Vendor>Service interface {
    DoInquiry(
        ctx context.Context,
        inquiryRequest *models.InquiryRequest,
        product *models.Product,
        switchInfo *models.SwitchInfo,
        vendor *models.Vendor,
        inquiryID uint64,
    ) (models.InquiryResponse, error)
}
```

The vendor adapter returns the internal `models.InquiryResponse` shape directly. The vendor's response is parsed inside the adapter.

## Consequences

**Positive:**

- The vendor's quirks are explicit in the type signature. A vendor that requires an extra field, a different field name, or a non-standard encoding is handled in its own adapter, with no risk of accidentally affecting other vendors.
- Adding a new vendor is a copy-paste-modify exercise: copy an existing vendor's directory, change the proto, regenerate, update the implementation. The shape of the change is well-understood.
- The orchestration code is straightforward: a switch on vendor code, a call to the right adapter's `DoInquiry`. No reflection, no type assertions, no map wrangling.
- Each vendor's `.proto` file documents the wire contract independently of the Go code. The contract is reviewable by non-Go reviewers.

**Negative:**

- There is significant duplication. 30+ vendor directories, each with a similar shape. Adding a new field to the request that all vendors should support (e.g., a new trace ID) requires updating all 30+ vendor adapters.
- The orchestration code has a switch on vendor code. Adding a new vendor requires adding a case to the switch. This is a known maintainability cost.
- The per-vendor interfaces are not interchangeable. A test that wants to mock the vendor layer for a specific vendor must implement that vendor's specific interface, not a generic one.
- There is no compile-time guarantee that all vendor adapters implement the same shape. The shape is enforced by convention and by tests, not by the type system.

## Alternatives Considered

- **Generic interface with `map[string]interface{}`.** Rejected because it would lose type safety on the wire and force every vendor adapter to do unsafe type assertions or runtime conversions. The maintenance cost of debugging generic-vendor-typed bugs is higher than the cost of the duplication.
- **Generic interface with a canonical vendor request/response struct.** Rejected because the canonical struct would have to be a superset of every vendor's fields, which would either be too permissive (allowing invalid requests) or too restrictive (excluding legitimate vendor-specific fields). Either way, the abstraction would leak.
- **Single vendor interface implemented by all 30+ adapters, with each adapter's request/response types being the same.** Rejected because the vendors' actual APIs differ in their request and response shapes. A single shared request/response type would either be too generic or too restrictive.

## References

- `implements` the vendor dispatch step in `../flows/inquiry-v2.md`.
- `implemented_by` every vendor adapter in `../providers/`.
- `references` `../decisions/ADR-0001-grpc-for-vendor-comms.md` (the gRPC choice that this decision builds on).
- `relates_to` `../troubleshooting/vendor-unavailable.md` (operational response to a vendor failure, which is detected at the vendor adapter level).
