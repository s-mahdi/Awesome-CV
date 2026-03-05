---
name: add-skill
description: Add a new skill category or update an existing one in the resume
---

# Add or Update Skill

Modify `src/resume/skills.tex` to add new skills.

## Steps

1. Read `src/resume/skills.tex` to see existing skill blocks.
2. If the user wants to add to an existing category, append the new skills to that `\skillblock`.
3. If the user wants a new category, add a new `\skillblock{Category}{skills...}` with `\acvDashedSeparatorRoomy` before it.
4. Build with `make resume.pdf` to verify.

## Rules
- Match the existing formatting style.
- Keep skill lists comma-separated within each `\skillblock`.
- Do not duplicate skills already listed.
