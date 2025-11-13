# 🇧🇾 Вопросы по адаптации PartScout для белорусского рынка

## 📋 Структурированные вопросы

### **Вопрос 1: Географическая специфика и выбор каталога**

**Проблема:** 
Я первоначально предложил **exist.ru** как авторитетный каталог запчастей. Однако деятельность автосервиса осуществляется в **Беларуси**. 

Согласно справке Perplexity, exist.ru предоставляет API доступ через менеджеров, но заказывать запчасти из России в Беларусь вряд ли целесообразно.

**Вопрос:** Какие белорусские альтернативы каталогов существуют и подходят для интеграции?

**Справка Perplexity:**
```
Белорусские сервисы с API:

1. auto1.by
   - WebAPI для поиска по артикулу/бренду
   - Аутентификация: логин + пароль
   - Информация о складских запасах
   - Поиск по точному совпадению

2. armtek.by
   - Подбор по VIN-коду
   - Подбор по марке/модели
   - Подбор по номеру кузова
   - Минимизация ошибок подбора

3. zapchasti24.by, av-parts.by
   - Онлайн-подбор по VIN, модели, артикулу
   - API не всегда явно указано
   - Ориентированы на точный поиск

Требования к API:
- Регистрация + API-ключ (логин/пароль)
- Передача: артикул, марка, модель, VIN
- Форматы: JSON/XML
```

---

### **Вопрос 2: Проблема не-заводских артикулов**

**Проблема:** 
Каталоги обычно имеют хороший встроенный поиск на сайте. Но что делать, если **сайт поставщика не использует заводские артикулы**, а свою внутреннюю номенклатуру?

**Предложенное решение:**
Создавать собственную базу данных, где будут указаны **посадочные страницы сайтов** на конкретные детали с маппингом:
```
Заводской артикул → URL страницы конкретного поставщика
```

**Вопрос:** Может ли здесь помочь RAG для поиска соответствий?

---

### **Вопрос 3: Применение RAG с существующим n8n решением**

**Контекст:**
У меня уже реализовано RAG-решение на **n8n** со следующим функционалом:

#### Архитектура существующего решения:

**1. Мониторинг документов (Google Drive):**
```
Google Drive → n8n Webhook
├── Отслеживание добавления документов
├── Отслеживание изменений документов
└── Автоматическая обработка
```

**2. Обработка по типам документов:**

```yaml
Текстовые форматы (текст, markdown, PDF, DOCX):
  → Embeddings
  → Supabase: запись с метаданными + векторы

Excel/табличные данные:
  → НЕ embeddings
  → Supabase: запись в таблице JSON
  → Сохранение схемы извлечения данных
  → Возможность SQL-запросов
```

**3. Агентный поиск информации:**

```
Запрос пользователя
    ↓
AI Агент принимает решение:
    ├── Инструмент 1: Векторный поиск (embeddings)
    ├── Инструмент 2: Получить список документов
    ├── Инструмент 3: Получить содержимое оригинального документа
    └── Инструмент 4: SQL-запрос (для табличных данных)
        └── Пример: "Выборка за прошлый год + расчет среднего чека"
```

**4. Преимущества гибридного подхода:**

```
Классический RAG (только embeddings):
❌ Может ошибаться
❌ Плохо работает с точными данными
❌ Не подходит для вычислений

Гибридный подход (SQL + векторы + графы):
✅ Более точный
✅ Работает с числовыми данными
✅ Поддерживает аналитику
✅ Агент выбирает оптимальный инструмент
```

**Вопрос:** Как интегрировать это решение в PartScout для:
- Хранения маппингов артикулов на URLs поставщиков
- Работы с прайс-листами поставщиков (Excel)
- Поиска соответствий не-заводских артикулов

---

## ✅ Ответы и рекомендации

### **Ответ 1: Белорусские каталоги - стратегия интеграции**

#### Рекомендуемая архитектура для Беларуси:

```python
# services/catalog/catalog_service.py

class CatalogService:
    """Мульти-каталожный сервис для белорусского рынка"""
    
    def __init__(self):
        self.providers = {
            'auto1': Auto1Provider(),      # Приоритет 1
            'armtek': ArmtekProvider(),    # Приоритет 2
            'zapchasti24': Zapchasti24Provider(),  # Резерв
        }
    
    async def get_article_by_vin(self, vin: str) -> List[Part]:
        """Получение артикулов по VIN через белорусские каталоги"""
        results = []
        
        # Пробуем по приоритету
        for name, provider in self.providers.items():
            try:
                parts = await provider.search_by_vin(vin)
                if parts:
                    results.extend(parts)
                    break  # Нашли - не идем дальше
            except Exception as e:
                logger.warning(f"{name} failed: {e}")
                continue
        
        return self._deduplicate_parts(results)
```

#### MCP Server для белорусских каталогов:

```python
# services/catalog/mcp_server.py

from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("partscout-catalog-by")

@server.tool()
async def search_by_vin(vin: str, preferred_catalog: str = "auto1") -> dict:
    """
    Поиск запчастей по VIN в белорусских каталогах
    
    Args:
        vin: VIN-код автомобиля (17 символов)
        preferred_catalog: auto1, armtek, zapchasti24
    
    Returns:
        {
            "parts": [...],
            "vehicle_info": {...},
            "source": "auto1.by"
        }
    """
    catalog = get_catalog(preferred_catalog)
    return await catalog.search_by_vin(vin)

@server.tool()
async def search_by_article(article: str, brand: str = None) -> dict:
    """
    Поиск по артикулу через все каталоги с fallback
    """
    # Пробуем auto1.by (точное совпадение)
    result = await auto1.search(article=article, brand=brand)
    if result:
        return result
    
    # Fallback на armtek.by
    result = await armtek.search(article=article)
    return result or {"error": "Not found in any catalog"}
```

#### Конкретные провайдеры:

```python
# services/catalog/providers/auto1_by.py

import httpx
from typing import List, Dict

class Auto1Provider:
    """
    Интеграция с auto1.by WebAPI
    Документация: https://auto1.by/api (предположительно)
    """
    
    BASE_URL = "https://api.auto1.by/v1"
    
    def __init__(self, login: str, password: str):
        self.auth = (login, password)
        self.session = httpx.AsyncClient()
    
    async def authenticate(self):
        """Получение токена авторизации"""
        response = await self.session.post(
            f"{self.BASE_URL}/auth",
            json={"login": self.login, "password": self.password}
        )
        self.token = response.json()["access_token"]
    
    async def search_by_article(
        self, 
        article: str, 
        brand: str = None
    ) -> List[Dict]:
        """
        Поиск по точному совпадению артикула
        
        API endpoint: /search
        Метод: POST
        Формат: JSON
        """
        payload = {
            "article": article,
            "searchType": "exact"  # точное совпадение
        }
        
        if brand:
            payload["brand"] = brand
        
        response = await self.session.post(
            f"{self.BASE_URL}/search",
            json=payload,
            headers={"Authorization": f"Bearer {self.token}"}
        )
        
        data = response.json()
        
        return [
            {
                "article": item["article"],
                "brand": item["brand"],
                "name": item["name"],
                "price": item["price"],
                "stock": item["stock"],  # складские запасы
                "warehouse": item["warehouse"],
                "delivery_days": item["deliveryDays"],
                "source": "auto1.by"
            }
            for item in data.get("items", [])
        ]
```

```python
# services/catalog/providers/armtek_by.py

class ArmtekProvider:
    """
    Интеграция с armtek.by
    Специализация: VIN-декодирование + подбор
    """
    
    BASE_URL = "https://api.armtek.by"
    
    async def search_by_vin(self, vin: str) -> Dict:
        """
        Подбор по VIN-коду
        
        Преимущество: минимизация ошибок подбора
        Возвращает: точную модификацию авто + каталог деталей
        """
        # 1. Декодируем VIN
        vehicle_info = await self._decode_vin(vin)
        
        # 2. Получаем каталог деталей для этого авто
        parts_catalog = await self._get_parts_catalog(
            make=vehicle_info["make"],
            model=vehicle_info["model"],
            year=vehicle_info["year"],
            engine=vehicle_info["engine"]
        )
        
        return {
            "vehicle": vehicle_info,
            "parts": parts_catalog,
            "source": "armtek.by"
        }
    
    async def _decode_vin(self, vin: str) -> Dict:
        """Расшифровка VIN-кода"""
        response = await self.session.get(
            f"{self.BASE_URL}/vin/decode",
            params={"vin": vin}
        )
        return response.json()
```

#### Стратегия выбора каталога:

```python
# Логика выбора для разных сценариев

def choose_catalog(query_type: str) -> str:
    """Выбор оптимального каталога"""
    
    strategies = {
        # Есть VIN → armtek (лучший VIN-декодер)
        "vin": "armtek",
        
        # Точный артикул + бренд → auto1 (точное совпадение)
        "exact_article": "auto1",
        
        # Марка/модель → armtek (хороший каталог)
        "model": "armtek",
        
        # Fallback → auto1 (самый большой ассортимент?)
        "default": "auto1"
    }
    
    return strategies.get(query_type, "auto1")
```

---

### **Ответ 2: Проблема не-заводских артикулов + RAG**

#### Проблема более детально:

```
Сценарий:
1. Заводской артикул: 04152-YZZA6 (Toyota фильтр)
2. Поставщик "АвтоМагБел" использует: AM-F-1234
3. Каталоги не знают про AM-F-1234
4. Поиск по заводскому артикулу на сайте поставщика не работает

Нужно:
├── Маппинг: 04152-YZZA6 → AM-F-1234
└── Или: 04152-YZZA6 → https://automagbel.by/filters/am-f-1234
```

#### ✅ **RAG ИДЕАЛЬНО подходит для этой задачи!**

Вот почему твое существующее решение решает проблему:

---

### **Ответ 3: Интеграция RAG в PartScout**

#### Архитектура с твоим n8n RAG:

```
┌─────────────────────────────────────────────────────────┐
│              PARTSCOUT + RAG INTEGRATION                │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ 1. НАКОПЛЕНИЕ ЗНАНИЙ (Google Drive → n8n → Supabase)   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Google Drive (папка PartScout):                       │
│  ├── /Маппинги поставщиков/                            │
│  │   ├── АвтоМагБел_маппинг.xlsx                       │
│  │   │   Колонки: [Заводской артикул | Артикул         │
│  │   │             поставщика | URL | Название]        │
│  │   │                                                  │
│  │   ├── ЗапчастиПлюс_каталог.pdf                      │
│  │   └── ОбщиеПравилаПодбора.md                        │
│  │                                                      │
│  ├── /Прайс-листы/                                     │
│  │   ├── Поставщик1_прайс_2025.xlsx                    │
│  │   └── Поставщик2_прайс_2025.xlsx                    │
│  │                                                      │
│  └── /База знаний/                                     │
│      ├── СовместимостьДеталей.md                       │
│      ├── ЧастоЗадаваемыеВопросы.docx                   │
│      └── ИсторияПроблемныхСлучаев.txt                  │
│                                                         │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│ 2. n8n WORKFLOW: Обработка документов                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  [Google Drive Trigger]                                │
│         ↓                                              │
│  [Определить тип файла]                                │
│         ↓                                              │
│    ┌────┴─────┐                                        │
│    ▼          ▼                                        │
│  [Excel]   [Текст/PDF/MD]                             │
│    │          │                                        │
│    │          ├→ [Create Embeddings]                   │
│    │          └→ [Supabase: vectors table]            │
│    │                                                    │
│    └→ [Parse Excel]                                    │
│       └→ [Supabase: suppliers_mapping table]          │
│          Структура:                                    │
│          {                                             │
│            original_article: "04152-YZZA6",            │
│            supplier_article: "AM-F-1234",              │
│            supplier_name: "АвтоМагБел",                │
│            url: "https://...",                         │
│            price: 450,                                 │
│            metadata: {...}                             │
│          }                                             │
│                                                        │
└─────────────────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│ 3. MCP SERVER: RAG + SQL Tools                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  @server.tool()                                        │
│  async def find_supplier_mapping(article: str):        │
│      """                                               │
│      Найти соответствие артикула у поставщиков         │
│                                                         │
│      Агент решает:                                     │
│      1. Есть в маппинге? → SQL запрос                  │
│      2. Нет в маппинге? → Векторный поиск похожих      │
│      3. Совсем не нашли? → Поиск в документах (RAG)    │
│      """                                               │
│                                                         │
│      # 1. Точный поиск в таблице маппингов             │
│      sql_result = await supabase.rpc(                  │
│          'find_exact_mapping',                         │
│          {'article': article}                          │
│      )                                                 │
│                                                         │
│      if sql_result:                                    │
│          return sql_result                             │
│                                                         │
│      # 2. Векторный поиск похожих артикулов            │
│      embedding = await get_embedding(article)          │
│      similar = await supabase.rpc(                     │
│          'match_similar_articles',                     │
│          {'query_embedding': embedding, 'limit': 5}    │
│      )                                                 │
│                                                         │
│      if similar:                                       │
│          return similar                                │
│                                                         │
│      # 3. RAG поиск в документах                       │
│      docs = await vector_search_docs(article)          │
│      context = "\n".join([d.content for d in docs])    │
│                                                         │
│      answer = await gemini.generate(                   │
│          prompt=f"""                                   │
│          Найди информацию об артикуле {article}        │
│          в следующих документах:                       │
│                                                         │
│          {context}                                     │
│          """                                           │
│      )                                                 │
│                                                         │
│      return answer                                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

#### Конкретные таблицы Supabase:

```sql
-- 1. Маппинги артикулов (из Excel прайсов)
CREATE TABLE suppliers_mapping (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    original_article TEXT NOT NULL,  -- Заводской
    supplier_id UUID REFERENCES suppliers(id),
    supplier_article TEXT,  -- Артикул поставщика
    supplier_name TEXT,
    url TEXT,  -- Прямая ссылка на товар
    price DECIMAL(10,2),
    stock_quantity INTEGER,
    metadata JSONB,  -- Любая доп. информация
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Индексы для быстрого поиска
CREATE INDEX idx_original_article ON suppliers_mapping(original_article);
CREATE INDEX idx_supplier_article ON suppliers_mapping(supplier_article);
CREATE INDEX idx_supplier_name ON suppliers_mapping(supplier_name);

-- 2. Векторы документов (PDF, MD, DOCX)
CREATE TABLE knowledge_base (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_name TEXT,
    document_type TEXT,  -- pdf, markdown, docx, text
    content TEXT,
    embedding vector(1536),  -- Gemini embeddings
    metadata JSONB,
    google_drive_id TEXT,  -- ID файла в Drive
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX ON knowledge_base 
USING ivfflat (embedding vector_cosine_ops);

-- 3. История прайсов (для аналитики)
CREATE TABLE price_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    article TEXT,
    supplier_id UUID,
    price DECIMAL(10,2),
    stock_quantity INTEGER,
    date DATE DEFAULT CURRENT_DATE
);

-- Для SQL-запросов типа "средняя цена за прошлый год"
CREATE INDEX idx_price_article_date 
ON price_history(article, date DESC);
```

---

#### n8n Workflow для обработки прайсов:

```json
{
  "name": "PartScout: Process Supplier Pricelists",
  "nodes": [
    {
      "name": "Google Drive Trigger",
      "type": "@n8n/n8n-nodes-google-drive.GoogleDriveTrigger",
      "parameters": {
        "folderId": "{{ $env.GDRIVE_PRICELISTS_FOLDER }}",
        "event": "file.created,file.updated"
      }
    },
    {
      "name": "Check File Type",
      "type": "@n8n/n8n-nodes-base.switch",
      "parameters": {
        "rules": [
          {
            "condition": "{{ $json.mimeType }} === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'"
          },
          {
            "condition": "{{ $json.mimeType }} === 'application/pdf'"
          }
        ]
      }
    },
    {
      "name": "Parse Excel Pricelist",
      "type": "@n8n/n8n-nodes-base.spreadsheetFile",
      "parameters": {
        "operation": "read",
        "options": {
          "headerRow": 1
        }
      }
    },
    {
      "name": "Transform to Mapping Format",
      "type": "@n8n/n8n-nodes-base.code",
      "parameters": {
        "jsCode": `
          const items = [];
          const supplierName = $input.item.json.name.split('_')[0];
          
          for (const row of $input.all()) {
            items.push({
              original_article: row.json['Артикул OEM'],
              supplier_article: row.json['Артикул поставщика'],
              supplier_name: supplierName,
              url: row.json['Ссылка'],
              price: parseFloat(row.json['Цена']),
              stock_quantity: parseInt(row.json['Наличие']),
              metadata: {
                brand: row.json['Бренд'],
                name: row.json['Название']
              }
            });
          }
          
          return items;
        `
      }
    },
    {
      "name": "Supabase: Upsert Mappings",
      "type": "@n8n/n8n-nodes-supabase.Supabase",
      "parameters": {
        "operation": "upsert",
        "table": "suppliers_mapping",
        "options": {
          "onConflict": "original_article,supplier_name"
        }
      }
    },
    {
      "name": "Create Embeddings (PDF/Docs)",
      "type": "@n8n/n8n-nodes-langchain.embeddingsGoogleGemini",
      "parameters": {
        "modelName": "text-embedding-004"
      }
    },
    {
      "name": "Supabase: Insert Vectors",
      "type": "@n8n/n8n-nodes-supabase.Supabase",
      "parameters": {
        "operation": "insert",
        "table": "knowledge_base"
      }
    }
  ]
}
```

---

#### MCP Server: RAG + SQL гибрид

```python
# services/rag/mcp_server.py

from mcp.server import Server
from supabase import create_client
import google.generativeai as genai

server = Server("partscout-rag")

@server.tool()
async def smart_article_search(
    article: str,
    search_strategy: str = "auto"  # auto, sql, vector, rag
) -> dict:
    """
    Умный поиск артикула с выбором стратегии
    
    Стратегии:
    - sql: Точный поиск в таблице маппингов
    - vector: Семантический поиск похожих артикулов
    - rag: Поиск информации в документах
    - auto: Агент сам решает (рекомендуется)
    """
    
    if search_strategy == "auto":
        # Агент Gemini решает какой инструмент использовать
        decision = await agent_decide_strategy(article)
        search_strategy = decision["strategy"]
    
    if search_strategy == "sql":
        return await sql_exact_search(article)
    
    elif search_strategy == "vector":
        return await vector_similar_search(article)
    
    elif search_strategy == "rag":
        return await rag_document_search(article)


async def sql_exact_search(article: str) -> dict:
    """Точный SQL поиск в маппингах"""
    result = await supabase.rpc(
        'find_supplier_mappings',
        {'search_article': article}
    )
    
    return {
        "strategy": "sql",
        "exact_match": True,
        "results": result.data
    }


async def vector_similar_search(article: str, threshold: float = 0.8) -> dict:
    """Векторный поиск похожих артикулов"""
    
    # Создаем embedding запроса
    embedding = genai.embed_content(
        model="models/text-embedding-004",
        content=article
    )["embedding"]
    
    # Поиск похожих в векторной БД
    result = await supabase.rpc(
        'match_similar_articles',
        {
            'query_embedding': embedding,
            'match_threshold': threshold,
            'match_count': 10
        }
    )
    
    return {
        "strategy": "vector",
        "exact_match": False,
        "similarity_threshold": threshold,
        "results": result.data
    }


async def rag_document_search(article: str) -> dict:
    """RAG поиск в документах базы знаний"""
    
    # 1. Векторный поиск релевантных документов
    embedding = genai.embed_content(
        model="models/text-embedding-004",
        content=f"Информация об артикуле {article}"
    )["embedding"]
    
    docs = await supabase.rpc(
        'match_documents',
        {
            'query_embedding': embedding,
            'match_count': 5
        }
    )
    
    # 2. Формируем контекст из найденных документов
    context = "\n\n---\n\n".join([
        f"Документ: {doc['document_name']}\n{doc['content']}"
        for doc in docs.data
    ])
    
    # 3. Gemini анализирует контекст
    model = genai.GenerativeModel('gemini-2.0-flash-exp')
    response = model.generate_content(f"""
    Проанализируй следующие документы и найди информацию об артикуле {article}.
    
    Что нужно найти:
    - Артикулы поставщиков (если есть маппинг)
    - URLs на страницы товара
    - Совместимые аналоги
    - Любую полезную информацию
    
    Документы:
    {context}
    
    Ответ дай в формате JSON:
    {{
      "found": true/false,
      "mappings": [...],
      "urls": [...],
      "notes": "..."
    }}
    """)
    
    return {
        "strategy": "rag",
        "exact_match": False,
        "ai_analysis": response.text,
        "source_documents": [doc['document_name'] for doc in docs.data]
    }


@server.tool()
async def analyze_pricelist(
    supplier_name: str,
    time_period: str = "last_month",
    query: str = None
) -> dict:
    """
    Анализ прайс-листа через SQL
    
    Примеры запросов:
    - "Средняя цена за прошлый год"
    - "Топ-10 самых дорогих деталей"
    - "Динамика цен на масляные фильтры"
    """
    
    # Gemini генерирует SQL на основе естественного запроса
    sql_query = await generate_sql_from_natural_language(
        query=query,
        table="price_history",
        filters={"supplier_name": supplier_name}
    )
    
    # Выполняем SQL
    result = await supabase.rpc('execute_analytics_query', {
        'sql': sql_query
    })
    
    return {
        "query": query,
        "generated_sql": sql_query,
        "results": result.data
    }
```

---

#### Пример использования в Gemini CLI:

```bash
# Менеджер в терминале
$ gemini "Найди масляный фильтр 04152-YZZA6 у белорусских поставщиков"

# Под капотом:
1. catalog-server: Ищем в auto1.by, armtek.by
   → Находим базовую информацию
   
2. rag-server.smart_article_search("04152-YZZA6")
   → Агент решает: сначала SQL (быстро)
   → SQL находит маппинги:
      - АвтоМагБел: AM-F-1234, 450 BYN, https://...
      - ЗапчастиПлюс: ZP-5678, 470 BYN, https://...
   
3. browser-server: Парсим найденные URLs
   → Актуальные цены и наличие
   
4. analysis-server: Сравниваем предложения
   
5. report-server: Генерируем отчет

# Менеджер получает:
✅ Найдено в каталогах: 3 варианта
✅ Найдено через RAG маппинг: 2 локальных поставщика
✅ Всего: 5 предложений
📄 Отчет готов
```

---

### **Итоговая архитектура с RAG:**

```
┌─────────────────────────────────────────────────────┐
│                  PARTSCOUT + RAG                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Слой 1: Каталоги (структурированные данные)       │
│  ├── auto1.by API                                  │
│  ├── armtek.by API                                 │
│  └── zapchasti24.by (парсинг)                      │
│                                                     │
│  Слой 2: RAG (неструктурированные + маппинги)      │
│  ├── Векторный поиск (pgvector)                    │
│  ├── SQL для Excel таблиц                          │
│  └── Гибридный поиск (агент выбирает)              │
│                                                     │
│  Слой 3: Парсинг (прямой доступ к сайтам)         │
│  └── Browser-server (Playwright + MCP)             │
│                                                     │
│  Слой 4: AI Анализ                                 │
│  └── Gemini 2.0 Flash (сравнение всех источников)  │
│                                                     │
└─────────────────────────────────────────────────────┘

Workflow:
1. Запрос → Каталоги (быстро, точно)
2. Если не нашли → RAG (маппинги, документы)
3. Если нашли URLs → Browser парсинг (актуальные данные)
4. Все результаты → AI анализ → Отчет
```

---

## 🎯 Финальные рекомендации

### **1. Стек для белорусского рынка:**

```yaml
Каталоги:
  ✅ auto1.by - приоритет (API + точный поиск)
  ✅ armtek.by - VIN-декодирование
  ⏳ zapchasti24.by - резерв (парсинг)

RAG (твое решение на n8n):
  ✅ Google Drive мониторинг
  ✅ Excel → SQL (прайсы, маппинги)
  ✅ PDF/MD/DOCX → Embeddings
  ✅ Supabase pgvector
  ✅ Агентный выбор инструмента

Парсинг:
  ✅ Playwright + MCP
  ✅ Для сайтов без API
```

### **2. RAG - обязательно использовать!**

```
Причины:
✅ Решает проблему не-заводских артикулов
✅ Работа с прайс-листами Excel (у тебя уже есть)
✅ База знаний о совместимости
✅ SQL для аналитики цен
✅ Гибридный поход точнее чистого RAG
```

### **3. Приоритет разработки:**

```
Phase 1 MVP:
1. Интеграция auto1.by + armtek.by (API)
2. Интеграция твоего n8n RAG workflow
3. Supabase таблицы (маппинги + векторы)
4. MCP серверы (catalog + rag + browser)
5. Gemini CLI настройка

Phase 2:
1. Добавить больше поставщиков (парсинг)
2. Расширить базу знаний (Google Drive)
3. Аналитика цен (SQL на price_history)
```

---

## ❓ Следующие шаги

1. **Получить доступ к auto1.by и armtek.by API** (регистрация, тестовые ключи)
2. **Адаптировать твой n8n workflow** под структуру PartScout
3. **Создать схему таблиц Supabase** (suppliers_mapping + knowledge_base)
4. **Настроить Google Drive** (папки для прайсов и документов)

**Готов создавать файлы?** Начнем с:
- Схемы БД Supabase (SQL миграции)
- n8n workflow (адаптация твоего решения)
- MCP серверы (catalog-by + rag)
