# ADR-0003: Prepend "0" to account numbers for certain prepaid categories

## Status

Accepted (inherited from a prior change request: CR-1130).

## Context

For Indonesian mobile and digital products (paket data, pulsa, e-money, Telkom), the customer-entered account number is often missing the leading "0" that the vendor expects. For example, a customer may enter `81234567890` for a mobile number when the vendor expects `081234567890`.

Options considered:

- **Reject the inquiry with a validation error.** Force the customer to enter the leading "0". This is correct from a data-integrity perspective but creates a poor user experience and increases support burden.
- **Normalize at the edge (api-gateway or openapi-gateway).** Apply the leading "0" rule before forwarding to controlroom-inquiry. This keeps the inquiry service free of normalization logic but couples the normalization rule to the upstream API contract.
- **Normalize inside the inquiry service.** Apply the leading "0" rule in the `inquiryService.Inquiry` function for the affected category codes. This keeps the rule close to the routing logic and is the option that was chosen.

## Decision

In `inquiryService.Inquiry`, before calling the prepaid domain service, prepend "0" to the account number if:

- The product type is `PREPAID`.
- The product's category code is one of: `CTAYPD` (paket data), `CTAYPL` (pulsa), `CTAYEM` (e-money), `CTAYTK` (Telkom).
- The account number is non-empty and does not already start with "0".

The mutation is performed in place on the `inquiryRequest.AccountNumber` field. The original (customer-entered) value is not preserved.

The change originated in a specific change request (CR-1130). The reference to that change request is preserved in the inline comment in the code. The rule is hardcoded in the `inquiryService.Inquiry` function; it is not data-driven from the routing table.

## Consequences

**Positive:**

- Customers can enter mobile numbers without the leading "0", which matches typical user behavior on Indonesian mobile platforms.
- The rule is centralized in the inquiry service, so the upstream API contract is simple and the rule does not need to be replicated in every client.
- The rule is visible and reviewable in the code, in a single function with a clear comment.

**Negative:**

- The rule is hardcoded. Adding a new category that needs the same treatment requires a code change. There is no way for a product manager to add a new category without a deploy.
- The rule mutates the request in place. The original customer-entered value is not preserved, so the audit log and the response to the client show the normalized value, not the original. If the normalization is wrong, debugging is harder because the original is gone.
- The rule is category-specific. A category that is conceptually similar (e.g., another mobile operator category) but uses a different category code will not get the treatment. The rule is not extensible to new categories without a code change.
- The rule is silently applied. The customer is not told that their input was modified. If the customer's downstream system expects the original value, it will not match.

## Alternatives Considered

- **Reject with a validation error.** Rejected because it creates a poor user experience and pushes the problem to the client (or to customer support).
- **Normalize at the edge.** Rejected because it couples the normalization rule to the upstream API contract. The same rule would need to be replicated in every client (api-gateway, openapi-gateway, partner integrations).
- **Make the rule data-driven.** Considered but not adopted. The routing table already holds category-level metadata. Adding a `requires_leading_zero` boolean to the category metadata would make this rule data-driven. The change is a candidate for a future refactor (see FINDINGS F19).

## References

- `implements` the account number normalization step in `../flows/inquiry-v2.md`.
- `implemented_by` the inline logic in `../inquiry/service.go` (the `case PREPAID` block).
- `relates_to` the `VOUCHER` product type, which is recognized in the schema but not implemented (see FINDINGS F18).
