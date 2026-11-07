# 04 — Design Decisions and Trade-offs

The following choices are recommended for an initial solution. They should be checked against existing repository policies, test frameworks and Rally access before implementation.

## Decision summary

| Decision | Why | Limitation |
| --- | --- | --- |
| Index test metadata without an LLM | Avoids model cost for thousands of tests | Static analysis cannot explain every test behavior |
| Keep a catalog per automation repo | Teams keep ownership of their tests | Queries may need data from several repos |
| Store catalogs on `qa-catalog` branches | Generated changes stay out of `main` | Publication and version checks are needed |
| Read catalogs through `gh` | Works with local coding agents without MCP | Clients need authentication, caching and search |
| Refresh Rally separately | Rally records and results change without code merges | Rally data can be stale or access restricted |
| Search before creating tests | Reduces unnecessary test creation | Search results still need coverage review |
| Start with local search | No new central infrastructure for a pilot | Cross-repo search may become slower at scale |

## Catalog storage: main, separate branch or external storage?

Committing the catalog to `main` would keep source and generated data together, but every update would add another commit or PR change. It could also conflict with branch protection rules.

A separate `qa-catalog` branch keeps the source history clean and lets Git track catalog versions. The catalog must identify the source revision it describes. Publishing an older snapshot after a newer merge must not make that older snapshot appear current.

External storage could simplify search across many repositories, but adds infrastructure and permissions to manage. There is no clear need for it before a pilot measures local search performance.

**Recommendation:** start with a separate catalog branch. Reconsider storage if branch permissions, size or cross-repo access become difficult to manage.

## Catalog access: GitHub CLI or a service?

A coding agent already able to run terminal tools can use `gh` to read catalog content at a specific branch. A local search utility can filter it before returning candidates to the model.

This avoids deploying an MCP server or requiring catalog branches to be checked out. However, `gh` only retrieves data; it is not a search engine. Good filtering, caching and candidate ranking still need to be provided outside the LLM.

**Recommendation:** use `gh` for access in the first version. Consider a shared search service only when multiple users or many repositories make local search impractical.

## Indexing: source signals or AI-generated descriptions?

An LLM could summarize every test, but doing this after each merge would create recurring cost. The summaries could also claim behavior that the code does not actually verify.

Framework discovery and source analysis offer a more predictable starting point. They can collect names, tags, file locations and some assertion or API signals without model calls.

The trade-off is search quality. A shared E2E test may be relevant even if none of those fields name the changed service. Source inspection and wider repository search remain necessary.

**Recommendation:** begin with source-derived information and measure how often it misses tests known to QA. Add more advanced search only if the results justify it.

## Rally: one catalog or separate update cycles?

Linking Rally information to a test catalog is useful, but source code and Rally records do not change at the same time. A test result might be recorded in Rally days after the last source merge.

A separate Rally refresh lets the system update that information without rescanning the tests. The catalog must preserve the distinction between a source revision, the time Rally was checked, and the result's actual recorded date.

An unavailable Rally TC should be flagged for review, not marked deleted without evidence. A stored Pass or Fail should not be presented as the status of the latest CI run.

**Recommendation:** keep Rally observations separate from source-indexed test information, even if the initial storage is in the same branch. Review data access before publishing Rally fields to GitHub.

## When can QA reuse or create a test?

A test title or search score can point QA to a candidate; it cannot confirm coverage. A useful decision needs the scenario, expected outcome, existing test setup and assertions. The required test level also matters: API integration coverage does not always replace E2E coverage.

**Recommendation:** use the Evaluator–Optimizer pattern to review and refine these decisions. When search scope, source evidence or requirements are incomplete, return REVIEW rather than making a confident CREATE or REUSE recommendation.

## Constraints to resolve

GitHub Actions runners may not be available in every enterprise repository. The indexing approach should therefore be usable with another approved CI system. Repository permissions and Rally visibility must also be checked before deciding where catalog information can be stored.

No decision here depends on a particular LLM provider, database or programming language.
