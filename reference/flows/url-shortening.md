# Flow: URL Shortening

The end-to-end process of accepting a long URL from a client, generating a short code, persisting the mapping, and returning the short URL.

## Trigger

A client sends `POST /shorten` to api-service with a JSON body containing the long URL.

## Preconditions

- The client is authenticated (via API key).
- The long URL is syntactically valid and not on the blocklist.
- The long URL is no longer than 2048 characters.

## Steps

1. api-service receives the request and validates the input.
2. api-service generates a candidate short code (see `../decisions/ADR-0001-base62-short-codes.md`).
3. api-service writes the mapping to the primary store (Postgres). If the code already exists, regenerate and retry (up to 3 times).
4. api-service writes the mapping to the cache (Redis) as a best-effort optimization.
5. api-service returns the short URL `https://sho.rt/{code}` to the client with HTTP 201.

## Outcome

On success, the client receives a 201 with the short URL and a `Location` header.

## Failure handling

- Validation failure: return 400 with an error code and message.
- Primary store write failure: return 503. The client retries with exponential backoff.
- Cache write failure: log and continue. The redirect path will repopulate the cache on first access.
- All retries exhausted on collision: return 503 with `code: collision_exhausted`.

## Relationships

- `references` `../services/api-service.md` (the service that executes the flow).
- `references` `../decisions/ADR-0001-base62-short-codes.md` (the short code generation strategy).
