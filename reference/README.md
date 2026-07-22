# URL Shortener — Reference Project

A fully documented fictional engineering system that exercises the Engineering Knowledge Specification (EKS).

## Purpose

This project exists to validate the EKS specification through practical use. It is not a real product. The intent is to expose strengths and weaknesses of the current specification, not to model a real system with perfect fidelity.

The project is the dogfooding exercise. The lessons learned are in `RETROSPECTIVE.md` and `FINDINGS.md`.

## System Overview

A URL shortener accepts long URLs, assigns them short codes, and redirects users from short codes back to the original URLs. It also tracks click analytics for the shortened links.

The system is composed of three services and two data stores. The full architecture is in `architecture.md`.

## Knowledge Assets

- `architecture.md` — system overview
- `glossary.md` — terms and definitions
- `services/`
  - `api-service.md` — write path
  - `redirect-service.md` — read path
  - `analytics-service.md` — click event aggregation
- `flows/`
  - `url-shortening.md` — create a short URL end-to-end
  - `url-redirection.md` — resolve a short URL end-to-end
- `decisions/`
  - `ADR-0001-base62-short-codes.md` — how short codes are generated
  - `ADR-0002-read-through-cache.md` — how the redirect cache works
- `playbooks/`
  - `cache-stampede-mitigation.md` — how to handle a cache miss storm
- `troubleshooting/`
  - `elevated-redirect-latency.md` — known issue with redirect path latency

## Analysis

- `RETROSPECTIVE.md` — observations from creating this project
- `FINDINGS.md` — missing concepts and concrete improvement candidates
