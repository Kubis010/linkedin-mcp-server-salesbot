# LinkedIn MCP Server (Salesbot)

> **What is this?** `linkedin-mcp-server-salesbot` is a **Model Context Protocol (MCP) server for AI‑assisted LinkedIn relationship operations**. It lets AI assistants — **Claude Desktop, ChatGPT, Cursor** — help you research and organize professional contacts, draft deeply personalized messages **for your review and approval**, sync inbox conversations, enrich profiles, and pull web context — all under your direction. It's built for **hyper‑targeted, meaningful outreach** (find 5 ideal contacts, read their recent posts, write 5 thoughtful notes), **not bulk blasting**. Every send is gated by **human‑in‑the‑loop approval** and **enforced, server‑side daily/hourly safety thresholds** that keep your LinkedIn account within safe limits.

It runs as a Supabase Edge Function (Deno + [Hono](https://hono.dev) + [mcp-lite](https://www.npmjs.com/package/mcp-lite)) exposing the MCP **Streamable HTTP** transport. LinkedIn actions go through a third‑party LinkedIn integration provider; LinkedIn credentials are never stored by the AI.

- **Keywords:** model context protocol, mcp server, linkedin api, linkedin automation, claude desktop, cursor, ai agents, sales automation.
- **Compatible clients:** Claude Desktop, Claude API/MCP, Cursor, any MCP Streamable‑HTTP client.

## Quick facts

| | |
|---|---|
| **Endpoint** | `https://app.salesbot.cz/api/mcp` |
| **Transport** | MCP Streamable HTTP (POST + SSE) |
| **Auth header** | `x-mcp-api-key: sb_mcp_…` (a Supabase JWT in `Authorization` also works) |
| **Tool count** | 47 |
| **License** | MIT |

## How do I connect? (Claude Desktop / Cursor)

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

> **Important:** send the key in the **`x-mcp-api-key`** header, **not** `Authorization: Bearer`. The Supabase API gateway rejects unknown Bearer tokens before they reach the server.

## Authentication

- **MCP API key** (`sb_mcp_…`) — long‑lived; generated in the Salesbot app, stored only as a SHA‑256 hash. Send in `x-mcp-api-key`.
- **Supabase JWT** — a signed‑in user session token in `Authorization: Bearer`.
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
{ "name": "search_job_postings",       "input": { "keywords": "string (required)", "location": "string", "locationId": "string", "seniority": "string[]", "job_type": "string[]", "presence": "string[]", "date_posted": "number", "easy_apply": "boolean", "limit": "number 1-50" } }
{ "name": "search_web",                "input": { "query": "string (required)", "limit": "number 1-30", "country": "string (default cz)", "language": "string (default cs)" } }
{ "name": "scrape_website",            "input": { "url": "string (required)", "max_chars": "number (default 8000, max 20000)" } }
```
`search_google_xray` saves the profiles it finds into a "Google X-Ray" contact list (deduplicated) and returns their `contact_id`s — ready to enrich, add to a campaign, or push into the CRM.

`search_job_postings` searches LinkedIn job postings via the connected account (Classic search, no Recruiter needed). Returns job offers with company info — great for finding companies actively hiring for a specific role. Combine with `search_linkedin_people` to find the hiring manager.

`search_web` is a general-purpose Google search (not restricted to LinkedIn). Use Google operators like `site:jobs.cz`, `intitle:`, `OR` to search job portals, company websites, or news. Results are NOT saved to contacts — this is a research/discovery tool.

### Contacts
```json
{ "name": "get_contact_profile", "input": { "contact_id": "uuid (required)" } }
{ "name": "list_contacts",       "input": { "list_id": "uuid (required)", "limit": "number", "offset": "number" } }
{ "name": "enrich_contacts",     "input": { "contact_ids": "uuid[] (required, max 8)", "profile_id": "uuid (optional)" } }
```
`enrich_contacts` scrapes each contact's full LinkedIn profile via the connected account (headline, location, current company & position, full work history, education, skills) and saves it onto the contact. Great right after `search_google_xray`.

### Campaigns
```json
{ "name": "list_campaigns",           "input": { "status": "draft|running|paused|completed|stopped (optional)" } }
{ "name": "create_campaign",          "input": { "name": "string (required)", "profile_id": "uuid (required)", "description": "string", "daily_limit": "number", "sender_context": "string", "steps": "[{ action: 'connect'|'message'|'visit', delay_hours, use_ai, ai_prompt, ai_template, send_without_message }] (required)" } }
{ "name": "update_campaign_settings", "input": { "campaign_id": "uuid (required)", "name": "string", "description": "string", "daily_limit": "number", "sender_context": "string", "auto_approve_messages": "boolean", "status": "running|paused|draft|stopped" } }
{ "name": "start_campaign",           "input": { "campaign_id": "uuid (required)" } }
{ "name": "stop_campaign",            "input": { "campaign_id": "uuid (required)" } }
{ "name": "add_contacts_to_campaign", "input": { "campaign_id": "uuid (required)", "contact_ids": "uuid[] (required)" } }
```

### AI messaging (write → approve → send)
```json
{ "name": "generate_campaign_message", "input": { "campaign_contact_id": "uuid (required)", "step_id": "uuid (required)", "custom_instructions": "string" } }
{ "name": "list_pending_approvals",    "input": { "campaign_id": "uuid", "limit": "number" } }
{ "name": "approve_message",           "input": { "campaign_contact_id": "uuid (required)", "edited_messages": "[{step_id, message}]", "skip_gpt_check": "boolean" } }
{ "name": "reject_message",            "input": { "campaign_contact_id": "uuid (required)", "reason": "string (required)" } }
```

### Direct LinkedIn actions
```json
{ "name": "send_connection_request", "input": { "linkedin_id": "string (required)", "profile_id": "uuid (required)", "contact_id": "uuid" } }
{ "name": "send_linkedin_message",   "input": { "linkedin_id": "string (required)", "message": "string ≤5000 (required)", "profile_id": "uuid (required)" } }
{ "name": "publish_linkedin_post",   "input": { "profile_id": "uuid (required)", "text": "string ≤3000 (required)", "external_link": "string", "as_organization": "string", "auto_publish": "boolean" } }
{ "name": "get_daily_limits",        "input": { "profile_id": "uuid (optional)" } }
```

### Inbox (real‑time)
```json
{ "name": "list_inbox_chats",  "input": { "profile_id": "uuid (optional)", "limit": "number 1-50", "cursor": "string" } }
{ "name": "get_chat_messages", "input": { "chat_id": "string (required)", "profile_id": "uuid (optional)", "limit": "number 1-50", "cursor": "string" } }
{ "name": "reply_to_chat",     "input": { "chat_id": "string (required)", "message": "string ≤5000 (required)", "profile_id": "uuid (optional)" } }
{ "name": "mark_chat_read",    "input": { "chat_id": "string (required)", "profile_id": "uuid (optional)" } }
```

### CRM (pipeline, notes, tasks, message store)
The CRM is a persistent pipeline separate from contacts. A lead enters it when added to a campaign, or when any of these tools first touch it. It also acts as a durable store for generated outreach copy: save email / LinkedIn drafts and follow-ups with `save_lead_message`, read them back with `list_lead_messages` or `get_lead_context`, then send them through the right channel's own MCP (e.g. Smartlead for email) — this server never sends them itself.
```json
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
```
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
              "messages": { "used": 0, "limit": 40, "effective_limit": 40 } } }
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
| `SAFETY_BLOCKED` | text looks like prompt‑injection / unrequested URL |
| `REPLY_LIMIT_REACHED` | already 2 AI replies in this conversation |
| `SCRAPE_LIMIT_REACHED` | weekly web‑scrape quota reached |
| `VALIDATION_ERROR` / `NOT_FOUND` / `UPSTREAM_ERROR` | bad input / not found / upstream failure |

## Safety & responsible use

**Built-in LinkedIn algorithmic protection and daily safety thresholds.** This is a relationship tool, not a mass-mailer — it's designed to send a few highly personalized, human-approved messages, and the server actively prevents bulk abuse:

- Per‑account **daily limits** with gradual ramp‑up for new accounts; per‑hour MCP throttle; a general per‑user request rate limit.
- **Human‑in‑the‑loop** approval queue for outbound actions (configurable).
- **Allowed‑hours / days** windows and randomized anti‑detection delays.
- **Prompt‑injection defense:** untrusted CRM/inbox text is treated as data; outbound text is scanned before sending.
- **Inbox:** max 2 AI replies per conversation (anti‑overflow); replies are injection‑scanned.
- **Account protection:** on a LinkedIn block (provider 403) campaigns auto‑pause and the user is emailed.

## FAQ

**Which AI clients work?** Any MCP Streamable‑HTTP client — Claude Desktop, the Claude API, Cursor, and similar.

**Why `x-mcp-api-key` and not `Authorization`?** The Supabase gateway validates `Authorization` bearer tokens and rejects unknown ones; the custom header passes through untouched.

**Does the AI see my LinkedIn password?** No. Authentication happens through a hosted provider flow (white‑labeled at `auth.salesbot.cz`); the MCP server only uses an account handle.

**Can the AI send messages without me?** Only if you disable approval. By default outbound actions are queued for human approval.

**Is it safe for my LinkedIn account?** Daily/hourly limits, ramp‑up, allowed‑hours, randomized delays, and auto‑pause on a detected block are all enforced server‑side.

## Deploy

Runs on the Salesbot Supabase backend. With the [Supabase CLI](https://supabase.com/docs/guides/cli):

```bash
supabase functions deploy mcp-server --no-verify-jwt --project-ref <your-project-ref>
```

Required function secrets: `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_ANON_KEY`, the LinkedIn‑provider credentials, `CRON_SECRET`, `APP_URL`. The server does its own auth, hence `--no-verify-jwt`.

## License

MIT — see [LICENSE](LICENSE).
