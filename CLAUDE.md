# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in this repository.

## Project Overview

LaTeX-based CV/resume/cover-letter project built on [Awesome-CV](https://github.com/posquit0/Awesome-CV). The main build uses `latexmk -xelatex`.

## Repository Map

- `src/`: production documents.
  - roots: `src/resume.tex`, `src/cv.tex`, `src/coverletter.tex`
  - modular sections: `src/resume/*.tex`, `src/cv/*.tex`
  - assets: `src/assets/`
- `build/`: packaged artifact output from `make build`.
- `awesome-cv.cls`: shared class; changes affect all documents.
- `docs/examples/`: research-backed reference patterns:
  - `small-doc/` (single-file + `latexmkrc`)
  - `medium-doc/` (modular `\input` structure)
  - `large-doc/` (chapter layout, local `.sty`, glossary, CI workflow)
- `docs/plans/`: future feature plans (`.plan.md` files with Mermaid diagrams).
- `docs/logs/`: session logs tracking decisions and rationale.
- `docs/research/`: research artifacts for skills and features.
- `job-list/`: per-company application artifacts (cover letters, match reports, reviews).

## Build and Debug Commands

```bash
make                    # build resume.pdf, cv.pdf, coverletter.pdf
make resume.pdf         # build resume only
make cv.pdf             # build cv only
make coverletter.pdf    # build cover letter only
make build              # copy src/resume.pdf to build/ release filename
make clean              # remove generated PDFs and aux files
```

Example commands from `docs/examples`:

```bash
latexmk -pdf -interaction=nonstopmode -file-line-error docs/examples/small-doc/main.tex
latexmk -cd -pdf -interaction=nonstopmode -file-line-error docs/examples/medium-doc/src/main.tex
make -C docs/examples/large-doc pdf
make -C docs/examples/large-doc glossary
```

## Editing Conventions

- Keep files UTF-8 and XeLaTeX-compatible.
- Use two-space indentation inside LaTeX environments.
- Keep filenames lowercase and focused (`summary.tex`, `languages.tex`).
- Prefer `\input` for modular composition; reserve `\include` for chapter-level split with `\includeonly`.
- Keep shared macros in local `.sty` files when logic grows (see `docs/examples/large-doc/src/styles/project.sty`).
- Load `hyperref` near the end of the preamble.
- Keep `microtype` and `booktabs` unless there is a specific layout reason to remove them.

## Validation and CI

- Primary quality gate is successful compile plus visual PDF review.
- Run `make clean && make` before finalizing edits.
- Use `-interaction=nonstopmode -file-line-error` for actionable failures.
- CI compiles deterministic roots (`make` here; `root_file: src/main.tex` in the large-doc example workflow).

## Commits and PRs

- Follow Conventional Commits seen in repo history (`feat(resume): ...`, `fix(makefile): ...`, `docs(readme): ...`).
- Use scoped messages for the touched area (`resume`, `cv`, `makefile`, `docs`).
- Include summary, linked issue (if any), and updated PDFs/screenshots when visual output changes.

## Skills

Custom skills live in `.claude/skills/` and are invoked with `/skill-name`:

| Skill | Description |
| ----- | ----------- |
| `/add-experience` | Add a new work experience entry to the resume |
| `/add-section` | Add a new section to the resume by creating a `.tex` file and wiring it into `resume.tex` |
| `/add-claude-skill` | Create or update a Claude skill file and mirror it between `.claude/skills` and `.codex/skills` |
| `/add-skill` | Add or update resume skills, and mirror any new agent skill in both `.claude/skills` and `.codex/skills` |
| `/ats-evaluate` | Score the resume against a job description using ATS-aligned evaluation dimensions |
| `/build` | Build the resume PDF and report any LaTeX errors |
| `/find-skills` | Discover and install agent skills from the open skills ecosystem (skills.sh) |
| `/ci` | Generate or update a GitHub Actions workflow for building LaTeX documents |
| `/commit` | Stage and commit changes following atomic commits and Conventional Commits best practices |
| `/coverletter` | Generate or update a tailored cover letter for a job posting |
| `/lint` | Lint LaTeX source files for common issues like undefined references, formatting inconsistencies, and prose quality |
| `/review` | Review the resume for clarity, impact, consistency, and ATS-friendliness |
| `/scaffold` | Scaffold a new LaTeX project from small, medium, or large document templates in `docs/examples` |
| `/tailor` | Tailor the resume for a specific job posting by adjusting summary, skills, and experience bullets |
| `/track-job` | Analyze a JD against the resume and create a ClickUp task in the Tracking list with evaluation details, plus upload the JD to Google Drive |

## Sub-Agents

Custom sub-agents live in `.claude/agents/` and are auto-discovered by Claude Code:

| Agent | Model | Description |
| ----- | ----- | ----------- |
| `search` | Haiku | Perform web research on a topic and save structured findings to `docs/research/` |
| `jd-analyzer` | Sonnet | Extract 20 job-tracking metrics from a JD + resume; returns YAML block for fit evaluation. Dispatched by `/track-job`. |

## Multi-Agent Strategy

This project uses multiple AI tools to optimize cost and quality:

- **Claude (Opus):** Orchestrator, strategic content generation, complex reasoning.
- **Gemini CLI:** Job-match evaluation, ATS scoring, tone/grammar review, company research.
- **Codex CLI:** LaTeX code generation, syntax validation, mechanical tasks.

Offload extraction, classification, and validation to Gemini/Codex. Reserve Claude for orchestration and tasks requiring deep reasoning. See `docs/plans/` for detailed architecture (Mermaid diagrams).

## Diagrams

Use Mermaid syntax (`\`\`\`mermaid`) for all architecture and flow diagrams in plan and documentation files.

## Security Note

Default to CI-safe builds without unrestricted shell-escape. Prefer `listings`; use `minted` only when shell-escape and external tooling are explicitly approved.

