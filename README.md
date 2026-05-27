# global-remote-job-engine-1hxu8

## Overview

This n8n workflow autonomously finds, evaluates, and applies for remote job positions tailored to a specific candidate profile. It orchestrates multiple job board APIs and RSS feeds, consolidates the data, uses AI to score job relevance and generate cover letters, and then either auto-applies via an external Playwright service, flags for human review, or archives low-scoring/non-remote roles, logging all actions to Google Sheets and sending Slack notifications upon completion.

## Features

- Multi-source job scraping (Apify tasks for LinkedIn, Indeed, Dice; direct APIs for Google Jobs, Remotive, Arbeitnow, Jobicy; RSS for WeWorkRemotely).
- Handles asynchronous Apify scraper runs with polling and merging.
- Robust job data normalization and schema mapping across diverse sources.
- Deduplication of job listings based on apply URL and previously applied jobs.
- Hard filtering for 'remote' jobs based on keywords and AI verification.
- AI-powered job scoring and justification using Anthropic Claude.
- AI-generated personalized cover letters.
- Automated job application via an external Playwright service for high-scoring, verified remote jobs.
- Tiered job processing: auto-apply (score >= 8), human review (score 6-7), archive (score < 6 or not remote).
- Comprehensive logging of all job interactions to Google Sheets (submitted, review, low score, not remote).
- Scheduled execution every 6 and 24 hours.
- Slack notifications for workflow run completion.
- Error handling for individual scraper and API calls to ensure workflow continuity.

## Services Used

- Google Sheets
- Apify
- SerpAPI
- Remotive API
- Arbeitnow API
- Jobicy API
- WeWorkRemotely RSS
- Anthropic Claude AI
- Custom Playwright Application Service
- Slack

## Trigger

Two Schedule Trigger nodes: 'Every 24 Hours' and 'Every 6 Hours'.

## Prerequisites

- An n8n instance (self-hosted or cloud).
- Google Cloud Project (for Google Sheets API access).
- Apify account with pre-configured tasks for LinkedIn, Indeed, and Dice scrapers.
- SerpAPI account.
- Anthropic API key.
- An external Playwright-based application service (custom-built or third-party).
- A publicly accessible URL for your resume (e.g., S3 bucket, Google Drive).
- Slack workspace and an incoming webhook.

## Credentials

- Google Sheets (OAuth2 API - `CRED_GSHEETS`)
- Apify API Token (manual input for HTTP Request nodes and Code node)
- SerpAPI Key (manual input for HTTP Request node)
- Anthropic API Key (manual input for HTTP Request node)
- Playwright Service Token (manual input for HTTP Request node)
- Slack Webhook URL (manual input for HTTP Request node)

## Configuration

1. Replace "YOUR_GOOGLE_SHEET_ID" in all Google Sheets nodes with your actual Google Sheet ID. Create a 'Human Review' sheet/tab in your Google Sheet, or adjust the sheetName in 'Log: HUMAN_REVIEW' node.
2. Set up Google Sheets OAuth2 credentials in n8n (named 'Google Sheets' with ID 'CRED_GSHEETS').
3. Replace "APIFY_TASK_LINKEDIN", "APIFY_TASK_INDEED", "APIFY_TASK_DICE" with your Apify task IDs.
4. Replace "APIFY_TOKEN" in HTTP Request nodes and the 'Poll Apify + Merge All Sources' Code node with your Apify API token.
5. Replace "SERPAPI_KEY" in the 'Google Jobs via SerpAPI' node with your SerpAPI key.
6. Update search keywords and locations in scraper nodes (LinkedIn, Indeed, Dice, Google Jobs, Remotive, Arbeitnow, Jobicy) to match desired job profiles.
7. Replace "ANTHROPIC_API_KEY" in the 'Claude AI — Score + Cover Letter' node with your Anthropic API key. Update the `CANDIDATE PROFILE` in the node's JSON body to reflect your actual skills and preferences.
8. Replace "YOUR_PLAYWRIGHT_SERVICE" and "PLAYWRIGHT_SERVICE_TOKEN" in the 'Playwright Apply Engine' node with your Playwright service endpoint and token.
9. Replace "YOUR_BUCKET/resume.pdf" with the public URL to your resume, and fill in "YOUR_FULL_NAME", "YOUR_EMAIL", "YOUR_PHONE", "YOUR_LINKEDIN_URL" in the 'Playwright Apply Engine' node.
10. Replace "YOUR_SLACK_WEBHOOK" in the 'Slack Notification' node with your Slack incoming webhook URL.

## Usage

1. Ensure all prerequisites are met and configuration steps are completed and saved.
2. Activate the n8n workflow. The workflow will automatically trigger every 6 and 24 hours based on the schedule triggers.
3. Monitor the Google Sheet for new entries in 'AUTO_SUBMITTED', 'HUMAN_REVIEW', 'LOW_SCORE_ARCHIVED', and 'REJECTED_NOT_REMOTE' statuses.
4. Check Slack for run completion notifications.
5. Periodically review the 'Human Review' tab in your Google Sheet for jobs requiring manual application, where draft cover letters and justifications are provided.

## Troubleshooting

- If the workflow gets stuck or retries repeatedly at 'Poll Apify + Merge All Sources', verify your Apify tokens are valid and the Apify tasks are completing successfully. The workflow is designed to wait for Apify jobs.
- If no jobs are processed after 'Normalise + Dedup + Remote Filter' (indicated by `_empty: true`), check scraper configurations, search terms, and the `REMOTE_SIGNALS` in the Code node, as filters might be too strict or sources are yielding no results.
- If 'Parse Claude Response' fails or produces poor data, review Claude's API response in execution history. Ensure the Anthropic API key is valid and the prompt strictly enforces JSON output as specified ('RETURN ONLY THIS JSON OBJECT').
- Many HTTP Request nodes use `onError: 'continueErrorOutput'`. This prevents workflow failure but means errors in individual API calls might be masked. Monitor n8n execution logs closely for specific node errors.
- If jobs are not being applied by 'Playwright Apply Engine', verify the Playwright service is running, accessible, and correctly authenticated. Check the `jobMeta` and application details being sent.
- If data in Google Sheets is incomplete or incorrect, verify the data mapping expressions in the Google Sheets nodes and the JSON structure output by preceding Code nodes.

## Security Notes

- API keys and sensitive personal information (name, email, phone, LinkedIn) are directly embedded in the workflow JSON (hardcoded). For production, it is strongly recommended to use n8n's built-in credentials management or environment variables (`={{ $env.MY_VAR }}`) for all sensitive data.
- The Playwright service endpoint and token are critical. Ensure the Playwright service is secured with appropriate authentication/authorization and is only accessible by trusted n8n instances.
- The Google Sheet ID is exposed in the workflow. Ensure the Google Sheet's sharing settings are configured to prevent unauthorized access.
- Web scraping activities from various job boards should comply with their respective terms of service to avoid potential legal or access issues.
