# Notion Schema — Strategy

Reference schema for all Strategy databases. Load this file only when you need
to create or update entries and need exact property names, types, and options.

## Shared properties

These appear in most databases with identical type and options:

| Property | Type | Options |
|---|---|---|
| Status | status | Not started, In progress, Done |
| Priority | select | High, Medium, Low |
| Team | multi_select | Strategy, Governance, Global |
| Due date | date | — |
| Assignee / Owner | people | — |
| People / Person | relation | -> Team Members |
| Attach file | files | — |

Note: Key Results has additional Status option: **Failed**.

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

### Fundraising
| Property | Type | Options |
|---|---|---|
| Investor name | title | — |
| Status | status | Not started, Diligence, Pitched, Lost, Won |
| Description | rich_text | — |
| Partner name | rich_text | — |
| Partner email | email | — |
| Warm contact name | rich_text | — |
| Warm contact email | email | — |
| Firm LinkedIn | rich_text | — |
| Portfolio | rich_text | — |
| Average check size | number ($) | — |
| Capital committed | number ($) | — |
| Last contact date | date | — |
| Lost reason | rich_text | — |
| Projects | relation (single) | -> Projects |

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

### Team Members
| Property | Type | Options |
|---|---|---|
| Name | title | — |
| Status | status | Active, Part-time, Contract, On Leave, Inactive |
| Role | select | Developer, Designer, Manager, Product Manager, Marketing, Sales, Support, HR, Finance |
| Department | select | Engineering, Design, Sales, Marketing, Operations, Product, Finance, Human Resources, Customer Success |
| Location | select | Remote, New York Office, London Office, San Francisco Office, Tokyo Office, Hybrid |
| Teamspace | multi_select | Strategy, Governance |
| Email | email | — |
| Phone | phone_number | — |
| Photo | url | — |
| Start Date | date | — |

Relation rollups (auto-populated):
- OKRs (Strategy), OKRs (Gov)
- Projects (Strategy), Projects (Gov)
- Tasks (Strategy), Tasks (Gov)
- Meetings (Strategy), Meetings (Gov)
- Docs (Strategy), Docs (Gov)
- Workflows (Strategy), Workflows (Gov)
