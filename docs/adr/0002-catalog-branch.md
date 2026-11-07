# ADR-0002: Versioned snapshot on a separate qa-catalog branch

- **Status:** Proposed
- **Context:** Generated catalogs should not pollute feature branch history or require model-time checkout.
- **Decision:** Publish manifest, JSONL and diagnostics as atomic snapshots to a dedicated qa-catalog branch in the same automation repository.
- **Benefits:** Versioning, auditable provenance, source repository ownership, simple gh-based access, no extra database for the PoC.
- **Trade-offs:** Need branch write rights, consistency checks, manifest caching; full snapshots may grow Git history.
- **Alternatives:** Commit catalog to main; short-retention GHA artifacts; separate catalog repo; central object storage/database.
- **Why not main:** Extra generated commits, possible branch protection/CI issues and irrelevant diffs.
- **Invariants:** manifest.source_commit_sha identifies exact source; older publish cannot replace newer as CURRENT; failure retains last valid.
- **Revisit when:** Branch size, permissions or cross-repo query latency make separate storage more economical.
