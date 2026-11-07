# 04 — Architectural Decisions and Trade-offs

These are **proposed decisions**, intended to be reviewed against the real enterprise environment. They establish direction without defining implementation.

## Decision summary

| # | Decision | Rationale | Trade-off |
| --- | --- | --- | --- |
| D1 | **Deterministic, framework-aware indexing** | Inventory 1,000+ TCs without LLM cost | Does not infer full semantic behavior |
| D2 | **Keep catalogs per automation repo** | Preserves team ownership and source provenance | Federated search must handle many catalogs |
| D3 | **Separate `qa-catalog` branch from `main`** | Derived files do not pollute development history | Need controlled publication and freshness tracking |
| D4 | **Use `gh` CLI as initial access path** | No checkout or MCP service required | Local caching and search responsibility |
| D5 | **Refresh Rally observations independently** | Results and Rally TC existence can change without code merges | Two distinct freshness dimensions |
| D6 | **Normalize requirements before QA planning** | Supports SpecKit and custom SDD | Requires agreement on a common obligation model |
| D7 | **Reuse-first, evidence-backed evaluation** | Limits duplicate automation and false coverage claims | Some decisions require more investigation or human review |
| D8 | **Add centralized/semantic search only when needed** | Keeps the PoC lightweight | Lexical retrieval can initially miss conceptually related tests |

## Why a separate branch instead of generated commits on main?

**Considered approaches:** commit catalog to main, publish to a dedicated branch, or store externally.

**Proposed:** a separate branch per automation repo.

- Keeps feature PRs and main history focused on automation changes.
- Maintains a source-linked, auditable derived artifact.
- Makes adoption possible without a new always-on search service.

**Costs and limits:** branch access permissions, publication ordering and repository history growth need governance. If those become expensive, storage can evolve independently of the catalog's logical contract.

## Why GitHub CLI instead of MCP?

The QA agent already has terminal capabilities and can leverage authorized GitHub tooling. A small CLI-oriented retrieval layer can fetch selected catalog content, cache it, search locally and return bounded results.

MCP is **not prohibited as a future interface**, but it is not an architectural prerequisite. More important than the interface is ensuring that raw catalog data does not flood the LLM context.

## Why not enrich every test using an LLM at indexing time?

At thousands of tests per repository, full AI enrichment adds ongoing cost and creates subjective representations that may look authoritative while being incorrect.

Instead, indexing records objectively observable information. Semantic understanding is applied to a small number of retrieved candidates during QA analysis.

This leaves a real challenge: lexical metadata can miss relevant E2E tests with indirect service relationships. Retrieval quality needs measurement and may justify selective enrichment later.

## Why are Rally data and source data separate?

A source merge changes automation code; a Rally record may be deleted, restored, edited or updated with a recorded result without a merge. Forcing both into a single source-tied snapshot would either produce stale Rally metadata or require unnecessary source rescans.

**Proposed:** one source-linked catalog and one independently observed Rally metadata layer, joined only when identity and freshness can be established.

Not finding a Rally TC is **not** equivalent to confirmed deletion. Last recorded verdict is **not** live runtime status or proof that an assertion is correct.

## Why not treat similar tests as duplicate?

Two tests can cover the same business rule but validate different integration boundaries and risks—for example, API integration vs browser E2E. Reuse decisions consider verification objectives, test layer, test evidence and purpose, not string or embedding similarity alone.

## Evaluator–Optimizer's role

The pattern can help refine requirement interpretation, expand searches, inspect ambiguous coverage and improve the proposed QA plan. It should not automatically invent missing business requirements, delete tests linked to unavailable Rally cases, or create new automation because retrieval found no initial match.

## Alternatives deliberately left open

Central search infrastructure, graph-based impact analysis, embeddings/vector search, richer Rally field integration and detailed test-identity mechanisms remain options. They are not requirements for the architecture's first validation.

## Architectural constraints

- Existing enterprise GitHub permissions and branch policies must be respected.
- The solution must support repositories where GitHub Actions runners are unavailable, via another approved CI capability.
- The catalog must remain portable and model-provider-agnostic.
- Human QA retains authority for uncertain high-risk coverage and test-retirement decisions.
