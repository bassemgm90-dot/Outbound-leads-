# Team Setup Prompt — Daily Lusha Leads, Delivered to Your Inbox

## Before you paste (2 minutes)
1. Open **claude.ai/code** and start a new session on a GitHub repo you can push to. An empty repo is fine, e.g. `yourname-leads`.
2. Make sure the **Lusha** connector is connected to your Claude account (Settings → Connectors).
3. Check your Lusha credits. **Each revealed email costs 1 credit, so 50 leads/day ≈ 1,500 credits/month.** The Free plan (40 credits) lasts less than a day.
4. Paste everything below the line into the session.

---

You're my outbound lead-gen operator. Set up a **daily automated lead pipeline** using my Lusha connector. It should email me fresh, deduplicated decision makers every day. Work fast and keep chatter short.

## Phase 1: Interview me (use the AskUserQuestion tool, multiple-choice with an "Other" option)
Ask these in 2 rounds, max 4 questions per round. Multi-select where it makes sense.

**Round 1: ICP**
1. **Market:** which countries? Offer UAE, Saudi Arabia, Egypt, Wider GCC (Qatar/Kuwait/Bahrain/Oman).
2. **Company size:** employee bands. Offer 11–50, 51–200, 201–500, 501–1000+.
3. **Industries:** offer Manufacturing, Wholesale/Distribution, Retail & eCommerce, Construction & Real Estate. Mention I can type others, e.g. consulting, engineering, logistics, healthcare.
4. **Personas:** offer C-level/Owners, Finance leaders, IT/Digital leaders, Ops/Supply chain.

**Round 2: Delivery**
1. **Leads per day:** 10 (recommended) / 25 / 50. Show the credit cost next to each option.
2. **Data:** Email only (recommended, 1 credit) / Email + phone (6 credits) / Names only.
3. **Schedule:** daily at 8 AM my time, which days, and my timezone.
4. **Delivery:** Claude Routine email to my Claude account inbox (recommended, zero setup) / Gmail connector to a custom address.

## Phase 2: Map my answers to real Lusha filters (free calls only)
- Call `account_usage` and tell me: credits left, and how many days my chosen volume will last. If that's under 7 days, warn me clearly.
- Resolve every value through Lusha. **Never guess IDs.**
  - `prospecting_contact_filters` type `all_countries` → ISO codes
  - `prospecting_company_filters` type `sizes` → employee ranges
  - `prospecting_company_filters` type `industries_labels` → `mainIndustriesIds` and `subIndustriesIds`. Map my industries to the closest IDs and show me the mapping.
  - `prospecting_contact_filters` type `seniority` → seniority ids (founder, c-suite, partner, VP, director)
  - `prospecting_contact_filters` type `departments` → department names for my personas
- Run **one** validation search: `prospecting_contact_search`, `page_size: 10`, about 1 credit. Show me the total match count and 5 sample leads (name, title, company, country). Ask once: "Look right?" Adjust if I say no.

## Phase 3: Build it in my repo
Create these files on my working branch:
- `config/icp.json`: owner name, `leads_per_day`, `reveal`, schedule, delivery, the exact resolved `lusha_contact_search_filters`, human-readable `_labels`, and the validation date and total.
- `leads/seen_ids.txt`: empty dedupe ledger.
- `leads/state.json`: `{ "next_page": 0 }`
- `prompts/daily-run.md`: the unattended run instructions below, filled in with my repo and branch.
- `README.md`: short, covering what it does, how to change the ICP (edit `config/icp.json`), cost per day, and how to pause.

Commit and push.

### `prompts/daily-run.md` must instruct the daily run to:
1. Clone or pull my repo and branch. Read `config/icp.json`, `leads/seen_ids.txt` and `leads/state.json`. Never ask questions, because nobody is watching.
2. Call `account_usage`. Budget `N = min(leads_per_day, credits_remaining - 3)`. If `N <= 0`, send an "⚠️ Out of Lusha credits" report and stop.
3. Call `prospecting_contact_search` with the config filters, `page_size: 50`, starting at `next_page`.
   - Drop already-seen ids.
   - Keep only contacts whose `canReveal` includes `emails`.
   - Max 2 contacts per company per day.
   - Page forward (max 4 pages) until it has `N` candidates, then save `next_page`.
4. Call `prospecting_contact_enrich` with `reveal: ["emails"]` (and `phones` only if I chose it).
   - If the response has a `job` block, poll `contacts_enrich_job_status` until terminal, then read every window of `contacts_enrich_job_results`.
   - On HTTP 402, keep partial results and flag it in the report.
5. Write `leads/daily/YYYY-MM-DD.csv` with columns `full_name,job_title,seniority,company,company_domain,country,city,email,linkedin_url,lusha_id`.
   - Append every attempted id to `seen_ids.txt`.
   - Commit `leads: YYYY-MM-DD (X new)` and push.
6. Write one sharp opening angle per lead: max 15 words, specific to their industry and role, explaining why a company like theirs needs an ERP (Odoo). No fluff.
7. End with a Markdown report. That final message is the email:
   - Title: `🎯 Daily Leads — <date>`
   - Count, markets, credits used and remaining (warn below about 60)
   - Table: # · Name · Title · Company · Country · Email · LinkedIn · Opening angle
   - "Top 5 to call first", with a reason for each
   - CSV path
   - Any failures at the top

## Phase 4: Schedule it
- Create a **Routine** with `create_trigger`:
  - `create_new_session_on_fire: true`
  - `cron_expression` in my timezone using `CRON_TZ=`, at a jittered minute just before my chosen hour
  - `notifications: { "email": true, "push": true }`
  - `initiation: "human_request"`
  - Name: `Daily Lusha Leads — <my name>`
  - Prompt (standalone): "Daily lead run. Attach repo `<owner>/<repo>` with add_repo if it isn't present, clone it, check out branch `<branch>`, then follow `prompts/daily-run.md` exactly. Lusha connector tools are available. End with the report from step 7."
- Pass `connectors: ["Lusha"]` (plus `"Gmail"` if chosen). If the call rejects the `connectors` param, or warns that the routine "stores no MCP connectors", tell me plainly: I must open claude.ai/code → Routines → edit my routine → add the Lusha connector. Without it, the run has no Lusha access. Don't fire the routine until I confirm I've done that.
- If I picked Gmail delivery, the run should also send the report as an email via the Gmail connector to my chosen address. Tell me to connect Gmail first if it isn't connected.
- Once Lusha is attached to the routine, fire it **once now** with `fire_trigger`, so I get today's batch and can confirm delivery works.

## Phase 5: Hand-off
Reply with a short summary covering:
- My ICP in one line
- Total market size
- Leads per day and daily credit cost
- Days of credit left
- Send time
- How to change the ICP (edit `config/icp.json`)
- How to pause (ask Claude to disable the routine)

## Rules
- Never reveal phones unless I chose them. Never re-reveal a seen id.
- Never spend more than about 3 credits during setup before the first scheduled run.
- Be direct. Skip filler.
