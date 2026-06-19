# Lead Qualification Automation

---

## Overview

This n8n workflow automates the entire lead qualification process from form submission to calendar scheduling. When a prospect fills out a lead capture form, the workflow validates their data, scores them using a custom algorithm, logs everything across multiple Google Sheets, notifies the sales team on Slack, sends the lead a confirmation email, and books a follow-up meeting on Google Calendar — all without any manual effort.

---

## Problem Statement

Sales teams waste significant time manually reviewing, scoring, and following up on every incoming lead. Without automation, hot leads can go cold due to slow response times, data entry errors, and inconsistent follow-up scheduling. There is also no reliable way to separate high-value prospects from low-quality submissions in real time.

---

## Solution

This workflow captures leads through a structured form, immediately validates and scores each submission using budget and email type signals, and routes them into three priority tiers: Hot, Warm, and Cold. Each tier triggers a different follow-up schedule on Google Calendar. The sales team is notified instantly on Slack, and every lead is tracked across dedicated Google Sheets tabs for full pipeline visibility.

---

## Workflow Architecture

```
Form Submission (n8n Form Trigger)
        │
        ▼
Data Validation (If Node)
        │
   ┌────┴──────────────────┐
   │                       │
Valid Lead             Invalid Lead
   │                       │
   ▼                       ▼
Crypto (Generate       Rejected Leads
 Unique Lead ID)       (Google Sheets)
   │
   ▼
Approved Leads Sheet (Google Sheets)
   │
   ▼
Lead Scoring (JavaScript)
   │
   ▼
CRM Entry Sheet (Google Sheets)
   │
   ▼
Confirmation Email (Gmail)
   │
   ▼
Sales Team Alert (Slack)
   │
   ▼
Lead Status Switch (Hot / Warm / Cold)
   │
   ▼
Google Calendar Event (Scheduled per tier)
```

---

## Workflow Steps

### Trigger
- **On Form Submission** — An n8n form captures six fields from the prospect: Name, Email, Phone Number, Company Name, Budget, and Service (dropdown: AI Automation, Web Development, App Development).

### Validation
- **If Node** — Validates the submission against two strict conditions. The email must match a standard regex pattern, and the phone number field must not be empty. Both conditions must pass for the lead to proceed.

### Invalid Path
- **Append Row (Rejected Leads Sheet)** — Leads that fail validation are logged to the "Rejected Leads" tab in the Google Sheet for review. No further processing occurs.

### Valid Path
- **Crypto Node** — Generates a unique cryptographic Lead ID for each valid submission, ensuring every record can be traced and deduplicated.
- **Append Row (Approved Leads Sheet)** — The full lead data including the generated Lead ID and submission timestamp is written to the "Approved Leads" tab.

### Scoring
- **Code in JavaScript** — A custom scoring algorithm evaluates each lead on two criteria:
  - Budget above $5,000 scores 50 points; between $1,000 and $5,000 scores 30 points; below $1,000 scores 10 points.
  - A corporate email (non-Gmail, non-Yahoo, non-Outlook) scores 20 points; a free email provider scores 5 points.
  - Final status: **Hot Lead** (70+ points), **Warm Lead** (40 to 69 points), **Cold Lead** (below 40 points).

### CRM Logging
- **Append Row (CRM Entry Sheet)** — The scored lead record including Lead ID, Name, Email, Phone, Score, and Lead Status is written to the "CRM Entry" tab for the sales team's pipeline view.

### Notifications
- **Send Confirmation Email (Gmail)** — The lead receives an automated email thanking them for their interest in the selected service and informing them a consultant will reach out within 24 hours.
- **Slack Alert (sales-team channel)** — The sales team receives an instant notification in the `#sales-team` Slack channel showing the lead's status, ID, name, budget, and score.

### Scheduling
- **Switch Node** — Routes the lead to one of three calendar booking paths based on their status.
- **Google Calendar (Warm Lead)** — Books a 30-minute meeting 3 days from the submission date at 9:00 AM.
- **Google Calendar (Hot Lead)** — Books a 30-minute meeting 1 day from the submission date at 9:00 AM.
- **Google Calendar (Cold Lead)** — Books a 30-minute meeting 7 days from the submission date at 9:00 AM.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **n8n** | Workflow automation platform |
| **n8n Form Trigger** | Lead capture form |
| **Google Sheets (OAuth2)** | Rejected Leads, Approved Leads, and CRM Entry storage |
| **Google Calendar (OAuth2)** | Automated meeting scheduling per lead tier |
| **Gmail (OAuth2)** | Confirmation email to the lead |
| **Slack** | Real-time sales team notifications |
| **n8n Crypto Node** | Unique Lead ID generation |
| **n8n Code Node (JavaScript)** | Custom lead scoring logic |

---

## Features

- Real-time lead capture through a branded n8n form with dropdown service selection.
- Automatic data validation using regex email matching and phone number checks.
- Custom JavaScript scoring engine based on budget size and email type.
- Three-tier lead classification: Hot, Warm, and Cold with different follow-up urgency.
- Separate Google Sheets tabs for rejected leads, approved leads, and CRM records.
- Instant Slack notification to the sales team with lead score and status.
- Automated confirmation email sent to the prospect immediately after submission.
- Dynamic Google Calendar event creation with timing based on lead priority.
- Unique Lead ID generation for full traceability across all records.

---

## Use Cases

- **Digital marketing agencies** managing inbound inquiries for AI automation, web, or app development services.
- **SaaS sales teams** looking to automate top-of-funnel lead triage and first-touch follow-up.
- **Freelancers and consultancies** who want to respond to high-value prospects faster without manual sorting.
- **B2B service providers** needing a CRM-lite solution that tracks, scores, and schedules leads automatically.
- **Growth teams** running paid campaigns where fast, consistent lead response is critical to conversion.

---

## Setup Instructions

### 1. Import the Workflow
- Open your n8n instance.
- Go to **Workflows → Import from File**.
- Upload `Lead_Qualification_Automation.json`.

### 2. Configure Credentials

| Credential | Node(s) |
|---|---|
| **Google Sheets OAuth2** | Append row in sheet, sheet1, sheet2 |
| **Gmail OAuth2** | Send a message |
| **Google Calendar OAuth2** | Create an event (Hot / Warm / Cold Lead) |
| **Slack API** | Send a message1 |

Connect each credential under **Settings → Credentials** in n8n before activating.

### 3. Set Up Google Sheets
Create a Google Spreadsheet with three tabs named exactly:
- `Rejected Leads` — columns: Name, Email, Phone, Company, Budget, Service
- `Approved Leads` — columns: Name, Email, Phone, Company, Budget, Service, Lead ID, TimeStamp
- `CRM Entry` — columns: Lead ID, Name, Email, Phone, Score, Lead Status

Update the `documentId` value in each Google Sheets node to match your spreadsheet's ID (found in the spreadsheet URL).

### 4. Configure Slack Channel
- In the **Send a message1** node, update the `channelId` to match your target Slack channel.
- The current config targets a channel with ID `C0BAJ6K2K0R` (named `sales-team`).

### 5. Configure Google Calendar
- In each of the three calendar nodes, update the `calendar` field to your Google Calendar email address.
- The current config uses `shtalha636@gmail.com` as the calendar owner.

### 6. Review Scoring Logic (Optional)
- Open the **Code in JavaScript** node to adjust budget thresholds or scoring weights to match your business criteria.

### 7. Activate the Workflow
- Toggle the workflow to **Active** in n8n.
- The form trigger will generate a public URL you can embed or share with prospects.

### 8. Test the Workflow
- Submit a test entry through the form URL.
- Verify the lead appears in the correct Google Sheet tab.
- Confirm the Slack notification, confirmation email, and calendar event are all created as expected.