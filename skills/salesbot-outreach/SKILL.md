---
name: salesbot-outreach
description: Find a handful of well-matched LinkedIn leads with Salesbot, enrich them and reach them with personal messages. Use when the user asks to find, research or contact people on LinkedIn, or to add leads to the Salesbot CRM.
---

# Targeted LinkedIn outreach with Salesbot

Salesbot is built for a few well-chosen people, not bulk blasting. Prefer 5 strong leads over 50 weak ones.

## Before anything else
1. `get_linkedin_status` — if the account is not connected, call `connect_linkedin` and give the user the link. Stop until it is connected.
2. `get_daily_limits` — know how many invitations and messages are left today. Never try to work around a limit.

## Finding people
- Known role and place: `search_linkedin_people` (results are not saved).
- A Sales Navigator search or list URL from the user: `search_linkedin_navigator`.
- Companies first: `import_linkedin_company_list` → `list_companies` → research with `search_web` → `update_company_research` → find the decision-maker with `search_linkedin_people`.
- Companies that are hiring: `search_job_postings` → `get_job_posting_details` (its `hiring_team` is the person to contact).

Save every person you want to keep with `upsert_linkedin_contact` (pass `prospect_company_id` when you have it). Check `check_blacklist` before contacting a company.

## Research
`enrich_contacts` (max 8 per call) loads the full profile. It visits the profile on LinkedIn, so enrich only people you really intend to contact. Use `get_lead_context` for everything Salesbot already knows about a lead.

## Contacting
- For more than one or two people, use a campaign (see the `salesbot-campaign` skill): it paces the sending and keeps the account safe.
- One-off actions: `send_connection_request`, `send_linkedin_message`, `reply_to_chat`. Write the message for that person — mention something concrete from their profile or company.
- If a tool answers `APPROVAL_REQUIRED`, the message waits for the user in the Salesbot app. Tell the user; never say it was sent.

## Afterwards
Record what you learned: `log_crm_note`, `set_deal_stage`, `create_task` for follow-ups.

## Errors
Errors come as `{ ok: false, code, error }`. `RATE_LIMITED`, `DAILY_LIMIT_REACHED`, `HOURLY_LIMIT_REACHED`, `OUTSIDE_ALLOWED_HOURS`: stop and tell the user when it can continue — do not retry in a loop. `ACCOUNT_BLOCKED`: stop all LinkedIn actions and tell the user.
