# Agentic Development Guide

How to build product features using AI agents. This guide is for non-technical operators who use Claude Code and Cursor as their "hands" to write code.

---

## What you need

| Tool | What it does | Where to get it |
|---|---|---|
| **Claude Code** | AI agent in the terminal — plans, reviews, coordinates | Already installed on your machine |
| **Cursor** | AI code editor — writes the actual code | Already installed on your machine |
| **GitHub** | Where the code lives | You have access to the repos |
| **Slack** | Where agents communicate during a sprint | You're in the workspace |

---

## The big picture

Building a feature follows this loop:

```
1. BRAINSTORM       →  define what to build (personal agent, Claude Code)
2. SPRINT PLANNING  →  break the feature into macro steps (personal agent, Claude Code)
3. FOR EACH STEP:
   a. STEP PLAN     →  Claude Code analyzes the repo and writes a detailed
                        operational plan for this specific step (product repo, Claude Code)
   b. DEVELOP       →  Cursor reads the plan and writes the code (product repo, Cursor)
   c. STEP REVIEW   →  Claude Code checks what Cursor did (product repo, Claude Code)
   d. SHIP          →  push code and open PR (product repo, Claude Code)
4. COORDINATE       →  merge PRs, advance to next step (personal agent, Claude Code)
5. CLOSE            →  final review and merge to main (personal agent, Claude Code)
```

**The key thing to understand:** Sprint planning (step 2) creates a *high-level* plan with macro steps. But before Cursor writes any code, Claude Code does a *second, detailed* planning pass inside the repo for each step (step 3a). This is where the agent reads the actual codebase, understands the current state, and writes specific instructions for Cursor. Cursor never works from the high-level sprint plan — it always gets a detailed, repo-specific operational guide.

You are the **Human In the Loop (HIL)**. The agents propose, you approve. Nothing ships without your OK.

---

## Phase 1 — Brainstorm

**Where:** Claude Code, open in the `personal agent` folder

**What to do:**
1. Type `/ops-brainstorm` and press Enter
2. The agent will ask you:
   - **Which reasoning mode** — pick `product-builder` (it's designed for defining features)
   - **Topic** — describe what you want to build in plain language
   - **Output path** — where to save the result (e.g. `areas/product/brainstorms/`)
3. The agent enters **plan mode** — it proposes a brainstorming plan. Review it, give feedback, approve when ready.
4. The agent works interactively — it asks you questions, searches for data, proposes ideas. This is a conversation. Push back, redirect, add context.
5. At the end, the agent writes a structured output file with findings, conclusions, and next steps.

**Your output:** A brainstorm file (e.g. `areas/product/brainstorms/my-feature.md`) that becomes the input for sprint planning.

**Tips:**
- Be specific about the problem you're solving and for whom
- If the agent suggests connections to other data (calls, Notion, etc.), say yes — it enriches the analysis
- The brainstorm file should have a `## Sprint input` section with: goal, repos involved, constraints, key decisions

---

## Phase 2 — Sprint Planning

**Where:** Claude Code, open in the `personal agent` folder

**What to do:**
1. Type `/sprint-planner` and press Enter
2. The agent asks for parameters:
   - **Brainstorm input** — path to your brainstorm file from Phase 1
   - **Target repos** — which GitHub repos are involved (e.g. `ametyst-dev/core, ametyst-dev/api`)
   - **Branch name** — a short name for the feature (e.g. `onboarding-v2`)
   - **Sprint ID** — a 4-digit number (e.g. `0005`)
   - **Context depth** — how deep to analyze the repos (start with `2`)
3. The agent loads context from the repos (automatic, takes a minute)
4. The agent enters **plan mode** and proposes a sprint plan:
   - Overview of what will be built
   - Steps in order (Step 1, Step 2, ...) with what each step does and which repos it touches
   - Architecture decisions
   - Test strategy
5. **This is the most important review.** Read every step carefully. Ask questions. Request changes. The plan is the foundation — if it's wrong, everything downstream is wrong.
6. Once you approve, the agent writes the plan to branches in each repo and creates a Slack channel for coordination.

**Your output:** A sprint is live — branches created, Slack channel open, plan files deployed to each repo.

**What the agent will ask you to do manually:**
- Create the Slack channel (it can't do this itself) — just create it with the name the agent gives you and confirm

---

## Phase 3 — Execute steps (repeat for each step)

This is where code gets written. **Each macro step goes through its own full cycle** of planning, development, review, and shipping. You repeat this cycle for every step in the sprint.

> **Why a second planning phase?** The sprint plan (Phase 2) says *what* each step should accomplish at a high level. But before Cursor can write code, Claude Code needs to read the actual codebase, understand the current state of the files, and translate the high-level step into specific, line-by-line instructions. This is what `/step-plan` does — it's a deep, repo-specific planning pass that happens *inside* each product repo, *before* Cursor touches anything.

### 3a. Plan the step — `/step-plan` (Claude Code, in the product repo)

**Where:** Claude Code, open in the **product repo** (not personal agent)

**What to do:**
1. Type `/step-plan` and press Enter
2. The agent checks Slack for coordination signals, then identifies the next macro step to work on
3. It enters **plan mode** — this is a full analysis phase where the agent:
   - Reads the actual code files that will be changed
   - Understands current implementations, patterns, and dependencies
   - Translates the high-level sprint step into a detailed operational guide
   - Breaks the guide into **chunks** — small, self-contained tasks with clear boundaries
4. The agent presents the full plan to you. Review it carefully:
   - Does each chunk make sense?
   - Is the order logical?
   - Are there any concerns you want to raise?
5. Give feedback or approve. The agent iterates until you say OK.
6. Once approved, the agent writes the plan to disk and creates a sub-branch for this step.

**Your output:** Two files are ready for Cursor:
- `current-plan.md` — the full operational guide
- `queue.md` — the plan split into executable chunks

**What the agent tells you next:** "Open Cursor and hand it to Cursor."

---

### 3b. Develop — `develop` (in Cursor)

**Where:** Cursor, open in the **same product repo**

**What to do:**
1. Open Cursor in the repo
2. Tell Cursor: "Run the develop skill" (or reference the plan file)
3. Cursor reads the chunk queue and executes chunks one by one — writing code, running tests, committing
4. **You don't need to do anything during execution.** Cursor works autonomously.
5. When Cursor finishes, it writes an execution report to `current-execution.md`

**Your output:** Code is written, tests run, execution report ready.

**If Cursor gets stuck or asks a question:** Answer based on the plan. If you don't know, say "I'll check with the Claude Code agent" and go back to Claude Code.

---

### 3c. Review the step — `/step-review`

**Where:** Claude Code, open in the **product repo**

**What to do:**
1. Type `/step-review` and press Enter
2. The agent reads the execution report, compares it against the plan, and runs tests independently
3. It presents a verdict:
   - **Looks good** → approve to mark as done
   - **Has issues** → the agent writes corrected instructions, you go back to Cursor (repeat 3b)
4. If approved, the agent may identify **upgrade proposals** — changes needed in another repo because of what was built here. These get communicated via Slack.

**Your output:** Step is validated and ready to ship.

---

### 3d. Ship the step — `/step-ship`

**Where:** Claude Code, open in the **product repo**

**What to do:**
1. Type `/step-ship` and press Enter
2. The agent runs final tests, then asks for your approval to:
   - **Push the branch** to GitHub
   - **Open a Pull Request** to the sprint branch
3. You approve each action when Claude Code shows the permission prompt
4. The agent notifies Slack that the step is done, including the PR link

**Your output:** A PR is open on GitHub, Slack is notified.

---

## Phase 4 — Coordinate across repos

**Where:** Claude Code, open in the `personal agent` folder

**What to do:**
1. Type `/sprint-delegate` and press Enter
2. The agent reads all messages from the sprint Slack channel and presents a status report:
   - Which repos finished their step
   - Which PRs are ready for review
   - Any upgrade proposals between repos
   - Whether all repos are done with the current step
3. For each pending decision, the agent asks for your approval:
   - **PR review + merge** — the agent suggests running `/pr-review [PR URL]` first, then merging
   - **Upgrades** — accept or reject cross-repo changes
   - **Advance to next step** — signal all repos to start the next step
4. The agent executes your decisions: merges PRs, sends Slack signals, advances the sprint.

**Your output:** PRs merged, next step signaled, sprint advances.

**Repeat:** After the delegate advances the sprint, go back to Phase 3 for each repo that has work in the next step.

---

## Phase 5 — Close the sprint

**Where:** Claude Code, open in the `personal agent` folder

**What to do:**
1. When all steps are done, tell the sprint delegate: "close sprint"
2. The agent generates a closing report with:
   - Delivery summary
   - Product improvement suggestions → synced to the backlog
   - Workflow improvement suggestions → saved to skill memory for future sprints
3. The agent opens a **final PR** from the sprint branch to `main` in each repo
4. **You merge the final PR on GitHub** (the agent cannot do this — it's a safety measure)

**Your output:** Feature is on `main`, sprint is closed, learnings captured.

---

## Quick reference — command cheat sheet

| Phase | Where | Command | What it does |
|---|---|---|---|
| Brainstorm | personal agent | `/ops-brainstorm` | Define what to build |
| Plan sprint | personal agent | `/sprint-planner` | Break it into steps, create branches |
| Plan step | product repo | `/step-plan` | Detail the next step for Cursor |
| Develop | Cursor | `develop` | Write the code |
| Review step | product repo | `/step-review` | Validate what Cursor wrote |
| Ship step | product repo | `/step-ship` | Push and open PR |
| Coordinate | personal agent | `/sprint-delegate` | Merge PRs, advance sprint |
| Review PR | personal agent | `/pr-review [URL]` | Review a PR before merging |
| Debug | product repo | `/debug` | Fix errors during execution |
| Close sprint | personal agent | `/sprint-delegate` | "close sprint" to finish |

---

## When something goes wrong

### Tests fail during develop
Cursor will try to fix them (up to 2 attempts). If it can't, it stops and writes a partial report. Run `/step-review` — the review agent will detect the failure and either write corrected instructions or suggest running `/debug`.

### Debug mode
If `/step-review` identifies a bug (not a simple implementation error), it writes a debug context file and tells you to run `/debug`. The debug agent:
1. Reads the error and forms hypotheses
2. Writes an instrumentation plan (adds debug logs)
3. You hand the plan to Cursor → Cursor adds the logs
4. You reproduce the error and share the output
5. The agent identifies the root cause and writes a fix plan
6. You hand the fix plan to Cursor → Cursor fixes it
7. Max 2 investigation rounds — if still stuck, the agent escalates to you with everything it knows

### Cross-repo issues
If a change in repo A requires a change in repo B, the review agent proposes an **upgrade**. The sprint delegate receives it, asks for your approval, and routes it to repo B. This is automatic — you just approve or reject.

---

## Key concepts

**HIL (Human In the Loop):** You. Every important decision goes through you. Agents propose, you approve.

**Macro step:** A major milestone in the sprint (e.g. "Set up database schema", "Build API endpoints"). Each macro step maps to one cycle of plan → develop → review → ship.

**Chunk:** A macro step broken into smaller tasks that Cursor executes one at a time. Chunks have clear boundaries and test points.

**Upgrade:** A change needed in one repo because of work done in another repo. Coordinated automatically through Slack.

### Branches — how code is organized

Think of branches like parallel drafts of the same document. The `main` branch is the "official" version of the code — what's live in production. You never write directly on `main`. Instead, you create branches to work on, and when the work is done and reviewed, you merge it back.

The sprint uses **two levels of branches:**

```
main                          ← the official, production code
 └── onboarding-v2            ← sprint branch (one per feature)
      ├── onboarding-v2/step-1    ← sub-branch for step 1
      ├── onboarding-v2/step-2    ← sub-branch for step 2
      └── onboarding-v2/step-3    ← sub-branch for step 3
```

**Sprint branch** (e.g. `onboarding-v2`):
- Created once during sprint planning (Phase 2)
- This is where all the work for the feature accumulates
- Each completed step gets merged here via a Pull Request
- At the end of the sprint, this entire branch gets merged into `main`

**Sub-branch** (e.g. `onboarding-v2/step-1`):
- Created automatically by `/step-plan` at the start of each macro step
- This is where Cursor writes code for that specific step
- When the step is done and reviewed, `/step-ship` opens a Pull Request from the sub-branch into the sprint branch
- After the PR is merged, the sub-branch is deleted — it has served its purpose

**The flow of code through branches:**

```
1. Cursor writes code on       onboarding-v2/step-1
2. /step-ship opens a PR:      onboarding-v2/step-1 → onboarding-v2
3. Sprint delegate merges PR:  step-1 work is now on the sprint branch
4. Next step starts on:        onboarding-v2/step-2
   ...repeat for each step...
5. Sprint closes — final PR:   onboarding-v2 → main
6. You merge on GitHub:        feature is live
```

**Why this structure?** It keeps each step isolated. If step 2 goes wrong, it doesn't break what step 1 already delivered. And it gives you a review point (the PR) before any code reaches the sprint branch.

**Pull Request (PR):** A request to merge code from one branch into another. It shows all the changes and lets you (or the agents) review them before approving. PRs are the gates between branches — nothing moves without one.

---

## Tips for success

1. **The brainstorm quality determines everything.** Spend time here. Be specific about what you want, why, and for whom.
2. **Read the sprint plan carefully.** If a step doesn't make sense to you, ask the agent to explain it differently. You don't need to understand the code, but you should understand what each step accomplishes.
3. **Don't rush approvals.** When the agent asks "approve?", take a moment to read what it's proposing. A wrong approval costs more time than a question.
4. **Trust the agents but verify.** The review agent runs tests independently. The delegate checks cross-repo consistency. Let them do their job.
5. **When in doubt, ask.** Type your question in natural language. The agents are designed to explain things to non-technical operators.
6. **One step at a time.** Don't try to run multiple steps in parallel across repos unless the sprint delegate tells you it's safe.
