# 10 — Open Architecture Questions

This design deliberately leaves the following decisions for a real repository PoC rather than inventing project-specific capabilities.

| ID | Question | Default assumption |
| --- | --- | --- |
| OQ-01 | Which exact testing frameworks, versions and runners are present? | Adapter per repository |
| OQ-02 | Is GHA runner access available to each enterprise repo? | Use approved runner or alternate CI |
| OQ-03 | Can GITHUB_TOKEN publish to qa-catalog under repository rules? | Validate policies; least privilege |
| OQ-04 | Should qa-catalog have unrelated history to main? | Prefer isolated generated history |
| OQ-05 | What is the maximum expected catalog size and publish frequency? | Full snapshots initially |
| OQ-06 | Do existing tests have explicit stable IDs or requirement labels? | No; fallback to qualified test identity |
| OQ-07 | How much dynamic/parameterized discovery exists? | Track variants separately |
| OQ-08 | Are test fixtures/helpers shared across repos or published packages? | Record unresolved external dependencies |
| OQ-09 | Which GitHub Enterprise host(s) and auth boundaries apply? | gh configured per host |
| OQ-10 | Is a local SQLite cache permitted on development machines? | Yes for PoC; encrypt/protect as needed |
| OQ-11 | Which historical features form the QA-labeled retrieval evaluation set? | QA lead curates examples |
| OQ-12 | How are SpecKit/custom SDD artifacts normalized? | Upstream canonical requirement model |
| OQ-13 | What qualifies as FULL vs PARTIAL coverage across IT and E2E layers? | Rubric defined with QA |
| OQ-14 | What freshness lag is acceptable? | No silent staleness |
| OQ-15 | When should deterministic search add semantic retrieval? | When Recall@K benchmark justifies |
| OQ-16 | How will cross-repo permissions be maintained under a centralized cache? | Enforce source repo access on every query |

## Additional Rally questions

- Where are Rally TC references stored in each framework (annotations, tags, test metadata, external mappings)?
- Is linkage one-to-one, many-to-one, or occasionally one-to-many?
- What constitutes a "valid" Rally TC: existence, lifecycle state, or an organization's custom field?
- Which Rally instance, workspace and project scope are authorized for lookup?
- Should latest result be global per Test Case or scoped to Test Set/environment?
- Which WSAPI last-result fields are available in the enterprise tenant?
- Are Rally result metadata and project names allowed on GitHub catalog branches under security policy?
- How often do Rally results update without code merges, and what freshness threshold is required?

## Decisions intentionally deferred

1. Specific indexer programming language.
2. Whether the search utility uses SQLite FTS5 or in-memory indexes.
3. How to package CLI installation across Copilot, Codex and Claude Code.
4. Whether a central federated index becomes operationally necessary.
5. Exact CI YAML and runner image.
6. Exact language parser versions and dependency graph extraction depth.
7. Fixed Top-K and score thresholds.
8. Any embedding model/vector database configuration.

## Proposal for review meeting

Begin by reviewing one representative automation repository, identifying framework discovery output, mapping 10–20 tests including parameterized and helper-heavy cases, and validating the data contract. Only after that should CI automation and publication be approved.

## Review status

**Proposed.** No implementation work has been performed in this design repository.
