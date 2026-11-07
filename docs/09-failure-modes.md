# 09 — Failure Modes and Recovery

The system must preserve previous valid data and communicate uncertainty instead of silently producing an apparently complete catalog.

| Failure | Detection | Required response |
| --- | --- | --- |
| Runner unavailable | Job not scheduled/executed | Signal freshness lag; allow approved alternate CI |
| Project build/discovery fails | Nonzero exit or missing enumeration | Reject snapshot; retain prior |
| Dynamic tests cannot be resolved | Unmapped discovered cases | Publish partial only under explicit policy, flagged INCOMPLETE |
| Adapter crashes on syntax | Parser errors / missing spans | Per-file diagnostics; fail completeness threshold |
| Source paths change | Reconciliation mismatch | Resolve alias or delete/create; do not preserve wrong pointers |
| Shared helper changes | Dependency fingerprints | Invalidate affected metadata or mark unresolved |
| Source main moves while indexing | Head SHA differs from S | Mark superseded, rebuild newer SHA |
| Branch publication races | Optimistic ref conflict | Reload latest branch and validate source ordering |
| Branch protection rejects write | API 403/ruleset error | Fail visibly; seek authorized policy-compliant publisher |
| gh API throttling | Rate-limit response | Cache/backoff; do not claim empty catalog |
| Credentials lack repo scope | 401/403/404 ambiguity | UNAVAILABLE; no data leakage |
| Catalog JSONL corrupt | Hash/schema/line parse error | Reject or ignore bad snapshot |
| Cross-repo cache partly stale | Manifest mismatch | Report per-repo freshness; do not collapse to one green status |
| Search misses E2E behavior | Low Recall@K benchmark | Expand queries and repo scope; avoid automatic NEW |
| Too many candidate records | Result cap exceeded | Rank/filter before tool output |
| LLM evaluates incomplete evidence | Missing obligations or source | REVIEW/UNKNOWN, not confident REUSE or NEW |
| Sensitive literals extracted | Redaction/schema scanning | Reject publication and remediate source/extractor |

## Recovery levels

**Retryable transient:** GitHub API failures, branch CAS conflicts, rate limits. Retry with bounded backoff and SHA checks.

**New-source supersession:** A later main commit exists. Never publish old run as current; queue or rely on the later event to produce a new snapshot.

**Hard failure:** Invalid schema, duplicate IDs, untrusted source path. Do not publish.

**Degraded but observable:** Individual unsupported dynamic declarations may be recorded as unmapped diagnostics if policy permits a partial snapshot. Consumer must treat it as INCOMPLETE.

## Reader fallback

If a fresh snapshot is unavailable, return the last valid snapshot with an explicit stale status and exact source SHA. Do not hide this warning in the prompt; include it in the structured search response. If task risk is high, block conclusive coverage decisions.

## Human review triggers

- Conflicting semantic evidence.
- Requirements missing critical postconditions.
- Source code cannot be inspected due to access policy.
- A proposed deletion/obsolescence of a shared E2E test.
- Discovered and mapped test counts cannot be reconciled.
- Candidate NEW conclusion from an incomplete multi-repo search.

## Incident record

For operational incidents, capture affected repository, source SHA, attempted catalog revision, adapter/schema versions, failure category, whether the prior snapshot is retained, and the corrective action. Avoid logging confidential test data.
