# Flow: URL Redirection

The end-to-end process of resolving a short code to a long URL and redirecting the user, while recording a click event for analytics.

## Trigger

A user issues an HTTP `GET` to `https://sho.rt/{code}` from a browser or client.

## Preconditions

- The short code is well-formed (7 base62 characters, case-sensitive).

## Steps

1. redirect-service receives the request.
2. redirect-service looks up the code in Redis (see `../decisions/ADR-0002-read-through-cache.md` for the cache strategy).
3. On cache hit: continue to step 6.
4. On cache miss: query Postgres, populate Redis with a 1-hour TTL, then continue.
5. If the code is not in Postgres, return HTTP 404.
6. redirect-service publishes a click event to the message queue (best-effort, fire-and-forget).
7. redirect-service returns HTTP 302 with `Location: {long_url}` to the client.

## Outcome

The user is redirected to the original long URL. The click event is enqueued for analytics-service.

## Failure handling

- Code not found in cache and Postgres: return HTTP 404.
- Postgres unavailable: return HTTP 503; the client retries. See `../playbooks/cache-stampede-mitigation.md` for the cache miss storm scenario.
- Cache write failure: log and continue. The next request for the same code will retry the cache populate.
- Message queue unavailable: log and drop the event. Analytics is allowed to lag; user redirect is not blocked.

## Relationships

- `references` `../services/redirect-service.md` (the service that executes the flow).
- `references` `../services/analytics-service.md` (the consumer of click events).
- `references` `../decisions/ADR-0002-read-through-cache.md` (the cache strategy).
- `relates_to` `../playbooks/cache-stampede-mitigation.md` (operational response to a related failure mode).
- `relates_to` `../troubleshooting/elevated-redirect-latency.md` (the user-visible symptom of a related failure mode).
