---
status: Accepted
last_validated: 2026-07-23
owner: platform-team
scope: system
out_of_scope:
  - Authentication (handled by api-gateway, not this service)
  - Rate limiting at service layer (caller's responsibility)
  - Built-in circuit breaker (planned at switch level, not this service)
  - Caching of the routing table (Postgres queried on every request)
  - VOUCHER product type implementation (schema exists, not implemented)
  - Per-vendor retry policy customization (uniform policy across all vendors)
  - Direct client exposure (api-gateway is the only caller)
---

# Architecture

The `controlroom-inquiry` service is the bill-inquiry component of AyoPop's payment platform. It receives inquiry requests from `api-gateway`, resolves the appropriate vendor, calls the vendor over gRPC, and returns the bill details (customer name, amount, validity, product info) to the caller.

This document covers the architecture of `controlroom-inquiry` itself. For the surrounding payment system (api-gateway, controlroom-payment, callback handling, scheduler), see the architecture documents in those respective repositories.

## Components

The service is a single Go binary exposing a single gRPC service. Internally it is organized by concern: handler (gRPC), service (business orchestration), vendor adapters (per-vendor gRPC clients), repository (database access), and config (per-vendor and per-environment settings).

### gRPC Service

Exposes a single RPC, `InquiryV2(InquiryV2Request) returns (InquiryV2Response)`. The contract is defined in `grpc/action.proto`. The response uses a key-value pair pattern (`CustomerDetails[]`, `BillDetails[]`, `ProductDetails[]`, `ExtraFields[]`) so that the same shape can carry vendor-specific data without proto changes.

### Data Stores

- **Postgres** (read-only from this service). Holds the unified product-switch-vendor mapping. The `inquiry_registry.GetUnifiedInquiryInfo(ProductCode)` call resolves a product code to the full routing context: product, switch, and vendor. Also holds the inquiry log (`UpdateOpenAPIResponse` writes back the final response for audit and OpenAPI replay).
- **Redis** (read-only from this service). Not used directly. Vendor services may use Redis internally; this service does not.

### Vendor Mesh

The service integrates with 30+ Indonesian payment vendors. Each vendor is implemented as a **separate gRPC service** running in its own process. The `providers/<vendor>/` directory contains:

- `<vendor>.go` — the controlroom-inquiry-side adapter. Implements `I<Vendor>Service` and exposes `DoInquiry`.
- `interface.go` — the `I<Vendor>Service` interface.
- `action.proto` — the vendor's gRPC contract. Each vendor has its own because the request/response shapes differ.
- `action.pb.go`, `action_grpc.pb.go` — generated code.

Vendors are called over gRPC using the connection factory in `common/grpc_client.go`. The factory applies a uniform resilience policy: round-robin load balancing, three retries on `UNAVAILABLE`, exponential backoff (0.1s → 1s, multiplier 2.0), and a unary logging interceptor that records peer address, method, and duration for every call.

The gRPC channel uses insecure credentials. Authentication between `controlroom-inquiry` and vendor services is handled at the network layer (mTLS or service mesh, depending on the deployment environment) rather than in code.

## Data Flow

The end-to-end inquiry flow:

**Inbound.** `api-gateway` calls `InquiryV2` on this service with a `ProductCode` and an `AccountNumber`. The handler validates the request (parses `ClientID`, parses `DirectConnection` flag) and constructs an internal `models.InquiryRequest`.

**Routing.** `inquiryService.Inquiry` calls `inquiryRegistry.GetUnifiedInquiryInfo(ProductCode)` to look up the product, switch, and vendor in a single database round-trip. If the product is inactive, the category is unmapped, the switch is missing, or the vendor is inactive, the request fails fast with a specific response code.

**Validation.** The service applies product-specific rules. `CTAYBP` (BPJS Ketenagakerjaan) requires a valid `Month` (1-12). `CTAYLS` (Listrik / PLN) requires the account number to be 11-12 digits for prepaid, exactly 12 for postpaid. `KAI` requires an active enrollment before the inquiry is allowed.

**Dispatch.** Based on the product type (`PREPAID`, `POSTPAID`), the service calls `prepaidService.CreatePrepaidInquiry` or `postpaidService.CreatePostpaidInquiry`. These services own the vendor adapter selection and the request transformation.

**Vendor call.** The vendor adapter (`providers/<vendor>/<vendor>.go`) builds a vendor-specific gRPC request from the unified info and calls the vendor's gRPC service. The response is parsed back into the internal `models.InquiryResponse` shape.

**Response shaping.** The service marshals the internal response into JSON, runs it through `inquiryFormatter.Response` to produce the OpenAPI-format response, and maps it back to the gRPC proto. If the request had a non-empty `RequestID`, the OpenAPI response is written back to the inquiry log in a background goroutine. The background context (`ValueOnlyContext`) carries the request's values (including trace IDs) but ignores the request's cancellation and deadline, so the update completes even if the original request was cancelled.

The detailed flow is in `flows/inquiry-v2.md`.

## Boundaries

- The service is low-latency and synchronous. p99 target: under 200ms. Vendor calls dominate the latency budget; a single retry can double the wall-clock time.
- The service is stateless horizontally. Multiple instances run behind a load balancer.
- The service is read-heavy against Postgres (one query per inquiry for routing) and write-light (one background write per successful inquiry for the OpenAPI replay log).
- Vendor selection is **deterministic per product**: each product code maps to exactly one switch and exactly one vendor. There is no runtime failover between vendors at the inquiry stage. If the assigned vendor is down, the inquiry fails; failover (when implemented) will happen at the switch level, not in this service.

## Operational Concerns

For operational questions — what to do when something breaks, why the service is shaped the way it is — start here before reading individual assets.

### Vendor failures

- **Vendor gRPC returns `UNAVAILABLE`.** Retried up to 3 times by the gRPC client. If still unavailable, the inquiry fails with `models.InquiryFail`. The vendor is presumed to be down. See `troubleshooting/vendor-unavailable.md`.
- **Vendor returns an error response code.** No retry; the response is propagated to the client. Specific codes (e.g., "customer not found", "wrong IDPEL") are mapped to specific response codes in the vendor adapter.
- **Vendor is marked inactive in the routing table.** The service returns `models.InquiryVendorInactive` without calling the vendor. No retry; this is a configuration issue, not a runtime failure.

### Latency or error spikes

- **High p99 on `InquiryV2`.** Most likely a slow vendor. The unary logging interceptor in `common/grpc_client.go` records per-call duration; correlate with the vendor's `peer` field to identify the slow vendor. There is no built-in circuit breaker; the gRPC retry policy will amplify load on a slow vendor.
- **High error rate on a specific product code.** Most likely a vendor-side issue. Check the vendor's status page or contact the vendor's ops team.

### Data store failure

- **Postgres unavailable.** The service returns `models.InquiryFail` because the routing lookup fails. The client (api-gateway) retries with backoff. No automated fallback; the service has no cached routing table.

### Why the service is shaped this way

- **Why gRPC for vendor communication?** See `decisions/ADR-0001-grpc-for-vendor-comms.md`.
- **Why a per-vendor interface instead of a generic vendor abstraction?** See `decisions/ADR-0002-per-vendor-interface.md`.
- **Why prepend "0" to account numbers for certain prepaid categories?** See `decisions/ADR-0003-account-number-prefix.md`.
- **Why does the background update ignore cancellation?** The `ValueOnlyContext` pattern is used to ensure audit log writes complete even if the client disconnects. The trade-off is documented inline in `inquiry/handler.go`.

## Relationships

- This asset is referenced by every other knowledge asset in the project.
- `references` the inbound API contract in `grpc/action.proto`.
- `references` the gRPC client policy in `common/grpc_client.go`.
- `references` `services/controlroom-inquiry.md` for the service-level detail.
- `references` `flows/inquiry-v2.md` for the end-to-end flow.
- `references` `decisions/ADR-0001-grpc-for-vendor-comms.md` for the vendor comms choice.
- `references` `decisions/ADR-0002-per-vendor-interface.md` for the vendor interface choice.
- `references` `troubleshooting/vendor-unavailable.md` for the most common operational failure mode.
- `relates_to` the upstream `api-gateway` service (in the `api-gateway` repository) and the parallel `controlroom-payment` service (in the `controlroom-payment` repository). These are sibling services in the same payment platform; they are not part of this repository but the inquiry flow depends on both.
