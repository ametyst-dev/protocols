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

## Guides
When the user asks "how do I...?" about ecosystem operations (adding a teamspace, onboarding, etc.), check `.claude/guides/` for relevant how-to files before answering. These guides contain step-by-step instructions maintained by the admin.

## Submodules map
- `context/self-context/` — read-only context about <Name>; load when you need to ground reasoning in their perspective
- `context/company-context/` — read-only Ametyst knowledge base; load only the specific files relevant to your task
- `.claude/guides/` — how-to guides for ecosystem operations; check here when the user asks how to do something

## Operations supported
- Do: reference `context/self-context/` to ground agent reasoning
- Do: reference `context/company-context/` for Ametyst-specific context
- Don't: modify `context/self-context/` or `context/company-context/` — both are read-only

## Boundaries
- `context/self-context/` and `context/company-context/` are **read-only** — never modify files there
- Always read a sub-module's `CLAUDE.md` before operating inside it

## Language
- All files created or modified must be written in **English**
