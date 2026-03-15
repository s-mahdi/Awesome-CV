# Apply Skill — Phase 0 Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `/apply` skill that accepts a company name and pasted JD, runs fit evaluation + tailoring suggestions + cover letter generation, and saves all artifacts to `job-list/COMPANY_NAME/` — enabling manual job applications from Day 0.

**Architecture:** A thin orchestration skill (`/apply`) coordinates three existing skills/subagents in sequence: `job-fit-evaluator` → `tailor` (suggestions-only) → `coverletter`. It also requires two small updates: removing the hard remote-gate from `job-fit-evaluator`, and documenting the suggestions-only invocation contract in the `tailor` skill. Companion plan-file housekeeping moves the directory structure to match the new phase numbering.

**Tech Stack:** Markdown skill files (`.claude/skills/`, `.codex/skills/`), agent definition files (`.claude/agents/`, `.codex/agents/`), LaTeX (`make coverletter.pdf`), fish shell.

**Spec:** `docs/superpowers/specs/2026-03-15-pipeline-phases-design.md`

---

## Chunk 1: The `/apply` skill

### Files
- Create: `.claude/skills/apply/SKILL.md`
- Create: `.codex/skills/apply/SKILL.md`
- Modify: `CLAUDE.md` (add `/apply` to the skills table)

---

### Task 1: Create `.claude/skills/apply/SKILL.md`

- [ ] **Step 1: Create the skill file**

```markdown
---
name: apply
description: End-to-end Phase 0 apply workflow — given a company name and pasted JD, runs fit evaluation, tailoring suggestions, and cover letter generation, then saves all artifacts to job-list/COMPANY_NAME/
user-invocable: true
---

# Apply to a Job

Orchestrates the full Phase 0 job-application workflow. Accepts a company name and
raw JD text, then produces a ready-to-use artifact folder under `job-list/`.

## Inputs

When invoked, prompt the user interactively:

1. **"Company name?"** — used as the subfolder name under `job-list/`. Normalize to
   `UPPER_SNAKE_CASE` (e.g., "Stripe" → `STRIPE`, "Two Sigma" → `TWO_SIGMA`).
2. **"Paste the job description:"** — the user pastes raw JD text. Accept everything
   until the user signals done (empty line + Enter, or explicit "done").

## Steps

Execute all steps regardless of fit verdict. If a step fails, report the error and
continue with the remaining steps.

### Step 1 — Write job-posting.md

Create `job-list/COMPANY_NAME/job-posting.md` with:
- Company name
- Role title (extract from JD; use "Unknown Role" if not determinable)
- Date: today's date
- Full JD text verbatim

```markdown
# <Role Title> — <Company Name>

> Date: YYYY-MM-DD

## Job Description

<full JD text>
```

### Step 2 — Run job-fit-evaluator

Pass the JD text to the `job-fit-evaluator` subagent. Save its output as
`job-list/COMPANY_NAME/job-analysis.md`.

The evaluator does not require a remote-gate result. Pass only the JD text and
resume file paths.

### Step 3 — Run tailor (suggestions-only)

Invoke the `tailor` skill with this explicit instruction:

> "Output bullet-level tailoring suggestions to `job-list/COMPANY_NAME/resume-notes.md`
> based on the following JD. Do NOT modify any files in `src/resume/`. List each
> suggested change as: [file] → [current bullet] → [suggested rewrite] with the
> JD keyword it targets."

Do not run `make resume.pdf`. Do not write to any `src/` file.

### Step 4 — Run coverletter

Invoke the `coverletter` skill with the JD text. It will write `src/coverletter.tex`
and build `src/coverletter.pdf` via `make coverletter.pdf`.

After the build succeeds, copy `src/coverletter.tex` to
`job-list/COMPANY_NAME/cover-letter.tex`:

```bash
cp src/coverletter.tex job-list/COMPANY_NAME/cover-letter.tex
```

### Step 5 — Report summary

Print a summary:

```
✓ Artifacts saved to job-list/COMPANY_NAME/

  job-posting.md    — JD captured
  job-analysis.md   — Fit: <verdict> (<score>/100)
  resume-notes.md   — <N> tailoring suggestions
  cover-letter.tex  — Cover letter built (coverletter.pdf)

Next steps:
  1. Review job-list/COMPANY_NAME/resume-notes.md and apply any changes manually
  2. Review job-list/COMPANY_NAME/cover-letter.tex — build PDF with: make coverletter.pdf
  3. Submit your application with the generated cover letter PDF
```

## Rules

- No remote-only gate. Apply to any role the user provides.
- Never modify `src/resume/*.tex` files.
- All four output files must be written even if a step returns a poor-fit verdict.
- If `make coverletter.pdf` fails, report the LaTeX error but still copy
  `src/coverletter.tex` to `job-list/COMPANY_NAME/cover-letter.tex`.
- Create `job-list/COMPANY_NAME/` if it does not exist.
```

Save this to `.claude/skills/apply/SKILL.md`.

- [ ] **Step 2: Verify the file exists and is well-formed**

```bash
cat .claude/skills/apply/SKILL.md | head -5
```

Expected: frontmatter `name: apply` on line 2.

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/apply/SKILL.md
git commit -m "feat(skills): add /apply Phase 0 orchestration skill"
```

---

### Task 2: Mirror to `.codex/skills/apply/SKILL.md`

- [ ] **Step 1: Copy to codex**

```bash
mkdir -p .codex/skills/apply
cp .claude/skills/apply/SKILL.md .codex/skills/apply/SKILL.md
```

- [ ] **Step 2: Verify**

```bash
diff .claude/skills/apply/SKILL.md .codex/skills/apply/SKILL.md
```

Expected: no output (files are identical).

- [ ] **Step 3: Commit**

```bash
git add .codex/skills/apply/SKILL.md
git commit -m "feat(skills): mirror /apply skill to codex"
```

---

### Task 3: Register `/apply` in CLAUDE.md

- [ ] **Step 1: Open `CLAUDE.md` and find the Skills table**

The table starts with `| Skill | Description |` under the `## Skills` heading.

- [ ] **Step 2: Add the `/apply` row in alphabetical order**

Insert after the existing `/add-skill` row:

```markdown
| `/apply` | End-to-end Phase 0 apply workflow — fit eval, resume tailoring notes, and cover letter from a pasted JD |
```

- [ ] **Step 3: Verify the table renders correctly**

```bash
grep -A2 "apply" CLAUDE.md
```

Expected: the new row appears.

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "docs(claude): register /apply skill in skills table"
```

---

## Chunk 2: Agent and skill updates

### Files
- Modify: `.claude/agents/job-fit-evaluator.md`
- Modify: `.codex/agents/job-fit-evaluator.md`
- Modify: `.claude/skills/tailor/SKILL.md`
- Modify: `.codex/skills/tailor/SKILL.md`

---

### Task 4: Remove hard remote-gate from `job-fit-evaluator`

The current rule `Respect the remote-only gate result from the JD capture step.` makes
the agent unusable without a prior JD capture step. Phase 0 passes JD text directly.

- [ ] **Step 1: Edit `.claude/agents/job-fit-evaluator.md`**

Replace:
```
- Respect the remote-only gate result from the JD capture step.
```

With:
```
- If a remote-only gate result is provided by a prior JD capture step, respect it. Otherwise, evaluate the role regardless of workplace type.
```

- [ ] **Step 2: Verify**

```bash
grep "remote" .claude/agents/job-fit-evaluator.md
```

Expected: the updated conditional wording appears; the hard rule is gone.

- [ ] **Step 3: Mirror to codex**

```bash
cp .claude/agents/job-fit-evaluator.md .codex/agents/job-fit-evaluator.md
```

- [ ] **Step 4: Verify mirror**

```bash
diff .claude/agents/job-fit-evaluator.md .codex/agents/job-fit-evaluator.md
```

Expected: no output.

- [ ] **Step 5: Commit**

```bash
git add .claude/agents/job-fit-evaluator.md .codex/agents/job-fit-evaluator.md
git commit -m "fix(agent): make remote-gate optional in job-fit-evaluator"
```

---

### Task 5: Document suggestions-only mode in `tailor` skill

The `tailor` skill currently always edits `src/resume/*.tex`. Add a documented
invocation path that suppresses source edits for use by `/apply`.

- [ ] **Step 1: Edit `.claude/skills/tailor/SKILL.md`**

After the existing `## Rules` section, append:

```markdown
## Suggestions-Only Mode

When invoked by the `/apply` skill (or explicitly instructed not to modify source
files), operate in **suggestions-only mode**:

1. Read the JD and resume files as normal.
2. Write a `resume-notes.md` file at the path specified in the instruction with the
   following format per suggestion:

   ```
   [file] → [current text] → [suggested rewrite]
   Targets: <JD keyword>
   ```

3. Do NOT write to any `src/resume/*.tex` file.
4. Do NOT run `make resume.pdf`.
5. Summarize the number of suggestions written.

This mode is triggered when the caller explicitly says "Do not modify any files in
`src/resume/`" or "suggestions-only".
```

- [ ] **Step 2: Verify**

```bash
grep -A5 "Suggestions-Only" .claude/skills/tailor/SKILL.md
```

Expected: the new section appears.

- [ ] **Step 3: Mirror to codex**

```bash
cp .claude/skills/tailor/SKILL.md .codex/skills/tailor/SKILL.md
```

- [ ] **Step 4: Verify mirror**

```bash
diff .claude/skills/tailor/SKILL.md .codex/skills/tailor/SKILL.md
```

Expected: no output.

- [ ] **Step 5: Commit**

```bash
git add .claude/skills/tailor/SKILL.md .codex/skills/tailor/SKILL.md
git commit -m "feat(skills): add suggestions-only mode to tailor skill"
```

---

## Chunk 3: Plan file housekeeping

### Files
- Modify: `docs/plans/phase-1/linkedin-remote-match.plan.md`
- Modify: `docs/plans/phase-2/hr-email-outreach.plan.md`
- Move: `docs/plans/phase-6/` → `docs/plans/phase-5/`
- Move: `docs/plans/phase-5/clickup-tracking.plan.md` → `docs/plans/deferred/`
- Modify: `docs/plans/README.md`
- Modify: `docs/plans/job-application-pipeline.plan.md`

---

### Task 6: Update `linkedin-remote-match.plan.md` — remove remote gate

- [ ] **Step 1: Edit `docs/plans/phase-1/linkedin-remote-match.plan.md`**

In the `## Rules` section, replace:
```
- Treat remote-only as a hard gate.
```
With:
```
- Remote-only is no longer a hard gate. The user decides which roles to pursue.
  If the role's workplace type is captured, include it in job-posting.md for reference.
```

Also update the `## Overview` paragraph to remove the phrase "reject non-remote roles".

- [ ] **Step 2: Verify**

```bash
grep -n "remote" docs/plans/phase-1/linkedin-remote-match.plan.md
```

Expected: no hard-gate language; only informational references.

- [ ] **Step 3: Commit**

```bash
git add docs/plans/phase-1/linkedin-remote-match.plan.md
git commit -m "docs(plans): remove remote-only hard gate from phase-1 plan"
```

---

### Task 7: Update `hr-email-outreach.plan.md` — rename output artifact

- [ ] **Step 1: Edit `docs/plans/phase-2/hr-email-outreach.plan.md`**

Find all occurrences of `hr-email.md` and replace with `outreach-draft.md`.

- [ ] **Step 2: Verify**

```bash
grep "hr-email" docs/plans/phase-2/hr-email-outreach.plan.md
```

Expected: no output (all occurrences replaced).

- [ ] **Step 3: Commit**

```bash
git add docs/plans/phase-2/hr-email-outreach.plan.md
git commit -m "docs(plans): rename output artifact to outreach-draft.md in phase-2 plan"
```

---

### Task 8: Restructure `docs/plans/` directory

Move phase-6 files to phase-5; move clickup to deferred.

- [ ] **Step 1: Create new directories and move files**

```bash
mkdir -p docs/plans/phase-5 docs/plans/deferred
mv docs/plans/phase-6/email-monitoring.plan.md docs/plans/phase-5/
mv docs/plans/phase-6/telegram-notifications.plan.md docs/plans/phase-5/
mv docs/plans/phase-5/clickup-tracking.plan.md docs/plans/deferred/
rmdir docs/plans/phase-6
```

- [ ] **Step 2: Verify**

```bash
ls docs/plans/phase-5/ && ls docs/plans/deferred/
```

Expected:
```
email-monitoring.plan.md  telegram-notifications.plan.md
clickup-tracking.plan.md
```

- [ ] **Step 3: Commit**

```bash
git add docs/plans/
git commit -m "refactor(plans): move phase-6 to phase-5; defer clickup tracking"
```

---

### Task 9: Update `docs/plans/README.md`

- [ ] **Step 1: Update the Plan Catalog table**

Replace all `phase-6/` references with `phase-5/`. Update the `phase-5/clickup-tracking.plan.md` row to show `deferred/clickup-tracking.plan.md` with status `Deferred`. Add the new `phase-0` concept (manual apply) to the Phase Map section.

The Phase Map section should read:

```markdown
| Phase | Goal | Primary Plan Files | Current State |
| --- | --- | --- | --- |
| Phase 0 | Paste JD → fit eval + tailoring notes + cover letter | `/apply` skill | Implement now |
| Foundation | Establish local browser-workflow skills and subagents | [`foundation/chrome-skill-foundation.plan.md`](foundation/chrome-skill-foundation.plan.md) | Implemented |
| Phase 1 | Capture JD from LinkedIn via Chrome | [`phase-1/linkedin-remote-match.plan.md`](phase-1/linkedin-remote-match.plan.md) | Implemented (remote gate removed) |
| Phase 2 | Recruiter/HR outreach draft | [`phase-2/hr-email-outreach.plan.md`](phase-2/hr-email-outreach.plan.md) | Partially implemented |
| Phase 3 | Autofill and submit applications | [`phase-3/auto-apply-fill.plan.md`](phase-3/auto-apply-fill.plan.md) | Planned |
| Phase 4 | Persist artifacts to Google Docs and Sheets | [`phase-4/google-storage.plan.md`](phase-4/google-storage.plan.md) | Planned |
| Phase 5 | Monitor email and send Telegram alerts | [`phase-5/email-monitoring.plan.md`](phase-5/email-monitoring.plan.md), [`phase-5/telegram-notifications.plan.md`](phase-5/telegram-notifications.plan.md) | Planned |
| Deferred | ClickUp tracking | [`deferred/clickup-tracking.plan.md`](deferred/clickup-tracking.plan.md) | Deferred |
```

- [ ] **Step 2: Update the "Not Yet Implemented" list**

Remove `clickup-tracking.plan.md` from the list (it's deferred). Update the `telegram-notifications` and `email-monitoring` paths to `phase-5/`.

- [ ] **Step 3: Commit**

```bash
git add docs/plans/README.md
git commit -m "docs(plans): update README to reflect phase-0 and new directory structure"
```

---

### Task 10: Update `job-application-pipeline.plan.md`

- [ ] **Step 1: Update the Phase Map table**

Replace the existing Phase Map table with the new one matching the spec:

```markdown
| Phase | Goal | Status | Plan File |
| --- | --- | --- | --- |
| 0 | Paste JD → fit eval + tailoring notes + cover letter (manual apply) | **Active** | `/apply` skill (`.claude/skills/apply/SKILL.md`) |
| 1 | LinkedIn JD capture via Chrome | Active | `phase-1/linkedin-remote-match.plan.md` |
| 2 | Recruiter/HR outreach draft | Planned | `phase-2/hr-email-outreach.plan.md` |
| 3 | External application auto-fill and submit | Planned | `phase-3/auto-apply-fill.plan.md` |
| 4 | Google Docs and Google Sheets storage | Planned | `phase-4/google-storage.plan.md` |
| 5 | Gmail monitoring and Telegram alerts | Planned | `phase-5/email-monitoring.plan.md`, `phase-5/telegram-notifications.plan.md` |
```

- [ ] **Step 2: Update the Overview paragraph**

Replace the current implementation target paragraph:

```
The current implementation target is **Phase 0 only**:
- user provides a company name and pastes a job description
- fit is evaluated against the resume
- tailoring suggestions are written to resume-notes.md (source files untouched)
- a cover letter is generated and saved
- all artifacts are stored under `job-list/COMPANY_NAME/` for manual application

Phase 1 (Chrome capture) and beyond are defined but not yet the active target.
```

- [ ] **Step 3: Commit**

```bash
git add docs/plans/job-application-pipeline.plan.md
git commit -m "docs(plans): update master pipeline plan to Phase 0 target"
```

---

## Acceptance Checklist

Run these checks after all tasks are complete:

- [ ] `/apply` skill file exists at `.claude/skills/apply/SKILL.md` and `.codex/skills/apply/SKILL.md`
- [ ] `CLAUDE.md` skills table contains `/apply`
- [ ] `job-fit-evaluator.md` no longer has a hard remote-gate rule
- [ ] `tailor/SKILL.md` documents suggestions-only mode
- [ ] `docs/plans/phase-5/` contains `email-monitoring.plan.md` and `telegram-notifications.plan.md`
- [ ] `docs/plans/deferred/` contains `clickup-tracking.plan.md`
- [ ] `docs/plans/phase-6/` no longer exists
- [ ] `docs/plans/README.md` Phase Map includes Phase 0
- [ ] `docs/plans/job-application-pipeline.plan.md` targets Phase 0
- [ ] `docs/plans/phase-2/hr-email-outreach.plan.md` contains no references to `hr-email.md` (`grep "hr-email" docs/plans/phase-2/hr-email-outreach.plan.md` returns no output)

### Smoke test

Verify `/apply` skill can be invoked end-to-end with a real JD:

1. Run `/apply`
2. Enter company name: `TEST_COMPANY`
3. Paste a short JD (3–5 lines is enough)
4. Confirm all four files appear under `job-list/TEST_COMPANY/`
5. Confirm no `src/resume/*.tex` files were modified (`git diff src/`)
6. Confirm `make coverletter.pdf` succeeds

```bash
# Verify no resume source changes
git diff src/resume/

# Verify artifact folder
ls job-list/TEST_COMPANY/
# Expected: cover-letter.tex  job-analysis.md  job-posting.md  resume-notes.md
```
