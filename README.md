# Obsidian Intelligence — Predixion AI

A live CRM and sales intelligence system built inside Obsidian. It auto-syncs call transcripts from Fireflies, deal stages from HubSpot, and emails from Outlook directly into structured markdown notes. The whole thing runs as a Python automation layer on top of two Obsidian vaults.

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
