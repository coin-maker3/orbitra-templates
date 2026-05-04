# Troubleshooting: Klaviyo Abandoned Cart Recovery Workflow with Smart Discount Tiers

## The workflow fails immediately on import

**Cause:** n8n version mismatch (workflow saved on a newer n8n than your instance)
**Fix:** Update n8n to the latest version. On n8n cloud, this is automatic. On self-hosted: `docker pull n8nio/n8n` then restart your container.

## A credential shows as 'invalid' in red

**Cause:** API key copied incorrectly OR missing scopes
**Fix:** Re-generate the API key. Make sure you copy the WHOLE string. Check your platform's docs for required scopes (listed in `SETUP_GUIDE.md`).

## Google Sheets node fails with 'sheet not found'

**Cause:** Sheet name typo or sheet/spreadsheet permissions
**Fix:** The 'sheetName' field must match the tab name EXACTLY (case-sensitive). Make sure the Google account connected has Editor access to the sheet.


## Still stuck?
Reply to your Gumroad receipt email. Include:
- The exact error message (screenshot is fine)
- Which node failed
- What you tried

We respond within 24 hours, fix or refund.
