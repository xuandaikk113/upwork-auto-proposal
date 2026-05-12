# Upwork Auto-Proposal Automation

An n8n automation workflow that receives Upwork job postings via webhook (from Chrome extension), scores them using a local LLM (LM Studio), generates personalized proposals with solution architecture diagrams, and outputs everything to Google Sheets for manual review and submission.

> **Note:** Upwork discontinued RSS feeds in August 2024. This project uses the free, open-source [Upwork Job Scraper](https://github.com/richardadonnell/Upwork-Job-Scraper) Chrome extension.

## Features

- **Automated Job Monitoring** - Receives jobs via webhook from Chrome extension
- **AI-Powered Scoring** - Scores jobs 1-10 based on resume/portfolio fit using local LLM
- **Smart Filtering** - Skips unverified clients and duplicate jobs
- **Proposal Generation** - Creates personalized cover letters with proven templates
- **Solution Diagrams** - Generates Mermaid architecture diagrams for each job
- **Google Docs Integration** - Creates shareable proposal documents automatically
- **Telegram Notifications** - Alerts you when high-scoring jobs appear
- **Google Sheets Dashboard** - Central hub for tracking all jobs and proposals

## Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                              Host Machine                             │
│                                                                       │
│  ┌─────────────────┐                                                 │
│  │ Chrome Browser  │                                                 │
│  │ + Upwork Job    │──── Scheduled scraping (configurable)          │
│  │   Scraper Ext   │                                                 │
│  └────────┬────────┘                                                 │
│           │ HTTP POST (webhook)                                       │
│           ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      Docker Container                        │    │
│  │  ┌─────────────┐                                            │    │
│  │  │    n8n      │◄──── Webhook Trigger                       │    │
│  │  └──────┬──────┘                                            │    │
│  └─────────┼────────────────────────────────────────────────────┘    │
│            ▼                                                          │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐            │
│  │ LM Studio   │────►│Google Sheets│     │ Telegram    │            │
│  │ (Host:1234) │     │ + Docs API  │     │ Bot API     │            │
│  └─────────────┘     └─────────────┘     └─────────────┘            │
└──────────────────────────────────────────────────────────────────────┘
```

## Prerequisites

- [Docker Desktop](https://docker.com)
- [LM Studio](https://lmstudio.ai) - Local LLM server
- [Upwork Job Scraper](https://chromewebstore.google.com/detail/upwork-job-scraper/mojpfejnpifdgjjknalhghclnaifnjkg) browser extension (works on Chrome, Edge, or any Chromium-based browser)
- Google Cloud Project with APIs enabled
- Telegram Bot (via @BotFather)
- Upwork account with saved search

## Quick Start

### 1. Clone and Configure

```bash
git clone <repository-url>
cd upwork-auto-proposal
cp .env.example .env
```

### 2. Install Upwork Job Scraper Chrome Extension

> **Note:** Upwork discontinued RSS feeds in August 2024. We use the free, open-source Upwork Job Scraper extension instead.

1. Install [Upwork Job Scraper](https://chromewebstore.google.com/detail/upwork-job-scraper/mojpfejnpifdgjjknalhghclnaifnjkg) from Chrome Web Store (works on Edge too)
2. Log in to [Upwork](https://www.upwork.com)
3. Go to **Find Work** and set your search filters:
   - Keywords: `AI`, `Python`, `LLM`, `Machine Learning`, etc.
   - Category: Select relevant categories
   - Budget: Set minimum if desired
   - Client info: Payment verified = Yes
4. Click **Save Search** and copy the URL from your browser
5. Open the extension settings and paste your saved search URL
6. Configure scrape interval (recommended: 15-30 minutes)
7. The webhook URL will be configured after n8n setup (Step 8)

### 3. Setup Google Cloud

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select existing one
3. Enable these APIs:
   - Google Sheets API
   - Google Docs API
   - Google Drive API
4. Create a Service Account:
   ```
   IAM & Admin > Service Accounts > Create Service Account
   ```
5. Download the JSON key and save as `data/google-credentials.json`
6. Create a new Google Sheet
7. Share the sheet with the service account email (Editor access)
8. Copy the Sheet ID from URL:
   ```
   https://docs.google.com/spreadsheets/d/[SHEET_ID_HERE]/edit
   ```

### 4. Setup Telegram Bot

1. Open Telegram and find **@BotFather**
2. Send `/newbot` and follow the prompts
3. Copy the **Bot Token** (format: `123456789:ABCdefGHI...`)
4. Send any message to your new bot
5. Get your Chat ID by visiting:
   ```
   https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
   ```
   Look for `"chat":{"id":123456789}` in the response

### 5. Setup LM Studio

1. Download and install [LM Studio](https://lmstudio.ai)
2. Download a model (recommended: **Llama 3.1 8B** or **Mistral 7B Instruct**)
3. Go to the **Local Server** tab
4. Load your model and click **Start Server**
5. Verify it's running: Open browser and visit `http://localhost:1234/v1/models`

### 6. Configure Environment

Edit `.env` with your values:

```env
# Webhook URL (will be generated by n8n - see Step 8)
# N8N_WEBHOOK_URL=http://localhost:5678/webhook/upwork-jobs

# Google Sheets
GOOGLE_SHEET_ID=your-sheet-id
GOOGLE_CREDENTIALS_FILE=/data/google-credentials.json

# LM Studio (running on host)
LLM_HOST=http://host.docker.internal:1234
LLM_MODEL=your-model-name

# Telegram Notifications
TELEGRAM_BOT_TOKEN=your-bot-token
TELEGRAM_CHAT_ID=your-chat-id

# Notification Threshold (1-10)
NOTIFY_THRESHOLD=9

# n8n Authentication
N8N_USER=admin
N8N_PASSWORD=your-secure-password
```

### 7. Start n8n

```bash
docker compose up -d
```

Access n8n UI at: **http://localhost:5678**

Login with credentials from `.env`

### 8. Import Workflow & Configure Webhook

1. Open n8n UI
2. Go to **Workflows > Import**
3. Import `workflows/upwork-automation.json`
4. Configure credentials for Google services and Telegram
5. Click on the **Webhook** node and copy the webhook URL (e.g., `http://localhost:5678/webhook/upwork-jobs`)
6. Open the **Upwork Job Scraper** Chrome extension settings
7. Paste the webhook URL and select **v3** payload mode
8. Activate the workflow in n8n
9. Click "Run scrape now" in the extension to test

## Project Structure

```
upwork-auto-proposal/
├── docker-compose.yml          # n8n service configuration
├── .env.example                 # Environment variables template
├── .env                         # Your configuration (gitignored)
├── data/
│   ├── resume.md               # Your professional background
│   ├── portfolio.md            # Your past projects
│   ├── proposal-guidelines.md  # Your proposal style guide
│   └── google-credentials.json # Google service account key
├── prompts/
│   ├── scoring-prompt.md       # Job scoring prompt template
│   ├── mermaid-prompt.md       # Diagram generation prompt
│   └── proposal-prompt.md      # Cover letter generation prompt
├── workflows/
│   └── upwork-automation.json  # Exportable n8n workflow
└── docs/
    ├── setup.md                # Detailed setup instructions
    └── google-doc-template.md  # Google Doc template guide
```

## Customization

### Update Your Profile

Edit these files to match your background:
- `data/resume.md` - Your professional experience
- `data/portfolio.md` - Your highlight projects
- `data/proposal-guidelines.md` - Your writing style preferences

### Adjust Prompts

Modify prompts in `prompts/` folder to tune AI behavior:
- `scoring-prompt.md` - How jobs are scored
- `mermaid-prompt.md` - How diagrams are generated
- `proposal-prompt.md` - How cover letters are written

### Change Schedule

The schedule is configured in n8n UI (Schedule Trigger node). Default: every hour.

## Google Sheet Columns

| Column | Description |
|--------|-------------|
| `job_id` | Unique Upwork job identifier |
| `timestamp` | When job was processed |
| `title` | Job title |
| `url` | Link to job posting |
| `budget` | Posted budget/rate |
| `posted_date` | When job was posted |
| `proposals_submitted` | Competition level |
| `client_country` | Client location |
| `client_payment_verified` | Payment status |
| `client_hire_rate` | Client's hire rate |
| `client_total_spent` | Total spent on Upwork |
| `score` | AI match score (1-10) |
| `match_reasons` | Why this score |
| `red_flags` | Potential concerns |
| `suggested_rate` | Recommended rate |
| `estimated_effort` | Time estimate |
| `proposal_doc_url` | Google Doc link |
| `proposal_draft` | Cover letter text |
| `status` | Tracking status (dropdown) |

> **Row Highlighting:** Jobs with `score >= 7` are automatically highlighted with a light green background for quick identification.

## Troubleshooting

### LM Studio not accessible from Docker

- Ensure `extra_hosts` is configured in `docker-compose.yml`
- Check LM Studio is running and listening on `0.0.0.0:1234`
- Test connectivity:
  ```bash
  docker exec -it upwork-auto-proposal-n8n-1 curl http://host.docker.internal:1234/v1/models
  ```

### Google API Errors

- Verify service account has Editor access to the sheet
- Check credentials file path is correct
- Ensure all 3 APIs are enabled in Google Cloud Console

### Chrome Extension Issues

- **Captcha Required**: Upwork detected automation - complete captcha manually in browser
- **Logged Out**: Re-login to Upwork in your browser
- **Webhook not receiving**: Ensure n8n is running and webhook URL is correct
- **Extension paused**: After 4 consecutive scrapes, extension auto-pauses to avoid rate limiting - this is normal

### No Jobs Appearing

- Verify the browser extension is running and your browser (Chrome/Edge) is open
- Check that the webhook URL is correctly configured in the extension
- Test webhook by clicking "Run scrape now" in extension settings
- Check n8n execution logs for incoming webhook requests
- Verify your Upwork saved search URL is valid

## References

- [Nick Saraev's Upwork Automation](https://n8n.io/workflows/6174-automate-personalized-upwork-proposals-with-gpt-4-google-docs-and-mermaid-diagrams/)
- [Upwork Job Scraper - GitHub](https://github.com/richardadonnell/Upwork-Job-Scraper)
- [Upwork Job Scraper - Chrome Web Store](https://chromewebstore.google.com/detail/upwork-job-scraper/mojpfejnpifdgjjknalhghclnaifnjkg)
- [n8n Documentation](https://docs.n8n.io/)
- [LM Studio Documentation](https://lmstudio.ai/docs)

## License

MIT
