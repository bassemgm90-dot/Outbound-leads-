# Daily Job Shortlist Run: Operating Instructions

This is an unattended daily job. Nobody is watching: never stop to ask questions. **Never apply to any job and never message anyone on LinkedIn.** Your job ends with a shortlist. Bassem applies himself.

Repo: `bassemgm90-dot/Outbound-leads-`, branch `claude/lucid-babbage-shxt4s`.

## 1. Load state
- `config/job_scope.json`: tiers, markets, exclusions, limits.
- `profile/candidate.md`: who Bassem is and the proof points to use.
- `jobs/seen_job_ids.txt`: LinkedIn job ids already processed. **Skip these.**

## 2. Collect jobs from Gmail
- Search Gmail: `from:(jobalerts-noreply@linkedin.com OR jobs-noreply@linkedin.com OR jobs-listings@linkedin.com) newer_than:2d`. On the first run, when `seen_job_ids.txt` holds only its header, use `newer_than:14d` instead. Read each thread with `get_thread` and `messageFormat: PLAIN_TEXT`.
- In those emails each job is a block of lines: **title**, **company**, **location**, optional badges ("Top applicant", "Fast growing"), then `View job: https://www.linkedin.com/comm/jobs/view/<JOB_ID>/...`. Pull the title, company, location and JOB_ID from each block.
- The subject line of single-job emails also works ("<Title> at <Company>", "<Company> is hiring: <Title>").
- The clean link is `https://www.linkedin.com/jobs/view/<JOB_ID>/`. Remove all tracking parameters.
- Drop any JOB_ID that's in `seen_job_ids.txt` or appears twice in this run.

## 3. Score each job (0–10)
- Apply `exclude` first, so excluded jobs score 0 (Odoo, junior/SDR titles, retail, commission-only, out-of-market Egypt roles).
- Put each job in a tier (A, B or C) using `target_tiers`. **Company size decides between A and B.** Estimate it from what you know or a quick web search, and write "size unverified" if you can't confirm it. Don't fetch linkedin.com pages, because they block automated access.
- Base the score on title fit, size fit, market (primary > secondary > remote), and `boost` matches.
- Shortlist the jobs scoring `>= min_score_to_shortlist`, up to `shortlist_max_per_day`, highest first.

## 4. For each shortlisted job, write:
- **Why it fits:** one line pointing to a specific part of the profile.
- **Risk:** one line on the gap a recruiter will see and how to answer it.
- **Pitch:** 3 sentences, 70 words max, in first person, ready to paste into Easy Apply's "message to hiring team" or a cover note. Open with the most relevant proof point from `candidate.md`. Never make up numbers. Use only facts from `candidate.md`.

## 5. Hiring manager lookup (only if `lusha_hiring_manager_lookup.enabled` is true)
- Only for jobs scored 8 or higher, and at most `max_per_day`. Use Lusha `prospecting_contact_search` for that company with sales/general-management seniority Director or above. Reveal **emails only**. Put the name and email in the `hiring_manager` column. Skip this step completely when it's disabled.

## 6. Persist
- Append **every** processed JOB_ID (shortlisted or not) to `jobs/seen_job_ids.txt`.
- Append shortlisted jobs to `jobs/pipeline.csv` with `status=shortlisted`.
- Write the report to `jobs/shortlist/YYYY-MM-DD.md` (Asia/Dubai date).
- Commit with the message `jobs: YYYY-MM-DD (X shortlisted / Y scanned)`, then run `git push -u origin claude/lucid-babbage-shxt4s`. If the push fails on the network, retry with backoff (2s, 4s, 8s, 16s).

## 7. Deliver
- Send the report with Gmail `send_message` to `bassem.gm90@gmail.com` **only**, with the subject `🎯 Job shortlist: <DD Mon> (<X> roles)`. **Never send to any @odoo.com address.** His employer must not see this.
- Then end with the same report as your final message.

Report format:
```
# 🎯 Job Shortlist: <Weekday, DD Mon YYYY>
**<X> roles worth applying to** (from <Y> new jobs scanned)

## 1. <Title> · <Company> · <Location>: **<score>/10** (Tier <A/B/C>)
Apply: https://www.linkedin.com/jobs/view/<id>/
Why: ...
Risk: ...
Pitch: "..."

...

**Apply first:** the top 3 and why.
**Skipped:** <n> (one line on the main reasons)
```
If there were no new jobs, send a one-line report that says so and suggests broadening the LinkedIn Job Alerts. If something failed (Gmail, push), put an **Issues** line at the top. Don't hide failures.
