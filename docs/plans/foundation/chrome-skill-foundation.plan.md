# Chrome Skill Foundation

> Status: Active
> Created: 2026-03-15
> Priority: High

## Overview

This plan defines the project-local skills and subagents that support the browser-driven job workflow. It also records the narrow Claude settings change needed to reduce permission prompts for `claude --chrome`.

## Skills

### `prompt-generator`
- Source to import and normalize: `/Users/mahdi/.claude/skills/prompt-writer/prompt-writer.md`
- Project targets:
  - `.claude/skills/prompt-generator/SKILL.md`
  - `.codex/skills/prompt-generator/SKILL.md`
- Responsibility:
  - turn vague workflow requests into one final execution prompt
  - require repo-specific context for resume-driven job tasks
  - emit the final prompt in a fenced `prompt` block

### `chrome-linkedin-match`
- Project targets:
  - `.claude/skills/chrome-linkedin-match/SKILL.md`
  - `.codex/skills/chrome-linkedin-match/SKILL.md`
- Responsibility:
  - read resume context
  - use `prompt-generator` before building the browser prompt
  - run `claude --chrome`
  - route output through JD capture and fit-evaluation subagents
  - stop after fit evaluation in the current phase

## Subagents

### `jd-capture`
- Purpose: normalize browser output into structured LinkedIn job metadata and clean JD text

### `job-fit-evaluator`
- Purpose: compare normalized JD content with the resume and return a score, verdict, and evidence

### `recruiter-message-drafter`
- Purpose: later-phase draft generation when a recruiter or hiring contact is visible

### `coverletter-prep`
- Purpose: later-phase handoff package for the existing `coverletter` skill

## Prompt Template

- Canonical browser prompt template lives in `docs/prompts/chrome-linkedin-match.prompt.md`.
- The generated runtime prompt should always include:
  - current phase boundary
  - remote-only filter
  - required JD fields
  - recruiter/contact extraction if visible
  - structured markdown output contract

## Project-Local Claude Settings

- Add a narrow allow rule for `claude --chrome`.
- Do not add global settings in this phase.
- If the exact command still triggers approval because of unsupported matching behavior, record that limitation and keep the allow rule as narrow as possible.
