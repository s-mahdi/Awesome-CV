---
name: add-experience
description: Add a new work experience entry to the resume
---

# Add Experience Entry

Add a new `\cventry` to `src/resume/experience.tex`.

## Steps

1. Ask the user for (if not already provided):
   - Job title
   - Company name
   - Location
   - Start month/year and end month/year (use `0/0` for current/present)
   - 3-5 bullet points describing achievements (with metrics where possible)
2. Read `src/resume/experience.tex` to understand the current format.
3. Insert the new entry at the correct position (most recent first), using:
   - `\cventry{Job title}{Company}{Location}{\acvDateRange{startM}{startY}{endM}{endY}}{...}`
   - `\begin{cvitems}` with `\item` bullets
   - `\acvDashedSeparator` between entries
4. Build with `make resume.pdf` to verify.

## Rules
- Follow the exact formatting of existing entries in the file.
- Bold key technologies with `\textbf{}`.
- Include quantified impact where possible.
