# 📄 Файл 1: `.env.example`

```bash
# ============================================================================
# PARTSCOUT - Environment Variables Template
# ============================================================================
# Скопируйте этот файл в .env и заполните реальными значениями
# Команда: cp .env.example .env
# ============================================================================

# ----------------------------------------------------------------------------
# AI & LANGUAGE MODELS
# ----------------------------------------------------------------------------

# Gemini API (основной AI)
# Получить: https://ai.google.dev/
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-2.0-flash-exp

# Embeddings для RAG
GEMINI_EMBEDDING_MODEL=text-embedding-004

# ----------------------------------------------------------------------------
# DATABASE & STORAGE (Supabase)
# ----------------------------------------------------------------------------

# Supabase Project
# Получить: https://supabase.com/dashboard
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_KEY=your_supabase_service_role_key

# Database Connection (для прямого доступа если нужен)
SUPABASE_DB_HOST=db.your-project.supabase.co
SUPABASE_DB_PORT=5432
SUPABASE_DB_NAME=postgres
SUPABASE_DB_USER=postgres
SUPABASE_DB_PASSWORD=your_database_password

# ----------------------------------------------------------------------------
# TELEGRAM BOTS
# ----------------------------------------------------------------------------

# Bot для менеджера
# Получить токен: https://t.me/BotFather
TELEGRAM_MANAGER_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz
TELEGRAM_MANAGER_CHAT_ID=your_manager_telegram_chat_id

# Bot для B2B сети (Phase 2)
TELEGRAM_B2B_BOT_TOKEN=
TELEGRAM_B2B_ADMIN_CHAT_ID=

# ----------------------------------------------------------------------------
# BELARUS AUTO PARTS CATALOGS
# ----------------------------------------------------------------------------

# auto1.by API
AUTO1_BY_API_URL=https://api.auto1.by/v1
AUTO1_BY_LOGIN=your_login
AUTO1_BY_PASSWORD=your_password
AUTO1_BY_API_KEY=

# armtek.by API
ARMTEK_BY_API_URL=https://api.armtek.by
ARMTEK_BY_API_KEY=your_armtek_api_key
ARMTEK_BY_LOGIN=
ARMTEK_BY_PASSWORD=

# zapchasti24.by (если будет API или для парсинга)
ZAPCHASTI24_BY_URL=https://zapchasti24.by

# ----------------------------------------------------------------------------
# n8n AUTOMATION
# ----------------------------------------------------------------------------

# n8n Configuration
N8N_HOST=n8n.partscout.local
N8N_PORT=5678
N8N_PROTOCOL=https

# n8n Database (использует Supabase PostgreSQL)
N8N_DB_TYPE=postgresdb
N8N_DB_POSTGRESDB_HOST=${SUPABASE_DB_HOST}
N8N_DB_POSTGRESDB_PORT=${SUPABASE_DB_PORT}
N8N_DB_POSTGRESDB_DATABASE=n8n
N8N_DB_POSTGRESDB_USER=${SUPABASE_DB_USER}
N8N_DB_POSTGRESDB_PASSWORD=${SUPABASE_DB_PASSWORD}

# n8n Security
N8N_ENCRYPTION_KEY=your_random_32_char_encryption_key_here
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_secure_password

# Webhooks
N8N_WEBHOOK_URL=https://n8n.partscout.local/

# ----------------------------------------------------------------------------
# FASTAPI APPLICATION
# ----------------------------------------------------------------------------

# FastAPI Server
FASTAPI_HOST=0.0.0.0
FASTAPI_PORT=8000
FASTAPI_WORKERS=4
FASTAPI_RELOAD=false

# API Settings
API_V1_PREFIX=/api/v1
API_TITLE=PartScout API
API_DEBUG=false

# CORS (если нужен веб-интерфейс)
API_CORS_ORIGINS=["http://localhost:3000", "https://app.partscout.local"]

# ----------------------------------------------------------------------------
# CLOUDFLARE TUNNEL
# ----------------------------------------------------------------------------

# Cloudflare Tunnel Token
# Получить: https://one.dash.cloudflare.com/
CLOUDFLARE_TUNNEL_TOKEN=your_cloudflare_tunnel_token

# Tunnel Configuration (для разных сервисов)
TUNNEL_N8N_HOSTNAME=n8n.partscout.yourdomain.com
TUNNEL_API_HOSTNAME=api.partscout.yourdomain.com
TUNNEL_BROWSER_HOSTNAME=browser.partscout.yourdomain.com

# ----------------------------------------------------------------------------
# MCP SERVERS CONFIGURATION
# ----------------------------------------------------------------------------

# Пути к MCP серверам (для Gemini CLI)
MCP_CATALOG_SERVER_PATH=./services/catalog/mcp_server.py
MCP_BROWSER_SERVER_PATH=./services/scraper/mcp_server.py
MCP_ANALYSIS_SERVER_PATH=./services/ai-agent/mcp_server.py
MCP_RAG_SERVER_PATH=./services/rag/mcp_server.py
MCP_ASSISTED_SEARCH_PATH=./services/assisted-search/mcp_server.py
MCP_MANUAL_MODE_PATH=./services/manual-mode/mcp_server.py

# ----------------------------------------------------------------------------
# BROWSER AUTOMATION (Playwright)
# ----------------------------------------------------------------------------

# Playwright Settings
PLAYWRIGHT_HEADLESS=true
PLAYWRIGHT_BROWSER=chromium
PLAYWRIGHT_TIMEOUT=30000
PLAYWRIGHT_USER_AGENT=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36

# Прокси (опционально, для защиты от блокировок)
PLAYWRIGHT_PROXY_SERVER=
PLAYWRIGHT_PROXY_USERNAME=
PLAYWRIGHT_PROXY_PASSWORD=

# ScrapingBee (резервный вариант для сложных сайтов)
SCRAPINGBEE_API_KEY=

# ----------------------------------------------------------------------------
# GOOGLE DRIVE (для RAG)
# ----------------------------------------------------------------------------

# Google Drive API
# Получить: https://console.cloud.google.com/
GOOGLE_DRIVE_CREDENTIALS_PATH=./credentials/google-drive-credentials.json
GOOGLE_DRIVE_PRICELISTS_FOLDER_ID=your_folder_id_for_pricelists
GOOGLE_DRIVE_KNOWLEDGE_FOLDER_ID=your_folder_id_for_knowledge_base

# ----------------------------------------------------------------------------
# EMAIL NOTIFICATIONS (опционально)
# ----------------------------------------------------------------------------

# SMTP Settings (для отправки отчетов на email)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_specific_password
SMTP_FROM_EMAIL=partscout@yourdomain.com
SMTP_FROM_NAME=PartScout System

# ----------------------------------------------------------------------------
# REDIS (опционально для кэша и очередей)
# ----------------------------------------------------------------------------

# Redis Connection
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0

# Cache TTL (в секундах)
CACHE_PRICES_TTL=21600  # 6 часов
CACHE_CATALOG_TTL=86400  # 24 часа

# ----------------------------------------------------------------------------
# LOGGING & MONITORING
# ----------------------------------------------------------------------------

# Log Level
LOG_LEVEL=INFO  # DEBUG, INFO, WARNING, ERROR, CRITICAL

# Sentry (опционально, для отслеживания ошибок)
SENTRY_DSN=

# ----------------------------------------------------------------------------
# DEVELOPMENT & TESTING
# ----------------------------------------------------------------------------

# Environment
ENVIRONMENT=development  # development, staging, production

# Debug Mode
DEBUG=true

# Testing
TEST_MODE=false
TEST_SUPPLIER_URL=https://example.com/test-page

# ----------------------------------------------------------------------------
# SECURITY
# ----------------------------------------------------------------------------

# Secret Keys
SECRET_KEY=your_random_secret_key_min_32_characters
JWT_SECRET_KEY=your_jwt_secret_key_for_api_auth

# API Rate Limiting
RATE_LIMIT_PER_MINUTE=60

# ----------------------------------------------------------------------------
# FEATURE FLAGS (для постепенного включения функций)
# ----------------------------------------------------------------------------

# Phase 1 Features
FEATURE_CATALOG_INTEGRATION=true
FEATURE_BROWSER_PARSING=true
FEATURE_AI_ANALYSIS=true
FEATURE_PDF_REPORTS=true
FEATURE_TELEGRAM_NOTIFICATIONS=true

# Phase 2 Features (выключены для MVP)
FEATURE_B2B_NETWORK=false
FEATURE_VECTOR_SEARCH=false
FEATURE_PRICE_ANALYTICS=false
FEATURE_MOBILE_APP=false

# Experimental Features
FEATURE_ASSISTED_SEARCH=true
FEATURE_MANUAL_MODE=true
FEATURE_RAG_LEARNING=true

# ----------------------------------------------------------------------------
# BUSINESS LOGIC
# ----------------------------------------------------------------------------

# Поиск поставщиков
MAX_SUPPLIERS_TO_CHECK=10
PARALLEL_PARSING_LIMIT=5
PARSING_TIMEOUT_SECONDS=30

# Критерии анализа
PRIORITY_IN_STOCK=true
PRIORITY_LOCAL_SUPPLIERS=true
MAX_DELIVERY_DAYS=7

# B2B сеть (Phase 2)
B2B_BROADCAST_TIMEOUT_MINUTES=5
B2B_MAX_RADIUS_KM=60

# ============================================================================
# END OF CONFIGURATION
# ============================================================================

# ВАЖНО: Не коммитьте файл .env в Git!
# Файл .env должен быть в .gitignore
```

---

## 📌 Комментарий к файлу

### Структура:

1. **Четкие секции** - каждый блок переменных логически сгруппирован
2. **Комментарии** - где получить ключи, для чего нужно
3. **Defaults** - разумные значения по умолчанию где возможно
4. **Feature Flags** - возможность включать/выключать функции
5. **Бизнес-логика** - настройки поведения системы

### Особенности:

- ✅ Учтены все обсуждаемые компоненты
- ✅ Белорусские каталоги (auto1.by, armtek.by)
- ✅ MCP серверы (пути к ним)
- ✅ Gemini CLI совместимость
- ✅ Cloudflare Tunnel для каждого сервиса
- ✅ Feature flags для поэтапного запуска
- ✅ Гибридный режим (assisted search, manual mode)

---

## 🔖 Git Commit

```bash
git add .env.example
git commit -m "feat: add comprehensive environment variables template

- Add AI/ML configuration (Gemini API)
- Add Supabase database and storage settings
- Add Telegram bots configuration (manager + B2B)
- Add Belarus catalogs integration (auto1.by, armtek.by)
- Add n8n automation variables
- Add FastAPI application settings
- Add Cloudflare Tunnel configuration
- Add MCP servers paths for Gemini CLI
- Add browser automation (Playwright) settings
- Add Google Drive integration for RAG
- Add feature flags for phased rollout
- Add business logic parameters

This template provides all necessary environment variables
for development, staging, and production environments.
Copy to .env and fill with actual values."
```

---

## ✅ Готово!

**Файл создан:** `.env.example`

**Следующий файл:** `docker-compose.yml`

Готов создавать следующий файл? 🚀