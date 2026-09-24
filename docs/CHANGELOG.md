# Change ledger

Record material repository changes in reverse chronological order. Each entry
must explain the reason and the evidence used to verify the result.

## 2026-09-24 — Stack-agnostic project examples

- Agent/client: Codex
- Change: Replaced README project-name examples with `newproject`, removed
  prescribed application technologies, and clarified the distinction between
  the template's Node.js/npm tooling and the application's chosen stack.
- Reason: Make initialization and migration instructions usable across stacks
  while preserving the user's existing README edits.
- Behavior/compatibility impact: Documentation only; existing memory tooling,
  commands, and configuration remain unchanged. Application dependencies and
  verification commands must be supplied for the selected stack.
- Verification: Baseline and final `./scripts/check.sh`; `git diff --check`;
  checked removal of old names and stack assumptions and compared against the
  pre-edit README to verify preservation of the user's initial-memory prompt.

## 2026-09-24 — Existing-project migration and GitHub preparation guide

- Agent/client: Codex
- Change: Added README walkthroughs for checkpointing an existing project,
  switching accounts in place, cloning or copying to a destination, restoring
  ignored MCP memory, verifying a fresh-session handoff, and reviewing a GitHub
  publication. Added navigation alongside the existing new-project instructions.
- Reason: Make account and machine handoffs repeatable without losing recorded
  context or accidentally publishing local memory and credentials.
- Behavior/compatibility impact: Documentation only; existing initialization,
  agent configurations, and memory storage behavior remain unchanged. Agent
  conversation transfer is not guaranteed by the documented migration process.
- Verification: Baseline and final `./scripts/check.sh`; `git diff --check`;
  temporary-directory clone/restore and full-copy checks; ignore-rule checks;
  review of tracked configuration and pattern scan of tracked files and all
  27 historical file blobs found no obvious credentials. Cross-account login
  and remote-machine transfer were not performed.

## 2026-09-23 — OpenCode and Hermes memory setup

- Agent/client: Codex
- Change: Added a project MCP configuration for OpenCode and a reusable Hermes
  MCP entry, with setup and sequential handoff instructions in the README.
- Reason: Make the template usable with the two agents used frequently by the
  project owner while keeping each project's memory isolated.
- Behavior/compatibility impact: OpenCode can connect to project memory after
  `npm ci`; Hermes uses a one-time entry in its user config that follows the
  project from which it is launched. The entry was installed on this machine.
- Verification: `./scripts/check.sh`; `opencode mcp list` connected;
  `hermes mcp test project-memory` connected and discovered nine tools.

## 2026-09-22 — Human-facing template initialization guide

- Agent/client: Codex
- Change: Replaced the brief template README with a complete, step-by-step guide
  for creating a new project from `portable`, using `reactapp` as the example.
- Reason: Let humans repeat the proven initialization process without reconstructing
  commands or relying on an earlier agent conversation.
- Behavior/compatibility impact: No application behavior changed. The guide covers
  safe template copying, dependency installation, document customization, Git
  initialization, first memory creation, verification, and the first application
  setup request.
- Verification: `./scripts/check.sh`.

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
