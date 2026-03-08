# ATS Resume-to-JD Evaluation Reference

Research-backed reference for scoring a resume against a job description, distilled from official documentation of six major ATS platforms (Workday, Greenhouse, Oracle Recruiting, SmartRecruiters, iCIMS, Lever).

---

## How ATS Evaluation Works

ATS platforms evaluate resumes in two stages:

1. **Parsing** -- converts the resume into structured fields (skills, job titles, dates, employers, education, contact details)
2. **Matching** -- compares parsed fields to job requisition criteria and produces ranked lists, match scores, or match categories

Match scores are consistently framed as **decision support**, not autonomous hiring decisions. Recruiters remain the decision-makers.

---

## Scoring Dimensions

These are the evaluation dimensions documented across major ATS platforms, ordered by prevalence:

### 1. Skills match (universal)

Every reviewed platform anchors matching around skills.

- Compares candidate skills extracted from resume against required/preferred skills in the job description
- Tracks: matched skills, missing required skills, and adjacent relevant skills not listed in the JD
- Several platforms use **semantic matching** (embeddings, NLP) rather than exact keyword matching -- synonym and acronym resolution is common

### 2. Experience alignment (universal)

- Job titles compared to required titles (including semantic similarity)
- Years of experience extracted from employment dates
- Seniority level and career trajectory/progression
- Work history relevance to the role

### 3. Education and certifications (common)

- Degree level and field of study compared to requirements
- Certifications matched against JD requirements
- Weight is configurable -- some platforms let admins increase/decrease education weight relative to skills and experience

### 4. Screening-question outcomes (common)

- Hard disqualifiers based on structured yes/no or multiple-choice questions (e.g., "Do you have X license?")
- This is the **only documented mechanism for true auto-rejection** -- distinct from match scoring
- Match scores do NOT trigger auto-rejection in any reviewed platform

### 5. Job title relevance (some platforms)

- Current and past job titles scored for relevance to the target role
- Semantic matching handles synonyms (e.g., "Software Engineer" ~ "Software Developer")

### 6. Industry alignment (some platforms)

- Derived industry classification from prior employers
- Used as a secondary signal, not a primary scoring dimension

---

## Scoring Model Patterns

### Multi-dimension ensemble scoring

SmartRecruiters documents this most explicitly: each dimension (skills, experience, education, title relevance, career progression) is scored separately, then combined into a **weighted ensemble score** with job-specific weights.

### Match categories

Rather than a single percentage, most platforms produce **categorical labels**:

- Greenhouse: Strong Match, Good Match, Partial Match, Limited Match
- SmartRecruiters: Very High, High, Medium, Low
- Oracle: 0--5 scale per dimension (education, experience, skills, profile)

### Recruiter calibration

Greenhouse and Oracle let recruiters define and weight criteria before scoring. The system then calculates match scores against those weighted criteria.

---

## Three-Layer Evaluation Model

A well-designed resume-to-JD evaluation should separate three distinct layers:

| Layer | What it measures | Example |
| --- | --- | --- |
| **Parsing quality** | Extraction coverage and error handling | Were all skills, titles, and dates correctly parsed? |
| **Match strength** | Skills and experience alignment with explainable gaps | 7/10 required skills matched; missing: Kubernetes, Terraform |
| **Workflow gating** | Hard disqualifiers from screening questions or business rules | "Do you require sponsorship?" -- Yes (auto-reject) |

Key insight: match scoring and auto-rejection are **separate mechanisms**. Greenhouse explicitly separates AI match scoring (assistive, no auto-disposition) from auto-reject rules (deterministic, question-based).

---

## Explainability Artifacts

Effective ATS-style evaluation should surface these signals (documented across multiple vendors):

- **Matched skills** -- skills present in both resume and JD
- **Missing required skills** -- JD requirements absent from the resume
- **Adjacent relevant skills** -- candidate skills not in the JD but worth consideration
- **Semantic matches** -- skills or titles matched via similarity rather than exact keyword (highlighted distinctly from exact matches)
- **Experience gaps** -- required experience level vs demonstrated experience
- **Score justification summary** -- plain-language explanation of the overall match assessment

---

## Common Misconceptions

| Misconception | Reality |
| --- | --- |
| "ATS auto-rejects based on keyword matches" | Auto-rejection is documented only for screening-question rules, never for match scores. Match scores support prioritization, not disposition. |
| "ATS matching = exact keyword matching" | Multiple platforms use semantic matching via embeddings and NLP -- synonyms, acronyms, and related terms are resolved. |
| "Formatting determines rejection" | Parse failures trigger manual data entry fallback, not automatic rejection. Formatting affects data capture quality, not disposition. |
| "Match scores decide hiring outcomes" | Vendors consistently frame scores as decision support. Recruiters and hiring managers make advancement and rejection decisions. |
| "Every ATS uses a single match percentage" | Outputs vary: categorical labels, 0--5 scales, multi-dimension breakdowns, weighted ensembles. There is no universal format. |

---

## Practical Implications for Resume-to-JD Scoring

1. **Anchor on skills match** -- it is the single most universal and heavily weighted dimension
2. **Use semantic matching** -- don't penalize resumes for using synonyms or alternate phrasing
3. **Separate hard requirements from nice-to-haves** -- mirror how ATS platforms separate screening-question gating from match scoring
4. **Score multiple dimensions independently** then combine -- mirroring the ensemble approach (skills, experience, education, title relevance)
5. **Surface explainable gaps** -- list matched, missing, and adjacent skills rather than just a number
6. **Use categorical labels over false-precision percentages** -- match the ATS pattern of Strong/Good/Partial/Limited rather than implying 73.2% accuracy
7. **Account for parsing variability** -- extraction quality affects downstream scoring; flag low-confidence parses
