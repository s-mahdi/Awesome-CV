# HR Email Outreach

> Status: Future
> Created: 2026-03-07
> Priority: Low
> Depends on: job-application-pipeline

## Overview

Auto-generate a short, professional email to send to the HR manager or recruiter alongside or after submitting a job application. The email should complement the cover letter without duplicating it.

## Email Structure

```
Subject: Application for {Role Title} - {Your Name}

Hi {HR Name / Hiring Team},

{1-2 sentence hook referencing the role and why you're a strong fit}

{1 sentence pointing to attached resume/cover letter or application portal confirmation}

{1 sentence expressing enthusiasm and availability for next steps}

Best regards,
{Your Name}
```

## Agent Strategy

- **Runner:** Gemini (lightweight text generation)
- **Input:** Job posting summary, cover letter content, company name, HR contact (if known)
- **Output:** `job-list/COMPANY_NAME/hr-email.md` (plain text, ready to copy-paste or send via Gmail MCP)

## Gmail MCP Integration (Future)

- Use `gmail_create_draft` to stage the email in Gmail
- User reviews and sends manually
- Track sent status in ClickUp task

## Open Questions

- [ ] How to find HR contact name? (LinkedIn scraping is out of scope)
- [ ] Should we support follow-up email templates (1-week, 2-week)?
- [ ] Integrate with Gmail MCP `gmail_create_draft` directly?
- [ ] Tone variants: formal / casual-professional / startup-friendly?
