# ADR-0001: Use gRPC for vendor communication

## Status

Accepted.

## Context

`controlroom-inquiry` integrates with 30+ Indonesian payment vendors. Each vendor has its own integration style (REST/HTTP, gRPC, message queue, file-based, etc.). The integration choice has implications for performance, type safety, error handling, and the operational complexity of adding a new vendor.

Options considered:

- **REST/HTTP per vendor.** Each vendor's adapter would use a different HTTP client configuration, request/response shape, and error code taxonomy. Type safety would be limited to whatever the vendor's API provides; many Indonesian vendor APIs are not formally typed.
- **Shared library / SDK per vendor.** Each vendor's adapter would call a vendor-provided Go SDK. This is a strong approach when vendors provide official SDKs with stable contracts, but most Indonesian payment vendors do not.
- **Message queue (RabbitMQ, Kafka).** The service would publish inquiry requests to a queue and consume responses asynchronously. This is appropriate for fire-and-forget payment processing but not for synchronous inquiry, which is the primary use case for this service. Latency and complexity would be too high.
- **gRPC with per-vendor proto contracts.** Each vendor defines its own `.proto` file. The vendor's adapter generates the client and server stubs. Requests and responses are strongly typed. Errors are typed via gRPC status codes. The gRPC client provides built-in retry, load balancing, and observability.

## Decision

Use gRPC for vendor communication, with one `.proto` file per vendor. The vendor is expected to expose a gRPC service; if the vendor does not natively support gRPC, an adapter service (in the vendor's repo) translates between the vendor's native protocol and gRPC. The controlroom-inquiry-side adapter is implemented in `providers/<vendor>/` and is paired with the vendor's `.proto` file.

The shared gRPC client policy is defined in `common/grpc_client.go` and applied uniformly across all vendor adapters:

- Round-robin load balancing.
- 3 retries on `UNAVAILABLE` only.
- Exponential backoff: 0.1s initial, 1s max, multiplier 2.0.
- Unary logging interceptor (records peer address, method, duration, error).

Insecure credentials are used in code; authentication between controlroom-inquiry and vendor services is handled at the network layer.

## Consequences

**Positive:**

- Strong type safety on the wire. Each vendor's request and response schema is defined in its own `.proto` file and enforced by the generated stubs.
- Built-in retry, load balancing, and structured errors via gRPC status codes. No need to reinvent these in each vendor adapter.
- Adding a new vendor is a bounded, well-defined task: define a proto, generate stubs, implement the adapter. The shape of the change is consistent across vendors.
- Observability hooks (the unary logging interceptor) are applied uniformly without per-vendor code.

**Negative:**

- Requires each vendor (or a translation service) to expose a gRPC endpoint. For vendors that only support REST, an extra translation layer is required.
- The vendor's `.proto` file must be maintained alongside the vendor's native API. If the vendor's API changes, the proto must be updated and stubs regenerated.
- The 3-retry policy on `UNAVAILABLE` can amplify load on a slow vendor. There is no built-in circuit breaker; this is a known gap (see FINDINGS).
- The gRPC client's retry policy is a uniform setting across all vendors. A vendor that benefits from a different retry policy (e.g., more retries, longer backoff) requires custom configuration not currently supported.

## Alternatives Considered

- **REST/HTTP per vendor.** Rejected due to inconsistent type safety, inconsistent error handling, and the high cost of building per-vendor resilience code. Many vendor APIs change frequently, and the lack of a typed contract makes change management expensive.
- **Message queue.** Rejected because the primary use case is synchronous inquiry with a low-latency target. Queue-based architecture would add unacceptable latency and complexity.
- **Generic vendor abstraction with map[string]interface{}.** Rejected in favor of per-vendor interfaces. See `ADR-0002-per-vendor-interface.md`.

## References

- `implements` the vendor communication path described in `../architecture.md`.
- `implements` the vendor call step in `../flows/inquiry-v2.md`.
- `implemented_by` the `GetGRPCConnection` factory in `../common/grpc_client.go`.
- `implemented_by` every vendor adapter in `../providers/`.
