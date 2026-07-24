# Architecture

The URL Shortener is a three-service system that turns long URLs into short codes and redirects users back to the originals. It also records click events for analytics.

## Components

The system is composed of three services and two data stores.

### Services

- **api-service.** Handles the write path. Receives `POST /shorten` requests, validates input, generates short codes, and persists URL mappings.
- **redirect-service.** Handles the read path. Receives `GET /:code` requests, resolves short codes to long URLs, and returns HTTP 302 responses.
- **analytics-service.** Consumes click events from the redirect path and aggregates them into metrics.

### Data Stores

- **Primary store (Postgres).** Source of truth for URL mappings. One row per short code, containing the long URL, creation timestamp, and expiration.
- **Cache (Redis).** Read-through cache in front of the primary store. Holds hot mappings to keep redirect latency low.

## Data Flow

The system has two main flows: shortening a URL and redirecting one.

**Shortening** (write path). A client sends `POST /shorten` to api-service. The service generates a short code, checks for collisions, and writes the mapping to Postgres. The mapping is also written to Redis as a best-effort cache populate. The short URL is returned to the client.

**Redirection** (read path). A user follows a short URL. The request reaches redirect-service, which checks Redis. On hit, the service returns 302 to the long URL. On miss, the service queries Postgres, populates Redis, and returns 302. The service also publishes a click event to a message queue for analytics-service.

The details are in `flows/url-shortening.md` and `flows/url-redirection.md`.

## Boundaries

- The write path is low-traffic and synchronous. p99 latency target: 200ms.
- The read path is high-traffic and latency-sensitive. p99 latency target: 50ms.
- The analytics path is asynchronous. It is allowed to lag by up to 60 seconds.

## Operational Concerns

For operational questions — what to do when something breaks, why the system is shaped the way it is — start here before reading individual assets. Each entry points to the asset that has the full answer.

### Latency or error spikes on the read path

- **Elevated redirect latency (p99 above 100ms).** See `troubleshooting/elevated-redirect-latency.md`. The most common cause is a cache miss storm.
- **Cache miss storm in progress (hit rate below 50% for 2+ minutes).** See `playbooks/cache-stampede-mitigation.md`. Engage request coalescing, warm hot keys, monitor until recovered.

### Data store failure

- **Postgres unavailable.** The redirect path returns HTTP 503. The write path also fails. There is no automated fallback; clients retry. See `flows/url-redirection.md` for the documented failure handling. No dedicated playbook exists for this scenario — it has not been a production issue to date.
- **Redis unavailable.** The redirect path falls through to Postgres on every request. Functionally this is a cache stampede; follow the playbook above.
- **Message queue unavailable.** The redirect path logs and drops click events. Users are still redirected. Analytics lags but does not fail.

### Why the system is shaped this way

- **Why base62 short codes?** See `decisions/ADR-0001-base62-short-codes.md`.
- **Why read-through cache with 1-hour TTL?** See `decisions/ADR-0002-read-through-cache.md`.

## Relationships

- This asset is referenced by every other knowledge asset in the project.
- `references` `decisions/ADR-0001-base62-short-codes.md` (short code generation strategy).
- `references` `decisions/ADR-0002-read-through-cache.md` (cache strategy).
- `references` `troubleshooting/elevated-redirect-latency.md` (operational concerns: latency symptoms and diagnosis).
- `references` `playbooks/cache-stampede-mitigation.md` (operational concerns: cache miss storm recovery).
