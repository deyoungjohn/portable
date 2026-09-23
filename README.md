# Portable

Portable is a reusable software-project template for handoffs between Codex,
Claude Code, Antigravity, OpenCode, Hermes, and other MCP-capable coding agents.

It combines two forms of project memory:

- Files under `docs/` are the authoritative, reviewable record committed to Git.
- A local MCP knowledge graph provides convenient recall across agents and
  sessions.

The instructions below show how to create a project named `reactapp` under
`/home/dell/Projects`. Replace `reactapp` with your desired project name when
creating another project.

You do not need to create `.agent-memory/memory.jsonl` yourself. The MCP server
creates it automatically the first time an agent writes a memory. If no memory
has been stored yet, the absence of that file is normal.

## 1. Copy the template without generated state

```bash
cd /home/dell/Projects

mkdir reactapp

rsync -a \
  --exclude='.git/' \
  --exclude='node_modules/' \
  --exclude='.agent-memory/memory.jsonl' \
  portable/ reactapp/
```

This copies the reusable configuration while excluding:

- The template's Git history
- Installed npm dependencies
- The template's local MCP memory

## 2. Enter the new project

```bash
cd /home/dell/Projects/reactapp
```

Confirm that the important files exist:

```bash
ls -la
ls -la .agent-memory .agents .codex docs scripts
ls -la opencode.json docs/hermes-mcp.yaml
```

The `.agent-memory` directory already exists because its README is part of the
template. `memory.jsonl` should not exist yet.

## 3. Rename the npm package

```bash
npm pkg set name=reactapp
npm pkg set version=0.1.0
npm install --package-lock-only
```

This updates both `package.json` and the root project metadata in
`package-lock.json`.

## 4. Install the pinned MCP dependency

```bash
npm ci
```

This installs the memory server under `node_modules/`. Future MCP launches do
not need to download it again.

## 5. Customize the project documents

Open the project documents in your preferred editor. For example:

```bash
nano README.md
nano docs/PROJECT_STATE.md
nano docs/CHANGELOG.md
```

In `docs/PROJECT_STATE.md`, replace the placeholder objective with something
like:

```markdown
## Objective

Build a production-ready React application.

## Current status

- Cross-agent workspace initialized.
- React application has not been scaffolded.
- Product requirements remain to be defined.

## Architecture

- Frontend: React with TypeScript
- Build tooling: Vite
- Test framework: To be selected
```

Add a new entry at the top of `docs/CHANGELOG.md`:

```markdown
## YYYY-MM-DD — Reactapp initialization

- Agent/client: Human
- Change: Created reactapp from the portable cross-agent template.
- Reason: Establish isolated project memory, agent instructions, and change tracking.
- Behavior/compatibility impact: No application behavior exists yet.
- Verification: `./scripts/check.sh`.
```

Replace `YYYY-MM-DD` with the actual date.

## 6. Initialize Git

```bash
git init -b main
```

Verify your Git identity:

```bash
git config --global --get user.name
git config --global --get user.email
```

If either value is empty, configure it:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-verified-email@example.com"
```

Use an email address verified by your Git hosting provider. If you use GitHub
and prefer not to expose your personal email, use the private `noreply` address
shown under GitHub **Settings → Emails**.

If you need different identities for personal and work repositories, omit
`--global` and configure the identity inside each repository.

## 7. Run the baseline checks

```bash
./scripts/check.sh
```

Expected result:

```text
All configured project checks passed.
```

As the project gains application code, extend `scripts/check.sh` so it remains
the single command that runs every required test, lint, typecheck, and build
check.

## 8. Create the initial commit

```bash
git add .
git commit -m "chore: initialize reactapp workspace" \
  -m "Create the project from the portable cross-agent template with isolated MCP memory, agent instructions, decision records, and change tracking." \
  -m "Checks: ./scripts/check.sh"
```

Optionally tag the clean baseline:

```bash
git tag baseline-0
```

## 9. Set up Hermes once, if you use it

OpenCode reads `opencode.json` and `AGENTS.md` from each project automatically.
This configuration targets the installed OpenCode 1.x format. If you later
upgrade to OpenCode 2.x, move `project-memory` under `mcp.servers` and remove
the `enabled` field, following the current OpenCode MCP documentation.
Hermes reads `AGENTS.md` from the project, but its MCP servers are configured in
`~/.hermes/config.yaml`. To connect Hermes to the memory of whichever portable
project you open, add the following entry under the existing `mcp_servers:` key
in that file. The same entry works for future projects copied from this template:

```yaml
mcp_servers:
  project-memory:
    command: bash
    args: ["scripts/memory-mcp.sh"]
```

If `mcp_servers:` already exists, add only the indented `project-memory` entry;
do not add a second `mcp_servers:` key. The same entry is saved in
`docs/hermes-mcp.yaml` for reference. Start Hermes from the root of the project
you want it to use; the relative script path then follows that project. Run
`hermes mcp test project-memory` from that project to check the connection.
On this machine, the Hermes entry has already been added; future projects only
need `npm ci` and a launch from their project root.

## 10. Start the first agent

For Codex:

```bash
codex
```

For Claude Code:

```bash
claude
```

For OpenCode:

```bash
opencode
```

For Hermes, after the one-time setup above:

```bash
hermes chat
```

Approve or trust the project configuration if prompted.

Then give the agent this initial prompt:

```text
Read AGENTS.md and docs/PROJECT_STATE.md.

Initialize the shared project memory. Create a project entity named
"reactapp" containing these observations:

- This project was created from the portable cross-agent template.
- Git-tracked documentation is the authoritative project memory.
- MCP memory is supplementary.
- The intended frontend stack is React with TypeScript and Vite.
- Run ./scripts/check.sh before and after every material change.

Do not modify application code yet. Confirm that the memory can be retrieved.
```

When the agent calls the MCP memory tool, this file is created automatically:

```text
/home/dell/Projects/reactapp/.agent-memory/memory.jsonl
```

Verify that it exists:

```bash
ls -lh .agent-memory/memory.jsonl
```

The file remains excluded from Git. It may contain conversation-derived
information, so do not store credentials, tokens, or other secrets in project
memory. Durable facts and decisions must also be written to the appropriate
versioned document under `docs/`.

You can work in Codex, close its session, and then open the same project in
OpenCode or Hermes. Memory remains in the project's `memory.jsonl` file. Wait
until the first agent has finished its final memory write and its MCP process
has exited before opening the next. Do not keep two memory-enabled sessions
open against the same project: the underlying file does not safely support
concurrent writers.

## 11. Begin the actual React setup

In the next agent request, use a prompt such as:

```text
Read the project instructions and shared memory. Scaffold a React TypeScript
application using Vite in this repository. Preserve the existing cross-agent
configuration, update the verification script, document the architecture
decision, run all checks, and update the change ledger.
```

## Initialization lifecycle

```text
Copy template
    ↓
Install dependencies
    ↓
Customize project documents
    ↓
Initialize Git and commit baseline
    ↓
Launch first agent
    ↓
First MCP memory write creates memory.jsonl
    ↓
Begin application development
```

Each project receives its own isolated `memory.jsonl`; you never need to create
that file manually.
