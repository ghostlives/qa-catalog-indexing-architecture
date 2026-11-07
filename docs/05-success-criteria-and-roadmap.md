# 05 — Validation and Next Steps

## What we need to prove

The value of this design is not the size of the catalog. It is whether QA can find the right existing tests, make better coverage decisions and spend less time preparing test plans.

We should test the approach using past features for which QA engineers already know which tests were relevant. Those examples provide a baseline for reviewing search results and coverage decisions.

## Measures

| Measure | What it tells us |
| --- | --- |
| Relevant test recall | How often the search finds existing tests that QA identified as relevant |
| Reuse decision accuracy | Whether REUSE recommendations are supported by the actual tests |
| Unnecessary new tests | How often the workflow proposes a TC that should have been reused or extended |
| QA review effort | How much manual investigation the proposed workflow still requires |
| Model usage | Tokens and cost spent on an accepted QA plan |
| Catalog completeness | Whether framework-discovered tests are represented or explicitly reported as unmapped |
| Catalog freshness | Whether test data matches the intended source revision |
| Rally link quality | How many references resolve and how many need investigation |

Set useful thresholds after reviewing actual repositories and historical examples. A single generic accuracy target would hide important differences between test discovery and coverage evaluation.

## Suggested validation stages

### Stage 1 — One automation repository

Pick an automation repository with a representative number of tests, ideally more than 1,000. Confirm that the catalog can account for framework-discovered tests and help find them by name, metadata and source location.

**Main question:** Is the catalog complete enough to use for initial test discovery?

### Stage 2 — Shared E2E coverage

Add a shared E2E repository that touches the same service domain. Use recent feature changes to see whether searches find tests outside the service's own automation repo.

**Main question:** Can the search find relevant cross-service tests that are not obvious from their names?

### Stage 3 — Requirement review

Compare requirements from SpecKit and custom SDD documents. Check whether QA can reach the same expected behaviors and identify missing information before evaluating test coverage.

**Main question:** Does the requirement format affect the quality of the QA plan?

### Stage 4 — Rally information

Use examples of valid Rally TCs, missing or inaccessible links, TCs with no results and TCs with older recorded results.

**Main question:** Can QA use Rally data without confusing historical results with current execution?

### Stage 5 — Evaluate QA decisions

Use historical features with QA-reviewed test mappings. Compare the proposed workflow with current manual search and a baseline that generates scenarios without checking the catalog.

**Main question:** Does the approach find more existing coverage and reduce unnecessary test creation at an acceptable cost?

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

## Decisions needed before detailed design

The architecture review should settle four things:

1. **Search scope:** Which service and shared E2E repositories should be considered for a feature?
2. **Coverage review:** What evidence is sufficient for REUSE, EXTEND or CREATE, and when must a QA engineer review it?
3. **Rally interpretation:** What defines a valid Rally TC in this organization, and which result information may be stored with the catalog?
4. **Operating constraints:** Which repositories have CI runners and permissions to publish catalog data, and what freshness is acceptable?

## Next step

Run a small proof of concept with one service automation repository and one shared E2E repository. Use a set of historical features to validate existing-test discovery, Rally references, requirement interpretation and token cost.

Only after that review should the team decide whether detailed indexing specifications, a shared search service or more advanced retrieval methods are necessary.
