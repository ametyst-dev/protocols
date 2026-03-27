---
description: Orchestrate end-to-end sprint planning — loads repo context, brainstorms a step-by-step implementation guide collaboratively, then writes the approved plan to a new branch in the target repos. Use whenever Patrick wants to plan a sprint, kick off a new feature, or turn a goal into a structured guide ready to execute. Use this skill any time the user says things like "let's plan", "plan a sprint", "sprint planning", or "create a sprint guide".
argument-hint: "[sprint goal description]"
allowed-tools: AskUserQuestion, Agent, Bash(gh *), mcp__slack__slack_list_channels, mcp__slack__slack_post_message, Read, Write, EnterPlanMode, ExitPlanMode
---

## Sprint Planner

You are a sprint planning orchestrator. You run in four phases: context loading, plan-mode analysis + brainstorming, writing, and branch pushing. Do not skip phases.

---

## SLACK MESSAGE GUARDRAIL — READ THIS FIRST (applies to Phase 3)

Before sending any Slack message, read `.claude/skills/eng-sprint-planner/protocols/slack-protocol.md`.

**Every Slack message you send MUST follow the format defined in that file exactly — character by character. No paraphrasing. No shortcuts. No variations. Any deviation from the format breaks the system. This is a hard constraint, not a suggestion.**

---

### Step 1 — Gather parameters

Ask the user the following in a single AskUserQuestion call:

1. **Brainstorm input** — link to an existing brainstorm/PRD file (e.g. `areas/product/brainstorms/PRD - Feature X.md`) **or** provide inline text describing the context. This is required — the sprint planner does not start without a starting context. If the user has not done a brainstorm yet, ask them to either:
   - Run `/brainstorm` first with the `product-builder` prompt and come back with the output file
   - Or write a short description here covering: what to build, for whom, why now, which repos are likely involved
2. **Target repos** — which GitHub repos are involved? (`org/repo` format, comma-separated list — e.g. `ametyst-dev/core, ametyst-dev/api`)
3. **Branch name** — what should the branch be called? (e.g. `auth-refactor`, `onboarding-v2`)
4. **Sprint ID** — 4-digit sprint number (e.g. `0005`). Used for the Slack channel name and file naming.
5. **Sprint folder location** — where should the sprint folder be created in domain-expansion? (e.g. `product/development/sprints` or `product/development/feature-x/sprints`) Default: `product/development/sprints`.
6. **Context depth** — how deep to load context from the repos?
   - `1` = root CLAUDE.md only
   - `2` = root + direct submodules
   - `3` = root + two levels deep

If a brainstorm file was provided, read it immediately and keep its content in context for Phase 2. Pay special attention to the `## Sprint input` section if present — it contains pre-validated goal, repos, constraints, and key decisions.

---

### Phase 1 — Context loading (sub-agent)

Launch a sub-agent of type `repo-crawler` with the following prompt:

```
Targets: {TARGET_REPOS}
Depth: {DEPTH}
Focus: all
```

The `repo-crawler` agent handles all crawling logic and returns a structured context snapshot. See `.claude/agents/repo-crawler.md` for its full instructions.

Once the sub-agent returns, keep the context snapshot in your context. Do NOT show the raw snapshot to Patrick yet.

**Test infrastructure check:** Before proceeding to Phase 2, scan each repo's CLAUDE.md (from the context snapshot) for a `## Testing` section. For each repo, note:
- Does a `## Testing` section exist?
- Does it specify a test command (e.g. `npm test`, `pytest`, `go test ./...`)?
- Does it mention a test framework?
- Does it reference a `.env.test` or test credentials?

Flag any repo missing test infrastructure — this will be used in Phase 2 to decide if a setup step is needed.

Proceed to Phase 2.

---

### Phase 2 — Analysis + Brainstorming (plan mode, interactive)

**Enter plan mode now** by calling `EnterPlanMode`. This prevents any file writes during the analysis and brainstorming phase — you can only read, search, and reason. All writing happens after Patrick approves and you exit plan mode.

This phase happens entirely in your conversation with Patrick. Do not launch sub-agents here.

1. **Deep analysis** — before proposing anything, use plan mode to explore the repos in depth:
   - Read key files referenced in the context snapshot (entry points, shared types, config files)
   - Map cross-repo dependencies relevant to the sprint goal
   - Identify architectural constraints, patterns in use, and potential risks
   - Use Glob, Grep, Read freely — this is what plan mode is for

2. **Present the context briefly** — 3-5 bullet points summarizing what you learned about the repos (purpose, current state, key areas relevant to the sprint goal).

3. **Architecture lens** — Before decomposing macro steps, analyze the sprint goal through an architecture lens:
   - **User journeys drive technical decisions.** Trace the user flow end-to-end across repos. Which service handles each step? Where does data flow?
   - **Embrace boring technology for stability.** Prefer patterns already in use in the repos. Only introduce new technology if existing patterns cannot solve the problem.
   - **Design simple solutions that scale when needed.** Do not over-engineer. If a simple function solves the problem, do not build a service. Developer productivity is architecture.
   - **Connect every decision to business value.** Every ADR must explain why this decision matters for the user or the business, not just why it's technically elegant.
   - Ground every recommendation in real-world trade-offs: what you gain, what you lose, what breaks if you're wrong.

   Write the ADR section of the sprint guide based on this analysis. Only then proceed to macro step decomposition.

4. **UX lens (only if sprint involves user-facing UI)** — Before decomposing macro steps, describe the user experience:
   - **Paint the picture.** Tell the user's story so you FEEL the problem: who is this person, what are they trying to do, what frustrates them today?
   - **Start simple, evolve through feedback.** Describe the simplest version of the UI flow that solves the problem. Do not add screens or interactions that aren't essential for v1.
   - **Balance empathy with edge cases.** For each screen or interaction, note: what happens on error? What happens with empty state? What happens on slow connection?
   - **Data-informed.** If call transcripts, user feedback, or analytics exist in domain-expansion, reference them to justify design choices.

   Include the UX flow description in the Context section of the guidelines for repos that implement UI. If no UI is involved in this sprint, skip this step entirely.

5. **Present a sprint proposal** — follow this order strictly:
   - **First: decompose all steps end-to-end** — list every step with title, what, why, output. Do NOT assign touches yet. Get the full step list right before anything else.
   - **Then: assign touches per step** — once the step list is agreed, go through each step and identify which repos/packages it touches, with what specific changes. Never mix decomposition and touch analysis.
   - **Always specify target servers for endpoints** — when a step involves API endpoints, always specify which server hosts each endpoint and which auth guard it uses. Never leave endpoint placement ambiguous.
   - **Always include a stabilization step as the last macro step** — after all feature steps, add a final "Stabilization" step for cross-repo hotfixes, wiring issues, and integration fixes that emerge during execution. This step has no pre-defined touches — it absorbs upgrades and fixes that surface after the main steps are done.
   - **Test infrastructure step (Step 0)** — if Phase 1 flagged any repo as missing test infrastructure (`## Testing` section, test command, test framework, or `.env.test`), insert a **Step 0 — Test infrastructure setup** before the first feature step. This step configures the test framework, creates `.env.test` with sandbox credentials, adds the `## Testing` section to the repo's CLAUDE.md, and writes 2-3 exemplary tests as reference. If all repos already have test infrastructure, skip Step 0. If only some repos are missing it, Step 0 targets only those repos.
   - Keep it concrete and actionable. Include order of execution and any dependencies.

6. **Iterate** — Patrick will give feedback. Adjust the proposal until he explicitly approves it. You can read additional files during iteration if needed to answer questions or verify assumptions.

7. **Finalize the guide** — when Patrick says OK (or "looks good", "proceed", "go ahead"), produce the final sprint guide in this format:

    # Sprint Guide — {branch-name}

    **Date:** {today's date}
    **Goal:** {sprint goal}
    **Target repos:** {list}

    ---

    ## Overview

    {1-2 paragraph summary of what this sprint accomplishes and why}

    ---

    ## Steps

    ### Step 1 — {step title}
    **What:** {clear description of what to do}
    **Why:** {why this is needed / what it unlocks}
    **Output:** {what the expected result is}
    **Touches:**
    - `{org/repo}` → `{package/folder}` — {what changes here}
    - `{org/repo}` → `{package/folder}` — {what changes here}

    ### Step 2 — {step title}
    ...

    ---

    ## Architecture Decision Records

    {For every decision that impacts more than one repo — data formats, communication protocols, API contracts, naming conventions, auth flows — write an ADR. If the sprint touches a single repo, this section is optional.}

    ### ADR-001 — {Decision title}
    - **Context:** {why this decision is needed}
    - **Decision:** {what we decided}
    - **Alternatives:** {what we considered and rejected, with reasons}
    - **Consequences:** {impact on each repo involved}

    ---

    ## Test strategy

    {High-level test plan for the sprint. What gets tested, at which level, and what does NOT need tests.}

    - **Unit tests:** {what business logic / pure functions to test}
    - **Integration tests:** {which API endpoints or cross-module interactions to test}
    - **E2E tests:** {which user flows to test end-to-end, if any}
    - **Not tested:** {what is explicitly out of scope for testing — config, migrations, etc.}
    - **Credentials / environment:** {known sandbox keys, test DB, .env.test references}

    ---

    ## Dependencies and risks

    - {any dependencies between steps}
    - {any risks or things to watch out for}

    ---

    ## Definition of done

    - [ ] {checklist item}
    - [ ] {checklist item}
    - [ ] All tests pass in every repo

8. **Exit plan mode** — once Patrick approves the final guide, call `ExitPlanMode`. This unlocks file writes for Phase 3.

Proceed to Phase 3.

---

### Phase 3 — Slack setup

Before writing branches, create the sprint Slack channel and read the protocol file.

1. **Ask HIL to create the Slack channel manually:**

   Tell HIL: "Please create a Slack channel named `sprint-{SPRINT_ID}-{BRANCH_NAME}` (lowercase, hyphens only) and confirm when done."

   Wait for HIL confirmation before continuing.

   Then use `mcp__slack__slack_list_channels` to find the channel and retrieve its ID. Save it as `{CHANNEL_ID}` — use this ID (not the name) in all subsequent `mcp__slack__slack_post_message` calls.

2. **Discover and read all protocol files** from this repo (domain-expansion):

   Glob `.claude/skills/eng-sprint-planner/protocols/**/*` to find all files inside `protocols/`. Read every file found. These will be passed to the branch writer for deployment to each repo branch.

   **Path mapping rules** (the branch writer uses these to decide where each file goes):
   - `protocols/cursor/**` → `.cursor/` in the target repo (preserving subpath, e.g. `protocols/cursor/skills/develop/SKILL.md` → `.cursor/skills/develop/SKILL.md`)
   - `protocols/skills/**` → `.claude/skills/` in the target repo (preserving subpath, e.g. `protocols/skills/step-plan/SKILL.md` → `.claude/skills/step-plan/SKILL.md`)
   - `protocols/*.md` (root-level files like `slack-protocol.md`) → `brainstormings/sprint-{SPRINT_ID}-{BRANCH_NAME}/` in the target repo

   Keep all file contents and their relative paths (relative to `protocols/`) — you will pass them to the branch writer as a list.

3. **Send SPRINT_START** using `mcp__slack__post_message`:

```
[SPRINT_START] sprint-{SPRINT_ID}
from: sprint-planner
to: all
---
repos: [{TARGET_REPOS}]
goal: {SPRINT_GOAL}
```

---

### Phase 3b — Create sprint folder in domain-expansion

The sprint folder in domain-expansion is the master record of this sprint. It contains the aggregated view across all repos and is used by sprint-delegate for orchestration.

Sprint folder path: `{SPRINT_FOLDER_LOCATION}/sprint-{SPRINT_ID}-{BRANCH_NAME}/`

1. **Create `README.md`** at `{SPRINT_FOLDER_LOCATION}/sprint-{SPRINT_ID}-{BRANCH_NAME}/README.md`:

```markdown
# Sprint {SPRINT_ID} — {BRANCH_NAME}

**Date:** {TODAY_DATE}
**Goal:** {SPRINT_GOAL}
**Branch:** {BRANCH_NAME}
**Channel:** sprint-{SPRINT_ID}-{BRANCH_NAME}
**Repos:** {TARGET_REPOS}

---

## Overview

{the 1-2 paragraph overview from the approved sprint guide}

---

## Steps

{the full steps section from the approved sprint guide, including all Touches entries for every repo}

---

## Dependencies and risks

{from the approved sprint guide}

---

## Definition of done

{from the approved sprint guide}
```

2. **Create `sprint-memory.md`** at `{SPRINT_FOLDER_LOCATION}/sprint-{SPRINT_ID}-{BRANCH_NAME}/sprint-memory.md`:

   Read the template at `.claude/skills/eng-sprint-planner/templates/sprint-memory-template.md`. Fill in the placeholders (`{SPRINT_ID}`, `{AGENT_NAME}` = `sprint-planner`, `{SLACK_CHANNEL}` = `sprint-{SPRINT_ID}-{BRANCH_NAME}`, `{DATE}` = today's date) and write the result.

3. **Append to `.claude/skills/eng-sprint-delegate/memory.md`** — add the new sprint to the active sprints list (do NOT overwrite existing entries):

```markdown
- sprint_id: {SPRINT_ID}
  channel: sprint-{SPRINT_ID}-{BRANCH_NAME}
  active_sprint_path: {SPRINT_FOLDER_LOCATION}/sprint-{SPRINT_ID}-{BRANCH_NAME}
```

If the file does not exist, create it with this structure:
```markdown
# Sprint Delegate — Active Sprints

## Sprints

- sprint_id: {SPRINT_ID}
  channel: sprint-{SPRINT_ID}-{BRANCH_NAME}
  active_sprint_path: {SPRINT_FOLDER_LOCATION}/sprint-{SPRINT_ID}-{BRANCH_NAME}
```

---

### Phase 4 — Branch writing (sub-agent)

Launch a sub-agent of type `branch-writer` with the following prompt:

```
Branch name: {BRANCH_NAME}
Sprint ID: {SPRINT_ID}
Slack channel: sprint-{SPRINT_ID}-{BRANCH_NAME}
Target repos: {TARGET_REPOS}
Sprint memory template content:
---
{read from templates/sprint-memory-template.md}
---
Guidelines template content:
---
{read from templates/guidelines-template.md}
---
Protocol files:
{For each file discovered in protocols/, include an entry:}
---
path: {relative path from protocols/, e.g. "cursor/skills/develop/SKILL.md"}
content:
{FILE_CONTENT}
---
Guide content:
---
{GUIDE_CONTENT}
---
```

The `branch-writer` agent handles branch creation, file writing, per-repo content tailoring, and pushing to GitHub. See `.claude/agents/branch-writer.md` for its full instructions.

---

### Final step

When the branch writer sub-agent returns, show Patrick the summary and say:

"Sprint `{SPRINT_ID}` is live. Branch `{BRANCH_NAME}` pushed to {repo list}. Slack channel `sprint-{SPRINT_ID}-{BRANCH_NAME}` is open. Ready when you are."
