# Session Log: Sub-Agent Planning

**Date:** 2026-03-07

## Context

Researched and planned multi-agent architecture for the Awesome-CV project. The goal is to automate the job application pipeline using Claude as orchestrator while offloading work to Gemini CLI and Codex CLI to minimize token costs.

## Decisions Made

1. **Gemini CLI** is the primary runner for job-match evaluation and ATS scoring — performs well for this task and is cost-effective.
2. **Codex CLI** handles LaTeX code generation and syntax validation — good at mechanical, tool-driven tasks.
3. **Claude** orchestrates the pipeline and handles strategic content generation and review.
4. **Multi-agent review panel** (3 reviewers: Codex, Gemini, Claude) will review cover letters from different angles before finalizing.
5. **Cover letter is required** per application; resume changes are optional suggestions only.
6. **Output structure:** `job-list/COMPANY_NAME/` with all artifacts per application.

## Plans Created

| Plan File | Status | Priority |
|-----------|--------|----------|
| `job-application-pipeline.plan.md` | Planning | High |
| `clickup-tracking.plan.md` | Future | Medium |
| `hr-email-outreach.plan.md` | Future | Low |
| `email-monitoring.plan.md` | Future | Medium |
| `auto-apply-fill.plan.md` | Planning | High |

## 2026-03-08: Auto-Fill Plan Added

Researched browser automation tools for auto-filling job applications on career portals. Evaluated Stagehand, browser-use, Skyvern, OpenAI/Claude computer use, JobCopilot, and Simplify.

**Decision:** Use **Playwright + browser-use** as primary stack.

Key design choices:
1. **Mandatory human approval** before every submission — screenshot + field summary review.
2. **Answer memory** (`job-list/.answer-memory.json`) stores previously approved screening question answers for reuse.
3. **ATS platform detection** — specific strategies for Greenhouse, Lever, Workday; AI vision fallback for unknown portals.
4. **Profile data** (`job-list/.profile.json`) provides deterministic fields (name, email, phone, URLs).
5. **Sensitive fields** (salary, EEO, sponsorship) always require human input, never auto-filled.
6. This becomes **Phase 5** of the existing job application pipeline, after the review panel.

## 2026-03-08: Job Match Evaluation Prompt & Gemini Thinking Model

Created `docs/prompts/job-match-evaluation.prompt.md` — full evaluation prompt for Phase 1 (job-resume fit scoring).

**Gemini CLI research findings:**
- Gemini CLI v0.32.1 installed at `/opt/homebrew/bin/gemini`
- Model `gemini-2.5-pro` has thinking/reasoning on by default — no extra flags needed
- Headless mode: `-p` flag for non-interactive, `-o text` for plain output
- System instructions via `GEMINI.md` in project root (auto-loaded)
- No `--thinking-budget` or `--system-instruction` CLI flags — those are API-only parameters

**Updated:** `job-application-pipeline.plan.md` Phase 1 now references the prompt file, specifies `gemini-2.5-pro` model, and includes the exact CLI invocation command.

## Next Steps

- ~~Create Gemini prompts for job-match evaluation~~ Done
- Implement the `/apply` skill as the main orchestrator
- Set up `job-list/` folder structure conventions
- Wire up cover letter generation pipeline
- Prototype Playwright + browser-use for Greenhouse form filling
- Design `.profile.json` schema and populate with user data
- Build answer memory read/write utilities
