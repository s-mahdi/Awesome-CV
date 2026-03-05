---
name: add-section
description: Add a new section to the resume by creating a .tex file and wiring it into resume.tex
---

# Add Resume Section

Create a new section file and integrate it into the resume layout.

## Steps

1. Ask the user for:
   - **Section name** (e.g., "Projects", "Certifications", "Volunteering")
   - **Column placement**: left column (main content) or right column (sidebar)
   - **Content type**: entries (`cventries`), skills (`cvskills`), honors (`cvhonors`), or paragraph (`cvparagraph`)

2. Read `src/resume.tex` to understand the current two-column layout and active `\input` lines.

3. Create `src/resume/<section-name>.tex` with:
   - Standard section header comment block
   - `\cvsection{Section Title}`
   - Appropriate environment with a placeholder entry matching the chosen content type

4. Add `\input{resume/<section-name>.tex}` to `src/resume.tex` in the correct minipage:
   - Left minipage (`\acvLeftColumnWidth`) for main content like experience, projects
   - Right minipage (`\acvRightColumnWidth`) for sidebar content like skills, education, certifications

5. Build with `make resume.pdf` to verify compilation.

## Rules

- Use lowercase filenames (per CLAUDE.md conventions).
- Follow the existing section file format (see `src/resume/experience.tex`, `src/resume/skills.tex`).
- Place `\input` lines in logical order within the column.
