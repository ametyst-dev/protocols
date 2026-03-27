---
description: Lightweight onboarding skill that connects a new team member's agent to the Ametyst ecosystem using only Slack and GitHub — no Notion required. Downloads protocols and company context from a shared GitHub repo, configures Slack communication, builds self-context, and generates CLAUDE.md. Use whenever a new person needs quick access to start working with Ametyst agents, or when someone says "light setup", "quick setup", "onboard without Notion".
allowed-tools: Read, Write, Edit, Glob, Bash(gh *), Bash(node *), Bash(npm *), AskUserQuestion, mcp__slack__slack_post_message, mcp__slack__slack_get_channel_history
---

## light-setup-agent

Guide a new team member through a lightweight onboarding that connects their agent to the Ametyst ecosystem using only Slack and a shared GitHub repo. No Notion access required — this is the fast path for people who need to start working quickly.

**Important:** This skill must run on **Claude Code** (CLI). The user's teamspace is **General**.

**Prerequisite:** The admin (Patrick) must have already:
1. Created a Slack app for this member (or shared an existing bot token)
2. Ensured the member has access to the `ametyst-dev` GitHub org
3. Populated the `ametyst-dev/protocols` repo with the relevant protocols

---

### Step 1 — Gather parameters

Use `AskUserQuestion` to ask the user:

1. **Name** — their first name (used throughout the setup)
2. **Working directory** — the root path of the repo/folder where the agent will operate. Default: current working directory.

Tell the user: "This is the light setup — Slack + GitHub protocols. No Notion needed. If you need Notion access later, your admin can upgrade you with the full setup."

---

### Step 2 — Receive Slack token from admin

Tell the user:

```
Your admin (Patrick) should have sent you a Slack Bot Token (starts with `xoxb-`).

If you haven't received it yet, ask Patrick to send it to you securely.

Do you have the token?
```

Use `AskUserQuestion` to wait for the user to provide the token or confirm they have it.

---

### Step 3 — Prerequisites & MCP server configuration

Tell the user:

```
Let's make sure you have the right tools and configure the Slack MCP server.

## 1. Check prerequisites

Open a terminal and run:

  node --version
  npm --version

### If both return a version number → skip to step 2.

### If Node.js / npm are NOT installed:

**macOS:**
  - If you have Homebrew: brew install node
  - If not: download from https://nodejs.org

**Windows:**
  - Download the installer from https://nodejs.org
  - Run it (make sure "Add to PATH" is checked)
  - Restart your terminal

**Linux:**
  - Ubuntu/Debian: sudo apt update && sudo apt install nodejs npm
  - Or use nvm: curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
    then: nvm install --lts

Verify with: node --version && npm --version

## 2. Install the Slack MCP server

Run:

  npm install -g @modelcontextprotocol/server-slack

## 3. Configure settings.json

Open (or create) the file ~/.claude/settings.json and add the following
under the "mcpServers" key:

{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-paste-your-token-here",
        "SLACK_TEAM_ID": "T0A9Z6HAXDX"
      }
    }
  }
}

Replace the placeholder token with the one you received from Patrick.
The SLACK_TEAM_ID is already set to the Ametyst workspace.

After saving, restart Claude Code so the MCP server loads.
Tell me when you're done.
```

Use `AskUserQuestion` to wait for confirmation.

---

### Step 4 — Slack connection verification

Post a test message to #general (channel ID `C0A9L8KDFEW`):

```
:wave: *New agent connecting — <Name>*
Testing Slack connection. Light setup in progress.
```

If the post fails with a `not_in_channel` error, tell the user:
"The bot hasn't been invited to #general yet. Ask your admin to type `/invite @ametyst-agent-<name>` in #general, then tell me to retry."

If the post succeeds, confirm: "Slack connection verified!"

Do NOT proceed until Slack is working.

---

### Step 5 — Download protocols from GitHub

Clone the shared protocols repo:

```bash
gh repo clone ametyst-dev/protocols /tmp/ametyst-protocols
```

If the clone fails (no access), tell the user:
"Cannot access the protocols repo. Ask your admin to confirm you have access to the ametyst-dev GitHub org."

Once cloned, create the folder structure in the user's working directory:

```
.claude/rules/
.claude/skills/
.claude/agents/
context/company-context/
```

**Download `company-context`** and **`claude-template.md`** (always, from root):
- `company-context/*` → `context/company-context/`
- `claude-template.md` → `context/company-context/claude-template.md`

**Only download `general`** — no other teamspaces. The teamspace subfolder is NOT replicated locally — it's only for organization in the repo. Copy files using this mapping:

1. **Communication** (`rules/<teamspace>/communication/*`):
   → Copy to `.claude/rules/<teamspace>-teamspace-communication/`
   → **Skip** any file with `notion` in the name (notion-protocol.md, notion-schema.md) — this is light setup, no Notion.

2. **Rules** (`rules/<teamspace>/*` — non-folder files):
   → Copy to `.claude/rules/`

3. **Skills** (`skills/<teamspace>/*` — each subfolder or file is a skill):
   → Copy each subfolder to `.claude/skills/<name>/` (preserving internal structure)
   → For single .md files, create `.claude/skills/<name>/SKILL.md`

4. **Agents** (`agents/<teamspace>/*`):
   → Copy to `.claude/agents/`

5. **Guides** (`guides/<teamspace>/*`):
   → Copy to `.claude/guides/`

Tell the user which files were downloaded.

Clean up:
```bash
rm -rf /tmp/ametyst-protocols
```

---

### Step 6 — Create communication-routing.md

Create `.claude/rules/communication-routing.md` with this exact content:

```markdown
# Communication Routing

## Rule
Before any Slack message, read the relevant protocol files below.

## Routing table

| Context | Protocol files |
|---|---|
| #general, #standup, #news | `.claude/rules/general-teamspace-communication/` |

All paths are relative to the project root.

## Loading rules
- Load `slack-protocol.md` before posting any message
- Follow message formats exactly — no paraphrasing

## Note
This is a light setup — Slack only, no Notion. To add Notion access and additional teamspaces, ask your admin to run the full setup upgrade.
```

Tell the user that communication routing has been configured.

---

### Step 7 — Build self-context

Create the `context/self-context/` folder.

Ask the user in two rounds using `AskUserQuestion`:

**Round 1 — Identity:**
1. **Who are you?** — "What's your role at Ametyst? What's your background and expertise?"
2. **Mission** — "What drives you? What do you believe about the problem Ametyst is solving?"
3. **What you work on** — "What's your main area of focus at Ametyst right now?"

Wait for answers, then proceed.

**Round 2 — Working style:**
4. **How you work** — "How do you prefer to work? Any specific habits, principles, or patterns?"
5. **What you dislike** — "What do you find unhelpful or annoying in how an AI agent works with you?"
6. **How to work with you** — "Any specific instructions for your agent? What makes collaboration effective?"

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

### Step 8 — Generate CLAUDE.md

Check if `context/company-context/claude-template.md` exists (it should have been downloaded in Step 5 from the repo root).

If it exists, read it and use it as the base template, replacing:
- `<Name>` with the user's name
- `<project-name>` with the working directory name
- Remove any Notion-related sections (ops-sync-protocols in startup, Notion references)
- Remove any teamspace references beyond General

If it does not exist, use this fallback template:

```markdown
# CLAUDE.md — <project-name>/

## Purpose
<Name>'s operative system within Ametyst. This is the environment from which <Name> launches agents and operates within the Ametyst ecosystem.

## Context loading protocol
1. **Always auto-load** `context/self-context/<Name>.md` and relevant files in `context/company-context/`
2. **Never explore freely** through folders without being asked — load only what the task requires

## Communication protocols
Before any Slack message, read `.claude/rules/communication-routing.md` to find the correct protocol files.

## Submodules map
- `context/self-context/` — read-only context about <Name>; load when you need to ground reasoning in their perspective
- `context/company-context/` — read-only Ametyst knowledge base; load only the specific files relevant to your task

## Operations supported
- ✅ Do: reference `context/self-context/` to ground agent reasoning
- ✅ Do: reference `context/company-context/` for Ametyst-specific context
- ❌ Don't: modify `context/self-context/` or `context/company-context/` — both are read-only

## Boundaries
- `context/self-context/` and `context/company-context/` are **read-only** — never modify files there

## Language
- All files created or modified must be written in **English**

## Note
This agent was set up with light setup (Slack + GitHub protocols, no Notion).
To upgrade to full access (Notion databases, teamspace sync), ask your admin.
```

Write to `.claude/CLAUDE.md`.

Tell the user their CLAUDE.md has been generated.

---

### Step 9 — Summary + welcome message

Post a welcome message to #general (channel ID `C0A9L8KDFEW`):

```
:wave: *New agent online — <Name> (light setup)*
Teamspace: General
Connected via: Slack + GitHub protocols
Setup complete — company context and communication loaded.
```

Before posting, show the user the message and ask for confirmation.

Then **dynamically discover** what was installed by scanning the local file system:

1. **Skills** — list all folders in `.claude/skills/`. For each, read the skill file (SKILL.md or the main .md) and extract the `description` from the frontmatter or first paragraph. Build a list of skill name + one-line description.
2. **Agents** — list all files in `.claude/agents/`. For each, read the file and extract the description. Build a list of agent name + one-line description.
3. **Guides** — list all files in `.claude/guides/` (if it exists). For each, read the title (first `#` heading). Build a list of guide name + one-line description.

Present the full recap to the user:

```
Light setup complete! Here's what was configured:

- Slack: connected and verified
- Protocols: downloaded from GitHub (ametyst-dev/protocols)
- Company context: synced to context/company-context/
- Communication: Slack protocols loaded (General teamspace)
- Self-context: <Name>.md created
- CLAUDE.md: generated

Your agent is now connected to the Ametyst ecosystem via Slack.

What you CAN do:
- Communicate on Slack channels
- Access company context (who we are, what we build, competition)
- Work with Cursor on code tasks
- Collaborate with other agents via Slack

Skills installed:
<dynamically generated list — skill name + one-line description>

Agents installed:
<dynamically generated list — agent name + one-line description>
(omit section if no agents were installed)

Guides installed:
<dynamically generated list — guide name + one-line description>
(omit section if no guides were installed)

What you CANNOT do yet (requires full setup):
- Access Notion databases (OKRs, projects, tasks, docs)
- Sync protocol updates automatically from Notion
- Access Strategy or Governance teamspaces

To upgrade, ask Patrick to run the full setup for you.
```
