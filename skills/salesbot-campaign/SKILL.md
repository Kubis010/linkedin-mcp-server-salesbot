---
name: salesbot-campaign
description: Set up and run a Salesbot LinkedIn campaign — steps, sample message, AI personalisation and approval. Use when the user wants to create, fill, start or check a LinkedIn campaign in Salesbot.
---

# Salesbot LinkedIn campaigns

A campaign sends a sequence of steps (visit, connection request, messages) to its contacts, paced within the account's daily limits and sending hours.

## Create
`create_campaign` with `name`, `profile_id` (the LinkedIn profile that sends) and `steps`. The campaign is saved as a draft and sends nothing yet.

For each message step decide `message_mode` with the user:
- `template` — the user's sample message (`ai_template`) is sent as written; AI only fills the fields such as `{oslovení}`, `{{first_name}}`, `{{company}}` (shortened, e.g. "Constellium" instead of "Constellium Extrusions Děčín s.r.o."). Choose this when the user already has a message they like.
- `creative` — AI writes its own version for every contact, following `ai_prompt`; the sample is only a style example.

The campaign's `sender_context` ("About me") is NOT used automatically. Write `{{o_mne}}` in `ai_prompt` where the AI should use it. Links in the prompt or the sample are kept exactly where they are written.

## Fill and prepare
1. `add_contacts_to_campaign` with saved `contact_id`s.
2. `prepare_campaign_messages` (max 10 per call; repeat while `remaining_pending` > 0).
3. `list_pending_approvals` — show the drafts to the user. `approve_message` (optionally with `edited_messages`) or `reject_message` with a reason.

If the user turned on AI approval (`auto_approve_messages` in `update_campaign_settings`), Salesbot checks each draft itself — the contact's name and gender, company, leftover placeholders, signature and grammar — and approves only what passes; the rest waits for the user.

## Start and watch
`start_campaign` starts sending approved contacts. `list_campaign_queue` shows what is planned, `stop_campaign` pauses it. Remove a person with `exclude_contacts_from_campaign`.

Never start a campaign the user has not asked to start, and never approve messages on their behalf unless they asked you to.
