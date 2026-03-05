---
name: coverletter
description: Generate or update a tailored cover letter for a job posting
---

# Generate Cover Letter

Create or update `src/coverletter.tex` with a tailored cover letter for a specific job posting.

## Steps

1. The user provides a job description (pasted text or URL).

2. Read the current resume content to understand the candidate's background:
   - `src/resume/summary.tex`
   - `src/resume/experience.tex`
   - `src/resume/skills.tex`

3. Read `src/coverletter.tex` to understand the existing template structure.

4. Update `src/coverletter.tex`:
   - **Personal info**: Use the same info from `src/resume.tex` (name, contact, etc.)
   - **Recipient**: Company name and address from the job posting
   - **Letter title**: "Job Application for [Role Title]"
   - **Opening**: Professional greeting
   - **Body sections** using `\lettersection{}`:
     - **About Me**: Brief introduction connecting background to the role
     - **Why [Company]?**: What draws the candidate to this specific company
     - **Why Me?**: Key achievements and skills that match the job requirements
   - **Closing**: Professional sign-off with resume enclosure

5. Build with `make coverletter.pdf` to verify compilation.

## Rules

- Never fabricate experience or achievements not present in the resume.
- Keep the tone professional but personable.
- Reference specific technologies and achievements from the resume that match the job requirements.
- Match the accent color to the resume (`acvBlue` / `awesome-red` as configured).
- Keep the letter concise (one page).
