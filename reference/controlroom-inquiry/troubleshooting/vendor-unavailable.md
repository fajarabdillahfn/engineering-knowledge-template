# Troubleshooting: Vendor gRPC UNAVAILABLE

A vendor's gRPC service is returning `UNAVAILABLE` for one or more products. The gRPC client retries up to 3 times on `UNAVAILABLE`; if the vendor is still down after retries, the inquiry fails with `models.InquiryFail`.

## Issue

Inquiries for products routed to a specific vendor are failing with a high error rate or a high latency. The `peer` field in the gRPC logging interceptor (configured in `common/grpc_client.go`) identifies the affected vendor.

## Symptoms

- Alert: error rate for `InquiryV2` is elevated, OR p99 latency is elevated (because the gRPC client is retrying).
- Customer reports: "I can't check my bill" for a specific type of bill.
- Dashboard: per-vendor error rate spikes for one vendor, or per-vendor latency spikes for one vendor.

## Root Cause

The most common cause is that the vendor's gRPC service is down, restarting, or under heavy load. Less common causes:

- A network issue between this service and the vendor's gRPC endpoint.
- A misconfigured vendor endpoint in the routing table (the target host/port is wrong).
- A recent deploy to the vendor service that introduced a regression.

The gRPC client retries up to 3 times on `UNAVAILABLE` with exponential backoff (0.1s → 1s). If the vendor is genuinely down, all 3 attempts will fail and the inquiry returns `models.InquiryFail`. If the vendor is just slow, the retry may succeed; the call will take longer than usual but the customer will not see a failure.

## Diagnosis

1. Identify the affected vendor. The unary logging interceptor in `common/grpc_client.go` records the `peer` field for every outgoing gRPC call. Filter logs by the affected product codes and identify the vendor by the `peer` address.
2. Check the vendor's status page (if the vendor has one). Many Indonesian payment vendors maintain a status page or have a public incident report channel.
3. Check the network between this service and the vendor's gRPC endpoint. Use `kubectl exec` to shell into a pod and try `nc -zv <vendor-host> <vendor-port>`. If the connection fails, the issue is at the network layer.
4. Check the routing table in Postgres. Verify that the product code is mapped to the expected vendor code and that the vendor's target address is correct. The target address is configured in `config/<vendor>.go`.
5. Check the recent deploy history for this service. A recent change to `config/<vendor>.go` or to the gRPC client policy could have introduced a misconfiguration.

## Resolution

For a vendor-side outage:

1. There is no automated fallback in this service. The inquiry for the affected product will fail.
2. If the vendor is the only one mapped to the affected product, there is no way to recover without the vendor coming back up. The customer-facing service (api-gateway) will receive the failure and may retry.
3. Communicate the incident. The service does not have a built-in incident communication mechanism. The platform team should be notified through the usual channels (PagerDuty, Slack, etc.).
4. Once the vendor is back, no action is required on this service. The next inquiry will succeed.

For a network issue:

1. Engage the network on-call. The issue is between this service and the vendor's gRPC endpoint.
2. If the vendor's gRPC endpoint is region-specific, failover to the secondary region if available. The failover procedure is not documented in this service; consult the platform team's runbooks.

For a misconfiguration:

1. Identify the misconfigured field. The most common is the vendor's target address in `config/<vendor>.go`.
2. Fix the configuration. The change is loaded at startup; a restart of the service is required.
3. Verify that the next inquiry succeeds. The gRPC logging interceptor will show the new `peer` address.

## Prevention

- **Monitor per-vendor error rate and latency.** A sudden spike for one vendor is the earliest signal. The gRPC logging interceptor provides the data; the alerting system needs to be configured to consume it.
- **Add a circuit breaker.** This service currently has no circuit breaker. The 3-retry policy can amplify load on a slow vendor. A circuit breaker (open after N consecutive failures, close after M successes) would prevent this. This is a known gap (see FINDINGS F18).
- **Cache the routing table.** The current implementation queries Postgres on every inquiry. A cached routing table (with TTL and invalidation on update) would make the service resilient to Postgres outages. This is also a known gap.
- **Set up a vendor health check.** A periodic probe (e.g., a gRPC health check) would detect vendor outages faster than customer-driven inquiries. The gRPC standard health-check protocol is supported; it is not currently used in this service.

## References

- `references` `../services/controlroom-inquiry.md` (the service that detects this failure).
- `references` `../flows/inquiry-v2.md` (the flow in which this failure occurs).
- `references` the gRPC client policy in `../common/grpc_client.go` (the retry policy that amplifies load).
- `references` `../decisions/ADR-0001-grpc-for-vendor-comms.md` (the choice of gRPC for vendor communication, which makes this failure mode possible).
- `relates_to` the vendor-specific troubleshooting entry. If a single vendor has a known issue, the vendor's repository should have its own entry.
