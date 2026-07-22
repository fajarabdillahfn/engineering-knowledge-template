# Playbook: Cache Stampede Mitigation

How to detect and mitigate a cache stampede on the redirect path, where a flush or expiry of Redis causes a thundering herd of requests to hit Postgres.

## Context

A cache stampede occurs when a large number of redirect requests miss the cache simultaneously and hit Postgres. This can be triggered by:
- A Redis flush or restart.
- A mass key expiry (e.g., after a TTL configuration change).
- A Redis outage.
- A new short code going viral before the cache has been warmed.

Without intervention, the stampede can:
- Drive Postgres CPU to 100%.
- Cause redirect p99 latency to spike from 50ms to 1s or more.
- Potentially cause Postgres to fall over, cascading to api-service and analytics-service.

## Preconditions

- You have access to the production Redis cluster and Postgres.
- You have alerting access (PagerDuty or equivalent).
- You can deploy configuration changes to redirect-service.
- You have runbook access to the cache warmer script.

## Steps

1. **Confirm the stampede.** Check the cache hit rate dashboard. A hit rate below 50% sustained for more than 2 minutes indicates a stampede.
2. **Identify the trigger.** Check recent deploys, recent Redis operations, and Redis health. Look for mass `FLUSHDB` or `FLUSHALL` commands, mass `EXPIRE` changes, or Redis OOM events.
3. **Engage request coalescing.** Enable the request-coalescing feature in redirect-service if it is not already enabled. This causes concurrent requests for the same key to share a single Postgres query, dramatically reducing load.
4. **Warm the cache selectively.** If the trigger is a known set of hot keys (e.g., the top 1000 codes), use the `cache warmer` script to repopulate them.
5. **Monitor.** Watch the hit rate recover and the Postgres CPU drop. Once the hit rate is back above 90% and Postgres CPU is below 60%, the stampede is resolved.
6. **Postmortem.** Within 48 hours, write up the incident: trigger, detection time, mitigation time, impact. Add a Troubleshooting asset if the issue is novel.

## Rollback

If request coalescing causes a regression (e.g., a deadlock or memory issue), disable it via feature flag. The system falls back to the previous behavior (no coalescing), which is less efficient but stable.

## Time estimate

- Step 1 (confirm): 2 minutes.
- Step 2 (identify): 5-10 minutes.
- Step 3 (engage request coalescing): 2 minutes for a config push, plus restart time.
- Step 4 (warm cache): 10-30 minutes depending on key count.
- Step 5 (monitor): ongoing until resolved.

Total active time: roughly 30-45 minutes. The mitigation is usually not complete until the hit rate stabilizes.

## References

- `references` `../services/redirect-service.md`.
- `references` the cache strategy in `../decisions/ADR-0002-read-through-cache.md`.
- `references` the flow in `../flows/url-redirection.md`.
- `relates_to` `../troubleshooting/elevated-redirect-latency.md` (the user-visible symptom of the underlying issue).
