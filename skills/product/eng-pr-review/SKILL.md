---
description: "Review a pull request by analyzing its diff for security, pattern compliance, completeness, and test coverage. Use whenever Patrick says 'review PR', 'review this PR', or provides a PR URL to review."
argument-hint: "[PR URL]"
allowed-tools: Read, Glob, Grep, Bash(gh pr view *), Bash(gh pr diff *), Bash(gh pr checks *), Bash(gh api *)
---

## PR Review

You review a pull request by analyzing its full diff. You look for security issues, pattern violations, completeness gaps, and test coverage holes. You produce a structured verdict that HIL uses to decide whether to merge.

---

### Step 1 — Load PR

If no PR URL was provided as argument, ask HIL for it.

Fetch PR metadata:
```bash
gh pr view {PR_URL} --json title,body,baseRefName,headRefName,additions,deletions,changedFiles,url
```

Fetch the full diff:
```bash
gh pr diff {PR_URL}
```

Fetch CI check status (if available):
```bash
gh pr checks {PR_URL}
```

Present a brief summary to HIL:
- PR title
- Base ← Head (e.g. `my-feature` ← `my-feature/step-1`)
- Lines changed: +{additions} -{deletions}
- Files changed: {count}

---

### Step 2 — Review the diff

Analyze the entire diff through four lenses:

**Security checklist:**
- Input validation: is user/external input validated before use?
- Secrets: are API keys, passwords, tokens, or credentials hardcoded?
- Injection: SQL injection, command injection, XSS vectors?
- Auth: do new endpoints have proper authentication/authorization guards?
- Dependencies: are new dependencies from trusted sources with no known vulnerabilities?

**Pattern compliance:**
- Do the changes follow naming conventions used elsewhere in the repo?
- Is error handling consistent with the rest of the codebase?
- Is file/folder structure consistent with existing patterns?
- Are imports organized the same way as other files?

**Completeness:**
- Does the diff cover everything the PR body says it covers?
- Are there files touched that shouldn't be? (scope creep)
- Are there files that should be touched but aren't? (incomplete work)
- Are there TODO/FIXME/HACK comments that should have been resolved?

**Test coverage:**
- Are new features/endpoints covered by tests?
- Do tests cover happy path + at least one error case?
- Are tests realistic (not just mocking everything)?
- Do test assertions actually verify meaningful behavior?

---

### Step 3 — Produce verdict

Present the review to HIL in this format:

```markdown
## PR Review — {PR title}
**Verdict:** approved | changes_requested
**PR:** {PR_URL}
**Summary:** {1-2 sentences on overall quality}

### Critical issues (must fix before merge)
- [{file}:{line}] {description of the issue}

### Warnings (should fix, not blocking)
- [{file}:{line}] {description}

### Suggestions (optional improvements)
- {suggestion}

### Test coverage assessment
- {what's well tested}
- {what's missing}
```

**Severity rules:**
- **Critical** (blocks merge): security vulnerabilities, broken auth, data loss risk, completely missing tests for critical paths
- **Warning** (should fix): pattern violations, incomplete error handling, weak test assertions
- **Suggestion** (nice to have): style improvements, refactoring opportunities, additional test cases

If verdict is **approved**:
> "This PR looks good. You can merge it from the sprint-delegate or directly on GitHub."

If verdict is **changes_requested**:
> "This PR needs changes before merging. The critical issues above must be fixed. You can send corrections back to the repo agent or fix them directly."
