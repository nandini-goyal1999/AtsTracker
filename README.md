# AtsTracker: AI Recruiting Pipeline

An AI-assisted recruiting workflow built in **n8n**. It screens every new application against the job description, routes each candidate automatically, sends the right email, follows up, and sends a weekly recruiting report. A Google Sheet works as a lightweight ATS (applicant tracking system).

Built for a sample **Data Science Analyst** role.

## What it does

1. **Collects applications** through a Google Form (name, email, resume PDF).
2. **Screens each resume with AI.** OpenAI reads the resume and returns a score (0 to 100), 3 strengths, 2 gaps and a recommendation.
3. **Routes the candidate automatically** by score:

| Score | Stage | Email sent |
|---|---|---|
| 70 or higher | `Assessment` | Assessment invite |
| 40 to 69 | `Review` | "Application received", plus an alert to the hiring lead |
| Below 40 | `Rejected` | Polite rejection |

4. **Sends stage emails** when the hiring lead moves a candidate to `Interview` or `Hired` (interview invite, welcome email with an onboarding checklist). Each email is sent only once.
5. **Follows up daily.** Candidates waiting in `Assessment` or `Review` for 3+ days are flagged to the hiring lead.
6. **Reports weekly.** Every Monday it counts candidates per stage, total and new applications, and the average score, writes a short AI summary, emails it to management and logs it in the `Weekly Report` tab.
7. **Alerts on failure.** If a workflow fails, an email names the failed step and links to the run.

## Flow

```
Google Form ──> Google Sheet (ATS) ──> 01 Intake & Screening
                                         │  download PDF → extract text → AI score
                                         ├─ 70+    → Assessment + invite email
                                         ├─ 40-69  → Review + alert to hiring lead
                                         └─ <40    → Rejected + rejection email

Hiring lead sets Stage ──> 02 Stage Emails (Interview / Hired)
Daily 9 AM ─────────────> 03 Follow-ups (3+ days without contact)
Monday 9 AM ────────────> 04 Weekly Report (email + Weekly Report tab)
```

## Results from testing

Tested with **15 test candidates** with made-up names and resumes, ranging from strong to weak matches. All were submitted in under 30 seconds.

| Outcome | Candidates |
|---|---|
| Assessment (score 70+) | 5 |
| Review (score 40 to 69) | 3 |
| Rejected (score below 40) | 7 |

- Scores ranged from **3 to 96**, with an average of **49**.
- Every candidate was scored, logged with strengths and gaps, moved to the right stage and emailed. No manual steps.

## Workflows

| File | Trigger | What it does |
|---|---|---|
| `workflows/01-intake-screening.json` | New form row | AI screening, scoring, routing and the first email |
| `workflows/02-stage-emails.json` | Every 2 minutes | Interview and Hired emails, sent once per candidate |
| `workflows/03-follow-ups.json` | Daily, 9 AM | Flags candidates with no contact for 3+ days |
| `workflows/04-weekly-report.json` | Monday, 9 AM | Stage counts, average score, AI summary, email and log |

## Sheet columns (ATS)

`Timestamp, Candidate Name, Candidate Email, Upload Resume, Score, Strengths, Gaps, Recommendation, Stage, Notes, Last Contacted`

A second tab, `Weekly Report`, stores one row per weekly report.

## Tech stack

- **n8n** (workflow automation)
- **OpenAI** (resume scoring and report summaries)
- **Google Forms, Google Sheets, Google Drive, Gmail**

## How to run it

1. Create a Google Form with **Candidate Name**, **Candidate Email** and **Upload Resume** (file upload, PDF), and link it to a new Google Sheet.
2. Add the extra columns listed above to the responses tab, rename the tab `Candidates`, and add a `Weekly Report` tab.
3. In n8n, import the 4 JSON files from `workflows/` (**⋯ → Import from file**).
4. Connect your credentials: Google Sheets, Google Drive, Gmail and OpenAI.
5. Replace `YOUR_SHEET_ID` with your Sheet ID and `hiring-lead@example.com` with your email.
6. Replace `[assessment link]` and `[booking link]` in the email templates with real links.
7. Publish all 4 workflows and submit a test application through the form.

## Repo structure

```
AtsTracker/
├── workflows/   # 4 n8n workflow exports (JSON)
├── ss/          # screenshots: sheet, emails, weekly report
└── README.md
```

## Screenshots

**ATS sheet after AI screening and routing**
![ATS sheet](ss/Screenshot%202026-11-25%20151103.png)

**Assessment invite (score 70+)**
![Assessment email](ss/Screenshot%202026-11-25%20151204.png)

**Application received (Review, score 40 to 69)**
![Received email](ss/Screenshot%202026-11-25%20151233.png)

**Rejection (score below 40)**
![Rejection email](ss/Screenshot%202026-11-25%20151258.png)

**Weekly recruiting report**
![Weekly report](ss/Screenshot%202026-11-25%20151054.png)

## Notes

- All test candidates are fictional. Test emails used `+` aliases of one inbox.
- The AI score is a screening aid. Candidates in `Review` always go to a person before any decision.
