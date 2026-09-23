# Project state

Last updated: 2026-09-23

## Objective

Define the product objective before implementation begins.

## Current status

- The cross-agent workspace has been initialized.
- Product requirements, technology stack, and implementation milestones remain
  to be defined.
- No product code exists yet.

## Architecture

No product architecture has been selected.

The workspace itself uses Git-tracked documents as authoritative project memory
and a local MCP knowledge graph for convenient recall across supported agents.
OpenCode has a project MCP configuration. Hermes uses one user-level MCP entry
that follows the active portable project's workspace directory.

## Required verification

Run:

```bash
./scripts/check.sh
```

Extend that script when the project stack is chosen. It must remain the single
entry point for all required checks.

## Known limitations

- The local MCP memory package should not be written by multiple agent processes
  concurrently. Close one agent and its MCP process before opening the next.
- MCP memory is supplementary and does not replace tests, Git history, decision
  records, specifications, or the change ledger.
