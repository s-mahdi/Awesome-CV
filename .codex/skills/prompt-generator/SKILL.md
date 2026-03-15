---
name: prompt-generator
description: Generate polished execution prompts from vague or multi-constraint requests. Use this before browser automation, `claude --chrome`, or any resume-driven job workflow in this repo.
---

# Prompt Generator

Turn a messy request into one clear prompt that another AI runner can execute reliably.

## Use This For

- browser tasks driven by `claude --chrome`
- LinkedIn job evaluation
- recruiter message drafting
- cover-letter preparation
- any workflow where the final prompt must reflect project-specific constraints

## Resume-Driven Job Workflow Rules

When the prompt is for a job workflow, read these files before finalizing the prompt:
- `src/resume/summary.tex`
- `src/resume/experience.tex`
- `src/resume/skills.tex`

The final prompt must preserve the current phase boundary:
- Phase 1: JD capture and fit evaluation only
- no application submission
- no recruiter outreach
- no cover-letter generation unless the caller explicitly says that phase is active

Always include the relevant constraints:
- remote-only roles
- never fabricate details missing from the JD
- reuse `job-list/COMPANY_NAME/` as the local artifact destination when artifacts are mentioned

## Output Format

Always return exactly one fenced code block labeled `prompt`.

## Writing Rules

- Use plain, direct English.
- Make the task order explicit.
- Name the required output sections if the prompt expects structured markdown.
- Include blockers and stop conditions when the task depends on page state or user access.
- Ask targeted clarifying questions only when a missing detail would materially change the prompt.
