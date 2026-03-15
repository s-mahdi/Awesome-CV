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
3. Save the original content of each file before editing.
4. Adjust the resume:
   - **Summary**: Rewrite to align with the target role's language and priorities.
   - **Skills**: Reorder skill blocks so the most relevant categories appear first. Add missing skills the candidate actually has.
   - **Experience bullets**: Reword to emphasize achievements matching the job requirements. Do NOT fabricate experience.
5. Build with `make resume.pdf` to verify compilation.
6. Summarize the changes made.

## Rules
- Never invent experience, skills, or metrics the candidate does not have.
- Preserve the existing LaTeX formatting conventions (see CLAUDE.md).
- Keep the resume to one page.

## Writing Style

Apply these rules to all rewritten summary text and experience bullets:

- Use clear, simple language.
- Use active voice. Avoid passive voice.
- Support claims with data and specific examples.
- Avoid unnecessary adjectives and adverbs.
- Use only commas, periods, semicolons, colons, and parentheses. Do not use em dashes.
- Avoid metaphors and clichés.
- Avoid generalizations.
- Do not use these words: can, may, just, that, very, really, literally, actually, certainly, probably, basically, could, maybe, delve, embark, enlightening, esteemed, shed light, craft, crafting, imagine, realm, game-changer, unlock, discover, skyrocket, revolutionize, disruptive, utilize, utilizing, tapestry, illuminate, unveil, pivotal, intricate, elucidate, hence, furthermore, however, harness, exciting, groundbreaking, cutting-edge, remarkable, remains to be seen, navigating, landscape, stark, testament, in summary, in conclusion, moreover.

## Suggestions-Only Mode

When invoked by the `/apply` skill (or explicitly instructed not to modify source
files), operate in **suggestions-only mode**:

1. Read the JD and resume files as normal.
2. Write a `resume-notes.md` file at the path specified in the instruction with the
   following format per suggestion:

   ```
   [file] → [current text] → [suggested rewrite]
   Targets: <JD keyword>
   ```

3. Do NOT write to any `src/resume/*.tex` file.
4. Do NOT run `make resume.pdf`.
5. Summarize the number of suggestions written.

This mode is triggered when the caller explicitly says "Do not modify any files in
`src/resume/`" or "suggestions-only".
