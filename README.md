# Ametyst Protocols

Shared protocols, company context, and agent configuration for the Ametyst ecosystem. This repo is the distribution hub for onboarding new team members and syncing protocol updates.

## Structure

```
company-context/    — Ametyst knowledge base (company, product, market, competitors)
rules/              — Communication protocols and rules, organized by teamspace
skills/             — Agent skills, organized by teamspace
agents/             — Agent definitions (sub-agents used by skills)
guides/             — How-to guides for ecosystem operations
```

Each folder (except `company-context/`) is organized by **teamspace**: `general/`, `product/`, `strategy/`, `governance/`. The teamspace subfolder is for organization only — when downloaded locally, files go directly into `.claude/` without the teamspace prefix.

## How it's used

- **Admin** pushes files here via `/share-protocols` from domain-expansion
- **Light setup** (`/light-setup-agent`) clones this repo to onboard new members without Notion
- **Full setup** (`/setup-agent`) also clones from here for protocols
- **Sync** (`/ops-sync-protocols`) fetches individual files when Slack announces updates

## Who maintains this

Only the admin (Patrick) pushes to this repo via `/share-protocols`. Team members pull from it — they never push directly.
