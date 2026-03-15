---
name: add-claude-skill
description: Create or update a Claude skill file in the project. Use this whenever adding a new skill folder, editing an existing SKILL.md, or ensuring skills are mirrored between .claude/skills and .codex/skills.
user-invocable: true
---

# Add or Update a Claude Skill

Create or update a skill file and keep both `.claude/skills` and `.codex/skills` in sync.

## Steps

1. Determine the skill name (kebab-case, e.g. `my-skill`).
2. Check if `.claude/skills/<name>/SKILL.md` already exists — read it if so.
3. Write or update `.claude/skills/<name>/SKILL.md` with:
   - YAML frontmatter: `name`, `description`, `user-invocable: true`
   - Skill body: steps, rules, examples
4. Mirror the same content to `.codex/skills/<name>/SKILL.md` (create the directory if needed).
5. If the skill is new, add a row to the Skills table in `CLAUDE.md`.

## Rules

- Keep both skill versions identical — `.claude` is the source of truth.
- If only updating one file, update both before finishing.
- Skill names are kebab-case and match the folder name.
- Never remove a skill from `CLAUDE.md` without explicit instruction.
