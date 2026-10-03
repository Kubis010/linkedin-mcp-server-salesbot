---
name: salesbot-email
description: Send personal outreach e-mails through Salesbot — Smartlead, Instantly or the user's own mailbox — including how to react when Salesbot's check refuses an e-mail. Use when the user asks to e-mail a lead or contact from Salesbot.
---

# E-mail outreach with Salesbot

## Pick the channel
`list_email_integrations` shows what is connected:
- **Smartlead / Instantly** — `list_email_campaigns`, then use the campaign id as `provider_campaign_id`. The provider campaign's sequence must use `{{email_subject}}` and `{{email_body}}` so your text is what gets sent. The provider then sends on its own schedule and the e-mail cannot be withdrawn from Salesbot.
- **Own mailbox** (Outlook / IMAP) — `provider: "mailbox"`, `provider_campaign_id` = its `mailbox_id`. Salesbot sends it within the sending hours, a few minutes apart, capped per day. It can be withdrawn with `cancel_email` until it is sent.

## Write and send
`send_email` with `contact_id` or `crm_lead_id`, `subject` and `body` (plain text). Placeholders such as `{{first_name}}`, `{{company}}` and `{{oslovení}}` are filled per person. If the person has no address, pass `email`; it is saved to the contact.

Write each e-mail for that person. Do **not** sign the body when the mailbox has its own signature — Salesbot adds it, and a signed body would end up signed twice.

## Salesbot checks every e-mail
Before anything is sent, Salesbot checks the greeting (right name, and pane/paní by the contact's gender), the company, leftover placeholders such as `{{company}}` or `(název firmy)`, Czech grammar and a doubled signature.

- `AI_CHECK_FAILED` — nothing was sent. Fix **exactly** what the error says (keep the rest of the e-mail) and call `send_email` again once.
- If the second attempt also fails, the e-mail goes to the user's manual approval with the note. Tell the user; do not try a third time.
- `pending_approval` — the e-mail waits for the user in Salesbot (CRM → E-maily). Never say it was sent.

Check progress later with `list_email_outreach` or `get_email_status`.
