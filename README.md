# Instagram Content Automation System

An end-to-end n8n workflow that automates the Instagram content pipeline: AI drafts captions, hashtags, and image prompts from a content idea, routes them through a one-click email approval flow, and syncs everything to a live Airtable content calendar.

Built and tested end-to-end — not a theoretical template.

---

## What it does

- **Auto-drafts content** on a schedule — AI generates captions, hashtags, and image prompts from your content ideas
- **Built-in approval flow** — review and approve drafts via email before anything is marked ready
- **Centralized content calendar** — every post, status, and asset tracked in one Airtable base
- **Does not auto-publish to Instagram** — handles drafting, approval, and organization; pairs with your existing scheduler

---

## Tech stack

- [n8n](https://n8n.io) — workflow automation
- [OpenAI](https://platform.openai.com) — caption, hashtag, and image prompt generation
- [Airtable](https://airtable.com) — content calendar / data store
- Gmail — approval email delivery

---

## Architecture

**Workflow 1 — Drafting pipeline**
Schedule Trigger → Search Records (Airtable) → Generate Image Prompt (OpenAI)

→ Generate Hashtags (OpenAI) → Generate Caption (OpenAI) → Update Record (Airtable)

→ Send approval email (Gmail)

**Workflow 2 — Approval flow**
Webhook (Approve/Reject link clicked) → Switch (Approved / Rejected)

→ Update Record (Airtable)

---

## Requirements

- n8n account (free Cloud tier works)
- Airtable account
- OpenAI API key
- Gmail account (for sending/receiving approval emails)

---

## Setup

1. **Duplicate the Airtable base** (template included) — creates a `Content Calendar` table with these fields:
   - Content Idea
   - Caption
   - Hashtags
   - Image Prompt
   - Status
   - Scheduled Date
   - Post URL
   - Comments Summary

2. **Import the workflow** into n8n: `⋯` menu → Import from File → select the workflow JSON.

3. **Connect your credentials** on each node:

   | Node(s) | Credential | Get it from |
   |---|---|---|
   | Search records, Update record(s) | Airtable — Personal Access Token | airtable.com/create/tokens |
   | Generate Caption / Hashtags / Image Prompt | OpenAI API Key | platform.openai.com/api-keys |
   | Send a message | Gmail (OAuth2) | Sign in via the credential prompt |

   > Personal Access Token is recommended over OAuth for Airtable — more reliable for this workflow.

4. **Point the Airtable nodes to your base** — on each of the 5 Airtable nodes, select your duplicated base and the `Content Calendar` table (these fields are intentionally left empty so you don't accidentally point at the wrong base).

5. **Test before activating** — run the workflow manually once, confirm a record drafts correctly, approve it via the email link, and confirm the status updates in Airtable.

6. **Activate** — set your Schedule Trigger interval and toggle the workflow to Active.

Full step-by-step instructions are in `setup_guide.pdf`.

---

## Lessons learned / known issues

- **Airtable's native Trigger node is unreliable** — throws inconsistent "field does not exist" errors. Fixed by using Schedule Trigger + Search Records instead.
- **OAuth caused intermittent Airtable auth failures** — switched to Personal Access Token credentials.
- **Image generation nodes returning binary output break Airtable's URL-based attachment fields** — removed from the pipeline entirely.
- **Approval filter formula** that reliably matches the webhook record:
RECORD_ID() = '{{ $('Webhook').item.json.query.recordId }}'

---

## Troubleshooting

| Issue | Likely cause |
|---|---|
| Airtable node shows "Field does not exist" | Base or Table field is empty — re-select your duplicated base |
| Approval email never arrives | Gmail credential not connected, or workflow didn't execute — check the Executions tab |
| Workflow runs but Airtable doesn't update | Field names don't match exactly, or wrong base/table selected |

---

