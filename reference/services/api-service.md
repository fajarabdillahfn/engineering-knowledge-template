# Service: api-service

Handles the write path for the URL Shortener. Receives `POST /shorten` requests, generates short codes, validates input, and persists URL mappings.

## Purpose

api-service is the entry point for creating short URLs. It is the only service that writes to the primary store directly. Other services do not modify mappings.

## Responsibilities

- Accepting `POST /shorten` requests.
- Validating input URLs (syntax, length, blocklist).
- Generating short codes (see `../decisions/ADR-0001-base62-short-codes.md`).
- Writing mappings to the primary store.
- Writing mappings to the cache as a best-effort optimization.
- Returning the short URL to the client.

## Required characteristics

- Stateless. Multiple instances run behind a load balancer.
- Associated with the platform team.
- Identified by the name `api-service`.
- Latency target: p99 under 200ms.

## MUST represent

- The request flow: input validation, code generation, persistence, response.
- The dependencies: the primary store (Postgres) and the cache (Redis).
- The operation: how to deploy, how to scale, how to observe.
- The contract: request format, response format, error codes.

## MUST NOT represent

- Business rules that belong in `../architecture.md` (e.g., overall write/read split, why the system exists).
- Operational procedures that belong in `../playbooks/` (e.g., how to migrate the database).
- Specific incidents that belong in `../troubleshooting/`.

## Relationships

- `documents` the write path of the system. Referenced by `../flows/url-shortening.md`.
- `depends_on` `../decisions/ADR-0001-base62-short-codes.md` for short code generation.
- `depends_on` the primary store and the cache.
