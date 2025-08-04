## Job Opportunities Automation (n8n)

An n8n workflow that reads job/internship emails, extracts structured details with an AI agent, appends them to Google Sheets (separate sheets for relevant and non-relevant roles), and notifies by email. No credentials are included in this repo.

### What this is
- An importable n8n workflow (`n8n-Job-Opportunities-Automation.json`)
- Triggers on Gmail, cleans and parses messages, classifies relevance, writes to Google Sheets, and sends an email notification
- Uses environment variables to avoid secrets and IDs in the workflow JSON
- Exported without credentials or personal identifiers; inactive by default

### Required environment variables
- `JOBS_SENDER`: Email address to filter incoming messages from
- `JOBS_SHEET_ID`: Google Sheet ID (the long ID in the spreadsheet URL)
- `JOBS_SHEET_GID_COMPUTING`: GID for the Computing sheet (number after `gid=`)
- `JOBS_SHEET_GID_NONCOMPUTING`: GID for the Non-Computing sheet
- `JOBS_NOTIFY_TO`: Notification recipient email address

See `env.example` for a template you can copy and set values for your environment.

### How to use
1. Import the workflow
   - In n8n: Workflows → Import from File → choose `Job Opportunities Automated.json`.

2. Add your credentials in n8n (not in this repo)
   - Gmail OAuth2: create/select a Gmail credential for the Gmail Trigger and Gmail nodes.
   - Google Sheets OAuth2: create/select a credential for the Google Sheets nodes.
   - Model provider (optional, for the AI agent): add credentials for DeepSeek/OpenRouter or switch the model node.

3. Set environment variables
   - Configure the variables listed above in your n8n environment or replace the expressions in the nodes.

4. Test and activate
   - Run once to verify parsing, sheet append, and notification, then toggle the workflow Active.

### Notes
- The email notification includes a dynamic link to the appropriate sheet based on relevance