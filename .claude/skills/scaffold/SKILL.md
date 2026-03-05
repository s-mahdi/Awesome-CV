---
name: scaffold
description: Scaffold a new LaTeX project from small, medium, or large document templates in docs/examples
user-invocable: true
---

# Scaffold LaTeX Project

Generate a new LaTeX project structure based on the reference layouts in `docs/examples/`.

## Steps

1. Ask the user for:
   - **Scale**: small (single-file), medium (modular sections), or large (chapters, CI, .sty, glossary)
   - **Target directory**: where to create the project
   - **Document title and author**

2. Read the matching template from `docs/examples/`:
   - `docs/examples/small-doc/` — single `main.tex` + `latexmkrc` + `references.bib`
   - `docs/examples/medium-doc/` — `src/main.tex` + `src/sections/` + `latexmkrc` + `bib/`
   - `docs/examples/large-doc/` — full structure with `Makefile`, `.github/workflows/latex.yml`, `src/styles/project.sty`, `glossary/`, `figures/`, `build/`

3. Create the file tree in the target directory, substituting title/author into the templates.

4. Confirm the structure with a file listing and suggest the build command.

## Rules

- Follow the exact patterns from the reference examples.
- Use `latexmk` as the build orchestrator (per `docs/latex-research-report.md`).
- For large-doc, include the GitHub Actions workflow with `xu-cheng/latex-action@v4`.
- Keep `hyperref` near the end of the preamble.
- Default to pdfLaTeX for small/medium; LuaLaTeX for large (per research report recommendations).
