# Communication Routing

## Rule
Before any Slack message or Notion API call, read the relevant protocol files below.

## Routing table

| Context | Protocol files |
|---|---|
| #general, #standup, #news, Knowledge Base | `.claude/rules/general-teamspace-communication/` |
| #governance, Governance databases (Objectives, KRs, Meetings, Projects, Tasks, Docs, Workflows) | `.claude/rules/governance-teamspace-communication/` |
| #strategy, Strategy databases (Objectives, KRs, Meetings, Projects, Tasks, Fundraising, Docs, Workflows, Team Members) | `.claude/rules/strategy-teamspace-communication/` |
| Sprint channels, engineering coordination | `.claude/skills/eng-sprint-planner/protocols/slack-protocol.md` |

All paths are relative to the project root.

## Loading rules
- Always load `notion-protocol.md` first (IDs, queries, write rules)
- Load `notion-schema.md` only when creating or updating entries
- Load `slack-protocol.md` before posting any message
- Follow message formats exactly — no paraphrasing
