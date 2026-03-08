---
name: ats-evaluate
description: Score the resume against a job description using ATS-aligned evaluation dimensions
user-invocable: true
---

# ATS Resume-to-JD Evaluation

The user will provide a job description (pasted text or URL). Evaluate the current resume against it using ATS-aligned scoring dimensions.

Reference: `docs/research/ATS-evaluation-research.md`

## Steps

1. Read the job description and extract:
   - **Required skills** (must-have)
   - **Preferred skills** (nice-to-have)
   - **Required experience** (years, seniority, domain)
   - **Education / certification requirements**
   - **Target job title(s)**
2. Read the current resume files:
   - `src/resume/summary.tex`
   - `src/resume/skills.tex`
   - `src/resume/experience.tex`
   - `src/resume/education.tex`
3. Score each dimension independently:

   | Dimension | What to evaluate |
   | --- | --- |
   | **Skills match** | Matched, missing required, missing preferred, and adjacent relevant skills. Use semantic matching -- treat synonyms and acronyms as matches (e.g., "K8s" = "Kubernetes"). |
   | **Experience alignment** | Job title relevance, years of experience vs requirement, seniority fit, domain relevance. |
   | **Education & certs** | Degree level and field vs requirements; relevant certifications present or absent. |
   | **Title relevance** | How closely current/past titles match the target role (semantic, not exact). |

4. Assign a categorical match label per dimension:
   - **Strong Match** -- meets or exceeds requirements
   - **Good Match** -- meets most requirements with minor gaps
   - **Partial Match** -- some relevant alignment but notable gaps
   - **Limited Match** -- significant gaps

5. Produce an overall match label (weighted: skills 40%, experience 30%, education 15%, title 15%).

6. Output the evaluation report in this format:

   ```
   ## ATS Evaluation: [Job Title] at [Company]

   **Overall: [Label]**

   ### Skills Match: [Label]
   - Matched: skill1, skill2, ...
   - Missing required: skill3, skill4, ...
   - Missing preferred: skill5, ...
   - Adjacent relevant: skill6, skill7, ...

   ### Experience Alignment: [Label]
   - [Observations about years, seniority, domain fit]

   ### Education & Certifications: [Label]
   - [Observations]

   ### Title Relevance: [Label]
   - [Observations about title similarity]

   ### Recommendations
   1. [Actionable suggestion to improve match]
   2. ...
   ```

## Rules

- Use semantic matching -- never penalize for synonyms, abbreviations, or alternate phrasing.
- Separate hard requirements from nice-to-haves in the skills breakdown.
- Use categorical labels (Strong/Good/Partial/Limited), not false-precision percentages.
- Do not fabricate or assume candidate skills not present in the resume.
- Do not make changes to the resume -- this skill is evaluation only. Suggest the user run `/tailor` afterward if improvements are needed.
- Keep the report concise and actionable.
