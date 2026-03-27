# Ametyst Protocols

Shared protocols, company context, and agent configuration for the Ametyst ecosystem. This repository is the central hub for onboarding new team members and setting up their personal AI agent within Ametyst.

## What is this?

At Ametyst, every team member operates with a personal AI agent (powered by Claude Code) that is connected to the company's communication channels, knowledge base, and development workflows. This repository contains everything needed to configure that agent.

When a new person joins Ametyst, they clone this repo and run a setup skill that automatically configures their agent with the right protocols, company context, and communication rules.

## Setup paths

There are two onboarding paths depending on the level of access needed:

### Light setup

A fast onboarding path that connects the agent to **Slack and GitHub only**. No Notion access required. Ideal for:
- New team members in their trial period
- External collaborators who need to work with Ametyst agents quickly
- Co-founders testing the development workflow

The agent receives: company context (who we are, what we build, market, competition), Slack communication protocols, and any skills assigned to the selected teamspaces. Setup takes approximately 10 minutes.

### Full setup

A complete onboarding that includes everything in light setup plus **Notion database access**. This enables the agent to read and write to Notion databases (OKRs, projects, tasks, docs, meetings) and sync protocol updates automatically. Ideal for:
- Confirmed team members who need full operational access
- Anyone who needs to interact with Notion-based workflows

Full setup can be run from scratch or as an upgrade from an existing light setup. Setup takes approximately 15 minutes.

## Repository structure

```
company-context/    — Ametyst knowledge base (company, product, market, competitors)
rules/              — Communication protocols and rules, organized by teamspace
skills/             — Agent skills, organized by teamspace
agents/             — Agent definitions (sub-agents used by skills)
guides/             — How-to guides for ecosystem operations
claude-template.md  — Template used to generate each agent's CLAUDE.md
```

Each folder (except `company-context/`) is organized by **teamspace**: `general/`, `product/`, `strategy/`, `governance/`. Teamspaces group protocols by domain — during setup, agents download one or more teamspaces based on their role.

## How it works

1. **Admin** pushes protocol files here from the central management system
2. **New member** runs `/light-setup` or `/full-setup-agent` in Claude Code — the skill clones this repo and configures the agent automatically
3. **Existing member** can add teamspaces later with `/set-fetch-teamspace`
4. **Protocol updates** are announced on Slack — agents fetch updated files from this repo when notified

## Who maintains this

This repository is maintained by the admin. Team members pull from it during setup and sync — they do not push directly.
