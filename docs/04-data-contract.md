# 04 — Catalog Data Contract

## Design goals

The contract must represent both logical test cases and execution variants, support multiple frameworks, and carry enough evidence for deterministic search without pretending that extracted metadata proves functional coverage.

**Format:** a UTF-8 JSON Lines file (one test record per line), with a separate JSON manifest. The snapshot is immutable with respect to the source SHA and indexer version.

## Test record: logical fields

| Field | Kind | Required | Semantics |
| --- | --- | --- | --- |
| schema_version | string | Yes | Version of canonical catalog record |
| test_id | string | Yes | Repository-scoped stable logical key |
| framework | string | Yes | e.g., xunit, nunit, jest, playwright |
| test_level | enum/unknown | Yes | unit, integration, e2e, contract, unknown |
| source.repo | string | Yes | Full owner/repo identity (and host if enterprise) |
| source.commit_sha | string | Yes | Exact source revision analyzed |
| source.path | string | Yes | Source file path |
| source.symbol | string/null | Yes | Fully qualified test symbol if extractable |
| source.lines | object/null | No | Source span for inspection |
| logical_name | string | Yes | Test case definition label |
| variants | array | Yes | Discovered execution identities/configurations |
| tags | array | Yes | Explicit framework tags/traits/labels |
| searchable | object | Yes | Lexical/structured retrieval signals |
| evidence | array | Yes | Extracted facts with source references |
| dependencies | array | Yes | Referenced helpers/fixtures/API clients |
| extraction | object | Yes | Adapter version, capabilities, confidence/unknowns |
| diagnostics | array | Yes | Unmapped or ambiguous discovery/extraction warnings |

## Stable test identity

Identity is distinct from title. Order of preference:

1. Explicit stable test ID declared by the automation framework/team, if available.
2. Repository + test project + fully qualified symbol + stable variant key.
3. Repository + source path + syntax declaration location/name when stronger identifiers are unavailable.

Potential pitfalls:

- Renaming a test can produce a new ID if there is no explicit stable identifier.
- Moving a file can cause apparent delete/create under a path-derived ID.
- Runtime-generated parameter values may be nondeterministic and unsuitable for ID construction.
- Browser, environment and test data variants are not necessarily separate logical TCs.
- Source line numbers are pointers, not good durable IDs.

## Assertion evidence vs inferred coverage

An evidence entry MUST preserve:

- Signal type: assertion, HTTP operation, fixture, helper reference, tag, etc.
- Extractor: static AST, framework discovery, or explicitly user-maintained annotation.
- Source span: path and approximate line range when present.
- Value/operation in normalized form where safe.
- Confidence category: observed, partial, or unresolved.

**Never persist inferred AC coverage as fact from static text alone.** A field such as coverage_verified should only be set by an independently tracked, evidence-backed evaluation performed later.

## Searchable fields

Recommended compact keyword index:

- Tokenized test name and suite hierarchy.
- Test framework and test level.
- API method/path templates and status literals.
- Explicit domain tags and requirement IDs.
- Assertion function names and non-sensitive expected literals.
- Direct helper/fixture references and known service hints.

Store source code pointers, not full method bodies. Large fixtures, secrets, request payloads, tokens, PII and volatile test data should not be embedded in the catalog.

## Manifest contract

| Field | Required | Purpose |
| --- | --- | --- |
| schema_version | Yes | Catalog schema version |
| repository | Yes | Owner/repo identifier |
| source_branch | Yes | Usually main |
| source_commit_sha | Yes | Exact source SHA indexed |
| catalog_revision | Yes | Content revision/hash |
| indexer_version | Yes | Extraction tool version |
| config_revision | Yes | Discovery configuration revision |
| snapshot_status | Yes | complete, partial, rejected |
| total_logical_tests | Yes | Logical test definitions |
| total_execution_variants | Yes | Discovered runnable variants |
| unmapped_discoveries | Yes | Count of discovery cases not mapped to source |
| files | Yes | Catalog artifacts and integrity hashes |

Time-of-generation is useful operational metadata but is excluded from deterministic content comparison.

## Forward compatibility

- Schema changes use semantic versioning.
- Optional new fields can be added without changing consumer behavior.
- Consumers must reject unsupported breaking versions and avoid silently ignoring required fields.
- Detailed source excerpts remain retrievable only through lazy inspection.

## Synthetic examples

See [mock-data/manifest.json](../mock-data/manifest.json) and [mock-data/tests.jsonl](../mock-data/tests.jsonl). IDs, SHAs and paths in those files are invented and must never be treated as actual test evidence.
