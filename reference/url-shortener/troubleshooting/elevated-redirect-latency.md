# Troubleshooting: Elevated Redirect Latency

Redirect p99 latency has risen above the 100ms threshold, far above the 50ms target. User-visible impact: slow page loads for users following short URLs.

## Issue

The redirect path is slower than its SLO. Affects all users in production.

## Symptoms

- Alert: redirect p99 latency > 100ms for 5+ minutes.
- User reports: "short URLs are slow to redirect".
- Dashboard: redirect p99 latency trending up; cache hit rate trending down.

## Root cause

The most common cause is a cache miss storm: many requests missing Redis simultaneously, causing elevated Postgres load and longer query times. This can be triggered by:
- A Redis flush or restart.
- A mass key expiry (e.g., after a TTL configuration change).
- A new short code going viral before the cache has been warmed.

Less common causes:
- A Postgres query plan regression after a migration.
- A network issue between redirect-service and Postgres or Redis.
- A noisy-neighbor issue on shared infrastructure.

## Diagnosis

1. Check the cache hit rate dashboard. If it is below 90%, the issue is likely cache-related.
2. Check the Postgres slow query log. If there are many slow `SELECT` queries for the mappings table, the issue is Postgres load.
3. Check the recent deploy history for redirect-service. A recent deploy can introduce a regression.
4. Check the network metrics between redirect-service and the data tier.
5. Check the analytics-service queue depth. A backlog can sometimes be correlated with redirect latency if the message queue is shared.

## Resolution

For a cache miss storm:
1. Engage the cache stampede mitigation playbook: `../playbooks/cache-stampede-mitigation.md`.
2. Enable request coalescing in redirect-service.
3. Warm the cache for the top hot keys.

For a Postgres query plan regression:
1. Identify the slow query.
2. Roll back the migration that caused it.
3. Re-run the query plan analyzer to confirm the regression is gone.

For a network issue:
1. Engage the network on-call.
2. Failover to the secondary region if the issue is region-wide.

## Prevention

- Monitor the cache hit rate continuously. Alert if it drops below 90%.
- Set Redis memory limits to prevent silent eviction of hot keys.
- Test deployments with realistic load before rolling out to production.
- Have the cache warming script readily available and tested.

## References

- `references` `../services/redirect-service.md`.
- `references` `../playbooks/cache-stampede-mitigation.md` for mitigation steps.
- `references` the cache strategy in `../decisions/ADR-0002-read-through-cache.md`.
- `references` the flow in `../flows/url-redirection.md`.
