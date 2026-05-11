# CLAUDE.md - AI Agent Guidelines

This file contains instructions for AI agents (Claude, Copilot, Cursor, etc.) working on this project.

## Critical Rules

### 1. No Auto-Commit

**NEVER automatically commit changes.** Always wait for user review and manual commit.

- Stage changes and show diff if needed
- Explain what was changed
- Let the user decide when to commit
- User will run `git commit` manually

### 2. Environment Variables

**Always update `.env.example` when adding or modifying environment variables.**

- If you add a new env var in code, add it to `.env.example` with a placeholder value
- If you rename an env var, update `.env.example` accordingly
- If you remove an env var, remove it from `.env.example`
- Include comments explaining what each variable does

### 3. English Only for All Content

**All project content must be written in English** to maintain professionalism:

- Code (variables, functions, comments)
- Documentation (README, docs/, markdown files)
- Commit messages
- Comments in configuration files
- Prompt templates
- Error messages

**Note:** The user may communicate in Vietnamese or English during development. However, all written artifacts (code, docs, markdown, etc.) must be in English.

## Project Context

This is an n8n automation project for Upwork job proposal automation:

- **Stack:** n8n (Docker), LM Studio (local LLM), Google APIs, Telegram
- **Purpose:** Monitor Upwork jobs, score them with AI, generate proposals
- **Input:** RSS feed, resume/portfolio markdown files
- **Output:** Google Sheets dashboard, Google Docs proposals, Telegram notifications

## Key Files

- `data/resume.md` - User's professional background (feed to LLM)
- `data/portfolio.md` - User's project portfolio (feed to LLM)
- `data/proposal-guidelines.md` - Proposal writing style guide
- `prompts/*.md` - LLM prompt templates
- `workflows/*.json` - n8n workflow exports
- `docker-compose.yml` - Container configuration

## Development Guidelines

1. **Test locally** before suggesting changes to workflow
2. **Preserve exact content** in resume.md and portfolio.md - these are from user's CV
3. **Use OpenAI-compatible API format** for LM Studio calls
4. **Mermaid diagrams** should be rendered via mermaid.ink API
5. **Google Docs** use placeholder syntax: `{{variableName}}`

## n8n Workflow Structure

The main workflow follows this flow:
1. Schedule Trigger (hourly)
2. RSS Feed Read
3. Parse Jobs
4. For Each Job:
   - Check payment verified
   - Check duplicate
   - Score with LLM
   - Generate Mermaid diagram
   - Create Google Doc
   - Generate cover letter
   - Write to Google Sheets
   - Send Telegram if score >= threshold
