# Claude job-search agent

A personal AI agent that screens job postings for me three times every weekday, scores each role against a fact bank of what I have actually done, and only bothers me when something is worth applying to. Built with Claude (scheduled tasks, skills and connectors). No code to deploy: the "program" is a set of prompts, a skill and a fact bank, with permissions kept as narrow as possible.

## What it does

1. **Collects** new postings from Indeed (search) and from LinkedIn job-alert emails in Gmail.
2. **Filters** them by title, date, language and geography.
3. **Reads the full job description** for anything that passes, within a call budget.
4. **Scores** each role from 0 to 100 against the fact bank: must-haves met / partial / missing, top priorities, red flags, and a verdict (Strong match, Stretch, Poor fit).
5. **Reports** a ranked shortlist and pushes a notification, but only when there is something new.
6. **On request**, tailors a CV for a chosen role using the same fact bank, then reviews the CV against the job description.

## How it works

```
 Gmail (LinkedIn alerts) ──┐
                           ├─► filter (title, date, language, geography)
 Indeed search ────────────┘          │
                                      ▼
                      read full job description (budgeted)
                                      │
                                      ▼
              score against fact bank  ◄── master-resume-template (facts only)
                                      │
                ┌─────────────────────┼──────────────────────┐
                ▼                     ▼                      ▼
        ranked shortlist     tracker row per role     Gmail label "screened"
        + push notification   (versioned updates)      (dedupe across runs)
```

| Piece | File | Role |
|---|---|---|
| Scheduled task | [`routines/daily-job-screen.md`](routines/daily-job-screen.md) | The prompt that runs at ~08:00, ~14:00 and ~19:00. The morning run does the full Indeed sweep; later runs only process new alerts. |
| Skill | [`skills/tailor-cv/SKILL.md`](skills/tailor-cv/SKILL.md) | The scoring method (Phase 0 duplicate check, Phase 1 recruiter analysis) and CV tailoring rules (Phase 2). |
| Fact bank | [`templates/fact-bank.example.md`](templates/fact-bank.example.md) | Every claim a CV may make, with scope notes ("designed, not live", "executed by Engineering"). The example uses a fictional candidate; my real one stays private. |
| Permissions | [`PERMISSIONS.md`](PERMISSIONS.md) | What the agent may and may not touch, and why. |
| Sample output | [`examples/sample-shortlist.md`](examples/sample-shortlist.md) | One real morning run, trimmed. |

## Design choices

- **Least privilege.** Gmail access is read plus one label; it can never send, reply or delete. It never browses LinkedIn. Indeed calls have a budget per run and stop after one retry. Details in [PERMISSIONS.md](PERMISSIONS.md).
- **No score without evidence.** A role whose full description couldn't be read is listed as "title-only", never scored.
- **One source of truth.** CVs can only use facts from the fact bank. When I confirm a new fact, it goes into the fact bank first, then into CVs.
- **Blunt scoring.** The prompt tells it not to inflate. Most roles come back as Poor fit, and that is what saves time.
- **Quiet when there's nothing to say.** Midday and evening runs notify only if there's a new Strong match, Stretch, or in-target title-only role.

## Results so far

- 80+ roles evaluated with the same method between September and October 2026, each with a saved analysis.
- A typical morning run reads ~15 job descriptions out of 45+ collected and shortlists 0–2.

## Set it up yourself

1. Create a Claude project for your search and add your own fact bank (start from `templates/fact-bank.example.md`) and a target-titles doc.
2. Add `skills/tailor-cv/SKILL.md` as a skill and replace the `{{PLACEHOLDERS}}`.
3. Connect Gmail and Indeed in Claude, and create a Gmail label for processed alerts; put its ID in the routine.
4. Create a scheduled task with the text of `routines/daily-job-screen.md`, filled in with your titles, locations and languages, running three times on weekdays.
5. Optional: a tracker (any table the agent can write to) for one row per role.

## What I'd improve next

- A per-run audit log of every tool call, so it's possible to check after the fact that the agent stayed inside its limits.
- Enforcing the call budget and the "label only" rule in the tool layer instead of in the prompt.
- Scoring calibration: compare scores with real outcomes (interviews, rejections) once there are enough of them.

## License

MIT. See [LICENSE](LICENSE).
