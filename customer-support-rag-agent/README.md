# Customer Support RAG Agent — Automated Gmail Response System
 
---
 
## Overview
 
This n8n workflow automates customer support email handling using a Retrieval-Augmented Generation (RAG) architecture. It monitors a Gmail inbox every minute, classifies incoming emails, and — for customer support queries — generates intelligent, context-aware replies using OpenAI GPT-4o-mini and a Pinecone vector knowledge base. Replies are sent automatically without any human intervention.
 
---
 
## Problem Statement
 
Support teams are overwhelmed by repetitive customer emails asking about policies, FAQs, product details, and service queries. Manual responses are slow, inconsistent, and resource-intensive — leading to delayed replies, poor customer experience, and burnout.
 
---
 
## Solution
 
This workflow acts as an always-on AI support agent. Every incoming email is automatically classified. If it is a customer support query, the AI agent retrieves relevant answers from a pre-loaded Pinecone knowledge base (FAQs and policies), composes a friendly, branded reply, labels the email in Gmail, and sends the response — all within seconds, with zero manual effort.
 
---
 
## Workflow Architecture
 
```
Gmail Trigger (Poll every minute)
        │
        ▼
Text Classifier (GPT-4o-mini)
        │
   ┌────┴────────────────┐
   │                     │
Customer Support       Other
   │                     │
   ▼                     ▼
AI Agent            No Operation
(GPT-4o-mini +          (Stop)
 Pinecone RAG)
   │
   ▼
Add Gmail Label
   │
   ▼
Reply to Email (Gmail)
```
 
---
 
## Workflow Steps
 
### Trigger
- **Gmail Trigger** — Polls the connected Gmail inbox every minute for new emails. Captures the full email content including body text, sender, and message ID.
### Classification
- **Text Classifier** — Uses GPT-4o-mini to classify each email into one of two categories:
  - `Customer Support` — Questions about products, services, or company policies.
  - `Other` — Any unrelated emails (newsletters, spam, internal messages, etc.).
### Routing
- **Customer Support path** → Passes email text to the AI Agent for processing.
- **Other path** → Routes to a `No Operation` node and stops — no reply is generated.
### AI Response Generation
- **AI Agent** — Orchestrates the RAG pipeline. Receives the email text and uses the Pinecone knowledge base tool to retrieve relevant FAQ and policy content before composing a response. Configured with a branded persona: *Mr. Helpful from Tech Heaven*. Tone is friendly and emoji-inclusive.
- **OpenAI Chat Model (GPT-4o-mini)** — Powers the AI Agent's language understanding and response generation.
- **Pinecone Vector Store** — Retrieves semantically relevant documents from the `ragagent` Pinecone index under the `FAQ` namespace. Operates in `retrieve-as-tool` mode, callable by the AI Agent.
- **Embeddings OpenAI** — Converts query text into 512-dimension vector embeddings used to query the Pinecone index.
### Output
- **Add Label to Message** — Tags the handled email with the `CATEGORY_PERSONAL` Gmail label for tracking and organization.
- **Reply to a Message** — Sends the AI-generated response as a direct reply to the original email thread using the Gmail API.
---
 
## Tech Stack
 
| Tool | Purpose |
|---|---|
| **n8n** | Workflow automation platform |
| **Gmail (OAuth2)** | Email trigger, labeling, and reply |
| **OpenAI GPT-4o-mini** | Email classification and response generation |
| **OpenAI Embeddings API** | Text vectorization (512 dimensions) |
| **Pinecone** | Vector database for FAQ and policy knowledge base |
| **LangChain nodes (n8n)** | Text Classifier, AI Agent, Vector Store integration |
 
---
 
## Features
 
- **Automated inbox monitoring** — polls Gmail every minute with no manual trigger required.
- **Intelligent email classification** — filters out irrelevant emails before any AI processing occurs.
- **RAG-powered responses** — AI answers are grounded in your actual business knowledge base, not hallucinated.
- **Branded persona** — replies are signed consistently as *Mr. Helpful from Tech Heaven* with a friendly, emoji-rich tone.
- **Auto-labeling** — handled emails are automatically labeled in Gmail for audit trails.
- **Threaded replies** — responses are sent within the original email thread for professional continuity.
- **Fully autonomous** — end-to-end operation with zero human intervention required.
---
 
## Use Cases
 
- **E-commerce & retail** — Handle order status, return policy, and shipping queries automatically.
- **SaaS products** — Answer subscription, billing, and feature-related questions from users.
- **Tech support desks** — Resolve common troubleshooting FAQs without assigning tickets.
- **Service businesses** — Respond to service availability, pricing, and appointment queries.
- **Startups** — Provide 24/7 customer support coverage without a dedicated support team.
---
 
## Setup Instructions
 
### 1. Import the Workflow
- Open your n8n instance.
- Go to **Workflows → Import from File**.
- Upload `Customer_Support_RAG_Agent.json`.
### 2. Configure Credentials
 
| Credential | Where to Configure |
|---|---|
| **Gmail OAuth2** | n8n Credentials → Google OAuth2 → connect your support Gmail account |
| **OpenAI API Key** | n8n Credentials → OpenAI → add your API key |
| **Pinecone API Key** | n8n Credentials → Pinecone → add your API key and select region |
 
### 3. Set Up Pinecone Knowledge Base
- Create a Pinecone index named `ragagent`.
- Create a namespace called `FAQ`.
- Populate it with your company's FAQs, product details, and policy documents using OpenAI embeddings (512 dimensions).
### 4. Configure Gmail Label
- In the **Add Label to Message** node, update `labelIds` to match a label that exists in your Gmail account, or create a new one named `Customer Support`.
### 5. Review AI Agent Persona *(Optional)*
- Open the **AI Agent** node and edit the system message to update the company name (`Tech Heaven`) and any tone/guideline adjustments to match your brand.
### 6. Activate the Workflow
- Toggle the workflow to **Active** in n8n.
- The Gmail trigger will begin polling immediately.
### 7. Test the Workflow
- Send a test email to the connected Gmail account with a customer-style question (e.g., *"What is your return policy?"*).
- Monitor the n8n execution log to verify classification, AI response generation, labeling, and reply sending.
---

 