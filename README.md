# Portable

Portable is a software project workspace prepared for handoffs between Codex,
Claude Code, Antigravity, and other MCP-capable coding agents.

## Start here

1. Read `AGENTS.md` and `docs/PROJECT_STATE.md`.
2. Run `./scripts/check.sh` before changing code.
3. Use one coding agent at a time when the shared MCP memory is enabled.
4. Run `./scripts/check.sh` again before finishing.
5. Record every material change and its reason in `docs/CHANGELOG.md`.

## Shared memory

The project uses two complementary forms of memory:

- Files under `docs/` are the authoritative, reviewable record committed to Git.
- The local MCP knowledge graph provides convenient cross-session recall.

The MCP server stores its local state in `.agent-memory/memory.jsonl`. That file
is intentionally excluded from Git because it is generated state and may contain
conversation-derived information. Durable facts and decisions must also be
written to the appropriate file under `docs/`.

Project-scoped MCP configuration is provided for:

- Claude Code: `.mcp.json`
- Codex: `.codex/config.toml`
- Antigravity: `.agents/mcp_config.json`

Each client starts the same pinned MCP server through
`scripts/memory-mcp.sh`. Launch clients from this repository root. The first
checkout requires one dependency installation:

```bash
npm ci
```

After that installation, starting the memory server requires no network access.

## Verification

```bash
./scripts/check.sh
```

As the technology stack is added, extend `scripts/check.sh` so this remains the
single command that runs every required test, lint, typecheck, and build check.
