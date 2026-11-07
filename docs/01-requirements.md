# 01 — Context, Scope and Requirements

## Context

- Backend and frontend microservices are maintained in separate repositories.
- Each service may have a separate automation repository with more than 1,000 TCs.
- Shared E2E automation repositories cover journeys across multiple services.
- Feature and modernization specifications vary (e.g., SpecKit, custom SDD).
- QA must reuse existing tests where they already verify the required behavior.
- Local coding agents are available with GitHub CLI; a dedicated MCP server is not desired.
- Every automation TC is expected to have a Rally Test Case reference. Linked Rally TCs may be unavailable/deleted; found TCs may have a last recorded Rally result that is distinct from live CI execution.
- GitHub Actions can be used where runners and repository policy permit it.

## Goals

**G1 — Discoverability:** Build a complete, queryable inventory of discoverable tests for one automation repository.

**G2 — Reliable provenance:** Maintain source commit, path, test identity, analysis version, and confidence/evidence for all extracted facts.

**G3 — Low token usage:** Perform indexing, normalization, and initial retrieval deterministically without LLM calls.

**G4 — Change correctness:** Reflect added, modified, renamed, and deleted tests without silently retaining stale records.

**G5 — Reuse-first:** Enable a downstream QA evaluator to verify existing test coverage before proposing new tests.

**G6 — Portable architecture:** Define a contract and CLI interaction that works across .NET, Jest, Playwright, and future frameworks.

## In scope

- Event trigger after a push/merge to main.
- Source checkout at the exact triggering commit.
- Framework-specific discovery and reconciliation with AST extraction.
- Snapshot validation and publication to qa-catalog branch.
- Lightweight remote manifest fetch and catalog caching through gh.
- Query-time shortlist, evidence inspection, and coverage decision contract.
- Operational controls, metrics, security, and recovery planning.

## Out of scope

- Executable GitHub workflow or indexer code in this repository.
- An enterprise-wide semantic knowledge graph in phase one.
- Continuous runtime tracing or deep transitive source analysis across every dependency.
- SpecKit/custom-SDD adapters (only their eventual normalized inputs are recognized).
- Automatically modifying or deleting existing tests.

## Functional requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-01 | Trigger indexing after source main advances | Must |
| FR-02 | Enumerate all discoverable tests and variants for configured projects | Must |
| FR-03 | Extract test identifiers, source locations, framework metadata, and assertion/HTTP/API signals when supported | Must |
| FR-04 | Record uncertainty and incomplete extraction explicitly | Must |
| FR-05 | Produce schema-validated full snapshot and manifest | Must |
| FR-06 | Publish only validated snapshots, with source commit SHA | Must |
| FR-07 | Support delete/rename detection through snapshot reconciliation | Must |
| FR-08 | Allow gh-based remote read without checking out catalog branch | Must |
| FR-09 | Return bounded, compact search results to agent | Must |
| FR-10 | Fetch source evidence lazily at pinned SHA | Must |
| FR-11 | Support multi-repo federation through a shared schema | Should |
| FR-12 | Incrementally reuse unchanged parsed results when measured need exists | Could |
| FR-13 | Extract Rally TC references during source indexing without calling Rally | Must |
| FR-14 | Resolve Rally link identity/existence using read-only WSAPI enrichment with explicit uncertainty | Should |
| FR-15 | Capture latest recorded Rally result with provenance/date, never as current runtime test status | Should |
| FR-16 | Refresh Rally metadata without rebuilding source snapshot and track independent freshness | Should |

## Non-functional requirements

| Dimension | Requirement |
| --- | --- |
| Correctness | No stale snapshot promoted as current; no silent discovery gaps |
| Security | Least-privilege repository permissions; preserve repo-level access boundaries |
| Audit | Every candidate result traceable to repository, source SHA and extractor version |
| Cost | Zero LLM calls for indexing; token budgets enforced at QA evaluation time |
| Reproducibility | Snapshot is deterministic for the same source SHA, extractor and config versions, excluding timestamp fields |
| Reliability | Failed jobs retain the previous good snapshot; alerts expose freshness lag |
| Performance | Full scan first; optimize from measurements rather than assumptions |
| Portability | Catalog reader independent of storage backend and QA model provider |

## Threats to correctness

- Framework discovery omits dynamically registered or conditional tests.
- Many discovered variants correspond to one logical test.
- Helper/fixture changes invalidate inferred signals without direct test edits.
- E2E tests verify service behavior via UI, with no direct service-name references.
- Assertion text can be present without the assertion being executed on all paths.
- Similarity scores cannot prove behavior coverage.
- A stale catalog or inaccessible repo cannot support a conclusive NEW decision.

## Stakeholders and responsibilities

| Role | Responsibility |
| --- | --- |
| Automation repository owner | Configure discovery targets and grant runner access |
| Indexer maintainer | Maintain adapters and schemas |
| Platform/DevOps | CI authentication, publishing rights, monitoring |
| QA architect | Coverage rubrics and verified historical evaluation set |
| QA agent | Query, inspect evidence, evaluate coverage, propose reuse/extension/new |
| Human QA lead | Resolve ambiguous business coverage and high-risk gaps |

## Success criteria (proposal)

- All tests discoverable by the configured framework are included or explicitly reported as unmapped.
- Snapshot change accounting correctly reflects add/modify/remove cases.
- Any published record can be located in source at its pinned commit.
- A source mismatch or incomplete discovery is visible, not silently accepted.
- No LLM token cost from scanning 1,000+ TCs.
- On historical features, retrieval recall and reuse precision are measured before enabling automated NEW recommendations.

These criteria must be calibrated to the actual automation frameworks and CI constraints of each enterprise repository.
