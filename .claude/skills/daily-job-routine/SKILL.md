# Daily Job Routine

**name:** daily-job-routine
**description:** An optional daily job-search routine. Finds brand-new (≤1 day old) postings that genuinely match your profile, dedupes against already-applied jobs, appends new matches to apply-queue.csv, and syncs Gmail to mark/record jobs you have applied to. Triggers on: daily job routine, daily job search, scheduled job search, run my job routine.
**allowed-tools:** Read, Write, Edit, Glob, Grep, WebFetch, WebSearch, Bash, AskUserQuestion

---

## Purpose
Run once per day. Two jobs:
1. **Discover** — surface only postings from the **last 24 hours** that truly match your stack and are net-new (not already applied).
2. **Sync** — check Gmail for newly-submitted applications and update the tracking files so the discovery half never re-surfaces them.

Keep it tight and honest: quality over volume, JD-matched not title-matched, no fabrication.

---

## Files this routine reads/writes (repo root unless noted)
- `applied-jobs.md` — canonical log of companies already applied to (dedup source + sink). Generated at runtime; gitignored.
- `apply-queue.csv` — actionable queue (`recommend,company,role,fit,job_url,cv_pdf,applied,notes`). Generated at runtime; gitignored.
- `.claude/skills/job-scraper/search-queries.md` — boards + your role queries + filters
- `CLAUDE.md` / `.claude/skills/job-application-assistant/01-candidate-profile.md` — profile for fit-matching

## Candidate constraints (hard rules — set these from your own profile)
- **Work arrangement:** [your rule — e.g. remote preferred; hybrid in <metro> OK; on-site reject]. For cross-border roles, note any visa/sponsorship requirement and answer truthfully.
- Net-new only: **exclude every company listed in `applied-jobs.md`.**
- [Language / other hard requirements, e.g. "English-only roles".]
- [Company-stage / salary-floor preferences, if any.]

---

## Step 0 — Load state
Read `applied-jobs.md` (build the applied-company set), `apply-queue.csv` (existing queue), `search-queries.md`, and the profile. Note today's date.

## Step 1 — Find postings from the last 24 hours
Pick the mode based on what's available this run:

**Mode A — logged-in browser (local runs, preferred):** if the `chrome-devtools` MCP is connected and Chrome is logged into LinkedIn, search LinkedIn Jobs with **`f_TPR=r86400`** (past 24h) and the Work Type filter that matches your preference (`f_WT=2` Remote, `f_WT=3` Hybrid, `f_WT=1` On-site). Set `location` to your target country/region, relevance sort, across your role queries. Use the accumulate-while-scrolling extractor (LinkedIn virtualizes the list to ~11 rows; grab cards during incremental scroll to capture ~25/query). See `job-scraper/SKILL.md` for the extractor pattern.

**Mode B — no browser (remote/headless cron runs):** the `chrome-devtools` MCP and logged-in session are usually **not** available in a scheduled remote agent. Fall back to `WebSearch` with the `site:` queries in `search-queries.md`, restricting to the last 24h (prefer results dated today/yesterday), then `WebFetch` promising results. Note in the report that this run used WebSearch fallback.

## Step 2 — JD-match (MANDATORY — do NOT match on title)
For each candidate posting, fetch the full description (`WebFetch` on `https://www.linkedin.com/jobs-guest/jobs/api/jobPosting/<id>`, the public endpoint) and check it against your stack (from `01-candidate-profile.md`).
Drop a candidate if any of these are true:
- It only matched on a job-title keyword; the body doesn't map to your skills.
- The work arrangement violates your hard rule (e.g. on-site when you require remote).
- A **hard requirement** you lack is mandatory (e.g. a primary language/framework you don't use, a required spoken language, a years-of-experience floor above yours).
- The company is in `applied-jobs.md`.

## Step 3 — Score + append new matches
For survivors, assign a quick fit (0-100) and `recommend` (APPLY ≥68 / REVIEW 55-67 / SKIP <55 or caveats). Append each net-new match as a row to `apply-queue.csv` (`cv_pdf` blank — CVs are drafted on demand via the CV-tailoring workflow, not in this routine; `applied` blank). Do not duplicate rows already in the queue.

## Step 4 — Gmail application sync
If the Gmail connector is available this run (it may be **absent in headless/remote runs** — if so, skip this step and say so):
- Run the application-confirmation search over the window since the last run (default `newer_than:2d`):
  `({from:greenhouse-mail.io from:hire.lever.co from:ashbyhq.com from:myworkday.com from:bamboohr.com from:ats.rippling.com from:candidates.workablemail.com from:breezy-mail.com from:applytojob.com from:gem.com from:smartrecruiters.com from:icims.com from:jobvite.com} OR subject:("thank you for applying" OR "received your application" OR "thanks for applying" OR "your application to" OR "thank you for your application")) newer_than:2d -in:sent`
- For each new confirmation: extract company, role, date, status (applied/rejected/interview). Add to `applied-jobs.md` (dedup by company, case-insensitive). If that company is in `apply-queue.csv`, set its `applied` column to `yes (YYYY-MM-DD)`.

## Step 5 — Report
Summarize: `N` new ≤24h matches added (list company/role/fit/link), `M` applications synced from Gmail (and any rejections), and which mode/tools were available. If nothing new matched, say so plainly — that is a valid and common result for a 1-day window.

## Notes
- This routine **does not draft CVs or submit applications** — discovery + tracking only. CV tailoring is a separate on-demand step (over `apply-queue.csv` APPLY rows); submitting is always your manual step.
- A 24h window often yields 0-3 real matches. Do not pad with staffing-agency reposts, AI data-labeling gigs, or aggregator relistings.
