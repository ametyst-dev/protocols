# Notion Protocol — General

## Backend
- **Backend page:** `300830d3-fd33-40bd-a431-cc46f3025ecd`
- **Area:** General (cross-team knowledge, shared docs)
- **Schema reference:** see `notion-schema.md` in this folder

## Database registry

| Database | data_source_id (fetch) | database_id (write) | Title field |
|---|---|---|---|
| Protocols | `86503a95-04f3-4d76-9e9a-898b71006742` | `75aa4626-9c80-4c2b-87f2-b9e5218544cb` | Doc name |

## Fetching

Use `API-query-data-source` with `data_source_id`.

### Filter by Team
```json
{
  "data_source_id": "86503a95-04f3-4d76-9e9a-898b71006742",
  "filter": { "property": "Team", "multi_select": { "contains": "Global" } }
}
```

### Filter by Category
```json
{
  "data_source_id": "86503a95-04f3-4d76-9e9a-898b71006742",
  "filter": { "property": "Category", "multi_select": { "contains": "Business plan" } }
}
```

## Content format — Markdown

### Writing
Page body content MUST always be written as a single `code` block with `language: "markdown"` containing the verbatim markdown source. Do NOT convert markdown to Notion-native blocks (paragraphs, headings, bullets, etc.) unless the user explicitly asks for it.

**CRITICAL: Verbatim content only.** When pushing a local file to Notion, the content MUST be the exact copy of the source file — read it and use it as-is. Never reconstruct, rephrase, summarize, or shorten. If the Notion content does not match the local file, it is wrong.

Each `rich_text` content item has a 2000-character limit. If the content exceeds 2000 characters, split it into multiple `rich_text` items within the same code block. Split at natural line breaks.

### Reading
When fetching page content, check the block type first. If the body is a `code` block with language `markdown`, treat its content as verbatim markdown. If the body consists of Notion-native blocks (paragraphs, headings, bullets, etc.), read them as standard Notion content.

## Writing

Use `API-post-page` with `database_id` as parent.

| Database | Required | Recommended |
|---|---|---|
| Protocols | Doc name | Category, Team |

## Relations
- `Person` -> Team Members (`31c7ea63-e252-80eb-b0bb-000bec4b470c`, single_property)
