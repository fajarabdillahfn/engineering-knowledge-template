# Service: controlroom-inquiry

Handles the bill-inquiry path. Receives gRPC `InquiryV2` requests from `api-gateway`, resolves the appropriate vendor, calls the vendor over gRPC, and returns the bill details to the caller.

## Purpose

`controlroom-inquiry` is the synchronous, low-latency component of the payment platform. It exists to answer "what does this customer owe / what can they buy" without the caller needing to know which vendor handles which product. The vendor abstraction is owned entirely by this service; the caller only knows about product codes.

## Responsibilities

- Accepting gRPC `InquiryV2` requests.
- Validating request fields (`ClientID`, `DirectConnection`).
- Resolving the routing context: product, switch, vendor.
- Applying product-specific validation rules (BPJS month, PLN IDPEL length, KAI enrollment).
- Dispatching to the appropriate vendor adapter over gRPC.
- Shaping the response to OpenAPI format.
- Writing the OpenAPI response back to the inquiry log in the background.
- Handling all error paths: invalid product, inactive product, missing switch, inactive vendor, vendor error, vendor unavailable, internal error.

## Required Characteristics

- Stateless. Multiple instances run behind a load balancer.
- gRPC server. Listens on a port configured at startup.
- Latency-sensitive. p99 target under 200ms. Vendor calls dominate the latency budget.
- Read-heavy against Postgres (one routing lookup per request) and write-light (one background write per successful request).
- No built-in authentication. The service assumes `api-gateway` has authenticated the caller before forwarding.
- No rate limiting at the service layer. The caller is expected to enforce quotas.

## MUST Represent

- The request flow: validation → routing → product-specific rules → vendor dispatch → response shaping.
- The dependencies: Postgres (routing and inquiry log), vendor gRPC services.
- The vendor abstraction: per-vendor interface, per-vendor gRPC client, shared retry policy in `common/grpc_client.go`.
- The operation: how to deploy, how to scale, how to observe (logging is via the `ct-logger` library; metrics are not currently emitted from this service).

## MUST NOT Represent

- Business rules that belong in `architecture.md` (e.g., the overall write/read split of the payment platform, the role of `controlroom-payment` as a sibling).
- Operational procedures that belong in `playbooks/` (e.g., how to migrate the routing table, how to onboard a new vendor).
- Specific incidents that belong in `troubleshooting/`.

## Relationships

- `documents` the inquiry path of the payment platform. Referenced by `../flows/inquiry-v2.md`.
- `depends_on` `../decisions/ADR-0001-grpc-for-vendor-comms.md` for the choice of gRPC for vendor communication.
- `depends_on` `../decisions/ADR-0002-per-vendor-interface.md` for the choice of per-vendor interfaces over a generic abstraction.
- `depends_on` `../decisions/ADR-0003-account-number-prefix.md` for the account number normalization rule.
- `depends_on` Postgres (routing table, inquiry log).
- `relates_to` `../troubleshooting/vendor-unavailable.md` (the most common operational failure mode).
- `relates_to` the upstream `api-gateway` service and the parallel `controlroom-payment` service. These are sibling services in the same payment platform; they are documented in their own repositories.
- `implements` the gRPC contract defined in `../grpc/action.proto`.

## Key Files

- `main.go` — entry point. Loads config, initializes logger, starts the gRPC server.
- `grpc/action.proto` — the gRPC contract exposed to `api-gateway`.
- `inquiry/handler.go` — the gRPC handler. Validates the request, dispatches to the service, shapes the response, fires the background update.
- `inquiry/service.go` — the orchestration service. Resolves routing, applies validation, dispatches to prepaid/postpaid.
- `prepaid/service.go`, `postpaid/service.go` — domain services for the two product types.
- `providers/<vendor>/` — vendor adapters. Each implements `I<Vendor>Service` and calls the vendor's gRPC service.
- `common/grpc_client.go` — the gRPC connection factory. Defines the retry policy used by all vendor adapters.
- `models/` — internal data structures. The `InquiryRequest` / `InquiryResponse` / `InquiryData` types are the canonical shapes.
- `repository/` — data access for the routing table and the inquiry log.
- `config/` — per-vendor and per-environment configuration. One file per vendor plus a few shared files (`app.go`, `db.go`, `featureFlags.go`, `log.go`).
