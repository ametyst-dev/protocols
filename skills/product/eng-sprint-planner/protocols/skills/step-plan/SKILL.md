---
description: Start a new macro step cycle in this repo — polls Slack for incoming messages, identifies the next macro step to execute, expands it into a detailed operational guide for Cursor, and confirms with HIL before starting. Use at the beginning of each macro step cycle.
argument-hint: ""
allowed-tools: Read, Write, Glob, Grep, Bash(git checkout *), Bash(git pull *), Bash(git branch *), mcp__slack__slack_get_channel_history, mcp__slack__slack_post_message, EnterPlanMode, ExitPlanMode
---

## step-plan

You are a repo agent running in a product repo branch. Your job is to plan the next macro step for execution.

---

## SLACK MESSAGE GUARDRAIL — READ THIS FIRST

Before doing anything else, read `brainstormings/sprint-*/slack-protocol.md`.

**Every Slack message you send MUST follow the format defined in that file exactly — character by character. No paraphrasing. No shortcuts. No variations. If the format says:**

```
[TYPE] sprint-XXXX
from: {sender}
to: {recipient}
---
{payload}
```

**Then that is exactly what you post. Any deviation from the format breaks the system. This is a hard constraint, not a suggestion.**

When polling Slack, parse incoming messages using the same format. Filter by `to: {agent_name}` or `to: all`.

---

### Step 1 — Load state

**Ensure you are on the sprint branch.** Extract the sprint branch name from the folder structure (e.g. `brainstormings/sprint-0005-my-feature` → branch `my-feature`). Run:
```bash
git checkout {SPRINT_BRANCH}
git pull origin {SPRINT_BRANCH}
```
This ensures you have the latest code including merges from previous step PRs. If you are already on the sprint branch and it's up to date, this is a no-op.

Glob for `brainstormings/sprint-*/sprint-guidelines.md`. If multiple results are returned, use the folder that contains a `sprint-memory.md` with recent activity — or ask HIL to confirm if still ambiguous. Save the resolved sprint folder path (e.g. `brainstormings/sprint-0005-my-feature`) as `{SPRINT_FOLDER}` — use this in all subsequent file paths, never the glob pattern.

Read:
- `{SPRINT_FOLDER}/sprint-guidelines.md` → extract `channel`, `agent_name`, `sprint_goal`, and the full macro steps list. Extract SPRINT_ID from the `channel` field (e.g. channel `sprint-0005-my-feature` → SPRINT_ID `0005`). Extract `{SPRINT_BRANCH}` from the channel by removing the `sprint-{SPRINT_ID}-` prefix (e.g. `sprint-0005-my-feature` → `my-feature`).
- `{SPRINT_FOLDER}/sprint-memory.md` → understand what has already been sent and received. If this file does not exist, create it now:

```markdown
# Sprint Memory — {channel}

## Info
channel: {channel}
started: {TODAY_DATE}

## Message log

### Sent
<!-- format: [DATETIME] [TYPE] to: {recipient} — {one-line summary} -->

### Received
<!-- format: [DATETIME] [TYPE] from: {sender} — {one-line summary} -->
```

- `{SPRINT_FOLDER}/slack-protocol.md` → load the message format (see guardrail above)

---

### Step 2 — Poll Slack

Read the last **20** messages from `channel` using `mcp__slack__slack_get_channel_history`.

Filter for messages addressed `to: {agent_name}` or `to: all` that are not already in the Received log of `{SPRINT_FOLDER}/sprint-memory.md`.

For each new message: append to the Received section of `{SPRINT_FOLDER}/sprint-memory.md`:
```
[DATETIME] [TYPE] from: {sender} — {one-line summary}
```

Note any `UPGRADE_REDIRECT` messages not yet handled — these must be addressed before the next macro step.

---

### Step 3 — Enter plan mode and decide the next step

**Enter plan mode now** by calling `EnterPlanMode`. This prevents any file writes during analysis and brainstorming — you can only read, search, and reason. All writing happens after HIL approves and you exit plan mode.

From the Sent log in `{SPRINT_FOLDER}/sprint-memory.md`, identify which macro steps have already been marked as STEP_DONE.

Find the first macro step in `{SPRINT_FOLDER}/sprint-guidelines.md` that has NOT yet appeared as STEP_DONE in the Sent log.

If there are pending `UPGRADE_REDIRECT` items → treat them as the current step and plan those first.

If all steps are done → call `ExitPlanMode`, tell HIL: "All macro steps complete for this repo. Nothing left to plan." and end the skill.

**Early exit — no changes for this repo:**
If the identified macro step contains "No changes required in this repo" for this agent → do NOT expand a plan. Instead, call `ExitPlanMode` and tell HIL:
> "Step N has no changes for this repo. Should I send STEP_DONE to domain-expansion and move on?"

If HIL confirms → go directly to Step 5 (send STEP_DONE, skip plan writing and Cursor entirely). If HIL says no → end the skill.

---

### Step 3b — Codebase analysis (still in plan mode)

Before proposing the plan to HIL, use plan mode to deeply understand the current state of the code:

- Read the files that the macro step will touch (check `sprint-guidelines.md` Touches section)
- Understand current implementations, patterns, imports, and types
- Use Grep and Glob to find related code, dependencies, and usages
- Identify gaps between what the guidelines describe and the actual code state
- Note anything that could affect the plan (unexpected state, missing files, changed APIs)
- **Read the `## Testing` section of the repo's CLAUDE.md** (or root README) to know the test command, framework, and setup. If no `## Testing` section exists, flag it — this macro step may need a test setup chunk before any test chunks.
- **Read the `## Test strategy` and `## Architecture Decision Records` sections of `sprint-guidelines.md`** — these inform what to test and which cross-repo decisions to respect in the plan.

This analysis grounds your plan in reality rather than just the abstract guidelines.

---

### Step 3c — Implementation readiness self-check (still in plan mode)

Before presenting the plan to HIL, verify internally:

1. **ADR compliance** — does the plan respect all ADRs from `sprint-guidelines.md`? If an ADR says "communication via WebSocket", the plan must not introduce HTTP polling for the same purpose.
2. **File existence** — do the files the plan references actually exist in the codebase? (use Glob to verify). If a file doesn't exist, the plan must explicitly say "create new file" instead of "modify".
3. **Test feasibility** — does the repo have a `## Testing` section in CLAUDE.md with a working test command? If not, the first chunk must set up test infrastructure before any test chunks can run.
4. **Chunk dependency chain** — walk through the chunk order: does any chunk consume something that hasn't been created yet? If yes, reorder or merge chunks.

If you find inconsistencies, fix the plan silently before presenting. Do not present a plan with known issues to HIL.

---

### Step 4 — HIL gate: brainstorm and approve the plan in chat (still in plan mode)

**You are still in plan mode — no file writes are possible.** Propose the plan inline in chat.

Present to HIL:
- The macro step identified (number, title, any pending Slack upgrades)
- Key findings from your codebase analysis (current state, patterns observed, any surprises)
- The full operational guide as a **chat draft** — specific enough for a senior engineer using Cursor to execute without questions. Include:
  - What exactly to build or change
  - Which files to touch and how
  - Patterns and constraints from `sprint-guidelines.md` to respect
  - Expected output — what "done" looks like
  - **Consumer verification list** — if this step introduces or changes a function/API/method, list every consumer that calls it and what parameters it must pass. The engineer must verify each consumer is wired correctly after implementing.
  - **Decisions** — if this macro step involves architectural choices local to this repo (which pattern, which library, which approach), list them with a short rationale. Reference ADRs from `sprint-guidelines.md` for cross-repo decisions. Format:
    ```
    ## Decisions
    - {Decision}: {rationale}
    - ADR-001 applies: {how it affects this step}
    ```

Ask: "Does this plan look correct? Approve to write the file and send the start signal, or give feedback to adjust."

If HIL requests changes → revise the draft in chat and re-present. You can read additional files during iteration if needed. Repeat until approved.

Once HIL approves → call `ExitPlanMode` to unlock file writes, then proceed to **Step 4b**.

---

### Step 4b — Write the plan to disk

## HARD GATE — DO NOT SKIP THIS STEP UNDER ANY CIRCUMSTANCE

**The skill is NOT complete until `current-plan.md` AND `queue.md` exist on disk.** If you skip this step, Cursor has nothing to execute and the entire sprint cycle breaks. This is the most important step in the entire skill.

**Immediately after calling ExitPlanMode, your very next action MUST be a Write tool call.** Do not respond to HIL, do not summarize, do not do anything else first. Write the file.

Take the full operational guide that HIL approved in Step 4 and write it **verbatim** to:

```
{SPRINT_FOLDER}/step/current-plan.md
```

Overwrite the file completely. The content must be the exact plan as approved — do not summarize, shorten, or skip sections. This file is the handoff artifact that Cursor reads to execute the step.

**Verification:** After writing, read `{SPRINT_FOLDER}/step/current-plan.md` back and confirm it is non-empty and matches the approved plan. If it is empty or missing, write it again.

After writing and verifying the file, proceed to Step 4c.

---

### Step 4c — Write chunk queue

Take the approved plan from `current-plan.md` and split it into **self-contained chunks** that Cursor can execute one at a time. This is critical — Cursor struggles with long plans and performs much better when given small, focused tasks in sequence.

**Chunking rules:**
- Each chunk must be **fully self-contained** — Cursor reads ONLY the current chunk, never the others. Never write "added in previous chunk", "see above", or "as described earlier". Every chunk must state explicitly:
  - Exact file paths to read, create, or modify
  - Exact function/class/type names with their full signatures (e.g. `calculateFee(amount: number, rate: number): number` in `src/services/payment-service.ts`)
  - Exact import statements needed (e.g. `import { PaymentService } from '../services/payment-service'`)
  - If a chunk uses something from a previous chunk, restate it as if the developer has never seen the other chunks
- **Group data-dependent tasks** into the same chunk. If function A creates something that function B consumes and they are tightly coupled, keep them together.
- If a chunk would exceed ~80 lines of instructions, find a natural split point where outputs stabilize (e.g. after a type definition is complete, after a service is implemented but before consumers are wired).
- Each chunk must specify: which files to touch, what to do, what "done" looks like for that chunk.
- **Order chunks** so foundational work comes first: types/interfaces/schemas → implementations → wiring/integration → consumers/tests.

**Chunking quality check — before presenting to HIL:**
- **Every requirement crystal clear.** Each chunk's "Done when" must be a verifiable statement, not vague ("works correctly" is wrong; "POST /payments returns 201 with body `{id, amount, status}`" is right).
- **Zero tolerance for ambiguity.** If a chunk instruction could be interpreted two ways, rewrite it until there's only one interpretation. The developer must never need to guess intent.
- **Dependency order verified.** Walk through the chunks in sequence: does each chunk have everything it needs from previous chunks? If chunk 4 needs a type defined in chunk 2, is the type name, file path, and signature stated explicitly in chunk 4?
- **Realistic scope per chunk.** No chunk should require more than ~30 minutes of focused work. If it does, split it.

**Present the chunking to HIL** after the quality check. Show:
- How many chunks
- Title and scope of each chunk
- Why things are grouped this way (especially dependency grouping)

HIL can merge, split, or reorder chunks. Once approved, write to:

```
{SPRINT_FOLDER}/step/queue.md
```

**Chunk types and commit points:**

There are two types of chunks:
- **Implementation chunks** — write code, add files, modify config. These do NOT trigger a commit on their own.
- **Test chunks** — write and run tests that validate the preceding implementation chunks. These ARE commit points.

A **commit point** groups the test chunk with all preceding implementation chunks since the last commit point. When Cursor executes a test chunk and all tests pass, it commits everything from the last commit point up to and including the test chunk.

**Rules for test chunks:**
- Reference the `## Test strategy` from `sprint-guidelines.md` and the `## Testing` section from the repo's CLAUDE.md to decide what to test and how.
- A test chunk may validate 1 or more preceding implementation chunks (e.g. model + service + endpoint → tested together).
- Group integration tests with the chunks they validate — don't test a service before the endpoint that uses it is in place, if the test needs the endpoint.
- If a macro step has no testable output (e.g. documentation, config-only changes), the last chunk is still a commit point but with `**Test:** verify existing tests still pass` instead of new tests.

Format:
```markdown
# Chunk Queue

**Total chunks:** N
**Test command:** {from repo CLAUDE.md ## Testing section}
**Source plan:** {SPRINT_FOLDER}/step/current-plan.md

---

## Chunk 1 — [title]
**Status:** pending
**Files:** [list of files this chunk touches]
**Done when:** [1-line description of what "done" looks like]

[operational instructions for this chunk only — specific enough for a senior dev to execute without questions]

---

## Chunk 2 — [title]
**Status:** pending
**Files:** [list of files this chunk touches]
**Done when:** [1-line description]

[operational instructions]

---

## Chunk 3 — [title] [TEST]
**Status:** pending
**Type:** test
**Commit point:** chunks 1-3
**Files:** [test files to create or modify]
**Done when:** all tests pass
**Test:**
- Unit: [what to unit test]
- Integration: [what to integration test, if applicable]

[operational instructions for writing and running the tests]
```

After writing `queue.md`, proceed to Step 5. Do NOT touch any other files besides `current-plan.md`, `queue.md`, `sprint-memory.md`, and Slack.

---

### Step 5 — Create sub-branch, send STEP_START, update memory

1. **Create sub-branch** for this macro step:
```bash
git checkout -b {SPRINT_BRANCH}/step-{N}
```
This creates a sub-branch off the sprint branch. All work for this macro step happens here. The PR will merge this sub-branch back into the sprint branch (not main).

2. Post to Slack using `mcp__slack__slack_post_message` — follow the slack-protocol.md format exactly:

```
[STEP_START] sprint-{SPRINT_ID}
from: {agent_name}
to: domain-expansion
---
macro_step: {N}
branch: {SPRINT_BRANCH}/step-{N}
description: {one-line summary of what this agent is starting}
```

3. Append to the Sent section of `{SPRINT_FOLDER}/sprint-memory.md`:
```
[DATETIME] [STEP_START] to: domain-expansion — starting macro step N on branch {SPRINT_BRANCH}/step-{N}: {description}
```

4. Tell HIL: "Step N plan is live on branch `{SPRINT_BRANCH}/step-{N}`. Open {SPRINT_FOLDER}/step/current-plan.md and hand it to Cursor."

---

### Step 5 (early exit path) — Send STEP_DONE for no-changes step

If HIL confirmed the early exit in Step 3:

1. Post to Slack using `mcp__slack__slack_post_message`:

```
[STEP_DONE] sprint-{SPRINT_ID}
from: {agent_name}
to: domain-expansion
---
macro_step: {N}
summary: No changes required for this repo in this step.
pr_link: none
upgrades_proposed: none
```

2. Append to the Sent section of `{SPRINT_FOLDER}/sprint-memory.md`:
```
[DATETIME] [STEP_DONE] to: domain-expansion — macro step N: no changes required, step skipped.
```

3. Tell HIL: "Step N marked as done (no changes). Run step-plan again to move to the next step."
