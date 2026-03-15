# Telegram Notifications

> Status: Future
> Created: 2026-03-15
> Priority: Medium
> Depends on: linkedin-remote-match

## Overview

Telegram alerts are reserved for roles that pass the Phase 1 fit gate. The intent is to notify only on roles worth pursuing instead of sending alerts for every scraped job.

## Trigger Rules

- remote-only gate passed
- match verdict is `strong fit`
- no blocker recorded for login, missing JD, or incomplete page access

## Notification Payload

- company
- role
- LinkedIn URL
- match score
- top two reasons it matches
- recruiter/contact note if visible

## Delivery Options

### Direct bot delivery
- send a plain text message via Telegram Bot API

### MCP-backed delivery
- use a Telegram MCP server if one is added later

## Integration Point

- consume `job-analysis.md`
- send after local artifacts are saved
- stay independent from Gmail monitoring and ClickUp tracking

## Open Questions

- which bot token and chat target should be used
- whether notifications should be immediate or batched
- whether a moderate-fit role should generate a lower-priority alert
