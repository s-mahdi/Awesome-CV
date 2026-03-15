# Plans Index

> Last Updated: 2026-03-15
> Scope: `docs/plans/`

This directory tracks the job-workflow roadmap, from the current LinkedIn remote-match phase through later automation, storage, and notification phases.

## Summary

| Metric | Count |
| --- | ---: |
| Total plan files | 9 |
| Active plans | 2 |
| Planning plans | 2 |
| Future plans | 5 |
| Plans with any repo-backed implementation | 4 |
| Fully implemented plans | 2 |
| Partially implemented plans | 2 |

## Plan Catalog

| Plan | Phase | Status | Implementation | Priority | Depends On | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| [`job-application-pipeline.plan.md`](job-application-pipeline.plan.md) | Master index | Planning | Partial | High | None | Master workflow map; Phase 1 is the current execution target. |
| [`foundation/chrome-skill-foundation.plan.md`](foundation/chrome-skill-foundation.plan.md) | Foundation | Active | Implemented | High | None | Project-local skills, subagents, prompt template, and Chrome permission setup are present in the repo. |
| [`phase-1/linkedin-remote-match.plan.md`](phase-1/linkedin-remote-match.plan.md) | Phase 1 | Active | Implemented | High | `chrome-skill-foundation` | Remote-only LinkedIn JD capture and fit-evaluation workflow is defined and backed by local skills/subagents. |
| [`phase-2/hr-email-outreach.plan.md`](phase-2/hr-email-outreach.plan.md) | Phase 2 | Future | Partial | Low | `job-application-pipeline` | Recruiter message drafting support exists, but the full outreach workflow is not active yet. |
| [`phase-3/auto-apply-fill.plan.md`](phase-3/auto-apply-fill.plan.md) | Phase 3 | Planning | Not implemented | High | `job-application-pipeline` | Browser automation and human approval flow are still design-only. |
| [`phase-4/google-storage.plan.md`](phase-4/google-storage.plan.md) | Phase 4 | Future | Not implemented | Medium | `linkedin-remote-match` | Google Docs and Sheets persistence is still unbuilt. |
| [`phase-5/clickup-tracking.plan.md`](phase-5/clickup-tracking.plan.md) | Phase 5 | Future | Not implemented | Medium | `job-application-pipeline` | ClickUp task creation and status sync are still unbuilt. |
| [`phase-6/email-monitoring.plan.md`](phase-6/email-monitoring.plan.md) | Phase 6 | Future | Not implemented | Medium | `clickup-tracking` (optional) | Gmail polling and alert classification are still unbuilt. |
| [`phase-6/telegram-notifications.plan.md`](phase-6/telegram-notifications.plan.md) | Phase 6 | Future | Not implemented | Medium | `linkedin-remote-match` | Telegram alert delivery is specified, but no delivery implementation is in the repo yet. |

## Phase Map

| Phase | Goal | Primary Plan Files | Current State |
| --- | --- | --- | --- |
| Foundation | Establish local browser-workflow skills and subagents | [`foundation/chrome-skill-foundation.plan.md`](foundation/chrome-skill-foundation.plan.md) | Implemented |
| Phase 1 | Evaluate LinkedIn jobs for remote-only fit | [`phase-1/linkedin-remote-match.plan.md`](phase-1/linkedin-remote-match.plan.md) | Implemented |
| Phase 2 | Prepare outreach and cover-letter-adjacent assets | [`phase-2/hr-email-outreach.plan.md`](phase-2/hr-email-outreach.plan.md) | Partially implemented |
| Phase 3 | Autofill and submit applications | [`phase-3/auto-apply-fill.plan.md`](phase-3/auto-apply-fill.plan.md) | Planned |
| Phase 4 | Persist artifacts to Google Docs and Sheets | [`phase-4/google-storage.plan.md`](phase-4/google-storage.plan.md) | Planned |
| Phase 5 | Track applications in ClickUp | [`phase-5/clickup-tracking.plan.md`](phase-5/clickup-tracking.plan.md) | Planned |
| Phase 6 | Monitor email and send Telegram alerts | [`phase-6/email-monitoring.plan.md`](phase-6/email-monitoring.plan.md), [`phase-6/telegram-notifications.plan.md`](phase-6/telegram-notifications.plan.md) | Planned |

## Implemented Repo Assets

This table tracks which plans already have concrete repo assets behind them.

| Plan | Implemented Assets | Evidence |
| --- | --- | --- |
| [`foundation/chrome-skill-foundation.plan.md`](foundation/chrome-skill-foundation.plan.md) | Prompt generator skill, Chrome LinkedIn match skill, JD capture subagent, job-fit evaluator subagent, recruiter-message-drafter subagent, coverletter-prep subagent, prompt template | `.claude/skills/prompt-generator/SKILL.md`, `.codex/skills/prompt-generator/SKILL.md`, `.claude/skills/chrome-linkedin-match/SKILL.md`, `.codex/skills/chrome-linkedin-match/SKILL.md`, `.claude/agents/jd-capture.md`, `.codex/agents/jd-capture.md`, `.claude/agents/job-fit-evaluator.md`, `.codex/agents/job-fit-evaluator.md`, `.claude/agents/recruiter-message-drafter.md`, `.codex/agents/recruiter-message-drafter.md`, `.claude/agents/coverletter-prep.md`, `.codex/agents/coverletter-prep.md`, `docs/prompts/chrome-linkedin-match.prompt.md` |
| [`phase-1/linkedin-remote-match.plan.md`](phase-1/linkedin-remote-match.plan.md) | Phase 1 workflow contract and local execution path | `docs/plans/phase-1/linkedin-remote-match.plan.md`, project-local Chrome workflow assets listed above |
| [`job-application-pipeline.plan.md`](job-application-pipeline.plan.md) | Master workflow index with active Phase 1 boundary | `docs/plans/job-application-pipeline.plan.md` |
| [`phase-2/hr-email-outreach.plan.md`](phase-2/hr-email-outreach.plan.md) | Early support via recruiter message drafting subagent only | `.claude/agents/recruiter-message-drafter.md`, `.codex/agents/recruiter-message-drafter.md` |

## Not Yet Implemented

The following plans currently exist as design documents only:

- [`phase-3/auto-apply-fill.plan.md`](phase-3/auto-apply-fill.plan.md)
- [`phase-4/google-storage.plan.md`](phase-4/google-storage.plan.md)
- [`phase-5/clickup-tracking.plan.md`](phase-5/clickup-tracking.plan.md)
- [`phase-6/email-monitoring.plan.md`](phase-6/email-monitoring.plan.md)
- [`phase-6/telegram-notifications.plan.md`](phase-6/telegram-notifications.plan.md)

## Maintenance Notes

- Update this file when a plan status changes or a new plan file is added.
- Prefer marking implementation as `Implemented`, `Partial`, or `Not implemented` based on repo evidence, not intent.
- Keep `job-application-pipeline.plan.md` as the workflow narrative and this file as the directory index.
