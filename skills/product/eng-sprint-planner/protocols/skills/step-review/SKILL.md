---
description: Review the output from Cursor after a macro step execution — reads the execution summary, evaluates correctness against the current plan, and either sends the agent back to Cursor with new instructions or confirms the step as done on Slack. Use after Cursor has finished executing a step and HIL has saved the summary to step/current-execution.md.
argument-hint: ""
allowed-tools: Read, Write, Glob, Grep, Bash(test *), Bash(npm *), Bash(npx *), Bash(pytest *), Bash(go test *), mcp__slack__slack_post_message
---

## step-review

You are a repo agent running in a product repo branch. Your job is to review what Cursor has done and decide if the step is complete.

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

---

### Step 1 — Load state

Glob for `brainstormings/sprint-*/sprint-guidelines.md`. If multiple results are returned, use the folder that contains a `sprint-memory.md` with recent activity — or ask HIL to confirm if still ambiguous. Save the resolved sprint folder path (e.g. `brainstormings/sprint-0005-my-feature`) as `{SPRINT_FOLDER}` — use this in all subsequent file paths.

Read:
- `{SPRINT_FOLDER}/sprint-guidelines.md` → extract `channel`, `agent_name`, `sprint_goal`, constraints. Extract SPRINT_ID from the `channel` field (e.g. channel `sprint-0005-my-feature` → SPRINT_ID `0005`). Extract the current macro step number N from `sprint-memory.md`.
- `{SPRINT_FOLDER}/step/current-plan.md` → the operational guide that was given to Cursor
- `{SPRINT_FOLDER}/step/queue.md` → the chunk queue (if it exists — may not exist for legacy plans)
- `{SPRINT_FOLDER}/step/current-execution.md` → the execution report written by Cursor (structured per-chunk if queue.md was used)
- `{SPRINT_FOLDER}/slack-protocol.md` → load the message format (see guardrail above)

If `{SPRINT_FOLDER}/step/current-execution.md` does not exist → tell HIL: "No execution summary found. Please save Cursor's output to `{SPRINT_FOLDER}/step/current-execution.md` and run step-review again." End the skill.

If `{SPRINT_FOLDER}/step/current-plan.md` does not exist → tell HIL: "No current plan found. Run step-plan first." End the skill.

---

### Step 2 — Evaluate

Compare `current-execution.md` against `current-plan.md` (and `queue.md` if it exists).

**If chunk-based execution was used:** evaluate each chunk individually — it's easier to spot which chunk had issues. Check:
- Did each chunk produce the expected output (see "Done when" in queue.md)?
- Were all chunks completed, or did execution stop early?

**Test verification:**
- For each test chunk: check the `## Test results` section in `current-execution.md`. Did all tests pass?
- Run the test command yourself (from `**Test command:**` in `queue.md` or `## Testing` in the repo's CLAUDE.md) to independently verify tests pass. Do not trust the execution report blindly.
- If tests fail → this is a REDO, regardless of whether the implementation looks correct.

**Commit verification:**
- For each chunk with `**Commit point:**`: verify that `**Committed:** yes` appears in the execution report.
- If commits are missing for completed commit points, flag it — but this alone is not a REDO (commits can be done during ship phase).

**Quality checklist:**
After verifying correctness and tests, scan the changed files for:
- Unused imports or dead code introduced by this step
- Debug code left behind (console.log, print statements, TODO/FIXME that should have been resolved)
- Hardcoded values that should be config or environment variables
- Security: unvalidated user input, exposed secrets, SQL/command injection
- Patterns: do the changes follow the existing patterns in the repo? (naming conventions, file structure, error handling)

Flag quality issues to HIL but use judgment: minor issues (an extra import) can be noted without triggering a REDO; security issues or broken patterns should trigger a REDO.

**QA lens — after quality checklist:**
- **Coverage first.** Are the tests covering the main user scenarios? A test that checks "happy path only" is incomplete. At minimum: happy path, one error case, one edge case per feature.
- **Tests must be realistic.** Tests that mock everything and test nothing are worse than no tests. If the test doesn't exercise real logic, flag it.
- **Never trust unverified claims.** If `current-execution.md` says "all tests pass" but you haven't run them yourself, run them. The test command is in `queue.md` or the repo's CLAUDE.md.
- **Focus on what breaks in production.** Don't flag cosmetic test issues. Flag: missing error handling tests, missing boundary tests, tests that would pass even if the feature was deleted.

If the QA lens reveals gaps in test coverage, these are REDO-worthy only if they cover critical paths. For non-critical gaps, note them as suggestions for the next step, not blockers.

**For all executions (chunked or legacy):**
- Was the correct output produced?
- Were the right files touched as specified in the plan?
- Were the constraints from `sprint-guidelines.md` respected?
- Were the ADRs from `sprint-guidelines.md` respected?
- Are there obvious gaps, regressions, or deviations?
- Are there any changes made that suggest upgrades needed in other repos?

---

### Step 3 — HIL gate

Present to HIL:
1. The macro step that was executed
2. Your evaluation — what matches, what doesn't, any concerns
3. Any upgrade proposals you identified (changes in this repo that require action in another repo)

Then ask one of:
- **If work looks correct**: "This step looks good. Approve to mark as done and notify Slack."
- **If work has issues**: "This step has issues: {list}. I'll generate corrected Cursor instructions if you approve."

---

### Step 4a — REDO (if issues found or HIL rejects)

**Triage the issue type:**

- **Implementation error** (wrong file, missing import, field mismatch, incomplete wiring) → proceed with correction instructions below.
- **Bug** (test fails with unexpected behavior, logic error, runtime crash, race condition) → write `{SPRINT_FOLDER}/step/debug-context.md` and suggest debug:

  ```markdown
  # Debug Context

  **Macro step:** {N}
  **Failed chunks:** {list of chunk numbers and titles}
  **Test output:**
  ```
  {stderr/stdout from the test run that failed}
  ```
  **Expected:** {what the plan said should happen}
  **Actual:** {what actually happened}
  **Hypothesis:** {if you have an idea of what went wrong, state it here}
  ```

  Tell HIL: "This looks like a bug, not a simple implementation error. Run `/debug` — context is preloaded in `{SPRINT_FOLDER}/step/debug-context.md`."

  End the skill — debug takes over from here.

**For implementation errors, generate focused correction instructions.** Address only what was wrong — do not repeat what was already done correctly.

**If chunk-based execution was used:**
- Identify which specific chunk(s) had issues.
- In `{SPRINT_FOLDER}/step/queue.md`, reset only the failed chunk(s) to `**Status:** pending` and update their instructions with the corrections. Leave `done` chunks as-is.
- Delete the failed chunk sections from `current-execution.md` so Cursor rewrites them.
- When Cursor re-runs develop, it will skip done chunks and only re-execute the pending ones.

**If legacy execution (no queue.md):**
Overwrite `{SPRINT_FOLDER}/step/current-plan.md` with the new instructions.

Append to the Sent section of `{SPRINT_FOLDER}/sprint-memory.md`:
```
[DATETIME] [STEP_REDO] to: cursor — macro step N sent back for correction. {one-line reason}
```

Tell HIL: "Corrected instructions written to {SPRINT_FOLDER}/step/current-plan.md. Hand them to Cursor, then run step-review again when done."

End the skill.

---

### Step 4b — DONE (if approved)

**The step is approved but NOT shipped yet.** Step-review no longer sends STEP_DONE directly — that responsibility moves to `step-ship`, which handles push, PR creation, and Slack notification.

1. Note any upgrade proposals identified during review — step-ship will include them in the STEP_DONE message.

2. Write the upgrade proposals (if any) to `{SPRINT_FOLDER}/step/upgrades.md`:
```markdown
# Upgrade Proposals — Step {N}

- target_repo: {repo-name}
  description: {what to do}
  reason: {why}
```
If no upgrades: skip this file.
```

3. **Do NOT cleanup step files yet** — step-ship needs `current-plan.md` and `current-execution.md` to build the PR body. Cleanup happens after step-ship completes.

4. Tell HIL: "Step N review approved. Run `/step-ship` to push and open the PR."
