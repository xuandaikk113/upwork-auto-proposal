# Setup Guide

## Prerequisites

1. **Docker Desktop** - Install from https://docker.com
2. **LM Studio** - Install from https://lmstudio.ai
3. **Chrome or Edge** with [Upwork Job Scraper](https://chromewebstore.google.com/detail/upwork-job-scraper/mojpfejnpifdgjjknalhghclnaifnjkg) extension
4. **Google Cloud Project** - For Sheets and Docs API access
5. **Telegram Bot** - Create via @BotFather

> **Note:** Upwork discontinued RSS feeds in August 2024. We use the free, open-source Upwork Job Scraper Chrome extension instead.

## Step 1: Clone and Configure

```bash
# Copy environment template
cp .env.example .env

# Edit .env with your values
```

## Step 2: Set Up Google Cloud

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select existing
3. Enable APIs:
   - Google Sheets API
   - Google Docs API
   - Google Drive API
4. Create Service Account:
   - IAM & Admin > Service Accounts > Create
   - Download JSON key
   - Save as `data/google-credentials.json`
5. Create Google Sheet:
   - Create new spreadsheet
   - Share with service account email (Editor access)
   - Copy Sheet ID from URL

## Step 3: Set Up Upwork Job Scraper Chrome Extension

1. Install [Upwork Job Scraper](https://chromewebstore.google.com/detail/upwork-job-scraper/mojpfejnpifdgjjknalhghclnaifnjkg) from Chrome Web Store (works on Edge too)
2. Log in to Upwork in your browser
3. Go to Find Work and set your search filters (skills, budget, etc.)
4. Save the search and copy the URL from your browser
5. Open the extension settings (click extension icon > Options)
6. Paste your saved search URL
7. Set scrape interval (recommended: 15-30 minutes)
8. The webhook URL will be configured after n8n setup (Step 8)

## Step 4: Set Up Telegram Bot

1. Message @BotFather on Telegram
2. Send `/newbot` and follow prompts
3. Copy the bot token to `.env`
4. Get your chat ID:
   - Message your bot
   - Visit: `https://api.telegram.org/bot<TOKEN>/getUpdates`
   - Find your `chat.id` in the response
5. Add chat ID to `.env`

## Step 5: Set Up LM Studio

1. Open LM Studio
2. Download a model (recommended: Llama 3.1 8B or Mistral 7B)
3. Start the local server (default port: 1234)
4. Verify it's running: `curl http://localhost:1234/v1/models`

## Step 6: Create Google Doc Template

1. Create a new Google Doc
2. Add this content with placeholders:

```
# {{titleOfSystem}}
## {{briefExplanation}}

Hi! Here's my proposed solution for {{specificRequest}}.

## Solution Architecture

{{mermaidDiagramImage}}

## Approach

{{stepByStepPlan}}

## Why Me

{{relevantExperience}}

## Timeline & Next Steps

{{timeline}}
```

3. Share the doc (Anyone with link can view)
4. Copy the document ID for use in n8n workflow

## Step 7: Start the System

```bash
# Start n8n
docker compose up -d

# Access n8n UI
open http://localhost:5678
```

## Step 8: Import Workflow & Configure Webhook

1. Open n8n UI
2. Go to Workflows > Import
3. Import `workflows/upwork-automation.json`
4. Configure credentials for:
   - Google Sheets
   - Google Docs
   - Google Drive
   - Telegram
5. Click on the **Webhook** node and copy the webhook URL
   - Example: `http://localhost:5678/webhook/upwork-jobs`
6. Go back to the Chrome extension settings
7. Paste the webhook URL in the "Webhook URL" field
8. Select **v3** payload mode
9. Activate the workflow in n8n
10. Test by clicking "Run scrape now" in the extension

## Step 9: Fill In Your Data

Edit these files with your information:
- `data/resume.md` - Your professional background
- `data/portfolio.md` - Your past projects
- `data/proposal-guidelines.md` - Your proposal style preferences

## Troubleshooting

### LM Studio not accessible from Docker
- Ensure `extra_hosts` is set in docker-compose.yml
- Check LM Studio is running and listening on 0.0.0.0
- Test: `docker exec -it n8n curl http://host.docker.internal:1234/v1/models`

### Google API errors
- Verify service account has access to the sheet
- Check credentials file path is correct
- Ensure APIs are enabled in Google Cloud Console

### No jobs appearing
- Verify your browser (Chrome/Edge) is open with the extension running
- Check webhook URL is correctly configured in extension
- Click "Run scrape now" in extension to test manually
- Check n8n execution logs for incoming webhook requests
- Verify your Upwork saved search URL returns results

### Chrome Extension Issues
- **Captcha Required**: Complete captcha manually in browser
- **Logged Out**: Re-login to Upwork in your browser
- **Auto-paused**: After 4 consecutive scrapes, the extension pauses to avoid rate limiting - this is normal behavior
