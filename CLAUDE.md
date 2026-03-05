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

## Security Note

Default to CI-safe builds without unrestricted shell-escape. Prefer `listings`; use `minted` only when shell-escape and external tooling are explicitly approved.
