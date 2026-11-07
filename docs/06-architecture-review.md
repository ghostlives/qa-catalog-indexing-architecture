# 06 — Principal Architecture Review

## Review conclusion

The proposed direction is suitable for a proof of concept. It correctly avoids bulk LLM indexing, keeps test ownership in existing repositories, separates source and Rally update cycles, and requires source inspection before making coverage claims.

The original proposal was not implementation-ready because several cross-component responsibilities were implicit. The revised architecture addresses those gaps with a repository/flow registry, immutable catalog snapshots, a validated publisher, explicit evidence records, safe discovery controls and measurable pilot gates.

Production adoption remains conditional on the open organizational and security decisions below.

## Findings and dispositions

| Priority | Finding | Risk | Disposition in this revision |
| --- | --- | --- | --- |
| Critical | Rally data may be visible to fewer people than the GitHub catalog | Unauthorized disclosure | Added an authorization gate and secure-store/live-fetch alternative |
| Critical | Framework discovery may execute repository code | Credential theft or network side effects | Added static-first extraction and isolated discovery requirements |
| High | “Latest Rally result” had no deterministic selection semantics | Wrong historical verdict shown to QA | Added an approved, versioned selection rule and ambiguous status |
| High | No deterministic way to choose all relevant repositories | False no-match and duplicate test creation | Added a versioned repository/flow registry |
| High | Source and Rally jobs could overwrite or mix branch state | Stale or internally inconsistent catalog | Added immutable snapshots and an atomic, conflict-aware publisher |
| High | Test and snapshot identity were underspecified | Wrong Rally/test joins and non-reproducible review | Added minimum logical contracts and commit-pinned identity |
| High | Empty search could be interpreted as a coverage gap | Incorrect CREATE recommendation | Added evidence-completeness states and fail-closed REVIEW behavior |
| High | Recommendations lacked a durable evidence trail | Decisions cannot be audited or evaluated | Added a pinned QA evidence record |
| Medium | Framework, dynamic and parameterized test support was implicit | Silent catalog omissions | Added adapter capability reporting and discovery reconciliation |
| Medium | Branch history and local caches had no lifecycle | Unbounded growth and stale reads | Added retention, checksum, cache-key and compaction requirements |
| Medium | Evaluator loop had no operational stop conditions | Cost and latency growth | Added iteration, evidence-change and cost limits |
| Medium | Success measures had no definitions or gates | Pilot could not produce a go/no-go decision | Added formulas, benchmark design and provisional gates |
| Medium | Failure, rollback and observability were absent | Slow detection and unsafe recovery | Added last-known-good manifest behavior and operational signals |

## Required decisions before implementation

The accountable architecture, QA, security and Rally owners must record owners and due dates for these gates:

| Gate | Approval needed |
| --- | --- |
| Repository registry | System of record, owner, update workflow and domain coverage |
| Rally data | Field allowlist, audience equivalence, result-selection rule, retention and secure fallback location |
| Framework adapters | Pilot frameworks/versions, authoritative discovery command and isolation profile |
| Publication | Publisher identity, branch protection, concurrency strategy and rollback authority |
| Service objectives | Source/Rally freshness, search latency, availability and retention targets |
| Human control | Which recommendation types require approval and where that approval is recorded |
| Evaluation | Frozen benchmark, reviewer agreement process and final exit thresholds |

## Recommended proof-of-concept shape

Use one service repository and one shared E2E repository in the same business flow. Implement only the adapters required by those repositories. Start with deterministic metadata and lexical search through `gh`; do not add a vector database or central service during the first measurement cycle.

The proof of concept should demonstrate the entire thin slice: registry resolution, isolated discovery, deterministic snapshot, conflict-safe publication, authorized Rally observation, local shortlist, source evidence review, recommendation record and failure recovery. A successful demo without reconciliation, authorization and benchmark evidence is not sufficient to approve expansion.

## Deliberately deferred decisions

- Central search service or vector database.
- Semantic embeddings for test retrieval.
- Current CI result ingestion.
- Automated modification or creation of test code.
- Organization-wide catalog schema beyond the minimum logical contract.
- Provider-specific LLM, implementation language and CI product.

These can be revisited using pilot evidence. None is required to test the central hypothesis that deterministic catalogs and evidence-bounded analysis improve QA planning.
