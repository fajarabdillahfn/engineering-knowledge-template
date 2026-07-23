# Glossary

Terms used in the `controlroom-inquiry` project. Domain-specific terms (Indonesian payment / vendor) and code-internal terms are both included.

## Product and Category Terms

### Product Code
A short identifier for a billable product. Examples: `PLNPRE`, `PLNPOST`, `BPJSKS`, `KAITIKET`. A product code resolves through the routing table to exactly one switch and exactly one vendor. See `decisions/ADR-0001-grpc-for-vendor-comms.md` for how routing is resolved.

### Category Code
A higher-level grouping of products. Examples: `CTAYPD` (paket data), `CTAYPL` (pulsa), `CTAYEM` (e-money), `CTAYTK` (Telkom), `CTAYBP` (BPJS Ketenagakerjaan), `CTAYLS` (Listrik / PLN). Category codes are used for product-specific validation rules (see `inquiry/service.go:validateProductSpecificRules`).

### Product Type
One of `PREPAID`, `POSTPAID`, `VOUCHER`. Determines which sub-service handles the inquiry.
- `PREPAID`: Inquiry returns an amount to be paid; no bill is owed yet (e.g., PLN token, pulsa).
- `POSTPAID`: Inquiry returns a bill to be paid (e.g., PLN Pascabayar, BPJS).
- `VOUCHER`: Not implemented in this service. The product type exists in the database schema but there is no `VOUCHER` handler.

### Switch
The routing layer between a product and a vendor. Each product has one or more switches; each switch maps to exactly one vendor. The switch abstraction exists so that vendors can be swapped without changing the product code, but in this service the mapping is effectively 1:1:1 (product → switch → vendor).

### Vendor
An Indonesian payment service provider that the platform integrates with. Each vendor exposes its own gRPC service with its own request/response schema. Examples: BNI, BRI, KAI, PLN, BPJS Ketenagakerjaan, Indosat, GoPay, ShopeePay, Traveloka, Dana, OVO. See `providers/` for the full list.

## Common Indonesian Vendor Names

- **PLN** — Perusahaan Listrik Negara. The Indonesian state electricity company. PLN products are typically `CTAYLS` (Listrik). Inquiries can be prepaid (token) or postpaid (tagihan).
- **BPJS** — Badan Penyelenggara Jaminan Sosial. Indonesia's social security agency. Two main products appear in this service: BPJS Kesehatan (`CTAYBK`) and BPJS Ketenagakerjaan (`CTAYBP`).
- **KAI** — Kereta Api Indonesia. The Indonesian railway company. KAI inquiries require an active enrollment before they can proceed.
- **PDAM** — Perusahaan Daerah Air Minum. Local water utilities. Inquiries return water bills for the customer.
- **Indosat** — Indonesian telecommunications operator. Pulsa, data packages, and postpaid mobile plans.

## Inquiry Lifecycle

### Inquiry
The act of asking a vendor "what does this customer owe / what can they buy". For prepaid, the inquiry returns the price and product details; the customer then pays to receive the product. For postpaid, the inquiry returns the outstanding bill.

### RefNumber
A reference number assigned by `api-gateway` (or higher) and passed through to this service. Used to correlate logs across services for a single user action. Surfaced as `RefNumber` in the gRPC request.

### RequestID
A unique identifier for this inquiry, generated inside `controlroom-inquiry` (or accepted from the upstream caller). Written to the inquiry log so the response can be replayed through the OpenAPI pipeline. Empty if the inquiry failed before request creation.

### InquiryID
A numeric database identifier for the inquiry record. Different from `RequestID`; the database ID is auto-increment.

## Routing and Validation

### IDPEL
"Identitas Pelanggan" — PLN customer ID. For PLN inquiries, the account number is the IDPEL. The validation rules require:
- Prepaid: 11 or 12 digits.
- Postpaid: exactly 12 digits.
- See `inquiry/service.go:validateProductSpecificRules`.

### Enrollment
For some vendors (notably KAI), the customer must be enrolled before an inquiry can be performed. The enrollment record is held in the `enrollment_registry` table. If the customer is not enrolled or the enrollment is inactive, the inquiry fails.

## Technical Terms

### gRPC
A high-performance RPC framework using Protocol Buffers for serialization. Used for both inbound (api-gateway → controlroom-inquiry) and outbound (controlroom-inquiry → vendor) communication in this service.

### Vendor Adapter
A Go package in `providers/<vendor>/` that implements the `I<Vendor>Service` interface for a single vendor. Each adapter handles the vendor-specific request transformation, gRPC call, and response parsing. Adapters are intentionally not interchangeable — each vendor has its own schema and quirks.

### Inquiry Formatter
A helper service (`formatter.InquiryFormatter`) that translates between the internal response format and the OpenAPI response format. The internal format is rich; the OpenAPI format is constrained by what the OpenAPI gateway expects. The formatter handles both successful and error response shaping.

### ValueOnlyContext
A custom `context.Context` implementation in `inquiry/handler.go` that carries values (including trace IDs) but ignores cancellation and deadlines. Used for the background update of the OpenAPI response so the audit log write completes even if the original client request was cancelled.

### Direct Connection
A boolean flag on the `InquiryV2` request indicating whether the inquiry should bypass certain caching or shortcut paths. Parsed from the request and threaded into the context as `models.DIRECTCONNECTION`. The downstream behavior of this flag is not visible in this service's code; it is set in the context for downstream consumers.

## Service Identifiers

- `inquiryService` — the orchestration layer. Owns the routing, validation, and dispatch logic.
- `inquiryHandler` — the gRPC layer. Owns request validation, response shaping, and the background update.
- `prepaidService` / `postpaidService` — the domain services. Own the vendor adapter selection for their respective product types.
- `inquiryRegistry` — the data access layer for the routing table and inquiry log.
- `enrolmentRegistry` — the data access layer for the enrollment table.
