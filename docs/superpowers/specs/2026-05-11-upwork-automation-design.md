# Upwork Auto-Proposal Automation

**Date:** 2026-05-11  
**Status:** Approved  
**Author:** Nguyen Vo Xuan Dai + Claude

## Overview

An n8n automation workflow that receives Upwork job postings via webhook (from the Upwork Job Scraper Chrome extension), scores them using a local LLM (LM Studio), generates personalized proposals with solution architecture diagrams, and outputs everything to Google Sheets for manual review and submission.

> **Note:** Upwork discontinued native RSS feeds on August 20, 2024. This project uses the free, open-source [Upwork Job Scraper](https://github.com/richardadonnell/Upwork-Job-Scraper) Chrome extension which scrapes job listings and sends them to a webhook.

## Goals

1. Save time on Upwork job hunting by automating the initial screening and proposal drafting
2. Score jobs based on fit with resume/portfolio (1-10 scale)
3. Generate professional proposals with visual solution diagrams
4. Notify via Telegram when high-scoring jobs appear
5. Maintain a Google Sheet as a central dashboard for job tracking

## Architecture

### System Components

```
┌──────────────────────────────────────────────────────────────────────┐
│                              Host Machine                             │
│                                                                       │
│  ┌─────────────────┐                                                 │
│  │ Browser         │                                                 │
│  │ (Chrome/Edge)   │──── Scheduled scraping (configurable)          │
│  │ + Job Scraper   │                                                 │
│  └────────┬────────┘                                                 │
│           │ HTTP POST (webhook)                                       │
│           ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      Docker Container                        │    │
│  │  ┌─────────────┐                                            │    │
│  │  │    n8n      │◄──── Webhook Trigger (receives job data)   │    │
│  │  │             │                                            │    │
│  │  └──────┬──────┘                                            │    │
│  │         │                                                    │    │
│  └─────────┼────────────────────────────────────────────────────┘    │
│            │                                                          │
│            ▼                                                          │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐            │
│  │ LM Studio   │────►│Google Sheets│     │Google Docs  │            │
│  │ (Host:1234) │     │ API         │     │ API         │            │
│  └─────────────┘     └─────────────┘     └─────────────┘            │
│                                                 │                     │
│                                                 ▼                     │
│                                          ┌─────────────┐            │
│                                          │ Telegram    │            │
│                                          │ Bot API     │            │
│                                          └─────────────┘            │
└──────────────────────────────────────────────────────────────────────┘
```

### Workflow Flow

```
Webhook Trigger (receives POST from Chrome extension)
    │
    ▼
Parse Webhook Payload (extract jobs array from v3 payload)
    │
    ▼
┌─────────────────────────────────────────┐
│           For Each Job                   │
│  ┌─────────────────────────────────┐    │
│  │ Check: Payment Verified?        │    │
│  │         └─ No ──► Skip          │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │ Check: Duplicate in Sheet?      │    │
│  │         └─ Yes ──► Skip         │    │
│  └─────────────────────────────────┘    │
│                 │                        │
│                 ▼                        │
│  Read Data Files (resume, portfolio,    │
│                   proposal-guidelines)   │
│                 │                        │
│                 ▼                        │
│  LM Studio: Score Job (1-10)            │
│  Output: score, match_reasons,          │
│          red_flags, suggested_rate,     │
│          estimated_effort, questions    │
│                 │                        │
│                 ▼                        │
│  LM Studio: Generate Mermaid Diagram    │
│                 │                        │
│                 ▼                        │
│  Google Drive: Copy Doc Template        │
│                 │                        │
│                 ▼                        │
│  Google Drive: Share (anyone w/ link)   │
│                 │                        │
│                 ▼                        │
│  Google Docs: Replace Template Vars     │
│  (title, diagram, plan, experience)     │
│                 │                        │
│                 ▼                        │
│  LM Studio: Generate Cover Letter       │
│  (with $$$ placeholder)                 │
│                 │                        │
│                 ▼                        │
│  Replace $$$ with Google Doc URL        │
│                 │                        │
│                 ▼                        │
│  Google Sheets: Append Row              │
│                 │                        │
│                 ▼                        │
│  Score >= NOTIFY_THRESHOLD?             │
│         └─ Yes ──► Telegram Notify      │
└─────────────────────────────────────────┘
```

## Data Model

### Google Sheet Columns

| Column | Type | Description |
|--------|------|-------------|
| `job_id` | String | Unique Upwork job identifier (for dedup) |
| `timestamp` | DateTime | When job was processed |
| `title` | String | Job title |
| `url` | URL | Direct link to job posting |
| `budget` | String | Posted budget or hourly rate |
| `posted_date` | Date | When job was posted |
| `proposals_submitted` | String | Competition level (Less than 5, 10-15, 20-50, 50+) |
| `client_country` | String | Client's location |
| `client_payment_verified` | Boolean | Payment method verified (only process if Yes) |
| `client_hire_rate` | String | Client's hire rate percentage |
| `client_total_spent` | String | Total spent on Upwork |
| `category` | String | Job category |
| `required_skills` | String | Listed skills |
| `score` | Integer | AI match score (1-10) |
| `match_reasons` | Text | Why this score |
| `red_flags` | Text | Potential concerns |
| `suggested_rate` | String | AI-recommended rate |
| `estimated_effort` | String | Time estimate |
| `questions_for_client` | Text | Clarifying questions |
| `proposal_doc_url` | URL | Google Doc with full proposal |
| `proposal_draft` | Text | Cover letter text |
| `status` | Dropdown | new / reviewing / applied / skipped / won |

### Input Files

```
data/
├── resume.md              # Professional background
├── portfolio.md           # Past projects and achievements
└── proposal-guidelines.md # Personal style guide for proposals
```

### Prompt Files

```
prompts/
├── scoring-prompt.md      # Job scoring (outputs JSON)
├── mermaid-prompt.md      # Solution diagram generation
└── proposal-prompt.md     # Cover letter generation
```

## Configuration

### Environment Variables

```env
# n8n Webhook (generated by n8n, configure in Chrome extension)
# N8N_WEBHOOK_URL=http://localhost:5678/webhook/upwork-jobs

# Google Sheets
GOOGLE_SHEET_ID=your-sheet-id-here
GOOGLE_CREDENTIALS_FILE=/data/google-credentials.json

# LM Studio (running on host)
LLM_HOST=http://host.docker.internal:1234
LLM_MODEL=your-model-name

# Telegram Notifications
TELEGRAM_BOT_TOKEN=your-bot-token
TELEGRAM_CHAT_ID=your-chat-id

# Notification Threshold (1-10)
NOTIFY_THRESHOLD=9

# n8n Auth
N8N_USER=admin
N8N_PASSWORD=your-secure-password
```

### Schedule

Configured directly in n8n UI (Schedule Trigger node). Default: every hour.

## Google Doc Template

Create a Google Doc with these placeholders:

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

### Mermaid Diagram Rendering

The Mermaid code needs to be converted to an image before inserting into Google Docs. Options:

1. **Mermaid.ink API (Recommended):** Free service that renders Mermaid code to PNG via URL
   - URL format: `https://mermaid.ink/img/{base64_encoded_mermaid_code}`
   - Insert as image URL in Google Docs

2. **Alternative:** Include Mermaid code as text block with a link to https://mermaid.live for client to visualize

The workflow will use option 1 (Mermaid.ink) for automatic image generation.

### LLM API Format

LM Studio exposes an **OpenAI-compatible API** at `http://host.docker.internal:1234/v1/chat/completions`. The n8n workflow will use HTTP Request nodes with:
- Method: POST
- Headers: `Content-Type: application/json`
- Body format matching OpenAI Chat Completions API

## Proposal Strategy

Based on Nick Saraev's proven template approach:

1. **Speed** - Deliver proposals before competitors finish reading
2. **Perceived customization** - Reference specific job details
3. **Value demonstration** - Include visual diagram showing understanding
4. **Spartan tone** - Casual, concise, no flowery language or emojis

**Cover Letter Template:**
```
Hi [name], I do {relevant_thing} all the time.
So confident I'm the right fit that I created a solution diagram + proposal: $$$

About me: {brief_relevant_background}

{specific_value_you_bring}

Happy to discuss—just respond to this proposal.

Thank you!
```

## Docker Setup

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./data:/data:ro
      - ./prompts:/prompts:ro
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  n8n_data:
```

## Browser Extension Setup

### Upwork Job Scraper

Install the [Upwork Job Scraper](https://chromewebstore.google.com/detail/upwork-job-scraper/mojpfejnpifdgjjknalhghclnaifnjkg) extension (free, open-source). Works on Chrome, Edge, or any Chromium-based browser.

**Configuration:**
1. Set your Upwork saved search URL in the extension
2. Configure scrape schedule (recommended: every 15-30 minutes)
3. Set webhook URL to your n8n webhook endpoint
4. Select payload mode: **v3** (default, recommended for n8n)

### Webhook Payload Format (v3)

```json
{
  "status": "success",
  "targetName": "AI Engineer Jobs",
  "jobs": [
    {
      "uid": "~01abc123...",
      "title": "AI Engineer for LLM Project",
      "url": "https://www.upwork.com/jobs/~01abc123...",
      "datePosted": "2026-05-12T10:30:00Z",
      "postedAtMs": 1747044600000,
      "description": "Looking for an AI engineer...",
      "jobType": "Fixed price",
      "budget": "$500",
      "experienceLevel": "Expert",
      "skills": ["Python", "LLM", "PyTorch"],
      "paymentVerified": true,
      "clientRating": 4.8,
      "clientTotalSpent": "$50,000+",
      "proposals": "10 to 15"
    }
  ],
  "timestamp": "2026-05-12T10:35:00Z"
}
```

**Status values:**
- `success` - Jobs found and included
- `captcha_required` - Manual intervention needed
- `logged_out` - Re-login to Upwork required
- `error` - General error occurred

> **Note:** `status: 'no_results'` is NOT sent to webhook (only browser notification).

## Filtering Rules

Jobs are **skipped** if:
1. Client payment is NOT verified
2. Job ID already exists in Google Sheet (duplicate)

## Notifications

Telegram message sent when `score >= NOTIFY_THRESHOLD` containing:
- Job title
- Score
- Budget
- URL
- Brief match summary

## Success Criteria

1. Workflow runs hourly without errors
2. Jobs scored accurately based on resume/portfolio fit
3. Proposals are personalized and reference job details
4. Diagrams are relevant to the proposed solution
5. No duplicate jobs in sheet
6. Telegram notifications arrive for high-scoring jobs

## Out of Scope

- Automatic proposal submission (manual review required)
- Multiple RSS feeds (single feed for now)
- Proposal A/B testing
- Analytics/reporting dashboard

## References

- [Nick Saraev's n8n Upwork Automation](https://n8n.io/workflows/6174-automate-personalized-upwork-proposals-with-gpt-4-google-docs-and-mermaid-diagrams/)
- [Upwork Job Scraper - GitHub](https://github.com/richardadonnell/Upwork-Job-Scraper)
- [Upwork Job Scraper - Chrome Web Store](https://chromewebstore.google.com/detail/upwork-job-scraper/mojpfejnpifdgjjknalhghclnaifnjkg)
- Sample workflow: `sample-n8n-flow.json`

## Appendix: Why Not RSS?

Upwork officially discontinued RSS feeds on **August 20, 2024**. The Upwork Job Scraper Chrome extension is the recommended free alternative that provides:
- Webhook integration for automation workflows
- Rich job data including client verification status
- Configurable scrape schedules
- Open-source (GPL-3.0) with active maintenance
