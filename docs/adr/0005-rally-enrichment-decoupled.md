# ADR-0005: Rally enrichment is a separate, versioned observation stream

- **Status:** Proposed
- **Context:** Every automation TC has a Rally link. Rally Test Cases may disappear or become inaccessible, and their last recorded results can change without any merge to the automation repository.
- **Decision:** Index Rally references deterministically during source indexing, then resolve Rally case existence and last recorded result via a separate, read-only WSAPI enrichment job. Store the results as a separately versioned overlay in qa-catalog if permissions permit.
- **Rationale:** Code freshness is tied to source SHA; Rally freshness is tied to `fetched_at` and workspace/query scope. Combining them in one snapshot would either leave Rally metadata stale indefinitely or repeatedly rebuild unchanged source indexes.
- **Schema consequences:** Static `rally_links[]`; overlay `rally-manifest.json` + `rally-enrichment.jsonl`; distinguish `FOUND`, `NOT_FOUND_UNCONFIRMED`, `DELETED_CONFIRMED`, `ACCESS_DENIED`, `LOOKUP_ERROR`, `UNKNOWN`.
- **Historical result semantics:** Rally LastVerdict/LastResult refers to stored Rally outcomes, not the actual CI/runtime automation status. Preserve result date, fetched time, source field, optional result ObjectID and origin.
- **Safety:** No 404-is-deleted inference; no auto-removal of test code based on Rally existence/result. Rally credentials never enter Git. Apply authorization before sharing metadata.
- **Alternatives:** Enrich only on main merge; call Rally on every QA request; store Rally fields directly in tests.jsonl.
- **Why not:** Stale outcomes between merges, runtime latency/API costs, and conflated version semantics.
- **Revisit when:** Tenant-specific Rally semantics, access controls, or API limits make branch overlays inappropriate.
