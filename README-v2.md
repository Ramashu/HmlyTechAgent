# HmlyTech Agent Platform

A production-ready SaaS platform for creating, managing, and deploying AI agents via API — similar to app.emergent.sh.

## 🧱 Tech Stack

| Layer     | Technology                       |
|-----------|----------------------------------|
| Frontend  | Next.js 15, TypeScript, Tailwind, Zustand |
| Backend   | FastAPI, SQLAlchemy (async), PostgreSQL |
| Auth      | JWT + Google OAuth              |
| AI        | OpenAI GPT-4o, Anthropic Claude |
| Infra     | Docker, Vercel, Railway/AWS     |

---

## 🚀 Quick Start (Docker — recommended)

### 1. Clone & configure

```bash
git clone <repo>
cd hmlytech
cp .env.example .env
# Edit .env — fill in OPENAI_API_KEY, JWT_SECRET, etc.
```

### 2. Start everything

```bash
docker-compose up --build
```

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- API Docs (Swagger): http://localhost:8000/docs

---

## 🛠️ Local Development

### Backend

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env — minimum required: DATABASE_URL, JWT_SECRET, OPENAI_API_KEY

# Start PostgreSQL (or use Docker just for DB)
docker run -d \
  --name hmlytech_pg \
  -e POSTGRES_DB=hmlytech \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  postgres:16-alpine

# Run the API server
uvicorn app.main:app --reload --port 8000
```

### Frontend

```bash
cd frontend

# Install dependencies
npm install

# Configure environment
cp .env.example .env.local
# Set NEXT_PUBLIC_API_URL=http://localhost:8000

# Run dev server
npm run dev
```

Visit http://localhost:3000

---

## 🔐 Environment Variables

### Backend (`backend/.env`)

| Variable               | Required | Description                              |
|------------------------|----------|------------------------------------------|
| `DATABASE_URL`         | ✅       | PostgreSQL async URL                     |
| `JWT_SECRET`           | ✅       | Long random string for JWT signing       |
| `OPENAI_API_KEY`       | ✅       | OpenAI API key                           |
| `ANTHROPIC_API_KEY`    | ⬜       | For Claude models                        |
| `GOOGLE_CLIENT_ID`     | ⬜       | For Google OAuth                         |
| `GOOGLE_CLIENT_SECRET` | ⬜       | For Google OAuth                         |
| `FRONTEND_URL`         | ⬜       | Frontend URL for redirects               |

### Frontend (`frontend/.env.local`)

| Variable                  | Required | Description           |
|---------------------------|----------|-----------------------|
| `NEXT_PUBLIC_API_URL`     | ✅       | Backend API base URL  |

---

## 📁 Project Structure

```
hmlytech/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app entry point
│   │   ├── core/
│   │   │   ├── config.py        # Settings (pydantic-settings)
│   │   │   ├── database.py      # SQLAlchemy async engine
│   │   │   └── security.py      # JWT, password hashing
│   │   ├── models/
│   │   │   ├── user.py          # User ORM model
│   │   │   ├── agent.py         # Agent, AgentVersion, Execution models
│   │   │   └── memory.py        # Memory/conversation history
│   │   ├── schemas/
│   │   │   ├── auth.py          # Pydantic request/response schemas
│   │   │   └── agent.py         # Agent schemas
│   │   ├── services/
│   │   │   ├── auth_service.py      # Register, login, Google OAuth
│   │   │   ├── agent_service.py     # Agent CRUD + versioning
│   │   │   ├── execution_service.py # OpenAI/Claude integration + streaming
│   │   │   ├── tools_service.py     # HTTP, web search, calculator tools
│   │   │   └── memory_service.py    # Conversation memory
│   │   └── routes/
│   │       ├── auth.py          # /api/auth/*
│   │       ├── agents.py        # /api/agents/*
│   │       ├── execution.py     # /api/agent/{id}/run
│   │       ├── tools.py         # /api/tools
│   │       └── memory.py        # /api/memory/*
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
│
├── frontend/
│   ├── app/
│   │   ├── layout.tsx           # Root layout (fonts, Toaster)
│   │   ├── globals.css          # Tailwind + custom CSS
│   │   ├── page.tsx             # Landing page
│   │   ├── login/page.tsx       # Login
│   │   ├── register/page.tsx    # Register
│   │   ├── auth/callback/       # Google OAuth callback
│   │   ├── dashboard/
│   │   │   ├── layout.tsx       # Protected layout + Sidebar
│   │   │   ├── page.tsx         # Overview / stats
│   │   │   ├── agents/
│   │   │   │   ├── page.tsx     # Agent list
│   │   │   │   ├── new/         # Create agent
│   │   │   │   └── [id]/        # Agent detail / edit / API docs
│   │   │   ├── playground/      # Chat interface
│   │   │   └── settings/        # API key + account
│   │   └── components/
│   │       ├── layout/Sidebar.tsx
│   │       └── agent/AgentForm.tsx
│   ├── lib/
│   │   ├── api/client.ts        # Axios client + typed API methods
│   │   ├── store/
│   │   │   ├── auth.ts          # Zustand auth store
│   │   │   └── agents.ts        # Zustand agents store
│   │   └── utils.ts             # cn(), formatters, constants
│   ├── package.json
│   ├── tailwind.config.js
│   ├── next.config.js
│   ├── Dockerfile
│   └── .env.example
│
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 🧩 Features

### Auth System
- Email/password registration & login
- Google OAuth 2.0
- JWT access token (1 hour) + refresh token (30 days)
- Auto-refresh on 401 responses
- Per-user API key management

### Agent System
- Create agents with name, description, goal, system prompt
- Model selection: GPT-4o, GPT-4o-mini, GPT-4 Turbo, Claude 3.5 Sonnet, Claude Haiku, etc.
- Temperature & max tokens configuration
- Public/private visibility
- Tag system
- Automatic version history on prompt/model changes

### Execution Engine
- Full conversation context support
- Session-based memory
- Tool calling (HTTP request, web search, calculator)
- Token usage and latency tracking
- Execution history per agent

### Tools
| Tool         | Description                               |
|--------------|-------------------------------------------|
| HTTP Request | Make GET/POST/PUT/DELETE to any URL       |
| Web Search   | DuckDuckGo-powered search                 |
| Calculator   | Safe math expression evaluator            |

### API Access
Every agent exposes:
```
POST /api/agent/{id}/run/public
Header: X-API-Key: your_key
Body: { "input": "...", "session_id": "..." }
```

---

## 🚢 Deployment

### Frontend → Vercel
```bash
cd frontend
npx vercel deploy --prod
# Set env: NEXT_PUBLIC_API_URL=https://your-backend.railway.app
```

### Backend → Railway
1. Connect GitHub repo
2. Set root directory to `backend/`
3. Add environment variables from `.env.example`
4. Railway auto-detects Python + provides PostgreSQL

### Backend → AWS / Any VPS
```bash
# On server:
docker-compose up -d --build

# With nginx reverse proxy:
# frontend: proxy_pass http://localhost:3000
# backend:  proxy_pass http://localhost:8000
```

---

## 📖 API Documentation

Interactive Swagger UI available at:
```
http://localhost:8000/docs
```

ReDoc available at:
```
http://localhost:8000/redoc
```

---

## 🔑 Creating Your First Agent (API)

```bash
# 1. Register
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com","username":"yourname","password":"secret123"}'

# 2. Create agent (use access_token from above)
curl -X POST http://localhost:8000/api/agents \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Assistant",
    "system_prompt": "You are a concise and helpful assistant.",
    "model": "gpt-4o",
    "is_public": true
  }'

# 3. Run agent
curl -X POST http://localhost:8000/api/agent/1/run/public \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"input": "What is the capital of France?"}'
```

---

## 👥 Üye & Admin Sistemi (v2)

### Roller

| Rol | Yetkiler |
|-----|----------|
| `super_admin` | Tam yetki. Kullanıcı sil, rol ata, tüm ayarları düzenle |
| `admin` | Kullanıcı rol/durum/plan güncelle, agent yönet, ayar güncelle |
| `plus` | Genişletilmiş agent limiti (25). Super admin tarafından atanır |
| `member` | Standart üyelik (5 agent). Kayıt olunca otomatik atanır |

### Varsayılan Admin Hesabı

İlk başlatmada otomatik oluşturulur:
```
E-posta : admin@hmlytech.com
Şifre   : admin123
```
> **Önemli:** Production'da `.env` dosyasından değiştirin!

### Paneller

| URL | Açıklama | Erişim |
|-----|----------|--------|
| `/member/dashboard` | Üye genel bakış | Tüm üyeler |
| `/member/profile`   | Profil & şifre & API anahtarı | Tüm üyeler |
| `/admin/dashboard`  | Platform istatistikleri | Admin + |
| `/admin/users`      | Kullanıcı yönetimi | Admin + |
| `/admin/agents`     | Agent yönetimi | Admin + |
| `/admin/settings`   | Site içerik düzenleyici | Admin + |

### E-posta Doğrulama

Kayıt sırasında:
1. Kullanıcı kayıt formu doldurur
2. Sahte/geçici e-posta kontrolü yapılır (50+ domain engelli)
3. `mail@hmlytech.com` adresinden hoşgeldiniz + doğrulama linki gönderilir
4. Link 24 saat geçerlidir
5. Doğrulama yapılmadan giriş yapılamaz

### SMTP Yapılandırması

Gmail için:
```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=mail@hmlytech.com
SMTP_PASS=xxxx-xxxx-xxxx-xxxx  # Google App Password
SMTP_TLS=True
```

Zoho / Mailgun / SendGrid de desteklenir. SMTP ayarı yapılmadan platform çalışmaya devam eder, sadece e-postalar gönderilmez.
