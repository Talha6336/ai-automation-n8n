# AI Resume Analyser — Automated Candidate Screening System

---

## Overview

This n8n workflow automates the full candidate screening process from resume submission to structured evaluation. When an applicant fills out a job application form and uploads their CV, the workflow stores the file in Google Drive, extracts its text, compares it against a predefined job description, generates a detailed AI screening report using GPT-4o-mini, sends the applicant a confirmation email, and logs the complete evaluation in a Google Sheet — all without any human involvement.

---

## Problem Statement

Reviewing resumes manually is one of the most time-consuming parts of any hiring process. Recruiters must read dozens of applications, cross-reference each one against job requirements, and form consistent judgments under time pressure. This leads to missed talent, inconsistent evaluations, and significant delays in the hiring pipeline.

---

## Solution

This workflow acts as an automated technical recruiter. The moment a candidate submits their application, their CV is processed and evaluated against the active job description stored in Google Drive. The AI agent produces a structured report covering candidate strengths, weaknesses, risk and reward factors, an overall fit rating out of 10, and a written justification. All output is saved to a centralized applicant tracking sheet for the hiring team to review at a glance.

---

## Workflow Architecture

```
Form Submission (n8n Form Trigger)
        │
        ▼
Upload CV to Google Drive (Job Applications folder)
        │
        ▼
Download CV from Google Drive
        │
        ▼
Extract Text from CV (PDF Extraction)
        │
        ▼
Download Job Description PDF from Google Drive
        │
        ▼
Extract Text from Job Description
        │
        ▼
AI Agent (GPT-4o-mini + Structured Output Parser)
        │
        ▼
Send Confirmation Email (Gmail)
        │
        ▼
Log Evaluation to Google Sheets (Job Applicant Tracking)
```

---

## Workflow Steps

### Trigger
- **On Form Submission** — An n8n form titled "Job Application" captures four fields from the applicant: Full Name (required), Email, Phone Number, and CV file upload.

### CV Handling
- **Upload File (Google Drive)** — The uploaded CV is stored in the designated Google Drive folder named `Job Applications(n8n)` using the original filename.
- **Download File (Google Drive)** — The newly uploaded file is immediately downloaded back into the workflow as binary data for text extraction.
- **Extract from File** — The PDF is parsed and its raw text content is extracted for use by the AI agent.

### Job Description Loading
- **Initial Job (Google Drive)** — A pre-configured job description PDF (`Full Stack Web Developer.pdf`) is downloaded from Google Drive. This file defines the role requirements the AI evaluates every candidate against.
- **Extract Data from Initial Job** — The job description PDF is parsed to extract its full text, which is passed alongside the resume into the AI agent.

### AI Evaluation
- **AI Agent (GPT-4o-mini)** — The agent receives both the applicant's resume text and the job description text. It acts as an expert technical recruiter and produces a full screening report structured around six evaluation dimensions:
  - **Candidate Strengths** — Top relevant qualifications specific to the job description.
  - **Candidate Weaknesses** — Gaps or mismatches between the resume and the role requirements.
  - **Risk Factor** — A score of Low, Medium, or High, plus a worst-case hiring scenario.
  - **Reward Factor** — A score of Low, Medium, or High, plus a best-case value description and whether the candidate is a short-term or long-term fit.
  - **Overall Fit Rating** — A whole number between 0 and 10 (no decimals).
  - **Justification for Rating** — A written explanation referencing specific resume content vs job requirements.
- **Structured Output Parser** — Enforces a strict JSON schema on the AI output, ensuring every evaluation field is consistently typed and present before proceeding.
- **OpenAI Chat Model (GPT-4o-mini)** — Powers the AI agent's language reasoning and structured generation.

### Notifications and Logging
- **Send a Message (Gmail)** — The applicant receives an automated confirmation email addressed by their first name, sent from the Cascade Digital team, acknowledging receipt and informing them the team will follow up soon.
- **Append Row in Sheet (Google Sheets)** — The full evaluation is written to the "Job Applicant Tracking" spreadsheet with these columns: Applicant Name, Email, Phone, Application Date, Resume (Google Drive link), Strengths, Weaknesses, Risk Factor, Reward Factor, Overall Fit, and Justification.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **n8n** | Workflow automation platform |
| **n8n Form Trigger** | Applicant intake form with file upload |
| **Google Drive (OAuth2)** | CV storage and job description retrieval |
| **n8n Extract from File** | PDF text extraction for CV and job description |
| **OpenAI GPT-4o-mini** | Resume analysis and evaluation generation |
| **Gmail (OAuth2)** | Applicant confirmation email |
| **Google Sheets (OAuth2)** | Centralized applicant tracking and evaluation log |

---

## Features

- End-to-end automated screening from form submission to structured report, with no manual steps.
- PDF resume parsing directly from applicant uploads, no format conversion needed.
- Job description loaded dynamically from Google Drive, making it easy to swap roles without editing the workflow.
- Six-dimensional AI evaluation covering strengths, weaknesses, risk, reward, fit rating, and written justification.
- Strict JSON schema enforcement via Structured Output Parser, guaranteeing consistent data in every row.
- Automated first-touch confirmation email sent to applicants under the Cascade Digital brand.
- Resume stored in Google Drive with a direct web view link logged in the tracking sheet for instant access.
- Application date automatically stamped in `yyyy-MM-dd` format at the time of submission.

---

## Use Cases

- **Tech companies and agencies** screening candidates for software, AI, or automation roles at volume.
- **HR teams** that want consistent, bias-reduced initial evaluation across all applicants.
- **Recruitment consultancies** managing multiple job openings who need fast first-pass filtering.
- **Startups** without a dedicated HR team who need a structured hiring process from day one.
- **Hiring managers** who want a ranked, documented shortlist before conducting any interviews.

---

## Setup Instructions

### 1. Import the Workflow
- Open your n8n instance.
- Go to **Workflows → Import from File**.
- Upload `Ai_Resume_Analyser.json`.

### 2. Configure Credentials

| Credential | Node(s) |
|---|---|
| **Google Drive OAuth2** | Upload file, Download file, Initial Job |
| **Google Sheets OAuth2** | Append row in sheet |
| **Gmail OAuth2** | Send a message |
| **OpenAI API Key** | OpenAI Chat Model |

Connect each credential under **Settings → Credentials** in n8n.

### 3. Set Up Google Drive

Create a folder in Google Drive named `Job Applications(n8n)` and copy its folder ID from the URL. Update the `folderId` value in the **Upload file** node to match.

Upload the job description PDF for the role you are hiring for. Open the **Initial Job** node and update the `fileId` to point to your job description file. The current config references `Full Stack Web Developer.pdf`.

### 4. Set Up Google Sheets

Create a spreadsheet named `Job Applicant Tracking` with the following columns in Sheet1:

```
Applicant Name | Email | Phone | Application Date | Resume | Strengths | Weaknesses | Risk Factor | Reward Factor | Overall Fitt | Justification
```

Update the `documentId` in the **Append row in sheet** node with your spreadsheet's ID (found in its URL).

### 5. Update Confirmation Email Sender

In the **Send a message** node, confirm the Gmail credential is connected to the email address you want applicants to receive their confirmation from. The sign-off is currently set to `Cascade Digital` — update the message body to reflect your company name.

### 6. Activate the Workflow
- Toggle the workflow to **Active** in n8n.
- The form trigger will generate a public URL to share with applicants.

### 7. Test the Workflow
- Submit a test application through the form URL using a sample PDF resume.
- Confirm the CV appears in the Google Drive folder.
- Verify the AI evaluation is complete and accurate in the Google Sheet.
- Check that the confirmation email is received by the test applicant email address.

---
