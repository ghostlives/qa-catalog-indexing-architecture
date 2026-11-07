# 05 — Success Criteria, Scope and Evolution

## How should this solution be judged?

The architecture should be validated by whether it makes QA decisions **more correct and more efficient**, not by how many catalog records or AI-generated test cases it produces.

| Outcome | Measure | Why it matters |
| --- | --- | --- |
| Better existing-test discovery | Recall of QA-confirmed relevant TCs among retrieved candidates | Prevents missed reuse opportunities |
| Safer reuse decisions | Precision of COVERED/REUSE assessments | Avoids false confidence and missing tests |
| Lower duplicate creation | Reduction in tests later judged unnecessary | Reduces long-term maintenance |
| Better traceability | Ability to link requirements, automation source and Rally references | Explains the decision |
| Token efficiency | Model context and cost per accepted QA plan | Makes scaling across repos viable |
| Trustworthy metadata | Visibility of catalog/Rally freshness and unresolved links | Prevents misleading conclusions |
| Better QA workflow | Human acceptance and time required for QA analysis | Measures operational usefulness |

These are **evaluation dimensions**, not predetermined pass thresholds. Thresholds should be set using real historical features and QA-reviewed ground truth.

## Suggested architectural validation journey

### Stage 1 — Validate the problem with one service domain

Select a representative service automation repo and a relevant shared E2E repo. Review recent features for which teams know which tests should have been reused or extended.

**Question:** Can the proposed catalog describe the relevant test inventory without losing key context?

### Stage 2 — Validate reuse-first discovery

Demonstrate that a QA workflow can retrieve relevant candidates without feeding an entire catalog into the model, and can explain why a test does or does not cover an acceptance criterion.

**Question:** Does this reduce duplicate proposals while maintaining safe coverage decisions?

### Stage 3 — Validate Rally traceability

Examine linked Rally TCs with several states: accessible, unavailable, confirmed deleted, no recorded result and historical pass/fail. Ensure these are distinguished from CI runtime execution.

**Question:** Does Rally enrichment improve decisions without creating false certainty?

### Stage 4 — Assess multi-repository viability

Evaluate additional service-level and shared E2E repos for discovery accuracy, query relevance, access boundaries and operating cost.

**Question:** Can this remain lightweight, or is centralized search justified?

## Key risks and mitigations

| Risk | Architecture-level mitigation |
| --- | --- |
| Incomplete test discovery | Explicit catalog completeness and uncertainty |
| Test name/title does not reveal behavior | Source-backed evidence and staged inspection |
| E2E dependencies are not obvious | Cross-repo search scope with future impact graph enrichment |
| Rally missing records caused by permissions | Separate unavailable/unknown from confirmed deletion |
| Historical Rally results are confused with live outcomes | Separate recorded-result and runtime-state concepts |
| Stale catalogs after merges | Pin catalog to source revision and expose freshness |
| Sensitive Rally fields leak through GitHub | Enforce source-specific authorization and field governance |
| LLM overconfidence and duplicate test generation | Reuse-first evaluation and human review for ambiguity |
| GitHub Actions runner unavailable | CI-independent indexing capability and alternate approved runner |

## High-level implementation boundary

This repository is **not** intended to define:
- Framework-specific AST/parser mappings.
- GitHub Actions YAML and authentication configuration.
- JSON schema, test identity algorithms, or example payloads.
- Rally WSAPI request structure, pagination or field-level integration.
- CLI flags, commands, cache tables, data models or source code.
- Operational playbooks and step-by-step delivery tasks.

Such artifacts belong in a later **Detailed Design / Implementation** repository or workstream once the solution architecture is approved.

## Open decisions for architecture review

1. What repository groups should be searchable together for a feature?
2. What minimum evidence allows REUSE vs EXTEND vs CREATE?
3. What is the acceptable freshness of source and Rally metadata?
4. Can Rally-derived metadata be stored inside automation repos under existing access controls?
5. Do source tests always have a single Rally link, or are many-to-one and one-to-many relations common?
6. What does “valid Rally TC” mean in this organization: existence, lifecycle state, or project-specific rules?
7. Are GitHub Actions runners available for all target repos?
8. How will human-labeled historical features be collected for evaluation?
9. When would local retrieval stop meeting multi-repository scale/performance needs?

## Decision requested

Approve the **architecture direction**—per-repository catalogs, independent Rally enrichment, requirement normalization, `gh`-based selective retrieval, and evidence-first reuse decisions—before investing in detailed implementation design.
