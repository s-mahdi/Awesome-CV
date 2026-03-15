# Job Application Pipeline — Phase Restructuring Design

> Created: 2026-03-15
> Status: Approved

## Problem

The original phase structure deferred any actual application capability until Phase 3 (browser auto-fill). Phases 1 and 2 only evaluated fit and drafted outreach — no submittable artifacts were produced. The user cannot apply to jobs until automation is fully built.

## Design Goal

Every phase must produce a complete, submittable application. Later phases reduce manual effort but never remove the prior path. The user can start applying to real jobs on Day 0.

## Core Principle

> Each phase = a working end-to-end apply workflow + one new automation layer.

## Phase Map

| Phase | Input | New Capability | Deliverable | Status |
|---|---|---|---|---|
| 0 | User pastes JD text | Fit eval + resume tailoring + cover letter | `job-list/` folder; user applies manually | **Implement now** |
| 1 | LinkedIn URL | `claude --chrome` JD capture replaces manual paste | Same as Phase 0, input automated | Next |
| 2 | — | Recruiter/HR outreach email draft | Phase 0 artifacts + `outreach-draft.md` | After Phase 1 |
| 3 | — | Browser auto-fill and submit | Fully automated application submission | Future |
| 4 | — | Google Sheets/Docs persistence | Applications logged automatically | Future |
| 5 | — | Gmail monitoring + Telegram alerts | Notifications on recruiter responses | Future |

## Phase 0 Detail

### Input
- Company name (string)
- Raw JD text (pasted by user)

### Steps
1. `job-fit-evaluator` subagent — score fit, identify gaps, produce verdict
2. `tailor` skill — resume adjustment suggestions based on JD keywords
3. `coverletter` skill — tailored cover letter draft

### Output
```
job-list/COMPANY_NAME/
  job-posting.md       ← company, role title, cleaned JD text
  job-analysis.md      ← fit score, verdict, matched skills, gaps
  cover-letter.tex     ← tailored cover letter source
  resume-notes.md      ← bullet-level tailoring suggestions
```

### Rules
- No remote-only gate (user decides which roles to pursue)
- No browser dependency
- All steps use existing implemented skills/subagents
- Entry point: new `/apply` skill that orchestrates the sequence

### Acceptance Criteria
- Given a company name and pasted JD, all four output files are produced
- `cover-letter.tex` compiles without errors
- `resume-notes.md` lists specific bullet changes keyed to JD requirements
- Workflow completes without any manual skill invocation mid-run

## Phase 1 Detail

### What Changes from Phase 0
- User provides a LinkedIn URL instead of pasting JD text
- `chrome-linkedin-match` skill drives `claude --chrome` to capture the JD
- `jd-capture` subagent normalizes the browser output into the same JD text format Phase 0 expects
- Everything downstream (fit eval, tailor, cover letter) is unchanged

### Rules
- Remote-only gate is removed (was a Phase 1 constraint in the old design)
- If Chrome capture fails or JD is truncated, fall back to prompting the user to paste the JD manually (Phase 0 path)

## Phase 2 Detail

Adds a recruiter/HR outreach email draft to the Phase 0 output folder:
```
job-list/COMPANY_NAME/
  outreach-draft.md    ← short recruiter message from recruiter-message-drafter subagent
```

The `recruiter-message-drafter` subagent already exists; this phase wires it into the apply flow.

## Phases 3–5

Design-only at this stage. Each phase plan file owns its own detailed spec:
- Phase 3: `docs/plans/phase-3/auto-apply-fill.plan.md`
- Phase 4: `docs/plans/phase-4/google-storage.plan.md`
- Phase 5: `docs/plans/phase-5/` (email-monitoring + telegram-notifications)

## Impact on Existing Plans

| Old Plan | Change |
|---|---|
| `foundation/chrome-skill-foundation.plan.md` | Skills are already implemented; foundation is complete. No new work. |
| `phase-1/linkedin-remote-match.plan.md` | Renamed conceptually to Phase 1 (chrome capture). Remote gate removed. |
| `phase-2/hr-email-outreach.plan.md` | Becomes Phase 2 (outreach draft appended to apply output). |
| `phase-3/auto-apply-fill.plan.md` | Becomes Phase 3. Unchanged. |
| `phase-4/google-storage.plan.md` | Becomes Phase 4. Unchanged. |
| `phase-5/clickup-tracking.plan.md` | Merged into Phase 4 (tracking). |
| `phase-6/email-monitoring.plan.md` | Becomes Phase 5. |
| `phase-6/telegram-notifications.plan.md` | Becomes Phase 5. |

## New Skill Required

`/apply` — orchestrates the Phase 0 workflow:
1. Accept company name + JD text
2. Run `job-fit-evaluator`
3. Run `tailor`
4. Run `coverletter`
5. Save artifacts to `job-list/COMPANY_NAME/`
6. Report summary to user

All constituent skills/subagents already exist. This skill is a thin orchestration wrapper.
