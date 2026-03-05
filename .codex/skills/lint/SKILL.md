---
name: lint
description: Lint LaTeX source files for common issues like undefined references, formatting inconsistencies, and prose quality
---

# Lint LaTeX Sources

Review `.tex` files under `src/resume/` (and optionally `src/cv/`) for common issues.

## Checks to Perform

1. **LaTeX correctness**
   - Unclosed environments or braces
   - Undefined or duplicate labels
   - Missing `\item` inside list environments (`cvitems`, `itemize`)
   - Incorrect nesting of environments

2. **Formatting consistency**
   - Consistent use of `\textbf{}` for technology names across all entries
   - Consistent date format via `\acvDateRange`
   - Matching separators (`\acvDashedSeparator` between entries)
   - Two-space indentation inside environments (per CLAUDE.md conventions)

3. **Prose quality**
   - Bullet points start with strong action verbs
   - Metrics and quantified impact are present where possible
   - No orphaned or overly long bullets (aim for 1-2 lines each)
   - Consistent tense (past for previous roles, present for current)
   - No spelling or grammar errors

4. **Build verification**
   - Run `make resume.pdf` and check for warnings (overfull/underfull boxes, undefined references)
   - Report any LaTeX warnings from the build log

## Output

Present findings as a categorized list (Errors / Warnings / Suggestions). Do not make changes unless the user asks.

## Reference

See `docs/latex-research-report.md` section on linting for background on chktex and prose-quality patterns.
