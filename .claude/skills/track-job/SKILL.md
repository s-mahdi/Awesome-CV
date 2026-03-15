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

## Step 3 — Resolve ClickUp Assignee

Use the `clickup_resolve_assignees` MCP tool to look up the intended assignee.

- Default assignee name: **Mahdi**
- If resolved successfully, capture the `user_id`.
- If not found or ambiguous, use `AskUserQuestion` to confirm the correct member before
  proceeding. **Do NOT create the task until the assignee is confirmed.**

## Step 4 — Create ClickUp Task

Use `clickup_create_task` MCP:

```
list_id:   901816723053
name:      {organization} — {title}   (from YAML)
assignees: [user_id]
```

Capture the returned `task_id` and `task_url`.

## Step 5 — Set All 20 Custom Fields via REST API

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

## Step 6 — Report Result

Print a summary in this format:

```
✅ ClickUp task created: {organization} — {title}
🔗 {task_url}

Field Summary:
  verdict:              {verdict}
  final_recommendation: {final_recommendation}
  knockout_factor_check:{knockout_factor_check}
  work_arrangement:     {work_arrangement}
  visa_sponsorship:     {visa_sponsorship}
  relocation:           {relocation}
```

If any field calls failed, list them under "⚠️ Failed fields:" with the error.

## Error Handling

- **No JD provided** → ask before any action
- **Assignee not found** → ask before creating task
- **YAML parse failure** → show the raw output and ask whether to retry
- **ClickUp task creation fails** → surface the error; do not proceed to field-setting
- **Individual field curl failures** → log as warning, continue with remaining fields
