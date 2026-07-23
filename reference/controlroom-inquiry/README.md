# controlroom-inquiry — Reference Project

The EKS knowledge base for the `controlroom-inquiry` service. This is the second EKS dogfooding exercise; the first was a fictional URL Shortener. Unlike the URL Shortener, this is a real, in-production, multi-vendor Indonesian bill payment service.

## Purpose

This knowledge base exists to validate the EKS specification against a real system. The lessons learned are in `RETROSPECTIVE.md` and `FINDINGS.md`. The spec should NOT change unless real usage demonstrates a weakness.

The system itself is documented in the assets below. The knowledge base is intended to be useful to:

- New engineers joining the team (entry point to the system).
- On-call engineers handling incidents (navigation to troubleshooting and playbooks).
- Code reviewers (canonical reference for what the code is supposed to do).
- AI agents answering questions about the system (validated by the agent-readability methodology from the URL Shortener dogfooding).

## System Overview

`controlroom-inquiry` is a Go + gRPC service that handles bill inquiries for the AyoPop payment platform. It receives gRPC `InquiryV2` requests from `api-gateway`, resolves the appropriate vendor, calls the vendor over gRPC, and returns the bill details to the caller.

The service integrates with 30+ Indonesian payment vendors, including PLN, BPJS, KAI, BNI, BRI, Indosat, GoPay, ShopeePay, Traveloka, Dana, and OVO. Each vendor is implemented as a separate gRPC service in its own process; this service calls them through per-vendor adapter packages.

The full architecture is in `architecture.md`.

## Knowledge Assets

- `architecture.md` — system overview, components, data flow, operational concerns
- `glossary.md` — domain terms (PLN, BPJS, IDPEL, switch, vendor, etc.)
- `services/`
  - `controlroom-inquiry.md` — the service itself (the only service asset in this knowledge base)
- `flows/`
  - `inquiry-v2.md` — the end-to-end inquiry flow
- `decisions/`
  - `ADR-0001-grpc-for-vendor-comms.md` — why gRPC for vendor communication
  - `ADR-0002-per-vendor-interface.md` — why per-vendor interfaces instead of a generic abstraction
  - `ADR-0003-account-number-prefix.md` — why we prepend "0" to certain account numbers
- `troubleshooting/`
  - `vendor-unavailable.md` — what to do when a vendor's gRPC is down

## Analysis

- `RETROSPECTIVE.md` — observations from documenting this real system
- `FINDINGS.md` — candidate improvements to EKS based on this dogfooding (F15–F20)

## Reading Order

For a new engineer joining the team, read in this order:

1. `glossary.md` — domain terms first. Without these, the rest will not parse.
2. `architecture.md` — high-level system overview.
3. `flows/inquiry-v2.md` — the end-to-end flow.
4. `services/controlroom-inquiry.md` — the service in more detail.
5. `decisions/ADR-0001-grpc-for-vendor-comms.md` and `decisions/ADR-0002-per-vendor-interface.md` — the key architectural decisions.
6. `decisions/ADR-0003-account-number-prefix.md` — a specific business rule.
7. `troubleshooting/vendor-unavailable.md` — the most common operational failure mode.

For an on-call engineer handling an incident:

1. `architecture.md` — the "Operational Concerns" section at the bottom.
2. `troubleshooting/vendor-unavailable.md` — the most common failure mode.
3. The relevant vendor's code in `providers/<vendor>/` (if the issue is vendor-specific).

For an AI agent:

1. Start with `architecture.md`. It is the entry point and includes the "Operational Concerns" routing layer.
2. Follow the cross-references from there.
3. The `glossary.md` is the secondary entry point for domain terms.
