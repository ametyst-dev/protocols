# Notion Protocol — Governance

## Backend
- **Backend page:** `428e84c9-1bd6-4b9d-8ace-e17cedc1edf7`
- **Area:** Governance (legal, corporate, finance, compliance)
- **Schema reference:** see `notion-schema.md` in this folder

## Database registry

| Database | data_source_id (fetch) | database_id (write) | Title field |
|---|---|---|---|
| Objectives | `fb81376a-3ebb-490a-ab59-fe761778f593` | `6d05fc3d-083d-4ca7-b5fc-89fa66c91bf0` | Goal name |
| Key Results | `c2eeb22a-c716-4b50-a24e-742e91061fda` | `51b726bf-8cd8-4e26-b042-fbe20ff1d359` | Goal name |
| Meetings | `269f8204-8c51-4d59-a6ae-5ad534b172a6` | `684b83ca-8073-442b-84f2-a484db073955` | Meeting name |
| Projects | `b2d3dfa7-9ea1-4fc7-a9de-b02945d1ce79` | `59192c92-1a61-4794-ade5-c430b67fade9` | Project name |
| Tasks | `9d37ea98-f454-447b-9fc4-1e4cf535c023` | `23283c0a-6de9-4196-89b6-fa3fd2b50447` | Task name |
| Docs | `9fc68281-a797-4e82-9f42-b41076749f9e` | `9bff4a2f-3af1-4a28-8d1a-5523745e561b` | Doc name |
| Workflows | `216a9c3a-a3ba-491d-a980-fd8d155ded9f` | `95cf822a-764f-4047-a89c-585a7f625141` | Task name |


## Fetching

Use `API-query-data-source` with `data_source_id`.

### Always filter by Team
All databases use `Team` multi_select to separate Governance from Strategy. Always apply:
```json
{
  "data_source_id": "<id>",
  "filter": { "property": "Team", "multi_select": { "contains": "Governance" } }
}
```

### Compound filters
```json
{
  "data_source_id": "9d37ea98-f454-447b-9fc4-1e4cf535c023",
  "filter": {
    "and": [
      { "property": "Team", "multi_select": { "contains": "Governance" } },
      { "property": "Status", "status": { "equals": "In progress" } }
    ]
  }
}
```

### Filter patterns
- **status:** `{ "equals": "In progress" }`, `{ "does_not_equal": "Done" }`
- **select:** `{ "equals": "High" }`
- **multi_select:** `{ "contains": "Q1" }`
- **date:** `{ "before": "2026-04-01" }`, `{ "after": "2026-01-01" }`

## Content format — Markdown

### Writing
Page body content MUST always be written as a single `code` block with `language: "markdown"` containing the verbatim markdown source. Do NOT convert markdown to Notion-native blocks (paragraphs, headings, bullets, etc.) unless the user explicitly asks for it.

**CRITICAL: Verbatim content only.** When pushing a local file to Notion, the content MUST be the exact copy of the source file — read it and use it as-is. Never reconstruct, rephrase, summarize, or shorten. If the Notion content does not match the local file, it is wrong.

Each `rich_text` content item has a 2000-character limit. If the content exceeds 2000 characters, split it into multiple `rich_text` items within the same code block. Split at natural line breaks.

### Reading
When fetching page content, check the block type first. If the body is a `code` block with language `markdown`, treat its content as verbatim markdown. If the body consists of Notion-native blocks (paragraphs, headings, bullets, etc.), read them as standard Notion content.

## Writing

Use `API-post-page` with `database_id` as parent. Use `API-patch-page` to update.

### Required and recommended fields

| Database | Required | Recommended |
|---|---|---|
| Objectives | Goal name, Status, Priority, Quarter | Due date, Owner, Team=Governance |
| Key Results | Goal name, Status, Priority | Objectives (relation), Quarter, Due date |
| Meetings | Meeting name, Date | Category, Type, Team=Governance |
| Projects | Project name, Status, Priority | Team=Governance, Start/End date, Assignee |
| Tasks | Task name, Status | Priority, Team=Governance, Due date, Projects |
| Docs | Doc name | Category, Team=Governance |
| Workflows | Task name, Status | Priority, Team=Governance, Due date |
## Relations map
```
Objectives <-> Key Results (dual)
Objectives <-> Projects (dual)
Projects <-> Tasks (dual)
Key Results -> Projects (single)
All databases -> Team Members (via People/Person relation)
```

Team Members database lives in Strategy backend:
`data_source_id: 31c7ea63-e252-80eb-b0bb-000bec4b470c`
