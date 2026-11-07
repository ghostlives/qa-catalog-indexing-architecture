# 05 — Validation and Next Steps

## What we need to prove

The value of this design is not the size of the catalog. It is whether QA can find the right existing tests, make better coverage decisions and spend less time preparing test plans.

We should test the approach using past features for which QA engineers already know which tests were relevant. Those examples provide a baseline for reviewing search results and coverage decisions.

## Measures

| Measure | Definition | What it tells us |
| --- | --- | --- |
| Relevant test recall@K | QA-known relevant tests returned in the first K candidates / all QA-known relevant tests | Whether ranking finds useful tests within a reviewable set |
| Supported-test accounting | Discovered tests represented or explicitly excluded / all tests discovered by the supported framework | Whether catalog gaps are visible |
| Reuse precision | QA-approved REUSE decisions / all proposed REUSE decisions | Whether reuse claims are supported by actual tests |
| Unsupported CREATE rate | CREATE decisions overturned because existing coverage or incomplete evidence was missed / all CREATE decisions | Whether the workflow creates avoidable duplication |
| QA review effort | Median and p90 elapsed review time plus source files inspected per feature | How much manual investigation remains |
| Model usage | Tokens and cost per accepted QA plan, separated by search and evaluation iteration | Whether bounded retrieval controls cost |
| Source freshness | Time from default-branch commit to a valid current manifest | Whether the catalog matches the intended source revision |
| Rally freshness | Time since lookup, shown separately from result-recorded age | Whether Rally context is recent enough |
| Search performance | Cold and warm p50/p95 time to shortlist candidates across the resolved repository scope | Whether local retrieval remains practical |
| Reliability | Successful publications / attempted publications, plus stale-manifest duration | Whether the workflow fails visibly and recovers |
| Rally link quality | References by FOUND, hidden/forbidden, not found, no result and error status | Which data-quality problems need ownership |

Set final thresholds after baselining actual repositories and historical examples. The following are **provisional pilot gates**, to be approved before execution:

- 100% of tests returned by a supported framework are catalogued or carry an explicit exclusion/error reason.
- At least 90% recall@20 on the QA-reviewed historical benchmark, reported per repository type as well as overall.
- No confident CREATE or REUSE recommendation is allowed when a required catalog is stale, partial, failed, unauthorized or missing.
- Every sampled recommendation can be reproduced from pinned requirement, registry, manifest and source revisions.
- No unresolved high-severity finding from the security and Rally data-access reviews.
- Source and Rally freshness objectives are met in at least 95% of pilot observations once their target durations are approved.

These gates are safety and learning criteria, not a claim of production readiness. Report the benchmark size and confidence intervals; do not hide a weak repository behind an overall average.

## Benchmark design

Select historical features before tuning search. Include service-only, frontend, shared E2E, renamed tests, parameterized tests, missing/inaccessible Rally links, ambiguous requirements and known no-coverage cases. Two QA reviewers should agree on the expected relevant-test set and acceptable decision, resolving disagreements before scoring.

Keep a simple baseline: current manual QA search and keyword-only catalog search. Evaluate changes against the same frozen benchmark and record any training or tuning examples separately from the final evaluation set.

## Suggested validation stages

### Stage 1 — One automation repository

Pick an automation repository with a representative number of tests, ideally more than 1,000. Confirm that the catalog can account for framework-discovered tests and help find them by name, metadata and source location.

**Main question:** Is the catalog complete enough to use for initial test discovery?

**Exit evidence:** adapter support matrix, discovered/catalogued/excluded reconciliation, safe-discovery review, deterministic re-run and manifest publication/rollback test.

### Stage 2 — Shared E2E coverage

Add a shared E2E repository that touches the same service domain. Use recent feature changes to see whether searches find tests outside the service's own automation repo.

**Main question:** Can the search find relevant cross-service tests that are not obvious from their names?

**Exit evidence:** approved registry entries, recall@20 by repository type and documented false negatives.

### Stage 3 — Requirement review

Compare requirements from SpecKit and custom SDD documents. Check whether QA can reach the same expected behaviors and identify missing information before evaluating test coverage.

**Main question:** Does the requirement format affect the quality of the QA plan?

**Exit evidence:** normalized scenarios retain source IDs and versions; ambiguous requirements result in REVIEW rather than invented behavior.

### Stage 4 — Rally information

Use examples of valid Rally TCs, missing or inaccessible links, TCs with no results and TCs with older recorded results.

**Main question:** Can QA use Rally data without confusing historical results with current execution?

**Exit evidence:** approved audience/data classification, status mapping, deterministic latest-result selection, lookup/result timestamp display, ambiguous-result and rate-limit behavior, and no unauthorized persisted fields.

### Stage 5 — Evaluate QA decisions

Use historical features with QA-reviewed test mappings. Compare the proposed workflow with current manual search and a baseline that generates scenarios without checking the catalog.

**Main question:** Does the approach find more existing coverage and reduce unnecessary test creation at an acceptable cost?

**Exit evidence:** comparison with baselines, decision precision, unsupported CREATE rate, review time, reproducibility and token/cost distribution.

### Stage 6 — Failure and operational readiness

Exercise late jobs, concurrent source/Rally updates, malformed catalog output, checksum failure, unavailable GitHub/Rally, cache corruption, branch growth and publisher credential rotation.

**Main question:** Does failure preserve the last valid catalog, expose degraded freshness and recover without manual data repair?

## Main risks

| Risk | What to check during validation |
| --- | --- |
| Relevant E2E tests are not found | Search across related repositories and compare with QA-known cases |
| Tests are discovered but not understood | Check source assertions and helper dependencies for selected candidates |
| Source catalog is stale | Verify the referenced source revision before coverage review |
| Rally TC cannot be accessed | Report the uncertainty; do not assume deletion |
| Rally stored result is old | Show the result date separately from lookup date and CI status |
| Catalog access differs between teams | Confirm GitHub and Rally permissions before sharing data |
| Local search does not scale | Measure retrieval time and maintenance cost before proposing a central index |
| Wrong repositories are searched | Pin the registry revision and review scope overrides |
| Concurrent writers lose or mix updates | Use immutable snapshots and an atomic, conflict-aware manifest publisher |
| Discovery executes unsafe repository code | Prefer static parsing; sandbox required execution without secrets or open egress |
| Rally data reaches unauthorized readers | Complete audience mapping and keep restricted observations outside GitHub |
| Retrieved content influences agent instructions | Treat content as data, constrain tools and retain human approval for decisions |
| Generated branch grows indefinitely | Apply retention, monitor size and rehearse compaction |

## Decisions needed before detailed design

The architecture review must settle these implementation gates:

1. **Registry ownership:** Who maintains service/flow-to-repository mappings and approves changes?
2. **Coverage review:** What evidence is sufficient for REUSE, EXTEND or CREATE, and which decisions require a QA approver?
3. **Rally interpretation and authorization:** What defines a valid TC, how is the applicable latest result selected, and which fields may be persisted for which audience?
4. **Framework support:** Which discovery adapters and framework versions are in the pilot, including parameterized and dynamic tests?
5. **Operating targets:** What source/Rally freshness, retention, availability and search-latency objectives are acceptable?
6. **Security:** Which runner isolation, network, secret, logging and content-handling controls are mandatory?
7. **Pilot evaluation:** Which frozen benchmark and numerical exit gates authorize expansion?

## Next step

First close the Rally authorization, safe-discovery and registry-ownership gates. Then run a small proof of concept with one service automation repository and one shared E2E repository. Use a frozen set of historical features to validate existing-test discovery, Rally references, requirement interpretation, reproducibility and token cost.

Only after that review should the team decide whether detailed indexing specifications, a shared search service or more advanced retrieval methods are necessary.
