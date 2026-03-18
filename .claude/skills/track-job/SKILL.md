---
name: track-job
description: Ingest a raw job description, analyze it against the resume, and create
  a fully-populated ClickUp tracking task. Use when the user pastes a JD or provides
  a job posting and wants it tracked. Trigger on phrases like "track this job",
  "add this to ClickUp", "log this JD", or when a raw job description is provided.
---

# `/track-job` — JD Ingestion → ClickUp Task

## Trigger

The user provides a raw job description (pasted text or file path) and wants it tracked
in ClickUp. If it is unclear whether the JD was provided, ask before proceeding.

## Step 1 — Accept JD Input

- If the user's message contains the JD text directly, use it as-is.
- If the user references a file path, read the file.
- If neither is present, use `AskUserQuestion` to ask where the JD is before proceeding.

## Step 2 — Dispatch `jd-analyzer` Sub-Agent

Use the Agent tool with `subagent_type: jd-analyzer`. Pass the full JD text in the prompt.

> **Session-start note:** Claude Code discovers project agents at session start. If
> `jd-analyzer` was just created in the current session, use `subagent_type: general-purpose`
> and prepend the full contents of `.claude/agents/jd-analyzer.md` (the body after frontmatter)
> to the prompt so the agent behaves identically. Never skip this step — the analysis
> must run as an isolated sub-agent regardless of which type is available.

Wait for the agent to return a fenced YAML block. Parse all 20 fields from it.

If the YAML block is malformed or missing fields, ask the user whether to retry or proceed
with partial data.

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

Capture `root_folder_id` and `member_folders` from the result for use in Step 7.

## Step 4 — Resolve ClickUp Assignee

Use the `clickup_resolve_assignees` MCP tool to look up the intended assignee.

- Default assignee name: **Mahdi**
- If resolved successfully, capture the `user_id` and `display_name`.
- If not found or ambiguous, use `AskUserQuestion` to confirm the correct member before
  proceeding. **Do NOT create the task until the assignee is confirmed.**

## Step 5 — Create ClickUp Task

Use `clickup_create_task` MCP:

```
list_id:   901816723053
name:      {organization} — {title}   (from YAML)
assignees: [user_id]
```

Capture the returned `task_id` and `task_url`.

## Step 6 — Set All 20 Custom Fields via REST API

Load `CLICKUP_ACCESS_TOKEN` from the environment (it is already in `.env`; run
`source .env` or read via shell substitution).

For each field below, run:

```bash
curl -s -X POST "https://api.clickup.com/api/v2/task/{task_id}/field/{field_id}" \
  -H "Authorization: $CLICKUP_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"value": <mapped_value>}'
```

Run calls sequentially. Log any non-2xx responses as warnings but continue.

### Field ID Map and Value Rules

#### Text / URL / short_text fields — value is the string directly
> **URL-type fields** (`url`, `doc_link`): only set if a valid URL is present. Skip the curl call entirely if the value is `Unknown` — the API rejects non-URL strings with HTTP 400.

| Field | Field ID |
|---|---|
| 🔗 url | `9e47cdc3-30fe-421b-bdd1-4e5eedbabf1d` |
| 📝 description_text | `c0cd63c2-52d3-4622-8b66-c6e820f12a41` |
| 📄 doc_link | `1913844d-5f93-400f-bd73-177a36c3ca4a` |
| 🏢 organization | `edcb1e2d-d75d-4d78-a4b7-93ca59c43fb5` |
| 💼 title | `7305644b-2095-4059-ba04-cf90100bdd6f` |
| 📍 location_derived | `41d545a4-a89f-452a-8e13-0df6eace8e57` |
| 📡 source | `cf06a9e2-e5d6-467a-9dc0-926a50ec85d4` |
| 📋 mandatory_requirements | `ef059aaa-fb94-4552-9b1f-f90730112c88` |
| 🚫 missing_mandatory_skills | `8f7daf33-7a59-4f07-bb48-bcc8c282ff1a` |
| ✅ strengths | `c995bcaa-c9ba-432e-9031-8fee12df113f` |
| 🔍 gaps | `fc0d6ea4-6117-4929-9004-f18620ea65eb` |
| ✏️ resume_changes | `957f0119-1e88-4bba-a4d9-9dc5a85c25b5` |

#### Date field — convert ISO date to Unix milliseconds; use `null` if Unknown

| Field | Field ID |
|---|---|
| 📅 date_posted | `d0b6ddfe-ff18-4541-8ead-7d7259fcad5c` |

To convert: `date -j -f "%Y-%m-%d" "YYYY-MM-DD" "+%s"` × 1000 (macOS).
If `date_posted` is `Unknown`, set value to `null`.

#### Checkbox field — always `false` on creation

| Field | Field ID |
|---|---|
| ☑️ is_applied | `05888566-82b3-40d9-b318-889effa28454` |

Value: `false`

#### Dropdown fields — value is the option `orderindex` (integer)

| Field | Field ID | Options |
|---|---|---|
| 🏠 work_arrangement | `87fa3320-893d-40c5-8ca8-67ebff208f5d` | Remote=0, Hybrid=1, Onsite=2 |
| ⚖️ verdict | `8f627ca6-a6a0-4773-a6a1-8cc674b829a6` | Pass=0, Fail=1, Maybe=2 |
| 🛂 visa_sponsorship | `f0fc5021-86ba-428c-a454-d16a104c66f4` | Yes=0, No=1, Unknown=2 |
| ✈️ relocation | `d3dbfba3-647c-4763-a2ac-2a3fb96565bf` | Yes=0, No=1, Unknown=2 |
| 🔴 knockout_factor_check | `d91b28a7-037e-4a63-a452-bcd58b87261c` | Pass=0, Fail=1 |
| 🎯 final_recommendation | `96852b69-b6bc-4e6a-b0dd-88ceb3a07d7a` | Apply=0, Skip=1, Maybe=2 |

## Step 7 — Create Drive Folder and Google Doc

Dispatch a **Haiku** sub-agent (`subagent_type: general-purpose`, model: `claude-haiku-4-5-20251001`)
to create the job artifacts in Google Drive. Pass it the following context and instructions:

Context to inject into the prompt:
- Assignee display name (from Step 4)
- `member_folders` map (from Step 3): `{ "<name>": "<folder_id>" }`
- `root_folder_id` (from Step 3)
- Job title (YAML `title` field)
- Organization (YAML `organization` field)
- Full JD text (from Step 1)

> **Drive Artifact Agent Instructions**
>
> 1. Look up the assignee's folder ID from `member_folders` using the assignee display name
>    (case-insensitive match). If no match is found, use `root_folder_id` as the parent.
>
> 2. Create a subfolder named `[title] - [organization]` inside the assignee's folder
>    using gws-drive.
>
> 3. Create a Google Doc inside that new subfolder using gws-docs.
>    - Doc title: `[title] - [organization]`
>    - Doc body: the full job description text, verbatim.
>
> 4. Return JSON:
>    ```json
>    { "folder_url": "...", "doc_url": "..." }
>    ```

Capture `folder_url` and `doc_url` from the result for the Step 8 report.

If the agent returns an error, log it as a warning and continue — Drive artifact creation
is non-blocking for the overall task.

## Step 8 — Report Result

Print a summary in this format:

```
✅ ClickUp task created: {organization} — {title}
🔗 {task_url}

Drive Artifacts:
  📁 {folder_url}
  📄 {doc_url}

Field Summary:
  verdict:              {verdict}
  final_recommendation: {final_recommendation}
  knockout_factor_check:{knockout_factor_check}
  work_arrangement:     {work_arrangement}
  visa_sponsorship:     {visa_sponsorship}
  relocation:           {relocation}
```

If any field calls failed, list them under "⚠️ Failed fields:" with the error.
If Drive artifact creation failed, note it under "⚠️ Drive:" with the error.

## Error Handling

- **No JD provided** → ask before any action
- **gws install/auth failure** → surface error via `AskUserQuestion`, block until resolved
- **Assignee not found** → ask before creating task
- **YAML parse failure** → show the raw output and ask whether to retry
- **ClickUp task creation fails** → surface the error; do not proceed to field-setting
- **Individual field curl failures** → log as warning, continue with remaining fields
- **Drive folder/doc creation fails** → log as warning, still complete the report
