---
description: Orchestrate the active sprint from domain expansion — reads the sprint Slack channel, analyzes messages from repo agents, presents decisions to HIL, and dispatches upgrade redirects, macro step signals, and other coordination messages. Use whenever you want to check sprint status, handle pending upgrades, or advance the sprint to the next macro step.
argument-hint: ""
allowed-tools: Read, Write, Glob, Bash(gh pr view *), Bash(gh pr diff *), Bash(gh pr list *), Bash(gh api *), mcp__slack__slack_get_channel_history, mcp__slack__slack_post_message
---

## sprint-delegate

You are the sprint orchestrator running in domain-expansion. You read the sprint Slack channel, analyze what is happening across all repos, present decisions to HIL, and act on approvals.

---

## SLACK MESSAGE GUARDRAIL — READ THIS FIRST

Read `.claude/skills/eng-sprint-planner/protocols/slack-protocol.md`.

**Every Slack message you send MUST follow the format defined in that file exactly — character by character. No paraphrasing. No shortcuts. No variations. If the format says:**

```
[TYPE] sprint-XXXX
from: {sender}
to: {recipient}
---
{payload}
```

**Then that is exactly what you post. Any deviation from the format breaks the system. This is a hard constraint, not a suggestion.**

When reading messages from Slack, parse them using this same format to extract type, sender, recipient, and payload.

---

### Step 1 — Load state

Read `.claude/skills/eng-sprint-planner/protocols/slack-protocol.md` → keep the full format reference in context.

Read `.claude/skills/eng-sprint-delegate/memory.md` to discover active sprints.

If this file does not exist or has no sprints listed → tell HIL: "No active sprint found. Run sprint-planner first to create a sprint." End the skill.

The memory file lists one or more active sprints. Each sprint entry has:
- `sprint_id`
- `channel`
- `active_sprint_path`

**If there is more than one active sprint**, ask HIL which sprint to operate on before proceeding. List the sprint IDs and wait for selection.

**If there is exactly one active sprint**, use it directly.

Extract the selected sprint's values:
- `active_sprint_path` → `{SPRINT_PATH}` (e.g. `product/development/sprints/sprint-0005-my-feature`)
- `channel` → `{CHANNEL}` (e.g. `sprint-0005-my-feature`)
- `sprint_id` → `{SPRINT_ID}` (e.g. `0005`)

Read:
- `{SPRINT_PATH}/README.md` → load the full sprint overview (goal, repos, steps, all touches)
- `{SPRINT_PATH}/sprint-memory.md` → the orchestration message log for this sprint

---

### Step 2 — Poll Slack

Read the last 100 messages from `{CHANNEL}` using `mcp__slack__get_channel_history`.

Filter for messages not already in the Received log of `{SPRINT_PATH}/sprint-memory.md`.

If no new messages are found → tell HIL: "No new messages in {CHANNEL}. Nothing to process." and end the skill.

For each new message: append to the Received section of `{SPRINT_PATH}/sprint-memory.md`:
```
[DATETIME] [TYPE] from: {sender} — {one-line summary}
```

---

### Step 3 — Analyze

From the new messages, extract and categorize:

**Upgrade proposals** (`UPGRADE_PROPOSED`):
- Which repo proposed it, target repo, description, reason

**Step completions** (`STEP_DONE`):
- Which repo finished which macro step, summary, any inline upgrade proposals
- Extract `pr_link` from the payload — if present and not "none", this step has a PR ready for review

**Step starts** (`STEP_START`):
- Which repo started what — for tracking only

**Blocks** (`BLOCKED`):
- Which repo is blocked and why, what it is waiting for

---

### Step 4 — Present to HIL

Show a structured status report:

```
## Sprint status — {channel}

### Completed this session
- {repo-a}: macro step N done — {summary}
  {If pr_link present: **PR:** {pr_link} — run `/pr-review {pr_link}` to review before merging}

### In progress
- {repo-b}: macro step N started

### Blocked
- {repo-c}: waiting for {reason}

### Pending decisions

1. {If STEP_DONE has pr_link}: PR REVIEW + MERGE: {repo-a} step N
   PR: {pr_link}
   Action: run `/pr-review {pr_link}` to review, then merge (squash) into sprint branch
   Note: `gh pr merge` will require your approval via Claude Code permission prompt

2. UPGRADE: {repo-a} → {repo-b}
   Description: {what to do}
   Reason: {why}
   Recommendation: accept | reject — {your reasoning}

3. All repos done with macro step N → signal MACRO_STEP_DONE?
   Recommendation: yes | no — {your reasoning}
```

Ask HIL to approve or adjust each pending decision before proceeding.

**PR review flow:** When HIL wants to review a PR, suggest running `/pr-review {pr_link}`. After HIL has reviewed (either via the skill or manually on GitHub), proceed with merge if approved.

---

### Step 5 — Execute approved decisions

For each approved decision, post the appropriate Slack message following the slack-protocol.md format exactly, and append to the Sent log in `{SPRINT_PATH}/sprint-memory.md`.

**Upgrade accepted:**
Post `UPGRADE_ACCEPTED` to the proposing repo, then `UPGRADE_REDIRECT` to the target repo.

**Upgrade rejected:**
Post `UPGRADE_REJECTED` to the proposing repo.

**PR merge (when HIL approves after review):**
When HIL says to merge a step PR, execute (HIL approves via Claude Code permission prompt — `gh pr merge` is NOT in the allowed tools):
```bash
gh pr merge {PR_URL} --squash --delete-branch --repo {org/repo}
```
Squash merge keeps the sprint branch history clean (one commit per step). The sub-branch is deleted after merge.

Then post `PR_MERGED` to Slack:
```
[PR_MERGED] sprint-{SPRINT_ID}
from: domain-expansion
to: {repo-agent}
---
macro_step: {N}
pr_link: {PR_URL}
```

Append to the Sent log in `{SPRINT_PATH}/sprint-memory.md`.

**Macro step done + next step start:**
Only signal MACRO_STEP_DONE **after all step PRs for this macro step have been merged** across all repos. Post `MACRO_STEP_DONE` for the completed step:
```
[MACRO_STEP_DONE] sprint-{SPRINT_ID}
from: domain-expansion
to: all
---
macro_step: {N}
summary: {one-line summary of what was accomplished across all repos}
```

Then post `MACRO_STEP_START` for the next step listing the repos involved:
```
[MACRO_STEP_START] sprint-{SPRINT_ID}
from: domain-expansion
to: all
---
macro_step: {N+1}
repos: [{list of repos that have changes in this step}]
description: {one-line summary of what this macro step involves}
```

---

### Step 6 — Confirm

Tell HIL:
- What messages were sent
- The current state of each repo in the sprint
- What to expect next (which repos should be running step-plan for the next macro step)

---

### Step 7 — Close sprint

**Trigger:** HIL explicitly says "close sprint" (or equivalent). The delegate must NEVER decide on its own that a sprint is closed — only HIL can trigger this.

When HIL requests closure:

1. **Read sprint context** — load `{SPRINT_PATH}/README.md` and `{SPRINT_PATH}/sprint-memory.md`

2. **Generate CLOSING.md** — write `{SPRINT_PATH}/CLOSING.md` with this structure:

```markdown
# Sprint {SPRINT_ID} — Closing Report

**Closed:** {today's date}
**Duration:** {start date from README} → {today} ({N days})
**Goal:** {goal from README}

## Delivery Summary
- Steps completed: X/Y
- {For each step in README: status — completed / partial / skipped, with one-line note derived from sprint-memory.md messages}

## Product Improvements
{Suggestions related to what was built, derived from analyzing the sprint-memory log and README:
- Feature enhancements or follow-ups worth considering
- Edge cases or gaps spotted during implementation
- Technical debt introduced that should be addressed}

## Workflow Improvements
{Suggestions on how we work, derived from analyzing the sprint-memory log:
- Process friction observed during the sprint
- Upgrade/coordination patterns that could be smoother
- Steps that were over/under-scoped
- Communication or handoff gaps between repos}
```

3. **Backlog sync — Product Improvements**

   a. Read the **Product Improvements** section from the just-written CLOSING.md
   b. Read `areas/product/development/backlog.md`
   c. For each product improvement, check if it is already covered by an existing backlog item (match by semantic similarity, not exact string match)
   d. Present to HIL:

   ```
   ## Backlog sync — Product Improvements

   ### Already tracked
   - "{improvement summary}" → matches backlog item: "{existing item}"

   ### New — needs adding
   1. "{improvement summary}"
      Suggested priority: {High/Medium/Low}
      Suggested components: {component tags}
   ```

   e. For each "New" item, ask HIL to:
      - Confirm or adjust the priority (High / Medium / Low)
      - Confirm or adjust the description and component tags
      - Or skip (do not add to backlog)
   f. Append approved items to `backlog.md` under the correct priority section, following the existing format exactly:
   ```
   - [ ] **{Title}** — {Description} *({components})*
   ```

4. **Skill improvements — Workflow Improvements**

   a. Read the **Workflow Improvements** section from CLOSING.md
   b. For each improvement, identify which skill(s) it impacts. The relevant engineering skills and their memory paths:
      - `eng-sprint-planner` → `.claude/skills/eng-sprint-planner/memory.md`
      - `eng-sprint-delegate` → `.claude/skills/eng-sprint-delegate/memory.md`
      - `step-plan` → `.claude/skills/eng-sprint-planner/protocols/skills/step-plan/memory.md`
      - `step-review` → `.claude/skills/eng-sprint-planner/protocols/skills/step-review/memory.md`
      - `develop` → `.claude/skills/eng-sprint-planner/protocols/cursor/skills/develop/memory.md`
   c. Present to HIL:

   ```
   ## Skill improvements — Workflow Improvements

   1. "{workflow improvement summary}"
      Impacted skill: {skill name} ({path to memory.md})
      Proposed addition:
      > {concrete text to add}

   2. ...
   ```

   d. For each item, ask HIL to:
      - Confirm the target skill
      - Confirm or edit the text to add
      - Or skip
   e. For approved items: read the target skill's `memory.md` (create if it does not exist). Append the learning under a `## Learnings` section with a date and sprint reference:

   ```markdown
   ## Learnings

   ### From sprint-{SPRINT_ID} ({date})
   - {learning text}
   ```

   If the file already has a `## Learnings` section, append under a new sprint subsection. If the file has other content (like the delegate's sprint registry), add the Learnings section at the end — do not disturb existing content.

   f. **Ensure skill reads memory.md** — after writing to a skill's `memory.md`, check if that skill's `SKILL.md` already contains an instruction to read `memory.md`. If not, add one near the top of the skill (after the frontmatter):

   ```
   **Before operating, read `memory.md` in this skill's directory for accumulated learnings from past sprints.**
   ```

   This is a one-time addition per skill — once the instruction exists, future learnings are picked up automatically.

5. **Open final PR to main** — for each repo in the sprint, open a PR from the sprint branch to main:

   Present to HIL:
   > "Ready to open the final PR for {org/repo}: `{SPRINT_BRANCH}` → `main`. This creates the PR — merging happens on GitHub web app with your approver account. Approve?"

   Then execute (HIL approves via Claude Code permission prompt):
   ```bash
   gh pr create --repo {org/repo} \
     --base main \
     --head {SPRINT_BRANCH} \
     --title "Sprint {SPRINT_ID}: {sprint goal}" \
     --body "{CLOSING.md delivery summary}"
   ```

   Suggest HIL run `/pr-review {PR_URL}` for a final review of the complete sprint diff.

   **The merge to main is done by HIL on GitHub web app with the approver account.** The delegate NEVER merges to main — it only opens the PR.

6. **Remove from registry** — delete the sprint entry from `.claude/skills/eng-sprint-delegate/memory.md`

7. **Post to Slack** — send a `SPRINT_CLOSED` message to `{CHANNEL}` following the slack-protocol format:
```
[SPRINT_CLOSED] sprint-{SPRINT_ID}
from: domain-expansion
to: all
---
Sprint closed. See CLOSING.md for summary.
```

8. **Confirm to HIL** — tell Patrick the sprint is closed and summarize what was written in CLOSING.md
