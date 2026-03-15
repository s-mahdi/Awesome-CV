# LinkedIn Remote Match

> Status: Active
> Created: 2026-03-15
> Priority: High
> Depends on: chrome-skill-foundation

## Overview

Phase 1 focuses on fit evaluation only. The workflow opens a LinkedIn job page in Chrome mode, extracts the JD, checks whether the role is remote, and compares the role to the current resume before any application work begins.

## Inputs

- LinkedIn job URL or currently open LinkedIn job page
- Resume content from:
  - `src/resume/summary.tex`
  - `src/resume/experience.tex`
  - `src/resume/skills.tex`

## Outputs

Artifacts are stored under:

```text
job-list/
  COMPANY_NAME/
    job-posting.md
    job-analysis.md
```

### `job-posting.md`
- source URL
- role title
- company name
- workplace type
- location
- recruiter/contact hints if visible
- cleaned JD text

### `job-analysis.md`
- remote-only pass/fail
- match score
- verdict: strong / moderate / poor fit
- top matching evidence from the resume
- hard blockers or missing requirements
- recommendation: pursue / skip

## Execution Flow

1. Read the resume files needed for fit evaluation.
2. Generate a browser prompt with the local `prompt-generator` skill.
3. Run `claude --chrome "PROMPT"` against the LinkedIn job page.
4. Normalize the browser output with the `jd-capture` subagent.
5. Reject the role immediately if the normalized output does not confirm a remote workplace type.
6. Evaluate fit with the `job-fit-evaluator` subagent.
7. Save `job-posting.md` and `job-analysis.md`.
8. Report whether the role is a good match.

## Rules

- Treat remote-only as a hard gate.
- Do not apply, draft messages, or generate a cover letter in this phase.
- Record recruiter/contact details only if they are already visible on the page.
- If LinkedIn content is truncated or blocked by login state, return a blocker instead of fabricating missing JD text.
- Reuse `job-list/COMPANY_NAME/` so later phases can build on the same artifacts.

## Acceptance Criteria

- A LinkedIn job can be evaluated without editing resume files.
- Non-remote roles stop before fit evaluation is reported as actionable.
- Strong matches produce both local artifacts and a concise recommendation.
- Weak or blocked matches clearly explain why the role should be skipped.
