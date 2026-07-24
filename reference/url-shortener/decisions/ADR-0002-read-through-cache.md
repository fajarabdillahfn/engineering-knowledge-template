# ADR-0002: Use read-through cache with 1-hour TTL for redirects

## Status

Accepted.

## Context

The redirect path is high-traffic and latency-sensitive. Querying Postgres on every request would not meet the p99 target of 50ms at expected load. We needed a caching strategy.

Options considered:
- **Read-through cache with TTL.** The application checks the cache, falls back to the source, populates the cache, and returns.
- **Write-through cache.** The application writes to both the cache and the source on every mutation.
- **Cache-aside with manual invalidation.** The application explicitly invalidates the cache on writes.
- **No cache.** Rely entirely on Postgres.

## Decision

Use a read-through cache with a 1-hour TTL. redirect-service checks Redis first; on miss, reads Postgres, populates Redis, and returns. Writes from api-service populate Redis as a best-effort optimization but do not block on it.

## Consequences

Positive:
- The redirect path can be served entirely from cache for hot codes.
- Cache invalidation is implicit via TTL: stale mappings expire after 1 hour.
- The write path remains simple: best-effort cache populate, no cache invalidation logic.

Negative:
- Stale mappings can be served for up to 1 hour after a change. This is acceptable for the URL shortener use case (mappings are rarely modified after creation).
- Cache stampede is possible when the cache is flushed or expires en masse. See `../playbooks/cache-stampede-mitigation.md`.
- The hit rate must be monitored. A low hit rate negates the benefit. See `../troubleshooting/elevated-redirect-latency.md`.

## Alternatives considered

- **Write-through cache.** Rejected because it adds latency to the write path and requires careful consistency handling between the cache and the source.
- **Cache-aside with manual invalidation.** Rejected because it makes the write path responsible for cache management, complicating api-service.
- **No cache.** Rejected because Postgres alone cannot meet the p99 latency target at expected load.

## References

- `implements` the read path described in `../architecture.md`.
- `implements` the cache lookup in `../flows/url-redirection.md`.
- `implemented_by` `../services/redirect-service.md`.
- `relates_to` `../playbooks/cache-stampede-mitigation.md` (operational response to a known failure mode of this design).
- `relates_to` `../troubleshooting/elevated-redirect-latency.md` (the user-visible symptom of a cache miss storm).
