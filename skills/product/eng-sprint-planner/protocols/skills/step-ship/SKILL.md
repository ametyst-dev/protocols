---
description: Ship the current macro step — runs final test suite, pushes the sub-branch, opens a PR to the sprint branch, checks if documentation needs updating, and notifies Slack. Use after step-review confirms DONE.
argument-hint: ""
allowed-tools: Read, Write, Glob, Grep, Bash(git status *), Bash(git log *), Bash(git branch *), Bash(git diff *), Bash(gh pr view *), Bash(gh pr diff *), mcp__slack__slack_post_message
---

## step-ship

You are a repo agent running in a product repo sub-branch. Your job is to ship the completed macro step: run tests, push, open a PR to the sprint branch, and notify Slack.

---

## GIT REMOTE GUARDRAIL — READ THIS FIRST

**This skill does NOT have permission to execute write operations on git remotes.** Commands like `git push`, `gh pr create`, and `gh pr merge` are NOT in your allowed tools. When you need to execute them, Claude Code will show a native permission prompt to HIL. This is intentional — every remote write operation requires explicit human approval.

**Before requesting any remote write operation**, present to HIL exactly what you're about to do:
- Which command
- Which branch
- Which remote
- What the effect will be

Only proceed after HIL explicitly approves in the permission prompt.

---

## SLACK MESSAGE GUARDRAIL — READ THIS FIRST

Before sending any Slack message, read `brainstormings/sprint-*/slack-protocol.md`.

**Every Slack message you send MUST follow the format defined in that file exactly — character by character. No paraphrasing. No shortcuts. No variations.**

---

### Step 1 — Load state

Glob for `brainstormings/sprint-*/sprint-guidelines.md`. Resolve the sprint folder as `{SPRINT_FOLDER}`.

Read:
- `{SPRINT_FOLDER}/sprint-guidelines.md` → extract `channel`, `agent_name`, SPRINT_ID, macro steps list
- `{SPRINT_FOLDER}/sprint-memory.md` → find the current macro step N (last STEP_START sent)
- `{SPRINT_FOLDER}/slack-protocol.md` → load message format
- The repo's CLAUDE.md → find the `## Testing` section for the test command

Identify:
- Current sub-branch: `git branch --show-current` → should be `{SPRINT_BRANCH}/step-{N}`
- Sprint branch: remove `/step-{N}` from the sub-branch name → `{SPRINT_BRANCH}`

If you are not on a sub-branch (no `/step-{N}` suffix) → tell HIL: "Not on a step sub-branch. Run step-plan first to create one." End the skill.

---

### Step 2 — Pre-flight checks

1. **Uncommitted changes:**
   ```bash
   git status --porcelain
   ```
   If there are uncommitted changes → tell HIL: "There are uncommitted changes. Please commit or discard them before shipping." End the skill.

2. **Run test suite:**
   Run the test command from `## Testing` in the repo's CLAUDE.md.
   - If tests **pass** → proceed
   - If tests **fail** → tell HIL: "Tests are failing. Fix them before shipping (run `/debug` if needed)." End the skill.

3. **Verify commits exist:**
   ```bash
   git log {SPRINT_BRANCH}..HEAD --oneline
   ```
   If no commits → tell HIL: "No commits on this sub-branch. Nothing to ship." End the skill.

---

### Step 3 — Push sub-branch

Present to HIL:
> "Ready to push sub-branch `{SPRINT_BRANCH}/step-{N}` to remote. This uploads your local commits to GitHub. Approve?"

Then execute (HIL approves via Claude Code permission prompt):
```bash
git push origin {SPRINT_BRANCH}/step-{N}
```

---

### Step 4 — Open PR

Present to HIL:
> "Ready to open a PR: `{SPRINT_BRANCH}/step-{N}` → `{SPRINT_BRANCH}`. This creates a pull request for macro step {N} targeting the sprint branch (not main). Approve?"

Build the PR body from:
- Macro step title and description from `sprint-guidelines.md`
- Summary from `{SPRINT_FOLDER}/step/current-plan.md`
- Chunk completion summary from `{SPRINT_FOLDER}/step/current-execution.md`
- Test results (pass count)

Then execute (HIL approves via Claude Code permission prompt):
```bash
gh pr create \
  --base {SPRINT_BRANCH} \
  --head {SPRINT_BRANCH}/step-{N} \
  --title "[sprint-{SPRINT_ID}] Step {N}: {macro step title}" \
  --body "{PR body}"
```

Save the returned PR URL as `{PR_URL}`.

---

### Step 5 — Documentation check

Scan the files changed in this step:
```bash
git diff {SPRINT_BRANCH}..HEAD --name-only
```

If the changes touch:
- Public API endpoints → check if API docs or README need updating
- Configuration files → check if setup instructions need updating
- New dependencies → check if installation instructions need updating
- CLAUDE.md-referenced files (test command, project structure) → check if CLAUDE.md needs updating

If documentation updates are needed:
1. Make the changes
2. Commit: `[sprint-{SPRINT_ID}] step {N}: update documentation`
3. Push (HIL approves via permission prompt) — the PR auto-updates

If no documentation updates needed → proceed.

---

### Step 6 — Notify Slack

Post to Slack using `mcp__slack__slack_post_message`:

```
[STEP_DONE] sprint-{SPRINT_ID}
from: {agent_name}
to: domain-expansion
---
macro_step: {N}
summary: {2-3 sentence summary of what was accomplished}
pr_link: {PR_URL}
upgrades_proposed: none | {description of upgrade needed in repo X and why}
```

If upgrades are proposed, post a separate UPGRADE_PROPOSED message for each one (same format as in step-review).

Append to the Sent section of `{SPRINT_FOLDER}/sprint-memory.md`:
```
[DATETIME] [STEP_DONE] to: domain-expansion — macro step N done. PR: {PR_URL}
```

---

### Step 7 — Tell HIL

"Step {N} shipped. PR: {PR_URL}

Switch to domain-expansion and run `/sprint-delegate` to review the PR and advance to the next step."
