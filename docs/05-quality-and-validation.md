# 05 — Quality and Validation

## What success means

The method works when QA can find relevant existing tests, explain what they cover and avoid creating unnecessary automation. A large catalog is useful only if it improves those decisions.

Validation should use real examples from completed features for which QA engineers can identify the expected behavior and relevant existing tests.

## Quality measures

| Measure | How to assess it |
| --- | --- |
| **Relevant test recall** | Compare retrieved candidates with the tests QA engineers identify as relevant for a feature |
| **Reuse decision accuracy** | Check whether REUSE decisions are supported by reviewed setup and assertions |
| **Unnecessary CREATE rate** | Count new-test recommendations overturned because existing coverage was missed |
| **Catalog completeness** | Confirm that tests discovered by the supported framework are represented or explicitly excluded |
| **Repository scope coverage** | Check whether the relevant service, frontend and shared E2E repositories were included |
| **Source freshness** | Check that catalog records match the source commit under review |
| **Rally clarity** | Confirm that unresolved links and recorded results are presented with their correct dates and meanings |
| **QA review effort** | Compare the time and source investigation needed with the current process |
| **Model cost** | Track model usage per accepted QA plan; separate catalog search from LLM assessment |

Measurements should be reported by repository or test type where that matters. A strong overall score can hide failures in shared E2E retrieval.

## Scenarios the method must handle

| Scenario | Expected behavior |
| --- | --- |
| A test is renamed, added or removed | Catalog reflects the indexed source and retains the correct source reference |
| A test uses dynamic parameters or unsupported syntax | Catalog reports what it cannot account for |
| A relevant test is in a shared E2E repository | Repository scope includes that suite and search can return it |
| Search matches a title but not the required assertion | QA does not mark the requirement covered |
| Search returns no results from an incomplete repository set | Recommendation is REVIEW, not CREATE |
| The Rally TC is inaccessible | Link status is unresolved, not automatically deleted |
| Rally shows an old Pass while the current run fails | The historical Rally result is kept separate from runtime evidence |
| Rally is updated without a code merge | The Rally observation can refresh without rescanning tests |
| A catalog publication fails or finishes out of order | The last valid information remains available and stale status is visible |
| Requirements are ambiguous | QA review is requested instead of inventing a business rule |
| Repository content contains misleading instructions | Such text is treated as source data, not agent instructions |

## How to validate the method

Use one service automation repository and one related shared E2E repository as a representative test set. Choose completed features where QA already knows the relevant tests and expected results.

Evaluate the full review path: requirement understanding, repository selection, candidate search, source evidence, Rally interpretation and final recommendations.

Compare the outcome with the current manual QA process. Track missed tests, unsupported reuse recommendations, unnecessary CREATE recommendations, review time and model use.

Repeat the evaluation on more repositories and feature types to see whether the findings hold outside the original domain. Set acceptance thresholds using these results and the risk level of the applications involved.

## Ownership and review

| Responsibility | Owner |
| --- | --- |
| Source automation tests and repository catalog quality | Automation repository team |
| Service/flow-to-repository mapping | Service and QA owners |
| Rally result meaning, record visibility and access policy | Rally owner and security team |
| Coverage criteria and final QA approval | QA lead |
| CI execution boundaries and catalog publication | DevOps/platform team |
| Measurement of retrieval and decision quality | QA architecture and participating teams |

## Conditions for a reliable decision

Before QA treats REUSE or CREATE as supported, the relevant requirements must be clear, the intended repositories must be accounted for, catalog information must be sufficiently current, and the source evidence must have been inspected.

When these conditions cannot be met, REVIEW is the appropriate output. This protects the workflow from both duplicated automation and false confidence that important behavior is covered.

The solution remains a method for finding and reviewing existing tests. It does not define test implementation details or replace the judgment of the QA engineers responsible for coverage.
