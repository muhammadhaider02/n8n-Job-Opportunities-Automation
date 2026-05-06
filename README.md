<div align="center">

# n8n Job Opportunities Automation

**Automated Job Email Parser and Tracker**

[![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71?logo=n8n&logoColor=white)](https://n8n.io)
[![DeepSeek](https://img.shields.io/badge/DeepSeek-AI_Agent-4D6BFE)](https://deepseek.com)
[![Gmail](https://img.shields.io/badge/Gmail-Trigger-EA4335?logo=gmail&logoColor=white)](https://gmail.com)
[![Google Sheets](https://img.shields.io/badge/Google_Sheets-Tracker-34A853?logo=googlesheets&logoColor=white)](https://sheets.google.com)

Stops students from manually sifting through university placement emails. Every opportunity gets parsed by an AI agent and logged to Google Sheets automatically. Relevant roles also trigger a personal email notification so nothing slips through.

</div>

---

## Table of Contents

- [Overview](#overview)
- [Workflow](#workflow)
- [Extracted Fields](#extracted-fields)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Configuration](#configuration)

---

## Overview

University placement cells send bulk job and internship emails that are easy to miss or forget to track. This workflow runs in the background and processes every email from a monitored sender automatically. DeepSeek reads the raw email text and returns structured JSON for every opportunity in the email. Computing and CS-adjacent roles go to one sheet, everything else goes to another. For relevant roles, a personal notification email fires so the student knows immediately without checking the sheet.

No credentials are stored in the workflow JSON. All sensitive values are injected via environment variables.

---

## Workflow

```
┌─────────────────────────────────────────────────────────┐
│           Gmail Trigger (polls every minute)            │
│           Filters by JOBS_SENDER env variable           │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                      Cleaning                           │
│  Strip HTML tags, newlines and extra whitespace         │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   AI Agent (DeepSeek)                   │
│  Classify relevance, extract structured fields per role │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                     JSON Split                          │
│  Parse AI output into one item per opportunity          │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│               If  (IsRelevant == "yes"?)                │
└────────────┬─────────────────────────────┬──────────────┘
             │ yes                         │ no
             ▼                             ▼
┌─────────────────────┐       ┌───────────────────────────┐
│     Append row      │       │        Append row         │
│  (Computing sheet)  │       │  (Non-Computing sheet)    │
└──────────┬──────────┘       └───────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│  Send notification email                                │
│  Subject: Role @ Company                                │
│  Body: deadline + direct link to the relevant sheet     │
└─────────────────────────────────────────────────────────┘
```

---

## Extracted Fields

For every opportunity in an email, the AI agent extracts:

| Field | Description |
|:---|:---|
| `IsRelevant` | `yes` or `no` based on fit for CS/DS roles |
| `Company` | Company name |
| `Role/Title` | Job or internship title |
| `Deadline` | Application deadline or "Not mentioned" |
| `Application Link` | Direct apply link if available |
| `Application Email` | Apply-by-email address if mentioned |
| `Paid` | `Yes` or `No` |
| `Amount` | Stipend or salary if mentioned |
| `Work Arrangement` | Remote / On-Site / Hybrid |
| `Location` | Location if mentioned |

Relevant roles are classified against: AI Engineer, ML Engineer, Full Stack, Python, Web, Frontend, Backend, Flutter, Computer Vision, Data Scientist, Data Analyst, ETL Engineer and general CS/DS roles.

---

## Prerequisites

- A running [n8n](https://n8n.io) instance
- Gmail OAuth2 credential configured in n8n
- Google Sheets OAuth2 credential configured in n8n
- DeepSeek or OpenRouter API credential configured in n8n
- A Google Sheet with a Computing tab and a Non-Computing tab

---

## Setup

**Step 1: Import the workflow**
```
n8n → Workflows → Import from File → n8n-Job-Opportunities-Automation.json
```

**Step 2: Add credentials in n8n**

- Gmail OAuth2: attach to the Gmail Trigger and Gmail Send nodes
- Google Sheets OAuth2: attach to both Google Sheets nodes
- DeepSeek (or swap to any supported model): attach to the AI Agent node

**Step 3: Set environment variables**

Copy `env.example`, fill in your values and configure them in your n8n environment.

**Step 4: Test and activate**

Run the workflow once manually to verify parsing and sheet append, then toggle it Active.

---

## Configuration

| Variable | Purpose |
|:---|:---|
| `JOBS_SENDER` | Gmail address to monitor (filters incoming trigger) |
| `JOBS_SHEET_ID` | Google Sheet ID from the spreadsheet URL |
| `JOBS_SHEET_GID_COMPUTING` | Sheet tab GID for relevant CS/DS roles |
| `JOBS_SHEET_GID_NONCOMPUTING` | Sheet tab GID for non-relevant roles |
| `JOBS_NOTIFY_TO` | Email address to send notifications to |
