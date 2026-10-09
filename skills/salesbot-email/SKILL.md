---
name: salesbot-email
description: Send personal e-mails through Salesbot from the user's own mailbox (Outlook or IMAP/SMTP), respecting consent rules, and react when Salesbot's check refuses an e-mail. Use when the user asks to e-mail a lead or contact from Salesbot.
---

# E-mail outreach with Salesbot

## Consent first
In the Czech Republic a commercial e-mail needs the recipient's prior consent unless they are an existing customer (§ 7 of Act 480/2004 Coll.). Before e-mailing someone new, ask the user whether the person agreed to it, replied, asked for it or is a customer. For first contact with new people use LinkedIn instead. Never e-mail people on the do-not-contact list (Salesbot refuses with `BLACKLISTED`).

## Pick the mailbox
`list_email_integrations` shows the user's own mailboxes (Outlook or IMAP/SMTP). Use `provider: "mailbox"` and `provider_campaign_id` = its `mailbox_id`. Salesbot sends within the sending hours, a few minutes apart, capped per day, and the e-mail can be withdrawn with `cancel_email` until it is sent. Smartlead and Instantly are no longer supported.

## Write and send
`send_email` with `contact_id` or `crm_lead_id`, `subject` and `body` (plain text). Placeholders such as `{{first_name}}`, `{{company}}` and `{{oslovení}}` are filled per person. If the person has no address, pass `email`; it is saved to the contact.

Write each e-mail for that person. Do **not** sign the body when the mailbox has its own signature — Salesbot adds it, and a signed body would end up signed twice.

## Salesbot checks every e-mail
Before anything is sent, Salesbot checks the greeting (right name, and pane/paní by the contact's gender), the company, leftover placeholders such as `{{company}}` or `(název firmy)`, Czech grammar and a doubled signature.

- `AI_CHECK_FAILED` — nothing was sent. Fix **exactly** what the error says (keep the rest of the e-mail) and call `send_email` again once.
- If the second attempt also fails, the e-mail goes to the user's manual approval with the note. Tell the user; do not try a third time.
- `pending_approval` — the e-mail waits for the user in Salesbot (CRM → E-maily). Never say it was sent.

Check progress later with `list_email_outreach` or `get_email_status`.
