---
name: tailor
description: Tailor the resume for a specific job posting by adjusting summary, skills, and experience bullets
user-invocable: true
---

# Tailor Resume for Job Posting

The user will provide a job description (pasted text or URL). Adjust the resume to better match the role.

## Steps

1. Read the job description and extract key requirements, technologies, and soft skills.
2. Read the current resume files:
   - `src/resume/summary.tex`
   - `src/resume/skills.tex`
   - `src/resume/experience.tex`
3. Adjust the resume:
   - **Summary**: Rewrite to align with the target role's language and priorities.
   - **Skills**: Reorder skill blocks so the most relevant categories appear first. Add missing skills the candidate actually has.
   - **Experience bullets**: Reword to emphasize achievements matching the job requirements. Do NOT fabricate experience.
4. Build with `make resume.pdf` to verify compilation.
5. Summarize the changes made.

## Rules
- Never invent experience, skills, or metrics the candidate does not have.
- Preserve the existing LaTeX formatting conventions (see CLAUDE.md).
- Keep the resume to one page.
