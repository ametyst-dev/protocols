# Notion Protocol — Strategy

## Backend
- **Backend page:** `2f2d4f43-933f-48bc-b5dd-3fa64fa59a5a`
- **Area:** Strategy (product direction, fundraising, growth, partnerships)
- **Schema reference:** see `notion-schema.md` in this folder

## Database registry

| Database | data_source_id (fetch) | database_id (write) | Title field |
|---|---|---|---|
| Objectives | `4a52f6b5-ccbf-45b9-956e-88d8c32c5154` | `71ae8ba9-f495-4a23-bcc5-8a2f1d58b26d` | Goal name |
| Key Results | `cc1c8800-6607-43bb-be65-e05eba49cd05` | `08517b68-e7d1-4ca5-bd0c-85b67edf21b6` | Goal name |
| Meetings | `6853dbe5-e423-4bd1-80c9-b7257238fcc0` | `314eb224-6997-41dc-9637-1773563684d6` | Meeting name |
| Projects | `8555f43c-16ef-4140-a6de-51a013b7ca3d` | `6b2084fa-b686-4345-8a55-f7dcee1b2583` | Project name |
| Tasks | `bf4f1dc5-9732-4f6f-945c-13ca656d6bbf` | `cd5b376d-3abf-4ce8-88a3-4d7745173bf8` | Task name |
| Fundraising | `ca26f0dc-ae92-4cdf-9b57-262a4b364b23` | `93f9b67c-c4d8-440d-92a4-11380b207a5e` | Investor name |
| Docs | `2fc64926-5908-485f-aa8a-afe1d638b3d8` | `1de3b19c-eb86-48a9-aca7-c1aa9aba4eaf` | Doc name |
| Workflows | `acc15055-757b-42f7-b4e1-b2fc9c2ea0ac` | `d99c53ff-cd3d-4861-a8a4-661a4f7aadfe` | Task name |
| Team Members | `31c7ea63-e252-80eb-b0bb-000bec4b470c` | `31c7ea63-e252-8077-8183-df05bae0a91b` | Name |

## Fetching

Use `API-query-data-source` with `data_source_id`.

### Always filter by Team
All databases use `Team` multi_select. Always apply:
```json
{
  "data_source_id": "<id>",
  "filter": { "property": "Team", "multi_select": { "contains": "Strategy" } }
}
```

Fundraising and Team Members do not have a Team filter — they are Strategy-only.

### Compound filters
```json
{
  "data_source_id": "bf4f1dc5-9732-4f6f-945c-13ca656d6bbf",
  "filter": {
    "and": [
      { "property": "Team", "multi_select": { "contains": "Strategy" } },
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
| Objectives | Goal name, Status, Priority, Quarter | Due date, Owner, Team=Strategy |
| Key Results | Goal name, Status, Priority | Objectives (relation), Quarter, Due date |
| Meetings | Meeting name, Date | Category, Type, Team=Strategy |
| Projects | Project name, Status, Priority | Team=Strategy, Start/End date, Assignee |
| Tasks | Task name, Status | Priority, Team=Strategy, Due date, Projects |
| Fundraising | Investor name, Status | Partner name, Partner email, Average check size |
| Docs | Doc name | Category, Team=Strategy |
| Workflows | Task name, Status | Priority, Team=Strategy, Due date |
| Team Members | Name, Status, Role | Email, Teamspace, Department |

## Relations map
```
Objectives <-> Key Results (dual)
Objectives <-> Projects (dual)
Projects <-> Tasks (dual)
Key Results -> Projects (single)
Fundraising -> Projects (single)
All databases -> Team Members (via People/Person relation)
```

Team Members is the central hub — referenced by both Strategy and Governance backends.
