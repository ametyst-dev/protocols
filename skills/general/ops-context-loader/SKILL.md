---
description: Load structured context by crawling CLAUDE.md files and docs/ at configurable depth from local folders or GitHub repos. Use whenever you need to understand a repo's structure, purpose, or domain boundaries before working on a task — especially before sprint planning, feature development, or any multi-repo work. Use this skill any time the user says things like "give me context on", "read the repo", "load context", or "what's in this codebase".
argument-hint: "[local/path or org/repo] [depth 1-3]"
allowed-tools: Bash(gh *), Read, Glob, AskUserQuestion, mcp__notionApi__API-query-data-source
---

## Context Loader

> **Before executing:** read `MEMORY.md` in this folder — it contains learned rules that override default behavior.

Your job is to collect context from one or more local folders or GitHub repos, then return a clean context snapshot.

### Step 1 — Gather parameters

Ask the user the following using AskUserQuestion (in a single call, up to 3 questions):

1. **Targets** — which repos or folders to crawl?
   - Local folder: absolute path (e.g. `/Users/patrickpinta/Desktop/domain-expansion`)
   - GitHub repo: `org/repo` format (e.g. `ametyst-dev/ametyst-core`)
   - Accept multiple targets as a comma-separated list

2. **Depth** — how deep to crawl?
   - `1` = root CLAUDE.md only (+ its docs/README.md)
   - `2` = root + direct submodules (one level down)
   - `3` = root + two levels down

3. **Focus** (optional) — any specific domain to prioritize? (e.g. `customers`, `product`, `all`)
   - Default to `all` if not specified

4. **Notion context** (optional) — do you also want to load context from a Notion database?
   - If yes, ask which teamspace (Strategy / Governance) and which database
   - Reference the teamspace's notion-protocol file (loaded in CLAUDE.md rules context) for the database list and `data_source_id`
   - Query the database using `API-query-data-source` with appropriate Team filter
   - Include the results in the context snapshot alongside the file-based context

### Step 2 — Launch the sub-agent

Launch a sub-agent of type `repo-crawler` with the following prompt:

```
Targets: {TARGETS}
Depth: {DEPTH}
Focus: {FOCUS}
```

The `repo-crawler` agent handles all crawling logic, CLAUDE.md boundary rules, and output formatting. See `.claude/agents/repo-crawler.md` for its full instructions.

### Step 3 — Present the result

When the sub-agent returns:
1. Display the context snapshot to the user
2. Say: "Context loaded. Here's what I found — ready to work from this."
3. Keep the snapshot available in the current context for any follow-up work
