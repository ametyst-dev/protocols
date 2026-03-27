---
description: Step-by-step onboarding skill that connects a new team member's agent to the Ametyst ecosystem. The admin has already created Slack and Notion apps — this skill guides the member through token configuration, MCP setup, protocol download, knowledge base sync, self-context creation, and CLAUDE.md generation. Use whenever a new person joins Ametyst and needs to set up their agent, or when someone says "set up agent", "onboard", "connect to Ametyst".
allowed-tools: Read, Write, Edit, Glob, Bash, AskUserQuestion, mcp__notionApi__API-query-data-source, mcp__notionApi__API-get-block-children, mcp__notionApi__API-retrieve-a-page, mcp__slack__slack_post_message, mcp__slack__slack_get_channel_history
---

## set-up-agent

Guide a new team member through connecting their agent to the Ametyst ecosystem. Supports two paths:
1. **Full setup (from scratch)** — configures Slack + Notion + protocols + context + CLAUDE.md
2. **Upgrade from light setup** — the member already has Slack + GitHub protocols; this adds only Notion access

By the end, they have a fully configured working directory with protocols, company context, self-context, and a CLAUDE.md that keeps everything in sync.

**Important:** This skill must run on **Claude Code** (CLI). The user's teamspace is **General** — do not ask which teamspace they belong to.

**Prerequisite:** The admin must have already created the Slack app and Notion integration for this member. See the admin onboarding guide at `areas/governance/workflows/how-to-onboard.md`.

---

### Step 0 — Detect existing setup

Use `AskUserQuestion`: "Have you already completed the light setup (Slack + GitHub protocols)?"

**If YES → Upgrade path.** Skip to **Step 2b — Notion upgrade** below.

**If NO → Full setup.** Proceed with Step 1.

---

### Step 1 — Gather parameters

Use `AskUserQuestion` to ask the user:

1. **Name** — their first name (used throughout the setup)
2. **Working directory** — the root path of the repo/folder where the agent will operate. Default: current working directory.

Do NOT ask for teamspace — it is always General.

Tell the user: "Your teamspace is General."

---

### Step 2 — Receive tokens from admin

Tell the user:

```
Your admin (Patrick) has already created the Slack and Notion apps for you.
You should have received two tokens:

1. **Slack Bot Token** — starts with `xoxb-`
2. **Notion Integration Secret** — starts with `ntn_`

If you haven't received them yet, ask Patrick to send them to you securely.

Do you have both tokens?
```

Use `AskUserQuestion` to wait for the user to confirm they have both tokens.

---

### Step 3 — Prerequisites & MCP server configuration (human step)

Tell the user:

```
Now let's make sure you have the right tools installed and configure the MCP servers.

## 1. Check prerequisites

The MCP servers require Node.js (v18+) and npm. Let's check if you have them.

Open a terminal and run:

  node --version
  npm --version

### If both commands return a version number → skip to step 2.

### If Node.js / npm are NOT installed:

**macOS:**
  - If you have Homebrew: brew install node
  - If you don't have Homebrew:
    1. Install Homebrew first: /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    2. Then: brew install node
  - Alternative (no Homebrew): download the macOS installer from https://nodejs.org

**Windows:**
  - Download the Windows installer (.msi) from https://nodejs.org
  - Run it and follow the prompts (make sure "Add to PATH" is checked)
  - Restart your terminal after installation

**Linux:**
  - Ubuntu/Debian: sudo apt update && sudo apt install nodejs npm
  - Fedora: sudo dnf install nodejs npm
  - Or use nvm: curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
    then: nvm install --lts

After installing, verify with: node --version && npm --version

## 2. Install the MCP server packages

Run this in your terminal:

  npm install -g @modelcontextprotocol/server-slack @notionhq/notion-mcp-server

This installs both MCP servers globally so they're available to Claude Code.

## 3. Configure settings.json

Open (or create) the file ~/.claude/settings.json and add the following
under the "mcpServers" key. If the file already exists, merge this into it.

{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-paste-your-token-here",
        "SLACK_TEAM_ID": "T0A9Z6HAXDX"
      }
    },
    "notionApi": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "NOTION_TOKEN": "ntn_paste-your-token-here"
      }
    }
  }
}

Replace the placeholder tokens with the ones you received from Patrick.
The SLACK_TEAM_ID is already set to the Ametyst workspace.

After saving the file, restart Claude Code so the MCP servers load.
Tell me when you're done.
```

Use `AskUserQuestion` to wait for confirmation.

---

### Step 4 — Slack connection verification

Post a test message to #general (channel ID `C0A9L8KDFEW`):

```
:wave: *New agent connecting — <Name>*
Testing Slack connection. Setup in progress.
```

If the post fails with a `not_in_channel` error, tell the user:
"The bot hasn't been invited to #general yet. Ask your admin to type `/invite @ametyst-agent-<name>` in #general, then tell me to retry."

If the post succeeds, confirm: "Slack connection verified!"

Do NOT proceed until Slack is working.

---

### Step 5 — Download protocols from GitHub (General only)

Create the folder structure:

```
.claude/rules/
.claude/rules/general-teamspace-communication/
.claude/skills/
.claude/agents/
.claude/guides/
context/company-context/
```

Clone the shared protocols repo:

```bash
gh repo clone ametyst-dev/protocols /tmp/ametyst-protocols
```

If the clone fails (no access), tell the user:
"Cannot access the protocols repo. Ask your admin to confirm you have access to the ametyst-dev GitHub org."

Copy files from the clone to local. **The teamspace subfolder (e.g. `general/`, `product/`) is NOT replicated locally — it's only for organization in the repo.** The mapping is:

- `company-context/*` → `context/company-context/`
- `rules/general/communication/*` → `.claude/rules/general-teamspace-communication/`
- `rules/general/*` (non-folder files like `notion-content-rules.md`) → `.claude/rules/`
- `skills/general/*` → `.claude/skills/<skill-name>/` (preserving internal structure)
- `agents/general/*` → `.claude/agents/`

Clean up:
```bash
rm -rf /tmp/ametyst-protocols
```

Tell the user which protocols, skills, and guides were downloaded.

---

### Step 6 — Create communication-routing.md

Create `.claude/rules/communication-routing.md` with this exact content:

```markdown
# Communication Routing

## Rule
Before any Slack message or Notion API call, read the relevant protocol files below.

## Routing table

| Context | Protocol files |
|---|---|
| #general, #standup, #news, Knowledge Base | `.claude/rules/general-teamspace-communication/` |

All paths are relative to the project root.

## Loading rules
- Always load `notion-protocol.md` first (IDs, queries, write rules)
- Load `notion-schema.md` only when creating or updating entries
- Load `slack-protocol.md` before posting any message
- Follow message formats exactly — no paraphrasing
```

Tell the user that communication routing has been configured.

---

### Step 7 — Verify Knowledge Base

The company context files were already downloaded in Step 5 (from `general/company-context/` in the GitHub repo). Verify they exist:

```bash
ls context/company-context/
```

Expected files: COMPANY.md, PRODUCT.md, COMPETITORS.md, MARKET.md (and possibly others).

If the folder is empty or missing files → warn the user: "Company context files are missing from the protocols repo. Ask your admin to push them with `/share-protocols`."

Tell the user which company context files are available.

---

### Step 8 — Build self-context

Create the `context/self-context/` folder.

Ask the user in two rounds using `AskUserQuestion` to keep it conversational:

**Round 1 — Identity:**
1. **Who are you?** — "What's your role at Ametyst? What's your background and expertise?"
2. **Mission** — "What drives you? What do you believe about the problem Ametyst is solving?"
3. **What you work on** — "What's your main area of focus at Ametyst right now?"

Wait for answers, then proceed.

**Round 2 — Working style:**
4. **How you work** — "How do you prefer to work? Any specific habits, principles, or patterns? (e.g. structured execution, async-first, etc.)"
5. **What you dislike** — "What do you find unhelpful or annoying in how an AI agent works with you?"
6. **How to work with you** — "Any specific instructions for your agent? What makes collaboration with you effective?"

After collecting all answers, generate `context/self-context/<Name>.md` with this structure:

```markdown
# <Name>

## Who they are
<answer 1>

## Mission
<answer 2>

## What they work on
<answer 3>

## How they work
<answer 4>

## What they dislike
<answer 5>

## How to work with them
<answer 6>
```

Tell the user their self-context has been created.

---

### Step 9 — Generate CLAUDE.md

Check if `context/company-context/claude-template.md` exists (it should have been downloaded in Step 7 as part of the Knowledge Base).

If it exists, read it and use it as the base template, replacing:
- `<Name>` with the user's name
- `<project-name>` with the working directory name

If it does not exist, use this fallback template:

```markdown
# CLAUDE.md — <project-name>/

## Purpose
<Name>'s operative system within Ametyst. This is the environment from which <Name> launches agents and operates within the Ametyst ecosystem.

## Context loading protocol
1. **Always auto-load** `context/self-context/<Name>.md` and relevant files in `context/company-context/`
2. **Never explore freely** through folders without being asked — load only what the task requires

## Communication protocols
Before any Slack or Notion interaction, read `.claude/rules/communication-routing.md` to find the correct protocol files for the target channel or database. Skip this step only if the active skill already specifies which protocol files to read.

## Startup routine
On the first message of every session:
1. Run the `ops-sync-protocols` skill to check for any protocol updates across active teamspaces. Apply updates silently unless there are changes to report.
2. If `context/company-context/claude-template.md` was updated, compare it against this CLAUDE.md and report any structural differences to the user.

## Submodules map
- `context/self-context/` — read-only context about <Name>; load when you need to ground reasoning in their perspective
- `context/company-context/` — read-only Ametyst knowledge base; load only the specific files relevant to your task

## Operations supported
- ✅ Do: reference `context/self-context/` to ground agent reasoning
- ✅ Do: reference `context/company-context/` for Ametyst-specific context
- ❌ Don't: modify `context/self-context/` or `context/company-context/` — both are read-only

## Boundaries
- `context/self-context/` and `context/company-context/` are **read-only** — never modify files there
- Always read a sub-module's `CLAUDE.md` before operating inside it

## Language
- All files created or modified must be written in **English**
```

Write to `.claude/CLAUDE.md`.

Tell the user their CLAUDE.md has been generated.

---

### Step 10 — Summary + welcome message

Post a welcome message to #general (channel ID `C0A9L8KDFEW`):

```
:wave: *New agent online — <Name> (<Role>)*
Teamspace: General
Setup complete — protocols, knowledge base, and context loaded.
```

Before posting, show the user the message and ask for confirmation.

Then tell the user everything that was set up:

```
Setup complete! Here's what was configured:

- Tokens: Slack + Notion (provided by admin)
- MCP servers: connected and verified
- Protocols: <count> files downloaded
- Skills: <count> downloaded
- Agents: <count> downloaded
- Communication routing: configured for General
- Knowledge Base: <count> files synced to context/company-context/
- Self-context: <Name>.md created
- CLAUDE.md: generated with startup routine
- Welcome message: posted to #general

Your agent is now connected to the Ametyst ecosystem.
On every new session, it will automatically check for
protocol updates and sync them.
```

---
---

## UPGRADE PATH — Step 2b (for users who already completed light setup)

This section runs ONLY when the user answered "YES" in Step 0. They already have Slack, GitHub protocols, company context, self-context, and a CLAUDE.md from the light setup. This upgrade adds Notion access.

---

### Step 2b-1 — Verify existing setup

Check that the light setup artifacts exist:
- `.claude/CLAUDE.md` exists
- `.claude/rules/communication-routing.md` exists
- `.claude/rules/general-teamspace-communication/slack-protocol.md` exists
- `context/self-context/` has at least one `.md` file
- `context/company-context/` has at least one `.md` file

If any are missing, tell the user: "Your light setup seems incomplete. Run `/light-setup-agent` first to complete it, then come back here."

Extract the user's name from the self-context file (the filename without `.md`).

---

### Step 2b-2 — Receive Notion token

Tell the user:

```
Your admin (Patrick) should have sent you a Notion Integration Secret (starts with `ntn_`).

If you haven't received it yet, ask Patrick to send it to you securely.

Do you have the token?
```

Use `AskUserQuestion` to wait for confirmation.

---

### Step 2b-3 — Configure Notion MCP server

Tell the user:

```
Let's add the Notion MCP server to your configuration.

Open ~/.claude/settings.json and add the following entry inside the "mcpServers" key
(alongside the existing "slack" entry):

    "notionApi": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "NOTION_TOKEN": "ntn_paste-your-token-here"
      }
    }

Also install the package:

  npm install -g @notionhq/notion-mcp-server

After saving, restart Claude Code so the new MCP server loads.
Tell me when you're done.
```

Use `AskUserQuestion` to wait for confirmation.

---

### Step 2b-4 — Verify Notion connection

Test the connection by querying the General Protocols database:

```json
{ "data_source_id": "86503a95-04f3-4d76-9e9a-898b71006742" }
```

If the query succeeds → confirm: "Notion connection verified!"

If it fails → tell the user: "Cannot connect to Notion. Check that the token is correct and that the admin has added the integration to the General backend page."

Do NOT proceed until Notion is working.

---

### Step 2b-5 — Download Notion protocols

Download the full General protocols from Notion (same as Step 5 of the full setup):

Query the General Protocols database:
```json
{ "data_source_id": "86503a95-04f3-4d76-9e9a-898b71006742" }
```

For each entry, get page content via `API-get-block-children`.

Sort by Category and save:
- **Communication** → `.claude/rules/general-teamspace-communication/` (this overwrites Slack-only files from light setup and adds notion-protocol.md and notion-schema.md)
- **Skill** → `.claude/skills/<doc-name>/SKILL.md`
- **Agent** → `.claude/agents/<doc-name>.md`
- **Guide** → `.claude/guides/<doc-name>.md`
- **Other** → `.claude/rules/<doc-name>.md`

Tell the user which files were downloaded, highlighting what's NEW vs what was updated.

---

### Step 2b-6 — Update communication-routing.md

Read the existing `.claude/rules/communication-routing.md`. Update it to include Notion references:

```markdown
# Communication Routing

## Rule
Before any Slack message or Notion API call, read the relevant protocol files below.

## Routing table

| Context | Protocol files |
|---|---|
| #general, #standup, #news, Knowledge Base | `.claude/rules/general-teamspace-communication/` |

All paths are relative to the project root.

## Loading rules
- Always load `notion-protocol.md` first (IDs, queries, write rules)
- Load `notion-schema.md` only when creating or updating entries
- Load `slack-protocol.md` before posting any message
- Follow message formats exactly — no paraphrasing
```

---

### Step 2b-7 — Update CLAUDE.md

Read the existing `.claude/CLAUDE.md`. Add or update:

1. **Startup routine** — add:
   ```
   ## Startup routine
   On the first message of every session:
   1. Run the `ops-sync-protocols` skill to check for any protocol updates. Apply updates silently unless there are changes to report.
   2. If `context/company-context/claude-template.md` was updated, compare it against this CLAUDE.md and report any structural differences.
   ```

2. **Communication protocols** — update to reference Notion:
   ```
   Before any Slack or Notion interaction, read `.claude/rules/communication-routing.md`
   ```

3. **Remove the "light setup" note** at the bottom if present.

---

### Step 2b-8 — Confirm upgrade

Post to #general (channel ID `C0A9L8KDFEW`):

```
:arrow_up: *Agent upgraded — <Name>*
Notion access added. Full setup complete.
```

Before posting, show the user the message and ask for confirmation.

Then tell the user:

```
Upgrade complete! What was added:

- Notion: connected and verified
- Protocols: full set downloaded from Notion (including notion-protocol, notion-schema)
- Skills: Notion-dependent skills added (ops-sync-protocols, ops-fetch-teamspace)
- Communication routing: updated with Notion references
- CLAUDE.md: updated with Notion sync startup routine

You now have full access to the Ametyst ecosystem.
To add Strategy or Governance teamspaces, run /ops-fetch-teamspace.
```
