# AI Mail Agent

**An AI-powered email automation system using Google Gemini, FastAPI, and Gmail SMTP**

A small Python microservices app where users describe what they want to email, the agent drafts a professional message, and the mail service delivers it through Gmail SMTP.

**Author**: Kalash Sharma
**Version**: 3.0.0

> **v3.0.0** — Python-only stack with a FastAPI agent service, FastAPI mail service, and a static frontend.

---

## 📊 Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                         Frontend                             │
│                   Static UI on Port 3000                     │
└────────────────────┬─────────────────────────────────────────┘
                     │ HTTP POST /chat
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                     Email Agent                              │
│        FastAPI + Gemini + conversation/session state         │
│                    Port 8000                                 │
└────────────────────┬─────────────────────────────────────────┘
                     │ HTTP POST /api/email
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                     Mail Service                             │
│             FastAPI + smtplib + Gmail SMTP                  │
│                    Port 8080                                 │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     ▼
              Gmail SMTP Server
```

## ✨ Features

- **🤖 AI email drafting** — Gemini generates polished email copy from natural language prompts
- **📬 Direct SMTP delivery** — The mail service sends messages through Gmail using `smtplib`
- **💬 Conversational flow** — The agent can ask follow-up questions before drafting
- **🔁 Fallback support** — If Gemini is rate-limited, the agent can fall back to Ollama
- **📚 Interactive docs** — FastAPI Swagger docs available for both backend services
- **🐳 Docker Compose ready** — One command can start the frontend and both APIs

## 📋 Prerequisites

| Component | Recommended | Purpose |
|-----------|-------------|---------|
| **Python** | 3.10+ | Local development and container runtime |
| **Docker + Compose** | Latest stable | Easiest way to run the full stack |
| **Gmail account** | With 2FA enabled | SMTP delivery |
| **Gmail App Password** | Required | Auth for SMTP login |
| **Gemini API key** | Required | Email drafting model access |

## 🚀 Quick start with Docker Compose

1. Copy `.env.example` to `.env` in the project root and fill in your credentials.
2. Start the full stack:

```bash
docker compose up
```

3. Open the apps:

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Email Agent Swagger | http://localhost:8000/docs |
| Mail Service Swagger | http://localhost:8080/docs |

> The compose file uses the Python base image for each service, installs the service requirements on startup, and mounts the local source code for easy iteration.

## 🛠️ Manual local setup

### 1. Get credentials

- **Gmail App Password**: enable 2FA, then create an app password in your Google account
- **Gemini API Key**: generate one from [Google AI Studio](https://aistudio.google.com/apikey)

### 2. Configure environment

Root `.env` variables used by Docker Compose:

```dotenv
GOOGLE_API_KEY=your_gemini_api_key
MAIL_USERNAME=your_email@gmail.com
MAIL_PASSWORD=your_app_specific_password
MAIL_SERVICE_URL=http://localhost:8080
APP_ENV=development
LOG_LEVEL=INFO
```

### 3. Run services separately

**Mail service**

```bash
cd mail-service-python
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8080
```

**Email agent**

```bash
cd EmailAgent
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000
```

**Frontend**

```bash
cd frontend
python server.py
```

## 📁 Project structure

```text
Ai-MailAgent/
├── README.md
├── docker-compose.yml
├── .env.example
├── EmailAgent/
│   ├── main.py
│   ├── requirements.txt
│   └── .env.example
├── mail-service-python/
│   ├── main.py
│   └── requirements.txt
└── frontend/
    ├── index.html
    ├── script.js
    ├── style.css
    └── server.py
```

## 🔌 API reference

### POST `/chat` — Compose and send email

Send a prompt and the agent writes a professional email and delivers it.

**Request:**
```bash
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{
    "email": "recipient@example.com",
    "content": "Thank them for attending the meeting and confirm next steps",
    "words": 200
  }'
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `email` | string | *required* | Recipient email address |
| `content` | string | *required* | What the email should be about |
| `words` | int | `150` | Minimum word count |

**Response:**
```json
{
  "status": "Email Sent",
  "service_response": {
    "success": true,
    "data": "Email sent successfully"
  }
}
```

### POST `/api/email` — Direct email delivery

Send an email directly without AI composition.

```bash
curl -X POST http://localhost:8080/api/email \
  -H "Content-Type: application/json" \
  -d '{
    "to": "recipient@example.com",
    "subject": "Meeting Confirmation",
    "body": "Hi, confirming our meeting tomorrow at 2pm."
  }'
```

### Interactive documentation

| Service | URL |
|---------|-----|
| AI Agent Swagger | http://localhost:8000/docs |
| Mail Service Swagger | http://localhost:8080/docs |

## ⚙️ Environment Variables

| Variable | Service | Required | Description |
|----------|---------|----------|-------------|
| `GOOGLE_API_KEY` | EmailAgent | ✅ | Gemini API key used by the agent |
| `MAIL_USERNAME` | mail-service-python | ✅ | Gmail address used for SMTP login |
| `MAIL_PASSWORD` | mail-service-python | ✅ | Gmail app-specific password |
| `MAIL_SERVICE_URL` | EmailAgent | ❌ | Mail service URL; Docker Compose sets this to `http://mail-service:8080` |
| `APP_ENV` | Both backend services | ❌ | App mode, defaults to `development` |
| `LOG_LEVEL` | Both backend services | ❌ | Logging verbosity, defaults to `INFO` |

## 🧰 Tech stack

### EmailAgent

- FastAPI 0.135.1
- Uvicorn 0.41.0
- LangChain Core 0.3.86
- LangChain Google GenAI 4.2.1
- LangChain Ollama 0.3.3
- Pydantic 2.12.5
- httpx 0.28.1
- python-dotenv 1.2.2

### Mail service

- FastAPI 0.115.0
- Uvicorn 0.30.0
- python-dotenv 1.0.1
- Built-in `smtplib` for Gmail delivery

## 🔍 Troubleshooting

| Problem | Solution |
|---------|----------|
| `GOOGLE_API_KEY` missing | Create a root `.env` file or export the variable before starting the services |
| SMTP authentication failed | Use a Gmail [App Password](https://myaccount.google.com/apppasswords), not your normal password |
| Connection refused to mail service | Make sure `mail-service-python` is running on port 8080 first |
| Frontend cannot reach the agent | Confirm the agent is running on `http://localhost:8000` and that CORS is enabled |
| Module not found errors | Reinstall the requirements inside the correct virtual environment |

---