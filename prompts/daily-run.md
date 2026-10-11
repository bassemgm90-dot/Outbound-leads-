# Daily Lusha Lead Run — Operating Instructions

You are running an unattended daily lead-generation job. Nobody is watching: never stop to ask questions, never wait for approval. Do the job, then end with the email-ready report described in step 7. That final message IS the email the owner receives.

Repo: `bassemgm90-dot/Outbound-leads-`, branch `claude/lusha-daily-leads-automation-wvm2mb`.

## 1. Load state
- Read `config/icp.json` (filters, `leads_per_day`, `reveal`).
- Read `leads/seen_ids.txt`: one Lusha contact id per line. These were already delivered. **Never deliver or pay to reveal them again.**
- Read `leads/state.json` if it exists (`next_page`, defaults to 0).

## 2. Check budget (free)
- Call `account_usage`. Let `R` = credits remaining, `D` = daily rate-limit remaining.
- Budget: `N = min(leads_per_day, R - 3)`. Keep 3 credits back for searches.
- If `N <= 0`: skip to step 7 and send a **"⚠️ Out of Lusha credits"** report. Say how many credits are left and that the plan needs a top-up (dashboard.lusha.com). Don't search or reveal anything.

## 3. Find fresh candidates
- Call `prospecting_contact_search` with exactly the filters in `config/icp.json → lusha_contact_search_filters` (including the `exclude` block — it removes companies already on an ERP), `page_size: 50`, `page: next_page`. Each page of 50 costs about 2 credits.
- Drop any result whose id is in `seen_ids.txt`.
- Keep only results whose `canReveal` includes `emails`.
- Prefer a mix of companies: **max 2 contacts per company per day**.
- If you have fewer than `N` candidates, fetch the next page. Stop after 4 pages in one run.
- Save the page after the last one you used as `next_page` in `leads/state.json`.

## 4. Reveal emails
- Call `prospecting_contact_enrich` with the candidate ids (up to 50 per call) and `reveal: ["emails"]`. **Never reveal phones.**
- If the response has a `job` block, poll `contacts_enrich_job_status` until `isTerminal: true`. Then read every window of `contacts_enrich_job_results`.
- If a call returns HTTP 402 (out of credits), keep what was revealed so far and flag it in the report.
- A lead counts only if a business email came back. Contacts with no email still go into the seen list, so you don't retry them.

## 5. Persist
- Write `leads/daily/YYYY-MM-DD.csv` using today's Africa/Cairo date, with these columns:
  `full_name,job_title,seniority,company,company_domain,country,city,email,linkedin_url,lusha_id`
- Append every id you attempted (found or not) to `leads/seen_ids.txt`.
- Commit with the message `leads: YYYY-MM-DD (X new)` and run `git push -u origin claude/lusha-daily-leads-automation-wvm2mb`. If the push fails on the network, retry with backoff (2s, 4s, 8s, 16s).

## 6. Opening angle (no extra credits)
For each lead, write one short line (max 15 words): a specific reason a company like theirs feels pain that an ERP (Odoo) solves. Base it on their industry and role. Examples:
- Contractor: job costing and progress billing across multiple projects.
- Distributor: multi-warehouse stock accuracy and landed costs.
- CFO: month-end close speed and consolidated reporting.
No generic fluff.

## 7. Final report (this becomes the email)
End your run with exactly this structure, in Markdown:

```
# 🎯 Daily Leads — <Weekday, DD Mon YYYY>
**<X> new decision makers** · UAE / KSA / Egypt · 11–200 employees
Credits used today: <c> · Remaining: <r> <⚠️ if r < 60: "≈ <days> days left — top up soon">

| # | Name | Title | Company | Country | Email | LinkedIn | Opening angle |
|---|------|-------|---------|---------|-------|----------|---------------|
| 1 | ... |

**Today's top 5 to call first:** <names + one-line why (seniority, company fit)>

CSV: leads/daily/YYYY-MM-DD.csv (repo bassemgm90-dot/Outbound-leads-)
```

If something failed (credits, rate limit, push), add a short **Issues** line at the top. Don't hide failures.
