# How to add a new teamspace

This guide explains how to expand your agent's access to a new teamspace (Strategy or Governance). After setup, you start with General only. When your admin grants you access to another teamspace, follow these steps to download its protocols and start operating in it.

## Prerequisites

You need two things from your admin before proceeding:

### 1. Notion access

Ask your admin to add your Notion integration as a connection on the new teamspace's backend page:

1. Open the teamspace backend page in Notion (e.g. "Strategy Backend" or "Governance Backend")
2. Click "..." (top right) → "Add connections"
3. Search for your integration name (e.g. "Marco's Agent") and add it

Without this, your agent cannot read or write to the teamspace's databases.

### 2. Slack channel access

Ask your admin to invite your Slack bot to the teamspace's channel:

1. Open the teamspace's Slack channel (#strategy or #governance)
2. Type: `/invite @ametyst-agent-<your-name>`

Without this, your agent cannot post updates to the teamspace channel.

## Setup

Once your admin confirms both accesses are in place, run the `ops-fetch-teamspace` skill:

```
/ops-fetch-teamspace
```

The skill will:
1. Ask which teamspace to add (Strategy / Governance)
2. Download all communication protocols (notion-protocol, notion-schema, slack-protocol) into `.claude/rules/<teamspace>-teamspace-communication/`
3. Update `.claude/rules/communication-routing.md` with the new routing entry
4. Verify Notion access by running a test query
5. Confirm everything is set up

After this, your agent will automatically:
- Include the new teamspace when scanning for protocol updates (via `ops-sync-protocols`)
- Use the correct Notion databases and Slack channel when you work in that teamspace's context

## Troubleshooting

**"Notion integration doesn't have access"** — Your admin hasn't added your integration to the backend page yet. Ask them to do it and retry.

**"not_in_channel" Slack error** — Your bot hasn't been invited to the channel. Ask your admin to run `/invite @ametyst-agent-<your-name>` in the channel.

**Protocols look outdated** — Run `ops-sync-protocols` to pull the latest updates from Slack/Notion.
