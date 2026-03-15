# Job Application Pipeline

> Status: Planning
> Created: 2026-03-07
> Updated: 2026-03-15
> Priority: High

## Overview

This document is the master index for the job-application workflow. It records what is already implemented in the repo, what is still missing, and which plan file owns each phase.

The current implementation target is **Phase 1 only**:
- open LinkedIn in Chrome mode
- capture the job description
- reject non-remote roles
- evaluate fit against the resume
- flag strong matches

Application submission, recruiter outreach, cover-letter generation, Google storage, Telegram notifications, and ClickUp tracking remain downstream phases.

## Current Asset Audit

| Area | State | Evidence | Plan Owner |
| --- | --- | --- | --- |
| Resume tailoring skill | Implemented | `.claude/skills/tailor/SKILL.md`, `.codex/skills/tailor/SKILL.md` | Existing |
| Cover letter skill | Implemented | `.claude/skills/coverletter/SKILL.md`, `.codex/skills/coverletter/SKILL.md` | Existing |
| ATS review skill | Implemented | `.claude/skills/ats-evaluate/SKILL.md`, `.codex/skills/ats-evaluate/SKILL.md` | Existing |
| Research subagent | Implemented | `.claude/agents/search.md`, `.codex/agents/search.md` | Existing |
| Prompt generator skill | Implemented in project scope | `.claude/skills/prompt-generator/SKILL.md`, `.codex/skills/prompt-generator/SKILL.md` | `chrome-skill-foundation.plan.md` |
| Chrome LinkedIn match skill | Implemented in project scope | `.claude/skills/chrome-linkedin-match/SKILL.md`, `.codex/skills/chrome-linkedin-match/SKILL.md` | `chrome-skill-foundation.plan.md` |
| JD capture subagent | Implemented in project scope | `.claude/agents/jd-capture.md`, `.codex/agents/jd-capture.md` | `chrome-skill-foundation.plan.md` |
| Job fit evaluator subagent | Implemented in project scope | `.claude/agents/job-fit-evaluator.md`, `.codex/agents/job-fit-evaluator.md` | `chrome-skill-foundation.plan.md` |
| Recruiter message draft subagent | Implemented in project scope | `.claude/agents/recruiter-message-drafter.md`, `.codex/agents/recruiter-message-drafter.md` | `hr-email-outreach.plan.md` |
| Cover letter prep subagent | Implemented in project scope | `.claude/agents/coverletter-prep.md`, `.codex/agents/coverletter-prep.md` | `chrome-skill-foundation.plan.md` |
| LinkedIn phase-1 plan | Implemented | `docs/plans/linkedin-remote-match.plan.md` | `linkedin-remote-match.plan.md` |
| Auto-fill / external apply | Planned | `auto-apply-fill.plan.md` | Existing |
| ClickUp tracking | Planned | `clickup-tracking.plan.md` | Existing |
| Recruiter / HR outreach | Planned | `hr-email-outreach.plan.md` | Existing |
| Email monitoring | Planned | `email-monitoring.plan.md` | Existing |
| Google Docs / Sheets storage | Planned | `docs/plans/google-storage.plan.md` | `google-storage.plan.md` |
| Telegram notifications | Planned | `docs/plans/telegram-notifications.plan.md` | `telegram-notifications.plan.md` |

## Phase Map

| Phase | Goal | Status | Plan File |
| --- | --- | --- | --- |
| 1 | LinkedIn remote-only JD capture and fit evaluation | Active | `linkedin-remote-match.plan.md` |
| 2 | Cover-letter preparation and recruiter outreach drafts | Planned | `chrome-skill-foundation.plan.md`, `hr-email-outreach.plan.md` |
| 3 | External application auto-fill and submit | Planned | `auto-apply-fill.plan.md` |
| 4 | Google Docs and Google Sheets storage | Planned | `google-storage.plan.md` |
| 5 | ClickUp tracking | Planned | `clickup-tracking.plan.md` |
| 6 | Gmail monitoring and Telegram alerts | Planned | `email-monitoring.plan.md`, `telegram-notifications.plan.md` |

## Phase 1 Contract

### In Scope
- Drive LinkedIn pages through `claude --chrome`.
- Capture structured JD data and recruiter/contact hints if visible.
- Enforce remote-only gating before further work.
- Score fit using the current resume content.
- Save local artifacts under `job-list/COMPANY_NAME/`.

### Out of Scope
- Submitting applications
- Sending recruiter messages
- Generating or building cover letters
- Updating Google Docs, Google Sheets, ClickUp, or Telegram
- Applying outside LinkedIn

## Implemented Project Assets

### Skills
- `.claude/skills/prompt-generator/SKILL.md`
- `.codex/skills/prompt-generator/SKILL.md`
- `.claude/skills/chrome-linkedin-match/SKILL.md`
- `.codex/skills/chrome-linkedin-match/SKILL.md`

### Subagents
- `.claude/agents/jd-capture.md`
- `.claude/agents/job-fit-evaluator.md`
- `.claude/agents/recruiter-message-drafter.md`
- `.claude/agents/coverletter-prep.md`
- `.codex/agents/jd-capture.md`
- `.codex/agents/job-fit-evaluator.md`
- `.codex/agents/recruiter-message-drafter.md`
- `.codex/agents/coverletter-prep.md`

### Prompt Templates
- `docs/prompts/chrome-linkedin-match.prompt.md`

## Dependencies

- `chrome-linkedin-match` depends on the imported project-local `prompt-generator` skill.
- `job-fit-evaluator` depends on the current resume sources in `src/resume/`.
- `recruiter-message-drafter` and `coverletter-prep` are defined now but remain inactive until later phases.
- Project-local `.claude/settings.local.json` must allow the narrow `claude --chrome` command path needed by the skill.
