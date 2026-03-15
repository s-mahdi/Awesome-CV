<h1 align="center">
  <a href="https://github.com/s-mahdi/Awesome-CV" title="Mahdi's Awesome CV">
    <img alt="AwesomeCV" src="https://github.com/posquit0/Awesome-CV/raw/master/icon.png" width="200px" height="200px" />
  </a>
  <br />
  Mahdi Hoseini's CV
</h1>

<p align="center">
  My professional CV and cover letter built with LaTeX
</p>

<div align="center">
  <a href="https://raw.githubusercontent.com/s-mahdi/Awesome-CV/main/src/resume.pdf">
    <img alt="Resume PDF" src="https://img.shields.io/badge/resume-pdf-green.svg" />
  </a>
  <a href="https://raw.githubusercontent.com/s-mahdi/Awesome-CV/main/src/coverletter.pdf">
    <img alt="Cover Letter PDF" src="https://img.shields.io/badge/coverletter-pdf-green.svg" />
  </a>
</div>

<br />

## What is This Project?

This is **Mahdi Hoseini's** professional CV and cover letter, built using the **Awesome CV** LaTeX template. The template is inspired by [Fancy CV](https://www.sharelatex.com/templates/cv-or-resume/fancy-cv) and provides a clean, semantic markup that's easy to customize.

### Features
- **Professional Design**: Clean and modern layout
- **LaTeX Powered**: High-quality PDF output
- **Customizable**: Easy to modify colors, fonts, and content
- **Version Controlled**: All changes tracked with Git
- **CI/CD Ready**: Automated PDF generation with GitHub Actions


## Quick Start

### Build Commands

```bash
# Build resume
make resume.pdf

# Build cover letter
make coverletter.pdf

# Build everything
make examples

# Clean build artifacts
make clean
```

### Requirements

A full TeX distribution is required. [TeX Live](https://tug.org/texlive/) is recommended.

**macOS:**
```bash
brew install --cask mactex
```

**Ubuntu/Debian:**
```bash
sudo apt-get install texlive-full
```

### Docker Alternative

If you don't want to install TeX locally:

```bash
docker run --rm --user $(id -u):$(id -g) -i -w "/doc" -v "$PWD":/doc texlive/texlive:latest make
```

In either case, this should result in the creation of ``{your-cv}.pdf``

### Job Tracking (Optional)

The `/track-job` skill integrates with [ClickUp](https://clickup.com) to automatically
create a tracked task from any job description, including AI-powered fit evaluation
against your resume.

**Setup:**

1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```

2. Fill in your [ClickUp API token](https://app.clickup.com/settings/apps):
   ```
   CLICKUP_ACCESS_TOKEN=your_clickup_personal_access_token_here
   ```

3. The skill requires [Claude Code](https://claude.ai/code) with the ClickUp MCP
   configured in your Claude settings.

**Usage:**

Paste a raw job description into Claude Code and run `/track-job`. It will:
- Extract metadata (company, title, location, work arrangement, visa, etc.)
- Evaluate fit against your resume (strengths, gaps, verdict, recommendation)
- Create a ClickUp task with all 20 fields pre-populated


## Credit

[**LaTeX**](https://www.latex-project.org) is a fantastic typesetting program that a lot of people use these days, especially the math and computer science people in academia.

[**FontAwesome6 LaTeX Package**](https://github.com/braniii/fontawesome) is a LaTeX package that provides access to the [Font Awesome 6](https://fontawesome.com/v6/icons) icon set.

[**Roboto**](https://github.com/google/roboto) is the default font on Android and ChromeOS, and the recommended font for Google’s visual language, Material Design.

[**Source Sans Pro**](https://github.com/adobe-fonts/source-sans-pro) is a set of OpenType fonts that have been designed to work well in user interface (UI) environments.


## Contact

This is my personal CV repository. Feel free to use the LaTeX template as inspiration for your own resume, but please don't use my personal information or content without permission.

For questions about the LaTeX template or setup, feel free to open an issue on this repository.

## Author
- **Mahdi Hoseini** - [s-mahdi](https://github.com/s-mahdi)

## Template Credits

This CV is built using the **Awesome CV** LaTeX template originally created by [posquit0](https://github.com/posquit0).

### Dependencies
- [**LaTeX**](https://www.latex-project.org) - Typesetting system
- [**FontAwesome6**](https://github.com/braniii/fontawesome) - Icon package
- [**Roboto**](https://github.com/google/roboto) - Primary font
- [**Source Sans Pro**](https://github.com/adobe-fonts/source-sans-pro) - Secondary font
