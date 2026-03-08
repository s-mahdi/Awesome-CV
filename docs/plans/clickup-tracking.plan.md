# ClickUp Application Tracking

> Status: Future
> Created: 2026-03-07
> Priority: Medium
> Depends on: job-application-pipeline

## Overview

Use the ClickUp MCP server to automatically track job applications. Each application becomes a ClickUp task with structured fields, status transitions, and timeline tracking.

## Proposed ClickUp Structure

### Space/List Layout
```
Job Search (Space)
  └── Applications (List)
        ├── To Apply
        ├── Applied
        ├── Interview Scheduled
        ├── Interviewed
        ├── Offer
        ├── Rejected
        └── Withdrawn
```

### Task Fields per Application
| Field | Type | Source |
|-------|------|--------|
| Task Name | text | `{Company} - {Role Title}` |
| Status | status | Pipeline stage |
| Job Posting URL | url | User input |
| Match Score | number | Gemini job-match agent output |
| Applied Date | date | Auto-set when cover letter generated |
| Resume Version | text | Git commit hash of resume used |
| Cover Letter | attachment | PDF from `job-list/COMPANY/` |
| Notes | text | Key observations from match report |
| Follow-up Date | date | Auto-set to applied + 7 days |

## Integration Points

### After `/apply` Skill Completes
1. Create ClickUp task with match score, job URL, and artifacts
2. Set status to "Applied"
3. Set follow-up reminder date

### Manual Status Updates
- User updates status in ClickUp (interview, offer, rejection)
- Or a `/track` skill could update status from CLI

## MCP Server Requirements

- ClickUp MCP server configured in Claude Code
- API token with task create/update permissions
- List ID for the Applications list

## Open Questions

- [ ] Which ClickUp workspace/space to use?
- [ ] Should we create the space/list structure automatically?
- [ ] Attach PDF files or just link to local paths?
- [ ] Sync status changes back to local `job-list/` metadata?
