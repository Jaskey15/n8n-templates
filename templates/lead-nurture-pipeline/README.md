# Lead Nurture Pipeline

A 7-workflow system that automatically nurtures leads via SMS using AI-generated messages. Leads progress through three stages (Immediate, Active Nurture, Monthly) based on time, with GPT handling personalized outreach and reply management.

## Architecture

```
[Form/Inquiry] → [Initial Reachout] → [Pipeline Entry]
                                            ↓
                              ┌─────── Immediate ───────┐
                              │   (0-1 days, high freq)  │
                              │   Immediate Dispatcher   │
                              └───────────┬──────────────┘
                    Pipeline Progression (daily, moves after 1+ day)
                              ┌───────────┴──────────────┐
                              │    Active Nurture         │
                              │  (days 3, 7, 15)          │
                              │  Active Nurture Dispatcher│
                              └───────────┬──────────────┘
                    Pipeline Progression (daily, moves after 16+ days)
                              ┌───────────┴──────────────┐
                              │       Monthly             │
                              │  (days 30, 60, 90...360)  │
                              │    Monthly Dispatcher     │
                              └───────────┬──────────────┘
                    Pipeline Progression (daily, moves after 362+ days)
                              ┌───────────┴──────────────┐
                              │    Email Nurture (exit)   │
                              └──────────────────────────┘

At ANY stage, customer replies trigger:
  [Nurture Replies] → AI responds or → ##HANDOFF## → Sales rep takes over
```

## Workflows

| File | Trigger | Purpose |
|------|---------|---------|
| `pipeline-entry.json` | Webhook | Records entry date, checks DND status |
| `pipeline-progression.json` | Daily schedule (6:30 AM) | Moves opportunities between stages based on `entry_date` |
| `immediate-dispatcher.json` | Every 30 min (M-F business hours) | Sends warm follow-ups to fresh leads |
| `active-nurture-dispatcher.json` | Every 30 min (business hours) | Sends messages on days 3, 7, 15 |
| `monthly-dispatcher.json` | Every 30 min (business hours) | Sends monthly touchpoints (30-360 days) |
| `nurture-replies.json` | Webhook (customer reply) | AI-generated replies, buying signal detection, sales handoff |
| `initial-reachout.json` | Webhook (form submission) | AI qualification of new inquiries |

## Prerequisites

- [n8n](https://n8n.io) instance (self-hosted or cloud)
- [GoHighLevel](https://www.gohighlevel.com/) account with API access
- [OpenAI](https://platform.openai.com/) API key
- GHL pipeline with at least 4 stages
- GHL custom field for entry date tracking

## Setup

### 1. Create GHL Pipeline Stages

Create a pipeline in GoHighLevel with these stages:

| Stage | Purpose |
|-------|---------|
| Immediate | Fresh leads (0-1 days) |
| Active Nurture | Warm leads (days 2-16) |
| Monthly | Long-term nurture (days 17-362) |
| Email Nurture | Exit stage (SMS stops) |

Note the stage IDs from the GHL API or URL.

### 2. Create GHL Custom Fields

Create these custom fields on opportunities in GoHighLevel:

| Field Name | Type | Purpose |
|------------|------|---------|
| `entry_date` | Date | Tracks when the lead entered the pipeline (drives all timing) |
| `last_message_sent` | Date | Deduplication - prevents sending multiple messages per day |

Note their field IDs from the GHL API.

### 3. Set Up n8n Credentials

Create these credentials in n8n:

| Credential | Type | For |
|------------|------|-----|
| GHL API Key | HTTP Header Auth | All GHL API calls |
| OpenAI API Key | OpenAI | AI message generation |

### 4. Set Up n8n Variables

Create these n8n environment variables:

| Variable | Value |
|----------|-------|
| `GHL_LOCATION_ID` | Your GHL location ID |
| `CURRENT_PROMO` | Current promotional offer text (optional, leave empty if none) |

### 5. Create n8n DataTable

Create a DataTable called `nurture_evals` in n8n. This stores evaluation data from dispatcher runs. Note the DataTable ID and use it for the `YOUR_DATATABLE_ID` placeholder.

### 6. Import Workflows

Import all 7 `.json` files from the `workflows/` directory into n8n.

### 7. Replace Placeholders

Search each workflow for these placeholders and replace with your values:

| Placeholder | Where to Find Your Value |
|-------------|-------------------------|
| `YOUR_GHL_CREDENTIAL_ID` | n8n credentials page |
| `YOUR_OPENAI_CREDENTIAL_ID` | n8n credentials page |
| `YOUR_LOCATION_ID` | GHL Settings > Business Info |
| `YOUR_IMMEDIATE_STAGE_ID` | GHL pipeline stage URL/API |
| `YOUR_ACTIVE_NURTURE_STAGE_ID` | GHL pipeline stage URL/API |
| `YOUR_MONTHLY_STAGE_ID` | GHL pipeline stage URL/API |
| `YOUR_EMAIL_NURTURE_STAGE_ID` | GHL pipeline stage URL/API |
| `YOUR_HANDOFF_STAGE_ID` | GHL pipeline stage URL/API |
| `YOUR_DND_PIPELINE_STAGE_ID` | GHL pipeline stage for DND contacts |
| `YOUR_ENTRY_DATE_FIELD_ID` | GHL custom fields API (`entry_date` field) |
| `YOUR_LAST_SENT_FIELD_ID` | GHL custom fields API (`last_message_sent` field) |
| `YOUR_NOTES_FIELD_ID` | GHL custom fields API (`message`/notes field) |
| `YOUR_SERVICES_INTERESTED_FIELD_ID` | GHL custom fields API (`services_interested` field) |
| `YOUR_DESIRED_PACKAGES_FIELD_ID` | GHL custom fields API (`desired_packages` field) |
| `YOUR_SERVICES_CHECK_FIELD_ID` | GHL custom fields API (`services_interested_check` field) |
| `YOUR_EMPLOYEE_1_USER_ID` ... `YOUR_EMPLOYEE_4_USER_ID` | GHL team member user IDs |
| `YOUR_TAG_1_ID` | GHL tag ID used in dispatcher routing |
| `YOUR_TAG_2_ID` | GHL tag ID used in dispatcher routing |
| `YOUR_N8N_PROJECT_ID` | n8n project ID (Settings > General) |
| `YOUR_DATATABLE_ID` | n8n DataTable ID for `nurture_evals` table |
| `YOUR_TIMEZONE` | Your IANA timezone (e.g., `America/New_York`) |
| `YOUR_WEBHOOK_ID` | Auto-generated by n8n on import |
| `YOUR_WEBHOOK_PATH` | Choose your own webhook paths |
| `TEMPLATE_WORKFLOW_ID` | Auto-assigned by n8n on import |

### 8. Customize LLM Prompts

Each dispatcher and reply workflow contains a system prompt describing the AI's role, goals, style, and behavioral rules. These prompts are intentionally high-level, documenting the *pattern* (qualify then hand off, nurture without being pushy, etc.) rather than prescribing business-specific details.

To make them yours:

1. **Add your business context** - business name, team members, location, hours, and any relevant details
2. **Add your service catalog** - list your services, packages, and pricing so the AI can reference them
3. **Add conversation examples** - include 3-5 examples showing how you want the AI to handle common scenarios (new lead, pricing question, decline, handoff)
4. **Add business-specific rules** - service area restrictions, excluded services, sister companies, etc.

The prompts preserve key behavioral signals that the workflow routing depends on:
- `##HANDOFF##` - triggers sales rep handoff flow
- `##NO-REPLY##` - signals conversation end (no SMS sent)
- `##STOP##` - signals excluded service (initial reachout only)

### 9. Configure GHL Webhooks

Set up GHL workflow triggers to fire webhooks to your n8n instance:

- **Pipeline Entry:** Fire when a contact enters your nurture pipeline
- **Nurture Replies:** Fire when an inbound SMS is received from a contact in the pipeline
- **Initial Reachout:** Fire when a new form submission or inquiry comes in

Each webhook payload should include `contact_id` and relevant `customData` fields:

| Field | Used By |
|-------|---------|
| `customData.first_name` | All dispatchers, initial reachout |
| `customData.asset_description` | Initial reachout (customer's asset details) |
| `customData.services_interested` | Dispatchers (service interest context) |
| `customData.desired_packages` | Dispatchers (package preferences) |
| `customData.additional_info` | Initial reachout |

### 10. Activate Workflows

Activate all 7 workflows in n8n.

## Timing Reference

| Stage | Entry | Schedule | Message Days | Exit |
|-------|-------|----------|-------------|------|
| Immediate | Day 0 | Every 30 min, M-F business hours | Fresh lead follow-ups | Day 1 |
| Active Nurture | Day 1 | Every 30 min, 7 days/week business hours | Days 3, 7, 15 | Day 16 |
| Monthly | Day 16 | Every 30 min, 7 days/week business hours | Days 30, 60, 90, 120, 150, 180, 210, 240, 270, 300, 330, 360 | Day 362 |
| Email Nurture | Day 362 | -- | (SMS stops) | -- |

## Key Design Decisions

- **Single `entry_date` field** drives all timing. Set once on entry, never reset as leads move between stages.
- **Deduplication** via `last_message_sent` field prevents duplicate messages within the same day.
- **`##HANDOFF##` / `##NO-REPLY##` markers** in GPT output signal routing decisions without exposing system logic to the customer.
- **Business hours enforcement** ensures messages only send during appropriate times.
- **Rate limiting** (max 10 per dispatcher run) prevents API throttling.
