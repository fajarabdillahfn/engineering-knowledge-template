# Changelog

All notable changes to the Engineering Knowledge Specification (EKS) reference repository are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The project does not use formal semantic versioning; the spec itself evolves in place, with findings driving changes.

## [Unreleased]

### Added

- **Agent-readability validation methodology.** A repeatable test for validating that an EKS knowledge base is navigable by AI agents. The methodology: baseline test with 10 questions, apply a change, re-test, compare hop counts. First applied in `reference/url-shortener/AGENT_READABILITY_TEST.md`; replicated in `reference/controlroom-inquiry/AGENT_READABILITY_TEST_2.md` and in the post-refactor re-test.

- **`METADATA_DESIGN.md`.** A proposal for 5 conceptual metadata fields (`status`, `last_validated`, `owner`, `scope`, `out_of_scope`) to address the gap that EKS did not specify a metadata schema. Conceptual only — no serialization format prescribed. Validated empirically in the controlroom-inquiry dogfooding: applying the fields to 3 assets reduced agent hop count by 33% across a 10-question test (27 → 18 hops, with 4-hop answers eliminated and 1-hop count more than tripled).

- **Controlroom-inquiry dogfooding.** A real Go+gRPC bill payment service (Indonesian multi-vendor router for PLN, BPJS, KAI, etc.) documented in 11 EKS assets. The first reference project using a real system rather than a fictional one. See `reference/controlroom-inquiry/`.

- **F1–F24: Findings from three dogfoodings and two re-tests.** The findings drive EKS evolution. The most actionable: F1-F10 (URL Shortener, 8 adopted into `KNOWLEDGE_MODEL.md`), F11 (architecture `Operational Concerns` section, retrofit), F15 (cross-repository navigation, methodology added to `HANDBOOK.md`), F16 (ADR confidence levels, methodology added to `HANDBOOK.md`), F18 (F13 metadata design validated, 5 explicit non-decisions in one service), F21 (architecture routing gap resolved by `out_of_scope` field), F22-F24 (metadata validation findings). See the per-project `FINDINGS.md` for the full list.

- **`reference/url-shortener/` subdirectory.** All 15 URL Shortener files moved from `reference/` to `reference/url-shortener/` to match the `controlroom-inquiry/` directory pattern. Both reference projects now follow the same subdirectory layout, making it clear how to add a third project. Cross-references using project-relative paths still work.

- **`HANDBOOK.md` section 13 — Cross-Repository Navigation.** Methodology for how a service's architecture document should acknowledge its place in a larger platform. References sibling services, upstream gateways, and downstream dependencies by name and repository, with placeholders for not-yet-documented siblings.

- **`HANDBOOK.md` section 14 — ADR Confidence Levels.** Methodology for distinguishing ADRs by epistemic strength: `recorded` (written at decision time, high confidence), `inferred` (reverse-engineered from existing code, medium confidence), `unverified` (aspirational, low confidence). The reverse-engineering methodology is sometimes the only option for legacy systems, but it is not equivalent to recording.

- **`AGENTS.md` "Using Metadata" section.** Guidance for AI agents on the 5 metadata fields, with `out_of_scope` emphasized as the highest-leverage field for navigation. Tells agents to read metadata as part of the asset and to tolerate whatever representation the project uses (YAML, JSON, sidecar, graph database).

- **Roadmap section in `README.md`** with a clear separation between "Completed work" (foundation, both dogfoodings, agent-readability validation, metadata design proposal) and "Forward-looking work" (v0.2 Metadata Spec, v0.3 Validation Gate, v1.0 Reference Implementations).

### Changed

- **Roadmap cleanup.** Dropped the "Career Agent" framing (a product vision, not a spec concern). Reframed v0.3 as "Validation Gate" (not just validation) — the agent-readability test is the gate, not a deliverable. Reframed v1.0 as "Reference Implementations" (not "AI Agent Ecosystem") — vendor-neutral, dogfooding-driven evolution.

- **`KNOWLEDGE_MODEL.md`** — three new relationship types adopted from the URL Shortener dogfooding findings: `caused_by` (§5.7), `replaced_by` (§5.8), `applies_to` (§5.9). Also: clarifications on Playbook vs Troubleshooting (§4.5, §4.6), Flow vs Service (§4.3), intentional duplication (§3.3), and Glossary scope (§4.7).

- **Reference implementation layout.** URL Shortener moved into `reference/url-shortener/` to match the existing `reference/controlroom-inquiry/` pattern. Both projects now follow the same directory structure: `README.md`, `architecture.md`, `glossary.md`, `decisions/`, `flows/`, plus per-project subdirectories for service-specific or operational assets, plus `FINDINGS.md` and `RETROSPECTIVE.md`.

### Removed

- **5 empty directories** at the top of `reference/` (`decisions/`, `flows/`, `playbooks/`, `services/`, `troubleshooting/`) — removed during the URL Shortener refactor. Their content moved into `reference/url-shortener/`.

### Fixed

- **F11 spec-vs-reality gap.** The F9 fix in `KNOWLEDGE_MODEL.md` §4.1 required architecture documents to cross-reference operational assets. The fix was retrofitted into `reference/url-shortener/architecture.md` as a new "Operational Concerns" section. Re-test confirmed Q6 (3am p99 spike) and Q7 (Redis flushed) both dropped to 2 hops.

- **F21 routing gap.** The URL Shortener architecture's `Operational Concerns` section did not have a routing entry for "is X authenticated?" (Q10 in the agent-readability test). The agent-readability test 2 re-test with metadata applied resolved this: the `out_of_scope` field on `architecture.md` directly answered the question in 1 hop. No prose edit was required.

- **Stale path references.** 3 files had body text referring to the old `reference/architecture.md` path. Updated to `reference/url-shortener/architecture.md` after the refactor.

## Reference

For the underlying reasoning behind any of these changes, see the corresponding `FINDINGS.md` and `RETROSPECTIVE.md` in the project. Findings F1–F10 are in `reference/url-shortener/FINDINGS.md`; F11–F14 are in `reference/url-shortener/AGENT_READABILITY_TEST.md`; F15–F20 are in `reference/controlroom-inquiry/FINDINGS.md`; F21–F24 are in `reference/controlroom-inquiry/AGENT_READABILITY_TEST_2.md`.

The agent-readability test methodology — baseline test → apply change → re-test → compare hop counts — is the validation gate for any future spec change. Every proposed field, every new relationship type, every new asset type should pass this gate before adoption.
