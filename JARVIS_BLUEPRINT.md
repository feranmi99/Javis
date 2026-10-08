# 🤖 PROJECT JARVIS — Personal Autonomous AI Companion & Executive Assistant
*Architecture Blueprint, Feature Specification & Development Roadmap*

---

## 1. Vision & Core Philosophy

**Jarvis** is not just a chatbot you prompt; it is a **proactive, 24/7 personal executive assistant and companion**.
- **Always Active:** Lives on your local machine (with **Ollama**) during testing, then moves to a 24/7 Cloud VPS.
- **Zero Frontend Required:** Your primary interface is **WhatsApp** and **Email**, with optional voice note processing.
- **Proactive, Not Just Reactive:** Jarvis doesn't just wait for you to ask questions. It reaches out, monitors your schedule, alerts you of emergencies, and checks in on your well-being.
- **The Gatekeeper:** It shields your time by filtering noise, handling routine inquiries on WhatsApp, and escalating only what truly matters.

---

## 2. 🌟 Unique Features: What Makes Your Jarvis Truly Special

To make Jarvis feel like a real human assistant rather than a generic bot, here are 8 unique features designed for this project:

### 🎙️ 1. WhatsApp Voice Note Processing (Bi-directional Voice)
- **Listen:** When people (or you) send a WhatsApp voice note, Jarvis runs it through local/fast speech-to-text (`faster-whisper` or Groq Whisper).
- **Speak Back:** Jarvis can reply via realistic audio using free Microsoft Edge-TTS (`edge-tts`). You can literally talk to Jarvis while driving or walking and hear him talk back like the real Iron Man Jarvis.

### 🛡️ 2. "VIP Gatekeeper" & Context-Aware Auto-Reply
- Jarvis maintains a list of **Relationship Tiers**:
  - **Tier 1 (VIP / Family / Significant Other):** Never blocked; instant notification to you, gentle automated updates if you are genuinely unreachable.
  - **Tier 2 (Clients / Colleagues):** Polite executive tone: *"Hello! [Your Name] is currently offline / in a deep-work block. I have logged your message and notified him. If this is urgent, reply with 'URGENT'."*
  - **Tier 3 (Unknown / Group Chats):** Filtered or silently summarized in your daily digest.

### ☀️ 3. The 7:30 AM "Morning Briefing"
Every morning, Jarvis compiles a WhatsApp message:
- 📅 **Today's Agenda:** Meetings and Google Calendar events.
- 📩 **Inbox Triage:** "You received 8 emails last night. 6 are newsletters/spam (archived). 2 require your attention: [Summary]."
- 🎯 **Top 3 Priorities:** Reminds you of the tasks you set yesterday.
- ⛅ **Quick Weather / Quote / Morning Motivation.**

### 🌙 4. "Night Watch" & Sleep Protection
- Between 11:00 PM and 6:00 AM, Jarvis enters **Night Watch Mode**.
- Routine pings are silenced. 
- If an emergency condition is detected (e.g. key server goes down, VIP sends an urgent email, or someone triggers an emergency keyword), Jarvis can ring your phone or send high-priority alerts.

### 🧠 5. Long-Term Episodic Memory (Personal Life Log)
- Most bots forget you after a chat session. Jarvis has a persistent memory database (SQLite + Vector Store):
  - Remembers friends' names, birthdays, your food preferences, ongoing project milestones, and promises you made ("*I promised to send John the invoice by Friday*").
  - Periodic Memory Review: *"Hey, you promised to follow up with Sarah today about the logistics contract. Did you get that done?"*

### 🧘 6. Proactive Check-in & Mood Sensing
- If it's 2:00 PM on a workday and you haven't opened WhatsApp or responded to tasks, Jarvis pings:
  > *"Boss, noticed you've been off the radar today. Everything good? Taking a breather or stuck in meetings?"*
- Analyzes tone in conversations to understand your stress level and adapt its responses.

### ⚡ 7. Quick Natural Language Actions (Tool Calling)
You can text Jarvis quick instructions on WhatsApp like:
- *"Remind me to call Mom in 2 hours"*
- *"Email Dave that I will join the meeting 10 mins late"*
- *"Summarize the contract PDF attached to that email from yesterday"*
- *"Log expense: 15,000 for groceries"*

### 🔍 8. Automatic Email Draft & Approval Flow
- Instead of auto-sending dangerous emails, Jarvis drafts them and sends a WhatsApp preview with two quick buttons or reply triggers:
  > *"Client asked for a quote. I drafted a response: [Preview]. Reply 'SEND' to fire it, or tell me what to change."*

---

## 3. Technology Stack & Architecture

```
                    ┌────────────────────────────────────────┐
                    │               JARVIS CORE              │
                    │         (Python / FastAPI / Async)     │
                    └────┬──────────────┬──────────────┬─────┘
                         │              │              │
        ┌────────────────┴┐       ┌─────┴─────┐       ┌┴─────────────────┐
        ▼                 ▼       ▼           ▼       ▼                  ▼
┌──────────────┐   ┌────────────┐ ┌──────┐ ┌──────┐ ┌─────────────┐ ┌──────────────┐
│  Ollama      │   │ Cloud LLM  │ │SQLite│ │Chroma│ │ WhatsApp    │ │ Gmail / IMAP │
│(Llama 3/Mist)│   │(Groq/Gemini│ │ (DB) │ │ (Mem)│ │ (Baileys /  │ │   (Emails)   │
│  [LOCAL DEV] │   │  [LIVE]    │ └──────┘ └──────┘ │ WPPConnect) │ └──────────────┘
└──────────────┘   └────────────┘                   └─────────────┘
```

### Why This Tech Stack?

1. **Backend Core:** **Python 3.11+**
   - Best ecosystem for AI agents, LangChain/LiteLLM, tool calling, and automation.
   - Framework: **FastAPI** (for webhooks, asynchronous background tasks, and clean APIs).

2. **LLM Provider (Local vs Live Swappable):**
   - **Local Development:** **Ollama** running `llama3.2` or `mistral` / `qwen2.5`.
     - Cost: **$0.00**
     - Completely private and offline.
   - **Live Production:** **LiteLLM / OpenAI / Gemini / Groq API**
     - Fast inference (Groq gives ~300 tokens/sec for pennies).
     - Single configuration switch (`LLM_PROVIDER=ollama` vs `LLM_PROVIDER=groq`).

3. **WhatsApp Service (Node.js microservice or WPPConnect / Baileys):**
   - Runs a lightweight WhatsApp Web connection.
   - Communicates with your Python Jarvis core via local HTTP REST/Webhooks.
   - Handles multi-device QR pairing once, then stays connected permanently.

4. **Background Scheduler:**
   - **APScheduler (Advanced Python Scheduler)** or **Celery / Celery-Beat**.
   - Handles morning routines, periodic email inbox polling (every 5 mins), and reminder timeouts.

5. **Audio / Speech (Zero Extra Cost):**
   - Speech-to-Text: `faster-whisper` (runs locally) or Groq Whisper (free tier).
   - Text-to-Speech: `edge-tts` (high-quality Microsoft neural voices, 100% free, no API key needed).

---

## 4. Project Directory Structure

```
jarvis-assistant/
├── README.md
├── requirements.txt
├── .env.example
├── config.py                 # Switches between Ollama & Cloud APIs
│
├── core/                     # The AI Brain
│   ├── __init__.py
│   ├── agent.py              # Main Agent loop & prompt orchestration
│   ├── memory.py             # SQLite + Vector store for user facts & history
│   ├── tools.py              # Functions Jarvis can execute (send email, set reminder, etc.)
│   └── prompts.py            # Personality & System prompts
│
├── services/                 # External Integrations
│   ├── __init__.py
│   ├── llm_client.py         # Unified wrapper for Ollama / Groq / OpenAI
│   ├── whatsapp.py           # WhatsApp webhook receiver & sender
│   ├── email_service.py      # IMAP/Gmail API fetcher & SMTP sender
│   ├── voice_service.py      # Whisper STT & Edge-TTS audio synthesizer
│   └── scheduler.py          # Morning briefing, night watch, cron tasks
│
├── whatsapp_bridge/          # Lightweight Baileys/Node connector
│   ├── package.json
│   └── server.js             # Connects to WhatsApp & forwards to Python
│
├── storage/                  # Persistent data
│   ├── jarvis.db             # SQLite (contacts, messages, reminders)
│   └── vector_store/         # Long-term knowledge memories
│
└── main.py                   # FastAPI entrypoint & background runners
```

---

## 5. Development Phases: Local Test ➡️ Cloud Deployment

### 🔹 Phase 1: Local Brain & Email (Ollama + Python)
1. Initialize Python environment and configure `services/llm_client.py` with your local Ollama.
2. Build email reader (`services/email_service.py`) using Gmail App Passwords.
3. Test Jarvis reading an email, summarizing it, and writing a draft reply on your local terminal.

### 🔹 Phase 2: WhatsApp Link (Local Testing)
1. Set up the `whatsapp_bridge` with Baileys.
2. Scan the QR code once on your phone.
3. Send a test message from another phone or your own note-to-self chat to Jarvis.
4. Have Jarvis respond using Ollama.

### 🔹 Phase 3: Proactive Scheduler & Audio
1. Add `APScheduler` for morning briefings and reminder tasks.
2. Add voice note transcription with Whisper and Edge-TTS audio voice replies.

### 🔹 Phase 4: Deploying 24/7 to Cloud VPS
1. Pick a cheap $4–$5/month Linux VPS (Hetzner, DigitalOcean, or Linode).
2. Install Docker or run with `systemd` / `pm2`.
3. Switch `LLM_PROVIDER` in `.env` to Groq or Gemini API (since a small cloud VPS doesn't have a GPU to run large Ollama models; cloud APIs are faster and cheap).
4. Scan the WhatsApp QR once on the VPS, and Jarvis is live **24 hours a day, 7 days a week** without needing your laptop!
