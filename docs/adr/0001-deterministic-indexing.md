# ADR-0001: Deterministic indexing, no LLM at index time

- **Status:** Proposed
- **Context:** More than 1,000 TCs per repository, many repositories, high model token cost and unreliable semantic guesses.
- **Decision:** Use framework-native test discovery, AST/static extraction, normalization and validation. Do not call LLMs in the indexing pipeline.
- **Benefits:** Predictable cost, reproducibility, source traceability, incremental optimization opportunities, observable extraction failure.
- **Trade-offs:** Some dynamic and helper-heavy behaviors remain unresolved; language/framework adapters require maintenance.
- **Alternatives:** LLM-generated test summaries per TC; raw source embeddings at merge time; manual catalog curation.
- **Why alternatives were not chosen:** They introduce model cost, inconsistent semantic claims or operational burden before retrieval quality is measured.
- **Consequences:** Distinguish observed signals from coverage conclusions; use on-demand semantic evaluation later.
- **Revisit when:** QA retrieval benchmark cannot achieve acceptable recall with structural/lexical signals and bounded enrichment.
