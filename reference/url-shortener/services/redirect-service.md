# Service: redirect-service

Handles the read path. Receives `GET /:code` requests, resolves short codes to long URLs, and returns HTTP 302 responses. Also publishes click events for analytics.

## Purpose

redirect-service is the high-traffic component of the system. It exists to keep redirect latency low while preserving the source of truth in the primary store.

## Responsibilities

- Accepting `GET /:code` requests.
- Looking up mappings in the cache (Redis).
- On cache miss, reading from the primary store and populating the cache.
- Publishing click events to the message queue.
- Returning HTTP 302 responses to long URLs.

## Required characteristics

- Stateless. Multiple instances run behind a load balancer.
- Associated with the platform team.
- Identified by the name `redirect-service`.
- Latency-sensitive: p99 target is 50ms.

## MUST represent

- The request flow: cache lookup, fallback to primary store, cache populate, redirect.
- The cache strategy: read-through with 1-hour TTL (see `../decisions/ADR-0002-read-through-cache.md`).
- The click event publishing: format, queue, partitioning.
- The operation: how to deploy, how to scale, how to observe.

## MUST NOT represent

- Business rules that belong in `../architecture.md`.
- The full mechanics of cache stampede mitigation (see `../playbooks/cache-stampede-mitigation.md`).
- Specific incidents (see `../troubleshooting/elevated-redirect-latency.md`).

## Relationships

- `documents` the read path of the system. Referenced by `../flows/url-redirection.md`.
- `depends_on` `../decisions/ADR-0002-read-through-cache.md` for the cache strategy.
- `depends_on` the primary store, the cache, and the message queue.
- `relates_to` `../playbooks/cache-stampede-mitigation.md`.
- `relates_to` `../troubleshooting/elevated-redirect-latency.md`.
