# Upwork Auto-Proposal Automation

An n8n automation workflow that monitors Upwork job postings via RSS, scores them using a local LLM (LM Studio), generates personalized proposals with solution architecture diagrams, and outputs everything to Google Sheets for manual review and submission.

## Features

- **Automated Job Monitoring** - Fetches new jobs from Upwork RSS feed hourly
- **AI-Powered Scoring** - Scores jobs 1-10 based on resume/portfolio fit using local LLM
- **Smart Filtering** - Skips unverified clients and duplicate jobs
- **Proposal Generation** - Creates personalized cover letters with proven templates
- **Solution Diagrams** - Generates Mermaid architecture diagrams for each job
- **Google Docs Integration** - Creates shareable proposal documents automatically
- **Telegram Notifications** - Alerts you when high-scoring jobs appear
- **Google Sheets Dashboard** - Central hub for tracking all jobs and proposals

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Docker Host                              │
│  ┌─────────────┐                                                │
│  │    n8n      │◄──── Schedule Trigger (hourly)                 │
│  │  Container  │                                                │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│  │ RSS Feed    │────►│ LM Studio   │────►│Google Sheets│       │
│  │ (Upwork)    │     │ (Host)      │     │ API         │       │
│  └─────────────┘     └─────────────┘     └─────────────┘       │
│                             │                                    │
│                             ▼                                    │
│                      ┌─────────────┐     ┌─────────────┐       │
│                      │Google Docs  │     │ Telegram    │       │
│                      │ API         │     │ Bot API     │       │
│                      └─────────────┘     └─────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

## Prerequisites

- [Docker Desktop](https://docker.com)
- [LM Studio](https://lmstudio.ai) - Local LLM server
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

### 2. Get Upwork RSS URL

1. Log in to [Upwork](https://www.upwork.com)
2. Go to **Find Work** and set your search filters:
   - Keywords: `AI`, `Python`, `LLM`, `Machine Learning`, etc.
   - Category: Select relevant categories
   - Budget: Set minimum if desired
   - Client info: Payment verified = Yes
3. Click **Save Search** (top right)
4. After saving, click the **RSS icon** (orange wifi-like symbol) next to the search name
5. Copy the RSS URL - it will look like:
   ```
   https://www.upwork.com/ab/feed/jobs/rss?q=AI+engineer&sort=recency&paging=0%3B10&api_params=1&securityToken=xxx...
   ```

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
# Upwork RSS Feed
UPWORK_RSS_URL=https://www.upwork.com/ab/feed/jobs/rss?q=...

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

### 8. Import Workflow

1. Open n8n UI
2. Go to **Workflows > Import**
3. Import `workflows/upwork-automation.json`
4. Configure credentials for Google services and Telegram
5. Set your schedule in the Schedule Trigger node
6. Activate the workflow

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

### No Jobs Appearing

- Verify RSS feed URL is valid (test in browser)
- Check n8n execution logs for errors
- Ensure payment_verified filter matches your RSS feed

## References

- [Nick Saraev's Upwork Automation](https://n8n.io/workflows/6174-automate-personalized-upwork-proposals-with-gpt-4-google-docs-and-mermaid-diagrams/)
- [n8n Documentation](https://docs.n8n.io/)
- [LM Studio Documentation](https://lmstudio.ai/docs)

## License

MIT
