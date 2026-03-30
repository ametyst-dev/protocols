# Notion Schema — Governance

Reference schema for all Governance databases. Load this file only when you need
to create or update entries and need to know exact property names, types, and options.

## Shared properties

These properties appear in most databases with identical type and options:

| Property | Type | Options |
|---|---|---|
| Status | status | Not started, In progress, Done |
| Priority | select | High, Medium, Low |
| Team | multi_select | Strategy, Governance, Global |
| Due date | date | — |
| Assignee / Owner | people | — |
| People / Person | relation | -> Team Members |
| Attach file | files | — |

Note: Key Results has additional Status options: **Failed** (between In progress and Done).

## Per-database unique properties

### Objectives
| Property | Type | Options |
|---|---|---|
| Quarter | multi_select | Q1, Q2, Q3, Q4 |
| Start value | number | — |
| End value | number | — |
| Progress | formula | Start / End |
| Days until due | formula | days remaining or "Overdue" |
| Projects | relation (dual) | -> Projects.OKRs |
| Key Results | relation (dual) | -> Key Results.Objectives |

### Key Results
| Property | Type | Options |
|---|---|---|
| Quarter | multi_select | Q1, Q2, Q3, Q4 |
| Start value | number | — |
| End value | number | — |
| Progress | formula | Start / End |
| Days until due | formula | days remaining or "Overdue" |
| Objectives | relation (dual) | -> Objectives.Key Results |
| Projects | relation (single) | -> Projects |

### Meetings
| Property | Type | Options |
|---|---|---|
| Category | multi_select | Planning, Standup, Presentation, Retro, Customer call |
| Type | multi_select | Standup, Weekly Recap, To discuss Topic, Team Brainstorming |
| Tags | multi_select | Event, Bank meeting, Crypto meeting, I3P meeting, Legal meeting, Investor meeting, Professor meeting, Fintech meeting, Dev partnership, Incubator, Tech studio, Other, Brand identity |
| Attendees | people | — |

### Projects
| Property | Type | Options |
|---|---|---|
| Budget | number ($) | — |
| Start date | date | — |
| End date | date | — |
| Start value | number | — |
| End value | number | — |
| Progress | formula | Start / End |
| Tasks | relation (dual) | -> Tasks.Projects |
| OKRs | relation (dual) | -> Objectives.Projects |

### Tasks
| Property | Type | Options |
|---|---|---|
| Effort level | select | Small, Medium, Large |
| Task type | multi_select | Bug, Feature request, Polish |
| Description | rich_text | — |
| Past due | formula | "Past Due" or "" |
| Projects | relation (dual) | -> Projects.Tasks |

### Docs
| Property | Type | Options |
|---|---|---|
| Category | multi_select | Mission Vision & Values, Branding, Knowledge base, Incorporation, Shareholders, Cap Table, IP, Employment, Corporate governance, Financial dashboard, Financial model, Cash&Treasury, Pitchdeck, Business plan, Accounting, Data room |
| Reviewers | people | — |

### Workflows
| Property | Type | Options |
|---|---|---|
| Effort level | select | Small, Medium, Large |
| Task type | multi_select | Bug, Feature request, Polish |
| Description | rich_text | — |
| Past due | formula | "Past Due" or "" |
