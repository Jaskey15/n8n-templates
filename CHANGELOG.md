# Changelog

## 2026-02-21 — Simplify LLM System Prompts

Replaced all detailed, business-specific system prompts with short, high-level pattern descriptions. The goal is to make these templates accessible to any business with minimal customization, rather than requiring users to find-and-replace 20+ template markers.

### What Changed

#### 6 Workflow JSON Files — System Prompts Rewritten

| Workflow | Before | After |
|----------|--------|-------|
| `initial-reachout.json` | ~4,000 chars, 20+ `{{YOUR_*}}` markers, detailed qualification flow, service area checks, examples | ~900 chars, role/goal/style/behaviors pattern |
| `nurture-replies.json` | ~4,200 chars, 20+ markers, pricing handling, price sensitivity logic, examples | ~850 chars, high-level reply handling pattern |
| `immediate-dispatcher.json` | ~2,200 chars, 12+ markers, services reference skeleton, examples | ~750 chars, short follow-up pattern |
| `active-nurture-dispatcher.json` | ~2,800 chars, 12+ markers, services reference skeleton, examples | ~800 chars, varied re-engagement pattern |
| `monthly-dispatcher.json` | ~2,900 chars, 12+ markers, message objective block, examples | ~700 chars, low-pressure cold lead pattern |
| `call-transcript-summarizer.json` | ~1,500 chars, 5 markers, business-specific summary structure | ~750 chars, generic summary structure |

#### Removed from System Prompts

- All 20 `{{YOUR_*}}` template markers (`{{YOUR_AI_AGENT_NAME}}`, `{{YOUR_BUSINESS_NAME}}`, `{{YOUR_SERVICES_SUMMARY}}`, etc.)
- `<business_information>` sections (address, hours, Instagram, website, team, sister company)
- `<service_area_check>` sections (mobile vs. shop service logic)
- `<services_reference>` skeleton blocks (service catalog placeholders)
- `<qualification_flow>` detailed checklists (priority items, information gathering, handoff triggers)
- `<examples>` with domain-specific conversation scenarios
- `<pricing_references>` verbose pricing handling rules
- `<price_sensitivity_handling>` objection handling scripts
- `<current_promos>` promotional mention instructions
- `<knowledge_boundaries>` detailed can/cannot reference lists

#### Kept in System Prompts

- Functional routing signals: `##HANDOFF##`, `##NO-REPLY##`, `##STOP##`
- Universal formatting rules (plain text only, no em-dashes, no markdown)
- Tone philosophy (casual, like texting a friend, not salesy)
- A "CUSTOMIZE BELOW" instruction telling users what to add

#### 3 READMEs Updated

- **`templates/lead-nurture-pipeline/README.md`** — Replaced 22-row `{{YOUR_*}}` marker table (Section 8) with a 4-bullet customization guide explaining what to add to the prompts
- **`templates/call-transcript-summarizer/README.md`** — Replaced marker table (Section 4) with a 3-line customization note
- **`README.md`** (root) — Updated usage steps to reflect the new approach (search for `YOUR_` placeholders, then customize prompts separately)

### What Did NOT Change

- Workflow structure, node connections, and routing logic (unchanged)
- `YOUR_*` placeholders in code nodes (employee lookup maps, custom field IDs, credential IDs, stage IDs) — these are operational config that users must set
- `YOUR_*` references in sticky notes (workflow documentation)
- User prompt (`text` field) content — only system prompts (`message` field) were simplified
- All non-LLM workflows (`pipeline-entry.json`, `pipeline-progression.json`) — no changes

### Why

The original system prompts were deeply specific to one business (boat detailing with service area checks, sister company routing, ceramic coating packages, etc.). Even with template markers, a user couldn't just find-and-replace their way to a working system — the logic around the markers was too specialized.

The new prompts describe the *pattern and philosophy* (qualify then hand off, nurture without being pushy, vary monthly approaches) and let users fill in their own business details, service catalog, and examples from scratch. This is less work than trying to adapt someone else's detailed prompt.
