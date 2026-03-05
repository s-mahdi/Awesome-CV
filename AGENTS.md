# Repository Guidelines

## Project Structure & Module Organization
- `src/` is the CV codebase. Roots: `src/resume.tex`, `src/cv.tex`, and `src/coverletter.tex`; sections live in `src/resume/*.tex` and `src/cv/*.tex`.
- `src/assets/` stores images used by the documents; `build/` contains copied artifacts from `make build`.
- `docs/examples/` is the reference lab: `small-doc/` (single-file), `medium-doc/` (modular `\input` sections), and `large-doc/` (chapters, local `.sty`, glossary, CI/Makefile).
- `awesome-cv.cls` is a shared class dependency; treat edits as cross-document changes.

## Build, Test, and Development Commands
- `make`: Compile all top-level documents (`resume.pdf`, `cv.pdf`, `coverletter.pdf`) using `latexmk -xelatex`.
- `make resume.pdf` / `make cv.pdf` / `make coverletter.pdf`: Compile one document only.
- `make build`: Copy `src/resume.pdf` into `build/` with the release filename.
- `make clean`: Remove generated PDFs and LaTeX auxiliary files.
- Example builds from `docs/examples/`: `latexmk -pdf -interaction=nonstopmode -file-line-error docs/examples/small-doc/main.tex`, `latexmk -cd -pdf -interaction=nonstopmode -file-line-error docs/examples/medium-doc/src/main.tex`, `make -C docs/examples/large-doc pdf`, and `make -C docs/examples/large-doc glossary`.

## Coding Style & Naming Conventions
- Keep files UTF-8 and compatible with XeLaTeX.
- Follow existing LaTeX style: two-space indentation inside environments and concise `%` section comments.
- Keep section filenames short and lowercase (for example, `summary.tex`, `languages.tex`).
- Prefer modular content with `\input`; reserve `\include` for chapter-scale documents where `\includeonly` is needed.
- Keep shared macros/layout policy in local `.sty` files (see `docs/examples/large-doc/src/styles/project.sty`) and load `hyperref` near the end of the preamble.
- Keep typography defaults like `microtype` and `booktabs` unless a document has a specific reason to diverge.

## Testing Guidelines
- There is no unit-test framework; successful compilation is the primary check.
- Before submitting changes, run `make clean && make`.
- Use `-interaction=nonstopmode -file-line-error` for actionable failures.
- Validate generated PDFs in `src/` for layout regressions (overflow, spacing, page breaks, alignment, table quality).
- CI compiles deterministic root files (`make` in this repo; `root_file: src/main.tex` in the large example workflow).

## Commit & Pull Request Guidelines
- Use Conventional Commits, matching existing history (for example, `feat(resume): ...`, `fix(makefile): ...`, `docs(readme): ...`).
- Keep commit scopes tied to touched areas (`resume`, `cv`, `makefile`, `README`, etc.).
- PRs should include a short summary, linked issue (if any), and updated PDFs or screenshots when visual output changes.
- Confirm local build success (`make`) in the PR description.

## Security & Tooling Notes
- Default to CI-safe builds without unrestricted shell-escape. For code blocks, prefer `listings`; use `minted` only when the team explicitly accepts its external tooling and shell-escape requirements.
