# 08 — Proof of Concept and Rollout

## Objective

Prove that a deterministic test catalog can index a representative automation repository and enable a reuse-first QA search experience **without MCP and without an LLM call during indexing**.

## PoC scope

Choose:
- One backend or frontend automation repository with approximately 1,000–2,000 tests.
- Its current framework and runner configuration.
- A small set of historical features with QA-verified existing test mappings.
- Optionally one shared E2E repository in phase two, to expose cross-service search issues.

## Delivery phases

### Phase 0 — Baseline & inventory

Deliver:
- Framework/version and test projects identified.
- Expected discovered tests/variants captured.
- Authentication, runner availability and branch protection reviewed.
- Baseline QA test search task with existing human-labeled answers.

Exit: discovery target selection is unambiguous and runner permission is confirmed.

### Phase 1 — Catalog generation

Deliver:
- Full snapshot specification validated against real framework outputs.
- Test identity and variants mapping agreed.
- Observed static signals extracted with provenance.
- Unmapped/unknown cases surfaced.

Exit: discovered counts reconcile; no invisible omissions; no model calls.

### Phase 2 — Versioned publication

Deliver:
- qa-catalog branch publication design exercised.
- manifest carries exact source revision.
- Additional main merges, deletion/rename and concurrent jobs tested.
- Bad/incomplete runs do not replace the prior valid snapshot.

Exit: publication is auditable and stale snapshots cannot be mistaken for current.

### Phase 3 — gh reader & retrieval

Deliver:
- Remote manifest-based sync and local cache behavior.
- Metadata + lexical search with compact results.
- Lazy source evidence read pinned to source SHA.
- Top-K QA recall measured against historical mappings.

Exit: search works without checkout of qa-catalog branch, without MCP, and without passing full catalog into context.

### Phase 4 — QA evaluator trial (separate concern)

Deliver:
- Atomic requirement inputs normalized from at least two source spec styles.
- Human-labeled coverage decisions: REUSE, EXTEND, NEW, REVIEW.
- Evaluation report on recall, precision, unknowns, token cost and failure modes.

Exit: measured improvement over manual/full-context baseline and explicit human acceptance criteria.

## Phase 5 — Rally traceability enrichment

Deliver: source-extracted Rally references, canonical identity mapping, read-only Rally WSAPI resolution, last recorded result with provenance and independent refresh manifest. Validate success, missing/unconfirmed, permission-denied, deleted-confirmed, no-result, stale result and concurrent publication cases. Exit: a Rally result update is visible without a main merge, and the result is never presented as live CI status. See [Rally enrichment acceptance tests](11-rally-enrichment.md).

## Acceptance test matrix

| ID | Scenario | Expected outcome |
| --- | --- | --- |
| AT-01 | Merge new test into main | Catalog includes new logical identity |
| AT-02 | Update assertion only | Evidence and content revision change |
| AT-03 | Delete a test | Removed from new full snapshot |
| AT-04 | Rename test or file | Correct mapping or explicit delete/create |
| AT-05 | Change shared fixture | Dependency impact not silently ignored |
| AT-06 | Add Playwright browser project | Variants increase, logical TC count need not |
| AT-07 | Dynamic discovery cannot be mapped | Diagnostic and degraded completeness |
| AT-08 | Two merges in quick succession | Older result never becomes false current snapshot |
| AT-09 | Unauthorized catalog read | No data exposed |
| AT-10 | Source checkout failure | No publication; existing snapshot retained |
| AT-11 | JSONL schema invalid | Snapshot rejected |
| AT-12 | Catalog is stale vs main | Agent displays STALE |
| AT-13 | Search shared E2E tests | Scope expansion does not skip allowed E2E repositories |
| AT-14 | No match in incomplete scope | UNKNOWN rather than automatically NEW |
| AT-15 | Search result top-K | Output bounded and evidence-linked |
| AT-16 | Index 1,000+ TCs | Zero LLM calls during index |

## Suggested baseline targets (to validate, not guaranteed)

- 100% accounting of discoverable cases: every discovered test/variant maps to a record or explicit unmapped diagnostic.
- Zero duplicate catalog test IDs.
- Zero LLM API calls for indexing.
- 100% of published records have valid source pointers, or are explicitly flagged incomplete and excluded from trusted search.
- No stale snapshot reported as CURRENT.
- An agreed benchmark for Existing TC Recall@K and Coverage Decision Precision, established with QA reviewers.

Do not select fixed accuracy targets until the golden dataset and business risk tolerance are known.

## Rollout gates

| Gate | Decision maker | Evidence |
| --- | --- | --- |
| Architecture approval | QA / engineering leads | ADRs and review |
| Initial adapter validation | Automation owners | Discovery audit |
| Git permissions/branch review | DevOps/security | Least-privilege check |
| Search readiness | QA lead | Labeled retrieval benchmark |
| Enterprise expansion | Architecture board | Cost, quality, operations trends |

## Deferred optimization options

After PoC, consider incremental AST caching, indexed search federation, curated behavior keys, dependency graph enrichment, semantic reranking or vector search only if measurements show a need. Keep the canonical schema backward compatible where possible.
