# Slack Protocol — Strategy

## Channel
- **#strategy** (private)
- **Channel ID:** `C0AHN1ZB18T`

## When to post
An agent MUST post to #strategy after any of these events:
1. **Notion fetch** — report what was queried and key results
2. **Notion write/update** — report what was created or changed
3. **HIL request** — ask Patrick or a specific person for input/approval
4. **Status update** — progress on a strategy task, project, or fundraising activity

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

### Fundraising update
```
:moneybag: *Fundraising — [Investor name]*
Status: [old] -> [new]
Details: [key info]
```

### Status update
```
:bar_chart: *Status — [Project/Task name]*
Progress: [description]
Next: [what happens next]
```

## Rules
1. Always use channel ID `C0AHN1ZB18T`, never the channel name
2. Use threads for follow-up on the same topic
3. Check recent history before posting duplicate updates
4. Language: English
5. Keep messages concise — one screen max
