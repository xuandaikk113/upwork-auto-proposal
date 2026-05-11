# Google Doc Template Setup

## Overview

The workflow creates personalized proposal documents by copying a template and replacing placeholders with AI-generated content.

## Creating Your Template

### Step 1: Create the Document

1. Go to [Google Docs](https://docs.google.com)
2. Create a new blank document
3. Name it: "Upwork Proposal Template"

### Step 2: Add Template Content

Copy and paste this template:

---

# {{titleOfSystem}}

## {{briefExplanation}}

Hi! Here's my proposed solution for {{specificRequest}}.

---

## Solution Architecture

{{mermaidDiagramImage}}

*The diagram above shows the high-level architecture of my proposed solution.*

---

## My Approach

{{stepByStepPlan}}

---

## Why I'm the Right Fit

{{relevantExperience}}

---

## Timeline & Investment

{{timeline}}

---

## Next Steps

1. Reply to my proposal on Upwork
2. We'll schedule a quick call to discuss details
3. I'll provide a detailed project plan

Looking forward to working together!

---

### Step 3: Style the Document

- Use a clean, professional font (e.g., Inter, Roboto, or default)
- Add your logo/branding if desired
- Keep formatting minimal and readable

### Step 4: Get the Document ID

1. Look at the URL: `https://docs.google.com/document/d/DOCUMENT_ID/edit`
2. Copy the `DOCUMENT_ID` portion
3. This will be used in the n8n workflow

### Step 5: Set Permissions

1. Click Share
2. Change to "Anyone with the link can view"
3. Copy the share link for reference

## Placeholder Reference

| Placeholder | Filled With |
|------------|-------------|
| `{{titleOfSystem}}` | AI-generated title based on job |
| `{{briefExplanation}}` | One-line summary of solution |
| `{{specificRequest}}` | Key requirement from job description |
| `{{mermaidDiagramImage}}` | Rendered Mermaid diagram (or link to it) |
| `{{stepByStepPlan}}` | Bullet points of approach |
| `{{relevantExperience}}` | Matching experience from resume |
| `{{timeline}}` | Estimated timeline and rate |

## Notes

- The workflow copies this template for each job
- Original template is never modified
- Each copy is shared publicly for client access
- Consider creating multiple templates for different job types
