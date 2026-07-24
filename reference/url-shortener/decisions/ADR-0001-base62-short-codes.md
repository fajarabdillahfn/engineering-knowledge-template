# ADR-0001: Use base62 short codes generated per request

## Status

Accepted.

## Context

The system needs to assign a short code to each URL mapping. The code must be:
- Compact (short enough to fit comfortably in a short URL).
- URL-safe (no characters that require percent-encoding).
- Collision-resistant at expected scale.
- Generatable without coordination between api-service instances.

Options considered:
- **Auto-incrementing integers.** Simple, but reveals usage and requires coordination to scale horizontally.
- **UUIDs.** Globally unique, but too long (36 characters) for short URLs.
- **Hash of the long URL.** Deterministic, but collisions are common for similar URLs and require complex resolution.
- **Random base62 strings of fixed length.** Compact, URL-safe, collision-resistant with a retry loop, and trivially distributed.

## Decision

Use random base62 strings of 7 characters as short codes. Generate them in api-service per request, with up to 3 retry attempts on collision. Codes are case-sensitive.

## Consequences

Positive:
- Short URLs are compact (7 characters plus the `sho.rt/` prefix).
- Codes are URL-safe without escaping.
- The space of roughly 3.5 trillion codes is large enough to make collisions rare at expected scale.
- No coordination is needed between instances of api-service.

Negative:
- Collisions are possible, especially near the high end of the space. The retry loop handles them.
- The 7-character length is a fixed commitment. Growing beyond it requires a migration of the primary store.

## Alternatives considered

- **Auto-incrementing integers.** Rejected because they reveal usage and do not scale horizontally without coordination.
- **UUIDs.** Rejected because they are too long (36 characters) for short URLs.
- **Hash of the long URL.** Rejected because collisions are common for similar URLs and are hard to handle gracefully.

## References

- `implements` the write path described in `../architecture.md`.
- `implements` the code generation step in `../flows/url-shortening.md`.
- `implemented_by` `../services/api-service.md`.
