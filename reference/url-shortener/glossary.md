# Glossary

Terms used in the URL Shortener project.

## Short Code
A short, fixed-length string that maps to a long URL. The system uses base62-encoded random strings of 7 characters, giving roughly 3.5 trillion possible codes. See `decisions/ADR-0001-base62-short-codes.md` for the rationale.

## Long URL
The original URL provided by the user. Stored verbatim in the primary store.

## Mapping
A (short code, long URL) pair. The unit of data in the primary store.

## Cache Hit
A redirect request served from Redis without touching the primary store. The target for production hit rate is 95% or higher.

## Cache Miss
A redirect request that required a query to the primary store. May be caused by a cold cache, a recently expired TTL, or a key that was never cached.

## Cache Stampede
A situation where many concurrent requests for the same key all miss the cache simultaneously and hit the primary store. Can cause elevated database load and latency. See `playbooks/cache-stampede-mitigation.md`.

## TTL
Time-to-live. The duration a cache entry is considered fresh. Configured at 1 hour for URL mappings. See `decisions/ADR-0002-read-through-cache.md`.

## Click Event
A record of a redirect, published to the message queue for analytics consumption. Contains the short code, timestamp, and user agent.

## Hit Rate
The fraction of redirect requests served from cache over a window. Tracked in the analytics dashboard.

## Base62
An encoding using the characters `0-9`, `a-z`, `A-Z` (62 symbols total). Used to generate short codes that are URL-safe and compact.

## Read-Through Cache
A caching pattern where the application checks the cache first, and on miss, reads from the source of truth and populates the cache before returning. See `decisions/ADR-0002-read-through-cache.md`.
