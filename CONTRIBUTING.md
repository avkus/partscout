# 📄 Файл 4: `CONTRIBUTING.md`

```markdown
# 🤝 Contributing to PartScout

Спасибо за интерес к проекту! Мы рады любому вкладу — от исправления опечаток до новых функций.

---

## 📋 Содержание

- [Кодекс поведения](#кодекс-поведения)
- [С чего начать](#с-чего-начать)
- [Процесс разработки](#процесс-разработки)
- [Структура проекта](#структура-проекта)
- [Стандарты кода](#стандарты-кода)
- [Commit Conventions](#commit-conventions)
- [Pull Request Process](#pull-request-process)
- [Тестирование](#тестирование)
- [Документация](#документация)
- [Вопросы и поддержка](#вопросы-и-поддержка)

---

## 📜 Кодекс поведения

### Наши ценности

- 🤝 **Уважение** — ко всем участникам, независимо от опыта
- 💡 **Открытость** — к новым идеям и конструктивной критике
- 🎯 **Фокус** — на решении проблем автосервисов
- 🌍 **Инклюзивность** — проект для белорусского рынка, но открыт для всех

### Недопустимо

- ❌ Оскорбления и личные нападки
- ❌ Троллинг и провокации
- ❌ Публикация чужой приватной информации
- ❌ Неконструктивная критика

**Нарушения:** Пишите на [your.email@example.com]

---

## 🚀 С чего начать

### 1️⃣ Требования

Перед началом убедитесь что установлены:

- **Git** (2.x+)
- **Docker** (20.x+) и **Docker Compose** (2.x+)
- **Make** (для упрощения команд)
- **Python** 3.11+ (для локальной разработки)
- **Node.js** 18+ (опционально, для n8n workflows)

### 2️⃣ Первоначальная настройка

```bash
# 1. Fork репозитория
# Нажми "Fork" на GitHub

# 2. Клонируй свой fork
git clone https://github.com/YOUR_USERNAME/partscout.git
cd partscout

# 3. Добавь upstream remote
git remote add upstream https://github.com/ORIGINAL_OWNER/partscout.git

# 4. Настрой окружение
make setup

# 5. Создай .env из .env.example и заполни переменные
cp .env.example .env
nano .env

# 6. Запусти development окружение
make dev
```

### 3️⃣ Проверка что всё работает

```bash
# Проверь статус сервисов
make status

# Запусти тесты
make test

# Проверь health checks
make health
```

Если всё ✅ — готов к разработке!

---

## 🔄 Процесс разработки

### Workflow

```
1. Выбери задачу из Issues
   └── Назначь себя или создай новый Issue

2. Создай feature branch
   └── git checkout -b feature/your-feature-name

3. Разработка
   ├── Пиши код
   ├── Добавляй тесты
   ├── Проверяй линтером
   └── Обновляй документацию

4. Commit изменений
   └── Следуй Commit Conventions (см. ниже)

5. Push в свой fork
   └── git push origin feature/your-feature-name

6. Создай Pull Request
   └── Заполни шаблон PR

7. Code Review
   ├── Ответь на комментарии
   ├── Внеси правки если нужно
   └── Дождись approval

8. Merge! 🎉
```

### Типы задач

#### 🐛 **Bug Fix**
```bash
git checkout -b fix/description-of-bug
# Пример: fix/gemini-proxy-timeout
```

#### ✨ **New Feature**
```bash
git checkout -b feature/description-of-feature
# Пример: feature/manual-mode-ui
```

#### 📚 **Documentation**
```bash
git checkout -b docs/what-you-documenting
# Пример: docs/mcp-servers-guide
```

#### ♻️ **Refactoring**
```bash
git checkout -b refactor/what-you-refactoring
# Пример: refactor/catalog-service-structure
```

#### 🧪 **Tests**
```bash
git checkout -b test/what-you-testing
# Пример: test/browser-parser-edge-cases
```

---

## 📁 Структура проекта

### Основные директории

```
partscout/
├── docs/                      # Документация
│   ├── architecture.md        # Архитектурные решения
│   ├── api.md                 # API документация
│   ├── prompts/               # AI промпты для MCP серверов
│   └── manager_training/      # Обучение менеджера
│
├── services/                  # Микросервисы
│   ├── api/                   # FastAPI приложение
│   ├── scraper/               # Browser automation
│   ├── rag/                   # RAG и embeddings
│   ├── assisted-search/       # Помощник менеджера
│   └── manual-mode/           # Полуавтоматический режим
│
├── shared/                    # Общий код
│   ├── database/              # Модели и клиенты БД
│   ├── mcp/                   # MCP серверы (базовые)
│   └── utils/                 # Утилиты
│
├── n8n-workflows/             # n8n automation
├── tests/                     # Тесты
├── scripts/                   # Вспомогательные скрипты
└── infrastructure/            # Docker, K8s, CI/CD
```

### Где что добавлять

| Что добавляешь | Куда класть |
|----------------|-------------|
| Новый MCP сервер | `services/NEW_SERVICE/mcp_server.py` |
| Новый endpoint API | `services/api/routers/` |
| Модель БД | `shared/database/models.py` |
| Парсер поставщика | `services/scraper/providers/` |
| AI промпт | `docs/prompts/` |
| Утилиту | `shared/utils/` |
| Тест | `tests/unit/` или `tests/integration/` |
| Документацию | `docs/` |

---

## 📝 Стандарты кода

### Python

#### Style Guide

Мы следуем **PEP 8** с дополнениями:

```python
# ✅ Хорошо
def get_article_by_vin(
    vin: str,
    catalog: str = "auto1"
) -> Optional[Article]:
    """
    Получение артикула по VIN-коду.
    
    Args:
        vin: VIN-код автомобиля (17 символов)
        catalog: Каталог для поиска (auto1, armtek)
    
    Returns:
        Article объект или None если не найдено
    
    Raises:
        ValueError: Если VIN невалиден
    """
    if len(vin) != 17:
        raise ValueError("VIN must be 17 characters")
    
    # Implementation...
    return article


# ❌ Плохо
def get_art(v,c="auto1"):  # Непонятные имена
    if len(v)!=17:raise ValueError("bad vin")  # Нет пробелов
    # Нет docstring
    return art
```

#### Imports

```python
# Порядок импортов (автоматически через isort)

# 1. Стандартная библиотека
import os
import sys
from typing import Optional, List

# 2. Сторонние библиотеки
import httpx
from fastapi import FastAPI
from pydantic import BaseModel

# 3. Локальные импорты
from shared.database import supabase_client
from shared.utils import logger
```

#### Type Hints

```python
# ✅ Всегда используй type hints
def parse_supplier(
    url: str,
    timeout: int = 30
) -> dict[str, Any]:
    ...

# ❌ Избегай
def parse_supplier(url, timeout=30):  # Нет типов
    ...
```

#### Async/Await

```python
# ✅ Используй async для I/O операций
async def fetch_prices(article: str) -> list[Price]:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"/api/prices/{article}")
        return response.json()

# ❌ Не блокируй event loop
def fetch_prices(article: str) -> list[Price]:
    response = requests.get(...)  # Blocking!
    return response.json()
```

#### Error Handling

```python
# ✅ Специфичные исключения
try:
    article = get_article_by_vin(vin)
except ValueError as e:
    logger.error(f"Invalid VIN: {e}")
    raise
except CatalogUnavailable as e:
    logger.warning(f"Catalog down: {e}")
    return None

# ❌ Широкие except
try:
    article = get_article_by_vin(vin)
except Exception:  # Слишком широко
    pass  # И ничего не логируем!
```

### Инструменты форматирования

```bash
# Автоматическое форматирование
make format

# Проверка без изменений
make format-check

# Линтинг
make lint
```

#### Конфигурация (уже настроено)

```toml
# pyproject.toml

[tool.black]
line-length = 88
target-version = ['py311']

[tool.isort]
profile = "black"
line_length = 88

[tool.mypy]
python_version = "3.11"
strict = true
```

---

## 📝 Commit Conventions

Мы используем **Conventional Commits** для автоматической генерации CHANGELOG.

### Формат

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Когда использовать |
|------|-------------------|
| `feat` | Новая функциональность |
| `fix` | Исправление бага |
| `docs` | Изменения в документации |
| `style` | Форматирование (не влияет на код) |
| `refactor` | Рефакторинг без изменения поведения |
| `perf` | Улучшение производительности |
| `test` | Добавление/изменение тестов |
| `chore` | Рутинные задачи (deps, config) |
| `ci` | Изменения CI/CD |

### Scope (опционально)

Модуль который затронут: `api`, `browser`, `rag`, `catalog`, `mcp`, etc.

### Примеры

```bash
# ✅ Хорошие коммиты

feat(catalog): add armtek.by API integration

Implements catalog provider for armtek.by with:
- VIN decoding
- Article search by brand
- Stock availability check

Closes #42

---

fix(browser): handle timeout in playwright navigation

Increased default timeout from 30s to 60s for slow supplier sites.
Added retry logic with exponential backoff.

Fixes #38

---

docs(mcp): add usage examples for assisted-search server

Added code examples and workflow diagrams for manual mode.

---

refactor(rag): extract embeddings logic to separate service

No functional changes, improved code organization.

---

chore(deps): upgrade gemini-python-sdk to 2.0.1
```

```bash
# ❌ Плохие коммиты

fix bug                        # Нет описания
updated files                  # Неинформативно
WIP                           # Не коммить незавершённое
fixed everything              # Слишком общо
asdfasdf                      # Что это вообще?
```

### Проверка коммитов

```bash
# Перед коммитом проверь формат
git log --oneline -1

# Если нужно исправить последний коммит
git commit --amend
```

---

## 🔀 Pull Request Process

### 1️⃣ Перед созданием PR

```bash
# Убедись что твоя ветка актуальна
git fetch upstream
git rebase upstream/main

# Запусти проверки
make lint
make format-check
make test

# Всё ✅? Создавай PR!
```

### 2️⃣ Шаблон PR

При создании PR заполни шаблон:

```markdown
## Описание
Что делает этот PR? Почему это нужно?

## Тип изменений
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Связанные Issues
Closes #123

## Чеклист
- [ ] Код следует стандартам проекта
- [ ] Добавлены тесты
- [ ] Тесты проходят (`make test`)
- [ ] Обновлена документация
- [ ] Коммиты следуют Conventional Commits
- [ ] Прошёл линтинг (`make lint`)

## Скриншоты (если применимо)

## Дополнительный контекст
```

### 3️⃣ Code Review

**Для автора PR:**

- ✅ Отвечай на комментарии в течение 48 часов
- ✅ Будь открыт к конструктивной критике
- ✅ Объясняй решения если они неочевидны
- ✅ Вноси правки в отдельных коммитах (не force push)

**Для ревьюеров:**

- ✅ Будь вежлив и конструктивен
- ✅ Объясняй ЧТО не так и ПОЧЕМУ
- ✅ Предлагай альтернативы
- ✅ Хвали хорошие решения

### 4️⃣ Критерии для merge

PR будет смержен когда:

- ✅ Все проверки CI/CD прошли
- ✅ Минимум 1 approval от maintainer
- ✅ Нет unresolved комментариев
- ✅ Коммиты следуют conventions
- ✅ Документация обновлена

---

## 🧪 Тестирование

### Обязательные тесты

```python
# ✅ Unit тесты для всей бизнес-логики

# tests/unit/catalog/test_auto1_provider.py

import pytest
from services.catalog.providers.auto1_by import Auto1Provider

@pytest.mark.asyncio
async def test_search_by_article_found():
    """Тест успешного поиска по артикулу"""
    provider = Auto1Provider(login="test", password="test")
    
    # Mock API response
    with mock_httpx_client():
        result = await provider.search_by_article("04152-YZZA6")
    
    assert result is not None
    assert result.article == "04152-YZZA6"
    assert result.price > 0

@pytest.mark.asyncio
async def test_search_by_article_not_found():
    """Тест когда артикул не найден"""
    provider = Auto1Provider(login="test", password="test")
    
    with mock_httpx_client(response_status=404):
        result = await provider.search_by_article("INVALID")
    
    assert result is None

@pytest.mark.asyncio
async def test_search_handles_timeout():
    """Тест обработки timeout"""
    provider = Auto1Provider(login="test", password="test")
    
    with mock_httpx_client(timeout=True):
        with pytest.raises(TimeoutError):
            await provider.search_by_article("04152-YZZA6")
```

```python
# ✅ Integration тесты для критичных путей

# tests/integration/test_search_workflow.py

@pytest.mark.integration
@pytest.mark.asyncio
async def test_full_search_workflow():
    """Полный цикл: запрос → каталог → парсинг → анализ"""
    # Этот тест использует реальные сервисы (но тестовое окружение)
    
    query = "Масляный фильтр Toyota Camry 2015"
    
    # 1. Получение артикула
    article = await catalog_service.get_article(query)
    assert article is not None
    
    # 2. Поиск у поставщиков
    results = await browser_service.search_suppliers(article)
    assert len(results) > 0
    
    # 3. AI анализ
    recommendation = await analysis_service.analyze(results)
    assert recommendation.best_offer is not None
```

### Запуск тестов

```bash
# Все тесты
make test

# Только unit
make test-unit

# Только integration
make test-integration

# С покрытием кода
make test-cov
```

### Минимальное покрытие

- **Unit тесты:** 80%+ для новой логики
- **Integration тесты:** Критичные user flows
- **E2E тесты:** Не обязательны для MVP

---

## 📚 Документация

### Что документировать

#### 1. Код (docstrings)

```python
def search_by_vin(vin: str, catalogs: list[str] = None) -> Article:
    """
    Поиск запчастей по VIN-коду.
    
    Использует белорусские каталоги (auto1.by, armtek.by) для получения
    точной информации об автомобиле и доступных запчастях.
    
    Args:
        vin: VIN-код автомобиля, 17 символов
        catalogs: Список каталогов для поиска. 
                  По умолчанию ["auto1", "armtek"]
    
    Returns:
        Article объект с информацией о запчасти
    
    Raises:
        ValueError: Если VIN код невалиден
        CatalogUnavailable: Если все каталоги недоступны
    
    Example:
        >>> article = search_by_vin("JTDBR32E700123456")
        >>> print(article.name)
        "Масляный фильтр Toyota 04152-YZZA6"
    """
```

#### 2. API endpoints

```python
@router.post("/search", response_model=SearchResult)
async def search_parts(query: SearchQuery) -> SearchResult:
    """
    Поиск запчастей у поставщиков.
    
    **Workflow:**
    1. Определение артикула через каталоги
    2. Парсинг цен у поставщиков
    3. AI анализ и рекомендации
    4. Генерация отчёта
    
    **Примеры запросов:**
    ```json
    {
      "query": "Масляный фильтр Toyota Camry 2015",
      "mode": "automatic"
    }
    ```
    
    **Ответ:**
    ```json
    {
      "article": "04152-YZZA6",
      "offers": [...],
      "recommendation": {...}
    }
    ```
    """
```

#### 3. MCP серверы

```markdown
# docs/mcp-servers/catalog-server.md

## Catalog Server

### Описание
MCP сервер для работы с каталогами запчастей.

### Tools

#### get_article_by_vin
Получение артикула по VIN-коду.

**Параметры:**
- `vin` (string): VIN-код, 17 символов
- `catalog` (string, optional): auto1, armtek

**Возвращает:**
```json
{
  "article": "04152-YZZA6",
  "name": "Oil Filter",
  "brand": "Toyota"
}
```

**Пример использования в Gemini CLI:**
```bash
gemini "Найди артикул по VIN JTDBR32E700123456"
```
```

#### 4. Архитектурные решения

Если принимаешь важное решение — задокументируй в `docs/architecture.md`:

```markdown
## ADR: Использование Gemini Proxy для обхода блокировки

**Дата:** 2025-01-15
**Статус:** Принято

### Контекст
Gemini API заблокирован в BY/RU по GEO.

### Решение
Использовать Cloudflare Workers прокси с ротацией ключей.

### Альтернативы
1. VPN (ненадёжно)
2. Другие AI (дороже)

### Последствия
- Стабильный доступ к Gemini
- Небольшая задержка (+50-100ms)
- Зависимость от Cloudflare
```

### Обновление документации

```bash
# При изменении API
make docs

# Локальный просмотр
make docs-serve
# http://localhost:8001
```

---

## ❓ Вопросы и поддержка

### Где задавать вопросы

1. **GitHub Issues** — для багов и feature requests
2. **GitHub Discussions** — для общих вопросов
3. **Email:** [your.email@example.com] — для приватных вопросов

### Полезные ресурсы

- [README.md](README.md) — обзор проекта
- [docs/architecture.md](docs/architecture.md) — архитектура
- [docs/api.md](docs/api.md) — API документация
- [Makefile](Makefile) — все доступные команды

### Labels для Issues

| Label | Описание |
|-------|----------|
| `bug` | Что-то не работает |
| `feature` | Новая функциональность |
| `documentation` | Улучшения документации |
| `good first issue` | Хорошо для новичков |
| `help wanted` | Нужна помощь |
| `question` | Вопрос |
| `wontfix` | Не будем делать |

---

## 🎓 Специфика PartScout

### Белорусский рынок

При работе с каталогами и поставщиками:

- ✅ Используй белорусские источники (auto1.by, armtek.by)
- ✅ Учитывай BYN как валюту
- ✅ Время доставки в днях (не часах)
- ✅ Локальные магазины в радиусе 60км

### Gemini Proxy

Всегда используй прокси, не прямой API:

```python
# ✅ Правильно
client = genai.Client(
    api_key=GEMINI_API_KEY,
    base_url=GEMINI_PROXY_URL,
    headers={"X-Master-Key": GEMINI_PROXY_MASTER_KEY}
)

# ❌ Неправильно (не работает в BY/RU)
client = genai.Client(api_key=GEMINI_API_KEY)
```

### MCP серверы

При создании нового MCP сервера:

1. Создай папку `services/your-service/`
2. Добавь `mcp_server.py`
3. Зарегистрируй в `.env.example`
4. Добавь документацию в `docs/mcp-servers/`
5. Обнови `Makefile` с новыми командами

### Feature Flags

Для экспериментальных функций используй feature flags:

```python
from shared.config import settings

if settings.FEATURE_ASSISTED_SEARCH:
    # Новая функциональность
    result = await assisted_search(query)
else:
    # Старая логика
    result = await basic_search(query)
```

---

## 🏆 Contributors

Спасибо всем кто вносит вклад! 🙏

<!-- ALL-CONTRIBUTORS-LIST:START -->
<!-- Автоматически генерируется, не редактировать вручную -->
<!-- ALL-CONTRIBUTORS-LIST:END -->

Хочешь попасть в список? Сделай свой первый PR!

---

## 📄 Лицензия

Внося вклад в проект, вы соглашаетесь что ваш код будет распространяться под [MIT License](LICENSE).

---

## 🙏 Благодарности

Особая благодарность:

- Сообществу MCP за стандартизацию AI-интеграций
- Google за Gemini API
- Anthropic за Claude и вдохновение
- n8n за отличную платформу автоматизации
- Всем контрибьюторам! ❤️

---

## 📞 Контакты

- **Project Lead:** Kalina
- **Email:** kalinofff@mail.ru
- **Telegram:** @andrey_kalinau(https://t.me/andrey_kalinau)

---

**Снова спасибо за вклад в PartScout!** 🚀

Вместе мы делаем жизнь автосервисов проще! 🔧🤖
```

---
