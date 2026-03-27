---
description: Download communication protocols for a new teamspace (Strategy or Governance) from the shared GitHub repo and update communication-routing.md. Use when you've been given access to a new teamspace. Triggered by "fetch teamspace", "add teamspace", "expand to strategy", "expand to governance", or similar.
allowed-tools: Read, Write, Edit, Glob, Bash(gh *), AskUserQuestion, mcp__notionApi__API-query-data-source
---

## ops-fetch-teamspace

Download communication protocols for a new teamspace from GitHub and update communication-routing.md. Use this skill when you've been given access to Strategy or Governance and need to set up the protocols for that teamspace.

---

### Step 1 — Determine which teamspace to fetch

Use `AskUserQuestion` to ask: "Which teamspace do you want to add? (Strategy / Governance)"

If the user says both, process them sequentially — complete all steps for the first one before starting the second.

---

### Step 2 — Check if already fetched

Check if `.claude/rules/<teamspace>-teamspace-communication/` folder already exists.

If it does, use `AskUserQuestion` to ask: "This teamspace's protocols are already downloaded. Do you want to re-fetch and overwrite? (yes / no)"

If the user says no, skip to Step 6 (or proceed to the next teamspace if processing both).

---

### Step 3 — Download protocols from GitHub

Clone the shared protocols repo (or pull if already cloned):

```bash
gh repo clone ametyst-dev/protocols /tmp/ametyst-protocols 2>/dev/null || (cd /tmp/ametyst-protocols && git pull)
```

Copy the teamspace's files. **The teamspace subfolder is NOT replicated locally — it's only for organization in the repo:**

- `rules/<teamspace>/communication/*` → `.claude/rules/<teamspace>-teamspace-communication/`
- `skills/<teamspace>/*` → `.claude/skills/<skill-name>/` (preserving internal structure, if any exist)
- `agents/<teamspace>/*` → `.claude/agents/` (if any exist)

Create the folder if it doesn't exist.

Clean up:
```bash
rm -rf /tmp/ametyst-protocols
```

Tell the user which protocol files were downloaded.

---

### Step 4 — Update communication-routing.md

Read the existing `.claude/rules/communication-routing.md`.

Use Edit to add the new routing row to the Routing table, inserting it before the line `All paths are relative to the project root.`:

**For Strategy:**
```
| #strategy, Strategy databases (Objectives, KRs, Meetings, Projects, Tasks, Fundraising, Docs, Workflows, Protocols, Team Members) | `.claude/rules/strategy-teamspace-communication/` |
```

**For Governance:**
```
| #governance, Governance databases (Objectives, KRs, Meetings, Projects, Tasks, Docs, Workflows, Protocols) | `.claude/rules/governance-teamspace-communication/` |
```

Do NOT overwrite the file — only add the new row.

---

### Step 5 — Verify Notion access (if Notion is configured)

Check if the user has a Notion MCP server configured by looking for `notionApi` in the MCP tools available.

**If Notion is configured:**

Read the just-downloaded `.claude/rules/<teamspace>-teamspace-communication/notion-protocol.md` and find the Tasks database `data_source_id` from the Database registry table.

Query it with a Team filter:
```json
{
  "data_source_id": "<Tasks data_source_id from notion-protocol.md>",
  "filter": { "property": "Team", "multi_select": { "contains": "<teamspace>" } }
}
```

If the query fails: "Your Notion integration doesn't have access to the <teamspace> backend yet. Ask your admin to add your integration as a connection on the <teamspace> backend page in Notion."

If it succeeds: "Notion access verified for <teamspace>!"

**If Notion is NOT configured (light setup user):**

Skip this step. Tell the user: "Notion is not configured — teamspace protocols are available for Slack communication only. To access <teamspace> Notion databases, upgrade to full setup."

---

### Step 6 — Report

Tell the user:

```
Teamspace <teamspace> added successfully!

- Protocols: <count> files downloaded to .claude/rules/<teamspace>-teamspace-communication/
- Communication routing: updated with <teamspace> entry
- Notion access: <verified / not configured>

You now have access to:
- Slack: #<channel-name>
- Notion: <list of databases> (if configured)

Your agent will use these protocols automatically when interacting
with <teamspace> channels and databases.
```
