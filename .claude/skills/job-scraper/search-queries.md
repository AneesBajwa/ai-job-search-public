# Search Queries for Job Scraper

<!-- Template config. Run `/setup --section search` to tailor this to your roles, skills,
     and location. Replace the [bracketed] terms with your own. -->

## Search Sites

Aggregators (widest reach — search these first):
- **hiring.cafe** - large aggregator across company ATS boards; strong for remote + startup roles
- **linkedin.com/jobs** - largest job board; filter by location / Remote
- **indeed.com** (or `indeed.ca`, etc. for your country) - large aggregator (Cloudflare-protected for WebFetch; use WebSearch snippets or the logged-in browser)
- **glassdoor.com** - listings + company reviews
- **ziprecruiter.com** - aggregator with strong North American coverage

Startup / tech boards:
- **wellfound.com** (formerly AngelList) - startup roles, remote-friendly
- **workatastartup.com** - Y Combinator startups
- **builtin.com** - tech & startup roles by metro
- **otta.com / welcometothejungle.com** - curated tech roles

Remote-focused boards:
- **weworkremotely.com**, **remoteok.com**, **remote.co** - remote-first listings

Government / national boards (add the one for your country):
- e.g. **jobbank.gc.ca** (Canada), **usajobs.gov** (US federal), etc.

Secondary (company career pages via Google):
- Direct Google `site:` searches for target companies

## Query Categories

Queries are grouped by priority. Combine with location terms (`remote`, your city, your
region/state) where the site supports it. **Adjust the skill keywords below to your own stack.**

### Priority 1: [Your primary role — e.g. Senior Full-Stack]

```
site:linkedin.com/jobs "[primary role]" ([skill A] OR [skill B]) remote [country]
site:linkedin.com/jobs "[role]" [skill] remote [country]
site:wellfound.com "[role]" [skill] remote
site:builtin.com "[role]" senior
```

### Priority 2: [Your specialty — e.g. AI / LLM, Data, Platform]

```
site:linkedin.com/jobs ("[specialty term A]" OR "[specialty term B]") remote [country]
site:linkedin.com/jobs ([keyword] OR [keyword]) [skill] remote
site:wellfound.com ("[specialty]") remote
```

### Priority 3: [Adjacent / specialist roles]

```
site:linkedin.com/jobs "[adjacent role]" [skill] remote [country]
site:builtin.com [role] [skill]
```

### Priority 4: Broader — Staff / Tech Lead / etc.

```
site:linkedin.com/jobs ("staff engineer" OR "tech lead") ([skill] OR [skill]) remote [country]
site:linkedin.com/jobs "software engineer" ([skill] OR [skill]) startup remote [country]
site:wellfound.com "software engineer" senior remote
```

### Aggregator sweeps (run for maximum reach)

```
site:hiring.cafe ([skill] OR [skill] OR "[role]") remote
site:weworkremotely.com ([skill] OR [role])
site:remoteok.com ([skill] OR [skill]) senior
site:wellfound.com ([skill] OR "[role]") remote
workatastartup.com ([role]) remote
```

## Location Filter

Evaluate each result against your constraints (set these to your own situation):
- **Remote** - usually ideal
- **Hybrid in [your metro]** - [accept / reject per your preference]
- **Fully on-site** - [accept / reject]
- For cross-border roles, note any **work-authorization / visa-sponsorship** requirement and answer it truthfully in the application.

**On LinkedIn:** use the Work Type filter — Remote (`f_WT=2`), Hybrid (`f_WT=3`), On-site (`f_WT=1`) — to match your preference.

**Company-stage preference:** [e.g. prioritize startups and scale-ups]. Note stage when presenting matches.

## Date Filter

Only include jobs **posted within the last 7 days** (or with an application deadline that
has not yet passed). On LinkedIn/Indeed use the "Past week" date filter (`f_TPR=r604800`).

⚠ **WebSearch is unreliable for recency** — it indexes stale JDs, so a `site:` hit is
routinely weeks or months old. For a hard ≤7-day requirement, **prefer the logged-in browser
with `f_TPR`**, or verify each WebSearch hit's TRUE date via the ATS API before keeping it:
Greenhouse `https://boards-api.greenhouse.io/v1/boards/<org>/jobs/<id>` → `first_published`;
Lever `https://api.lever.co/v0/postings/<org>/<id>` → `createdAt` (epoch-ms). **Drop anything
older than 7 days**; only flag "date unknown" if no source resolves a date.

## Adapting Queries

If the user specifies a focus area (e.g. `/scrape AI`), select queries from the matching
category and generate 2-3 custom queries for that focus, always appending a remote/location term.
