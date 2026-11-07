# ADR-0003: GitHub CLI for catalog access, no MCP dependency

- **Status:** Proposed
- **Context:** QA agents already use shell tools in local development and can authenticate via gh.
- **Decision:** Use gh CLI/GitHub API as transport for manifest, catalog and pinned source evidence; implement deterministic search outside LLM context.
- **Benefits:** No MCP server to deploy, compatible with local coding assistants, no clone/checkout needed for catalog files.
- **Trade-offs:** gh auth, rate limits, local cache and access-control responsibilities remain; raw API content must be filtered before model context.
- **Alternatives:** Mandatory MCP server; direct full catalog prompt; full checkout of every qa-catalog branch.
- **Why alternatives were not chosen:** Additional service burden or uncontrolled prompt/token costs.
- **Consequences:** Define a logical CLI reader interface of sync, search, inspect, related, status; defer implementation.
- **Revisit when:** Centralized agents or remote workers need common always-on search and fine-grained tenancy.
