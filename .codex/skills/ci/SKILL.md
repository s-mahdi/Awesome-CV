---
name: ci
description: Generate or update a GitHub Actions workflow for building LaTeX documents
---

# Generate CI Workflow

Create or update `.github/workflows/latex.yml` for automated LaTeX builds.

## Steps

1. Check if `.github/workflows/latex.yml` already exists. If so, read it and propose updates.

2. Ask the user for (if not obvious from context):
   - **Documents to build**: resume, cv, coverletter, or all
   - **Engine**: xelatex (default for this project) or lualatex
   - **Triggers**: push, pull_request, or both

3. Generate the workflow based on the pattern from `docs/examples/large-doc/.github/workflows/latex.yml`:

   ```yaml
   name: build-latex
   on: [push, pull_request]
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: xu-cheng/latex-action@v4
           with:
             root_file: src/resume.tex
             work_in_root_file_dir: true
             latexmk_use_xelatex: true
         - uses: actions/upload-artifact@v4
           with:
             name: resume-pdf
             path: src/resume.pdf
   ```

4. For multiple documents, add separate steps or a matrix strategy.

5. Write the workflow file and confirm with the user.

## Rules

- Use `xu-cheng/latex-action@v4` (per docs/examples pattern).
- Pin action versions with `@v4` tags.
- Use `-interaction=nonstopmode -file-line-error` flags implicitly (handled by the action).
- This project uses XeLaTeX (`latexmk -xelatex`), so set `latexmk_use_xelatex: true`.
- Include artifact upload so the PDF is downloadable from the Actions run.

## Reference

See `docs/examples/large-doc/.github/workflows/latex.yml` and `docs/latex-research-report.md` CI section.
