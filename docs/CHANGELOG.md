# Change ledger

Record material repository changes in reverse chronological order. Each entry
must explain the reason and the evidence used to verify the result.

## 2026-09-22 — Workspace bootstrap

- Agent/client: Codex
- Change: Created the portable cross-agent workspace contract, project memory
  documents, decision record template, validation script, and project-scoped MCP
  configurations for Codex, Claude Code, and Antigravity. Pinned the MCP memory
  server as a local npm dependency so agent sandboxes can start it offline, and
  set the project npm registry to HTTPS because the machine default used HTTP.
- Reason: Establish persistent, auditable context and consistent engineering
  rules before product implementation begins.
- Behavior/compatibility impact: No product behavior exists yet. Agents launched
  from this repository can use a common local MCP knowledge graph while relying
  on Git-tracked files as the authoritative record.
- Verification: `./scripts/check.sh`; shell syntax validation; JSON parsing; TOML
  parsing when Python `tomllib` is available.
