# LaTeX Engineering Research Report

## 1) Executive summary
- For most new technical documents in 2026, defaulting to `latexmk` orchestration and a modular repo layout gives the best reliability-to-complexity ratio; `latexmk` is explicitly designed to run the needed pass graph automatically ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt), [CTAN latexmk](https://ctan.org/pkg/latexmk)).
- Engine choice should be requirement-driven: `pdfLaTeX` for conservative portability, `LuaLaTeX`/`XeLaTeX` for modern OpenType and Unicode workflows via `fontspec` ([Overleaf compiler guide](https://www.overleaf.com/learn/latex/Choosing_a_LaTeX_Compiler%23Other_compilers), [CTAN fontspec](https://ctan.org/pkg/fontspec)).
- `\include` and `\input` are not interchangeable: `\include` forces page breaks and separate `.aux` files and is intended for top-level units; `\input` is finer-grained and nestable ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [TeX FAQ](https://texfaq.org/FAQ-include)).
- Preamble stability is mostly about package order and option ownership; `hyperref` documentation still frames it as a near-last package with explicit compatibility exceptions ([hyperref manual](https://tug.ctan.org/macros/latex/contrib/hyperref/doc/hyperref-doc.html), [CTAN hyperref](https://ctan.org/pkg/hyperref)).
- For bibliography, `biblatex`+`biber` is the most capable modern stack (Unicode, localization, data model control), while classic BibTeX remains simpler and widely supported in legacy templates ([CTAN biblatex](https://ctan.org/pkg/biblatex), [biblatex manual](https://mirrors.ctan.org/macros/latex/contrib/biblatex/doc/biblatex.pdf), [LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).
- Typography quality gains from `microtype` are high impact and low risk in production documents ([CTAN microtype](https://ctan.org/pkg/microtype)).
- For code, `listings` is safer/default CI-friendly; `minted` is higher fidelity but depends on external tooling and shell-escape semantics ([CTAN listings](https://ctan.org/pkg/listings), [CTAN minted](https://ctan.org/pkg/minted), [minted manual](https://tug.ctan.org/macros/latex/contrib/minted/minted.pdf), [TeX FAQ shell-escape](https://texfaq.org/FAQ-spawnprog)).
- Lint + prose quality should be layered: TeX lint (`chktex`) + language/style lint (`Textidote` and/or `Vale`) + pre-commit hooks ([CTAN chktex](https://ctan.org/pkg/chktex), [Textidote](https://github.com/sylvainhalle/textidote), [Vale docs](https://vale.sh/docs), [pre-commit docs](https://pre-commit.com/)).
- CI should compile deterministic targets from explicit roots; `xu-cheng/latex-action` provides a practical baseline for GitHub Actions with root selection and engine options ([latex-action](https://github.com/xu-cheng/latex-action)).
- The best long-term pattern is: small documents stay minimal; medium/large documents migrate shared logic into local `.sty` and enforce build/lint in CI ([CTAN xparse](https://ctan.org/pkg/xparse), [CTAN arara](https://ctan.org/pkg/arara), [Overleaf latexmkrc](https://docs.overleaf.com/managing-projects-and-files/the-latexmkrc-file)).

## 2) Research methodology
### Search strategy (queries used)
- `latex2e include input includeonly`
- `hyperref load order package compatibility`
- `latexmk manual automatic runs`
- `minted shell escape pygments`
- `biblatex biber backend bibtex differences`
- `glossaries makeglossaries xindy makeindex`
- `chktex lacheck textidote vale pre-commit`
- `latex github actions root_file latex-action`
- `awesome-cv repository latex`
- `scientific thesis template latex repository`

Primary retrieval came from package docs, CTAN package pages, TeX FAQ, the LaTeX2e reference manual, and official tooling docs.

### Source selection criteria
- Included as authoritative:
  - Official package documentation and CTAN entries (maintainer-defined behavior and scope).
  - LaTeX Project and TeX ecosystem docs (manual semantics, build behavior).
  - TeX FAQ for canonical operational guidance.
  - Overleaf docs only for workflow clarification and compiler/tooling operational notes.
  - Official tool/project docs for CI/lint (`pre-commit`, `Vale`, `latex-action`).
- Excluded:
  - Unmaintained blogs, SEO posts, and uncited forum summaries.
  - Advice contradicting package manuals without technical evidence.

### Conflict resolution policy
- Priority order: package manual > CTAN package metadata > LaTeX2e/TeX FAQ semantics > operational docs (Overleaf/tool docs).
- When sources differed:
  - I preferred the most recent package/manual statement.
  - I marked operational guidance as context-dependent when not guaranteed by core TeX semantics.
  - I reduced confidence and marked “uncited” where no stable primary source was available.

## 3) Findings

### A. Engines and Unicode (pdfLaTeX vs XeLaTeX vs LuaLaTeX)
- `pdfLaTeX` is still the conservative baseline for compatibility with older templates and package assumptions, while XeLaTeX/LuaLaTeX are better for Unicode/OpenType-heavy projects ([Overleaf compiler guide](https://www.overleaf.com/learn/latex/Choosing_a_LaTeX_Compiler%23Other_compilers), [CTAN fontspec](https://ctan.org/pkg/fontspec)).
- `fontspec` is designed for XeTeX/LuaTeX and is the practical trigger condition to choose those engines ([CTAN fontspec](https://ctan.org/pkg/fontspec)).
- `unicode-math` is explicitly for Unicode math font workflows and therefore aligns with Unicode engines, not classic `pdfLaTeX` pipelines ([CTAN unicode-math](https://ctan.org/pkg/unicode-math)).
- Language handling is no longer binary: `babel` remains central and supports modern engines; `polyglossia` is a common Xe/Lua path when its language model fits project needs ([CTAN babel](https://ctan.org/pkg/babel), [CTAN polyglossia](https://ctan.org/pkg/polyglossia)).

Opinionated conclusion: default to `LuaLaTeX` when you need modern font control or multilingual Unicode; otherwise keep `pdfLaTeX` for speed and compatibility.

### B. Preamble design, package ordering, and conflict management
- The hyperref maintainers still document package compatibility/load-order caveats and recommend near-end loading with specific exceptions ([hyperref manual](https://tug.ctan.org/macros/latex/contrib/hyperref/doc/hyperref-doc.html)).
- Preamble complexity is a maintenance risk; controlling options centrally and minimizing overlapping package responsibility reduces hidden interactions ([CTAN hyperref](https://ctan.org/pkg/hyperref), [LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).
- For team repos, move custom macros and styling into project-local packages (`src/styles/*.sty`) to separate policy from content and reduce preamble drift ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).

### C. Modularity patterns: `\input` vs `\include`, file trees, per-chapter builds
- `\include` forces `\clearpage`, writes separate aux files, cannot be nested, and supports selective compilation with `\includeonly` ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).
- TeX FAQ guidance matches this: use `\include` for major units (chapters), `\input` for fine-grained composition ([TeX FAQ](https://texfaq.org/FAQ-include)).
- Per-chapter builds are best implemented as explicit tool behavior (`latexmk -cd`, subdocument tooling) rather than ad hoc path hacks ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt)).

Practical rule: use `\input` by default; reserve `\include` for chapter-level builds where `\includeonly` is valuable.

### D. Macros and local packages: when to create `.sty`, robust commands, xparse
- `xparse` as a separate package is marked obsolete in the CTAN entry because command parsing interfaces are now in the LaTeX kernel; this is a key modernization signal ([CTAN xparse](https://ctan.org/pkg/xparse)).
- For shared team macros, a local `.sty` with `\ProvidesPackage` is a cleaner maintenance boundary than accumulating raw preamble definitions across files ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).
- Robust command strategy still matters for moving arguments and hyperlinks; prefer modern declarative command definitions and avoid fragile legacy macro patterns where possible ([hyperref manual](https://tug.ctan.org/macros/latex/contrib/hyperref/doc/hyperref-doc.html), [CTAN xparse](https://ctan.org/pkg/xparse)).

### E. Typography and layout quality: microtype, fonts, spacing, floats
- `microtype` is one of the highest-value packages for production typography quality (protrusion/expansion and related micro-typographic controls) ([CTAN microtype](https://ctan.org/pkg/microtype)).
- Table quality is materially better with `booktabs`; package guidance emphasizes typographic rules over vertical-line grids ([CTAN booktabs](https://ctan.org/pkg/booktabs)).
- Page geometry should be managed explicitly with `geometry` rather than class defaults if output constraints matter (conference, print, institutional templates) ([CTAN geometry](https://ctan.org/pkg/geometry)).
- Float placement should be tuned conservatively (`htbp` defaults, deliberate breaks), with package additions only when baseline algorithms fail (uncited; moderate confidence).

### F. Tables/figures/diagrams/code listings: best packages, tradeoffs, pitfalls
- Use `graphicx` as the default for raster/vector inclusion and transformation ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).
- Use `booktabs` for publication-grade tables; avoid “spreadsheet style” ruled boxes ([CTAN booktabs](https://ctan.org/pkg/booktabs)).
- Diagram stack: `pgf/tikz` remains the most integrated native graphics ecosystem for complex technical diagrams ([CTAN pgf](https://ctan.org/pkg/pgf)).
- Code listing decision:
  - `listings`: pure-TeX, no external process dependency, safer CI default ([CTAN listings](https://ctan.org/pkg/listings)).
  - `minted`: better syntax highlighting, but depends on Pygments and shell-escape behavior ([CTAN minted](https://ctan.org/pkg/minted), [minted manual](https://tug.ctan.org/macros/latex/contrib/minted/minted.pdf), [TeX FAQ shell-escape](https://texfaq.org/FAQ-spawnprog)).

### G. Citations and bibliographies: BibTeX vs biblatex/biber, DOI/URL handling
- Classic BibTeX flow (`latex` -> `bibtex` -> `latex` -> `latex`) remains canonical and still works broadly with legacy styles ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).
- `biblatex` is architecturally more capable (style/data model separation, localization, backend choice), with Biber as its full-feature backend ([CTAN biblatex](https://ctan.org/pkg/biblatex), [biblatex manual](https://mirrors.ctan.org/macros/latex/contrib/biblatex/doc/biblatex.pdf), [biber manual](https://mirrors.ctan.org/biblio/biber/base/documentation/biber.pdf)).
- `biblatex` can run with BibTeX backend but with reduced capabilities; teams should treat this as compatibility mode, not target architecture ([biblatex manual](https://mirrors.ctan.org/macros/latex/contrib/biblatex/doc/biblatex.pdf)).
- DOI/URL/eprint display is policy-driven and configurable in modern bibliography stacks; decide and enforce at style level in repo standards ([biblatex manual](https://mirrors.ctan.org/macros/latex/contrib/biblatex/doc/biblatex.pdf)).

### H. Glossaries/acronyms/indexes: workflows and build steps
- `glossaries` is a mature workflow package but requires external indexing/sorting steps; treat it as multi-step build architecture, not one-pass LaTeX ([CTAN glossaries](https://ctan.org/pkg/glossaries), [CTAN glossaries-user](https://ctan.org/pkg/glossaries-user)).
- Index management is easier with tooling packages like `imakeidx`, but CI must explicitly run required indexing commands ([CTAN imakeidx](https://ctan.org/pkg/imakeidx)).
- For large repos, encapsulate glossary/index steps in Make/latexmk/arara rules; avoid manual one-off command sequences ([CTAN arara](https://ctan.org/pkg/arara), [latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt)).

### I. Tooling and automation: latexmk, arara, Makefile/justfile, Docker, CI
- `latexmk` should be the default orchestrator because it tracks dependencies and rerun conditions automatically ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt), [CTAN latexmk](https://ctan.org/pkg/latexmk)).
- `arara` is useful when you want directives inside TeX sources rather than external scripts; this is a workflow style choice, not a universal replacement ([CTAN arara](https://ctan.org/pkg/arara)).
- Overleaf explicitly supports project-level `latexmkrc`, which helps align local and cloud build behavior ([Overleaf latexmkrc](https://docs.overleaf.com/managing-projects-and-files/the-latexmkrc-file)).
- GitHub Actions baseline for LaTeX is straightforward with `latex-action` root selection and engine toggles ([latex-action](https://github.com/xu-cheng/latex-action)).
- Docker-based pinning is often useful for version stability, but exact image strategy is environment-dependent (uncited; moderate confidence).

### J. Linting and style: chktex, lacheck, TeXtidote, Vale, pre-commit
- `chktex` is the practical TeX syntax/style lint baseline; high signal for common anti-patterns and mistakes ([CTAN chktex](https://ctan.org/pkg/chktex)).
- `lacheck` exists but is older and narrower in scope; use mainly for legacy parity, not as sole lint gate ([CTAN lacheck](https://ctan.org/pkg/lacheck)).
- `Textidote` adds language/prose checks around TeX workflows and can complement TeX-specific linting ([Textidote](https://github.com/sylvainhalle/textidote)).
- `Vale` provides customizable editorial style rules and integrates cleanly in docs CI pipelines ([Vale docs](https://vale.sh/docs)).
- `pre-commit` is the easiest way to enforce consistent checks locally before push ([pre-commit docs](https://pre-commit.com/)).

### K. Debugging playbook: reading logs, common errors, checklists
- Build with file/line reporting (`-file-line-error`) and non-interactive mode in CI to make failures actionable ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt)).
- First-failure debugging still beats cascade chasing: fix the earliest fatal error in log order, then rebuild (uncited; high confidence from tool behavior).
- Common failure classes to triage first: missing package/font files, undefined control sequences, path errors in modular repos, stale aux state ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [TeX FAQ](https://texfaq.org/)).
- Use selective compile (`\includeonly`, subset build targets) to shorten turnaround in large projects ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html)).

### L. Security: shell-escape risks, minted, safe CI defaults
- Executing external programs from TeX is a known security-sensitive area; shell escape is intentionally constrained by default in typical setups ([TeX FAQ shell-escape](https://texfaq.org/FAQ-spawnprog)).
- `minted` explicitly relies on external tooling and therefore increases build attack surface relative to pure-TeX listing packages ([minted manual](https://tug.ctan.org/macros/latex/contrib/minted/minted.pdf), [CTAN minted](https://ctan.org/pkg/minted)).
- Safe CI default: avoid unrestricted shell-escape unless required; if required, isolate CI permissions and dependency set (uncited; moderate confidence).

## 4) “Gold standard” repo layouts (copy-paste)

All layouts below are implemented in this repo under [`docs/examples/`](/Users/mahdi/Work/Awesome-CV/docs/examples).

### Small doc
#### File tree
```text
docs/examples/small-doc/
├── latexmkrc
├── main.tex
└── references.bib
```

#### Minimal runnable files
`main.tex`
```tex
\documentclass[11pt]{article}
\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage{lmodern}
\usepackage[final]{microtype}
\usepackage{hyperref}
\title{Small Document Example}
\author{Your Name}
\date{\today}
\begin{document}
\maketitle
This is a minimal small-document layout.
\end{document}
```

`latexmkrc`
```perl
$pdf_mode = 1;
$bibtex_use = 2;
$max_repeat = 5;
```

#### Build commands
- macOS/Linux: `latexmk -pdf -interaction=nonstopmode -file-line-error main.tex`
- Windows (PowerShell): `latexmk -pdf -interaction=nonstopmode -file-line-error .\main.tex`

#### Git-friendly rules
- One sentence per line in prose-heavy documents to reduce diff noise.
- Label format: `sec:...`, `fig:...`, `tab:...`, `eq:...`.
- Keep filenames lowercase with hyphens.
- Keep package imports sorted/grouped by concern (encoding, typography, layout, refs).

### Medium doc
#### File tree
```text
docs/examples/medium-doc/
├── bib/
│   └── references.bib
├── latexmkrc
└── src/
    ├── main.tex
    └── sections/
        ├── intro.tex
        ├── method.tex
        └── results.tex
```

#### Minimal runnable files
`src/main.tex`
```tex
\documentclass[11pt]{report}
\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage{lmodern}
\usepackage[final]{microtype}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{hyperref}
\title{Medium Document Example}
\author{Your Name}
\date{\today}
\begin{document}
\maketitle
\tableofcontents
\input{sections/intro}
\input{sections/method}
\input{sections/results}
\end{document}
```

`src/sections/results.tex`
```tex
\chapter{Results}
\begin{table}[htbp]
\centering
\begin{tabular}{lr}
\toprule
Metric & Value \\
\midrule
Accuracy & 0.95 \\
Runtime (s) & 1.3 \\
\bottomrule
\end{tabular}
\caption{Example table using \texttt{booktabs}.}
\end{table}
```

#### Build commands
- macOS/Linux: `latexmk -cd -pdf -interaction=nonstopmode -file-line-error src/main.tex`
- Windows (PowerShell): `latexmk -cd -pdf -interaction=nonstopmode -file-line-error .\src\main.tex`

#### Git-friendly rules
- Keep each section/chapter in its own file.
- Never mix macro definitions with content files.
- Avoid line-wrap churn in tables/equations; prefer stable formatting conventions.
- CI compiles only canonical root (`src/main.tex`).

### Large doc (chapters, figures, bib, glossary, code)
#### File tree
```text
docs/examples/large-doc/
├── .github/workflows/latex.yml
├── Makefile
├── bib/
│   └── references.bib
├── glossary/
│   └── acronyms.tex
├── build/
├── figures/
└── src/
    ├── main.tex
    ├── appendix/
    │   └── a-reproducibility.tex
    ├── chapters/
    │   ├── 01-intro.tex
    │   ├── 02-methods.tex
    │   └── 03-results.tex
    ├── frontmatter/
    │   └── title.tex
    └── styles/
        └── project.sty
```

#### Minimal runnable files
`src/main.tex`
```tex
\documentclass[12pt]{report}
\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage{lmodern}
\usepackage[english]{babel}
\usepackage[final]{microtype}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage[hidelinks]{hyperref}
\usepackage{styles/project}
\title{Large Document Example}
\author{Your Team}
\date{\today}
\begin{document}
\pagenumbering{roman}
\input{frontmatter/title}
\tableofcontents
\clearpage
\pagenumbering{arabic}
\input{chapters/01-intro}
\input{chapters/02-methods}
\input{chapters/03-results}
\appendix
\input{appendix/a-reproducibility}
\end{document}
```

`src/styles/project.sty`
```tex
\NeedsTeXFormat{LaTeX2e}
\ProvidesPackage{project}[2026/03/04 Project-local macros]
\newcommand{\ProjectName}{LargeDocExample}
```

`Makefile`
```make
LATEXMK=latexmk
MAIN=src/main.tex

.PHONY: pdf clean

pdf:
	$(LATEXMK) -cd -pdf -interaction=nonstopmode -file-line-error $(MAIN)

clean:
	$(LATEXMK) -cd -C $(MAIN)
```

`.github/workflows/latex.yml`
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
          root_file: src/main.tex
          work_in_root_file_dir: true
```

#### Build commands
- macOS/Linux: `make pdf`
- Windows (PowerShell without make): `latexmk -cd -pdf -interaction=nonstopmode -file-line-error .\src\main.tex`
- Windows (with GNU Make): `make pdf`

#### Git-friendly rules
- Prefix chapter files with numeric order (`01-...`, `02-...`) for stable review ordering.
- Keep all local reusable commands in `src/styles/*.sty`.
- Keep figures immutable by name; never overwrite semantics under same filename.
- Require CI on every PR for compile + lint.

Supporting evidence for layout decisions: `\input`/`\include` semantics and `\includeonly` ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [TeX FAQ](https://texfaq.org/FAQ-include)), automation with `latexmk` ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt)), CI action root-file model ([latex-action](https://github.com/xu-cheng/latex-action)).

## 5) Package recommendations (opinionated)

| Category | Package | Purpose | When to avoid | Alternatives | Sources |
|---|---|---|---|---|---|
| Typography | `microtype` | Better line breaks and text color | Rare engine/path edge cases | none practical at same maturity | [CTAN microtype](https://ctan.org/pkg/microtype) |
| Links/refs | `hyperref` | Cross-doc links and PDF metadata | Never omit in digital docs unless class forbids | none (core baseline) | [CTAN hyperref](https://ctan.org/pkg/hyperref), [hyperref manual](https://tug.ctan.org/macros/latex/contrib/hyperref/doc/hyperref-doc.html) |
| Smart refs | `cleveref` | Auto-typed references (`Figure`, `Table`, etc.) | Very small docs or strict class conflicts | manual `\ref` wrappers | [CTAN cleveref](https://ctan.org/pkg/cleveref) |
| Tables | `booktabs` | Typographic-quality horizontal rules | Spreadsheet-like dense matrix tables | `tabularray` (context-dependent, uncited) | [CTAN booktabs](https://ctan.org/pkg/booktabs) |
| Page layout | `geometry` | Explicit margin/layout control | Fixed-template venues that forbid overrides | class defaults | [CTAN geometry](https://ctan.org/pkg/geometry) |
| Graphics | `graphicx` | Standard image inclusion | practically never | none | [LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html) |
| Bibliography | `biblatex` + `biber` | Modern citation model | Legacy-only journals with fixed BibTeX styles | BibTeX + `natbib` | [CTAN biblatex](https://ctan.org/pkg/biblatex), [biblatex manual](https://mirrors.ctan.org/macros/latex/contrib/biblatex/doc/biblatex.pdf), [biber manual](https://mirrors.ctan.org/biblio/biber/base/documentation/biber.pdf) |
| Code blocks | `listings` | Shell-escape-free syntax blocks | Need Pygments-grade highlighting | `minted` | [CTAN listings](https://ctan.org/pkg/listings), [CTAN minted](https://ctan.org/pkg/minted) |
| Glossary | `glossaries` | Acronyms/terms indexes | Tiny docs with no term management | manual acronym macros | [CTAN glossaries](https://ctan.org/pkg/glossaries) |
| Build | `latexmk` | Automatic multi-pass orchestration | Edge workflows needing explicit low-level script control | `arara`, Make | [CTAN latexmk](https://ctan.org/pkg/latexmk), [latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt) |
| Build directives | `arara` | Source-embedded build recipes | Teams preferring externalized build scripts only | Make/Just + latexmk | [CTAN arara](https://ctan.org/pkg/arara) |
| TeX lint | `chktex` | Fast static checks | none; tune rules instead | `lacheck` (legacy) | [CTAN chktex](https://ctan.org/pkg/chktex), [CTAN lacheck](https://ctan.org/pkg/lacheck) |
| Prose lint | `Vale` / `Textidote` | Style and grammar checks | math-heavy docs where prose lint noise dominates | selective rule scopes | [Vale docs](https://vale.sh/docs), [Textidote](https://github.com/sylvainhalle/textidote) |

## 6) Decision tree (engine, bib system, code listings, modularization)
```text
Start
|
+-- Need system/OpenType fonts, multilingual Unicode-heavy content, or unicode math?
|   |
|   +-- Yes -> LuaLaTeX (default) or XeLaTeX if toolchain requires it
|   |          Sources: fontspec, unicode-math, compiler guides
|   |
|   +-- No  -> pdfLaTeX
|
+-- Need advanced citation localization/model customization?
|   |
|   +-- Yes -> biblatex + biber
|   |
|   +-- No  -> BibTeX (+ natbib if needed by template)
|
+-- Need high-fidelity syntax highlighting from Pygments themes?
|   |
|   +-- Yes -> minted (accept shell-escape/tooling constraints)
|   |
|   +-- No  -> listings (safer CI baseline)
|
+-- Project scale?
    |
    +-- Small (<20 pages, low collaboration) -> single root + minimal includes
    |
    +-- Medium (multi-author sections) -> src/sections + \input + latexmk -cd
    |
    +-- Large (chapters, frontmatter, CI, reusable macros) ->
           src/{chapters,frontmatter,styles} + local .sty + CI compile root
```
References: [fontspec](https://ctan.org/pkg/fontspec), [unicode-math](https://ctan.org/pkg/unicode-math), [biblatex](https://ctan.org/pkg/biblatex), [LaTeX2e BibTeX flow](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [listings](https://ctan.org/pkg/listings), [minted](https://ctan.org/pkg/minted), [TeX FAQ include](https://texfaq.org/FAQ-include).

## 7) Myths vs reality
1. Myth: “`\include` is just like `\input` but cleaner.” Reality: `\include` forces page breaks and has separate aux behavior; semantics differ materially ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [TeX FAQ](https://texfaq.org/FAQ-include)).
2. Myth: “Hyperref can be loaded anywhere.” Reality: compatibility/load-order constraints are explicitly documented; near-last loading remains standard guidance ([hyperref manual](https://tug.ctan.org/macros/latex/contrib/hyperref/doc/hyperref-doc.html)).
3. Myth: “BibTeX is obsolete and always wrong.” Reality: BibTeX is still valid in many legacy/journal workflows; `biblatex+biber` is usually better for new projects ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [CTAN biblatex](https://ctan.org/pkg/biblatex)).
4. Myth: “XeLaTeX is always the modern default.” Reality: engine should be requirement-driven; LuaLaTeX often offers better modern pipeline ergonomics, while pdfLaTeX remains viable ([Overleaf compiler guide](https://www.overleaf.com/learn/latex/Choosing_a_LaTeX_Compiler%23Other_compilers), [CTAN fontspec](https://ctan.org/pkg/fontspec)).
5. Myth: “`minted` is just a prettier `listings` swap.” Reality: it adds external dependencies and shell-escape considerations ([CTAN minted](https://ctan.org/pkg/minted), [TeX FAQ shell-escape](https://texfaq.org/FAQ-spawnprog)).
6. Myth: “`xparse` must be loaded for modern commands.” Reality: CTAN marks standalone `xparse` as obsolete because interfaces moved into kernel ([CTAN xparse](https://ctan.org/pkg/xparse)).
7. Myth: “LaTeX builds are inherently flaky.” Reality: deterministic build orchestration is standard with `latexmk` and CI root targeting ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt), [latex-action](https://github.com/xu-cheng/latex-action)).
8. Myth: “Table quality is subjective.” Reality: `booktabs` codifies strong typographic practice that consistently improves technical tables ([CTAN booktabs](https://ctan.org/pkg/booktabs)).
9. Myth: “One linter is enough.” Reality: TeX-structure lint and prose-style lint solve different failure modes ([CTAN chktex](https://ctan.org/pkg/chktex), [Vale docs](https://vale.sh/docs), [Textidote](https://github.com/sylvainhalle/textidote)).
10. Myth: “Cloud editors remove need for local build config.” Reality: even Overleaf supports project-level `latexmkrc`, and CI still needs explicit reproducible configuration ([Overleaf latexmkrc](https://docs.overleaf.com/managing-projects-and-files/the-latexmkrc-file), [latex-action](https://github.com/xu-cheng/latex-action)).

## 8) Case studies

### Case study 1: Modular technical report repo
Reference repo: [Scientific Thesis Template](https://github.com/latextemplates/scientific-thesis-template).

Observed choices:
- Multi-file chapter organization and bibliography separation (repo structure evidence in template tree).
- Build automation around standard TeX tooling.

Tradeoffs:
- Benefit: clearer ownership and lower merge conflict rates in chapter-scale collaboration.
- Cost: path/build orchestration complexity, especially when compiling from different working directories.

Recommendation derived from this pattern:
- Compile from canonical root with `latexmk -cd` or equivalent, standardize CI root file, and enforce local style packages for shared macros ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt), [latex-action](https://github.com/xu-cheng/latex-action)).

### Case study 2: Resume/CV repo with tailorable sections
Reference repos: [Awesome-CV](https://github.com/posquit0/Awesome-CV) and this workspace’s modular resume sources.

Observed choices:
- Tailorable section files (`experience`, `education`, `summary`) and reusable class/macros (`awesome-cv.cls` pattern).
- Separation between content fragments and presentation logic.

Tradeoffs:
- Benefit: fast per-role tailoring with low risk of global formatting regressions.
- Cost: macro-heavy classes can become opaque without conventions and lint/checks.

Recommendation derived from this pattern:
- Keep role-specific toggles/content in separate files, centralize visual rules in class/local style package, and use CI compile for each target variant ([CTAN hyperref](https://ctan.org/pkg/hyperref), [CTAN latexmk](https://ctan.org/pkg/latexmk), [pre-commit docs](https://pre-commit.com/)).

## 9) Final production checklist (printable)
- [ ] Engine decision documented (`pdfLaTeX` vs `LuaLaTeX`/`XeLaTeX`) with rationale.
- [ ] Preamble package order verified; `hyperref` compatibility constraints handled.
- [ ] File tree follows selected size pattern (small/medium/large).
- [ ] Reusable macros moved into local `.sty` package.
- [ ] Labels follow stable conventions (`sec:`, `fig:`, `tab:`, `eq:`).
- [ ] Bibliography stack chosen and pinned (`BibTeX` or `biblatex+biber`).
- [ ] Code listing policy chosen (`listings` default or `minted` with explicit security posture).
- [ ] Glossary/index build steps scripted if used.
- [ ] `latexmk` command standardized for local + CI.
- [ ] CI compiles canonical root file(s) on pull requests.
- [ ] TeX lint enabled (`chktex` minimum).
- [ ] Prose/style lint enabled (`Vale`/`Textidote`) where writing quality matters.
- [ ] Pre-commit hooks installed and documented.
- [ ] Shell-escape disabled by default; enabled only where required and reviewed.
- [ ] Build is reproducible from clean checkout.

Checklist references: [LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [hyperref manual](https://tug.ctan.org/macros/latex/contrib/hyperref/doc/hyperref-doc.html), [latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt), [CTAN minted](https://ctan.org/pkg/minted), [TeX FAQ shell-escape](https://texfaq.org/FAQ-spawnprog), [pre-commit docs](https://pre-commit.com/), [Vale docs](https://vale.sh/docs), [CTAN chktex](https://ctan.org/pkg/chktex).

---

## Three perspectives compared (required synthesis)

### 1) Minimalist core LaTeX
- Stack: class + minimal package set + BibTeX + Make/latexmk.
- Best for: short papers, conservative submission environments.
- Risks: slower migration to Unicode/font requirements and modern citation models.
- Evidence: baseline LaTeX/BibTeX semantics and conservative engine workflows ([LaTeX2e reference manual](https://ctan.math.washington.edu/tex-archive/info/latex2e-help-texinfo/latex2e.html), [Overleaf compiler guide](https://www.overleaf.com/learn/latex/Choosing_a_LaTeX_Compiler%23Other_compilers)).

### 2) Modern developer tooling
- Stack: modular files + local `.sty` + `latexmk` + CI + lint + pre-commit.
- Best for: team docs, long-lived repositories, review-driven workflows.
- Risks: tooling setup overhead and occasional package/container drift.
- Evidence: orchestration/lint/CI tool docs ([latexmk docs](https://tug.ctan.org/support/latexmk/latexmk.txt), [latex-action](https://github.com/xu-cheng/latex-action), [CTAN chktex](https://ctan.org/pkg/chktex), [pre-commit docs](https://pre-commit.com/)).

### 3) Template-driven approach
- Stack: template class (`.cls`) + predefined section structure + optional role/variant files.
- Best for: resumes/CVs and institutional outputs needing consistent branding.
- Risks: hidden complexity inside class files; lock-in to template assumptions.
- Evidence: template repositories and class-centric patterns ([Awesome-CV](https://github.com/posquit0/Awesome-CV), [Scientific Thesis Template](https://github.com/latextemplates/scientific-thesis-template)).

Opinionated synthesis:
- Start minimalist for speed.
- Shift to modern tooling once 2+ contributors or 20+ pages.
- Use template-driven architecture when output consistency is more important than maximal flexibility.
