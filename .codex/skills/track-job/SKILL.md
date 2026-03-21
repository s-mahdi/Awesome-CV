---
name: track-job
description: Analyze a JD against the resume and create a ClickUp task in the Tracking
  list with evaluation details, plus upload the JD to Google Drive. Use when the user
  pastes a JD or says "track this job".
user-invocable: true
---

# `/track-job` — JD → ClickUp Task + Google Drive

## Trigger

The user provides a raw job description (pasted text or file path) and wants it tracked.
If it is unclear whether the JD was provided, ask before proceeding.

## Step 1 — Accept JD Input

- If the user's message contains the JD text directly, use it as-is.
- If the user references a file path, read the file.
- If neither is present, use `AskUserQuestion` to ask where the JD is.

## Step 2 — Dispatch `jd-analyzer` Sub-Agent

Use the Agent tool with `subagent_type: jd-analyzer`. Pass the full JD text in the prompt.

> **Session-start note:** Claude Code discovers project agents at session start. If
> `jd-analyzer` was just created in the current session, use `subagent_type: general-purpose`
> and prepend the full contents of `.claude/agents/jd-analyzer.md` (the body after frontmatter)
> to the prompt so the agent behaves identically.

Wait for the agent to return a fenced YAML block. Parse these fields:

- `organization`, `title`, `work_arrangement`, `location_derived`, `source`, `date_posted`, `url`
- `visa_sponsorship`, `relocation`
- `mandatory_requirements`, `missing_mandatory_skills`, `strengths`, `gaps`
- `verdict`, `knockout_factor_check`, `final_recommendation`, `resume_changes`

If the YAML block is malformed or missing fields, ask the user whether to retry or proceed
with partial data.

## Step 2b — Qualification Gate

Evaluate whether the job is **qualified** for artifact upload using these rules:

- `knockout_factor_check` must be `pass` (case-insensitive)
- `final_recommendation` must NOT be `skip` or `reject` (case-insensitive)

Set a boolean flag `IS_QUALIFIED = true` if both conditions are met, otherwise `false`.

> **If `IS_QUALIFIED = false`:** skip Step 6 entirely. Still run Steps 3–5 to log
> the job in ClickUp. In the Step 7 report, replace the Drive Artifacts section with:
> `⏭️  Drive upload skipped — job did not qualify (verdict: {verdict})`

## Step 3 — Google Drive Setup

Dispatch a **Haiku** sub-agent (`subagent_type: general-purpose`, model: `claude-haiku-4-5-20251001`)
to perform all Drive setup checks and provisioning. Pass it these instructions verbatim:

> **Drive Setup Agent Instructions**
>
> Work autonomously through each step. Do not stop on failures — fix them instead.
>
> **1. Verify or install the gws CLI**
>    - Run `which gws 2>/dev/null || echo "NOT_FOUND"`.
>    - If NOT_FOUND, detect the OS:
>      - macOS → run `brew install gws`. If brew is unavailable, fall back to
>        `npm install -g @google/gws`.
>      - Linux/Windows → run `npm install -g @google/gws`.
>    - After installing, run `gws version` to confirm success.
>    - If installation fails, output `{ "error": "gws install failed: <reason>" }` and stop.
>
> **2. Verify authentication**
>    - Run `gws whoami 2>/dev/null || echo "NOT_AUTHENTICATED"`.
>    - If NOT_AUTHENTICATED, run `gws login` to open the browser OAuth flow.
>    - After login, re-run `gws whoami` and confirm it returns a valid email.
>    - If still unauthenticated, output `{ "error": "gws auth failed" }` and stop.
>
> **3. Locate or create the "Yerevan Job Application" root folder**
>    - Use the gws-drive skill to search for a folder named exactly
>      `Yerevan Job Application` in the user's Drive root.
>    - If found, capture its id as ROOT_FOLDER_ID.
>    - If not found, create the folder using gws-drive and capture the new id as ROOT_FOLDER_ID.
>
> **4. Fetch ClickUp members and share the root folder with each**
>    - Call the `clickup_get_workspace_members` MCP tool to retrieve the full member list.
>    - Extract each member's display name and email.
>    - Use gws-drive to list current permissions on ROOT_FOLDER_ID.
>    - For any member whose email is not already a permission, share the folder with them
>      using gws-drive with `role: writer`.
>
> **5. Ensure per-member subfolders exist inside the root folder**
>    - For each ClickUp member display name, search for a subfolder with that exact name
>      inside ROOT_FOLDER_ID using gws-drive.
>    - If a subfolder does not exist, create it inside ROOT_FOLDER_ID using gws-drive.
>    - Build a map of `{ "<member_display_name>": "<subfolder_id>", ... }`.
>
> **Return** a JSON object:
> ```json
> { "root_folder_id": "...", "member_folders": { "<name>": "<folder_id>" } }
> ```

Wait for the Haiku agent to complete. If it returns an `error` key, surface the message
to the user with `AskUserQuestion` and **do not proceed** until resolved.

Capture `root_folder_id` and `member_folders` from the result for use in Step 6.

## Step 4 — Resolve ClickUp Assignee

- Default assignee name: **Mahdi**
- Use `clickup_find_member_by_name` or `clickup_resolve_assignees` MCP to resolve the user ID.
- If not found or ambiguous, use `AskUserQuestion` to confirm the correct member.
- **Do NOT create the task until the assignee is confirmed.**

## Step 5 — Create ClickUp Task

Use `clickup_create_task` MCP with:

- `list_id`: `901816813219`
- `name`: `{organization} — {title}`
- `assignees`: `[resolved_user_id]`
- `status`: `to evaluate`
- `markdown_description`: formatted analysis (see template below)

### Markdown description template

```
## Job Details

| Field | Value |
| ----- | ----- |
| Organization | {organization} |
| Title | {title} |
| Location | {location_derived} |
| Arrangement | {work_arrangement} |
| Source | {source} |
| Posted | {date_posted} |
| URL | {url} |
| Visa Sponsorship | {visa_sponsorship} |
| Relocation | {relocation} |

## Evaluation

| Field | Value |
| ----- | ----- |
| Verdict | {verdict} |
| Knockout Check | {knockout_factor_check} |
| Recommendation | {final_recommendation} |

### Mandatory Requirements
{mandatory_requirements}

### Missing Skills
{missing_mandatory_skills}

### Strengths
{strengths}

### Gaps
{gaps}

### Suggested Resume Changes
{resume_changes}
```

## Step 6 — Create Drive Folder and Upload Artifacts _(qualified jobs only)_

**Skip this step entirely if `IS_QUALIFIED = false`** (set in Step 2b).

Dispatch a **Haiku** sub-agent (`subagent_type: general-purpose`, model: `claude-haiku-4-5-20251001`)
to create the job artifacts in Google Drive. Pass it the following context and instructions:

Context to inject into the prompt:
- Assignee display name (from Step 4)
- `member_folders` map (from Step 3): `{ "<name>": "<folder_id>" }`
- `root_folder_id` (from Step 3)
- Job title (YAML `title` field)
- Organization (YAML `organization` field)
- Full JD text (from Step 1)
- Absolute paths to the built PDFs: `src/resume.pdf` and `src/coverletter.pdf`
  (resolve relative to the repo root — use `pwd` if needed)

> **Drive Artifact Agent Instructions**
>
> Work autonomously. Do not stop on non-fatal errors — log them and continue.
>
> 1. Look up the assignee's folder ID from `member_folders` using the assignee display name
>    (case-insensitive match). If no match is found, use `root_folder_id` as the parent.
>    Call this `ASSIGNEE_FOLDER_ID`.
>
> 2. Create a subfolder named `[organization] — [title]` inside `ASSIGNEE_FOLDER_ID`
>    using:
>    ```bash
>    gws drive files create \
>      --json '{"name": "[organization] — [title]", "mimeType": "application/vnd.google-apps.folder", "parents": ["ASSIGNEE_FOLDER_ID"]}'
>    ```
>    Capture the returned `id` as `JOB_FOLDER_ID`.
>
> 3. Upload the **resume PDF** into `JOB_FOLDER_ID`:
>    ```bash
>    gws drive files create \
>      --json '{"name": "resume.pdf", "parents": ["JOB_FOLDER_ID"]}' \
>      --upload /absolute/path/to/src/resume.pdf \
>      --upload-content-type application/pdf
>    ```
>    Capture the returned `id` as `RESUME_FILE_ID`.
>
> 4. Upload the **cover letter PDF** into `JOB_FOLDER_ID`:
>    ```bash
>    gws drive files create \
>      --json '{"name": "coverletter.pdf", "parents": ["JOB_FOLDER_ID"]}' \
>      --upload /absolute/path/to/src/coverletter.pdf \
>      --upload-content-type application/pdf
>    ```
>    Capture the returned `id` as `COVERLETTER_FILE_ID`.
>
> 5. Create a **Google Doc** for the job description inside `JOB_FOLDER_ID` using gws-docs.
>    - Doc title: `[organization] — [title] (JD)`
>    - Doc body: the full job description text, verbatim.
>    Capture the doc URL as `JD_DOC_URL`.
>
> 6. Return JSON:
>    ```json
>    {
>      "folder_url": "https://drive.google.com/drive/folders/JOB_FOLDER_ID",
>      "resume_url": "https://drive.google.com/file/d/RESUME_FILE_ID/view",
>      "coverletter_url": "https://drive.google.com/file/d/COVERLETTER_FILE_ID/view",
>      "jd_doc_url": "JD_DOC_URL"
>    }
>    ```

Capture `folder_url`, `resume_url`, `coverletter_url`, and `jd_doc_url` from the result
for the Step 7 report.

If the agent returns an error on any individual upload, log it as a warning and still
return partial results — Drive artifact creation is non-blocking for the overall task.

## Step 6.5 — Append Row to Google Sheets

Always run this step regardless of `IS_QUALIFIED`.

Append one row to the spreadsheet using the `gws` CLI directly (no sub-agent needed):

```bash
gws sheets spreadsheets values append \
  --params '{
    "spreadsheetId": "1sobd140tey4shkVlIu2EFRYrnAzZy4nNsqToOPezu6Q",
    "range": "Template!A:T",
    "valueInputOption": "USER_ENTERED"
  }' \
  --json '{
    "values": [[
      "{title}",
      "{organization}",
      "{url}",
      "{task_url}",
      "{description_text_truncated}",
      "{jd_doc_url_or_empty}",
      "{work_arrangement}",
      "{location_derived}",
      "{source}",
      "{date_posted}",
      "{verdict}",
      "{visa_sponsorship}",
      "{relocation}",
      "{mandatory_requirements}",
      "{missing_mandatory_skills}",
      "{strengths}",
      "{gaps}",
      "{knockout_factor_check}",
      "{final_recommendation}",
      "{resume_changes}"
    ]]
  }'
```

**Field notes:**
- `description_text_truncated`: first 1000 characters of the raw JD text (avoids cell size limits)
- `jd_doc_url_or_empty`: use `jd_doc_url` from Step 6 if qualified; otherwise empty string `""`
- `task_url`: the ClickUp task URL returned by Step 5
- All other fields: direct values from the jd-analyzer YAML output

If the append call fails, log a warning and continue — Sheets logging is non-blocking.

## Step 7 — Report

Print a summary:

```
✅ {organization} — {title}

   Status:         To Evaluate
   Assignee:       {assignee_name}
   Verdict:        {verdict}
   Recommendation: {final_recommendation}
   Task:           {task_url}
   Sheets:         ✅ Row appended   (or ⚠️ Sheets append failed: <error>)

Drive Artifacts:
   📁 Folder:       {folder_url}
   📄 Resume:       {resume_url}
   📄 Cover Letter: {coverletter_url}
   📝 JD Doc:       {jd_doc_url}
```

If any Drive artifact failed, note it under "⚠️ Drive:" with the error message.

## Error Handling

- **No JD provided** → ask before any action
- **gws install/auth failure** → surface error via `AskUserQuestion`, block until resolved
- **Assignee not found** → ask before creating task
- **YAML parse failure** → show the raw output and ask whether to retry
- **ClickUp task creation fails** → surface the error; do not proceed
- **Drive folder/doc creation fails** → log as warning, still complete the report
