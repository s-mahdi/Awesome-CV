# Chrome LinkedIn Match Prompt

Use this as the canonical structure when generating the runtime prompt for `claude --chrome`.

## Required Context

- Candidate profile comes from:
  - `src/resume/summary.tex`
  - `src/resume/experience.tex`
  - `src/resume/skills.tex`
- Current phase is fit evaluation only.
- Remote-only is a hard gate.

## Prompt Template

```prompt
You are evaluating a LinkedIn job posting for Mahdi.

Task:
1. Open the LinkedIn job page or use the currently visible LinkedIn job page.
2. Extract the company name, role title, job URL, workplace type, location, and any recruiter or hiring-contact clues that are already visible.
3. Capture the full job description text as clean markdown.
4. Determine whether the role is explicitly remote. If the role is not remote or cannot be confirmed as remote, mark it as a stop condition.
5. Return structured markdown with these sections exactly:
   - `# LinkedIn Job Snapshot`
   - `## Company`
   - `## Role`
   - `## URL`
   - `## Workplace Type`
   - `## Location`
   - `## Recruiter Clues`
   - `## Remote Gate`
   - `## Job Description`
   - `## Blockers`

Rules:
- Do not apply for the job.
- Do not message the recruiter.
- Do not invent hidden details.
- If the page is login-blocked, incomplete, or truncated, state that clearly in `## Blockers`.
```
