---
name: coverletter
description: Generate or update a tailored cover letter for a job posting
user-invocable: true
---

# Generate Cover Letter

Create or update `src/coverletter.tex` with a tailored cover letter for a specific job posting.

## Research-backed principles

- The cover letter is a writing sample used during screening — keep tone professional, direct, and personal.
- 250–400 words, four to five paragraphs, half to one page maximum.
- State the exact position name in the first paragraph.
- Use concrete achievements with measurable outcomes — never fabricate.
- Map achievements explicitly to top requirements in the posting; include ATS-relevant keywords.
- Avoid repeating the resume in paragraph form — expand on one or two accomplishments with context.
- Include a closing paragraph with a clear call to action (interview / discussion request).
- 81% of recruiters have rejected candidates based solely on cover letter content — errors and generic content are disqualifying.
- **No section header dividers** — write in flowing business-letter prose paragraphs (see Examples below).

## Steps

1. The user provides a job description (pasted text or URL).

2. Extract the top 3–5 requirements and keywords from the job posting.

3. Read the candidate's resume content:
   - `src/resume/summary.tex`
   - `src/resume/experience.tex`
   - `src/resume/skills.tex`

4. Map two or three of the candidate's strongest, most relevant achievements to those requirements.

5. Read `src/coverletter.tex` for the current template structure.

6. Update `src/coverletter.tex` using the four-paragraph structure below.

## Four-paragraph structure

### Paragraph 1 — Opening (3–5 sentences)
- State "applying for {exact Role Title} at {Company Name}".
- If the user has a referral or spoke with someone at the company, mention it here: "After speaking with {Name} at {event/context}, I…"
- State where the posting appeared if known.
- Show specific, genuine knowledge of the company — mission, product, market position, or a stated value — not generic praise.
- End with a fit claim: "I believe my background in X and Y positions me well to contribute to Z."

### Paragraph 2 — Strongest relevant experience (4–6 sentences)
- Lead with the most relevant role or project.
- Describe what you did and the concrete outcome with a metric.
- Explain why it is directly relevant to the role being applied for.
- Include keywords from the job description naturally.

### Paragraph 3 — Supporting experience or company fit (3–5 sentences)
- Bring in a second experience that covers a different required dimension.
- Or: demonstrate deeper knowledge of the company and explain alignment between the candidate's goals and the role's needs.

### Paragraph 4 — Closing (2 sentences)
- Restate enthusiasm and fit in one sentence.
- Request next steps: "I welcome the opportunity to speak with you further about how I can contribute to {Company}. Thank you for your time and consideration."

## LaTeX structure to use

Remove `\lettersection{}` headers — use plain paragraphs inside `cvletter`:

```latex
\begin{cvletter}

  % Paragraph 1 — Opening
  I am a {background} writing to apply for the {Role Title} position at {Company}.
  After speaking with {Name} at {event}, I … % include only if referral exists
  {Company-specific knowledge and fit claim.}

  % Paragraph 2 — Strongest relevant experience
  {Achievement 1 with metric, mapped to top job requirement.}

  % Paragraph 3 — Supporting experience or company fit
  {Achievement 2 or deeper company/values alignment.}

  % Paragraph 4 — Closing
  I welcome the opportunity to speak with you further about how I can contribute to {Company}.
  Thank you for your time and consideration.

\end{cvletter}
```

## Recipient and header fields

- `\recipient{Hiring Manager Name or "Hiring Team"}{Company Name \\ City, Country}`
- `\lettertitle{Application for {Exact Role Title}}`
- `\letteropening{Dear {Name},}` — use name when known, otherwise `Dear Hiring Manager,`
- `\letterclosing{Sincerely,}`
- `\letterenclosure[Attached]{Resume}`

## Examples (MIT GECD sample letters)

These real examples illustrate the patterns above. Use them as tone and structure references.

### Example A — Environmental Engineer (sustainability consulting)
> "I am a 2013 degree candidate for a Master of Engineering in Environmental Engineering from MIT… Based on my work and educational experience, and perhaps more importantly because of my interest and enthusiasm, I think I am well suited to pursue a career in sustainability consulting.
>
> I have a keen interest in the field of global warming and greenhouse gas management… During my time as a consultant, I was able to distinguish myself as a proficient and motivated employee. In particular I sought to engage in projects that focused on renewable energy, sustainable design, and energy efficiency.
>
> My experience includes: delivering a sustainable technology assessment… Therefore, I am highly confident that I can use my skills, knowledge, and enthusiasm to help businesses develop and implement sustainability initiatives.
>
> I welcome the opportunity to speak with you further about potential career opportunities."

**What works**: Specific thesis topic named, three concrete project types listed, closing is exactly two sentences.

### Example B — Business Analyst at Deloitte (referral-led opening)
> "I am a senior at MIT majoring in biology… I was extremely impressed with Deloitte's approach to consulting after speaking with Yelena Shklovskaya. Deloitte is unique in having the ability to form diverse teams to tackle all the problems a client may have… Therefore, I am writing to request an invitation to interview for a Business Analyst position with Deloitte.
>
> In the past two years, I have been involved in strategy consulting, pharmaceuticals, and government affairs for a non-profit healthcare organization. This summer, I worked in strategy consulting for Putnam Associates. My 6-member team evaluated the marketing efforts for a major pharmaceutical company's organ transplant drug. Through my management of recruitment and interviews with 98 physicians, I obtained primary research…
>
> I have been a volunteer in public policy for 7 years with the March of Dimes… Two years ago, I was appointed Director of Massachusetts Youth Public Affairs…
>
> I welcome the opportunity to speak with you about my qualifications… Thank you and I look forward to hearing from you soon."

**What works**: Referral in sentence 2, specific Deloitte differentiator stated (not generic praise), two entirely different experience areas covered, closing is two sentences.

### Example C — Consulting at Navigant (referral + values alignment)
> "I am a second year master's student in MIT's Technology and Policy Program writing to apply for a consulting position in Navigant's Emerging Technology & Business Strategy group. After speaking with John Smith at the MIT career fair, I realized that Navigant's values of excellence, continuous development, entrepreneurial spirit, and integrity align with the principles that guide me every day… I believe that my knowledge of the energy sector, passion for data analysis, polished communication skills, and four years of consulting experience will enable me to deliver superior value for Navigant's clients.
>
> As a graduate student in MIT's TPP, I spend every day at the cutting edge of the energy sector. In my capacity as an MIT Energy Initiative research assistant, I use statistical analysis to investigate trends in public acceptance and regulation related to emerging energy technologies…
>
> Even before MIT, my four years of work experience in consulting—first at LMN Research Group and then at XYZ Consulting—allowed me to develop the skillset that Navigant looks for in candidates…
>
> I take pride in my skills and experience in several domains: critical thinking and analysis, communication, and leadership. I note that Navigant values these same ideals, and I very much hope to use my abilities in service of the firm and its clients. I look forward to speaking with you when you visit the MIT campus on October 10th."

**What works**: Referral named, specific company values quoted back, explicit "skillset that Navigant looks for" mirrors the job description language, closing names a specific upcoming event.

### Example D — Systems Engineer at Raytheon
> "I am a recent graduate of MIT with a Bachelor of Science degree in Mechanical Engineering… I recently spoke with a Raytheon recruiter at MIT's xFair in February… I admire Raytheon's commitment to defense and security through the use of innovative technologies… I believe that I would make a great fit for the Systems Engineer position.
>
> During my internship with Airbus working with fluid mechanic technology I evaluated wind tunnel and flight test data in order to reduce external airframe noise emissions… I would be excited to use my analytical skills to improve hardware systems, especially early in their life-cycle at Raytheon, when recommendations can have a high impact and positive result for the end user.
>
> In addition to work experience, I have also practiced systems engineering in my coursework…
>
> I am very excited about the work that Raytheon and welcome the opportunity to speak with you further about career opportunities at Raytheon and how I can contribute. Thank you for your time and consideration."

**What works**: Recruiter contact named, one specific internship project with concrete technical detail, explicit bridge between internship outcome and role needs.

## Rules

- Never fabricate experience, companies, or achievements not in the resume.
- Keep the letter to one page; aim for 250–400 words.
- Do not use `\lettersection{}` section headers — write plain prose paragraphs.
- Reference specific technologies and quantified results that match job requirements.
- Do not repeat the resume — expand on selected achievements with context and connection to role.
- Avoid generic phrases: "I am a team player", "passionate about technology", "hard worker".
- Do not include salary expectations or reasons for leaving previous roles.
- Do not use "To Whom It May Concern".
- Proofread for spelling and grammar — errors are disqualifying.
- Match accent color and fonts to the resume (`acvBlue` / `awesome-red` as configured).
- Build with `make coverletter.pdf` to verify compilation after editing.

## Writing Style

- Use clear, simple language.
- Use active voice. Avoid passive voice.
- Use "you" and "your" to address the reader directly where natural.
- Support claims with data and specific examples.
- Use only commas, periods, semicolons, colons, and parentheses for punctuation. Do not use em dashes.
- Avoid constructions like "not just X, but also Y".
- Avoid metaphors and clichés.
- Avoid generalizations.
- Avoid setup phrases: "in conclusion", "in closing", "to summarize", and similar.
- Avoid unnecessary adjectives and adverbs.
- Do not use these words: can, may, just, that, very, really, literally, actually, certainly, probably, basically, could, maybe, delve, embark, enlightening, esteemed, shed light, craft, crafting, imagine, realm, game-changer, unlock, discover, skyrocket, revolutionize, disruptive, utilize, utilizing, tapestry, illuminate, unveil, pivotal, intricate, elucidate, hence, furthermore, however, harness, exciting, groundbreaking, cutting-edge, remarkable, remains to be seen, navigating, landscape, stark, testament, in summary, in conclusion, moreover.
