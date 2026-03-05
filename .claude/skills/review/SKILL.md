---
name: review
description: Review the resume for clarity, impact, consistency, and ATS-friendliness
user-invocable: true
---

# Review Resume

Perform a thorough review of the resume without making changes (unless asked).

## Steps

1. Read all active resume section files:
   - `src/resume/summary.tex`
   - `src/resume/experience.tex`
   - `src/resume/skills.tex`
   - `src/resume/education.tex`
   - `src/resume/languages.tex`
2. Evaluate against these criteria:
   - **Clarity**: Are bullets concise and easy to scan?
   - **Impact**: Do bullets lead with results and include metrics?
   - **Consistency**: Is tense, formatting, and style uniform across entries?
   - **ATS-friendliness**: Are keywords present for common ATS parsers? Are there formatting issues that could confuse parsers?
   - **Length**: Does everything fit on one page without feeling cramped?
3. Present findings as a prioritized list of suggestions.
4. Only make edits if the user explicitly asks.
