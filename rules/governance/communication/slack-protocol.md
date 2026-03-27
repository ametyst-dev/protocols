# Slack Protocol — Governance

## Channel
- **#governance** (private)
- **Channel ID:** `C0AKQDJEFFH`

## When to post
An agent MUST post to #governance after any of these events:
1. **Notion fetch** — report what was queried and key results
2. **Notion write/update** — report what was created or changed
3. **HIL request** — ask Patrick or a specific person for input/approval
4. **Status update** — progress on a governance task or project

## Message formats

### After a Notion fetch
```
:mag: *Fetch — [Database name]*
Filtered by: [filter description]
Results: [count] entries found
[Optional: key highlights or summary]
```

### After a Notion write/update
```
:pencil2: *Update — [Database name]*
Entry: [entry title]
Changes: [field] -> [new value]
[Optional: link to Notion page]
```

### After a Notion create
```
:heavy_plus_sign: *Created — [Database name]*
Entry: [entry title]
Fields: [key fields set]
```

### HIL request (to everyone)
```
:raising_hand: *Input needed*
Context: [what the agent is working on]
Question: [specific question or decision needed]
```

### HIL request (to specific people)
```
:raising_hand: *Input needed — @[person]*
Context: [what the agent is working on]
Question: [specific question or decision needed]
```

### Status update
```
:bar_chart: *Status — [Project/Task name]*
Progress: [description]
Next: [what happens next]
```

## Rules
1. Always use channel ID `C0AKQDJEFFH`, never the channel name
2. Use threads for follow-up on the same topic
3. Check recent history before posting duplicate updates
4. Language: English
5. Keep messages concise — one screen max
