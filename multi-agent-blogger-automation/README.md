# Multi Agent Blogger Automation — AI Newsletter Generation System

---

## Overview

This n8n workflow uses a three-agent AI pipeline to research, write, and publish a fully formatted HTML newsletter draft — entirely without human input. A Planning Agent discovers trending topics, a Section Writer produces sourced editorial content for each one, and an Editor Agent assembles everything into a responsive HTML email that lands directly in a Gmail draft, ready to send.

---

## Problem Statement

Creating a high-quality newsletter consistently requires hours of research, writing, editing, and formatting. Content teams must find relevant stories, distill them into readable sections, maintain a consistent tone, and produce clean HTML output — all before a single word reaches the audience. This process is slow, expensive, and hard to scale.

---

## Solution

This workflow replaces the entire pre-publication pipeline with a coordinated system of three specialized AI agents. Each agent handles one stage: planning, writing, and editing. Tavily provides real-time web research. Structured output parsers ensure every agent's output is machine-readable and passed cleanly to the next stage. The final output is a polished, inline-styled HTML email draft waiting in Gmail.

---

## Workflow Architecture

```
Manual Trigger
        │
        ▼
Tavily Search (News, past week, topic: agentic AI for small business)
        │
        ▼
Planning Agent (GPT-4o-mini)
Proposes 1 newsletter title + exactly 3 section topics
        │
        ▼
Structured Output Parser (title + topics schema)
        │
        ▼
Split Out (one item per topic)
        │
        ▼
Tavily Search per Topic (general, past month, raw content included)
        │
        ▼
Section Writer Agent (GPT-4o-mini) — runs once per topic
Writes a 350–500 word sourced Markdown section with inline citations
        │
        ▼
Aggregate (collects all 3 sections into one item)
        │
        ▼
Editor Agent (GPT-4o-mini)
Converts all sections into a responsive HTML newsletter body
        │
        ▼
Structured Output Parser (subject + content schema)
        │
        ▼
Gmail — Create Draft (HTML email, ready to review and send)
```

---

## Workflow Steps

### Trigger
- **When clicking "Execute Workflow"** — A manual trigger starts the pipeline on demand. The workflow can be adapted to a schedule trigger for fully automated publishing.

### Initial Research
- **Search (Tavily)** — Queries Tavily for news articles on the topic `"how agentic AI can help small business"` from the past week. Returns the top 3 results with titles, URLs, published dates, and content summaries.

### Agent 1 — Planning Agent
- **Planning Agent (GPT-4o-mini)** — Receives the 3 article digests and acts as an expert newsletter planner for an audience of small-business leaders. It proposes:
  - A catchy newsletter edition title (80 characters or fewer, no clickbait).
  - Exactly 3 concise, distinct section topics (3 to 5 words each, no duplicates).
- **Structured Output Parser** — Enforces a strict schema: `title` (string, min 10 chars) and `topics` (array of exactly 3 strings, each 3 to 50 chars). No extra text is allowed in the output.
- **Split Out** — Splits the 3 topics into individual items so each can be researched and written in parallel.

### Per-Topic Research and Writing
- **Search1 (Tavily)** — For each topic, performs a fresh Tavily search scoped to the past month on the general topic category, with raw page content included for deeper sourcing.
- **Section Writer (GPT-4o-mini)** — Writes one standalone newsletter section (350 to 500 words) per topic. Requirements enforced by the system prompt:
  - Starts with an H2 heading matching the topic.
  - Synthesizes facts from retrieved sources only (no invented content).
  - Adds inline citation markers like [1], [2] throughout the prose.
  - Appends a numbered Sources list with domain links after each section.
  - Tone is clear, expert, and engaging.
  - Output is plain Markdown for the editor to convert.

### Assembly and Editing
- **Aggregate** — Collects all 3 completed Markdown sections into a single item and passes them together to the Editor Agent.
- **Editor Agent (GPT-4o-mini)** — Receives the planning title and all 3 sections. Acts as the newsletter editor and layout stylist. Produces two outputs:
  - `subject` — A clean email subject line (80 characters or fewer, no emojis).
  - `content` — A valid, responsive HTML body (no `<!DOCTYPE>`, `<html>`, or `<head>` tags) structured with: a header containing the title and current date, a short intro paragraph, all 3 converted sections, a deduplicated Key Sources list, and a friendly sign-off. Inline CSS handles layout (max-width container, readable font and line height). All links include `rel="noopener noreferrer"` and `target="_blank"`.
- **Structured Output Parser1** — Enforces the final schema: `subject` (string) and `content` (string), both required.

### Output
- **Create a Draft (Gmail)** — Creates an HTML Gmail draft using the AI-generated subject and HTML content body. The draft is ready for a human to review and send, with no additional formatting work needed.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **n8n** | Workflow automation platform |
| **Tavily API** | Real-time web search and news retrieval |
| **OpenAI GPT-4o-mini** | Powers all three AI agents |
| **LangChain AI Agent nodes (n8n)** | Planning Agent, Section Writer, Editor Agent |
| **LangChain Structured Output Parsers** | Schema enforcement on Planning and Editor outputs |
| **n8n Split Out** | Distributes topics for per-topic research and writing |
| **n8n Aggregate** | Recombines written sections before editing |
| **Gmail (OAuth2)** | Creates the final HTML newsletter draft |

---

## Features

- Three specialized AI agents, each with a distinct role and system prompt, working in a coordinated pipeline.
- Real-time research via Tavily at two stages: broad topic discovery and deep per-section sourcing.
- Automatic inline citation generation with a numbered sources list per section.
- Strict structured output parsing at two points in the pipeline, ensuring clean data flow between agents.
- Fully responsive HTML email output with inline CSS, semantic headings, and accessibility-compliant link attributes.
- Gmail draft creation allows human review before sending, maintaining editorial control.
- Current date is dynamically injected into the newsletter header at runtime.
- No hardcoded content — the entire newsletter topic, structure, and writing is generated fresh each run.

---

## Use Cases

- **Content marketing teams** publishing weekly AI or tech industry newsletters without a full editorial team.
- **Agencies** producing branded newsletters for clients across multiple industries with topic-specific pipelines.
- **Solo founders and consultants** who want consistent thought-leadership content with minimal time investment.
- **Media and publishing startups** building automated newsletter products at scale.
- **Internal knowledge newsletters** that summarize industry developments for leadership or employees each week.

---

## Setup Instructions

### 1. Import the Workflow
- Open your n8n instance.
- Go to **Workflows → Import from File**.
- Upload `Multi_Agent_Blogger_Automation.json`.

### 2. Configure Credentials

| Credential | Node(s) |
|---|---|
| **Tavily API Key** | Search, Search1 |
| **OpenAI API Key** | OpenAI Chat Model, OpenAI Chat Model1, OpenAI Chat Model2 |
| **Gmail OAuth2** | Create a draft |

Connect each credential under **Settings → Credentials** in n8n before activating.

### 3. Customize the Research Topic
- Open the **Search** node.
- Update the `query` field from `"how agentic AI can help small business"` to your desired newsletter topic.
- Adjust `max_results` (currently 3) and `time_range` (currently `week`) as needed.

### 4. Adjust the Target Audience (Optional)
- Open the **Planning Agent** node and edit the system message to update the audience description from `"small-business leaders"` to your own readership.
- Open the **Section Writer** node and update the tone and audience reference in the system prompt to match.

### 5. Update the Newsletter Sign-off (Optional)
- The Editor Agent's system prompt controls the friendly sign-off at the end of the HTML email. Edit the system message in the **Editor Agent** node to reflect your brand name or author signature.

### 6. Schedule the Workflow (Optional)
- Replace the **Manual Trigger** node with a **Schedule Trigger** to run the pipeline automatically, for example every Monday morning for a weekly newsletter.

### 7. Activate and Run
- Toggle the workflow to **Active** if using a schedule trigger, or click **Execute Workflow** to run it manually.
- Check your Gmail Drafts folder for the generated newsletter.
- Review, adjust if needed, and send.

---
