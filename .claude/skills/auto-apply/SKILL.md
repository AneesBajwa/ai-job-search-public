# Auto-Apply

**name:** auto-apply
**description:** Apply to jobs from apply-queue.csv using your logged-in Chrome (chrome-devtools MCP). Detects LinkedIn Easy Apply vs external ATS, fills the form with your profile, uploads your resume, and (by default) stops before the final submit so you do the final click. Triggers on: apply to jobs, auto-apply, submit applications, apply to the next N.
**allowed-tools:** Task, Read, Bash, Glob, Grep

---

## ⚠ Before you use this
Browser automation of job applications is **sensitive**. LinkedIn and many ATSes have Terms
of Service that restrict automated interaction, and most have anti-bot / CAPTCHA defenses.
Use this with judgement, keep actions human-paced, and **never fabricate** anything. The
default below is **stop before the final submit** — you review and click Submit yourself.
Enabling auto-submit is your own decision; do it only where you're comfortable with the ToS.

## ⚡ Token rule (most important)
Browser page snapshots are **60K+ characters** — they blow up the main context fast. **NEVER drive the application forms in the orchestrator context.** Instead, **dispatch ONE subagent per job (or a small batch)** via the Task tool. The subagent loads the chrome-devtools MCP tools (`ToolSearch select:...`), does all the navigate/snapshot/fill/upload work, and returns a **one-paragraph status only**. The huge snapshots stay in the subagent's context.

The browser is a single shared Chrome instance, so run subagents **sequentially** (one job finishes before the next starts), not in parallel.

## Default policy (safe default — change it for yourself if you want)
- **Stop before the final Submit.** Fill + upload + answer required questions, then **stop and let the user click the final Submit / Apply.** (If you choose to enable auto-submit for yourself, do so knowingly and still STOP on: a CAPTCHA, a login/account-creation wall, a payment request, or any required field you cannot answer truthfully.)
- **GitHub:** use your GitHub URL from `01-candidate-profile.md` for any GitHub-URL field (leave blank if you have none).
- **Required free-text essays / "why us" / short-answers:** write them yourself (see *Writing essays* below), grounded only in the real profile.
- **EEO / voluntary self-ID:** these are **voluntary** — fill them per your own preference, or leave blank / "prefer not to answer." Never store demographic answers in this repo.

## Writing essays (humanized + truthful)
3-6 sentences, first person, that a real person would write — grounded ONLY in your actual experience from `CLAUDE.md` / `01-candidate-profile.md`. Tone = conversational-professional. **No em-dashes, no clichés** ("passionate about", "leverage", "hit the ground running"), no buzzword padding; open with a concrete specific; vary sentence length so it doesn't read AI-generated. For "why <company>", tie 1-2 real, verifiable facts about the company (from the JD/site) to your genuine motivation. **Never invent** skills, employers, metrics, or experience.

## Your profile (for filling forms — read it from your own files, never fabricate beyond it)
Pull these from `CLAUDE.md` and `.claude/skills/job-application-assistant/01-candidate-profile.md`:
- Name, Email, Phone, Location, LinkedIn URL, GitHub URL
- Full mailing address + postal/zip code (only when a form requires it)
- Current title / employer, years of experience, education
- **Work authorization:** answer **truthfully** for the country the role is in. If a role needs sponsorship/visa you do or don't qualify for, say so honestly. Where there's a free-text spot, add a one-line honest note about your status.
- **Salary:** use the posting's band (e.g. mid-upper); flag the number for the user to adjust.
- **Voluntary self-ID (EEO):** optional — answer per your own preference at submit time; don't hard-code it here.

## Resume to upload
Upload the tailored resume PDF for this role. **Tip:** the employer sees the upload filename, so name the file `[Your Name] - Resume.pdf` rather than `main_<company>.pdf`. The exact path is in `apply-queue.csv`'s `cv_pdf` column.

## Human-like interaction (anti-automation — apply on EVERY form; critical on Ashby / Lever / Gem / iCIMS / LinkedIn)
Bot detection flags input that is instant, simultaneous, and perfectly timed. Make the subagent's session look human:
- **Move before you click.** `hover` the target element first, *then* `click` it — don't click an element the cursor never visited. chrome-devtools `click`/`hover` already dispatch **trusted CDP mouse events** at the element (unlike JS `.click()`, which anti-bot scripts flag); hovering first adds the cursor-travel signal they expect.
- **Type, don't paste.** Use `type_text` (real keystrokes) for text fields, and **always** for essays / cover letters — never `fill`/`fill_form` a multi-sentence answer (an instant value-set is a bot tell). Split a long essay into 2-4 `type_text` calls (sentence chunks) so the keystroke stream isn't one uniform burst.
- **Pace yourself.** Pause a randomized interval between fields and before Submit: `evaluate_script(() => new Promise(r => setTimeout(r, 400 + Math.random()*1400)))` (~0.4-1.8s; use a longer ~2-4s "reading" pause before an essay box and before the final Submit).
- **One field at a time on sensitive ATSes** (Ashby, Lever, Gem, iCIMS, LinkedIn Easy Apply): hover → click → type → pause, per field. A single `fill_form` that sets everything at once is fine only on benign ATSes (Greenhouse, Appcast, BambooHR, Breezy, UltiPro).
- **Read-then-act order:** snapshot → hover → short pause → click/type. Let `click`/`hover` auto-scroll the element into view; don't jump the page.
- **Overall cadence:** ~20-60s for a simple form, longer with essays — don't machine-gun. If a security checkpoint or anti-bot warning appears, **STOP** (repeated instant retries confirm automation).

## Per-job procedure (give this to the subagent)
1. Open `https://www.linkedin.com/jobs/view/<id>`. Take a snapshot (it saves to a file because it's huge) and **grep that file** for `Easy Apply` vs `Apply on company website` + the external ATS URL (`linkedin.com/safety/go/?url=...` → decode the `url=` param).
2. **External ATS** (Lever / Greenhouse / Ashby / Workday / etc.): navigate to the ATS apply page. Snapshot (usually fits inline). **Upload the resume** (`upload_file` on the attach-resume element). `fill_form` (or field-by-field with hover→type→pause on anti-bot ATSes — see *Human-like interaction*): Full name, Email, Phone, Current location, Current company, LinkedIn URL; work-eligibility (answer truthfully for the role's country); salary (in-band, flag it). Leave demographic / EEO / pronouns blank unless you choose to fill them. **Do NOT click Submit** — leave it for the user (default).
3. **LinkedIn Easy Apply**: FIRST check the job page isn't already showing **"Application status: Application submitted — N days ago"** (if so, it's already applied → skip it and add it to `applied-jobs.md`). Otherwise click Easy Apply; the modal pre-fills your LinkedIn-stored resume + contact. Answer screening questions truthfully from the profile. The modal may be **localized** (e.g. French for some employers) → answer truthfully. Advance through steps to the final review, **but stop before the final "Submit application"** (leave the one-click submit for the user) unless you've explicitly chosen to auto-submit.
4. Return a one-paragraph status: company, type (Easy Apply / external+ATS name), what was filled/uploaded, status (**ready for your submit** / blocked + reason).

## Guardrails
- **Never fabricate** skills, experience, or authorization. Answer work-authorization truthfully.
- **Stop and flag** on: account-creation/login wall, email verification, CAPTCHA, or any required question that can't be answered truthfully.
- Demographic/EEO fields are voluntary — your choice; nothing demographic is stored in this repo.
- LinkedIn automation is ToS-sensitive — keep actions human-paced; stop on any security checkpoint.
- After a batch, update `apply-queue.csv` (`applied` column → `ready YYYY-MM-DD` or `submitted YYYY-MM-DD`).

## ATS quick reference (KEEP UPDATING as new fields/ATSes appear)
- **Lever** (`jobs.lever.co/...`): single page, no account. Full name / Email / Phone / Current location / Current company / LinkedIn URL + Resume attach + work-eligibility radios + **salary expectations** (required free-text; use posting band). Pronouns/EEO optional.
- **Ashby** (`jobs.ashbyhq.com`, also embedded on company career pages): no account. Name / Email / Phone / LinkedIn / Resume + a free-text **"Why <company>?" / motivation** box (2-3 tailored sentences from the profile) + a **privacy "Continue"** acknowledgment. Work-auth often scoped "authorized to work in <country>?" → answer truthfully; sponsorship → truthfully. Diversity survey optional.
- **Greenhouse** (`boards.greenhouse.io` / `job-boards.greenhouse.io`, often embedded on the company site): no account. First / Last / Email / Phone / Location / Resume / LinkedIn + custom: "how did you hear about us", "worked here before", work-authorization, sponsorship, an **acknowledgment checkbox**. If "how did you hear about us" has no LinkedIn option, use **Job Board**; the phone-country selector often defaults to **United States** → set your country. Country/phone/screening dropdowns are **react-select autocomplete** — type to filter, then click the surfaced listbox option. ⚠ Some Greenhouse forms require a **GitHub URL** and/or **free-text essay questions** → if you have no GitHub or need to write essays yourself, mark **partial** and flag for the user; never fabricate.
- **Appcast** (`apply.appcast.io`, reached via a `click.appcast.io` redirect): no account. 3-step wizard — (1) contact incl. **Country select** (State/Province auto-derives once Country is set), (2) Resume upload button, (3) employer questions as **styled button-radios** (click the label, not a native radio) + required **salary** free-text. reCAPTCHA present but doesn't block filling.
- **Breezy HR** (`*.breezy.hr`): single page, no account. The "Upload Resume" link **auto-parses the PDF** into work history/education. Required free-text **Experience Summary + Cover Letter** (draft from profile, flag for the user to personalize), required **Desired Salary** with a currency/period unit combobox, full **Address** in one field.
- **BambooHR** (`*.bamboohr.com/careers/...`): no account. First / Last / Email / Phone / **full address** (street / city / Province-State / **Country — often DEFAULTS TO US, change it** / postal) / LinkedIn / **Desired Pay** (free-text; posting band) / **Date Available (REQUIRED — fill MM/DD/YYYY; it silently blocks submit *before* the reCAPTCHA fires)**.
- **Workday** (`*.myworkdayjobs.com`): **requires account creation + email verify** → create an account (see Login-required ATS), verify via email, then continue.
- **Common gotchas:** country/province dropdowns often default to US — set your country. Names often split First/Last. Salary/"desired pay" sometimes required — use posting band, flag the number. EEO/demographic/pronouns are voluntary.

## Browser-automation gotchas (learned — read before driving Ashby/Greenhouse)
- **Ashby location/country/citizenship/level/"how did you hear" selects** are autocomplete comboboxes whose options are NOT a11y nodes — `fill` alone leaves the listbox empty. Pattern: `click` the field → `type_text` (fires the keystroke listener) → the `[role="listbox"]` options are `div`s; click the surfaced option. City fields use Google Places (type → `wait_for` the option text → click).
- **Ashby button-style Yes/No "radios"**: selected state shows as CSS class `_active_…`, NOT `aria-checked` — verify selection via className.
- **Ashby/Greenhouse embedded on a company domain** (e.g. `company.com/careers?ashby_jid=<id>`) render the form in a **cross-origin iframe** → `evaluate_script` is blocked there. For **Ashby**, navigate instead to the standalone `https://jobs.ashbyhq.com/<org-slug>/<job-id>/application`; the org-slug varies — grep the careers-page snapshot's iframe `RootWebArea url=` to get it. For **Greenhouse** iframes, drive via snapshot+click only (its react-select exposes real `[role=listbox]`/`option` nodes — click those uids).
- **Required free-text essays** ("why us", "an integration project", "AI-usage example", working-hours): **write them yourself** (humanized, truthful — see *Writing essays*). GitHub URL fields → use your GitHub URL. Neither blocks anymore.
- **CAPTCHA = hard blocker** (fill everything + attach resume, then flag for the user to solve + submit): **Lever** has an *invisible hCaptcha* on submit (button silently no-ops); **BambooHR** has a visible *reCAPTCHA "I'm not a robot"* checkbox.
- **Greenhouse email-verification:** submit is gated behind a security code emailed to you — fetch via the Gmail connector, enter it, then submit. A stale Greenhouse tab loses its resume binding on reload → re-attach on a fresh form. Company-embedded Greenhouse forces its iframe → drive the iframe.
- **Ashby React validation:** text/essay/LinkedIn fields set via `fill_form` don't register ("missing required field") → **clear + type via keyboard**. Ashby may throw a **"possible spam" anti-bot rejection** on automated submit — reload + fresh re-fill usually clears it, but Ashby's OWN careers site can reject persistently → flag for manual submit. (Ashby enforces a **3-applications / 60-day** limit.)
- **Appcast:** after submit, a **"Save your profile?" upsell modal** appears → click "No, thanks"; page then navigates to `/applied` (= success).
- **Greenhouse react-select decline option** is labeled **"I don't wish to answer"** (not "prefer not to answer").
- **Gem** (`jobs.gem.com/<org>`): no account. First/Last/Email/**LinkedIn (full `https://www.linkedin.com/in/...` required — bare `linkedin.com` rejected)**/Location + resume + essays + react-select. Use **"Apply without saving"**. Resume `<input>` is hidden & not in the snapshot — make it visible **in place** (inline CSS; do NOT reparent to body or React drops the binding), snapshot for its uid, then `upload_file`. **Submit = visible hCaptcha image challenge → hard blocker.**
- **iCIMS** (`*.icims.com`): opens with a GDPR consent gate (email + privacy-preference dropdown + "I accept" + Next) in an iCIMS iframe; **clicking Next fires a visible hCaptcha BEFORE the resume/name form loads → hard blocker.**
- **CAPTCHA is common (~1/3 of ATSes):** Lever (invisible hCaptcha), BambooHR (reCAPTCHA checkbox), Gem + iCIMS (visible hCaptcha). When hit: fill everything possible + attach resume + write essays, set `applied` = `blocked: captcha`, and flag for the user to solve + submit. Don't burn time trying to bypass it.
- **LinkedIn Easy Apply screening dropdowns** are real `<select>`s but `fill`/`click` on the option uids times out → set via `evaluate_script` (native value setter + dispatch input/change events), matching each select by its option-text signature. For selects **inside the cross-origin apply iframe**, pass the option's snapshot **uid as an `args` arg** to `evaluate_script` to resolve it across the frame.
- **LinkedIn Easy Apply city autocomplete**: plain type leaves "Please enter a valid answer" → click → Ctrl+A → Backspace → type → ArrowDown → Enter to commit the full "City, Province, Country".
- **Ashby checkbox-group "select all that apply"**: JS `.click()` toggles the DOM box but not Ashby's React state → use a real chrome-devtools `click` on each checkbox.
- **Ashby multi-word org-slug**: standalone `jobs.ashbyhq.com/<slug>` 404s when the slug has spaces — the real slug is URL-encoded; grep the embedded careers iframe's `RootWebArea url=...?embed=js` to recover it.

## Login-required ATS — account creation
If an ATS requires an account to apply (and you choose to create one):
- Use your email.
- Password: **generate a fresh strong random one per site** — `openssl rand -base64 18` via Bash. Never reuse across sites.
- **Record the credentials** (site, URL, email, password, date) in a **`credentials.md`** file at the repo root. This file is **gitignored — never commit or push it.**
- If **email verification** is required, use the **Gmail connector** (`mcp__claude_ai_Gmail__search_threads` / `get_thread`) to read the verification email/link and complete it.
- Then continue the normal flow (fill + upload + stop before final submit).
