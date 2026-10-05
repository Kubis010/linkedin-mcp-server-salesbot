# LinkedIn MCP Server with CRM (Salesbot)

> **What is this?** `linkedin-mcp-server-salesbot` is a **Model Context Protocol - MCP server for AI‑assisted LinkedIn relationship operations**. It lets AI assistants — **Claude Desktop, ChatGPT, Cursor** — help you research and organize professional contacts, draft deeply personalized messages **for your review and approval**, sync inbox conversations, enrich profiles, and pull web context — all under your direction. It's built for **hyper‑targeted, meaningful outreach** (find 5 ideal contacts, read their recent posts, write 5 thoughtful notes), **not bulk blasting**. Every send is gated by **human‑in‑the‑loop approval** and **enforced, server‑side daily/hourly safety thresholds** that keep your LinkedIn account within safe limits.

It runs as a hosted service at `app.salesbot.cz` and speaks the MCP **Streamable HTTP** transport. Users sign in with OAuth 2.1 (or use an API key in clients that cannot sign in). LinkedIn actions go through a third‑party LinkedIn integration provider; LinkedIn credentials are never stored by the AI.

> Product page and documentation: [https://salesbot.cz/en/mcp-server](https://salesbot.cz/en/mcp-server)

- **Keywords:** model context protocol, mcp server, linkedin api, linkedin automation, claude desktop, cursor, ai agents, sales automation.
- **Compatible clients:** Claude Desktop, Claude API/MCP, Cursor, any MCP Streamable‑HTTP client.

## Quick facts

| | |
|---|---|
| **Endpoint** | `https://app.salesbot.cz/api/mcp` |
| **Transport** | MCP Streamable HTTP (POST + SSE) |
| **Auth** | OAuth 2.1 sign-in, or `x-mcp-api-key: sb_mcp_…` header |
| **Tool count** | 82 |
| **License** | MIT |

## Connect with sign-in (OAuth): claude.ai, Cowork, Claude Code

The server supports OAuth 2.1, so no key needs to be copied:

- **claude.ai / Cowork:** Settings → Connectors → *Add custom connector* → `https://app.salesbot.cz/api/mcp` → *Connect*. Sign in to Salesbot and allow access on the consent page.
- **Claude Code plugin** (adds three skills on top of the connection: `salesbot-outreach`, `salesbot-campaign` and `salesbot-email`):

```bash
claude plugin marketplace add Kubis010/linkedin-mcp-server-salesbot
claude plugin install salesbot@salesbot
```

  Then start `claude`, run `/mcp`, select the Salesbot server and sign in.

  To update later, refresh the marketplace first (otherwise the old version is reported as current):

```bash
claude plugin marketplace update salesbot
claude plugin update salesbot@salesbot
```

You can see and remove connected apps in the Salesbot app under Settings → MCP → Connected apps. The plugin contains no code that runs on your machine; it only talks to `https://app.salesbot.cz/api/mcp`.

## ChatGPT and Codex

For a private ChatGPT connection, add `https://app.salesbot.cz/api/mcp` in developer mode and choose OAuth sign-in. Complete Salesbot consent in the browser. The discovery endpoints are available, but end-to-end ChatGPT sign-in has not yet been verified by this repository's readiness check.

The portable OpenAI package is defined by `plugin.json` and `mcp.json`; it reuses the existing workflows and logo without changing the Claude package. It is a **submission candidate, not an approved public listing**. See the [OpenAI readiness and submission checklist](docs/openai-submission.md) for tests, review-account requirements and the third-party integration policy gate.

Clients that cannot sign in can use an API key, see below.

## What the plugin runs and sends

- **Runs nothing on your machine.** The plugin is three Markdown skills and one remote MCP server entry; there are no hooks, scripts or binaries.
- **Talks to one server:** `https://app.salesbot.cz/api/mcp` (Salesbot, operated by Sales Robots s.r.o.). Sign-in uses OAuth 2.1 on `app.salesbot.cz`.
- **What reaches Salesbot:** the arguments of the tools Claude calls (for example a search query, a contact id, the text of a message or e-mail you asked to send). Salesbot then acts in your connected LinkedIn account and e-mail tools within your limits and approval settings. It does not read your conversation or local files.
- **Data handling:** see the [privacy policy](https://salesbot.cz/en/privacy) and [terms](https://salesbot.cz/en/terms). Remove access at any time in the Salesbot app → Settings → MCP → Connected apps.

## Connect with an API key (Cursor, Codex, Claude Desktop config)

Add this to your MCP client config. Get the `sb_mcp_…` key in the Salesbot app under **Settings → MCP**.

```json
{
  "mcpServers": {
    "linkedin-automation": {
      "url": "https://app.salesbot.cz/api/mcp",
      "headers": {
        "x-mcp-api-key": "sb_mcp_YOUR_API_KEY",
        "Accept": "application/json, text/event-stream"
      }
    }
  }
}
```

> **Important:** send the API key in the **`x-mcp-api-key`** header, **not** `Authorization: Bearer`; `Authorization` is reserved for OAuth access tokens.

## Authentication

- **OAuth 2.1** (authorization code + PKCE, dynamic client registration). A request without credentials gets `401` with `WWW-Authenticate: Bearer resource_metadata="https://app.salesbot.cz/.well-known/oauth-protected-resource"`; clients discover the authorization server from there. The access token goes in `Authorization: Bearer`.
- **MCP API key** (`sb_mcp_…`) — long‑lived; generated in the Salesbot app, stored only as a SHA‑256 hash. Send in `x-mcp-api-key`.
- An active subscription/trial is required.

## How do I authenticate LinkedIn?

The AI can do it without leaving the chat:

1. Call `get_linkedin_status` — reports whether LinkedIn is connected/active/blocked.
2. If not connected, call `connect_linkedin` — returns a white‑labeled `https://auth.salesbot.cz/…` link. The user opens it, completes LinkedIn login, done.

Or connect in the app: **Settings → LinkedIn → Connect**.

## Tools

Each tool returns text content; errors return `{ "ok": false, "code": "<CODE>", "error": "<message>" }`.

### Connection
```json
{ "name": "get_linkedin_status", "input": { "profile_id": "uuid (optional)" } }
{ "name": "connect_linkedin",   "input": { "profile_id": "uuid (optional)", "reconnect": "boolean (optional)" } }
```

### Lead discovery
```json
{ "name": "search_linkedin_people",    "input": { "title": "string (required)", "location": "string", "locationId": "string", "network": "['S'|'O']", "limit": "number 1-50" } }
{ "name": "search_google_xray",        "input": { "jobTitle": "string (required)", "location": "string", "keywords": "string[]", "excludeWords": "string[]", "limit": "number 1-100" } }
{ "name": "search_linkedin_navigator", "input": { "search_url": "string (required)", "limit": "number 1-100" } }
{ "name": "import_linkedin_company_list", "input": { "search_url": "string (required)", "list_name": "string (required)", "limit": "number", "cursor": "string", "save_to_crm": "boolean" } }
{ "name": "list_company_lists",          "input": {} }
{ "name": "list_companies",              "input": { "company_list_id": "uuid (required)", "limit": "number", "offset": "number", "query": "string" } }
{ "name": "update_company_research",     "input": { "prospect_company_id": "uuid (required)", "web_research": "string (required)", "website": "string", "domain": "string", "research_urls": "string[]" } }
{ "name": "search_job_postings",       "input": { "keywords": "string (required)", "location": "string", "locationId": "string", "seniority": "string[]", "job_type": "string[]", "presence": "string[]", "date_posted": "number", "easy_apply": "boolean", "limit": "number 1-50" } }
{ "name": "search_web",                "input": { "query": "string (required)", "limit": "number 1-30", "country": "string (default cz)", "language": "string (default cs)" } }
{ "name": "read_company_website",      "input": { "url": "string (required)", "extra_urls": "string[] (max 3, same domain)" } }
{ "name": "get_job_posting_details",   "input": { "job_id": "string (required)" } }
```
`search_google_xray` saves the profiles it finds into a "Google X-Ray" contact list (deduplicated) and returns their `contact_id`s — ready to enrich, add to a campaign, or push into the CRM.

`search_job_postings` searches LinkedIn job postings via the connected account (Classic search, no Recruiter needed). Returns job offers with company info — great for finding companies actively hiring for a specific role. Combine with `search_linkedin_people` to find the hiring manager.

`search_web` is a general-purpose Google search (not restricted to LinkedIn). Use Google operators like `site:jobs.cz`, `intitle:`, `OR` to search job portals, company websites, or news. Results are NOT saved to contacts — this is a research/discovery tool.

`read_company_website` reads a company's own public website – the homepage plus up to 3 contact/about/services pages on the same host – and returns a short extract with source URLs and generic company e-mails (info@, sales@ …); personal e-mails and phone numbers are masked and no employee lists are collected. It identifies as SalesbotBot, honours robots.txt (and does not read a site whose robots.txt cannot be checked), noindex/noai and TDM reservations, reads only public addresses, stops at logins, CAPTCHAs and paywalls, and is limited to 60 reads per account per day. Nothing is stored; save the summary with `update_company_research`. A found address is not consent to contact. Rules: [salesbot.cz/en/terms#website-reading](https://salesbot.cz/en/terms#website-reading).

`get_job_posting_details` takes a `job_id` from `search_job_postings` and returns the full posting — most importantly `hiring_team`, the recruiter or hiring manager who posted the role, with their LinkedIn id and whether a free InMail is available. Also returns `applicants_counter` / `views_counter` as urgency signals. Typical flow: `search_job_postings` → `get_job_posting_details` → `enrich_contacts` → campaign.

Company/account-list workflow: `import_linkedin_company_list` → `list_companies` → `search_web` → `update_company_research` → `search_linkedin_people` → `upsert_linkedin_contact` with the returned `prospect_company_id`. Companies are stored separately from people and can enter CRM before a decision-maker is known.

### Contacts
```json
{ "name": "upsert_linkedin_contact", "input": { "profile_url": "string (required)", "full_name": "string", "company": "string", "position": "string", "headline": "string", "prospect_company_id": "uuid" } }
{ "name": "get_contact_profile", "input": { "contact_id": "uuid (required)" } }
{ "name": "list_lead_lists",    "input": {} }
{ "name": "list_contacts",       "input": { "list_id": "uuid (required)", "limit": "number", "offset": "number" } }
{ "name": "enrich_contacts",     "input": { "contact_ids": "uuid[] (required, max 8)", "profile_id": "uuid (optional)" } }
{ "name": "create_lead_list",    "input": { "name": "string (required)", "description": "string" } }
{ "name": "add_contacts_to_list", "input": { "list_id": "uuid (required)", "contact_ids": "uuid[] (required)" } }
{ "name": "remove_contacts_from_list", "input": { "contact_ids": "uuid[] (required)", "list_id": "uuid" } }
{ "name": "sync_linkedin_connections", "input": { "cursor": "string", "sync_id": "string", "limit": "number ≤500" } }
{ "name": "list_blacklist",      "input": { "query": "string", "limit": "number", "offset": "number" } }
{ "name": "check_blacklist",     "input": { "contact_id": "uuid", "company": "string", "domain": "string" } }
{ "name": "delete_contacts",     "input": { "contact_ids": "uuid[] (required)", "confirm": "true (required)", "delete_crm_leads": "boolean", "force": "boolean" } }
```
Every contact lives in exactly one list: `add_contacts_to_list` moves contacts, `remove_contacts_from_list` moves them back to `CRM Imports` (nothing is deleted). `sync_linkedin_connections` pages through your own 1st-degree connections (max 500 per page, paced by the server). `list_blacklist` / `check_blacklist` read the company-wide blacklist.

`delete_contacts` deletes permanently, including campaign history. Call it only on the user's explicit request, after showing them the contacts, with `confirm: true`. To stop outreach to someone, use `exclude_contacts_from_campaign` instead.
`upsert_linkedin_contact` is the idempotent path for an exact, already-known LinkedIn profile URL. It creates the contact in the `CRM Imports` list or returns the existing `contact_id`, so CRM integrations can safely call it before `add_contacts_to_campaign` without relying on Google search.

`list_lead_lists` returns each contact group's `list_id`, name, description and contact count. Pass a returned `list_id` to `list_contacts`.

Typical contact workflow: `list_lead_lists` → `list_contacts` → `add_contacts_to_campaign`. Contacts still belong to a lead list, but adding an existing contact to a campaign only requires its `contact_id` and the target `campaign_id`.

`search_linkedin_people` and `search_linkedin_navigator` return raw LinkedIn search results. They do not persist contacts; call `upsert_linkedin_contact` for each profile you want to save or add to a campaign.

`enrich_contacts` scrapes each contact's full LinkedIn profile via the connected account (headline, location, current company & position, full work history, education, skills) and saves it onto the contact. Great right after `search_google_xray`.

### Campaigns
```json
{ "name": "list_campaigns",           "input": { "status": "draft|running|paused|completed|stopped (optional)" } }
{ "name": "create_campaign",          "input": { "name": "string (required)", "profile_id": "uuid (required)", "description": "string", "daily_limit": "number", "sender_context": "string", "steps": "[{ action: 'connect'|'message'|'visit', delay_hours, use_ai, ai_prompt, ai_template, message_mode: 'template'|'creative', send_without_message }] (required)" } }
{ "name": "update_campaign_settings", "input": { "campaign_id": "uuid (required)", "name": "string", "description": "string", "daily_limit": "number", "sender_context": "string", "auto_approve_messages": "boolean", "status": "running|paused|draft|stopped" } }
{ "name": "start_campaign",           "input": { "campaign_id": "uuid (required)" } }
{ "name": "stop_campaign",            "input": { "campaign_id": "uuid (required)" } }
{ "name": "add_contacts_to_campaign", "input": { "campaign_id": "uuid (required)", "contact_ids": "uuid[] (required)" } }
{ "name": "exclude_contacts_from_campaign", "input": { "campaign_id": "uuid (required)", "contact_ids": "uuid[] (required)", "reason": "string" } }
{ "name": "list_campaign_queue",      "input": { "campaign_id": "uuid (required)", "limit": "number" } }
```
`message_mode` decides how a message step uses its sample (`ai_template`): `template` sends the sample word for word and the AI only fills its fields (e.g. `[Firma]` → "Škoda Auto"); `creative` has the AI write a new message for each lead from `ai_prompt`, with the sample only as a style example. The campaign's `sender_context` ("About me") is used only where the prompt says `{{o_mne}}`. Links in the prompt or sample are kept as written.

`exclude_contacts_from_campaign` stops all further outreach to those contacts but keeps their history. `list_campaign_queue` shows each contact's state, invitation status and the next tool to call.

### AI messaging (write → approve → send)
```json
{ "name": "prepare_campaign_messages", "input": { "campaign_id": "uuid (required)", "limit": "number ≤10" } }
{ "name": "generate_campaign_message", "input": { "campaign_contact_id": "uuid (required)", "step_id": "uuid (required)", "custom_instructions": "string" } }
{ "name": "list_pending_approvals",    "input": { "campaign_id": "uuid", "limit": "number" } }
{ "name": "approve_message",           "input": { "campaign_contact_id": "uuid (required)", "edited_messages": "[{step_id, message}]", "skip_gpt_check": "boolean" } }
{ "name": "reject_message",            "input": { "campaign_contact_id": "uuid (required)", "reason": "string (required)" } }
{ "name": "list_mcp_pending_actions",  "input": { "action_type": "string", "limit": "number" } }
```
`prepare_campaign_messages` queues drafts for up to 10 pending contacts per call; repeat while `remaining_pending` > 0. When the campaign has AI approval on (`auto_approve_messages`), Salesbot checks each draft — the contact's name and gender, company, leftover placeholders, signature and grammar — and approves only what passes; the rest waits in `list_pending_approvals`. `list_mcp_pending_actions` lists one-off messages and invitations waiting for the user's approval (read-only).

### E-mail (Smartlead, Instantly or your own mailbox)
```json
{ "name": "list_email_integrations", "input": {} }
{ "name": "list_email_campaigns",    "input": { "provider": "smartlead|instantly (required)" } }
{ "name": "send_email",              "input": { "provider": "smartlead|instantly|mailbox (required)", "provider_campaign_id": "string (required)", "body": "string (required)", "subject": "string", "followup_body": "string", "crm_lead_id": "uuid", "contact_id": "uuid", "email": "string" } }
{ "name": "list_email_outreach",     "input": { "status": "string", "crm_lead_id": "uuid", "contact_id": "uuid", "limit": "number" } }
{ "name": "get_email_status",        "input": { "outreach_id": "uuid (required)" } }
{ "name": "cancel_email",            "input": { "outreach_id": "uuid (required)", "reason": "string" } }
```
`send_email` sends a personal e-mail to one person. With Smartlead / Instantly, pass a campaign id from `list_email_campaigns`; its sequence must use `{{email_subject}}` and `{{email_body}}`, and the provider then sends on its own schedule. With `provider: "mailbox"`, pass the `mailbox_id` from `list_email_integrations`; Salesbot sends it from your own Outlook / IMAP mailbox within your sending hours, a few minutes apart and capped per day.

Every e-mail is checked before it is sent: the greeting (right name, and pane/paní by the contact's gender), the company, leftover placeholders such as `{{company}}`, Czech grammar, and no signature in the body when the mailbox adds its own. A failed check returns `AI_CHECK_FAILED` with the reason and nothing is sent; fix it and call again. A second failure goes to the user's manual approval. If approval is required, the e-mail waits as `pending_approval`. `cancel_email` withdraws an e-mail that has not gone out yet.

### Direct LinkedIn actions
```json
{ "name": "send_connection_request", "input": { "linkedin_id": "string (required)", "profile_id": "uuid (required)", "contact_id": "uuid" } }
{ "name": "send_linkedin_message",   "input": { "linkedin_id": "string (required)", "message": "string ≤5000 (required)", "profile_id": "uuid (required)" } }
{ "name": "publish_linkedin_post",   "input": { "profile_id": "uuid (required)", "text": "string ≤3000 (required)", "external_link": "string", "as_organization": "string", "auto_publish": "boolean" } }
{ "name": "get_daily_limits",        "input": { "profile_id": "uuid (optional)" } }
```

`publish_linkedin_post` has its own server-side safety limit of **1 published post per LinkedIn account in a rolling 60-minute window**. This applies to both personal and organization posts and is reported as the `posts` quota by `get_daily_limits`. When the quota is exhausted, the tool returns `HOURLY_LIMIT_REACHED` and does not publish the post.

### Inbox (real‑time)
```json
{ "name": "list_inbox_chats",  "input": { "profile_id": "uuid (optional)", "limit": "number 1-50", "cursor": "string" } }
{ "name": "get_chat_messages", "input": { "chat_id": "string (required)", "profile_id": "uuid (optional)", "limit": "number 1-50", "cursor": "string" } }
{ "name": "reply_to_chat",     "input": { "chat_id": "string (required)", "message": "string ≤5000 (required)", "profile_id": "uuid (optional)" } }
{ "name": "mark_chat_read",    "input": { "chat_id": "string (required)", "profile_id": "uuid (optional)" } }
```

### CRM (pipeline, notes, tasks, message store)
The CRM is a persistent pipeline separate from contacts. A lead enters it when added to a campaign, or when any of these tools first touch it. It also acts as a durable store for generated outreach copy: save email / LinkedIn drafts and follow-ups with `save_lead_message`, read them back with `list_lead_messages` or `get_lead_context`, and send e-mails with `send_email` (see E-mail above).
```json
{ "name": "add_companies_to_crm", "input": { "prospect_company_ids": "uuid[] (max 100)", "companies": "[{ name, website?, industry?, location?, headcount?, linkedin_url?, notes? }] (max 50; no LinkedIn needed)" } }
{ "name": "add_crm_contact", "input": { "full_name": "string (required)", "email": "string?", "phone": "string?", "company": "string?", "position": "string?", "website": "string?", "crm_company_id": "uuid?", "notes": "string?" } }
{ "name": "add_contacts_to_crm", "input": { "contact_ids": "uuid[] (required)" } }
{ "name": "search_crm_leads",   "input": { "query": "string", "stage": "string", "campaign_id": "uuid", "list_id": "uuid", "crm_company_id": "uuid", "sort_by": "string", "sort_direction": "asc|desc", "limit": "number", "offset": "number" } }
{ "name": "update_crm_lead",    "input": { "contact_id": "uuid", "crm_lead_id": "uuid", "stage": "string", "deal_value": "number", "clear_deal_value": "boolean", "email": "string", "company": "string", "note": "string" } }
{ "name": "set_deal_stage",     "input": { "contact_id": "uuid (required)", "stage": "string (required)", "note": "string" } }
{ "name": "log_crm_note",       "input": { "contact_id": "uuid (required)", "summary": "string (required)", "pain_points": "string[]", "sentiment": "positive|neutral|negative" } }
{ "name": "save_lead_message",  "input": { "contact_id": "uuid (required)", "body": "string (required)", "channel": "email|linkedin", "kind": "string e.g. initial|followup", "subject": "string", "status": "draft|queued|sent", "message_id": "uuid (update existing)" } }
{ "name": "list_lead_messages", "input": { "contact_id": "uuid (required)", "channel": "email|linkedin", "kind": "string", "limit": "number" } }
{ "name": "create_task",        "input": { "title": "string (required)", "contact_id": "uuid", "due_at": "ISO 8601", "details": "string" } }
{ "name": "list_tasks",         "input": { "status": "open|done|cancelled|all", "contact_id": "uuid", "limit": "number" } }
{ "name": "complete_task",      "input": { "task_id": "uuid (required)", "status": "done|open|cancelled" } }
{ "name": "get_lead_context",   "input": { "contact_id": "uuid (required)", "notes_limit": "number" } }
{ "name": "update_contact",     "input": { "contact_id": "uuid (required)", "email": "string", "phone": "string", "location": "string", "company": "string", "position": "string", "headline": "string" } }
{ "name": "set_lead_fields",    "input": { "contact_id": "uuid (required)", "fields": "object { field_key: value }" } }
{ "name": "export_crm",         "input": { "limit": "number (default 5000, max 20000)" } }
{ "name": "delete_crm_leads",   "input": { "crm_lead_ids": "uuid[] (required)", "confirm": "true (required)" } }
```
Companies (accounts) are created automatically from lead company names and LinkedIn company imports:
```json
{ "name": "list_crm_companies",  "input": { "query": "string", "stage": "string", "limit": "number", "offset": "number" } }
{ "name": "get_crm_company",     "input": { "crm_company_id": "uuid (required)" } }
{ "name": "update_crm_company",  "input": { "crm_company_id": "uuid (required)", "notes": "string", "append_notes": "boolean", "name": "string", "website": "string", "industry": "string", "location": "string", "headcount": "string" } }
{ "name": "merge_crm_companies", "input": { "keep_crm_company_id": "uuid (required)", "merge_crm_company_id": "uuid (required)" } }
{ "name": "delete_crm_company",  "input": { "crm_company_id": "uuid (required)", "confirm": "true (required)", "delete_leads": "boolean" } }
```
`delete_crm_leads`, `delete_crm_company` and `merge_crm_companies` cannot be undone: use them only on the user's explicit request, after showing them what will change.

`get_lead_context` returns the full 360° context for a lead — profile, pipeline stage, **custom fields**, **saved outreach messages**, conversation summaries, open tasks, recent LinkedIn interactions and stage history.

### CRM configuration (stages & custom fields)
Pipeline stages and custom fields are user-configurable.
```json
{ "name": "list_crm_stages",  "input": {} }
{ "name": "add_crm_stage",    "input": { "label": "string (required)", "color": "hex string" } }
{ "name": "rename_crm_stage", "input": { "key": "string (required)", "label": "string", "color": "hex string" } }
{ "name": "delete_crm_stage", "input": { "key": "string (required)", "reassign_to": "string" } }
{ "name": "list_crm_fields",  "input": {} }
{ "name": "add_crm_field",    "input": { "label": "string (required)", "type": "text|number|date|url" } }
{ "name": "delete_crm_field", "input": { "key": "string (required)" } }
```

## Example call

Request (MCP `tools/call`):

```json
{ "jsonrpc": "2.0", "id": 1, "method": "tools/call",
  "params": { "name": "get_daily_limits", "arguments": {} } }
```

Success result content (JSON inside the text part):

```json
{ "profile_active": true,
  "limits": { "connections": { "used": 0, "limit": 30, "effective_limit": 30 },
              "messages": { "used": 0, "limit": 40, "effective_limit": 40 },
              "posts": { "used": 0, "limit": 1, "effective_limit": 1, "window": "hour" } } }
```

Error result content:

```json
{ "ok": false, "code": "ACCOUNT_NOT_CONNECTED", "error": "Profile has no connected LinkedIn account." }
```

## Error codes

| Code | Meaning |
|------|---------|
| `AUTH_MISSING` / `AUTH_INVALID` / `AUTH_EXPIRED` | missing / wrong / expired key |
| `SUBSCRIPTION_REQUIRED` | trial expired or no active plan |
| `RATE_LIMITED` | too many MCP requests — slow down |
| `ACCOUNT_NOT_CONNECTED` | profile has no connected LinkedIn (call `connect_linkedin`) |
| `ACCOUNT_BLOCKED` | LinkedIn restricted the account (campaigns auto‑paused) |
| `PROFILE_INACTIVE` / `PROFILE_NOT_FOUND` / `ACCESS_DENIED` | profile / ownership |
| `DAILY_LIMIT_REACHED` / `HOURLY_LIMIT_REACHED` | quota reached |
| `OUTSIDE_ALLOWED_HOURS` | outside the account's sending window |
| `BLACKLISTED` | target company/domain blacklisted |
| `APPROVAL_REQUIRED` | queued for human approval before sending |
| `AI_CHECK_FAILED` | e-mail did not pass Salesbot's check — fix what the error says and send again |
| `NO_EMAIL` / `INVALID_EMAIL` / `DUPLICATE_OUTREACH` | no address / invalid address / already queued in that e-mail campaign |
| `SAFETY_BLOCKED` | text looks like prompt‑injection / unrequested URL |
| `REPLY_LIMIT_REACHED` | already 2 AI replies in this conversation |
| `VALIDATION_ERROR` / `NOT_FOUND` / `UPSTREAM_ERROR` | bad input / not found / upstream failure |

## Safety & responsible use

**Built-in LinkedIn algorithmic protection and daily safety thresholds.** This is a relationship tool, not a mass-mailer — it's designed to send a few highly personalized, human-approved messages, and the server actively prevents bulk abuse:

- Per‑account **daily limits** with gradual ramp‑up for new accounts; per‑hour MCP throttle; a general per‑user request rate limit.
- An independent per‑account **post limit of 1 published LinkedIn post per rolling hour**, including organization posts.
- **Human‑in‑the‑loop** approval queue for outbound actions (configurable).
- **Allowed‑hours / days** windows and randomized, human‑paced delays between actions.
- **Prompt‑injection defense:** untrusted CRM/inbox text is treated as data; outbound text is scanned before sending.
- **Inbox:** max 2 AI replies per conversation (anti‑overflow); replies are injection‑scanned.
- **Account protection:** on a LinkedIn block (provider 403) campaigns auto‑pause and the user is emailed.

## FAQ

**Which AI clients work?** Any MCP Streamable‑HTTP client — claude.ai, Cowork, Claude Code, Claude Desktop, the Claude API, Cursor, Codex and similar.

**Does the AI see my LinkedIn password?** No. Authentication happens through a hosted provider flow (white‑labeled at `auth.salesbot.cz`); the MCP server only uses an account handle.

**Can the AI send messages without me?** Only if you disable approval. By default outbound actions are queued for human approval.

**Is it safe for my LinkedIn account?** Daily/hourly limits, ramp‑up, allowed‑hours, randomized delays, and auto‑pause on a detected block are all enforced server‑side.

## License

MIT — see [LICENSE](LICENSE).
