# Job Scraper

**name:** job-scraper
**description:** Searches popular tech job boards (hiring.cafe, LinkedIn, Indeed, Glassdoor, Wellfound, BuiltIn, We Work Remotely, and more) for new positions matching your profile. Deduplicates across runs. Triggers on: job scrape, find jobs, search jobs, new jobs, job search, scrape jobs, /scrape
**allowed-tools:** Read, Write, Edit, Glob, Grep, WebFetch, WebSearch, Agent, AskUserQuestion

---

## How It Works

This skill searches multiple tech job boards using targeted queries based on your profile, deduplicates against previously seen jobs and the application tracker, and presents new matches with a quick fit assessment.

## Invocation

The user triggers this skill by saying things like:
- "Find new jobs"
- "Scrape for jobs"
- "Any new positions?"
- "/scrape"

Optional arguments:
- A focus area, e.g. "/scrape data science" or "/scrape frontend"
- "broad" to run all search categories, e.g. "/scrape broad"

---

## Execution Steps

### Step 0: Load State

1. Read `job_scraper/seen_jobs.json` (create if missing - start with `{"seen": {}}`)
2. Read `applied-jobs.md` (repo root) — the canonical log of companies/roles already applied to. **Exclude every company listed there from results.**
3. Read `job_search_tracker.csv` (if present) to extract any additional already-applied companies+roles
4. Read `search-queries.md` (this directory) for the search strategy

### Step 1: Search

**Two methods (pick what's available):**
- **Logged-in LinkedIn (preferred, local runs):** if the `chrome-devtools` MCP is connected with Chrome logged into LinkedIn, drive LinkedIn Jobs directly — use the Work Type filter that matches your preference (`f_WT=2` Remote, `f_WT=3` Hybrid, `f_WT=1` On-site), set `location` to your target region, relevance sort, recency `f_TPR=r86400` (24h) / `r604800` (7d) / `r2592000` (30d). LinkedIn virtualizes its result list (~11 rows in the DOM at once), so extract by **accumulating job cards during incremental scroll** (grab → scroll ~90% viewport → wait → grab) to capture the full ~25/page. Pull full JDs from the public endpoint `https://www.linkedin.com/jobs-guest/jobs/api/jobPosting/<id>`.
- **WebSearch (fallback / remote-cron runs where no browser/login is available):** run `site:` queries from `search-queries.md`.

Run **WebSearch** queries from `search-queries.md`. By default, run the top 3 priority categories. If the user said "broad", run all categories.

If the user specified a focus area (e.g. "data science"), prioritize queries from that category.

For each search:
- Use `WebSearch` with site-specific queries across hiring.cafe, linkedin.com/jobs, indeed, glassdoor, wellfound.com, builtin.com, weworkremotely.com, remoteok.com (see `search-queries.md` for the full list and queries)
- Target your configured geographic area
- Look for postings from the last 7 days (no more than 1 week old)

### Step 2: Fetch & Parse

For each promising result from Step 1:
- Use `WebFetch` to retrieve the job posting page
- Extract: **job title**, **company**, **location**, **posting date** (or "recent"), **URL**, **key requirements** (brief), **application deadline** (if listed)
- Skip if the URL or company+title combo already exists in `seen_jobs.json`
- **Skip if the company appears in `applied-jobs.md`** (already applied — never re-surface)
- Skip if the company+role already appears in `job_search_tracker.csv`

### Step 3: Quick Fit Assessment

For each new job, do a rapid fit check (NOT the full evaluation from `04-job-evaluation.md` - just a quick signal):

- **High match**: Role directly involves your core skills
- **Medium match**: Role is adjacent to your experience
- **Low match**: Role requires significant skills you lack

### Step 4: Deduplicate & Store

1. Add ALL fetched jobs (new and skipped) to `seen_jobs.json` with structure:
```json
{
  "seen": {
    "<url_or_company_title_key>": {
      "title": "...",
      "company": "...",
      "url": "...",
      "first_seen": "YYYY-MM-DD",
      "fit": "high/medium/low",
      "status": "new/skipped/evaluated"
    }
  }
}
```
2. Only present jobs NOT already in the seen list or tracker.

### Step 5: Present Results

Present new jobs in a table sorted by fit (high first):

```
## New Job Matches - YYYY-MM-DD

Found X new positions (Y high, Z medium, W low match).

| # | Fit | Title | Company | Location | Deadline | URL |
|---|-----|-------|---------|----------|----------|-----|
| 1 | High | ... | ... | ... | ... | [Link](...) |

### High-Match Highlights
For each high-match job, add 2-3 bullet points:
- Why it matches your profile
- Key requirements to check
- Any red flags
```

After presenting, ask:
> "Want me to evaluate any of these in detail? Just give me the number(s)."

If the user picks a number, invoke the **job-application-assistant** skill workflow (fit evaluation first, then CV + cover letter if approved).

### Step 6: Update Tracker (Optional)

If the user decides to apply to any job, add a row to `job_search_tracker.csv`.

---

## Important Rules

1. **Never fabricate job postings.** Only present jobs found via actual WebSearch/WebFetch results.
2. **Respect deduplication.** Always check `applied-jobs.md` (already-applied companies — hard exclude), `seen_jobs.json`, AND `job_search_tracker.csv` before presenting. Never present a company already in `applied-jobs.md`.
3. **Work arrangement.** Honor the user's preference from their profile (remote / hybrid / on-site). For cross-border roles, note any visa/sponsorship requirement so it can be answered truthfully.
4. **Match the JD, not the title.** Before presenting any role, open it and read the actual description; confirm it maps to the user's stack (from `01-candidate-profile.md`). Drop title-only matches and roles with a mandatory requirement the user lacks (e.g. a primary language/framework they don't use, a required spoken language).
5. **Drop staffing/aggregator noise.** Never present staffing-agency reposts, AI data-labeling gigs, or aggregator relistings.
6. **Only open positions.** Skip postings with expired deadlines or those marked as closed.
7. **Be efficient with WebFetch.** Don't fetch every search result - use titles and snippets to pre-filter before fetching.
8. **Parallel searches.** Use the Agent tool or parallel WebSearch calls to speed up the search phase.
