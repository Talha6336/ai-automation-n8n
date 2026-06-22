# Appointment Booking Agent — AI Powered Google Calendar Scheduling Assistant


## Overview

This n8n workflow is a conversational AI scheduling agent that manages Google Calendar appointments entirely through natural language chat. Users can book, reschedule, cancel, or check the availability of appointments by simply describing what they need. The agent understands context across a conversation, confirms every action clearly, and always verifies availability before making any calendar change.

---

## Problem Statement

Scheduling appointments manually through calendar interfaces requires users to check availability themselves, navigate booking forms, and handle rescheduling or cancellations one step at a time. For businesses handling frequent bookings, this creates friction for both staff and clients, and leaves room for double-bookings, missed confirmations, and inconsistent communication.

---

## Solution

This workflow deploys a GPT-4o-mini powered agent directly inside an n8n chat interface. The agent interprets user intent from plain language, extracts the relevant time variables, and executes the correct sequence of Google Calendar operations automatically. A sliding memory window keeps the last 10 messages in context, enabling natural back-and-forth conversations without the user needing to repeat themselves.

---

## Workflow Architecture

```
Chat Trigger (n8n Chat Interface)
        │
        ▼
AI Agent (GPT-4o-mini + Simple Memory)
        │
        ├── Check Availability (Google Calendar)
        ├── Book Appointment (Google Calendar — Create Event)
        ├── Get Appointment (Google Calendar — Get All by time range)
        ├── Update Appointment (Google Calendar — Update Event)
        └── Cancel Appointment (Google Calendar — Delete Event)
```

---

## Workflow Steps

### Trigger
- **When Chat Message Received** — An n8n chat trigger listens for incoming user messages through the built-in chat interface. This can be embedded in a website, internal tool, or accessed directly via the n8n chat URL.

### Memory
- **Simple Memory (Buffer Window)** — Maintains a rolling context window of the last 10 messages. This allows the agent to handle multi-turn conversations naturally, for example when a user provides the time after the agent asks a follow-up question.

### AI Agent
- **AI Agent (GPT-4o-mini)** — The core of the workflow. The agent receives each message alongside today's date (injected dynamically via `$now`) and determines the user's intent. It identifies four key variables from the conversation:
  - `action` — What the user wants: booking, reschedule, check availability, or cancel.
  - `booking_time` — The target time for a booking, cancellation, or the original time in a reschedule.
  - `reschedule_time` — The new time when rescheduling (defaults to checking within the next 3 days if not provided).
  - `availability_time` — The time or range to check for availability.

  If any required variable is missing, the agent politely asks for it before taking any action.

### Calendar Tools (All connected to Google Calendar)

- **Check Availability** — Queries the calendar using a `Start_Time` and `End_Time` range to determine whether a slot is free. This tool is always called before any booking or reschedule operation.

- **Book Appointment** — Creates a new 30-minute calendar event at the specified `Start` and `End` time with an AI-generated `Summary`. Only fires after availability is confirmed.

- **Get Appointment** — Retrieves all events within a given time window (`After` and `Before`) to locate an existing appointment before updating or cancelling it.

- **Update Appointment** — Updates an existing event by `Event_ID` with new time details. Used exclusively during reschedule flows after the new time is confirmed as available.

- **Cancel Appointment** — Deletes a calendar event by `Event_ID`. Only executes after the correct appointment has been positively identified via Get Appointment.

### Agent Decision Logic

| Action | Steps Taken |
|---|---|
| **Check Availability** | Check Availability → Report result. If asked for options, suggest two time ranges closest to now. |
| **Booking** | Check Availability → If free, Book Appointment → Confirm. If taken, report and ask for another time. |
| **Cancel** | Get Appointment → If found, Cancel Appointment → Confirm. If not found, report. |
| **Reschedule** | Get Appointment → If found, Check Availability at new time → If free, Update Appointment → Confirm. |

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **n8n** | Workflow automation platform |
| **n8n Chat Trigger** | Conversational interface for user input |
| **OpenAI GPT-4o-mini** | Natural language understanding and agent reasoning |
| **LangChain AI Agent (n8n)** | Orchestrates tool selection and multi-step flows |
| **LangChain Memory Buffer Window** | Maintains 10-message conversation context |
| **Google Calendar (OAuth2)** | Create, read, update, and delete calendar events |

---

## Features

- Full natural language scheduling — no forms, dropdowns, or manual calendar navigation required.
- Four complete scheduling actions: book, reschedule, cancel, and availability check.
- Availability is always verified before any booking or reschedule is executed — no double-bookings possible.
- Conversational memory across 10 messages allows natural multi-turn interactions.
- Dynamic date injection ensures the agent always knows the current date and time at runtime.
- Intelligent availability suggestions — when a slot is unavailable or the user asks for options, the agent proposes two alternative time ranges.
- All bookings default to 30 minutes and Asia/Karachi timezone unless the user specifies otherwise.
- Clear, consistent confirmation messages for every action taken or error encountered.
- The agent never modifies or deletes an event without first confirming the correct event has been identified.

---

## Use Cases

- **Clinics and healthcare providers** allowing patients to self-book or reschedule appointments through a chat widget.
- **Consultants and freelancers** managing client discovery calls without back-and-forth emails.
- **Service businesses** (salons, tutors, coaches) offering conversational booking on their website.
- **Internal team scheduling tools** for booking meeting rooms or one-on-one sessions.
- **Customer support workflows** where booking a follow-up call is part of the support resolution flow.

---

## Setup Instructions

### 1. Import the Workflow
- Open your n8n instance.
- Go to **Workflows → Import from File**.
- Upload `Appointment_Booking_Agent.json`.

### 2. Configure Credentials

| Credential | Node(s) |
|---|---|
| **OpenAI API Key** | OpenAI Chat Model |
| **Google Calendar OAuth2** | Book Appointment, Update Appointment, Get Appointment, Cancel Appointment, Check Availability |

Connect each credential under **Settings → Credentials** in n8n.

### 3. Update the Google Calendar Target
All five calendar tool nodes are currently pointed at `shtalha636@gmail.com`. Open each of the following nodes and update the `calendar` field to your own Google Calendar email address:

- Book Appointment
- Update Appointment
- Get Appointment
- Cancel Appointment
- Check Availability

### 4. Adjust Timezone and Default Duration (Optional)
- The agent defaults all bookings to **30 minutes** and the **Asia/Karachi** timezone.
- To change either, open the **AI Agent** node and edit the relevant lines at the bottom of the system message under the `# Notes` section.

### 5. Adjust Memory Window (Optional)
- The **Simple Memory** node is set to retain the last **10 messages**.
- Open the node and increase or decrease `contextWindowLength` based on how long your typical booking conversations are.

### 6. Activate the Workflow
- Toggle the workflow to **Active** in n8n.
- Access the chat interface via the Chat Trigger's generated URL to start taking bookings.

### 7. Test the Workflow
Use the following sample inputs to verify each action works correctly:

| Test Input | Expected Behaviour |
|---|---|
| `"Book me an appointment tomorrow at 10am"` | Checks availability, books if free, confirms. |
| `"Am I free on Thursday afternoon?"` | Returns availability status or suggests two open ranges. |
| `"Cancel my appointment on Friday at 2pm"` | Finds and deletes the event, confirms cancellation. |
| `"Reschedule my Monday 9am to Tuesday 11am"` | Finds original, checks new slot, moves event, confirms. |
| `"Book me at 3pm"` (no date) | Agent asks for the missing date before proceeding. |

