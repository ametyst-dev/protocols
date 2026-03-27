# Sprint {SPRINT_ID} — {SPRINT_NAME}

## Meta
channel: {SLACK_CHANNEL}
agent_name: {REPO_NAME}
sprint_goal: {SPRINT_GOAL}

## Context
{What the agent needs to know about this repo for this sprint — relevant files, current state, dependencies.}

## Architecture Decision Records
{Cross-repo decisions that every agent must respect. Only include ADRs that are relevant to THIS repo. If no cross-repo decisions affect this repo, write "No cross-repo ADRs for this repo."}

### ADR-001 — {Decision title}
- **Context:** {why this decision is needed}
- **Decision:** {what we decided}
- **Alternatives:** {what we considered and rejected, with reasons}
- **Consequences for this repo:** {specific impact on this repo's implementation}

## Test strategy
{Test plan for this repo in this sprint. What gets tested, at which level.}

- **Test command:** {the command to run tests — e.g. `npm test`, `pytest`}
- **Unit tests:** {what business logic to test in this repo}
- **Integration tests:** {which endpoints or cross-module interactions to test}
- **Not tested:** {what is out of scope}
- **Credentials:** {reference to .env.test or sandbox keys}

## Macro steps
1. {Macro step 1}
2. {Macro step 2}
3. {Macro step 3}

## Constraints
- {Technical constraints and patterns to follow}
- {Things explicitly NOT to do}

## Definition of done
- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] All tests pass
