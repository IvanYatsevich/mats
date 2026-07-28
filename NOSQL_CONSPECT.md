# Полный конспект по NoSQL базам данных

## Оглавление
1. [Что такое NoSQL](#что-такое-nosql)
2. [SQL vs NoSQL](#sql-vs-nosql)
3. [Почему использовать NoSQL](#почему-использовать-nosql)
4. [Четыре основных типа NoSQL баз данных](#четыре-основных-типа-nosql-баз-данных)
5. [Табулярные базы данных](#табулярные-базы-данных)
6. [Документ-ориентированные базы данных](#документ-ориентированные-базы-данных)
7. [Ключ-значение базы данных](#ключ-значение-базы-данных)
8. [Граф базы данных](#граф-базы-данных)

---

## Что такое NoSQL

### Определение
**NoSQL** — это **подход к управлению базами данных**, а не просто определенная технология. Важный момент: **"NoSQL" означает "Not Only SQL"** (не только SQL), а не "No SQL".

### Основные характеристики NoSQL:
1. **Нереляционные** — данные не организованы в таблицы с жесткой схемой
2. **Распределённые** — работают на кластерах машин, глобально распределены
3. **Масштабируемые** — способны хранить и обрабатывать большие объёмы данных
4. **Высокодоступные** — работают даже при отказе некоторых узлов (встроенная репликация)
5. **Отказоустойчивые** — работают при разделении сети

### Гибкость данных
NoSQL поддерживают различные модели данных:
- **Ключ-значение** — простые пары ключ→значение
- **Документы** — JSON-документы с произвольной структурой
- **Колонки** — табулярные данные с динамическими колонками
- **Графы** — данные с узлами и рёбрами, показывающими отношения

### CAP теорема
NoSQL базы данных не могут одновременно иметь все три свойства:
- **C** (Consistency) — согласованность
- **A** (Availability) — доступность
- **P** (Partition tolerance) — отказоустойчивость при разделении сети

**Примеры:**
- MongoDB и Kafka → выбирают **CP** (согласованность и отказоустойчивость)
- Apache Cassandra → выбирает **AP** (доступность и отказоустойчивость)

---

## SQL vs NoSQL

| Аспект | SQL (Реляционные БД) | NoSQL |
|--------|---------------------|-------|
| **Схема** | Жёсtkая, фиксированная | Гибкая, динамическая |
| **Появление** | Конец 1970-х | Около 2000-х |
| **Структура данных** | Таблицы с фиксированными колонками и строками | JSON документы, графы, ключ-значение пары |
| **Типы данных** | Строгие: TEXT, INTEGER, BOOLEAN и т.д. | Гибкие: строки, числа, булевы, массивы, объекты |
| **Масштабирование** | Вертикальное (scaling up) — мощнее CPU, больше RAM | Горизонтальное (scaling out) — больше машин |
| **ACID** | Атомарность, Консистентность, Изоляция, Долговечность | Гибкая согласованность |

### Пример разницы в типах:

```
SQL: Попытка сохранить булево значение в колонку INTEGER будет ОТКЛОНЕНО!

NoSQL: Возможно сохранить:
{
  "id": 1,
  "name": "John",
  "age": 25,           // число
  "email": "john@...", // строка
  "active": true,      // булево
  "tags": ["dev", "java"], // массив
  "metadata": {        // объект
    "lastLogin": "2024-01-15",
    "preferences": {...}
  }
}
```

---

## Почему использовать NoSQL

### Причина 1: Производительность разработки приложений

**Проблема с традиционными БД:**
- Много времени тратится на маппинг данных между структурой приложения и реляционной БД
- Жёсткая схема усложняет изменения

**Решение NoSQL:**
- Модель данных лучше подходит для нужд приложения
- Упрощает отладку и написание кода
- Позволяет быстро эволюционировать структуру данных

**Пример:**
```
Традиционный проект: проверка, валидация, маппинг → сохранение → восстановление → обратное маппинг
NoSQL: данные сохраняются как есть (JSON) → восстанавливаются как есть
```

### Причина 2: Обработка больших объёмов данных

**Проблема:**
- Организации хотят собирать максимум данных для аналитики
- Обработка больших объёмов дорога на традиционных БД

**Решение NoSQL:**
- Разработаны для запуска на дешёвых кластерах (множество простых машин)
- Масштабируются горизонтально (добавить машину = добавить мощность)
- Более экономично, чем масштабирование вертикально (дорогие серверы)

**Пример затрат:**
```
SQL: 1 мегамощный сервер = $100,000+
NoSQL: 100 обычных серверов = $50,000
```

---

## Четыре основных типа NoSQL баз данных

### 1. Табулярные (Columnar / Wide-column) базы данных

**Структура:**
- Данные организованы в таблицы с партиционированием по ключам
- Похожи на SQL таблицы, но с динамическими колонками
- Используют **Cassandra Query Language (CQL)** — очень похож на SQL

**Пример использования:** Apache Cassandra, DataStax Astra

**Ключевые концепции:**
- **Schema** (Схема) — определение колонок и их типов
- **Partition Key** (Ключ партиции) — ключ для распределения данных по узлам
- **Clustering Keys** — ключи для сортировки данных в партиции

#### Практический пример: Таблица книг

```sql
-- Создание таблицы
CREATE TABLE IF NOT EXISTS books (
    book_id UUID,           -- Уникальный ID
    author TEXT,            -- Автор
    title TEXT,             -- Название
    year INT,               -- Год выпуска
    categories SET<TEXT>,   -- Множество категорий
    added TIMESTAMP,        -- Дата добавления
    PRIMARY KEY (book_id)
);

-- Вставка данных
INSERT INTO books (book_id, author, title, year, categories, added)
VALUES (uuid(), 'Bobby Brown', 'Dealing with Tables', 1999, 
        {'programming', 'computers'}, toTimestamp(now()));

-- Получение всех данных
SELECT * FROM books;

-- Поиск по первичному ключу
SELECT * FROM books WHERE book_id = 550e8400-e29b-41d4-a716-446655440000;
```

#### Партиции и кластеризация

```sql
-- Таблица ресторанов, где данные партиционированы по странам
CREATE TABLE IF NOT EXISTS restaurant_by_country (
    country TEXT,      -- Partition Key (распределение по узлам)
    name TEXT,         -- Clustering Key (сортировка)
    cuisine TEXT,      -- Clustering Key
    url TEXT,
    PRIMARY KEY (country, name, cuisine)
);

-- Вставка данных
INSERT INTO restaurant_by_country VALUES ('Poland', 'Wasska', 'Traditional', 'www.wasska.pl');
INSERT INTO restaurant_by_country VALUES ('Singapore', 'The Shark', 'American', 'www.shark.sg');
INSERT INTO restaurant_by_country VALUES ('Singapore', 'The Heart', 'Lebanese', 'www.heart.sg');

-- ВАЖНО: Если две записи имеют то же значение Partition Key,
-- они будут сохранены на одном узле!

-- Получить все рестораны Сингапура
SELECT * FROM restaurant_by_country WHERE country = 'Singapore';
-- Результат: The Shark AND The Heart (обе записи с country='Singapore')
```

**ПреимуществаTabULAR:**
✅ Хорошая производительность для аналитики  
✅ Масштабируемость  
✅ Отказоустойчивость

---

### 2. Документ-ориентированные (Document) базы данных

**Структура:**
- БЕЗ схемы! Можно сохранять объекты любой структуры
- Данные хранятся/возвращаются как JSON документы
- Группы документов = **Коллекции** (вместо таблиц)

**Пример использования:** MongoDB, DataStax Astra (Document API)

**Основное преимущество:** Максимальная гибкость — не нужно определять схему заранее

#### Что такое JSON?
```json
{
  "id": 1,
  "title": "Fix bike",
  "description": "Repair the broken bike",
  "done": false,
  "tags": ["work", "urgent"],
  "metadata": {
    "priority": "high",
    "owner": "John"
  }
}
```

**Типы значений:** строки, числа, булевы, массивы, объекты, null

#### Практический пример: To-Do приложение

```bash
# Создание коллекции (POST запрос)
POST /api/v1/namespaces/document/collections
Headers: {
  "x-cassandra-token": "YOUR_TOKEN"
}
Body: {
  "name": "my_first_collection"
}
Response: 201 Created

# Добавление документа в коллекцию
POST /api/v1/namespaces/document/collections/my_first_collection/documents
Headers: {
  "x-cassandra-token": "YOUR_TOKEN"
}
Body: {
  "id": "1",
  "title": "Make dinner",
  "description": "Make dinner and apologize for breaking flatmate's bike",
  "done": false
}
Response: 201 Created

# Получить все документы из коллекции
GET /api/v1/namespaces/document/collections/my_first_collection
Headers: {
  "x-cassandra-token": "YOUR_TOKEN"
}

# Получить конкретный документ по ID
GET /api/v1/namespaces/document/collections/my_first_collection/documents/1
Headers: {
  "x-cassandra-token": "YOUR_TOKEN"
}

# Фильтрация: получить все задачи с title='Make dinner'
GET /api/v1/namespaces/document/collections/my_first_collection?where={"title": {"$eq": "Make dinner"}}
Headers: {
  "x-cassandra-token": "YOUR_TOKEN"
}

# Удалить документ
DELETE /api/v1/namespaces/document/collections/my_first_collection/documents/1
Headers: {
  "x-cassandra-token": "YOUR_TOKEN"
}
```

**Гибкость JSON:**
```json
// Вы можете иметь документы разной структуры в одной коллекции!

{"id": 1, "title": "Task 1", "done": false}

{"id": 2, "title": "Task 2", "done": true, "completedAt": "2024-01-15"}

{"id": 3, "title": "Task 3", "priority": "high", "tags": ["urgent", "work"]}
// Разные поля в каждом документе!
```

#### API и HTTP методы

**GET** — получить данные
```
GET /api/resource → получить ресурс
```

**POST** — создать данные
```
POST /api/resource + JSON → создать новый ресурс
```

**PUT** — обновить/заменить данные
```
PUT /api/resource/id + JSON → полностью заменить
```

**DELETE** — удалить данные
```
DELETE /api/resource/id → удалить ресурс
```

**Endpoints (конечные точки):**
```
GET    /api/v1/burgers          → все бургеры
POST   /api/v1/burgers          → создать новый бургер
PUT    /api/v1/burgers/5        → обновить бургер с ID=5
DELETE /api/v1/burgers/5        → удалить бургер с ID=5
```

---

### 3. Ключ-значение (Key-Value) базы данных

**Структура:**
- Самая простая модель: **ключ → значение**
- Похожа на хеш-таблицу или словарь
- Доступ ТОЛЬКО по ключу (первичному ключу/partition key)
- Можно иметь несколько колонок со значениями

**Пример использования:** Redis, DynamoDB, Cassandra (в режиме key-value)

**Использование GraphQL для взаимодействия**

#### Практический пример: Инвентарь магазина

```graphql
# Создание таблицы (mutation)
mutation {
  createTable(
    keyspaceName: "key_value"
    tableName: "shop_inventory"
    partitionKeys: [
      { name: "key", type: "text" }
    ]
    values: [
      { name: "value", type: "text" }
    ]
  ) {
    name
  }
}

# Добавление товара (mutation)
mutation {
  insertShopInventory(
    key: "item_001"
    value: "beans"
  ) {
    key
    value
  }
}

# Другие товары
mutation {
  insertShopInventory(key: "item_002", value: "shell")
}

mutation {
  insertShopInventory(key: "item_003", value: "coca_cola")
}

# Получить все товары (query)
query {
  shopInventory {
    key
    value
  }
}
# Результат:
# {
#   "shopInventory": [
#     {"key": "item_001", "value": "beans"},
#     {"key": "item_002", "value": "shell"},
#     {"key": "item_003", "value": "coca_cola"}
#   ]
# }

# Удалить товар (mutation) - ТОЛЬКО по ключу!
mutation {
  deleteShopInventory(key: "item_003")
}

# Для SQL консоли:
USE key_value;
SELECT * FROM shop_inventory;
SELECT * FROM shop_inventory WHERE key = 'item_001';
```

**ВАЖНОЕ ПРЕДОСТЕРЕЖЕНИЕ:**
- Можно удалять ТОЛЬКО по partition key (ключу)
- Нельзя удалять по значению (value)
- Это основное ограничение key-value моделей

---

### 4. Граф (Graph) базы данных

**Структура:**
- **Узлы (Nodes)** — сущности/объекты
- **Рёбра (Edges)** — связи/отношения между узлами
- Идеальны для хранения отношений между данными

**Пример использования:** Neo4j, DataStax Enterprise Graph

**Язык для работы с графами:** **Gremlin** (Apache TinkerPop)

#### Визуальный пример: Социальная сеть

```
Узлы: Люди (You, Friend1, Friend2, Friend3)

Рёбра: Дружба
You ←→ Friend1
You ←→ Friend2
You ←→ Friend3
Friend1 ←→ Friend2
Friend2 ←→ Friend3

Можно добавить рёбра с информацией:
You ---[knows since 2020]--→ Friend1
You ---[works with]--→ Friend2
```

#### Практический пример: Боги и монстры

```gremlin
// Создание узлов разных типов
g.addV('god').property('name', 'Saturn').property('age', 10000).next()
g.addV('god').property('name', 'Jupiter').property('age', 5000).next()
g.addV('demigod').property('name', 'Hercules').property('age', 30).next()
g.addV('human').property('name', 'Prometheus').property('age', 100).next()
g.addV('monster').property('name', 'Hydra').property('age', 500).next()
g.addV('location').property('name', 'Sky').next()

// Создание рёбер (связей)
g.V().has('name', 'Saturn').addE('father').to(g.V().has('name', 'Jupiter')).next()
// Saturn является отцом для Jupiter

g.V().has('name', 'Jupiter').addE('father').to(g.V().has('name', 'Hercules')).next()
// Jupiter является отцом для Hercules

g.V().has('name', 'Jupiter').addE('lives').to(g.V().has('name', 'Sky')).next()
// Jupiter живёт в Sky

g.V().has('name', 'Hercules').addE('battled').to(g.V().has('name', 'Hydra')).next()
// Hercules боролся с Hydra

// Запросы
g.V().has('name', 'Hercules').out('battled').values('name')
// Результат: Hydra

g.V().has('name', 'Jupiter').out('father').values('name')
// Результат: Hercules

g.V().has('name', 'Jupiter').out('lives').values('name')
// Результат: Sky
```

**Граф структура:**
```
Узел: Saturn (god, age: 10000)
  ├─ Ребро: father
  └─→ Узел: Jupiter (god, age: 5000)
       ├─ Ребро: father
       │  └─→ Узел: Hercules (demigod, age: 30)
       │       ├─ Ребро: battled
       │       └─→ Узел: Hydra (monster, age: 500)
       │
       └─ Ребро: lives
          └─→ Узел: Sky (location)
```

**Преимущества графов:**
✅ Быстрые запросы для отношений  
✅ Выглядят интуитивно  
✅ Идеальны для социальных сетей, рекомендаций, маршрутизации

---

## Табулярные базы данных

### Основное понятие: Keyspace (Пространство ключей)
Keyspace = логическое группирование таблиц в NoSQL БД

```
Database "fcc_tutorial"
├── Keyspace "Tabular"
│   ├── Table "books"
│   ├── Table "restaurant_by_country"
│   └── ...
├── Keyspace "document"
│   ├── Collection "my_first_collection"
│   └── ...
└── Keyspace "key_value"
    ├── Table "shop_inventory"
    └── ...
```

### Архитектура БД (3 слоя)

1. **Interface Layer** (Интерфейс)
   - Визуальная платформа для взаимодействия с данными
   - Форматы: SQL, CQL, GraphQL и т.д.

2. **Execution Layer** (Выполнение)
   - Анализирует входящие запросы
   - Диспетчеризует запросы

3. **Storage Layer** (Хранилище)
   - Индексирование данных
   - Физическое хранилище

### Практическое руководство

```sql
-- 1. Просмотр всех keyspaces
DESCRIBE KEYSPACES;

-- 2. Использование конкретного keyspace
USE Tabular;

-- 3. Создание таблицы
CREATE TABLE IF NOT EXISTS books (
    book_id UUID,
    author TEXT,
    title TEXT,
    year INT,
    categories SET<TEXT>,
    added TIMESTAMP,
    PRIMARY KEY (book_id)
);

-- 4. Просмотр структуры keyspace
DESCRIBE KEYSPACE Tabular;

-- 5. Вставка данных
INSERT INTO books (book_id, author, title, year, categories, added)
VALUES (
    uuid(),
    'Bobby Brown',
    'Dealing with Tables',
    1999,
    {'programming', 'computers'},
    toTimestamp(now())
);

-- 6. Выборка всех данных
SELECT * FROM books;

-- 7. Выборка с WHERE (по partition key)
SELECT * FROM books WHERE book_id = 550e8400-e29b-41d4-a716-446655440000;

-- 8. Выборка по partition key (группировка)
SELECT * FROM restaurant_by_country WHERE country = 'Singapore';
```

### Типы данных в CQL

| Тип | Описание | Пример |
|-----|---------|--------|
| TEXT | Строка | 'Bobby Brown' |
| INT | Целое число | 1999 |
| UUID | Уникальный ID | uuid() → 550e8400-e29b-41d4-a716-446655440000 |
| TIMESTAMP | Время | toTimestamp(now()) |
| SET<TEXT> | Множество | {'programming', 'computers'} |
| BOOLEAN | Да/Нет | true, false |

---

## Документ-ориентированные базы данных

### JSON структура

```json
{
  "id": 1,
  "title": "Make dinner",
  "description": "Cook and clean up",
  "done": false,
  "tags": ["cooking", "home"],
  "metadata": {
    "createdAt": "2024-01-15",
    "priority": "high"
  }
}
```

### Преимущества:
- ✅ БЕЗ схемы
- ✅ Гибкая структура
- ✅ Разные документы могут иметь разные поля
- ✅ Естественное представление объектов приложения

### Практическое руководство: REST API

```bash
# 1. Создать колекцию (POST)
curl -X POST https://api.datastax.com/v1/namespaces/document/collections \
  -H "x-cassandra-token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "my_first_collection"}'

# 2. Добавить документ (POST)
curl -X POST https://api.datastax.com/v1/namespaces/document/collections/my_first_collection/documents \
  -H "x-cassandra-token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "1",
    "title": "Make dinner",
    "description": "Apologize for breaking bike",
    "done": false
  }'

# 3. Получить все документы (GET)
curl -X GET https://api.datastax.com/v1/namespaces/document/collections/my_first_collection \
  -H "x-cassandra-token: YOUR_TOKEN"

# 4. Получить документ по ID (GET)
curl -X GET https://api.datastax.com/v1/namespaces/document/collections/my_first_collection/documents/1 \
  -H "x-cassandra-token: YOUR_TOKEN"

# 5. Фильтровать по полю (GET with WHERE)
curl -X GET 'https://api.datastax.com/v1/namespaces/document/collections/my_first_collection?where={"title":{"$eq":"Make dinner"}}&page-size=20' \
  -H "x-cassandra-token: YOUR_TOKEN"

# 6. Обновить документ (PUT)
curl -X PUT https://api.datastax.com/v1/namespaces/document/collections/my_first_collection/documents/1 \
  -H "x-cassandra-token: YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"done": true}'

# 7. Удалить документ (DELETE)
curl -X DELETE https://api.datastax.com/v1/namespaces/document/collections/my_first_collection/documents/1 \
  -H "x-cassandra-token: YOUR_TOKEN"
```

---

## Ключ-значение базы данных

### Основное правило
**Доступ только по ключу (partition key)**

Нельзя:
```graphql
# ❌ ОШИБКА! Не можно удалить по значению
mutation {
  deleteShopInventory(value: "beans")  # ERROR!
}
```

Правильно:
```graphql
# ✅ Правильно! Удаление по ключу
mutation {
  deleteShopInventory(key: "item_001")
}
```

### Полный пример: Полка магазина

```graphql
# GraphQL Query/Mutation примеры

query GetAllItems {
  shopInventory {
    key
    value
  }
}

mutation AddItem($key: String!, $value: String!) {
  insertShopInventory(key: $key, value: $value) {
    key
    value
  }
}

mutation DeleteItem($key: String!) {
  deleteShopInventory(key: $key)
}

# Использование:
# AddItem(key: "BANANAS_001", value: "Fresh Bananas")
# AddItem(key: "APPLES_001", value: "Red Apples")
# AddItem(key: "ORANGES_001", value: "Orange Juice")

# DeleteItem(key: "ORANGES_001")  # ✅ OK
# DeleteItem(value: "Red Apples")  # ❌ ERROR!
```

---

## Граф базы данных

### Основные элементы

**Узлы (Vertices):**
```gremlin
g.addV('person').property('name', 'Alice').property('age', 30)
g.addV('person').property('name', 'Bob').property('age', 25)
g.addV('company').property('name', 'Google').property('founded', 1998)
g.addV('location').property('city', 'San Francisco')
```

**Рёбра (Edges):**
```gremlin
// Создание рёбер с направлением
g.V().has('name', 'Alice').addE('works_at').to(g.V().has('name', 'Google'))
g.V().has('name', 'Bob').addE('works_at').to(g.V().has('name', 'Google'))
g.V().has('name', 'Google').addE('located_in').to(g.V().has('city', 'San Francisco'))
g.V().has('name', 'Alice').addE('knows').to(g.V().has('name', 'Bob'))
```

**Обход графа (Graph Traversal):**
```gremlin
// Кто работает в Google?
g.V().has('name', 'Google')
  .in('works_at')
  .values('name')
// Результат: Alice, Bob

// Где находится Google?
g.V().has('name', 'Google')
  .out('located_in')
  .values('city')
// Результат: San Francisco

// Коллеги Alice (люди, которые работают в одной компании)
g.V().has('name', 'Alice')
  .out('works_at')
  .in('works_at')
  .values('name')
// Результат: Alice, Bob

```

### График отношений: Поколение богов

```
Saturn (god, age: 10000)
    │
    ├─ father → Jupiter (god, age: 5000)
    │              │
    │              ├─ father → Hercules (demigod, age: 30)
    │              │              │
    │              │              └─ battled → Hydra (monster, age: 500)
    │              │
    │              └─ lives → Sky (location)
    │
    └─ brother → Neptune (god, age: 4800)
                    │
                    └─ lives → Sea (location)
```

**Граф Gremlin код:**
```gremlin
// Создание узлов
g.addV('god').property('name', 'Saturn').property('age', 10000)
g.addV('god').property('name', 'Jupiter').property('age', 5000)
g.addV('god').property('name', 'Neptune').property('age', 4800)
g.addV('demigod').property('name', 'Hercules').property('age', 30)
g.addV('monster').property('name', 'Hydra').property('age', 500)
g.addV('location').property('name', 'Sky')
g.addV('location').property('name', 'Sea')

// Создание рёбер
g.V().has('name', 'Saturn').addE('father').to(g.V().has('name', 'Jupiter'))
g.V().has('name', 'Saturn').addE('brother').to(g.V().has('name', 'Neptune'))
g.V().has('name', 'Jupiter').addE('father').to(g.V().has('name', 'Hercules'))
g.V().has('name', 'Jupiter').addE('lives').to(g.V().has('name', 'Sky'))
g.V().has('name', 'Neptune').addE('lives').to(g.V().has('name', 'Sea'))
g.V().has('name', 'Hercules').addE('battled').to(g.V().has('name', 'Hydra'))

// Запросы
g.V().has('name', 'Hercules').out('battled').values('name')        // → Hydra
g.V().has('name', 'Jupiter').out('father').values('name')          // → Hercules
g.V().has('name', 'Saturn').out('father').out('father').values('name') // → Hercules (внук!)
g.V().has('name', 'Hercules').in('father').values('name')          // → Jupiter (отец)
```

---

## Практический сравнительный пример

### Один и тот же сценарий в разных БД

**Сценарий:** Хранение информации о студентах и курсах

#### 1. Табулярная БД (Cassandra/CQL)

```sql
-- Таблица студентов
CREATE TABLE IF NOT EXISTS students (
    student_id UUID,
    name TEXT,
    email TEXT,
    enrollment_date TIMESTAMP,
    PRIMARY KEY (student_id)
);

-- Таблица записей студентов на курсы
CREATE TABLE IF NOT EXISTS enrollments (
    course_id UUID,
    student_id UUID,
    enrollment_date TIMESTAMP,
    grade TEXT,
    PRIMARY KEY (course_id, student_id)
);

INSERT INTO students VALUES (uuid(), 'Alice', 'alice@example.com', toTimestamp(now()));
INSERT INTO students VALUES (uuid(), 'Bob', 'bob@example.com', toTimestamp(now()));

INSERT INTO enrollments VALUES (uuid(), student_id_1, toTimestamp(now()), 'A');
INSERT INTO enrollments VALUES (uuid(), student_id_2, toTimestamp(now()), 'B+');
```

#### 2. Документ-ориентированная БД (MongoDB/JSON)

```json
// Коллекция students
[
  {
    "_id": "student_1",
    "name": "Alice",
    "email": "alice@example.com",
    "enrollmentDate": "2024-01-15",
    "courses": [
      {
        "courseId": "course_1",
        "courseName": "Python Basics",
        "grade": "A"
      },
      {
        "courseId": "course_2",
        "courseName": "Web Development",
        "grade": "A-"
      }
    ]
  },
  {
    "_id": "student_2",
    "name": "Bob",
    "email": "bob@example.com",
    "enrollmentDate": "2024-01-16",
    "courses": [
      {
        "courseId": "course_1",
        "courseName": "Python Basics",
        "grade": "B+"
      }
    ]
  }
]
```

#### 3. Ключ-значение БД (Redis)

```
key: "student:student_1"
value: "{\"name\":\"Alice\",\"email\":\"alice@example.com\",\"courses\":[\"course_1\",\"course_2\"]}"

key: "student:student_2"
value: "{\"name\":\"Bob\",\"email\":\"bob@example.com\",\"courses\":[\"course_1\"]}"

key: "course:course_1:students"
value: "[\"student_1\",\"student_2\"]"
```

#### 4. Граф БД (Neo4j/Gremlin)

```
Student(name: "Alice")
    │
    ├─ enrolled_in → Course(name: "Python Basics")
    │                    │
    │                    └─ has_grade → Grade(value: "A")
    │
    └─ enrolled_in → Course(name: "Web Development")
                         └─ has_grade → Grade(value: "A-")

Student(name: "Bob")
    │
    └─ enrolled_in → Course(name: "Python Basics")
                         └─ has_grade → Grade(value: "B+")
```

---

## Рекомендации по использованию

### Используйте Табулярные БД (Cassandra) когда:
- 📊 Нужна масштабируемая аналитика
- 📈 Большие объёмы временных рядов
- 🔍 Частые запросы по одному ключу
- 💾 Высокая пропускная способность записи

**Примеры:**
- Логирование событий
- Метрики мониторинга
- Временные ряды IoT данных

### Используйте Document-ориентированные БД (MongoDB) когда:
- 📝 Данные имеют сложную структуру
- 🔄 Частая эволюция схемы
- 🚀 Быстрая разработка прототипов
- 🎯 Данные лучше представляются объектами

**Примеры:**
- CMS системы
- Профили пользователей
- каталоги товаров
- To-Do приложения

### Используйте Key-Value БД (Redis) когда:
- ⚡ Нужна максимальная скорость
- 💾 Кеширование данных
- 📊 Простая структура данных
- 🔑 Доступ ТОЛЬКО по ключу

**Примеры:**
- Кеширование сессий
- Счётчики и рейтинги
- Очереди задач
- Инвентарь магазина

### Используйте Граф БД (Neo4j) когда:
- 🔗 Данные с много связей
- 👥 Социальные сети
- 🗺️ Поиск кратчайших путей
- 🎯 Рекомендательные системы

**Примеры:**
- Социальные сети
- Рекомендация друзей
- Поиск маршрутов
- Системы управления доступом
- Карты знаний

---

## Итоговая таблица сравнения

| Характеристика | Табулярная | Документы | Ключ-Значение | Граф |
|---|---|---|---|---|
| **Сложность данных** | Средняя | Высокая | Низкая | Высокая |
| **Гибкость схемы** | Средняя | Высокая | Нет | Средняя |
| **Скорость запросов** | Хорошая | Хорошая | Отличная | Хорошая (для связей) |
| **Масштабируемость** | Отличная | Хорошая | Отличная | Средняя |
| **Используемый язык** | CQL | JSON/API | Commands | Gremlin |
| **Лучше для** | Аналитика, TS | Приложения | Кеширование | Отношения |
| **Примеры БД** | Cassandra | MongoDB | Redis | Neo4j |

---

## Выводы

1. **NoSQL** — это не один стандарт, а подход с 4+ основными моделями
2. Каждый тип имеет свои **сильные и слабые стороны**
3. Выбор типа зависит от **характеристик данных и запросов**
4. Гибкость NoSQL делает разработку **быстрее**, а масштабирование **проще**
5. SQL vs NoSQL — это nicht "или-или", а "когда что использовать"

**Успехов в изучении NoSQL! 🚀**

