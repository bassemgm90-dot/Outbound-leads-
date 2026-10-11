# Outbound Leads — Daily Lusha Pipeline

Every morning, a scheduled Claude Routine pulls **fresh, deduplicated decision makers** from Lusha. It reveals their business emails, writes a tailored ERP opening angle for each lead, and emails you the list.

## Current ICP (Bassem)
| | |
|---|---|
| Markets | UAE · Saudi Arabia · Egypt |
| Company size | 11–50, 51–200 employees |
| Industries | Construction & contracting, Manufacturing, Real Estate, Retail & Wholesale, Business/Engineering consulting, eCommerce |
| Personas | Founders, C-suite, Partners, VPs, Directors in GM, Finance, IT, Ops |
| Volume | 50 leads/day, email only |
| Schedule | Every day, ~08:00 Cairo |
| Excluded | Companies already on a core ERP (Odoo ERP, SAP, NetSuite, Dynamics) |
| Matching pool | ~53,000 contacts (validated 2026-10-11) |

## Cost
- Search: 1 credit per 25 results, about 2–4 credits per day
- Email reveal: 1 credit per lead

50 leads/day ≈ **52–55 credits/day ≈ 1,600/month**. Contacts with no email found aren't charged, and they're logged so they never get retried.

## Files
- `config/icp.json`: filters and volume. **Edit this to change targeting.** The next run picks it up automatically.
- `prompts/daily-run.md`: the unattended run instructions the Routine follows.
- `prompts/TEAM_SETUP_PROMPT.md`: paste-ready prompt for team members to set up their own pipeline.
- `leads/seen_ids.txt`: dedupe ledger. Never delete it unless you want repeat leads.
- `leads/state.json`: search page cursor.
- `leads/daily/YYYY-MM-DD.csv`: each day's leads.

## Control
Ask Claude: "pause / resume / change the time of my Daily Lusha Leads routine." You can also manage it under Routines on claude.ai/code.
