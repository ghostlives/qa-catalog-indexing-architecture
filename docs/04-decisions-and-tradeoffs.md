# 04 — Design Decisions and Trade-offs

The following choices are recommended for an initial solution. They should be checked against existing repository policies, test frameworks and Rally access before implementation.

## Decision summary

| ID | Status | Decision | Why | Limitation |
| --- | --- | --- | --- | --- |
| D1 | Accepted for pilot | Index test metadata without an LLM | Avoids model cost for thousands of tests | Static analysis cannot explain every test behavior |
| D2 | Accepted for pilot | Keep a catalog per automation repo | Teams keep ownership of their tests | Queries need a registry to find several repos |
| D3 | Provisional | Store catalogs on `qa-catalog` branches | Generated changes stay out of `main` | Security, retention and publication checks must pass |
| D4 | Accepted for pilot | Read catalogs through `gh` | Works with local coding agents without MCP | Clients need authentication, caching and search |
| D5 | Accepted with gate | Refresh Rally separately | Rally records and results change without code merges | Data cannot be persisted until audiences are proven compatible |
| D6 | Accepted | Search before creating tests | Reduces unnecessary test creation | Search results still need source-level coverage review |
| D7 | Accepted for pilot | Start with local search | No new central infrastructure for a pilot | Cross-repo search may become slower at scale |
| D8 | Accepted | Use an owned repository/flow registry | Makes search scope explicit and reproducible | Registry maintenance becomes an operational responsibility |
| D9 | Accepted | Publish immutable snapshots through one logical publisher | Prevents mixed or regressed catalog state | Adds a manifest and retention lifecycle |
| D10 | Accepted | Fail closed to REVIEW on incomplete evidence | Avoids false confident CREATE/REUSE decisions | More cases require human review |

## Catalog storage: main, separate branch or external storage?

Committing the catalog to `main` would keep source and generated data together, but every update would add another commit or PR change. It could also conflict with branch protection rules.

A separate `qa-catalog` branch keeps the source history clean and lets Git track catalog versions. The catalog must identify the source revision it describes. Publishing an older snapshot after a newer merge must not make that older snapshot appear current.

External storage could simplify search across many repositories, but adds infrastructure and permissions to manage. There is no clear need for it before a pilot measures local search performance.

**Recommendation:** start with a separate catalog branch only after branch permissions, serialized publication and retention are proven. Store immutable snapshots and advance a validated manifest; do not let independent jobs overwrite shared mutable files. Reconsider storage if size, update frequency or access segmentation becomes difficult to manage.

## Catalog access: GitHub CLI or a service?

A coding agent already able to run terminal tools can use `gh` to read catalog content at a specific branch. A local search utility can filter it before returning candidates to the model.

This avoids deploying an MCP server or requiring catalog branches to be checked out. However, `gh` only retrieves data; it is not a search engine. Good filtering, caching and candidate ranking still need to be provided outside the LLM.

**Recommendation:** use `gh` for access in the first version. Consider a shared search service only when multiple users or many repositories make local search impractical.

Local clients pin a manifest revision, verify checksums and cache by immutable identity. This avoids combining files from two updates and makes a recommendation reproducible.

## Indexing: source signals or AI-generated descriptions?

An LLM could summarize every test, but doing this after each merge would create recurring cost. The summaries could also claim behavior that the code does not actually verify.

Framework discovery and source analysis offer a more predictable starting point. They can collect names, tags, file locations and some assertion or API signals without model calls. Framework adapters must report their supported versions and unsupported constructs.

The trade-off is search quality. A shared E2E test may be relevant even if none of those fields name the changed service. Source inspection and wider repository search remain necessary.

**Recommendation:** begin with source-derived information and measure how often it misses tests known to QA. Add more advanced search only if the results justify it.

Static parsing is preferred. Framework discovery that imports or executes tests is permitted only in a disposable, read-only sandbox without production secrets or unrestricted network access.

## Rally: one catalog or separate update cycles?

Linking Rally information to a test catalog is useful, but source code and Rally records do not change at the same time. A test result might be recorded in Rally days after the last source merge.

A separate Rally refresh lets the system update that information without rescanning the tests. The catalog must preserve the distinction between a source revision, the time Rally was checked, and the result's actual recorded date.

The phrase “latest result” is ambiguous unless the organization specifies which result relationship, workspace/project, environment or execution type, ordering timestamp and tie-break rule apply. Selecting the newest API row without those semantics can show the wrong status.

An unavailable Rally TC should be flagged for review, not marked deleted without evidence. A stored Pass or Fail should not be presented as the status of the latest CI run.

**Recommendation:** keep Rally observations separate from source-indexed test information, even if the initial storage is in the same branch. Review data access before publishing Rally fields to GitHub. If the Rally audience is narrower than repository readership, keep observations in an access-controlled store or fetch them during review.

## When can QA reuse or create a test?

A test title or search score can point QA to a candidate; it cannot confirm coverage. A useful decision needs the scenario, expected outcome, existing test setup and assertions. The required test level also matters: API integration coverage does not always replace E2E coverage.

**Recommendation:** use the Evaluator–Optimizer pattern to review and refine these decisions. When search scope, source evidence or requirements are incomplete, return REVIEW rather than making a confident CREATE or REUSE recommendation.

## Repository scope: inference or registry?

Inferring repository scope from a requirement on every request is flexible but non-deterministic. A maintained registry makes the default scope reviewable and enables ownership and completeness checks. It does not eliminate QA judgment; manual scope changes are recorded in the evidence.

**Recommendation:** require a versioned repository/flow registry for the pilot. Assign an owner and freshness review. Do not accept no-match conclusions for domains not covered by the registry.

## Catalog identity and updates

Mutable files on a shared branch allow late jobs and independent source/Rally writers to produce inconsistent combinations. Test names are also insufficient as durable identities because they can repeat, be parameterized or change.

**Recommendation:** identify a source snapshot by repository plus source commit; identify tests exactly within that snapshot using framework node identity and source location. Publish immutable content, then atomically advance a manifest through one logical publisher. Treat rename correlation across snapshots as best-effort.

## Security and trust

Repository code, test metadata and specifications are untrusted input. Test discovery may execute imports, and retrieved text may contain instructions aimed at the QA agent. Rally data can also have a narrower audience than GitHub source.

**Recommendation:** isolate executable discovery, strip write credentials from analysis jobs, restrict egress, apply size limits, keep content in a data-only boundary and log access without leaking source or Rally details. Complete threat modeling and data-classification review before the pilot handles production repositories.

## Implementation gates to resolve

1. Name the owner and review process for the repository/flow registry.
2. Approve the Rally-to-GitHub data classification, audience mapping and latest-result selection rule.
3. Select supported frameworks and prove safe discovery for each one.
4. Approve source and Rally freshness objectives, retention and branch-size limits.
5. Confirm an available CI runner and least-privilege publishing identity in pilot repositories.
6. Define the human approval policy for REUSE, EXTEND, CREATE and REVIEW outputs.
7. Approve the benchmark set and pilot exit thresholds.

GitHub Actions runners may not be available in every enterprise repository. The indexing approach must therefore remain usable with another approved CI system. No decision here depends on a particular LLM provider, database or programming language.
