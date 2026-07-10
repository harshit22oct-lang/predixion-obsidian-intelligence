# Obsidian Intelligence — Predixion AI

> A live CRM and sales intelligence system built inside Obsidian.  
> Auto-syncs call transcripts, deal stages, and emails from external APIs into structured markdown — running as a Python automation layer across two operational vaults.

---

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Vault Structure](#vault-structure)
- [Sync Scripts](#sync-scripts)
- [Skill Library](#skill-library)
- [Getting Started](#getting-started)
- [Key Design Decisions](#key-design-decisions)
- [Maintenance Guide](#maintenance-guide)
- [Products Referenced](#products-referenced)
- [Author](#author)

---

## Overview

Predixion AI sells three B2B SaaS products — **KOLLECT**, **VOIZ**, and **LEADX** — to enterprise BFSI clients. The sales and client success motion was previously managed through a mix of manual HubSpot updates, scattered email threads, and disconnected call notes.

**Obsidian Intelligence** replaces that with a self-updating, markdown-native intelligence layer. Every morning at 7:00 AM, a Python orchestrator pulls fresh data from three external APIs (Fireflies, HubSpot, Microsoft Graph) and writes structured updates directly into Obsidian vault files. Two Dataview dashboards then surface the current state of clients and prospects without any manual input.

The result: zero data-entry overhead, no stale CRM records, and full context on every account — searchable, linkable, and readable without any SaaS subscription.

**Built by:** Harshit Kumar  
**Organisation:** Predixion AI (Vaibhav Goyal, Founder)  
**System version:** Obsidian Intelligence v1.0

---

## System Architecture

```
External APIs                     Python Layer                   Obsidian Vaults
─────────────                     ────────────                   ───────────────
Fireflies API    ──►  fireflies_sync.py  ──►  Vault1: Client Intelligence
HubSpot CRM      ──►  hubspot_sync.py   ──►  Vault2: Demand Gen Pipeline
Microsoft Graph  ──►  outlook_sync.py   ──►  (Email logs → Client files)
                       ↓
                  daily_brief.py  (Orchestrator — runs all three + generates briefing)
                       ↓
              _Daily-Brief/YYYY-MM-DD.md  (Dated brief with dormancy alerts + open tasks)
```

**Data flow:**

1. `daily_brief.py` is triggered at 7:00 AM by Windows Task Scheduler
2. Each sync script authenticates against its respective API, fetches recent activity, and matches records to vault files by email domain
3. Updates are appended to the correct `Overview.md` with timestamps — idempotency tags prevent duplicate entries on re-runs
4. The Dataview dashboards in both vaults auto-refresh to reflect the latest frontmatter state

---

## Vault Structure

```
Obsidian_Intelligence prototype/
│
├── Vault1-ClientIntelligence/           # Tracks live client accounts
│   ├── Clients/                         # One sub-folder per client
│   │   └── <Client-Name>/
│   │       └── Overview.md              # Frontmatter + call log + email log
│   ├── Dashboard.md                     # Dataview: active roster + 7-day dormancy alerts
│   ├── _Daily-Brief/                    # Auto-generated morning briefings (dated)
│   ├── _Templates/
│   │   └── Client-Template.md           # Baseline template for onboarding new clients
│   └── .obsidian/                       # Vault config, CSS theme, plugin settings
│
├── Vault2-DemandGen/                    # Tracks the full prospect pipeline
│   ├── Prospects/
│   │   ├── Active/                      # Currently engaged prospects
│   │   ├── Converted/                   # Won deals — moved here on close
│   │   └── Dormant/                     # Auto-moved after 14 days without a touchpoint
│   ├── Pipeline-Dashboard.md            # Dataview: full funnel + product-fit filters
│   ├── _Templates/
│   │   └── Prospect-Template.md         # Baseline template for new leads
│   └── .obsidian/                       # Vault config, CSS theme, plugin settings
│
├── scripts/
│   ├── fireflies_sync.py
│   ├── hubspot_sync.py
│   ├── outlook_sync.py
│   ├── daily_brief.py
│   ├── mock_data_generator.py
│   ├── setup_task_scheduler.ps1
│   ├── requirements.txt
│   ├── .env.example                     # Reference template — copy to .env
│   └── .env                             # Not committed — add real credentials here
│
└── skill-library/                       # Reusable LLM prompt templates
    ├── transcript-analysis.md
    ├── email-draft.md
    ├── kollect-deck.md
    ├── voiz-sop.md
    ├── proposal.md
    └── intern-brief.md
```

---

## Sync Scripts

All scripts live in `/scripts` and are designed to run independently or together via `daily_brief.py`.

| Script | API | What it does |
|---|---|---|
| `fireflies_sync.py` | Fireflies REST API | Fetches latest call transcripts, matches participants by email domain, appends a formatted call log entry to the correct vault file |
| `hubspot_sync.py` | HubSpot Private App API | Pulls recent deal stage updates, writes them to prospect frontmatter, appends a timestamped log entry, moves stale prospects (14+ days) to `Dormant/` |
| `outlook_sync.py` | Microsoft Graph API | Authenticates via OAuth2, pulls recent emails, appends entries to the client's Email Log section |
| `daily_brief.py` | — (Orchestrator) | Runs all three sync scripts in sequence, then generates a dated brief in `_Daily-Brief/` covering dormancy alerts and open task summaries |
| `mock_data_generator.py` | — | Creates realistic test data across both vaults so the full system can be demonstrated without live API credentials |

**Note on Outlook integration:** The Microsoft Graph OAuth2 flow is fully implemented in `outlook_sync.py` but requires an Azure AD app registration with admin consent — not configurable on a personal tenant. Mock mode simulates identical append behaviour so the end-to-end flow is demonstrable in any environment.

---

## Skill Library

The `/skill-library` folder holds reusable prompt templates, each designed as a drop-in system prompt or custom instruction for any major LLM (ChatGPT, Gemini, Claude).

| Template | Purpose |
|---|---|
| `transcript-analysis.md` | Paste a Fireflies transcript → get structured deal sentiment, objections, tech requirements, and action items |
| `email-draft.md` | Provide recipient, objective, and bullet points → get a professional BFSI-appropriate email |
| `kollect-deck.md` | Drop in raw recovery data → get an executive-ready KOLLECT performance deck outline |
| `voiz-sop.md` | Generate a complete VOIZ deployment SOP covering scope, system prompt design, UAT protocol, and go-live runbook |
| `proposal.md` | Generate a structured commercial proposal for KOLLECT, VOIZ, or LEADX with outcome-first framing |
| `intern-brief.md` | Generate a self-contained intern task brief so they can start without follow-up questions |

---

## Getting Started

### Prerequisites

- Python 3.9+
- [Obsidian](https://obsidian.md/) with the [Dataview](https://github.com/blacksmithgu/obsidian-dataview) community plugin enabled in both vaults

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/obsidian-intelligence.git
cd obsidian-intelligence
```

### 2. Install dependencies

```bash
cd scripts
pip install -r requirements.txt
```

### 3. Configure credentials

```bash
cp scripts/.env.example scripts/.env
# Open .env and fill in your Fireflies, HubSpot, and Microsoft Graph credentials
```

`.env.example` documents every required key with inline comments.

### 4. Generate mock data (no API keys needed)

```bash
python scripts/mock_data_generator.py
```

This populates both vaults with realistic client and prospect files so you can see all dashboards and dormancy alerts working immediately — no live API calls required.

### 5. Run a full sync

```bash
python scripts/daily_brief.py
```

Runs all three sync scripts in sequence and writes today's brief to:
```
Vault1-ClientIntelligence/_Daily-Brief/YYYY-MM-DD.md
```

### 6. Run individual scripts

```bash
python scripts/fireflies_sync.py
python scripts/hubspot_sync.py
python scripts/outlook_sync.py
```

### 7. Schedule daily runs (Windows)

```powershell
# Run once as Administrator
.\scripts\setup_task_scheduler.ps1
```

Registers `daily_brief.py` to execute automatically every morning at 7:00 AM via Windows Task Scheduler.

---

## Key Design Decisions

### Markdown as the database

Every client and prospect is a plain markdown file with YAML frontmatter. Dataview queries the vault like a relational database — filtering, sorting, and computing fields at read time. The files stay fully human-readable and portable without Obsidian open.

### Idempotent syncs

Every log entry written by a sync script is tagged with a hidden HTML comment (e.g., `<!-- fireflies-id:abc123 -->`). On re-run, the script checks for the tag and skips existing entries. Syncs can run as frequently as needed without producing duplicates.

### Domain-based record matching

Scripts route incoming API data to vault files by matching email domains from call participants or deal contacts against a `DOMAIN_TO_CLIENT` dictionary. This eliminates the need for a separate lookup table that would have to stay in sync with HubSpot as contacts change.

### Per-account dormancy thresholds

The 14-day (prospect) and 7-day (client) thresholds are stored in each file's YAML frontmatter as `dormancy_threshold`. `hubspot_sync.py` reads this value at runtime, so individual high-priority accounts can have custom thresholds without touching the script.

### Two-vault separation

Client Intelligence and Demand Gen run as separate Obsidian vaults intentionally. This keeps the Dataview query scope tight, allows different CSS themes and plugin configs per vault, and mirrors the organisational boundary between client success and business development.

---

## Maintenance Guide

### Adding a new client

1. Create a folder: `Vault1-ClientIntelligence/Clients/<Client-Name>/`
2. Create `Overview.md` inside it using `_Templates/Client-Template.md` as the base
3. Fill in YAML frontmatter — especially `client`, `contact_email`, `products`, and `dormancy_threshold`
4. Register the client's email domain in both `fireflies_sync.py` and `outlook_sync.py`:
   ```python
   'clientdomain.com': 'Vault1-ClientIntelligence/Clients/Client-Name/Overview.md',
   ```
5. Run `daily_brief.py` to verify the client appears in `Dashboard.md`

### Adding a new prospect

1. Create a file: `Vault2-DemandGen/Prospects/Active/<Prospect-Name>.md`
2. Use `_Templates/Prospect-Template.md` as the base structure
3. Fill in YAML frontmatter — including `hubspot_deal_id` (from HubSpot CRM URL)
4. Register the prospect's email domain in `fireflies_sync.py`:
   ```python
   'prospectdomain.com': 'Vault2-DemandGen/Prospects/Active/Prospect-Name.md',
   ```
5. The prospect will automatically appear in `Pipeline-Dashboard.md` and dormancy tracking begins on the next sync

### Troubleshooting sync failures

| Symptom | Likely cause | Fix |
|---|---|---|
| `FIREFLIES_API_KEY not found` | `.env` not loaded | Verify `.env` exists in `scripts/` and contains the key |
| `No matching domain for transcript` | Domain not mapped | Add email domain to `DOMAIN_TO_CLIENT` in `fireflies_sync.py` |
| `Target file not found` | Vault file missing | Create the folder and `Overview.md` first |
| `WARNING: marker text didn't match` | Log marker manually edited | Restore exact text: `## Call Log\n<!-- fireflies_sync.py appends here -->` |
| Outlook returns `401` | Azure AD token issue | Re-run app registration, verify admin consent was granted |
| HubSpot returns `401` | Token expired or wrong scope | Regenerate Private App token in HubSpot → Settings → Integrations |
| Slack post fails | Webhook URL expired | Regenerate webhook at api.slack.com → your app → Incoming Webhooks |

---

## Products Referenced

| Product | Description |
|---|---|
| **KOLLECT** | AI-powered collections automation for BFSI — automates outbound dialling on overdue DPD accounts |
| **VOIZ** | Conversational voice AI for inbound and outbound customer communication |
| **LEADX** | AI lead qualification and outreach automation for B2B sales pipelines |

---

## Author

**Harshit Kumar**  
Built as a working prototype for Predixion AI's internal intelligence and automation workflow.  
System: Obsidian Intelligence v1.0 | Founder: Vaibhav Goyal

---

*If you're reviewing this as part of a hiring process — the mock data generator lets you spin up the full system locally in under two minutes without any API credentials. Run `python scripts/mock_data_generator.py` and open both vaults in Obsidian.*

---

## What it does

**Vault 1 — Client Intelligence**  
Tracks active clients. Each client has an `Overview.md` with deployment status, open issues, and a running call + email log. The dashboard uses Dataview to surface clients who haven't been contacted in 7+ days, so nothing slips through.

**Vault 2 — Demand Gen Pipeline**  
Tracks prospects across stages (Cold → Engaged → Proposal → Closed). When a prospect goes 14 days without a touchpoint, the sync script automatically moves them from `Active` to `Dormant`. The pipeline dashboard gives a live view of the funnel by stage and product fit.

**Sync Scripts (`/scripts`)**  
Three Python scripts pull data from external APIs and write it back into the vault files:

| Script | What it does |
|---|---|
| `fireflies_sync.py` | Pulls latest call transcripts from Fireflies API, matches participants by email domain, appends a formatted call log entry to the right vault file |
| `hubspot_sync.py` | Fetches recent deal updates from HubSpot, updates the stage in the prospect's YAML frontmatter, appends a timestamped log entry, and moves stale prospects to Dormant |
| `outlook_sync.py` | Pulls recent emails via Microsoft Graph API and logs them to the client's Email Log section |
| `daily_brief.py` | Orchestrates all three syncs and generates a dated daily brief in `_Daily-Brief/` with dormancy alerts and open task summaries |
| `mock_data_generator.py` | Creates realistic test data across both vaults so you can demo the whole system without live API calls |

---

## Repo structure

```
Obsidian_Intelligence prototype/
├── Vault1-ClientIntelligence/       # Live client accounts
│   ├── Clients/                     # One folder per client, Overview.md in each
│   ├── Dashboard.md                 # Dataview dashboard — active roster + dormancy alerts
│   ├── _Daily-Brief/                # Auto-generated morning briefings
│   ├── _Templates/                  # Client-Template.md for new accounts
│   └── .obsidian/                   # Vault config + CSS theme
│
├── Vault2-DemandGen/                # Prospect pipeline
│   ├── Prospects/
│   │   ├── Active/                  # Live prospects
│   │   ├── Converted/               # Won deals
│   │   └── Dormant/                 # Auto-moved here after 14 days silence
│   ├── Pipeline-Dashboard.md        # Full pipeline view + product-fit filters
│   ├── _Templates/                  # Prospect-Template.md for new leads
│   └── .obsidian/                   # Vault config + CSS theme
│
├── scripts/
│   ├── fireflies_sync.py
│   ├── hubspot_sync.py
│   ├── outlook_sync.py
│   ├── daily_brief.py
│   ├── mock_data_generator.py
│   ├── requirements.txt
│   ├── .env.example                 # Copy to .env and fill in your keys
│   └── .env                         # Not committed — add your real keys here
│
└── skill-library/                   # Reusable AI prompt templates
    ├── transcript-analysis.md       # Extract objections + action items from a call
    ├── email-draft.md               # Draft client emails in Vaibhav's voice
    └── kollect-deck.md              # Generate a performance deck for KOLLECT reviews
```

---

## How to run it

**Prerequisites**
- Python 3.9+
- Obsidian with the [Dataview](https://github.com/blacksmithgu/obsidian-dataview) community plugin enabled in both vaults

**Install dependencies**
```bash
cd scripts
pip install -r requirements.txt
```

**Configure API keys**
```bash
cp scripts/.env.example scripts/.env
# Open .env and fill in your Fireflies, HubSpot, and Outlook credentials
```

**Generate mock data (no API keys needed)**
```bash
# In .env, set: MOCK_MODE=False  (keep as is)
python scripts/mock_data_generator.py
```

This creates realistic client and prospect files across both vaults so you can see the dashboards and alerts working straight away.

**Run a full sync**
```bash
python scripts/daily_brief.py
```

This runs all three sync scripts in sequence and writes today's brief to `Vault1-ClientIntelligence/_Daily-Brief/YYYY-MM-DD.md`.

**Run individual sync scripts**
```bash
python scripts/fireflies_sync.py
python scripts/hubspot_sync.py
python scripts/outlook_sync.py
```

**Test mock sync (Fireflies only)**  
Set `MOCK_MODE=True` in `.env`, then run `fireflies_sync.py`. It uses a hardcoded sample transcript and appends it to the Muthoot Finance overview so you can see the output format without an active API key.

---

## Key design decisions

**Markdown as the database**  
Each client and prospect is a plain markdown file with YAML frontmatter. This means Dataview can query across the whole vault like a database, and the files are also human-readable without Obsidian open.

**Idempotent syncs**  
Every log entry written by a sync script is tagged with a hidden HTML comment (e.g., `<!-- fireflies-id:abc123 -->`). If the same transcript or HubSpot update is synced again, the script detects the tag and skips it — no duplicate entries.

**Domain-based routing**  
The scripts match incoming data to vault files by looking at email domains from call participants or deal contacts. This avoids needing a central lookup table that has to be kept in sync with HubSpot.

**Dormancy automation**  
The 14-day (prospect) and 7-day (client) thresholds are stored in each file's frontmatter as `dormancy_threshold`. `hubspot_sync.py` reads this at runtime, so individual accounts can have custom thresholds if needed.

**Outlook integration note**  
The Microsoft Graph API flow is fully designed in `outlook_sync.py` but requires an Azure AD app registration and admin consent grant — something that can't be configured on a candidate's personal tenant. The mock mode simulates the same append behaviour so the end-to-end flow is demonstrable.

---

## Skill Library

The `/skill-library` folder holds reusable prompt templates for common tasks:

- **`/transcript-analysis`** — paste a Fireflies transcript and get structured deal sentiment, objections, tech requirements, and action items back
- **`/email-draft`** — give it a recipient, objective, and bullet points; get a professional BFSI-appropriate email in return
- **`/kollect-deck`** — drop in raw recovery data and get an executive-ready performance deck outline
- **`/voiz-sop`** — generate a complete VOIZ deployment SOP covering scope, system prompt design, UAT protocol, and go-live runbook
- **`/proposal`** — generate a structured commercial proposal for KOLLECT, VOIZ, or LEADX with outcome-first framing
- **`/intern-brief`** — generate a self-contained intern task brief so they can start without follow-up questions

These are designed to be used with any LLM (ChatGPT, Gemini, Claude) as a system prompt or custom instruction.

---

## Maintenance Guide

### How to Add a New Client

1. Create a new folder: `Vault1-ClientIntelligence/Clients/<Client-Name>/`
2. Inside it, create: `Overview.md`, `Calls/`, `Emails/`
3. Copy `_Templates/Client-Template.md` into `Overview.md` and fill in the YAML frontmatter
4. Add the client's email domain to `DOMAIN_TO_CLIENT` in both `fireflies_sync.py` and `outlook_sync.py`:
   ```python
   'clientdomain.com': 'Vault1-ClientIntelligence/Clients/Client-Name/Overview.md',
   ```
5. Run `daily_brief.py` to verify the client appears in the Dashboard.md Dataview tables

### How to Add a New Prospect

1. Create a new file: `Vault2-DemandGen/Prospects/Active/<Prospect-Name>.md`
2. Copy `_Templates/Prospect-Template.md` as the starting structure
3. Fill in the YAML frontmatter — especially `hubspot_deal_id` (find this in HubSpot CRM)
4. Add the prospect's email domain to `DOMAIN_TO_CLIENT` in `fireflies_sync.py`:
   ```python
   'prospectdomain.com': 'Vault2-DemandGen/Prospects/Active/Prospect-Name.md',
   ```
5. The prospect will now appear in `Pipeline-Dashboard.md` and dormancy detection will start automatically

### How to Troubleshoot a Failed Sync

| Symptom | Likely cause | Fix |
|---|---|---|
| `FIREFLIES_API_KEY not found` | `.env` not loaded | Check `.env` file exists in `scripts/` and has the key |
| `No matching domain for transcript` | Email domain not in mapping | Add domain to `DOMAIN_TO_CLIENT` in `fireflies_sync.py` |
| `Target file not found` | Client folder/file doesn't exist | Create the folder and `Overview.md` first |
| `WARNING: marker text didn't match` | Manually edited the call log marker | Restore the exact text: `## Call Log\n<!-- fireflies_sync.py appends here -->` |
| Outlook returns 401 | Azure AD token issue | Re-run app registration steps in Section 6.2. Check admin consent was granted. |
| HubSpot returns 401 | Token expired or wrong scope | Regenerate Private App token in HubSpot settings |
| Slack post fails | Webhook URL expired or wrong | Regenerate webhook in api.slack.com → your app → Incoming Webhooks |

### Scheduling (Windows Task Scheduler)

Run the setup script once as Administrator:
```powershell
# Right-click → Run as Administrator
.\scripts\setup_task_scheduler.ps1
```
This registers `daily_brief.py` to run every morning at 7:00 AM automatically.

---

## Products referenced

| Product | Description |
|---|---|
| KOLLECT | AI-powered collections automation for BFSI — automates outbound dialling on overdue accounts |
| VOIZ | Conversational voice AI for inbound and outbound customer communication |
| LEADX | AI lead qualification and outreach for sales pipelines |

---

*Built by Harshit Kumar as a working prototype for Predixion AI's intelligence and automation workflow.*
