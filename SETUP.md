# Setup Guide

Step-by-step instructions for getting the AI Job Search framework running.

## 1. Prerequisites

### Claude Code

Install Claude Code (Anthropic's CLI for Claude):

```bash
npm install -g @anthropic-ai/claude-code
```

You'll need an Anthropic API key or a Claude Pro/Team subscription. See the [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code) for details.

### Python

Python 3.10+ is required for the salary lookup tool. Check with:

```bash
python --version
```

### Job search (no install needed)

Job search runs through the `job-scraper` skill using Claude's built-in WebSearch/WebFetch against mainstream job boards. There is no CLI to install.

### LaTeX (for compiling CVs and cover letters)

Install a LaTeX distribution to compile the generated `.tex` files to PDF:

- **Windows:** [MiKTeX](https://miktex.org/download)
- **macOS:** [MacTeX](https://tug.org/mactex/)
- **Linux:** `sudo apt install texlive-full` or `sudo dnf install texlive-scheme-full`

The CV compiles with `lualatex` (pdflatex often fails on modern MiKTeX installs with `fontawesome5` font-expansion errors). The cover letter compiles with `xelatex` because `cover.cls` requires `fontspec` for its custom Lato/Raleway fonts.

## 2. Clone (or fork)

```bash
git clone <this-repo-url> ai-job-search
cd ai-job-search
# or, to keep your own copy under your account:
# gh repo fork <owner>/ai-job-search --clone
```

## 3. Job search

No setup required. The `job-scraper` skill searches mainstream job boards (hiring.cafe, LinkedIn, Indeed, Glassdoor, Wellfound, BuiltIn, We Work Remotely, and more) via WebSearch/WebFetch. Configure or refine queries any time with `/setup --section search` or by editing `.claude/skills/job-scraper/search-queries.md` — set your own roles, skills, and location.

## 4. Run the setup interview

Start Claude Code in the repository:

```bash
claude
```

Then run the onboarding:

```
/setup
```

Claude will offer two paths:

- **Path A (recommended):** Share your existing CV (mention the file with `@` or paste the text). Claude extracts your information and asks follow-up questions for anything missing.
- **Path B:** Answer structured interview questions section by section.

Both paths produce the same result: fully populated profile files.

### What gets populated

| File | Content |
|------|---------|
| `CLAUDE.md` | Your full candidate profile |
| `01-candidate-profile.md` | Structured education, experience, skills |
| `02-behavioral-profile.md` | Behavioral assessment |
| `04-job-evaluation.md` | Personalized skill match areas and career goals |
| `05-cv-templates.md` | Profile statement templates for your background |
| `07-interview-prep.md` | STAR examples from your experience |
| `cv/main_example.tex` | Your LaTeX CV with actual details |
| `search-queries.md` | Job search queries for `/scrape` |

### Re-running setup

You can update specific sections later:

```
/setup --section skills
/setup --section experience
/setup --section search
```

The `--section search` option is especially useful as your priorities evolve. It re-runs the search configuration interview and suggests role types you may not have considered based on your full profile.

## 5. Optional: Set up salary benchmarking

If you have salary data (from a union, salary survey, Glassdoor, or personal research):

1. **Option A:** Create `salary_data.json` manually in the repo root (see `tools/README_SALARY_TOOL.md` for the format)
2. **Option B:** Convert from Excel:
   ```bash
   pip install openpyxl
   python tools/convert_salary_excel.py path/to/salary-data.xlsx --source "My Salary Data 2025"
   ```

This creates `salary_data.json` which the `/apply` workflow uses for salary benchmarking. If you skip this step, salary lookup is simply omitted.

## 5b. Optional: browser automation to fill out applications

The core workflow (`/scrape`, `/apply`) needs only WebSearch. If you also want Claude to
**fill out application forms for you in a real browser** — the optional `auto-apply` skill,
plus logged-in LinkedIn scraping — set up the `chrome-devtools` MCP:

1. Install [Google Chrome](https://www.google.com/chrome/) and Node.js (for `npx`).
2. This repo ships a pre-configured `.mcp.json`. When you start Claude Code in the repo, it
   will ask whether to enable the `chrome-devtools` MCP server — approve it. (You can also
   manage it with `claude mcp list`.)
3. The first time the skill runs, the MCP opens a Chrome window. **Log into LinkedIn** there
   (and any ATS accounts you use); the skill reuses that logged-in session on later runs.
4. Trigger it with something like "apply to the next 3 jobs in the queue." By default the
   skill fills everything and uploads your resume but **stops before the final Submit** so you
   review and click Submit yourself.

> ⚠ **Browser automation of job sites is Terms-of-Service-sensitive.** LinkedIn and many ATSes
> restrict automated interaction and use CAPTCHA / anti-bot defenses. Use it with judgement,
> keep actions human-paced, and never let it fabricate answers. This is an opt-in convenience,
> not a bulk-apply bot.

## 6. Test the workflow

Find a job posting you're interested in, then:

```
/apply https://www.linkedin.com/jobs/view/1234567890
```

Or paste the job description directly:

```
/apply [paste job posting text here]
```

Claude will:
1. Evaluate the fit against your profile
2. Ask if you want to proceed
3. Draft a tailored CV and cover letter
4. Have a reviewer agent critique the drafts
5. Revise and present the final output

## 7. Compile your documents

After `/apply` creates the LaTeX files:

```bash
# Compile CV
cd cv && lualatex main_<company>.tex && cd ..

# Compile cover letter
cd cover_letters && xelatex cover_<company>_<role>.tex && cd ..
```

## Troubleshooting

### "salary_data.json not found"
This is expected if you haven't set up salary benchmarking. The `/apply` workflow skips this step automatically.

### Job search returns nothing
The `job-scraper` skill relies on WebSearch/WebFetch. Some boards (Indeed, Glassdoor) are Cloudflare-protected and may not fetch directly — rely on WebSearch snippets, or use a logged-in browser for those. Refine queries in `.claude/skills/job-scraper/search-queries.md`.

### LaTeX compilation errors
- CV: uses `lualatex` (pdflatex often fails on modern MiKTeX with `fontawesome5` font-expansion errors; lualatex handles the same sources cleanly)
- Cover letter: uses `xelatex` (for custom fonts in `OpenFonts/fonts/`)
- Make sure your LaTeX distribution includes the `moderncv` package

### Fonts not found in cover letter
The cover letter template expects fonts in `cover_letters/OpenFonts/fonts/`. Make sure this directory exists and contains the Lato and Raleway font files.
