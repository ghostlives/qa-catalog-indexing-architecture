# 01 — Problem and Goals

## Current environment

The application landscape includes many backend microservices and frontend applications. Different teams own different parts of the system, but their changes often affect the same business flows.

Automation tests are organized in several ways:

- A service may have its own test repository.
- Some repositories contain more than 1,000 TCs.
- Shared E2E repositories test flows involving multiple services and UIs.
- Automated TCs are linked to Rally Test Cases.
- Feature requirements can come from SpecKit, custom SDD templates or other documents.

A change in one service can therefore affect tests in several repositories. Finding those tests is one of the most time-consuming parts of QA planning.

## Problems to solve

### 1. Existing tests are hard to find

Test names, folders and tags do not always describe what a test verifies. A shared E2E test might exercise Order, Payment and Notification services through a single user journey.

**Example:** A change to order cancellation may already be covered by an E2E test named `CustomerCheckoutFlow`. Searching only the Order automation repository—or only for "cancel"—could miss it.

### 2. New test plans can duplicate existing coverage

A model may generate a reasonable test scenario without knowing that the same behavior is already checked elsewhere. Adding another test is not always useful. It can increase execution time and maintenance work without improving coverage.

The opposite mistake also matters: a similar test name is not enough to conclude that the required behavior is covered.

### 3. Requirements arrive in different formats

SpecKit and custom SDD workflows describe requirements differently. Some documents include detailed acceptance criteria; others focus on system behavior or design changes.

QA needs to identify the expected behavior, conditions and outcomes without assuming that every team uses the same template.

### 4. Large catalogs do not belong in the model context

Sending thousands of TC descriptions and source files to an LLM for each feature would consume unnecessary tokens. It would also make it harder to focus on the few tests that matter.

Search should narrow the candidates before the model reads test details.

### 5. Rally links and results can be misleading

A test may still exist in source code even when its linked Rally TC cannot be found. A failed lookup does not tell us whether the TC was deleted or the user lacks access.

For an existing Rally TC, the latest recorded result may be useful context, but it may come from an earlier manual or automated execution. It cannot be treated as the status of the current CI run.

## Example: order cancellation change

Suppose a feature changes the behavior for cancelling shipped orders. QA must check whether existing tests cover:

- The cancellation request being rejected.
- The order state remaining unchanged.
- No refund being issued.
- The behavior through the customer-facing E2E flow, where applicable.

One service-level test may verify the HTTP response, another may check the order state, and a shared E2E test may cover the customer journey. QA should inspect these tests before deciding whether to extend an existing test or create a new one.

Rally may help find the linked TCs and their prior results. It does not replace inspection of actual assertions and setup.

## Goals

| Goal | What success looks like |
| --- | --- |
| Find existing tests | QA can locate relevant service and shared E2E tests across repositories |
| Reduce duplicate tests | New TCs are proposed only after existing coverage has been reviewed |
| Support different spec formats | QA can work from the behavior described in SpecKit or custom SDD documents |
| Control model usage | Catalog creation and initial search do not require LLM calls |
| Improve Rally visibility | Missing links and last recorded results are shown with clear status and dates |
| Explain decisions | A recommendation references the requirements, tests and information used |

## Outside the scope

The solution does not replace Rally, change the automation frameworks, or mandate a new TC naming convention. It also does not define CI configuration, indexing code, Rally API requests or a complete test-management system.

Test coverage decisions remain subject to QA review, especially for unclear requirements, complex E2E flows or incomplete search results.
