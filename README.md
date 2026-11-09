# Project Jarvis

**Personal Autonomous AI Companion & Executive Assistant (v0.1 / v0.2 Local Brain & Task Engine)**

Jarvis is a private, proactive AI executive assistant designed with a strict security boundary:
> **The LLM is intelligent, but it is never the authority.**

All permissions, memory persistence, reminders, single-use action approvals, and audit logs are enforced deterministically by the Python/FastAPI application layer.

---

## Architecture & Directory Structure

```text
├── config.py                 # Validated environment configuration (localhost-bound in dev)
├── main.py                   # FastAPI application & REST endpoints
├── core/
│   ├── agent.py              # 13-step Jarvis Agent orchestration loop
│   ├── db.py                 # SQLite persistence & repositories (WAL mode)
│   ├── permissions.py        # Tiered permission engine (Level 1/2/3) & single-use approvals
│   ├── prompts.py            # System prompt, dynamic grounding & untrusted data wrapping
│   └── tools.py              # Pydantic tool schemas & execution handlers
├── services/
│   ├── llm_client.py         # Provider-agnostic LLM client (Ollama adapter)
│   └── scheduler.py          # APScheduler maintenance & due-reminder notifications
├── storage/                  # Local SQLite database directory (gitignored)
├── tests/
│   └── test_api.py           # Automated test suite (auth, memory, reminders, approvals, tools)
└── whatsapp_bridge/          # Phase 3 Node.js/Baileys WhatsApp bridge skeleton
```

---

## Quickstart

### 1. Create Virtual Environment & Install Dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Configure Environment

```bash
cp .env.example .env
```

Generate a strong `API_KEY` for `.env`:

```bash
python3 -c 'import secrets; print(secrets.token_urlsafe(32))'
```

### 3. Ensure Ollama is Running

```bash
ollama serve
ollama pull llama3.2
```

### 4. Start Jarvis

```bash
uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```

### 5. Run Automated Tests

```bash
pytest -v
```
