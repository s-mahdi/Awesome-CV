---
name: chrome-linkedin-match
description: Evaluate LinkedIn jobs in Chrome mode using `claude --chrome`. Use this for remote-only fit checks against the resume before any application work starts.
user-invocable: true
---

# Chrome LinkedIn Match

Run the Phase 1 browser workflow for LinkedIn jobs.

## Steps

1. Read the current resume context:
   - `src/resume/summary.tex`
   - `src/resume/experience.tex`
   - `src/resume/skills.tex`
2. Use the local `prompt-generator` skill to generate the final browser prompt.
3. Use `docs/prompts/chrome-linkedin-match.prompt.md` as the canonical prompt shape.
4. Run `claude --chrome "PROMPT"`.
5. Normalize the browser output with the `jd-capture` subagent.
6. If the role is not remote, stop and report that it is out of scope.
7. Send the normalized JD and resume context to the `job-fit-evaluator` subagent.
8. Save the normalized JD to `job-list/COMPANY_NAME/job-posting.md`.
9. Save the fit report to `job-list/COMPANY_NAME/job-analysis.md`.

## Rules

- This skill is Phase 1 only.
- Do not submit applications.
- Do not draft or send recruiter messages.
- Do not generate cover letters.
- Record recruiter/contact information only if it is already visible on the page.
- If LinkedIn access is blocked or incomplete, return a blocker instead of guessing.

## Output

Return a short summary containing:
- company
- role
- remote gate result
- match score and verdict
- saved artifact paths
