# 01 — Problem Statement and Goals

## Business context

A large enterprise platform includes many microservices and frontend applications, each potentially maintained by different Agile teams. Automation responsibilities are distributed:

- Dedicated service-level automation repositories contain unit, integration, API and contract tests.
- Shared automation repositories contain end-to-end tests spanning multiple backend and frontend services.
- A repository can contain more than 1,000 test cases, and the total number grows substantially across the estate.
- Test cases in automation repositories are expected to link to Rally Test Cases.
- Feature specifications vary between SpecKit, custom SDD and other document conventions.

In this environment, the hardest QA question is not **“How do we write more tests?”** It is **“Which existing tests already verify the behavior being changed?”**

## Core problems

### P1 — Fragmented test knowledge

There is no reliable, consolidated view of which test definitions exist, where they live, what signals they assert, or which business journeys they touch.

**Impact:** high investigation effort, incomplete regression scope and hidden E2E dependencies.

### P2 — Unnecessary test creation

AI-generated test plans can propose tests based on requirement descriptions without properly discovering existing automation. Test names and superficial similarity are insufficient to establish overlap.

**Impact:** duplicated coverage, maintenance cost, inconsistent assertions and test suite growth.

### P3 — Heterogeneous requirement artifacts

Development teams may use different specification frameworks. QA planning cannot assume a single document structure or a consistent representation of acceptance criteria and business rules.

**Impact:** ambiguous plans, missed scenarios and repeated manual requirement interpretation.

### P4 — High token and runtime costs

Providing thousands of catalog entries or source files to an LLM on every feature request is costly and often reduces retrieval precision.

**Impact:** excessive context consumption, latency and unpredictable agent behavior.

### P5 — Rally traceability does not equal live test state

Rally-linked Test Cases may have been deleted, moved, or made inaccessible. When present, Rally may expose a last recorded test result that can be older than the current automation execution.

**Impact:** incorrect conclusions about test validity or test health if code state, Rally existence and historical results are conflated.

## Problem examples

| Situation | Wrong assumption | Correct architectural need |
| --- | --- | --- |
| An E2E test does not mention an affected service in its title | It is unrelated | Cross-repo candidate discovery and source-backed verification |
| A new acceptance criterion resembles an existing test name | Already covered | Evaluate preconditions, assertions and expected outcomes |
| A Rally ID returns no accessible record | The linked TC was deleted | Distinguish missing, inaccessible and confirmed-deleted |
| Rally's stored last result is Pass | The test passes right now | Keep recorded and runtime states separate |
| A service repo contains 1,500 tests | Send all 1,500 to the model | Filter outside the model; inspect only relevant evidence |

## Desired outcomes

1. QA can identify and justify reuse of existing tests across repositories.
2. New or extended test cases are proposed only for verified or explicitly reviewed coverage gaps.
3. QA analysis can start from different requirements formats through a common conceptual model.
4. Catalog indexing and routine discovery do not consume LLM tokens.
5. Rally linkage and historical result metadata improve traceability without corrupting runtime status semantics.
6. Decisions remain auditable: where evidence came from, its freshness and what is still unknown.

## Non-goals

- Replacing Rally or the existing test automation frameworks.
- Rewriting automation repositories or introducing new testing conventions enterprise-wide at once.
- Automatically deleting automation because a linked Rally case appears missing.
- Treating a search score or Rally verdict as proof of requirement coverage.
- Designing CI YAML, data schemas, parsers, CLI command syntax or an MCP service in this document.

## Stakeholders

QA leads own evaluation policy and approve ambiguous decisions; automation teams own source tests; engineering teams own service changes; platform/DevOps teams provide authorized CI and GitHub access; Rally administrators define permitted visibility and interpretation of Rally records.
