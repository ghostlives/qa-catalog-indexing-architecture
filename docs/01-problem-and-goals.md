# 01 — Problem and Goals

## Context

The application has many services and user interfaces maintained by different teams. Tests are spread across service-specific automation repositories and shared E2E repositories. A service change can affect a test in another repository, especially when the test covers an end-to-end business flow.

Repositories may contain more than 1,000 TCs each. Automated tests are linked to Rally TCs. Requirements come from SpecKit, custom software design documents (SDD) and other sources.

QA needs a reliable answer to a practical question:

**Which existing tests should be reused or extended for this change, and which behavior still needs a test?**

## Where the current process breaks down

### Tests are distributed

Test names, tags and folder structures do not consistently identify business behavior. An E2E test can cross several services without listing them explicitly. Searching only the repository owned by the team making the change can miss relevant coverage.

### Existing coverage is difficult to verify

Finding a test with a similar name is not enough. The test might use a different setup, assert only part of the requirement or mock a dependency that the new feature needs to exercise.

The reverse is also true: a test with an unrelated title may already verify the required behavior.

### Requirements are not written in one format

SpecKit and custom SDD documents describe rules and acceptance criteria differently. QA must extract conditions and expected outcomes without losing business meaning or relying on a particular file structure.

### Rally data can be misunderstood

A linked Rally TC might no longer be visible. That does not establish that the record was deleted. For accessible records, Rally may hold a last recorded result that came from an earlier manual or automated run.

Rally is useful for traceability; it cannot replace source-level checks or current CI execution evidence.

### Full-catalog prompting does not scale

Passing thousands of TC records to an LLM for every feature increases token use and still leaves the model to search an oversized context. Routine discovery is better handled by an index and ordinary search tools.

### Search results depend on the repositories included

A good search over the wrong repositories is still incomplete. QA needs an explicit, maintainable mapping of services and business flows to automation repositories, including shared E2E suites.

## Example: cancellation of shipped orders

Suppose a feature changes the behavior when a shipped order is cancelled. QA needs to verify that:

- The cancellation request is rejected.
- The order remains in its previous state.
- No refund is initiated.
- The relevant customer journey still behaves correctly.

One integration test might check the response, another might verify the saved state, and a shared E2E test might check the journey. These tests have different purposes; they should not be considered duplicates simply because they cover the same feature.

The useful outcome is a QA plan that identifies those existing tests, explains what they cover, and calls out any remaining gap.

## Goals

| Goal | Expected outcome |
| --- | --- |
| Find existing tests | Search across relevant service, frontend and E2E repositories |
| Reuse where appropriate | Avoid additional tests when existing ones provide the required coverage |
| Identify genuine gaps | Compare expected behavior with actual setup and assertions |
| Support different specifications | Interpret requirements consistently while retaining references to their sources |
| Keep model cost controlled | Index and shortlist tests without using an LLM |
| Use Rally correctly | Show link status and recorded results without treating them as live execution state |
| Make decisions reviewable | Record which requirements, repositories, tests and evidence led to the QA recommendation |

## Boundaries

The method does not replace Rally, move test code between repositories, require teams to adopt one test framework, or make test creation fully autonomous.

It also does not treat a failed lookup as a deleted Rally TC, a Pass in Rally as a current test pass, or an empty search as proof that no existing test covers a requirement.

Unclear requirements, missing repository access and incomplete test evidence require QA review rather than a confident coverage decision.
