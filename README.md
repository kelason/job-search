# Job Search — Philippines Edition (`ph-job-search`)

A personal job-search skill for a Philippines-based professional. It searches five Philippine job boards, scores each posting against your profile, and generates a tailored, ATS-ready resume + cover letter for every match — bundled in a zip, with application tracking.

> **Private repo.** [`.agents/skills/ph-job-search/SKILL.md`](.agents/skills/ph-job-search/SKILL.md) contains a personal profile (name, email, location, current employer, target roles). Do not make this repository public without removing that section first.

## What it does

- Searches **only** LinkedIn PH, Indeed PH, JobStreet PH, Kalibrr, and Glassdoor PH — Philippines-only, still-open postings only.
- Scores every result with a **Fitness Score** (skills overlap, experience, domain, seniority, project relevance, education).
- Generates a **tailored resume + cover letter per job**, checks them against the job description's keywords (70%+ ATS keyword match), and bundles everything in `[Name]_Applications_[Date].zip` — one folder per company.
- Verifies links are live before generating materials, filters out stale/expired postings, and tracks applications from `Found` to `Offer` / `Rejected` / `Ghosted`.

It does **not** submit applications for you. Boards require your own logged-in session (and often CAPTCHA), and many Apply buttons redirect to employer ATS sites (Workday, Greenhouse, Lever, etc.). You review and click Submit; the skill marks a job `Applied` only on your confirmation.

## Repository layout

| Path | What it is |
|---|---|
| [`.agents/skills/ph-job-search/SKILL.md`](.agents/skills/ph-job-search/SKILL.md) | The skill itself — instructions, profile, commands, and rules. This is the only skill file tracked. |
| [`README.md`](README.md) | This file. |
| [`.gitignore`](.gitignore) | Whitelist ignore: everything is ignored except `.gitignore`, `README.md`, and the skill. |

Local job-search output is **not** tracked in git (see [Ignored local files](#ignored-local-files)).

## Setup

### 1. Use the skill

Copy or keep the skill at:

```text
.agents/skills/ph-job-search/SKILL.md
```

Any agent that loads project skills from `.agents/skills/` will pick it up as `ph-job-search`. Invoke it with the commands below, or just use natural language (e.g. “Find Laravel jobs remote PH”).

### 2. Set your profile (first run only)

On first use the skill collects — or reuses the profile already in `SKILL.md`:

- **Resume** — PDF, DOCX, or plain text. Parsed for name, contact links, roles, stack, education, PH location, work-mode preferences, salary (PHP/month), and notice period.
- **Job preferences** — target roles, target locations (Metro Manila, Cebu, Davao, Clark, Iloilo, Remote PH), company type, seniority.
- **Optional** — salary range, industries/companies to focus on or avoid, deal-breakers (night shift, onsite-only, relocation), benefits expectations.

It summarizes the profile back for confirmation, then never asks again in that session.

### 3. Optional: Gmail status checks

For `/ph-job-search status` with Gmail connected, the skill reads only emails from companies in your tracker and classifies them (rejected, interview invite, offer, acknowledged, assessment). Without Gmail it uses manual updates. Gmail is always optional.

## Commands

| Command | What happens |
|---|---|
| `/ph-job-search help` | Shows the quick reference only. No search, no applications. |
| `/ph-job-search search` | Searches the 5 boards for PH-only matches, scores them, generates resume + cover letter for every result, returns a table + zip. |
| `/ph-job-search status` | Checks tracked applications (Gmail auto-detection or manual), flags 21+ day-old pending items as likely ghosted, shows stats. |
| `/ph-job-search automate` | Sets up a nightly scheduled search (you pick the PHT time/frequency) with a morning report. |

Natural language works too:

- “Find Laravel jobs remote PH”
- “Search Vue roles in Makati under 30 days”
- “Apply to this job: [paste URL]”
- “Status of my applications?”
- “Set up nightly search at 11 PM”

## How search works

### Platforms (5 only)

| # | Platform | Best for |
|---|---|---|
| 1 | LinkedIn PH | MNC and product roles |
| 2 | Indeed PH | Largest aggregator, provincial coverage |
| 3 | JobStreet PH | Largest PH volume, salary data |
| 4 | Kalibrr | Tech/startup PH roles |
| 5 | Glassdoor PH | Reviews + salary alongside listings |

All other boards (OnlineJobs.ph, Wellfound, WeWorkRemotely, Upwork, Bossjob, etc.) and company career pages are excluded unless you explicitly name one for that run — it is then labelled user-requested in the report.

### Philippines-only rule (hard)

A posting counts only if its location is in the Philippines (Remote PH / WFH Philippines, Metro Manila, Clark, Cebu, Davao, or another PH locality) or it explicitly states PH-based / PH-remote eligibility. Global-remote, US/EU-only, onsite-abroad, and location-unspecified postings are dropped. If PH eligibility cannot be confirmed from the posting, it is excluded — never guessed.

### Results table

Every result is a complete application package:

| Job Title | Company | Job ID | Platform | Location | Posted Date | Fitness Score | Cover Letter | Resume | Apply Link |
|---|---|---|---|---|---|---|---|---|---|

- **Job ID** — platform ID from the URL, else `[Platform]-[Company]-[ShortTitle]`.
- **Fitness Score** — e.g. `85% Fit` plus a short reason. 80%+ strong, 60–79% moderate, below 60% stretch (flagged).
- **Apply Link** — direct posting link, verified live this session. Never a search-results page.
- Below the table: 1–2 lines per result on fit and what was customized, plus the zip with one folder per company.

### Verification (before any materials are generated)

| Tier | Meaning | Label |
|---|---|---|
| Verified live | Posting page fetched this session, active Apply path seen | `Ready to Apply` |
| Listing-corroborated | Board blocks automated fetches (401/403/login wall); direct URL + job text + freshness from the board's own index | `Verify in your account first` |

Anything meeting neither tier is excluded, with one line in the report per exclusion (expired, non-PH, wrong board, search-page link, unverifiable). A bare HTTP 200 is never proof — e.g. JobStreet serves expired postings with 200, so the page body is checked for expiry flags.

### Freshness rules

- **0–7 days:** preferred, lead with these.
- **8–14 days:** include normally.
- **15–30 days:** include with a “verify before applying” flag, ranked below fresher matches.
- **30+ days:** excluded by default (at most 2–3 shown as “older but possibly open” if fresh results are under 3).
- Hard drops: 404/410, “position filled/closed”, dead apply link, past deadline.
- Your override (“last 7 days”, “include older”) always wins.

### Resume + cover letter per job

- **Resume:** 1–2 pages, ATS-safe (no tables/columns/graphics), mirrors exact job-description keywords, bullets reordered by relevance, role-targeted summary. Internal ATS keyword check must reach 70%+ (the score itself is not shown — Fitness Score is what you see). Experience, dates, degrees, and skills are never invented.
- **Cover letter:** under 300 words, mirrors 3–5 keywords, why this company, 2–3 mapped achievements, availability + PHP salary expectation if requested.
- **Tone:** plain, concrete, human. No AI clichés (“leveraged”, “spearheaded”, “results-driven”, “cutting-edge”, “passionate about”).

## Application tracking

Tracked in `job_tracker.xlsx` (local, ignored by git — see below):

`Date Found, Company, Role, Job ID, Platform, Location, Posted Date, Job URL, Fitness Score, Resume, Cover Letter, Status, Date Applied, Response Date, Outcome, Interview Stage, Notes, Follow-up Date, Next Action`

Status flow:

```text
Found → Ready to Apply → Applied → Acknowledged → Assessment
      → Interview R1/R2 → HR Round → Offer → Rejected → Ghosted (21+ days)
```

Manual updates work in plain language: “Globe — got interview”. Follow-ups are surfaced at 7–10 days after applying.

## Nightly automation (`automate`)

You pick a PHT time and frequency (nightly, weekdays, every other day, weekly). Example conversions (PHT = UTC+8):

| PHT | UTC cron |
|---|---|
| 11:00 PM | `0 15 * * *` |
| 11:30 PM | `30 15 * * *` |
| 12:00 AM | `0 16 * * *` |

Weekdays only: add `1-5` as day-of-week. Scheduled runs are unattended: no interactive questions, all target roles/locations, materials for Fitness ≥ 60%, tracker updated, morning report in this format:

```text
DAILY JOB SEARCH REPORT — [Date] (PHT)
SUMMARY / TOP MATCHES / OTHER MATCHES / STATUS UPDATES /
FOLLOW-UPS / WEEKLY STATS / MATERIALS: [zip] TRACKER: [xlsx]
```

## Weekly performance review

On request (or Sundays): response, shortlist, interview, and ghosted rates; best platform and role title. Self-correction is stated openly — e.g. low response after 15+ applications means check ATS matching and targeting; same-day rejections point to ATS filters; post-interview rejections point to interview prep. No fabricated diagnosis when there is no clear pattern.

## PH employment basics

Mentioned when relevant: salaries are quoted monthly in PHP, often excluding legally required 13th-month pay; SSS, PhilHealth, and Pag-IBIG are standard; a 6-month probationary period before regularization is common; government roles follow CSC eligibility.

## Safety

Stop and warn if a recruiter asks for GCash/bank transfers, training or placement fees, refuses to name the company or give an office address, skips interviews while offering above-market pay, or provides no written offer. Applications are never submitted on your behalf without explicit per-employer confirmation.

## Ignored local files

The [`.gitignore`](.gitignore) ignores everything except `.gitignore`, `README.md`, and the skill. These stay on your machine only:

- `applications_*/` — per-job resume/cover letter folders
- `Kevin-Sapang_Applications_*.zip` and extracted copies — bundled application packages
- `job_tracker.csv` / `job_tracker.xlsx` — application tracker
- `.DS_Store` and other OS/editor files

## Quick start

```text
/ph-job-search help        # see capabilities
/ph-job-search search      # find PH-only matches + tailored materials
/ph-job-search status      # check tracked applications
/ph-job-search automate    # nightly search + morning report
```
