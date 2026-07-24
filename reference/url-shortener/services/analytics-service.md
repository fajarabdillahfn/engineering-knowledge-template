# Service: analytics-service

Consumes click events from the message queue and aggregates them into metrics: clicks per hour, clicks per day, referrer breakdown, and per-short-code totals.

## Purpose

analytics-service is the asynchronous component of the system. It exists to provide usage insights without adding latency to the redirect path.

## Responsibilities

- Consuming click events from the message queue.
- Aggregating events into per-short-code, per-window metrics.
- Exposing metrics through an internal API and a dashboard.
- Retaining raw events for a configurable period (default: 90 days).

## Required characteristics

- Stateless horizontally; state lives in the analytics data store.
- Associated with the data team.
- Identified by the name `analytics-service`.
- Lag-tolerant: up to 60 seconds of lag is acceptable.

## MUST represent

- The event consumption: queue, consumer group, parallelism.
- The aggregation: windows, retention, query patterns.
- The operation: how to deploy, how to scale, how to backfill.

## MUST NOT represent

- Real-time user-facing concerns (those are the responsibility of redirect-service).
- The format of click events emitted by redirect-service (reference `redirect-service.md` for the contract).

## Relationships

- `depends_on` `redirect-service.md` for click events.
- `depends_on` the message queue and the analytics data store.
- `relates_to` `../architecture.md` for the overall data flow.
