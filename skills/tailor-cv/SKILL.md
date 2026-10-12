---
name: "tailor-cv"
description: "Evaluate and tailor the candidate's CV against a job description. Use when a job description is pasted or linked, or when asked to compare, score, or tailor the CV/resume for a role."
---

# Evaluate + Tailor CV to a Job Description

Source of truth lives in the Claude project that holds the job search. Read both docs fresh every run:
- `master-resume-template.md` (bullet bank, Professional Summary variants, skills lines, CV FORMAT RULES, honesty rules, checklist)
- `claude/target-titles-and-ats-keywords.md` (clusters, ranked titles, ATS keywords)

If the two docs conflict, the fact bank (master resume template) wins.

## Naming convention (applies to every run)
Everything this skill saves is named `<Company Name> - <Job Title>`:
- Company Name: the employer as written in the JD (not the recruiter/agency, unless the employer is undisclosed — then use the agency name).
- Job Title: the exact title from the JD, not the tailored CV title line. If the JD has no explicit title, name it from the role's scope and say so in the project doc.
- Keep normal capitalization and spaces. Remove characters that break paths: `/ \ : * ? " < > |` (replace `/` with `-`). Trim to ~80 characters if needed.
- Example: JD for "Senior Product Manager, Payments" at Kindred Group → `Kindred Group - Senior Product Manager, Payments`.
- If he runs the skill again for the same company + title, overwrite the existing project doc instead of creating a duplicate.

## Location policy
The candidate is willing to relocate (edit this policy to fit your situation). Location is NOT a factor anywhere in this skill:
- Do not lower the Match Score, flag a gap, raise a red flag, or change the verdict because of the job's city, country, or onsite/hybrid/remote arrangement.
- Do not count location or "must be based in X" as a must-have when tallying requirements.
- Exception (informational only, never scored): if the JD explicitly states a work-authorization or visa-sponsorship condition (e.g. "no sponsorship available", "must have the right to work in the UK"), mention it in one line under the Verdict as "Check: <quoted condition>". Do not guess his eligibility.
- Language requirements (e.g. "fluent German") are skills, not location — they still count normally.

## Input
- JD as pasted text, a URL (fetch it), or an attached file. If missing, ask for it and nothing else.
- If he says "just evaluate" or "score only", stop after Phase 1 (still save the project doc — see Save to project).

## Phase 0 — Already reviewed? (run before Phase 1, every time)
LinkedIn keeps showing old and reposted jobs, so a JD that looks new may already have been analysed or may be stale. Check before scoring anything.

1. **Identify the posting.** From the JD or URL, note: company, exact title, LinkedIn job ID (the number in `/jobs/view/<id>` or `currentJobId=<id>`) or other job board ID, and the posted date if shown ("Posted 3 weeks ago", "Reposted", a date). If he pasted text without a URL or date, ask once for the link or the posted date alongside your findings. Don't block on it.
2. **Search past reviews in these places:**
   - Project docs: `project_info` to list `applications/`, then `project_search` with the company name and title keywords. Read any doc that matches.
   - Project memory: `memory_list` with the prefix `/projects/<project-id>/applications/` and read any file whose name matches the company.
   - Tracker: `ArtifactData` `query` on the Job Search Pipeline artifact ({{TRACKER_ARTIFACT_URL}}), collection `roles`, `where company == <name>`. If the exact name finds nothing, `list` the collection and compare company names loosely (for example "Super" vs "Super Technologies", or the agency vs the employer).
   - Gmail: search `"<company>" (application OR candidatura OR applying)` to see if he already applied, or was already rejected.
   Past chat transcripts are not searchable from here. These four sources are the record, which is why every run must save to them (see "Save to project" and "Update the tracker").
3. **Decide what counts as a match.** It's a match if the job ID is the same, or the company is the same and the titles are the same or close (same role family and seniority, ignoring word order and punctuation). Several roles at the same company with different titles are not matches. List them as "other roles at this company" so he can see the pattern.
4. **If there's a match, stop before Phase 1** and report in 3-5 lines: date of the earlier review, Match Score, verdict, current status (Evaluated / CV built / Applied / Rejected / Closed / Skipped), whether a CV exists, and any Gmail outcome. Then ask one question with these choices: reuse the earlier analysis, re-run it (only worth it if the JD has changed or his bullet bank has grown since), or skip it. Do not re-score automatically.
5. **Check posting age even when there's no match.** If the posted date is more than 30 days ago, or the page says "Reposted", or the earlier record says the posting had closed, flag it in one line at the top of the reply ("This posting is ~6 weeks old; check it's still open before spending time on a CV") and continue with Phase 1. If the posting is visibly closed ("No longer accepting applications"), say so and stop.

## Phase 1 — Recruiter analysis
Act as a senior technical recruiter. Compare his background (the full bullet bank, not a tailored version) against the JD. Be blunt; do not inflate. Output exactly these sections:

1. **Match Score** — a realistic score out of 100 based strictly on hard requirements and transferable skills (excluding location, per the policy above). Show the basis in one line: how many must-haves are met / partially met / missing. Hard requirements he lacks (e.g., required years in a specific domain, a language, a degree, a certification) cap the score; transferable skills can't fully offset them.
2. **Top 5 Priorities** — the 5 requirements the employer weights most, inferred from wording, repetition, and placement (title, first bullets, "must"/"required" language). Quote the JD phrase for each. Skip location as a priority.
3. **Fit & Gaps** — for each core requirement: matching evidence from the bullet bank (name the specific bullet/achievement), or a flagged genuine gap. Mark each as Match / Partial / Gap. Never count design/assessment work (anything the fact bank marks as "designed" or "assessed") as implementation experience.
4. **Red Flags** — 3 concerns a hiring manager would spot in under 10 seconds (e.g., title mismatch, industry distance, missing hard requirement, seniority mismatch, language, employment gap). Never location. For each, one line on how the tailored CV or cover letter can pre-empt it, or state that it can't be fixed.
5. **Verdict** — Strong match / Stretch / Poor fit, with a straight one-to-two sentence recommendation on whether to apply. Add the work-authorization "Check:" line here only if the JD states one.

Also state the cluster tag(s) chosen for tailoring ([PM], [TECH], [OPS], [COMPLIANCE], [PO], [BA], [AI]).

## Gate
- **Strong match** → proceed to Phase 2 automatically.
- **Stretch** → ask once: "Build the tailored CV?" and wait.
- **Poor fit** → recommend not applying; build only if he explicitly says so.

## Phase 2 — Tailor
Follow the template's CV FORMAT RULES exactly. In order:
1. Header from template; title line mirrors the JD title only if it honestly matches the candidate's experience, otherwise the closest truthful title. If the job is located outside {{HOME_CITY}}, change the header location to `{{HOME_CITY}}, {{COUNTRY}} · Open to relocation` so a recruiter doesn't screen the candidate out on location. For [AI]/[TECH] roles, or when the JD mentions AI or prototyping, add the GitHub link from the template to the contact line and move the location to its own centered line below it.
2. **PROFESSIONAL SUMMARY** (use this exact heading, never "Summary"): pick ONE variant. The first sentence must follow: job title → years of experience → core functions held (from the list in the fact bank, ordered to fit the JD). Then rewrite the rest lightly to address the Top 5 Priorities in the JD's vocabulary. Keep it to about 4-5 lines on the page.
3. Skills: start from [ALL] + matching cluster lines; reorder and rephrase to mirror the JD's exact wording. Drop irrelevant items. For applications read by a person rather than an ATS (small teams, founder/hiring-manager screens), keep it to ~10-13 items instead of a keyword wall.
4. Experience — every role header is one line, separated by " | ": `Job Title | Company Name | Location | MM/YYYY – MM/YYYY`. Use this exact layout, most recent first:
   - The most recent role: one entry. Always include the role's headline bullet defined in the fact bank; add cluster-tagged bullets, leading with the one most relevant to the Top 5 Priorities.
   - Roles the fact bank marks as "combined" (several titles at one employer): ONE combined line (never stacked titles), then ONE bullet set of 3-5 from the condensed pool. No bullet may repeat another.
   - Earlier roles: 4-6 bullets each.
   Rephrase for JD vocabulary without changing the claim.
5. Key Achievements: pick 3-5 matching the cluster and Top 5 Priorities. Don't duplicate an Experience bullet unless the achievement adds a metric or context the bullet lacks; never repeat the same headline result (e.g. a single big metric) more than twice on the CV.
6. Include AI & Product Innovation only for [AI]/[TECH] roles or if the JD mentions AI. When included, end it with the template's personal-project sentence (with link).
7. Static sections unchanged.
8. Re-check the Phase 1 Red Flags against the tailored version: note which are now pre-empted and which remain.

## Hard rules (non-negotiable)
- Never invent metrics, tools, certifications, scope, or years of experience. Use only facts in the bullet bank.
- Anything the fact bank marks as design/assessment work is never "implemented", "led rollout", or "shipped".
- Use the official titles exactly as the fact bank records them, including any title rules it states (for example, which title to show for which kind of job).
- Company names exactly as the fact bank records them, including former names.
- Respect every scope note in the fact bank (for example "tested hands-on, no formal sign-off"; "designed, not live"; "executed by Engineering").
- Never attach technologies to a project that it doesn't use.
- Remove all tag markers, internal-context notes, and `[ADD METRIC ...]` / `[ADD NUMBERS ...]` placeholders from the output.
- Single column, no tables/icons/text boxes, standard fonts. Target 2 pages.
- Do not re-score the tailored CV — the Match Score reflects his real background, and rewording doesn't change it.

## Save to project (every run)
After Phase 1 — and again after Phase 2 if it runs — write a doc to the Claude project that holds the job search with the Projects tool (`project_write`):
- Path: `applications/<Company Name> - <Job Title>.md` (per the Naming convention above).
- Contents: date of the run, JD source (URL or "pasted"), job board ID and posted date if known (Phase 0 uses these to catch reposts), the full Phase 1 output (Match Score, Top 5 Priorities, Fit & Gaps, Red Flags, Verdict, cluster tags), and — if Phase 2 ran — the tailored CV filename, keyword coverage, remaining red flags, and open checklist items. Add a `Status:` line (Evaluated / CV built / Applied / Closed / Skipped) so the folder doubles as an application tracker.
- Re-running the same company + title overwrites this doc (read it first and keep any Status he updated by hand).
- Leave `present_to_user` unset; mention the saved doc name in one line.

## Update the tracker (every run)
After saving the project doc, upsert the role in the Job Search Pipeline tracker ({{TRACKER_ARTIFACT_URL}}) with `ArtifactData`, collection `roles`:
- Doc id: lowercase slug of company + title (letters, digits and hyphens). If Phase 0 found an existing row, update that row instead, pinning `if_version` to the version you read.
- Fields: `company`, `role`, `reviewed` (YYYY-MM-DD), `score`, `verdict`, `status` (`evaluated`, `cv_ready`, `skipped`; keep `applied` / `rejected` / `closed` / `in_process` if the row already has them), `applied`, `outcome`, `channel`, `notes` (one line: main gap, posting age if old, any "Check:" condition), `next`, `conversation` (false unless there was a call or interview), `updated`.
- Use `skipped` for a Poor fit he isn't pursuing, `cv_ready` once Phase 2 builds a CV, `closed` when the posting has closed.
- If the tracker write fails, say so in one line and carry on. The project doc is still the main record.

## Output
- Build the .docx using the docx skill. Filename: `{{CANDIDATE_FILE_PREFIX}}_CV_<Company Name> - <Job Title>.docx` (same Naming convention).
- Save it to `{{CV_OUTPUT_FOLDER}}` on the candidate's computer (request access if it isn't connected). After saving, verify the saved file actually contains the latest content (e.g. check its size or grep the docx XML for a phrase you just changed); if it doesn't, save it again with overwrite.
- After the file: keyword coverage (JD keywords included vs. missing), remaining red flags, and any template checklist item still needing his input.
- Do not print the full CV in chat.