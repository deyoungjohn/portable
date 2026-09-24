# Portable

Portable is a reusable software-project template for handoffs between Codex,
Claude Code, Antigravity, OpenCode, Hermes, and other MCP-capable coding agents.

The project's language, framework, and build tools are yours to choose. Node.js
and npm are required for the bundled MCP memory server and workspace checks;
they do not determine the application's stack.

It combines two forms of project memory:

- Files under `docs/` are the authoritative, reviewable record committed to Git.
- A local MCP knowledge graph provides convenient recall across agents and
  sessions.

Start with the section that matches your task:

- [Create a new project](#1-copy-the-template-without-generated-state)
- [Migrate an existing project](#migrate-an-existing-project)
- [Prepare this folder for GitHub](#prepare-this-folder-for-github)

The new-project instructions below create a project named `newproject` under
`/home/dell/Projects`. Replace `newproject` with your desired project name when
creating another project.

You do not need to create `.agent-memory/memory.jsonl` yourself. The MCP server
creates it automatically the first time an agent writes a memory. If no memory
has been stored yet, the absence of that file is normal.

## 1. Copy the template without generated state

```bash
cd /home/dell/Projects

mkdir newproject

rsync -a \
  --exclude='.git/' \
  --exclude='node_modules/' \
  --exclude='.agent-memory/memory.jsonl' \
  portable/ newproject/
```

This copies the reusable configuration while excluding:

- The template's Git history
- Installed npm dependencies
- The template's local MCP memory

## 2. Enter the new project

```bash
cd /home/dell/Projects/newproject
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
npm pkg set name=newproject
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

Not yet defined. Requirements will be established with the user before implementation.

## Current status

- Cross-agent workspace initialized.
- Application implementation has not started.
- Product requirements remain to be defined.

## Architecture

- Language and framework: To be selected based on project requirements
- Build tooling: To be selected
- Test framework: To be selected
```

Add a new entry at the top of `docs/CHANGELOG.md`:

```markdown
## YYYY-MM-DD — newproject initialization

- Agent/client: Human
- Change: Created newproject from the portable cross-agent template.
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
check. Add your chosen stack's verification commands directly to this script;
the template's optional npm script checks do not run other stacks' checks
automatically.

## 8. Create the initial commit

```bash
git add .
git commit -m "chore: initialize newproject workspace" \
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
Read AGENTS.md and docs/PROJECT_STATE.md

Initialize the shared project memory. Create a project entity named newproject containing these observations:

- This project was created from "Portable", a framework for building projects with persistent memory and context across agents.
- Git-tracked documentation is the authoritative project memory.
- MCP memory is supplementary.
- Run ./scripts/check.sh before and after every material change.

Do not modify application code yet. Confirm that the memory can be retrieved.
```

When the agent calls the MCP memory tool, this file is created automatically:

```text
/home/dell/Projects/newproject/.agent-memory/memory.jsonl
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

## 11. Begin project implementation

In the next agent request, use a prompt such as:

```text
Read the project instructions and shared memory. Establish the project objective,
requirements, and acceptance criteria with me, then select an appropriate stack
or use the stack already specified. Record these in the project documents before
implementation. Scaffold the project using the agreed stack. Preserve the existing
cross-agent configuration and MCP memory dependency, add the stack's required
checks to scripts/check.sh, document architecture decisions, run all checks, and
update the change ledger.
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

## Migrate an existing project

Use this walkthrough when continuing an existing portable-based project with
another account, agent, or computer. The examples use `newproject`; substitute
your project's name and paths. These steps preserve recorded project context.
They do not guarantee transfer of an agent's chat history or built-in memory.

Keep the existing project's code, Git history, documents, and MCP memory. Do not
repeat the new-project template copy, package rename, Git initialization, or
initial-memory creation steps. Do not copy the template over an existing project.

### 1. Record a handoff before leaving the current session

Do this at milestones and before the current account's quota runs out. Ask the
outgoing agent:

```text
Prepare this project for a handoff to a fresh agent session.

Follow AGENTS.md. Update docs/PROJECT_STATE.md, relevant specifications and
decisions, and docs/CHANGELOG.md with the actual state of the project.

Create or update docs/HANDOFF.md with:
- The objective, constraints, and acceptance criteria, or links to them.
- Completed milestones and the active milestone.
- Current branch and starting commit; describe any uncommitted work.
- Unfinished work, known failures, blockers, and unresolved questions.
- Important decisions and rejected approaches, with reasons or document links.
- Setup commands, required tool versions, and required environment variable
  names, without secret values.
- Assets or local data that are not in Git and how to restore them.
- Checks run, their results, and the exact next task.

Save durable, non-secret project facts in shared MCP memory as well.
Run ./scripts/check.sh and report any failures honestly.
Do not publish anything or change application behavior during this handoff.
```

Keep detailed explanations in the relevant documents and link to them from the
handoff. Context that only exists in a conversation may be unavailable later.

### 2. Review and commit the checkpoint

From the existing project root:

```bash
cd /home/dell/Projects/newproject
git branch --show-current
git status --short
git diff
./scripts/check.sh
```

Review new files too; `git diff` does not show untracked file contents. Resolve
unexpected failures, or record existing failures and unfinished work explicitly.
Save work from any other active branches or worktrees that must be migrated.
If the branch command is empty, create a named branch for the detached work
before proceeding.

Stage only reviewed files. If all pending changes belong to this checkpoint:

```bash
git add .
git diff --cached --stat
git diff --cached
git commit -m "docs: checkpoint project for handoff"
git rev-parse HEAD
git status --short
```

Use a commit message that describes the changes if the checkpoint includes
application code. Skip the commit if there is nothing to commit. Record the final
commit ID outside the commit itself so you can compare it at the destination.
Do not discard unrelated work to obtain a clean status.

### 3. Stop the outgoing agent and back up local memory

Wait for the final memory write, close the agent, and ensure its project-memory
MCP process has exited. Keep it stopped during backup and account switching.
Two MCP processes must not write the same memory file concurrently.

The memory file is ignored by Git. A commit, push, or clone will not include it.
Create a new private backup directory outside the repository; choose a fresh
suffix for each handoff:

```bash
mkdir -p /home/dell/Backups
mkdir -m 700 /home/dell/Backups/newproject-handoff-001
```

If `.agent-memory/memory.jsonl` exists, copy and verify it:

```bash
cp .agent-memory/memory.jsonl /home/dell/Backups/newproject-handoff-001/memory.jsonl
cmp .agent-memory/memory.jsonl /home/dell/Backups/newproject-handoff-001/memory.jsonl
```

No output from `cmp` with exit status zero means the files match. If no memory
has ever been written, the missing file is normal. If you expected memory and
the file is missing, locate the actual project memory before proceeding.

Back up required ignored assets and local data separately, following the
inventory in `docs/HANDOFF.md`. Stop applications before copying active databases
or use their supported backup commands. Keep credentials in a secure credential
store and recreate them at the destination; do not add them to Git or MCP memory.

### 4. Choose how to continue

**Same computer, same project folder:** no file transfer is needed. Keep the
folder, including `.git/` and `.agent-memory/`, and go to step 6.

**Another computer using Git:** push the checkpoint to your existing remote. For
example, if the project branch is `main` and the remote is `origin`:

```bash
git push -u origin main
```

Replace the branch and remote names with the actual values. Push any additional
branches or tags needed for the handoff explicitly. Then, on the destination:

```bash
mkdir -p /home/dell/Projects
cd /home/dell/Projects
git clone --branch main YOUR_REPOSITORY_URL newproject
cd newproject
git rev-parse HEAD
git status --short
```

Replace `YOUR_REPOSITORY_URL` and adjust the destination path for that computer.
Use a new destination folder and compare the commit ID with step 2. Transfer the
private memory backup through a secure channel such as SSH, then continue with
step 5. Git cloning does not restore ignored files or uncommitted work.

**Another folder or a privately mounted destination using a full copy:** with
the agents stopped, copy into a new directory. For example, on the same machine:

```bash
mkdir /home/dell/Projects/newproject-migrated
rsync -a --exclude='node_modules/' \
  /home/dell/Projects/newproject/ /home/dell/Projects/newproject-migrated/
cd /home/dell/Projects/newproject-migrated
git rev-parse HEAD
git status --short
cmp /home/dell/Projects/newproject/.agent-memory/memory.jsonl .agent-memory/memory.jsonl
```

Skip `cmp` if there was no memory file. This copy retains `.git/`, local branches,
uncommitted files, and MCP memory. It also copies ignored files such as `.env`,
so use a private destination and never upload the folder as a public archive.
For a project using linked Git worktrees or submodules, prefer Git cloning and
their normal setup commands: copied Git metadata can refer to the original
location. A full copy already contains memory, so continue with step 6.

### 5. Restore memory after a Git clone

On the destination, with no agent running, place the transferred backup in a
private directory. From the cloned project root, run:

```bash
mkdir -p .agent-memory
test ! -e .agent-memory/memory.jsonl && \
  cp /path/to/private-backup/memory.jsonl .agent-memory/memory.jsonl
cmp /path/to/private-backup/memory.jsonl .agent-memory/memory.jsonl
git check-ignore -v .agent-memory/memory.jsonl
```

Replace `/path/to/private-backup` with the actual location. The first command
pair refuses to overwrite existing memory. If memory already exists, stop and
determine which file belongs to this checkpoint; do not blindly combine JSONL
files. Skip restoration if the source never had memory.

Restore any required local data from the handoff inventory. Do not reuse the
new-project copy command: it deliberately excludes existing memory.

### 6. Restore the environment and sign into the intended account

From the destination project root, install the pinned MCP memory dependency if
this is a new clone or copy:

```bash
npm ci
```

Install the application's dependencies using its own package manager and the
tool versions and setup commands recorded in the handoff. Then run:

```bash
./scripts/check.sh
```

On an unchanged installation in the same folder, dependency reinstallation is
unnecessary. Project configuration files travel with the repository, but
user-level settings, globally installed skills, and external service logins may
need setup again. Hermes requires the user-level entry described in the
[Hermes setup section](#9-set-up-hermes-once-if-you-use-it).

For a Codex CLI account switch, after closing existing sessions:

```bash
codex logout
codex login
codex login status
```

Select the intended ChatGPT account in the browser login flow. ChatGPT login
uses that account's available plan access; API-key usage is billed separately.
The CLI and IDE extension share cached authentication. For app users, use the
app's sign-out/sign-in controls and reopen the existing project folder. See the
[official authentication documentation](https://learn.chatgpt.com/docs/auth).

Authenticate external services as needed. Do not copy account credentials into
the repository. These steps restore project files and recorded context; they do
not depend on cross-account conversation transfer.

### 7. Verify the handoff in a fresh session

Start one agent from the project root. For Codex:

```bash
codex
```

Use this prompt:

```text
Read AGENTS.md, docs/PROJECT_STATE.md, docs/HANDOFF.md, relevant specifications,
decisions, and recent changelog entries. Search shared MCP memory for this
project and report whether the expected project facts are available.

Inspect the current branch, commit, and uncommitted changes. Run
./scripts/check.sh and record any existing failures.

Before changing code, summarize the objective, completed milestones,
constraints, outstanding work, and the exact next task. Flag missing context
or contradictions rather than guessing.
```

Compare that summary with the handoff and confirm the branch and commit match.
If MCP memory cannot be retrieved, check dependency installation, project trust,
and the client's MCP configuration. Do not assume an empty graph is a successful
restore. Once the handoff is verified, ask the agent to continue the next task.

Keep the source and backup until the destination is verified. Repeat this
checkpoint process at milestones so continuity does not depend on one account
or a single long conversation. OpenAI's
[long-horizon guidance](https://developers.openai.com/blog/run-long-horizon-tasks-with-codex)
also recommends durable specifications, plans, and status files.

## Prepare this folder for GitHub

Commit the template's documentation, scripts, lockfile, and project configuration,
including `.codex/config.toml`, `.mcp.json`, `.agents/`, and `.npmrc`. The supplied
configurations contain local launch settings and the public npm registry, not
account credentials. Review any changes you make to those files before publishing.

The supplied `.gitignore` excludes MCP memory, `node_modules/`, `.env` files
(except `.env.example`), `*.local`, and common build outputs. The memory README
is intentionally tracked. A safe `.env.example` contains placeholders only.
Git itself manages `.git/`; do not upload it as a separate folder or archive.

### 1. Review what will be published

```bash
git status --short --untracked-files=all
git status --short --ignored
git ls-files
git ls-files -ci --exclude-standard
git check-ignore -v .agent-memory/memory.jsonl node_modules/ .env
```

`git ls-files -ci --exclude-standard` should produce no output: it detects
already-tracked files now covered by ignore rules. Ignore rules do not remove
previously committed files or erase them from history. Review the repository's
history as well as its current files before making it public.

Never force-add ignored memory or secrets. The ignore list is not a complete
secret detector: credential files with other names may still be included.
Remember that a public repository exposes documentation and Git author metadata
too. Choose repository visibility to suit the project's contents.

### 2. Run checks and review the staged changes

```bash
./scripts/check.sh
git add .
git diff --cached --stat
git diff --cached
git status --short
```

Check that only intended files are staged. The project checks validate the
workspace configuration; they are not a credential scanner.

### 3. Commit, then publish to your chosen repository

For these migration-guide changes, for example:

```bash
git commit -m "docs: explain project migration and GitHub preparation"
```

Skip the commit if there are no staged changes. Create the GitHub repository
with your chosen visibility, connect its remote if one is not already
configured, and push the intended branch. Keep the private memory backup
separate; the GitHub repository alone is not a complete memory backup.
