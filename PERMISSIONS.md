# Permissions and guardrails

This agent reads my inbox and job boards on a schedule while nobody is watching. The design question was the same one every agent raises: **what is it allowed to touch, and how do I know it stayed inside those limits?** The rule I followed was to give it the narrowest access that still does the job.

## What it can and can't do

| System | Allowed | Not allowed | Why |
|---|---|---|---|
| Gmail | Search job-alert emails; read them; add one label ("Claude screened") to threads it has processed, and create that label if it's missing | Send, reply, forward, delete, trash, mark as spam, or touch any other email | It only needs to read alerts and remember what it has seen. A label is the smallest write that does that. |
| LinkedIn | Nothing. It reads LinkedIn's alert emails, not LinkedIn itself | Log in, browse, or fetch job pages | Automated browsing of LinkedIn breaks its terms, and the agent doesn't need it: it looks the same job up on Indeed instead. |
| Indeed | Search and read job details, within a call budget per run (~45 in the morning, ~12 midday/evening) | Apply, message employers, or keep retrying | The account is rate-limited. On a limit error it waits once, retries once, then stops and reports what it skipped. |
| Project docs (Claude Projects) | Read the fact bank and target titles; read and write evaluation notes | Change the fact bank on its own | The fact bank is the single source of truth for what I have actually done. It only changes when I confirm a new fact. |
| Tracker | Add or update one row per evaluated role, with a version check so it never overwrites a change made elsewhere | Delete rows | Keeps a history of every decision. |
| Notifications | Push a summary only when there is something new worth acting on | Notify on empty runs | Attention is a cost. "Nothing new" runs stay silent. |

## Guardrails on output, not just access

Access limits stop the agent from *doing* the wrong thing. These rules stop it from *saying* the wrong thing:

- **No score without the full job description.** If it only has a title (common with LinkedIn alerts), it lists the job as "title-only" and asks me to paste the description.
- **No invented facts.** Every CV claim must come from the fact bank. Scope notes in the fact bank ("designed, not live", "executed by Engineering", "never claim X") are hard rules.
- **No silent re-scoring.** If a role was evaluated before, it reports the earlier result and asks whether to reuse, re-run or skip.
- **Deduplication across runs.** The same job appears in many alert emails; the label plus a 24-hour lookback prevents reporting it twice.
- **Blunt by default.** The prompt says "do not inflate scores". Most roles come back as Poor fit, which is the point.

## What I'd add next

- An audit log per run (which tools were called, with what inputs) written to the project, so I can check after the fact that the agent stayed inside these limits, not just trust that it did.
- A hard cap enforced by the tool layer rather than by the prompt. Today the Indeed budget and the "label only" rule live in the instructions; a stricter setup would only expose the read and label tools in the first place.
