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
| 0 | User pastes JD text | Fit eval + resume tailoring notes + cover letter | `job-list/` folder; user applies manually | **Implement now** |
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
1. **Dispatch `job-fit-evaluator` as a subagent** — runs in an isolated context with its own tool access. Pass the JD text and resume file paths. Receive `job-analysis.md` content back. No remote-gate precondition. Workflow continues regardless of verdict.
2. `tailor` skill — modifies `src/resume/*.tex` in place, builds `make resume.pdf`, copies `src/resume.pdf` to `job-list/COMPANY_NAME/resume.pdf`, then restores the original source file contents. Also writes `job-list/COMPANY_NAME/resume-notes.md` summarising each change made.
3. `coverletter` skill — writes `src/coverletter.tex` and builds `src/coverletter.pdf` via `make coverletter.pdf`, then `/apply` copies both `src/coverletter.tex` → `job-list/COMPANY_NAME/cover-letter.tex` and `src/coverletter.pdf` → `job-list/COMPANY_NAME/cover-letter.pdf`.

### Output
```
job-list/COMPANY_NAME/
  job-posting.md       ← company, role title, cleaned JD text
  job-analysis.md      ← fit score, verdict, matched skills, gaps
  resume.pdf           ← tailored resume PDF (src files restored after build)
  resume-notes.md      ← summary of changes made to produce the tailored resume
  cover-letter.tex     ← copy of src/coverletter.tex after skill run
  cover-letter.pdf     ← copy of src/coverletter.pdf after build
```

### Rules
- No remote-only gate (user decides which roles to pursue)
- No browser dependency
- `tailor` runs in suggestions-only mode — source resume files are never modified by `/apply`
- `coverletter` output is copied from `src/` into `job-list/COMPANY_NAME/` after the build
- Entry point: new `/apply` skill that orchestrates the sequence

### Acceptance Criteria
- Given a company name and pasted JD, all six output files are produced in `job-list/COMPANY_NAME/`
- `job-list/COMPANY_NAME/resume.pdf` exists and was built from the tailored source
- `src/resume/*.tex` files are restored to their original content after the tailored build
- `job-list/COMPANY_NAME/cover-letter.tex` exists and matches `src/coverletter.tex` after the skill run
- `job-list/COMPANY_NAME/cover-letter.pdf` exists and matches `src/coverletter.pdf` after the build
- `make coverletter.pdf` and `make resume.pdf` both succeed
- `resume-notes.md` summarises each change made to produce the tailored resume
- Workflow continues and produces all artifacts regardless of fit verdict (poor/moderate/strong)
- Workflow completes without any manual skill invocation mid-run

## Phase 1 Detail

### What Changes from Phase 0
- User provides a LinkedIn URL instead of pasting JD text
- `chrome-linkedin-match` skill drives `claude --chrome` to capture the JD
- `jd-capture` subagent normalizes the browser output into the same JD text format Phase 0 expects
- Everything downstream (fit eval, tailor, cover letter) is unchanged

### Rules
- Remote-only gate is removed (was a constraint in the old design; not carried forward)
- If Chrome capture fails or JD is truncated, fall back to prompting the user to paste the JD manually (Phase 0 path)
- `job-fit-evaluator` is still invoked without a remote-gate precondition (same as Phase 0)

## Phase 2 Detail

Adds a recruiter/HR outreach email draft to the output folder:
```
job-list/COMPANY_NAME/
  outreach-draft.md    ← short recruiter message from recruiter-message-drafter subagent
```

The `recruiter-message-drafter` subagent already exists; this phase wires it into the apply flow. The filename `outreach-draft.md` supersedes `hr-email.md` referenced in the old `phase-2/hr-email-outreach.plan.md`; that plan file will be updated accordingly.

## Phases 3–5

Design-only at this stage. Each phase plan file owns its own detailed spec.

Current disk locations (files will be moved to match new phase numbering as part of restructuring):
- Phase 3: `docs/plans/phase-3/auto-apply-fill.plan.md` (exists)
- Phase 4: `docs/plans/phase-4/google-storage.plan.md` (exists; ClickUp tracking deferred — see note below)
- Phase 5: currently at `docs/plans/phase-6/email-monitoring.plan.md` and `docs/plans/phase-6/telegram-notifications.plan.md` (will be moved to `phase-5/`)

**Note on ClickUp tracking:** The old `phase-5/clickup-tracking.plan.md` is deferred. Phase 4 uses Google Sheets/Docs only for persistence. ClickUp integration may be revisited after Phase 4 is implemented.

## Impact on Existing Plans

| Old Plan | Change |
|---|---|
| `foundation/chrome-skill-foundation.plan.md` | Skills are already implemented; foundation is complete. No new work. |
| `phase-1/linkedin-remote-match.plan.md` | Becomes Phase 1 (chrome capture). Remote gate removed. Plan file to be updated. |
| `phase-2/hr-email-outreach.plan.md` | Becomes Phase 2. Output filename changed from `hr-email.md` to `outreach-draft.md`. Plan file to be updated. |
| `phase-3/auto-apply-fill.plan.md` | Becomes Phase 3. Unchanged. |
| `phase-4/google-storage.plan.md` | Becomes Phase 4. Unchanged. |
| `phase-5/clickup-tracking.plan.md` | Deferred. Not part of the new phase ladder. |
| `phase-6/email-monitoring.plan.md` | Becomes Phase 5. Move to `phase-5/`. |
| `phase-6/telegram-notifications.plan.md` | Becomes Phase 5. Move to `phase-5/`. |

## New Skill Required

`/apply` — orchestrates the Phase 0 workflow:

**Invocation:** Interactive prompt. When invoked, the skill asks:
1. "Company name?" — single string, used as the folder name under `job-list/`
2. "Paste the job description:" — user pastes raw JD text inline

**Steps:**
1. Write `job-list/COMPANY_NAME/job-posting.md` with company, role title (extracted from JD), and full JD text
2. Run `job-fit-evaluator` (without remote-gate precondition) → write `job-analysis.md`
3. Run `tailor` with instruction: "Output suggestions only to `job-list/COMPANY_NAME/resume-notes.md`. Do not edit any source files." → write `resume-notes.md`
4. Run `coverletter` → build `src/coverletter.tex`, validate with `make coverletter.pdf`, copy to `job-list/COMPANY_NAME/cover-letter.tex`
5. Report summary: fit verdict, artifact paths, next manual steps

**Early-exit policy:** None. All steps run regardless of fit verdict. If a step fails, report the error and continue remaining steps.

All constituent skills/subagents already exist. This skill is a thin orchestration wrapper.

## Required Agent/Skill Updates

| Asset | Required Change |
|---|---|
| `job-fit-evaluator` subagent | Remove hard dependency on remote-gate precondition; treat it as optional |
| `tailor` skill | Document/enforce suggestions-only invocation path that does not modify `src/resume/*.tex` |
| `phase-2/hr-email-outreach.plan.md` | Update output filename to `outreach-draft.md` |
| `phase-1/linkedin-remote-match.plan.md` | Remove remote-only gate from rules |
