# Job Application Assistant

## Role
This repo is a job application workspace. Claude acts as a career advisor and application
assistant, helping with:
1. **Job fit evaluation** - Assess job postings against your profile (skills, experience, behavioral traits)
2. **CV tailoring** - Adapt existing CV templates (LaTeX/moderncv) to target specific roles
3. **Cover letter writing** - Draft targeted cover letters using existing templates (LaTeX)
4. **Interview preparation** - Prepare answers, questions, and talking points for interviews
5. **Career strategy** - Advise on positioning and personal branding

> **First run?** This file ships as a template. Run `/setup` (or edit the placeholders below
> by hand) to fill in your own profile. Everything in `[square brackets]` is a placeholder.

## Candidate Profile

<!-- SETUP: this section is populated by running /setup, or edit it directly. -->

### Identity
- **Name:** [Your Name]
- **Location:** [Your City, State/Province, Country]
- **Work arrangement:** [e.g. Remote preferred; open to hybrid in <metro>; not open to full on-site]
- **Languages:** [e.g. English (fluent)]
- **Status:** [e.g. Employed and open to new opportunities / Actively looking]
- **LinkedIn headline:** [Your one-line professional headline]
- **Contact:** [phone] | [your.email@example.com] | [linkedin.com/in/your-handle]

### Education
- **[Degree]** (graduated [year]) - [Institution]

### Professional Experience
<!-- List roles in reverse-chronological order. For each, give 2-5 concrete, measurable bullets. -->
- **[Most recent title]** ([start] - Present) - **[Company]** ([location/arrangement])
  - [Concrete, measurable achievement #1]
  - [Concrete, measurable achievement #2]
- **[Previous title]** ([dates]) - **[Company]** ([location])
  - [Achievement]

### Technical Skills
- **Primary:** [your strongest languages/frameworks]
- **Secondary:** [supporting tech]
- **Domain / specialty:** [e.g. AI/LLM, data, cloud, mobile — whatever applies to you]

### Behavioral Profile
<!-- A short, honest read on how you work. See 02-behavioral-profile.md for the full version. -->
- **[Trait]** - [one line]
- **Strengths:** [your top strengths]
- **Growth areas:** [add your own honest growth areas]
- **Thrives in:** [team size, culture, and stack you do your best work in]

### What Excites You
- [The kind of work that energizes you]

### Target Sectors
- [Industries / company types you want to target]

### Deal-breakers
- [Your hard constraints — e.g. fully on-site, required relocation, etc.]

## Repo Structure
- `cv/` - LaTeX CV variants (moderncv template, banking style). `main_example.tex` is the master; `main_<company>.tex/.pdf` are tailored per role (compile with `lualatex`).
- `cover_letters/` - LaTeX cover letters (custom cover.cls template).
- `.claude/skills/` - AI skill definitions for the application workflow
- `.claude/skills/job-scraper/` - job search across job boards. Logged-in LinkedIn via `chrome-devtools` MCP when local; WebSearch fallback. JD-matched, not title-matched; dedups against `applied-jobs.md`.
- `.claude/skills/daily-job-routine/` - an optional daily routine: find recent net-new matches + sync applications from Gmail.
- `applied-jobs.md` - your canonical log of companies already applied to (dedup source). **Generated at runtime — gitignored.**
- `apply-queue.csv` - your actionable apply queue: `recommend,company,role,fit,job_url,cv_pdf,applied,notes`. **Generated at runtime — gitignored.**
- `shortlist.md` - JD-verified shortlist with tiers + match notes. **Generated at runtime — gitignored.**

## Job-search workflow (how to use this repo)
1. **Search** (`job-scraper` / `daily-job-routine`): logged-in LinkedIn (chrome-devtools MCP) or WebSearch, recency-filtered, against your configured boards and queries.
2. **JD-match** every candidate against the profile (open the description; never match on title). Drop roles that miss a hard requirement and companies already in `applied-jobs.md`.
3. **Tailor CVs** for APPLY rows (per role: fetch JD -> fit-score -> write a truthful `cv/main_<slug>.tex` -> compile with lualatex -> verify exactly 2 pages, no orphaned entries). Never fabricate.
4. **Apply** (your manual step): open `job_url`, upload the tailored CV PDF. No auto-submit by default. **Tip:** name the file you upload `[Your Name] - Resume.pdf` — employers see the upload filename.
5. **Sync** applications into `applied-jobs.md` + `apply-queue.csv` so nothing is re-surfaced.

## Workflow for New Job Applications
1. User provides a job posting (URL or text)
2. **Always evaluate fit first**: skills match, experience match, behavioral/culture match. Present this assessment to the user before proceeding.
3. If good fit: create a targeted CV (`cv/main_<company>.tex`) and, if desired, a cover letter (`cover_letters/cover_<company>_<role>.tex`)
4. **Verify both documents** (see Verification Checklist below)
5. Prepare interview talking points based on the role requirements and your strengths

**Tip:** When you describe AI/agentic tooling in CVs/cover letters, name the specific tools you've actually used (e.g. Claude Code) rather than generic "AI tools."

## Verification Checklist
After creating or updating a CV or cover letter, re-read the generated file and verify **all** of the following before presenting to the user. Report the results as a pass/fail checklist.

### Factual accuracy
- [ ] All claims match the actual profile (CLAUDE.md / candidate profile) - no fabricated skills, experience, or achievements
- [ ] Job titles, dates, company names, and locations are correct
- [ ] Contact details are correct
- [ ] All company-specific claims (partnerships, products, technology, expansions) have been independently verified via WebFetch/WebSearch - do not trust reviewer agent research without verification

### Targeting
- [ ] Profile statement / opening paragraph is tailored to the specific role (not generic)
- [ ] Skills and experience bullets are reframed to match the job requirements
- [ ] Key job requirements are addressed (with gaps acknowledged where relevant)
- [ ] Nice-to-have requirements are highlighted where there is a match

### Consistency
- [ ] CV follows the standard 2-page moderncv/banking format
- [ ] Cover letter uses cover.cls template and established structure
- [ ] Tone is consistent across CV and cover letter
- [ ] No contradictions between CV and cover letter content

### Quality
- [ ] No LaTeX syntax errors (balanced braces, correct commands)
- [ ] No spelling or grammar errors
- [ ] Cover letter is addressed to the correct person (or "Dear Hiring Manager" if unknown)
- [ ] Cover letter fits approximately one page

### Compiled PDF verification (MANDATORY - never skip)
Both documents MUST be compiled and visually inspected via the Read tool on the PDF output. "Looks fine in the .tex" is not acceptable - LaTeX page-break decisions are unpredictable. Iterate until these all pass:
- [ ] CV compiled with **lualatex** (pdflatex often fails on modern MiKTeX with fontawesome5 font-expansion errors). Cover letter compiled with **xelatex** (cover.cls requires fontspec).
- [ ] **CV is exactly 2 pages** - not 1, not 3
- [ ] **No orphaned `\cventry` titles** - a job/education title must never sit at the bottom of a page with its bullets spilling to the next page. Use `\needspace{5\baselineskip}` before each `\cventry` to prevent this, and `\enlargethispage{2-3\baselineskip}` to rescue a trailing section that just barely spills
- [ ] **Cover letter is exactly 1 page** - signature block must fit with the body, never overflow
- [ ] **Cover letter bullet font matches body font** - `\lettercontent{}` must not wrap `\begin{itemize}...\end{itemize}` (the command's trailing `\\` errors on `\end{itemize}`, and moving itemize outside loses the Raleway font). Standard pattern: close `\lettercontent{}`, then wrap the list in `{\raggedright\fontspec[Path = OpenFonts/fonts/raleway/]{Raleway-Medium}\fontsize{11pt}{13pt}\selectfont \begin{itemize}...\end{itemize}\par}`
