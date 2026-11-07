# ADR-0004: Reuse-first search with lazy evidence verification

- **Status:** Proposed
- **Context:** Existing TCs may already cover new/modernized features across integration and E2E repositories, but titles can hide real behavior.
- **Decision:** Search deterministic catalog signals first. Shortlist candidates, inspect necessary source assertions/fixtures, then let an evaluator issue REUSE / EXTEND / NEW / REVIEW decisions per atomic verification obligation.
- **Benefits:** Reduces duplicate tests, limits model context and requires evidence-backed decisions.
- **Trade-offs:** Retrieval can miss semantically relevant tests. Low confidence or incomplete index must escalate to review.
- **Alternatives:** Generate new TCs by default; infer coverage from name similarity; load full catalog into prompt.
- **Why alternatives were not chosen:** Duplicate test proliferation, false positives and high token usage.
- **Invariants:** Search relevance is not proof of coverage; no-result is not proof of absence; level-specific tests may not be redundant.
- **Revisit when:** Labeled recall/precision data show another retrieval/evaluation workflow performs better.
