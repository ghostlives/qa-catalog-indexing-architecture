# 04 — Design Choices and Trade-offs

The method uses existing source repositories and GitHub tooling to make test discovery easier. The choices below describe the system's operating approach and the limitations QA must account for.

| Choice | Reason | Limitation |
| --- | --- | --- |
| Build catalogs from framework discovery and source analysis | Handles large test suites without an LLM reading every test | Some behaviors are hidden in helpers, fixtures or dynamic test setup |
| Keep a catalog with each automation repository | Test ownership and source references stay together | Searching a feature may involve several repositories |
| Publish catalogs to a separate `qa-catalog` branch | Generated data stays out of the `main` change history | Publication must prevent stale or conflicting updates |
| Use a maintained service/flow repository map | Search scope is explicit and can be reviewed | The map needs ownership and regular maintenance |
| Read catalog files through `gh` | Local agents can access the files without checkout or MCP | A separate local search capability is still needed |
| Refresh Rally records separately | Rally can change even when source code has not | Recorded results can be old or restricted by permissions |
| Search before asking a model to evaluate tests | Controls token use and focuses the review | Initial search can miss tests with indirect dependencies |
| Make QA decisions from actual test evidence | Avoids confusing similar names with equivalent coverage | Some cases still need human judgment |

## Catalog storage

Committing generated catalog data into `main` adds changes that are not part of the automation source and can conflict with normal review rules.

A separate `qa-catalog` branch avoids that problem while retaining Git-based provenance. The catalog references the source commit, so a late indexing run cannot be treated as current merely because it published last.

The trade-off is maintaining the generated branch and ensuring that source updates and Rally refreshes do not overwrite each other. This requires controlled publication and sensible retention of generated data.

External storage is possible, but the selected method does not depend on deploying a separate data service.

## GitHub CLI and search

GitHub CLI handles authenticated access to catalog files on the relevant branch. The agent does not need to clone each repository's catalog branch.

Local search still needs to filter, rank and cache records. It should also report which repositories were searched and which could not be accessed. GitHub CLI alone does not provide a complete cross-repository test search engine.

A central search index is not part of this method. It would only be justified if local catalog access no longer meets the expected coverage, latency or permission requirements.

## Rally information

The Rally reference identifies the managed TC associated with automation code. A lookup can establish whether that record is accessible and retrieve a recorded result under an agreed project and result-selection scope.

Not finding a Rally record does not prove deletion. Likewise, a Rally Pass or Fail does not establish the status of the current automation run.

Rally information must have its own refresh date. Access is also a design constraint: if GitHub repository readers do not have permission to see the Rally fields, those fields cannot be published to that branch.

## Search quality and the role of the model

Deterministic metadata search is inexpensive and easy to inspect, but it may not find an E2E test whose name does not mention the changed service. The service/flow map, related repository search and source inspection reduce this risk; they cannot eliminate it entirely.

An LLM is useful for interpreting requirements and comparing them with shortlisted test code. It should not be asked to read a complete catalog, treat source comments as instructions or conclude that no test exists after one unsuccessful query.

Evaluator–Optimizer can repeat the search or request more evidence, but every iteration has a cost. It stops when the result is clear, new evidence is no longer found, or human review is required.

## Evidence and approval

Each decision should be understandable to another engineer. It records the requirements, search scope, catalog commit, inspected tests and the reason for REUSE, EXTEND, CREATE or REVIEW.

A matching test title alone is insufficient for REUSE. A missing search result alone is insufficient for CREATE. QA retains responsibility for uncertain or high-risk decisions.

## Operational boundaries

The method must respect branch protection, runner availability and repository permissions. Framework discovery that executes code needs an isolated environment. Rally metadata must follow Rally access rules. Catalogs must expose failed or incomplete indexing rather than implying that every test is known.

These safeguards are part of the method, not separate implementation options.
