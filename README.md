# 🤖 AI Personal Assistant — n8n Workflow

An automated personal assistant built on [n8n](https://n8n.io) that handles voice messages, text commands, and automates tasks across a suite of Google Workspace integrations — all triggered through Telegram.

---

## 📐 Architecture Overview

```
Telegram Trigger
    ├── Voice Message Path
    │   ├── Download Voice File
    │   └── Transcribe Audio (Whisper)
    │
    └── Text Message Path
        └── Set Text
            │
            └──► AI Agent (OpenAI Chat Model)
                    │
                    ├── Structured Output Parser
                    │   ├── Get many events in Google Calendar
                    │   ├── Create an event in Google Calendar
                    │   ├── Update an event in Google Calendar
                    │   ├── Send a message in Gmail
                    │   ├── Append row in Google Sheets
                    │   └── Get many contacts in Google Contacts
                    │
                    ├── If1 ──► Send Audio (Telegram)
                    │       └── Switch1 (mode: Rules)
                    │
                    └── If2 ──► AI Agent1
                                ├── AI Transform1
                                ├── Code
                                ├── If
                                ├── Edit Fields (manual)
                                └── HTTP Request (POST)
```

---

## ✨ Features

| Feature | Description |
|---|---|
| 🎙️ Voice Input | Receives voice notes on Telegram, downloads and transcribes them via Whisper |
| 💬 Text Input | Accepts plain text commands through Telegram |
| 🧠 AI Reasoning | Central AI Agent (GPT-4) interprets intent and routes to the right tool |
| 📅 Calendar Management | Create, update, and fetch Google Calendar events |
| 📧 Gmail Integration | Send emails directly from a chat message |
| 📊 Google Sheets | Append data rows to spreadsheets |
| 👤 Google Contacts | Look up contacts from your Google address book |
| 🔁 Conditional Routing | Smart branching logic via `If` and `Switch` nodes |
| 🌐 HTTP Requests | Extensible webhook/API calls for custom integrations |

---

## 🛠️ Tech Stack

- **Workflow Engine** — [n8n](https://n8n.io)
- **Trigger** — Telegram Bot API
- **AI Model** — OpenAI GPT-4 (via OpenAI Chat Model node)
- **Speech-to-Text** — OpenAI Whisper
- **Google Services** — Calendar, Gmail, Sheets, Contacts (via OAuth2)
- **Custom Logic** — JavaScript Code node + AI Transform node

---

## 🚀 Getting Started

### Prerequisites

- n8n instance (self-hosted or cloud)
- Telegram Bot Token ([create one via BotFather](https://t.me/BotFather))
- OpenAI API Key
- Google OAuth2 credentials with scopes for Calendar, Gmail, Sheets, and Contacts

### Setup

1. **Import the workflow** into your n8n instance (import JSON via Editor → Import).
2. **Configure credentials**:
   - Add your Telegram Bot Token under the Telegram Trigger node.
   - Add your OpenAI API key under the OpenAI Chat Model node.
   - Connect Google OAuth2 for Calendar, Gmail, Sheets, and Contacts nodes.
3. **Activate the workflow** using the toggle in the top-right corner.
4. **Send a message** to your Telegram bot — text or voice — and watch it work.

---

## 💬 Example Commands

```
"Schedule a meeting with Ravi tomorrow at 3 PM"
"Send an email to team@example.com about the project update"
"What's on my calendar this week?"
"Add this contact: John Doe, john@example.com"
"Log today's sales: 12 units, ₹45,000"
```

---

## 🔧 Customization

- **Add new tools** to the AI Agent by connecting additional action nodes (e.g., Notion, Slack, Jira).
- **Modify the system prompt** in the AI Agent node to tune the assistant's personality or scope.
- **Extend routing logic** in the `Switch` and `If` nodes to handle more response types.
- **HTTP Request node** is pre-wired for custom webhook integrations — update the POST URL to your own endpoint.

---

## 📁 Repository Structure

```
.
├── README.md                  # This file
├── workflow.json              # Exported n8n workflow (import this)
└── docs/
    └── architecture.png       # Workflow screenshot / diagram
```

---

## 📄 License

MIT — free to use, modify, and distribute.

---

> Built with n8n · Powered by OpenAI · Integrated with Google Workspace
