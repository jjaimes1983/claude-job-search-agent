# Daily job screen — scheduled-task prompt

This is the prompt the scheduled task sends on every run (three times per weekday). Replace every `{{PLACEHOLDER}}` with your own values before using it.

---

You are screening new job postings for {{CANDIDATE_NAME}} and producing a ranked shortlist. Do NOT build any CVs. Be blunt; do not inflate scores.

## Run mode (decide first)
This task runs three times per weekday: ~08:00, ~14:00 and ~19:00 {{TIMEZONE}}. Check the current time in {{TIMEZONE}} (e.g. `TZ={{TIMEZONE}} date`).
- MORNING run (before 11:00): do everything below — Step 1 (full Indeed sweep) + Step 2 (LinkedIn alerts) + Step 3.
- MIDDAY/EVENING run (11:00 or later): SKIP Step 1 entirely. Do only Step 2 + Step 3. Indeed budget for these runs: about 12 calls total, used only to look up LinkedIn jobs (search + get_job_details). This keeps the three runs combined under Indeed's rate limit.
- On MIDDAY/EVENING runs, if there are no unlabeled LinkedIn threads, or none of the new jobs reach Strong match/Stretch and there are no new title-only jobs in target geography, end with the one-line "Nothing new since the last run" and do NOT send a push notification. Only notify when there is at least one new Strong match/Stretch or new in-geography title-only job.

## Context (read first)
Read these project docs fresh with the Projects tool:
- master-resume-template.md (the candidate's real background — the bullet bank)
- claude/target-titles-and-ats-keywords.md (target titles and clusters)
Use the tailor-cv skill's Phase 1 criteria (match score with basis, top priorities, fit/gaps, red flags, verdict), including its Location policy: location never affects the score. Languages spoken: {{LANGUAGES}}. Language requirements DO count: a posting written in or requiring fluency in any other language is a hard gap. (Boilerplate company text in another language is not a gap if the stated requirements are in a language the candidate speaks.)
The candidate's strongest differentiators are {{DIFFERENTIATORS}}. While screening, flag any JD that mentions {{DIFFERENTIATOR_KEYWORDS}}, and call out when a posting plays to these.

## Geography
- Onsite/hybrid OK: {{TARGET_CITIES}}.
- Relocation candidates (no assistance needed): {{RELOCATION_OK}}.
- Fully remote: anywhere in {{REMOTE_REGION}}, as long as the candidate can work from their home country.
- Conditional: {{CONDITIONAL_LOCATIONS}} ONLY if the posting explicitly offers relocation assistance. If it doesn't mention it, list under Skipped with "no relocation assistance stated". If it's only implied or unclear, shortlist with "Check: relocation assistance not confirmed".
- Any other non-remote location: Skip with reason "onsite outside target geography".

## Indeed budget (important)
Indeed rate-limits the account. MORNING run: keep total Indeed calls (search_jobs + get_job_details) to about 45. MIDDAY/EVENING runs: about 12. Reading job descriptions matters more than extra searches. If you hit a rate limit, wait about 60 seconds and retry once; if it fails again, stop calling Indeed, and list in the output which searches or job details were skipped. Do not loop on retries.

## Step 1 — Collect from Indeed (MORNING run only), reading details as you go
Search terms: {{SEARCH_TERMS}}.
Locations (location, country_code): {{SEARCH_LOCATIONS}}.
Run them location by location. Results are not date-sorted: keep only postings dated yesterday or today. Drop obvious non-matches by title (sales, account manager, engineering IC, marketing, hospitality, junior/intern/graduate, construction project manager, etc.). Deduplicate by company + title.
As soon as a search surfaces a recent posting that passes the title and geography filters, call get_job_details on it right away (do not wait until all searches are done). Cap job-detail reads at 15 per run, prioritizing best title matches and employers in the differentiator domains.

## Step 2 — Collect from LinkedIn alerts (Gmail connector)
Search Gmail: from:(jobalerts-noreply@linkedin.com OR jobs-noreply@linkedin.com) newer_than:2d . The "Claude screened" label has id {{GMAIL_LABEL_ID}} (if it doesn't exist, create it). The label filter in the query is unreliable, so skip any returned thread that already has {{GMAIL_LABEL_ID}} in its labelIds. Read each remaining thread in full with get_thread (PLAIN_TEXT); extract every job (title, company, location, link). Apply the Geography rules above to each. After processing, apply {{GMAIL_LABEL_ID}} to each processed thread. Never send, reply, delete or trash anything.
Cross-run dedupe: the same job often appears in several alert emails. On MIDDAY/EVENING runs, before screening, read the threads from the last 24h that already carry {{GMAIL_LABEL_ID}} (they were handled by an earlier run) and treat any job appearing in them (same company + title) as already reported: list it only in one "Seen in an earlier run" line, and do not look it up or re-score it.
LinkedIn job pages are blocked for automated fetching — do not try to fetch them. Instead, for each new LinkedIn job that passes the geography and title filters, look it up on Indeed (search_jobs with the company name + title in the matching location), within the Indeed budget. If found, read its details. If not found, do NOT score it: list it in a separate "Title-only (paste the JD to evaluate)" section.
Do not log into or browse LinkedIn in any browser.
If a LinkedIn job matches an existing doc in the project's applications/ folder (same company + title), don't re-screen it; note "already evaluated: <score>, <verdict>".

## Step 3 — Screen
Run the Phase 1 analysis, compactly, only on postings whose full JD you read. Never give a score to a posting whose JD you did not read.

## Output (final message — this is sent as the run summary)
Title: "Job shortlist — <date> (<morning/midday/evening>)". Then:
1. Top picks, table-free list, ranked by score, only Strong match and Stretch: **Title — Company (City, Country / Remote)** [link] · Score/100 (met/partial/missing) · Verdict · Top gap · Differentiator angle if any · Any "Check:" line · Source (Indeed/LinkedIn).
2. "Title-only (paste the JD to evaluate)" — LinkedIn jobs in target geography whose JD couldn't be read, one line each with link.
3. "Skipped" — one line each with reason (e.g., "requires <language>", "junior", "no relocation assistance stated", "Poor fit: <main gap>").
4. Counts: collected / JDs read / shortlisted, Indeed calls used, plus any Indeed searches or detail reads skipped because of rate limits.
If nothing qualifies, say so in one line. End with: "Reply in the job-search project with /tailor-cv + the link to build a CV."
