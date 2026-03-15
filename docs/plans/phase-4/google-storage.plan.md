# Google Storage

> Status: Future
> Created: 2026-03-15
> Priority: Medium
> Depends on: linkedin-remote-match

## Overview

Later phases should persist approved application artifacts outside the repo:
- store each JD in Google Docs
- create date-based folders
- append application tracking rows to Google Sheets

## Google Docs

### Goal
- save the cleaned JD and analysis summary to a Google Doc
- organize docs into folders by date, using `YYYY-MM-DD`

### Proposed Content
- title: `{Company} - {Role}`
- source URL
- remote status
- cleaned JD text
- fit summary
- recruiter/contact notes if visible

## Google Sheets

### Goal
- maintain a single application ledger for roles that move beyond fit evaluation

### Suggested Columns
- date
- company
- role
- source URL
- remote status
- match score
- verdict
- applied status
- notes

## Integration Point

- read from `job-list/COMPANY_NAME/`
- only write to Google Docs and Google Sheets after the job artifacts already exist locally
- leave submission-state updates to the future ClickUp and auto-apply phases

## Open Questions

- which Google account and folder root should be used
- whether Google Sheets writes happen after a strong match or only after an actual application
- whether Docs and Sheets should use MCP tools or external CLI/API automation
