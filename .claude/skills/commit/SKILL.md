---
name: commit
description: Stage and commit changes following atomic commits and Conventional Commits best practices
user-invocable: true
---

# Commit Changes

Create well-formatted git commits following atomic commit principles and Conventional Commits.

## Steps

1. Run `git status` and `git diff` (staged + unstaged) to review all changes.
2. Run `git log --oneline -10` to match the repo's commit message style.
3. **Group changes into atomic commits.** Each commit must represent exactly one logical change — a single feature, fix, refactor, or chore. If changes span multiple concerns, split them into separate commits.
4. For each atomic commit:
   a. Determine the type, scope, and summary:
      - **Type**: `feat`, `fix`, `docs`, `refactor`, `style`, `chore`, `test`, `build`
      - **Scope**: the area touched (`resume`, `cv`, `coverletter`, `makefile`, `docs`, `skills`, etc.)
      - **Summary**: imperative mood, lowercase, no period, max ~72 chars (e.g., "add new experience entry for Neoxi")
   b. Stage only the files belonging to this logical change (by name, never `git add -A` or `git add .`).
   c. Commit using a HEREDOC:
      ```
      type(scope): short imperative summary

      - Explain what changed and why (not how)
      - Additional detail if the change is non-obvious

      Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
      ```
   d. Run `git status` to verify before moving to the next commit.

5. If the user explicitly asks to bundle everything into **one commit**, combine all changes into a single commit with a summary that covers the full scope. Use multiple scopes or a broader scope if needed (e.g., `feat(resume): update experience and skills sections`).

## Commit Message Guidelines

- **Subject line**: imperative mood ("add", "fix", "update"), lowercase after type, no trailing period, max ~72 chars.
- **Body** (optional): explain the "why", not the "what". Wrap at 72 chars. Separate from subject with a blank line.
- **Breaking changes**: prefix body with `BREAKING CHANGE:` if applicable.
- **Footer**: always include the `Co-Authored-By` line.

## Atomic Commit Principles

- Each commit should be self-contained and buildable on its own.
- If you need to refactor before adding a feature, that's two commits: `refactor(scope): ...` then `feat(scope): ...`.
- Group related file changes together (e.g., a new section file + its `\input` line in `resume.tex` = one commit).
- Unrelated changes (e.g., fixing a typo in skills while adding a new experience) should be separate commits.

## Rules

- Follow Conventional Commits as seen in repo history (per CLAUDE.md).
- Never commit files that may contain secrets (`.env`, credentials, tokens).
- Never use `git add -A` or `git add .` — stage specific files by name.
- Never amend previous commits unless explicitly asked.
- Never push unless explicitly asked.
- Never skip hooks (`--no-verify`) unless explicitly asked.
- If a pre-commit hook fails, fix the issue and create a NEW commit (do not amend).
- Pass the commit message via HEREDOC for proper formatting.
- Default to separate atomic commits. Only bundle into one commit if the user explicitly asks.
