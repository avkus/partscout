# 📄 Файл 3: `Makefile`

```makefile
# ============================================================================
# PARTSCOUT - Development Makefile
# ============================================================================
# Упрощённые команды для разработки и деплоя проекта
# 
# Использование:
#   make help          - показать все доступные команды
#   make dev           - запустить development окружение
#   make logs          - просмотреть логи всех сервисов
#   make test          - запустить тесты
#
# Требования:
#   - Docker & Docker Compose
#   - Python 3.11+ (для локальной разработки)
#   - Make (обычно уже установлен в Linux/Mac)
# ============================================================================

.PHONY: help
.DEFAULT_GOAL := help

# ----------------------------------------------------------------------------
# ПЕРЕМЕННЫЕ
# ----------------------------------------------------------------------------

# Цвета для вывода (опционально, для красоты)
RESET := \033[0m
BOLD := \033[1m
GREEN := \033[32m
YELLOW := \033[33m
BLUE := \033[34m

# Docker Compose файлы
COMPOSE_FILE := docker-compose.yml
COMPOSE_CMD := docker-compose -f $(COMPOSE_FILE)

# Python
PYTHON := python3
VENV := venv
PIP := $(VENV)/bin/pip

# Переменные окружения
ENV_FILE := .env
ENV_EXAMPLE := .env.example

# ----------------------------------------------------------------------------
# HELP - Справка по командам
# ----------------------------------------------------------------------------

help: ## 📚 Показать эту справку
	@echo "$(BOLD)$(BLUE)PartScout - Доступные команды:$(RESET)"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | \
		awk 'BEGIN {FS = ":.*?## "}; {printf "  $(GREEN)%-20s$(RESET) %s\n", $$1, $$2}'
	@echo ""
	@echo "$(YELLOW)Примеры использования:$(RESET)"
	@echo "  make setup         # Первоначальная настройка проекта"
	@echo "  make dev           # Запустить development сервисы"
	@echo "  make logs          # Посмотреть логи"
	@echo "  make stop          # Остановить всё"

# ----------------------------------------------------------------------------
# SETUP - Первоначальная настройка
# ----------------------------------------------------------------------------

setup: ## 🚀 Первоначальная настройка проекта
	@echo "$(BOLD)$(BLUE)>>> Настройка PartScout...$(RESET)"
	@$(MAKE) check-env
	@$(MAKE) check-docker
	@echo "$(GREEN)✓ Все проверки пройдены!$(RESET)"
	@echo ""
	@echo "$(YELLOW)Следующие шаги:$(RESET)"
	@echo "  1. Заполни .env файл своими ключами"
	@echo "  2. Запусти: make dev"

check-env: ## 🔍 Проверить наличие .env файла
	@if [ ! -f $(ENV_FILE) ]; then \
		echo "$(YELLOW)⚠ Файл .env не найден. Создаю из шаблона...$(RESET)"; \
		cp $(ENV_EXAMPLE) $(ENV_FILE); \
		echo "$(GREEN)✓ Создан файл .env$(RESET)"; \
		echo "$(YELLOW)⚠ ВАЖНО: Заполни переменные окружения в .env!$(RESET)"; \
	else \
		echo "$(GREEN)✓ Файл .env найден$(RESET)"; \
	fi

check-docker: ## 🐳 Проверить Docker и Docker Compose
	@command -v docker >/dev/null 2>&1 || { \
		echo "$(YELLOW)⚠ Docker не установлен!$(RESET)"; \
		echo "Установи: https://docs.docker.com/get-docker/"; \
		exit 1; \
	}
	@command -v docker-compose >/dev/null 2>&1 || { \
		echo "$(YELLOW)⚠ Docker Compose не установлен!$(RESET)"; \
		echo "Установи: https://docs.docker.com/compose/install/"; \
		exit 1; \
	}
	@echo "$(GREEN)✓ Docker: $$(docker --version)$(RESET)"
	@echo "$(GREEN)✓ Docker Compose: $$(docker-compose --version)$(RESET)"

# ----------------------------------------------------------------------------
# DEVELOPMENT - Разработка
# ----------------------------------------------------------------------------

dev: ## 🛠 Запустить development окружение (API + Browser)
	@echo "$(BOLD)$(BLUE)>>> Запуск development сервисов...$(RESET)"
	@$(COMPOSE_CMD) up api browser

dev-full: ## 🛠 Запустить полный стек (включая RAG и туннели)
	@echo "$(BOLD)$(BLUE)>>> Запуск полного стека...$(RESET)"
	@$(COMPOSE_CMD) --profile rag-enabled --profile external-browser-access up

dev-d: ## 🛠 Запустить в фоновом режиме (detached)
	@echo "$(BOLD)$(BLUE)>>> Запуск в background...$(RESET)"
	@$(COMPOSE_CMD) up -d api browser
	@echo "$(GREEN)✓ Сервисы запущены в фоне$(RESET)"
	@echo "Просмотр логов: make logs"

build: ## 🔨 Пересобрать Docker образы
	@echo "$(BOLD)$(BLUE)>>> Сборка Docker образов...$(RESET)"
	@$(COMPOSE_CMD) build --no-cache

rebuild: ## 🔨 Пересобрать и перезапустить
	@$(MAKE) build
	@$(MAKE) dev

# ----------------------------------------------------------------------------
# SERVICES - Управление отдельными сервисами
# ----------------------------------------------------------------------------

api: ## 🔧 Запустить только API сервис
	@$(COMPOSE_CMD) up api

browser: ## 🌐 Запустить только Browser сервис
	@$(COMPOSE_CMD) up browser

rag: ## 🧠 Запустить только RAG сервис
	@$(COMPOSE_CMD) --profile rag-enabled up rag

# ----------------------------------------------------------------------------
# LOGS - Логи
# ----------------------------------------------------------------------------

logs: ## 📋 Показать логи всех сервисов
	@$(COMPOSE_CMD) logs -f

logs-api: ## 📋 Логи API сервиса
	@$(COMPOSE_CMD) logs -f api

logs-browser: ## 📋 Логи Browser сервиса
	@$(COMPOSE_CMD) logs -f browser

logs-rag: ## 📋 Логи RAG сервиса
	@$(COMPOSE_CMD) logs -f rag

# ----------------------------------------------------------------------------
# CONTROL - Управление
# ----------------------------------------------------------------------------

stop: ## ⏹ Остановить все сервисы
	@echo "$(YELLOW)>>> Остановка сервисов...$(RESET)"
	@$(COMPOSE_CMD) stop
	@echo "$(GREEN)✓ Все сервисы остановлены$(RESET)"

down: ## ⏹ Остановить и удалить контейнеры
	@echo "$(YELLOW)>>> Удаление контейнеров...$(RESET)"
	@$(COMPOSE_CMD) down
	@echo "$(GREEN)✓ Контейнеры удалены$(RESET)"

restart: ## 🔄 Перезапустить все сервисы
	@$(MAKE) stop
	@$(MAKE) dev

restart-api: ## 🔄 Перезапустить API
	@$(COMPOSE_CMD) restart api

restart-browser: ## 🔄 Перезапустить Browser
	@$(COMPOSE_CMD) restart browser

# ----------------------------------------------------------------------------
# STATUS - Статус и мониторинг
# ----------------------------------------------------------------------------

status: ## 📊 Показать статус сервисов
	@echo "$(BOLD)$(BLUE)>>> Статус сервисов:$(RESET)"
	@$(COMPOSE_CMD) ps

health: ## 🏥 Проверить health checks
	@echo "$(BOLD)$(BLUE)>>> Health проверки:$(RESET)"
	@docker ps --filter "label=partscout.service" --format "table {{.Names}}\t{{.Status}}"

stats: ## 📈 Показать статистику ресурсов
	@docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# ----------------------------------------------------------------------------
# SHELL ACCESS - Доступ к контейнерам
# ----------------------------------------------------------------------------

shell-api: ## 🐚 Открыть shell в API контейнере
	@$(COMPOSE_CMD) exec api /bin/bash

shell-browser: ## 🐚 Открыть shell в Browser контейнере
	@$(COMPOSE_CMD) exec browser /bin/bash

shell-rag: ## 🐚 Открыть shell в RAG контейнере
	@$(COMPOSE_CMD) exec rag /bin/bash

# ----------------------------------------------------------------------------
# DATABASE - Работа с базой данных
# ----------------------------------------------------------------------------

db-migrate: ## 🗄 Применить миграции БД
	@echo "$(BOLD)$(BLUE)>>> Применение миграций...$(RESET)"
	@$(COMPOSE_CMD) exec api python -m alembic upgrade head
	@echo "$(GREEN)✓ Миграции применены$(RESET)"

db-migration: ## 🗄 Создать новую миграцию
	@read -p "Название миграции: " name; \
	$(COMPOSE_CMD) exec api python -m alembic revision --autogenerate -m "$$name"

db-rollback: ## 🗄 Откатить последнюю миграцию
	@$(COMPOSE_CMD) exec api python -m alembic downgrade -1

db-reset: ## 🗄 Сбросить БД (ВНИМАНИЕ: удалит все данные!)
	@echo "$(YELLOW)⚠ ВНИМАНИЕ: Это удалит все данные!$(RESET)"
	@read -p "Продолжить? (yes/no): " confirm; \
	if [ "$$confirm" = "yes" ]; then \
		$(COMPOSE_CMD) exec api python scripts/reset_database.py; \
		echo "$(GREEN)✓ База данных сброшена$(RESET)"; \
	else \
		echo "Отменено"; \
	fi

# ----------------------------------------------------------------------------
# TESTING - Тесты
# ----------------------------------------------------------------------------

test: ## 🧪 Запустить все тесты
	@echo "$(BOLD)$(BLUE)>>> Запуск тестов...$(RESET)"
	@$(COMPOSE_CMD) exec api pytest tests/ -v

test-unit: ## 🧪 Запустить unit тесты
	@$(COMPOSE_CMD) exec api pytest tests/unit/ -v

test-integration: ## 🧪 Запустить integration тесты
	@$(COMPOSE_CMD) exec api pytest tests/integration/ -v

test-cov: ## 🧪 Запустить тесты с покрытием
	@$(COMPOSE_CMD) exec api pytest tests/ --cov=. --cov-report=html
	@echo "$(GREEN)✓ Отчёт о покрытии: htmlcov/index.html$(RESET)"

# ----------------------------------------------------------------------------
# LINTING & FORMATTING - Качество кода
# ----------------------------------------------------------------------------

lint: ## 🔍 Проверить код (flake8, mypy)
	@echo "$(BOLD)$(BLUE)>>> Проверка кода...$(RESET)"
	@$(COMPOSE_CMD) exec api flake8 services/ shared/
	@$(COMPOSE_CMD) exec api mypy services/ shared/

format: ## ✨ Форматировать код (black, isort)
	@echo "$(BOLD)$(BLUE)>>> Форматирование кода...$(RESET)"
	@$(COMPOSE_CMD) exec api black services/ shared/
	@$(COMPOSE_CMD) exec api isort services/ shared/
	@echo "$(GREEN)✓ Код отформатирован$(RESET)"

format-check: ## ✨ Проверить форматирование (без изменений)
	@$(COMPOSE_CMD) exec api black --check services/ shared/
	@$(COMPOSE_CMD) exec api isort --check services/ shared/

# ----------------------------------------------------------------------------
# CLEANUP - Очистка
# ----------------------------------------------------------------------------

clean: ## 🧹 Очистить временные файлы
	@echo "$(YELLOW)>>> Очистка временных файлов...$(RESET)"
	@find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
	@find . -type f -name "*.pyc" -delete 2>/dev/null || true
	@find . -type f -name "*.pyo" -delete 2>/dev/null || true
	@find . -type d -name "*.egg-info" -exec rm -rf {} + 2>/dev/null || true
	@find . -type d -name ".pytest_cache" -exec rm -rf {} + 2>/dev/null || true
	@find . -type d -name ".mypy_cache" -exec rm -rf {} + 2>/dev/null || true
	@echo "$(GREEN)✓ Временные файлы удалены$(RESET)"

clean-all: ## 🧹 Полная очистка (включая Docker volumes)
	@echo "$(YELLOW)⚠ ВНИМАНИЕ: Удалит все Docker volumes!$(RESET)"
	@read -p "Продолжить? (yes/no): " confirm; \
	if [ "$$confirm" = "yes" ]; then \
		$(MAKE) down; \
		docker volume prune -f; \
		$(MAKE) clean; \
		echo "$(GREEN)✓ Полная очистка завершена$(RESET)"; \
	else \
		echo "Отменено"; \
	fi

clean-logs: ## 🧹 Очистить логи
	@rm -rf logs/*.log
	@echo "$(GREEN)✓ Логи очищены$(RESET)"

# ----------------------------------------------------------------------------
# LOCAL DEVELOPMENT - Локальная разработка (без Docker)
# ----------------------------------------------------------------------------

venv: ## 🐍 Создать Python виртуальное окружение
	@echo "$(BOLD)$(BLUE)>>> Создание venv...$(RESET)"
	@$(PYTHON) -m venv $(VENV)
	@echo "$(GREEN)✓ Venv создано: $(VENV)$(RESET)"
	@echo "Активировать: source $(VENV)/bin/activate"

install: ## 📦 Установить зависимости локально
	@echo "$(BOLD)$(BLUE)>>> Установка зависимостей...$(RESET)"
	@$(PIP) install --upgrade pip
	@$(PIP) install -r requirements.txt
	@$(PIP) install -r requirements-dev.txt
	@echo "$(GREEN)✓ Зависимости установлены$(RESET)"

install-local: venv install ## 📦 Полная локальная установка (venv + зависимости)

run-local-api: ## 🏃 Запустить API локально (без Docker)
	@source $(VENV)/bin/activate && \
	cd services/api && \
	uvicorn main:app --reload --host 0.0.0.0 --port 8000

# ----------------------------------------------------------------------------
# MCP - Model Context Protocol серверы
# ----------------------------------------------------------------------------

mcp-test: ## 🔌 Протестировать MCP серверы
	@echo "$(BOLD)$(BLUE)>>> Тестирование MCP серверов...$(RESET)"
	@$(COMPOSE_CMD) exec api python scripts/test_mcp_servers.py

mcp-list: ## 🔌 Показать список MCP серверов
	@echo "$(BOLD)$(BLUE)Доступные MCP серверы:$(RESET)"
	@echo "  - catalog-server (каталоги запчастей)"
	@echo "  - browser-server (парсинг сайтов)"
	@echo "  - analysis-server (AI анализ)"
	@echo "  - rag-server (векторный поиск)"
	@echo "  - assisted-search-server (помощник менеджера)"
	@echo "  - manual-mode-server (полуавтоматический режим)"

# ----------------------------------------------------------------------------
# GEMINI PROXY - Проверка прокси
# ----------------------------------------------------------------------------

proxy-test: ## 🌐 Протестировать Gemini Proxy
	@echo "$(BOLD)$(BLUE)>>> Тестирование Gemini Proxy...$(RESET)"
	@curl -X POST $(GEMINI_PROXY_URL)/v1beta/models/gemini-pro:generateContent \
		-H "X-Master-Key: $(GEMINI_PROXY_MASTER_KEY)" \
		-H "Content-Type: application/json" \
		-d '{"contents":[{"parts":[{"text":"Test connection"}]}]}' \
		| jq '.candidates[0].content.parts[0].text' 2>/dev/null || \
		echo "$(YELLOW)⚠ Проверь GEMINI_PROXY_URL и GEMINI_PROXY_MASTER_KEY в .env$(RESET)"

proxy-stats: ## 🌐 Статистика Gemini Proxy
	@curl -s $(GEMINI_PROXY_URL)/__do-stats | jq .

# ----------------------------------------------------------------------------
# DOCUMENTATION - Документация
# ----------------------------------------------------------------------------

docs: ## 📖 Сгенерировать документацию API
	@$(COMPOSE_CMD) exec api python -m mkdocs build
	@echo "$(GREEN)✓ Документация: site/index.html$(RESET)"

docs-serve: ## 📖 Запустить локальный сервер документации
	@$(COMPOSE_CMD) exec api python -m mkdocs serve -a 0.0.0.0:8001

# ----------------------------------------------------------------------------
# DEPLOYMENT - Деплой
# ----------------------------------------------------------------------------

deploy-staging: ## 🚀 Деплой на staging
	@echo "$(BOLD)$(BLUE)>>> Деплой на staging...$(RESET)"
	@git push staging main
	@echo "$(GREEN)✓ Деплой завершён$(RESET)"

deploy-prod: ## 🚀 Деплой на production
	@echo "$(YELLOW)⚠ ВНИМАНИЕ: Деплой на PRODUCTION!$(RESET)"
	@read -p "Продолжить? (yes/no): " confirm; \
	if [ "$$confirm" = "yes" ]; then \
		git push production main; \
		echo "$(GREEN)✓ Деплой на production завершён$(RESET)"; \
	else \
		echo "Отменено"; \
	fi

# ----------------------------------------------------------------------------
# BACKUP - Резервное копирование
# ----------------------------------------------------------------------------

backup: ## 💾 Создать backup базы данных
	@echo "$(BOLD)$(BLUE)>>> Создание backup...$(RESET)"
	@mkdir -p backups
	@$(COMPOSE_CMD) exec api python scripts/backup_database.py
	@echo "$(GREEN)✓ Backup создан: backups/$(shell date +%Y%m%d_%H%M%S).sql$(RESET)"

# ----------------------------------------------------------------------------
# UTILITIES - Утилиты
# ----------------------------------------------------------------------------

update-deps: ## 📦 Обновить зависимости
	@echo "$(BOLD)$(BLUE)>>> Обновление зависимостей...$(RESET)"
	@$(PIP) install --upgrade pip
	@$(PIP) list --outdated
	@echo "$(YELLOW)Для обновления: pip install --upgrade <package>$(RESET)"

env-check: ## ✅ Проверить переменные окружения
	@echo "$(BOLD)$(BLUE)>>> Проверка критичных переменных:$(RESET)"
	@echo -n "GEMINI_PROXY_URL: "; \
		[ -n "$(shell grep GEMINI_PROXY_URL .env 2>/dev/null | cut -d= -f2)" ] && \
		echo "$(GREEN)✓$(RESET)" || echo "$(YELLOW)⚠ Не задан$(RESET)"
	@echo -n "SUPABASE_URL: "; \
		[ -n "$(shell grep SUPABASE_URL .env 2>/dev/null | cut -d= -f2)" ] && \
		echo "$(GREEN)✓$(RESET)" || echo "$(YELLOW)⚠ Не задан$(RESET)"
	@echo -n "TELEGRAM_MANAGER_BOT_TOKEN: "; \
		[ -n "$(shell grep TELEGRAM_MANAGER_BOT_TOKEN .env 2>/dev/null | cut -d= -f2)" ] && \
		echo "$(GREEN)✓$(RESET)" || echo "$(YELLOW)⚠ Не задан$(RESET)"

version: ## 📌 Показать версию проекта
	@echo "$(BOLD)PartScout v0.1.0-MVP$(RESET)"
	@echo "Git branch: $(shell git rev-parse --abbrev-ref HEAD)"
	@echo "Git commit: $(shell git rev-parse --short HEAD)"

# ============================================================================
# END OF MAKEFILE
# ============================================================================
```

---

# 📖 Инструкция по использованию Make

## 🎯 Что такое Make?

**Make** — утилита для автоматизации повторяющихся задач. Вместо того чтобы запоминать длинные команды Docker, Git, Python — вы пишете короткие и понятные команды.

---

## 🚀 Быстрый старт

### 1️⃣ Установка Make (если нет)

#### Linux (Debian/Ubuntu):
```bash
sudo apt-get install make
```

#### macOS:
```bash
# Обычно уже установлен
# Если нет:
xcode-select --install
```

#### Windows:
```bash
# Вариант 1: WSL (рекомендуется)
wsl --install
# Затем в WSL: sudo apt-get install make

# Вариант 2: Chocolatey
choco install make

# Вариант 3: Scoop
scoop install make
```

### 2️⃣ Проверка установки
```bash
make --version
# Должно вывести: GNU Make 4.x
```

---

## 📚 Основные команды

### Показать справку
```bash
make help
```
**Результат:** Список всех доступных команд с описанием

### Первоначальная настройка проекта
```bash
make setup
```
**Что делает:**
- Проверяет Docker и Docker Compose
- Создаёт .env из .env.example (если его нет)
- Даёт инструкции что делать дальше

### Запустить development окружение
```bash
make dev
```
**Что делает:**
- Запускает API и Browser сервисы
- Показывает логи в реальном времени
- Остановить: `Ctrl+C`

### Запустить в фоновом режиме
```bash
make dev-d
```
**Что делает:**
- Запускает сервисы в background
- Терминал свободен для других команд
- Остановить: `make stop`

### Просмотр логов
```bash
# Все сервисы
make logs

# Только API
make logs-api

# Только Browser
make logs-browser
```

### Остановить сервисы
```bash
# Остановить (контейнеры остаются)
make stop

# Остановить и удалить контейнеры
make down

# Перезапустить
make restart
```

---

## 🔧 Разработка

### Пересборка Docker образов
```bash
# После изменения Dockerfile
make build

# Пересобрать и сразу запустить
make rebuild
```

### Открыть shell в контейнере
```bash
# API контейнер
make shell-api

# Browser контейнер
make shell-browser
```

### Работа с базой данных
```bash
# Применить миграции
make db-migrate

# Создать новую миграцию
make db-migration

# Откатить последнюю миграцию
make db-rollback
```

### Тестирование
```bash
# Все тесты
make test

# Только unit тесты
make test-unit

# С покрытием кода
make test-cov
```

### Проверка кода
```bash
# Проверить стиль кода
make lint

# Отформатировать код
make format

# Проверить форматирование (без изменений)
make format-check
```

---

## 🌐 Специфичные для PartScout

### Тестирование Gemini Proxy
```bash
# Проверить что прокси работает
make proxy-test

# Посмотреть статистику использования ключей
make proxy-stats
```

### MCP серверы
```bash
# Список доступных серверов
make mcp-list

# Протестировать серверы
make mcp-test
```

### Статус и мониторинг
```bash
# Статус всех сервисов
make status

# Health checks
make health

# Использование ресурсов (CPU, RAM)
make stats
```

---

## 🧹 Очистка

### Очистить временные файлы
```bash
# Python __pycache__, .pyc, etc.
make clean

# Очистить логи
make clean-logs

# ПОЛНАЯ очистка (включая Docker volumes)
make clean-all
```

---

## 🐍 Локальная разработка (без Docker)

### Создать виртуальное окружение
```bash
# Создать venv
make venv

# Установить зависимости
make install

# Всё сразу
make install-local
```

### Активировать venv
```bash
source venv/bin/activate
```

### Запустить API локально
```bash
make run-local-api
```

---

## 📊 Полезные комбинации

### Полный рабочий цикл
```bash
# 1. Первый раз
make setup
# Заполнить .env
make dev

# 2. Ежедневная разработка
make dev-d          # Запустить в фоне
make logs-api       # Смотреть логи при разработке
# Код меняется автоматически (hot reload)
make restart-api    # Если нужен полный рестарт

# 3. Перед коммитом
make format         # Отформатировать код
make lint           # Проверить стиль
make test           # Запустить тесты
make clean          # Очистить temp файлы

# 4. После изменений в Dockerfile
make rebuild        # Пересобрать и запустить
```

### Быстрая отладка проблем
```bash
# Что-то не работает?
make status         # Все ли сервисы запущены?
make health         # Прошли ли health checks?
make logs           # Что в логах?
make shell-api      # Зайти внутрь контейнера

# Gemini не работает?
make proxy-test     # Проверить прокси
make env-check      # Проверить переменные окружения
```

---

## 🎓 Продвинутое использование

### Запуск нескольких команд
```bash
# Последовательно
make clean && make build && make dev

# Параллельно (не всегда безопасно)
make logs-api & make logs-browser
```

### Переопределение переменных
```bash
# Использовать другой compose файл
make dev COMPOSE_FILE=docker-compose.prod.yml

# Другой Python
make venv PYTHON=python3.12
```

### Создание собственных команд
```makefile
# Добавь в Makefile:
my-task: ## 🎯 Моя кастомная задача
	@echo "Делаю что-то полезное"
	@make clean
	@make test
```

---

## ⚡ Шпаргалка команд

| Команда | Что делает |
|---------|------------|
| `make help` | Справка |
| `make setup` | Первоначальная настройка |
| `make dev` | Запуск development |
| `make dev-d` | Запуск в фоне |
| `make logs` | Просмотр логов |
| `make stop` | Остановка |
| `make restart` | Перезапуск |
| `make status` | Статус сервисов |
| `make test` | Запуск тестов |
| `make lint` | Проверка кода |
| `make format` | Форматирование |
| `make clean` | Очистка |
| `make shell-api` | Shell в контейнер |
| `make proxy-test` | Тест Gemini Proxy |

---

## 🐛 Troubleshooting

### Ошибка: "make: command not found"
```bash
# Установи make (см. раздел "Установка Make")
```

### Ошибка: "No rule to make target"
```bash
# Проверь что Makefile в корне проекта
ls -la Makefile

# Проверь синтаксис команды
make help
```

### Команда не работает как ожидается
```bash
# Посмотри что именно выполняется
make -n dev  # Dry run (показать без выполнения)
```

### Docker команды не работают
```bash
# Проверь Docker
make check-docker

# Проверь что Docker запущен
docker ps
```

---

## 🔖 Git Commit

```bash
git add Makefile
git commit -m "feat: add comprehensive Makefile for development automation

Development workflow:
- Add setup commands (check-env, check-docker)
- Add docker-compose shortcuts (dev, build, rebuild)
- Add service management (start/stop individual services)
- Add log viewing commands (all services + individual)
- Add shell access to containers

Database operations:
- Add migration commands (migrate, rollback, new migration)
- Add database reset with confirmation

Testing & Quality:
- Add test commands (all, unit, integration, coverage)
- Add linting and formatting (flake8, mypy, black, isort)

PartScout specific:
- Add Gemini Proxy testing commands
- Add MCP servers management
- Add health checks and monitoring

Utilities:
- Add cleanup commands (clean, clean-all, clean-logs)
- Add local development setup (venv, install)
- Add environment validation (env-check)
- Add backup commands

Features:
- Colored output for better readability
- Confirmation prompts for destructive operations
- Comprehensive help system (make help)
- Examples and usage instructions
- Support for profiles (rag-enabled, external-access)

This Makefile provides ~50 commands covering full development
lifecycle from setup to deployment. All commands are documented
with inline help accessible via 'make help'."
```

---

## ✅ Готово!

**Файл создан:** `Makefile`

**Следующий файл:** `CONTRIBUTING.md` или `.github/workflows/ci.yml`?

Что создаём следующим? 🚀# 📄 Файл 3: `Makefile`

```makefile
# ============================================================================
# PARTSCOUT - Development Makefile
# ============================================================================
# Упрощённые команды для разработки и деплоя проекта
# 
# Использование:
#   make help          - показать все доступные команды
#   make dev           - запустить development окружение
#   make logs          - просмотреть логи всех сервисов
#   make test          - запустить тесты
#
# Требования:
#   - Docker & Docker Compose
#   - Python 3.11+ (для локальной разработки)
#   - Make (обычно уже установлен в Linux/Mac)
# ============================================================================

.PHONY: help
.DEFAULT_GOAL := help

# ----------------------------------------------------------------------------
# ПЕРЕМЕННЫЕ
# ----------------------------------------------------------------------------

# Цвета для вывода (опционально, для красоты)
RESET := \033[0m
BOLD := \033[1m
GREEN := \033[32m
YELLOW := \033[33m
BLUE := \033[34m

# Docker Compose файлы
COMPOSE_FILE := docker-compose.yml
COMPOSE_CMD := docker-compose -f $(COMPOSE_FILE)

# Python
PYTHON := python3
VENV := venv
PIP := $(VENV)/bin/pip

# Переменные окружения
ENV_FILE := .env
ENV_EXAMPLE := .env.example

# ----------------------------------------------------------------------------
# HELP - Справка по командам
# ----------------------------------------------------------------------------

help: ## 📚 Показать эту справку
	@echo "$(BOLD)$(BLUE)PartScout - Доступные команды:$(RESET)"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | \
		awk 'BEGIN {FS = ":.*?## "}; {printf "  $(GREEN)%-20s$(RESET) %s\n", $$1, $$2}'
	@echo ""
	@echo "$(YELLOW)Примеры использования:$(RESET)"
	@echo "  make setup         # Первоначальная настройка проекта"
	@echo "  make dev           # Запустить development сервисы"
	@echo "  make logs          # Посмотреть логи"
	@echo "  make stop          # Остановить всё"

# ----------------------------------------------------------------------------
# SETUP - Первоначальная настройка
# ----------------------------------------------------------------------------

setup: ## 🚀 Первоначальная настройка проекта
	@echo "$(BOLD)$(BLUE)>>> Настройка PartScout...$(RESET)"
	@$(MAKE) check-env
	@$(MAKE) check-docker
	@echo "$(GREEN)✓ Все проверки пройдены!$(RESET)"
	@echo ""
	@echo "$(YELLOW)Следующие шаги:$(RESET)"
	@echo "  1. Заполни .env файл своими ключами"
	@echo "  2. Запусти: make dev"

check-env: ## 🔍 Проверить наличие .env файла
	@if [ ! -f $(ENV_FILE) ]; then \
		echo "$(YELLOW)⚠ Файл .env не найден. Создаю из шаблона...$(RESET)"; \
		cp $(ENV_EXAMPLE) $(ENV_FILE); \
		echo "$(GREEN)✓ Создан файл .env$(RESET)"; \
		echo "$(YELLOW)⚠ ВАЖНО: Заполни переменные окружения в .env!$(RESET)"; \
	else \
		echo "$(GREEN)✓ Файл .env найден$(RESET)"; \
	fi

check-docker: ## 🐳 Проверить Docker и Docker Compose
	@command -v docker >/dev/null 2>&1 || { \
		echo "$(YELLOW)⚠ Docker не установлен!$(RESET)"; \
		echo "Установи: https://docs.docker.com/get-docker/"; \
		exit 1; \
	}
	@command -v docker-compose >/dev/null 2>&1 || { \
		echo "$(YELLOW)⚠ Docker Compose не установлен!$(RESET)"; \
		echo "Установи: https://docs.docker.com/compose/install/"; \
		exit 1; \
	}
	@echo "$(GREEN)✓ Docker: $$(docker --version)$(RESET)"
	@echo "$(GREEN)✓ Docker Compose: $$(docker-compose --version)$(RESET)"

# ----------------------------------------------------------------------------
# DEVELOPMENT - Разработка
# ----------------------------------------------------------------------------

dev: ## 🛠 Запустить development окружение (API + Browser)
	@echo "$(BOLD)$(BLUE)>>> Запуск development сервисов...$(RESET)"
	@$(COMPOSE_CMD) up api browser

dev-full: ## 🛠 Запустить полный стек (включая RAG и туннели)
	@echo "$(BOLD)$(BLUE)>>> Запуск полного стека...$(RESET)"
	@$(COMPOSE_CMD) --profile rag-enabled --profile external-browser-access up

dev-d: ## 🛠 Запустить в фоновом режиме (detached)
	@echo "$(BOLD)$(BLUE)>>> Запуск в background...$(RESET)"
	@$(COMPOSE_CMD) up -d api browser
	@echo "$(GREEN)✓ Сервисы запущены в фоне$(RESET)"
	@echo "Просмотр логов: make logs"

build: ## 🔨 Пересобрать Docker образы
	@echo "$(BOLD)$(BLUE)>>> Сборка Docker образов...$(RESET)"
	@$(COMPOSE_CMD) build --no-cache

rebuild: ## 🔨 Пересобрать и перезапустить
	@$(MAKE) build
	@$(MAKE) dev

# ----------------------------------------------------------------------------
# SERVICES - Управление отдельными сервисами
# ----------------------------------------------------------------------------

api: ## 🔧 Запустить только API сервис
	@$(COMPOSE_CMD) up api

browser: ## 🌐 Запустить только Browser сервис
	@$(COMPOSE_CMD) up browser

rag: ## 🧠 Запустить только RAG сервис
	@$(COMPOSE_CMD) --profile rag-enabled up rag

# ----------------------------------------------------------------------------
# LOGS - Логи
# ----------------------------------------------------------------------------

logs: ## 📋 Показать логи всех сервисов
	@$(COMPOSE_CMD) logs -f

logs-api: ## 📋 Логи API сервиса
	@$(COMPOSE_CMD) logs -f api

logs-browser: ## 📋 Логи Browser сервиса
	@$(COMPOSE_CMD) logs -f browser

logs-rag: ## 📋 Логи RAG сервиса
	@$(COMPOSE_CMD) logs -f rag

# ----------------------------------------------------------------------------
# CONTROL - Управление
# ----------------------------------------------------------------------------

stop: ## ⏹ Остановить все сервисы
	@echo "$(YELLOW)>>> Остановка сервисов...$(RESET)"
	@$(COMPOSE_CMD) stop
	@echo "$(GREEN)✓ Все сервисы остановлены$(RESET)"

down: ## ⏹ Остановить и удалить контейнеры
	@echo "$(YELLOW)>>> Удаление контейнеров...$(RESET)"
	@$(COMPOSE_CMD) down
	@echo "$(GREEN)✓ Контейнеры удалены$(RESET)"

restart: ## 🔄 Перезапустить все сервисы
	@$(MAKE) stop
	@$(MAKE) dev

restart-api: ## 🔄 Перезапустить API
	@$(COMPOSE_CMD) restart api

restart-browser: ## 🔄 Перезапустить Browser
	@$(COMPOSE_CMD) restart browser

# ----------------------------------------------------------------------------
# STATUS - Статус и мониторинг
# ----------------------------------------------------------------------------

status: ## 📊 Показать статус сервисов
	@echo "$(BOLD)$(BLUE)>>> Статус сервисов:$(RESET)"
	@$(COMPOSE_CMD) ps

health: ## 🏥 Проверить health checks
	@echo "$(BOLD)$(BLUE)>>> Health проверки:$(RESET)"
	@docker ps --filter "label=partscout.service" --format "table {{.Names}}\t{{.Status}}"

stats: ## 📈 Показать статистику ресурсов
	@docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# ----------------------------------------------------------------------------
# SHELL ACCESS - Доступ к контейнерам
# ----------------------------------------------------------------------------

shell-api: ## 🐚 Открыть shell в API контейнере
	@$(COMPOSE_CMD) exec api /bin/bash

shell-browser: ## 🐚 Открыть shell в Browser контейнере
	@$(COMPOSE_CMD) exec browser /bin/bash

shell-rag: ## 🐚 Открыть shell в RAG контейнере
	@$(COMPOSE_CMD) exec rag /bin/bash

# ----------------------------------------------------------------------------
# DATABASE - Работа с базой данных
# ----------------------------------------------------------------------------

db-migrate: ## 🗄 Применить миграции БД
	@echo "$(BOLD)$(BLUE)>>> Применение миграций...$(RESET)"
	@$(COMPOSE_CMD) exec api python -m alembic upgrade head
	@echo "$(GREEN)✓ Миграции применены$(RESET)"

db-migration: ## 🗄 Создать новую миграцию
	@read -p "Название миграции: " name; \
	$(COMPOSE_CMD) exec api python -m alembic revision --autogenerate -m "$$name"

db-rollback: ## 🗄 Откатить последнюю миграцию
	@$(COMPOSE_CMD) exec api python -m alembic downgrade -1

db-reset: ## 🗄 Сбросить БД (ВНИМАНИЕ: удалит все данные!)
	@echo "$(YELLOW)⚠ ВНИМАНИЕ: Это удалит все данные!$(RESET)"
	@read -p "Продолжить? (yes/no): " confirm; \
	if [ "$$confirm" = "yes" ]; then \
		$(COMPOSE_CMD) exec api python scripts/reset_database.py; \
		echo "$(GREEN)✓ База данных сброшена$(RESET)"; \
	else \
		echo "Отменено"; \
	fi

# ----------------------------------------------------------------------------
# TESTING - Тесты
# ----------------------------------------------------------------------------

test: ## 🧪 Запустить все тесты
	@echo "$(BOLD)$(BLUE)>>> Запуск тестов...$(RESET)"
	@$(COMPOSE_CMD) exec api pytest tests/ -v

test-unit: ## 🧪 Запустить unit тесты
	@$(COMPOSE_CMD) exec api pytest tests/unit/ -v

test-integration: ## 🧪 Запустить integration тесты
	@$(COMPOSE_CMD) exec api pytest tests/integration/ -v

test-cov: ## 🧪 Запустить тесты с покрытием
	@$(COMPOSE_CMD) exec api pytest tests/ --cov=. --cov-report=html
	@echo "$(GREEN)✓ Отчёт о покрытии: htmlcov/index.html$(RESET)"

# ----------------------------------------------------------------------------
# LINTING & FORMATTING - Качество кода
# ----------------------------------------------------------------------------

lint: ## 🔍 Проверить код (flake8, mypy)
	@echo "$(BOLD)$(BLUE)>>> Проверка кода...$(RESET)"
	@$(COMPOSE_CMD) exec api flake8 services/ shared/
	@$(COMPOSE_CMD) exec api mypy services/ shared/

format: ## ✨ Форматировать код (black, isort)
	@echo "$(BOLD)$(BLUE)>>> Форматирование кода...$(RESET)"
	@$(COMPOSE_CMD) exec api black services/ shared/
	@$(COMPOSE_CMD) exec api isort services/ shared/
	@echo "$(GREEN)✓ Код отформатирован$(RESET)"

format-check: ## ✨ Проверить форматирование (без изменений)
	@$(COMPOSE_CMD) exec api black --check services/ shared/
	@$(COMPOSE_CMD) exec api isort --check services/ shared/

# ----------------------------------------------------------------------------
# CLEANUP - Очистка
# ----------------------------------------------------------------------------

clean: ## 🧹 Очистить временные файлы
	@echo "$(YELLOW)>>> Очистка временных файлов...$(RESET)"
	@find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
	@find . -type f -name "*.pyc" -delete 2>/dev/null || true
	@find . -type f -name "*.pyo" -delete 2>/dev/null || true
	@find . -type d -name "*.egg-info" -exec rm -rf {} + 2>/dev/null || true
	@find . -type d -name ".pytest_cache" -exec rm -rf {} + 2>/dev/null || true
	@find . -type d -name ".mypy_cache" -exec rm -rf {} + 2>/dev/null || true
	@echo "$(GREEN)✓ Временные файлы удалены$(RESET)"

clean-all: ## 🧹 Полная очистка (включая Docker volumes)
	@echo "$(YELLOW)⚠ ВНИМАНИЕ: Удалит все Docker volumes!$(RESET)"
	@read -p "Продолжить? (yes/no): " confirm; \
	if [ "$$confirm" = "yes" ]; then \
		$(MAKE) down; \
		docker volume prune -f; \
		$(MAKE) clean; \
		echo "$(GREEN)✓ Полная очистка завершена$(RESET)"; \
	else \
		echo "Отменено"; \
	fi

clean-logs: ## 🧹 Очистить логи
	@rm -rf logs/*.log
	@echo "$(GREEN)✓ Логи очищены$(RESET)"

# ----------------------------------------------------------------------------
# LOCAL DEVELOPMENT - Локальная разработка (без Docker)
# ----------------------------------------------------------------------------

venv: ## 🐍 Создать Python виртуальное окружение
	@echo "$(BOLD)$(BLUE)>>> Создание venv...$(RESET)"
	@$(PYTHON) -m venv $(VENV)
	@echo "$(GREEN)✓ Venv создано: $(VENV)$(RESET)"
	@echo "Активировать: source $(VENV)/bin/activate"

install: ## 📦 Установить зависимости локально
	@echo "$(BOLD)$(BLUE)>>> Установка зависимостей...$(RESET)"
	@$(PIP) install --upgrade pip
	@$(PIP) install -r requirements.txt
	@$(PIP) install -r requirements-dev.txt
	@echo "$(GREEN)✓ Зависимости установлены$(RESET)"

install-local: venv install ## 📦 Полная локальная установка (venv + зависимости)

run-local-api: ## 🏃 Запустить API локально (без Docker)
	@source $(VENV)/bin/activate && \
	cd services/api && \
	uvicorn main:app --reload --host 0.0.0.0 --port 8000

# ----------------------------------------------------------------------------
# MCP - Model Context Protocol серверы
# ----------------------------------------------------------------------------

mcp-test: ## 🔌 Протестировать MCP серверы
	@echo "$(BOLD)$(BLUE)>>> Тестирование MCP серверов...$(RESET)"
	@$(COMPOSE_CMD) exec api python scripts/test_mcp_servers.py

mcp-list: ## 🔌 Показать список MCP серверов
	@echo "$(BOLD)$(BLUE)Доступные MCP серверы:$(RESET)"
	@echo "  - catalog-server (каталоги запчастей)"
	@echo "  - browser-server (парсинг сайтов)"
	@echo "  - analysis-server (AI анализ)"
	@echo "  - rag-server (векторный поиск)"
	@echo "  - assisted-search-server (помощник менеджера)"
	@echo "  - manual-mode-server (полуавтоматический режим)"

# ----------------------------------------------------------------------------
# GEMINI PROXY - Проверка прокси
# ----------------------------------------------------------------------------

proxy-test: ## 🌐 Протестировать Gemini Proxy
	@echo "$(BOLD)$(BLUE)>>> Тестирование Gemini Proxy...$(RESET)"
	@curl -X POST $(GEMINI_PROXY_URL)/v1beta/models/gemini-pro:generateContent \
		-H "X-Master-Key: $(GEMINI_PROXY_MASTER_KEY)" \
		-H "Content-Type: application/json" \
		-d '{"contents":[{"parts":[{"text":"Test connection"}]}]}' \
		| jq '.candidates[0].content.parts[0].text' 2>/dev/null || \
		echo "$(YELLOW)⚠ Проверь GEMINI_PROXY_URL и GEMINI_PROXY_MASTER_KEY в .env$(RESET)"

proxy-stats: ## 🌐 Статистика Gemini Proxy
	@curl -s $(GEMINI_PROXY_URL)/__do-stats | jq .

# ----------------------------------------------------------------------------
# DOCUMENTATION - Документация
# ----------------------------------------------------------------------------

docs: ## 📖 Сгенерировать документацию API
	@$(COMPOSE_CMD) exec api python -m mkdocs build
	@echo "$(GREEN)✓ Документация: site/index.html$(RESET)"

docs-serve: ## 📖 Запустить локальный сервер документации
	@$(COMPOSE_CMD) exec api python -m mkdocs serve -a 0.0.0.0:8001

# ----------------------------------------------------------------------------
# DEPLOYMENT - Деплой
# ----------------------------------------------------------------------------

deploy-staging: ## 🚀 Деплой на staging
	@echo "$(BOLD)$(BLUE)>>> Деплой на staging...$(RESET)"
	@git push staging main
	@echo "$(GREEN)✓ Деплой завершён$(RESET)"

deploy-prod: ## 🚀 Деплой на production
	@echo "$(YELLOW)⚠ ВНИМАНИЕ: Деплой на PRODUCTION!$(RESET)"
	@read -p "Продолжить? (yes/no): " confirm; \
	if [ "$$confirm" = "yes" ]; then \
		git push production main; \
		echo "$(GREEN)✓ Деплой на production завершён$(RESET)"; \
	else \
		echo "Отменено"; \
	fi

# ----------------------------------------------------------------------------
# BACKUP - Резервное копирование
# ----------------------------------------------------------------------------

backup: ## 💾 Создать backup базы данных
	@echo "$(BOLD)$(BLUE)>>> Создание backup...$(RESET)"
	@mkdir -p backups
	@$(COMPOSE_CMD) exec api python scripts/backup_database.py
	@echo "$(GREEN)✓ Backup создан: backups/$(shell date +%Y%m%d_%H%M%S).sql$(RESET)"

# ----------------------------------------------------------------------------
# UTILITIES - Утилиты
# ----------------------------------------------------------------------------

update-deps: ## 📦 Обновить зависимости
	@echo "$(BOLD)$(BLUE)>>> Обновление зависимостей...$(RESET)"
	@$(PIP) install --upgrade pip
	@$(PIP) list --outdated
	@echo "$(YELLOW)Для обновления: pip install --upgrade <package>$(RESET)"

env-check: ## ✅ Проверить переменные окружения
	@echo "$(BOLD)$(BLUE)>>> Проверка критичных переменных:$(RESET)"
	@echo -n "GEMINI_PROXY_URL: "; \
		[ -n "$(shell grep GEMINI_PROXY_URL .env 2>/dev/null | cut -d= -f2)" ] && \
		echo "$(GREEN)✓$(RESET)" || echo "$(YELLOW)⚠ Не задан$(RESET)"
	@echo -n "SUPABASE_URL: "; \
		[ -n "$(shell grep SUPABASE_URL .env 2>/dev/null | cut -d= -f2)" ] && \
		echo "$(GREEN)✓$(RESET)" || echo "$(YELLOW)⚠ Не задан$(RESET)"
	@echo -n "TELEGRAM_MANAGER_BOT_TOKEN: "; \
		[ -n "$(shell grep TELEGRAM_MANAGER_BOT_TOKEN .env 2>/dev/null | cut -d= -f2)" ] && \
		echo "$(GREEN)✓$(RESET)" || echo "$(YELLOW)⚠ Не задан$(RESET)"

version: ## 📌 Показать версию проекта
	@echo "$(BOLD)PartScout v0.1.0-MVP$(RESET)"
	@echo "Git branch: $(shell git rev-parse --abbrev-ref HEAD)"
	@echo "Git commit: $(shell git rev-parse --short HEAD)"

# ============================================================================
# END OF MAKEFILE
# ============================================================================
```

---

# 📖 Инструкция по использованию Make

## 🎯 Что такое Make?

**Make** — утилита для автоматизации повторяющихся задач. Вместо того чтобы запоминать длинные команды Docker, Git, Python — вы пишете короткие и понятные команды.

---

## 🚀 Быстрый старт

### 1️⃣ Установка Make (если нет)

#### Linux (Debian/Ubuntu):
```bash
sudo apt-get install make
```

#### macOS:
```bash
# Обычно уже установлен
# Если нет:
xcode-select --install
```

#### Windows:
```bash
# Вариант 1: WSL (рекомендуется)
wsl --install
# Затем в WSL: sudo apt-get install make

# Вариант 2: Chocolatey
choco install make

# Вариант 3: Scoop
scoop install make
```

### 2️⃣ Проверка установки
```bash
make --version
# Должно вывести: GNU Make 4.x
```

---

## 📚 Основные команды

### Показать справку
```bash
make help
```
**Результат:** Список всех доступных команд с описанием

### Первоначальная настройка проекта
```bash
make setup
```
**Что делает:**
- Проверяет Docker и Docker Compose
- Создаёт .env из .env.example (если его нет)
- Даёт инструкции что делать дальше

### Запустить development окружение
```bash
make dev
```
**Что делает:**
- Запускает API и Browser сервисы
- Показывает логи в реальном времени
- Остановить: `Ctrl+C`

### Запустить в фоновом режиме
```bash
make dev-d
```
**Что делает:**
- Запускает сервисы в background
- Терминал свободен для других команд
- Остановить: `make stop`

### Просмотр логов
```bash
# Все сервисы
make logs

# Только API
make logs-api

# Только Browser
make logs-browser
```

### Остановить сервисы
```bash
# Остановить (контейнеры остаются)
make stop

# Остановить и удалить контейнеры
make down

# Перезапустить
make restart
```

---

## 🔧 Разработка

### Пересборка Docker образов
```bash
# После изменения Dockerfile
make build

# Пересобрать и сразу запустить
make rebuild
```

### Открыть shell в контейнере
```bash
# API контейнер
make shell-api

# Browser контейнер
make shell-browser
```

### Работа с базой данных
```bash
# Применить миграции
make db-migrate

# Создать новую миграцию
make db-migration

# Откатить последнюю миграцию
make db-rollback
```

### Тестирование
```bash
# Все тесты
make test

# Только unit тесты
make test-unit

# С покрытием кода
make test-cov
```

### Проверка кода
```bash
# Проверить стиль кода
make lint

# Отформатировать код
make format

# Проверить форматирование (без изменений)
make format-check
```

---

## 🌐 Специфичные для PartScout

### Тестирование Gemini Proxy
```bash
# Проверить что прокси работает
make proxy-test

# Посмотреть статистику использования ключей
make proxy-stats
```

### MCP серверы
```bash
# Список доступных серверов
make mcp-list

# Протестировать серверы
make mcp-test
```

### Статус и мониторинг
```bash
# Статус всех сервисов
make status

# Health checks
make health

# Использование ресурсов (CPU, RAM)
make stats
```

---

## 🧹 Очистка

### Очистить временные файлы
```bash
# Python __pycache__, .pyc, etc.
make clean

# Очистить логи
make clean-logs

# ПОЛНАЯ очистка (включая Docker volumes)
make clean-all
```

---

## 🐍 Локальная разработка (без Docker)

### Создать виртуальное окружение
```bash
# Создать venv
make venv

# Установить зависимости
make install

# Всё сразу
make install-local
```

### Активировать venv
```bash
source venv/bin/activate
```

### Запустить API локально
```bash
make run-local-api
```

---

## 📊 Полезные комбинации

### Полный рабочий цикл
```bash
# 1. Первый раз
make setup
# Заполнить .env
make dev

# 2. Ежедневная разработка
make dev-d          # Запустить в фоне
make logs-api       # Смотреть логи при разработке
# Код меняется автоматически (hot reload)
make restart-api    # Если нужен полный рестарт

# 3. Перед коммитом
make format         # Отформатировать код
make lint           # Проверить стиль
make test           # Запустить тесты
make clean          # Очистить temp файлы

# 4. После изменений в Dockerfile
make rebuild        # Пересобрать и запустить
```

### Быстрая отладка проблем
```bash
# Что-то не работает?
make status         # Все ли сервисы запущены?
make health         # Прошли ли health checks?
make logs           # Что в логах?
make shell-api      # Зайти внутрь контейнера

# Gemini не работает?
make proxy-test     # Проверить прокси
make env-check      # Проверить переменные окружения
```

---

## 🎓 Продвинутое использование

### Запуск нескольких команд
```bash
# Последовательно
make clean && make build && make dev

# Параллельно (не всегда безопасно)
make logs-api & make logs-browser
```

### Переопределение переменных
```bash
# Использовать другой compose файл
make dev COMPOSE_FILE=docker-compose.prod.yml

# Другой Python
make venv PYTHON=python3.12
```

### Создание собственных команд
```makefile
# Добавь в Makefile:
my-task: ## 🎯 Моя кастомная задача
	@echo "Делаю что-то полезное"
	@make clean
	@make test
```

---

## ⚡ Шпаргалка команд

| Команда | Что делает |
|---------|------------|
| `make help` | Справка |
| `make setup` | Первоначальная настройка |
| `make dev` | Запуск development |
| `make dev-d` | Запуск в фоне |
| `make logs` | Просмотр логов |
| `make stop` | Остановка |
| `make restart` | Перезапуск |
| `make status` | Статус сервисов |
| `make test` | Запуск тестов |
| `make lint` | Проверка кода |
| `make format` | Форматирование |
| `make clean` | Очистка |
| `make shell-api` | Shell в контейнер |
| `make proxy-test` | Тест Gemini Proxy |

---

## 🐛 Troubleshooting

### Ошибка: "make: command not found"
```bash
# Установи make (см. раздел "Установка Make")
```

### Ошибка: "No rule to make target"
```bash
# Проверь что Makefile в корне проекта
ls -la Makefile

# Проверь синтаксис команды
make help
```

### Команда не работает как ожидается
```bash
# Посмотри что именно выполняется
make -n dev  # Dry run (показать без выполнения)
```

### Docker команды не работают
```bash
# Проверь Docker
make check-docker

# Проверь что Docker запущен
docker ps
```

---

## 🔖 Git Commit

```bash
git add Makefile
git commit -m "feat: add comprehensive Makefile for development automation

Development workflow:
- Add setup commands (check-env, check-docker)
- Add docker-compose shortcuts (dev, build, rebuild)
- Add service management (start/stop individual services)
- Add log viewing commands (all services + individual)
- Add shell access to containers

Database operations:
- Add migration commands (migrate, rollback, new migration)
- Add database reset with confirmation

Testing & Quality:
- Add test commands (all, unit, integration, coverage)
- Add linting and formatting (flake8, mypy, black, isort)

PartScout specific:
- Add Gemini Proxy testing commands
- Add MCP servers management
- Add health checks and monitoring

Utilities:
- Add cleanup commands (clean, clean-all, clean-logs)
- Add local development setup (venv, install)
- Add environment validation (env-check)
- Add backup commands

Features:
- Colored output for better readability
- Confirmation prompts for destructive operations
- Comprehensive help system (make help)
- Examples and usage instructions
- Support for profiles (rag-enabled, external-access)

This Makefile provides ~50 commands covering full development
lifecycle from setup to deployment. All commands are documented
with inline help accessible via 'make help'."
```
