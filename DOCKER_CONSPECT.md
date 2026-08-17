# Docker — Полный конспект

---

## Оглавление

1. [Что такое Docker?](#1-что-такое-docker)
2. [Какие проблемы решает Docker](#2-какие-проблемы-решает-docker)
3. [Виртуальная машина vs Docker](#3-виртуальная-машина-vs-docker)
4. [Установка Docker](#4-установка-docker)
5. [Docker Images vs Containers](#5-docker-images-vs-containers)
6. [Docker Registries (Реестры)](#6-docker-registries-реестры)
7. [Версии Docker-образов (Image Tags)](#7-версии-docker-образов-image-tags)
8. [Основные команды: pull и run](#8-основные-команды-pull-и-run)
9. [Port Binding (Привязка портов)](#9-port-binding-привязка-портов)
10. [Запуск и остановка контейнеров](#10-запуск-и-остановка-контейнеров)
11. [Приватные Docker-реестры](#11-приватные-docker-реестры)
12. [Registry vs Repository](#12-registry-vs-repository)
13. [Dockerfile — Dockerize Node.js приложения](#13-dockerfile--dockerize-nodejs-приложения)
14. [Сборка образа (Build Image)](#14-сборка-образа-build-image)
15. [Docker UI Client](#15-docker-ui-client)
16. [Docker в полном цикле разработки](#16-docker-в-полном-цикле-разработки)

---

## 1. Что такое Docker?

**Docker** — это программное обеспечение для виртуализации, которое значительно упрощает процесс разработки и развёртывания приложений.

Docker упаковывает приложение в **контейнер**, который содержит:
- Код приложения
- Все зависимости и библиотеки
- Runtime-окружение (Node.js, JRE и т.д.)
- Конфигурацию окружения

Таким образом, приложение и его окружение упакованы в **единый пакет**, который можно легко распространять и запускать на любой машине.

---

## 2. Какие проблемы решает Docker

### 🔴 Проблемы без Docker

#### На этапе разработки:
- Каждый разработчик должен **вручную устанавливать** все сервисы (БД, брокеры сообщений и т.д.) на свой компьютер
- Инструкция по установке **различается** в зависимости от ОС (macOS, Windows, Linux)
- Многошаговая установка — высокий шанс ошибки
- При наличии 10 сервисов — 10 отдельных установок

```
# Пример: чтобы запустить проект локально без Docker, нужно вручную:
# 1. Установить PostgreSQL под свою ОС
# 2. Установить Redis под свою ОС
# 3. Установить RabbitMQ под свою ОС
# ... и так для каждого сервиса
```

#### На этапе деплоя (без контейнеров):
- Dev-команда передаёт **артефакт + текстовую инструкцию** ops-команде
- Ops-команда устанавливает и настраивает всё вручную
- Конфликты версий зависимостей
- Ошибки из-за недопонимания между командами

---

### ✅ Как Docker решает эти проблемы

#### На этапе разработки:
- Не нужно устанавливать сервисы на ОС — они запускаются как **изолированные контейнеры**
- Одна и та же команда работает на любой ОС:

```bash
# Запустить PostgreSQL одной командой — на любой ОС одинаково!
docker run postgres

# Запустить Redis
docker run redis

# Запустить RabbitMQ
docker run rabbitmq
```

- Можно запускать **разные версии одного сервиса** без конфликтов:

```bash
docker run postgres:14
docker run postgres:15
```

#### На этапе деплоя:
- Разработчики упаковывают приложение **вместе с конфигурацией** в Docker-образ
- Ops-команда просто запускает контейнер — не нужно ничего настраивать вручную
- Единственное, что нужно установить на сервер — **Docker runtime** (один раз)

---

## 3. Виртуальная машина vs Docker

### Структура ОС
Операционная система состоит из двух слоёв:
1. **Ядро (Kernel)** — взаимодействует с железом (CPU, RAM, диск)
2. **Слой приложений** — программы, которые работают поверх ядра

### Что виртуализирует каждая технология?

| Характеристика | Виртуальная машина | Docker |
|---|---|---|
| Что виртуализируется | ОС целиком (ядро + приложения) | Только слой приложений |
| Собственное ядро | ✅ Да | ❌ Нет (использует ядро хоста) |
| Размер образа | Гигабайты | Мегабайты |
| Время запуска | Минуты | Миллисекунды |
| Запуск Linux-образа на Windows | ✅ Напрямую | ⚠️ Через Docker Desktop |

### Docker на Windows / macOS

Большинство Docker-образов основаны на **Linux**. Docker был изначально написан для Linux.

**Docker Desktop** решает проблему совместимости:
- Использует **гипервизор** с лёгким Linux-дистрибутивом
- Предоставляет нужное Linux-ядро для контейнеров
- Позволяет запускать Linux-контейнеры на Windows и macOS

```
┌────────────────────────────────────────────────┐
│               Windows / macOS                   │
│  ┌──────────────────────────────────────────┐  │
│  │         Docker Desktop (Hypervisor)       │  │
│  │  ┌────────────────────────────────────┐  │  │
│  │  │    Lightweight Linux (ядро)         │  │  │
│  │  │  ┌──────────┐  ┌──────────────┐   │  │  │
│  │  │  │ Container│  │  Container   │   │  │  │
│  │  │  └──────────┘  └──────────────┘   │  │  │
│  │  └────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

---

## 4. Установка Docker

1. Перейти на официальный сайт: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Выбрать версию для своей ОС (Windows / macOS Intel / macOS Apple Silicon / Linux)
3. Следовать инструкции по установке

**Docker Desktop включает:**
- 🔧 **Docker Engine** — основной движок виртуализации
- 💻 **Docker CLI** — клиент командной строки
- 🖥️ **Docker UI** — графический интерфейс

### Проверка установки:

```bash
# Проверить версию Docker
docker --version

# Должно вывести что-то вроде:
# Docker version 24.0.5, build ced0996
```

---

## 5. Docker Images vs Containers

### Docker Image (Образ)

**Образ** — это упакованный артефакт приложения, который содержит:
- Скомпилированный код приложения
- Слой приложений ОС (Linux)
- Установленные инструменты (Node.js, JRE, Python и т.д.)
- Переменные окружения
- Конфигурационные файлы и директории

> Аналогия: образ — это как ISO-файл операционной системы или JAR-файл Java-приложения, но мощнее.

### Docker Container (Контейнер)

**Контейнер** — это **запущенный экземпляр образа**.

```
Docker Image  ──► docker run ──►  Container (работающий процесс)
                                  Container (ещё один экземпляр)
                                  Container (ещё один экземпляр)
```

Из одного образа можно запустить **несколько контейнеров** одновременно — например, для масштабирования под нагрузку.

### Основные команды для работы с образами:

```bash
# Посмотреть список локальных образов
docker images

# Посмотреть запущенные контейнеры
docker ps

# Посмотреть все контейнеры (включая остановленные)
docker ps -a
```

---

## 6. Docker Registries (Реестры)

**Registry (Реестр)** — это хранилище Docker-образов. Откуда брать готовые образы сервисов?

### Docker Hub

**Docker Hub** — самый большой публичный реестр Docker-образов:
- 🔗 [https://hub.docker.com](https://hub.docker.com)
- Содержит **официальные образы** от создателей технологий
- Доступен без регистрации для скачивания образов

**Официальные образы** проверяются:
- Командой Docker
- Создателями технологий
- Экспертами по безопасности

### Пример поиска образа:

На Docker Hub можно найти образы для любого популярного сервиса:
- `redis` — кеш-сервер
- `postgres` — база данных PostgreSQL
- `nginx` — веб-сервер
- `mongo` — MongoDB
- `rabbitmq` — брокер сообщений

---

## 7. Версии Docker-образов (Image Tags)

Технологии обновляются — для каждой версии создаётся **отдельный тег образа**.

```bash
# Теги — это версии образа
postgres:14
postgres:15
postgres:15.3
postgres:latest    # последняя доступная версия
```

> ⚠️ **Лучшая практика:** всегда указывать конкретную версию (тег) вместо `latest`, чтобы гарантировать воспроизводимость.

```bash
# Плохо — непредсказуемо:
docker pull postgres

# Хорошо — предсказуемо:
docker pull postgres:15.3
```

---

## 8. Основные команды: pull и run

### docker pull — скачать образ

```bash
# Скачать конкретную версию
docker pull nginx:1.23

# Скачать последнюю версию (latest)
docker pull nginx
```

### docker run — запустить контейнер

```bash
# Запустить контейнер из образа
docker run nginx:1.23

# Запустить в фоновом режиме (detached mode)
docker run -d nginx:1.23

# docker run автоматически скачает образ, если его нет локально
# (не нужно делать pull заранее!)
docker run nginx:1.22-alpine
```

### docker logs — просмотр логов контейнера

```bash
# Посмотреть логи по ID контейнера
docker logs <container_id>

# Пример:
docker logs a1b2c3d4e5f6
```

### Полный пример рабочего процесса:

```bash
# 1. Скачиваем образ nginx версии 1.23
docker pull nginx:1.23

# 2. Смотрим доступные образы
docker images
# REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
# nginx        1.23      ...            ...            135MB

# 3. Запускаем контейнер в фоне
docker run -d nginx:1.23

# 4. Проверяем запущенные контейнеры
docker ps
# CONTAINER ID   IMAGE        COMMAND                CREATED         STATUS         PORTS     NAMES
# a1b2c3d4e5f6   nginx:1.23   "/docker-entrypoint…"  5 seconds ago   Up 4 seconds   80/tcp    funny_hopper
```

---

## 9. Port Binding (Привязка портов)

Контейнер работает в **изолированной Docker-сети** и по умолчанию недоступен извне. Чтобы получить доступ к приложению внутри контейнера с локальной машины, нужно **привязать порт контейнера к порту хоста**.

### Стандартные порты приложений:

| Сервис | Порт |
|--------|------|
| nginx | 80 |
| PostgreSQL | 5432 |
| MySQL | 3306 |
| Redis | 6379 |
| MongoDB | 27017 |
| Node.js (пример) | 3000 |

### Синтаксис флага `-p`:

```bash
docker run -d -p <HOST_PORT>:<CONTAINER_PORT> <image>

# Примеры:
docker run -d -p 9000:80 nginx:1.23        # localhost:9000 → контейнер:80
docker run -d -p 3306:3306 mysql:8.0       # localhost:3306 → контейнер:3306
docker run -d -p 6379:6379 redis:7         # localhost:6379 → контейнер:6379
```

```
Браузер/Клиент ──► localhost:9000 ──► Docker ──► контейнер:80 (nginx)
```

### Пример с nginx:

```bash
# Запускаем nginx, привязываем порт 9000 хоста к порту 80 контейнера
docker run -d -p 9000:80 nginx:1.23

# Проверяем
docker ps
# PORTS: 0.0.0.0:9000->80/tcp

# Теперь открываем в браузере: http://localhost:9000
# Увидим приветственную страницу nginx ✅
```

---

## 10. Запуск и остановка контейнеров

### docker stop / docker start

```bash
# Остановить контейнер (по ID или имени)
docker stop <container_id>
docker stop <container_name>

# Запустить остановленный контейнер повторно
docker start <container_id>
docker start <container_name>

# Остановить несколько контейнеров сразу
docker stop container1 container2
```

> ⚠️ **Важно!** `docker run` — создаёт **новый** контейнер каждый раз.  
> `docker start` — запускает уже **существующий** (остановленный) контейнер.

### Именование контейнеров

Docker автоматически генерирует случайные имена (`funny_hopper`, `clever_darwin`). Можно задать **своё имя** с флагом `--name`:

```bash
# Создать контейнер с именем "web-app"
docker run -d -p 9000:80 --name web-app nginx:1.23

# Теперь можно использовать имя вместо ID
docker stop web-app
docker start web-app
docker logs web-app
```

### Полный список команд для управления контейнерами:

```bash
# Список запущенных контейнеров
docker ps

# Список всех контейнеров (включая остановленные)
docker ps -a

# Остановить контейнер
docker stop web-app

# Запустить контейнер
docker start web-app

# Просмотр логов
docker logs web-app

# Удалить контейнер
docker rm web-app

# Удалить образ
docker rmi nginx:1.23
```

---

## 11. Приватные Docker-реестры

Когда компания создаёт образы своих приложений — они **не должны быть публичными**.

### Провайдеры приватных реестров:

| Провайдер | Сервис |
|-----------|--------|
| AWS | ECR (Elastic Container Registry) |
| Google Cloud | Artifact Registry |
| Microsoft Azure | Azure Container Registry |
| JetBrains / Self-hosted | Nexus Repository |
| Docker Inc. | Docker Hub (приватные репозитории) |

### Аутентификация в приватном реестре:

```bash
# Логин в Docker Hub
docker login

# Логин в AWS ECR (пример)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com
```

---

## 12. Registry vs Repository

Частая путаница — разница между **registry** и **repository**:

```
Registry (реестр) — это сервис/платформа для хранения образов
│
├── Repository (репозиторий) — хранилище образов одного приложения
│   ├── my-app:1.0
│   ├── my-app:1.1
│   └── my-app:2.0
│
├── Repository — другое приложение
│   ├── my-backend:latest
│   └── my-backend:3.5
│
└── Repository — ещё одно
    └── ...
```

| Понятие | Аналогия | Пример |
|---------|----------|--------|
| Registry | GitHub как платформа | Docker Hub, AWS ECR |
| Repository | Один репозиторий на GitHub | `mycompany/my-app` |
| Image Tag | Ветка или тег в git | `my-app:1.0`, `my-app:latest` |

---

## 13. Dockerfile — Dockerize Node.js приложения

**Dockerfile** — это текстовый файл с инструкциями по созданию Docker-образа.

### Структура примера приложения:

```
my-app/
├── Dockerfile
├── package.json
└── src/
    └── server.js
```

**server.js:**
```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
    res.send('Welcome to my awesome app!');
});

app.listen(PORT, () => {
    console.log(`App running on port ${PORT}`);
});
```

**package.json:**
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

### Dockerfile с подробными комментариями:

```dockerfile
# FROM — базовый образ, на основе которого строится наш образ
# node:19-alpine — Node.js 19 на базе лёгкого Alpine Linux
FROM node:19-alpine

# COPY — копируем файлы с хоста в контейнер
# package.json → /app/package.json
COPY package.json /app/

# COPY — копируем папку src в контейнер
# src/ → /app/src/
COPY src /app/src/

# WORKDIR — устанавливаем рабочую директорию
# Все следующие команды будут выполняться в /app
WORKDIR /app

# RUN — выполняем команду при сборке образа
# Устанавливаем зависимости из package.json
RUN npm install

# CMD — команда запуска приложения (выполняется при старте контейнера)
# Запускаем node с файлом server.js
CMD ["node", "src/server.js"]
```

### Ключевые директивы Dockerfile:

| Директива | Назначение | Пример |
|-----------|------------|--------|
| `FROM` | Базовый образ | `FROM node:19-alpine` |
| `COPY` | Копировать файлы с хоста в контейнер | `COPY . /app/` |
| `WORKDIR` | Установить рабочую директорию | `WORKDIR /app` |
| `RUN` | Выполнить команду при **сборке** | `RUN npm install` |
| `CMD` | Команда запуска при **старте контейнера** | `CMD ["node", "server.js"]` |
| `ENV` | Задать переменную окружения | `ENV PORT=3000` |
| `EXPOSE` | Документировать порт | `EXPOSE 3000` |

### Разница RUN vs CMD:

```dockerfile
# RUN — выполняется во время BUILD (сборки образа), результат сохраняется в слой
RUN npm install
RUN apt-get install -y curl

# CMD — выполняется при ЗАПУСКЕ контейнера, не при сборке
CMD ["node", "src/server.js"]
```

### Пример Dockerfile для Java-приложения:

```dockerfile
FROM eclipse-temurin:17-alpine
COPY target/my-app.jar /app/my-app.jar
WORKDIR /app
CMD ["java", "-jar", "my-app.jar"]
```

### Пример Dockerfile для Python-приложения:

```dockerfile
FROM python:3.11-slim
COPY requirements.txt /app/
WORKDIR /app
RUN pip install -r requirements.txt
COPY . /app/
CMD ["python", "app.py"]
```

---

## 14. Сборка образа (Build Image)

После написания Dockerfile — собираем образ командой `docker build`.

### Синтаксис:

```bash
docker build -t <имя-образа>:<тег> <путь-к-Dockerfile>
```

### Пример:

```bash
# Собрать образ с именем "node-app" и тегом "1.0"
# Точка (.) — текущая директория, где находится Dockerfile
docker build -t node-app:1.0 .

# Вывод в терминале:
# Step 1/6 : FROM node:19-alpine
# Step 2/6 : COPY package.json /app/
# Step 3/6 : COPY src /app/src/
# Step 4/6 : WORKDIR /app
# Step 5/6 : RUN npm install
# Step 6/6 : CMD ["node", "src/server.js"]
# Successfully built a1b2c3d4e5f6
# Successfully tagged node-app:1.0
```

### Запуск собранного образа:

```bash
# Проверяем наличие образа
docker images
# REPOSITORY   TAG   IMAGE ID       SIZE
# node-app     1.0   a1b2c3d4e5f6   180MB
# nginx        1.23  ...            135MB

# Запускаем контейнер из нашего образа
docker run -d -p 3000:3000 node-app:1.0

# Проверяем
docker ps

# Открываем в браузере: http://localhost:3000
# Увидим: "Welcome to my awesome app!" ✅
```

### Публикация образа в Docker Hub:

```bash
# Логинимся
docker login

# Переименовываем образ в формат dockerhub-username/repo-name:tag
docker tag node-app:1.0 myusername/node-app:1.0

# Публикуем
docker push myusername/node-app:1.0
```

---

## 15. Docker UI Client

**Docker Desktop** включает графический интерфейс (GUI):
- Список контейнеров (запущенные / остановленные)
- Список образов
- Управление контейнерами (запуск, остановка, удаление, просмотр логов)
- Создание контейнеров через UI

> 💡 GUI удобен для визуального контроля, но большинство разработчиков предпочитают CLI — он быстрее и лучше поддаётся автоматизации.

---

## 16. Docker в полном цикле разработки

### Упрощённый пример workflow:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Developer (локально)                         │
│                                                                  │
│  JavaScript App  ←──→  MongoDB (docker run mongo)               │
└──────────────┬──────────────────────────────────────────────────┘
               │  git commit / git push
               ▼
┌─────────────────────────────────────────────────────────────────┐
│                  CI/CD Pipeline (Jenkins / GitHub Actions)       │
│                                                                  │
│  1. Build JavaScript app                                         │
│  2. docker build -t mycompany/js-app:1.0 .                      │
│  3. docker push mycompany/js-app:1.0  ──► Private Registry      │
└──────────────────────────────────┬──────────────────────────────┘
                                   │  deploy
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Dev/Prod Server                             │
│                                                                  │
│  docker pull mycompany/js-app:1.0  ◄── Private Registry         │
│  docker run mycompany/js-app:1.0                                 │
│                                                                  │
│  docker run mongo  ◄── Docker Hub (публичный)                   │
│                                                                  │
│  [js-app container] ←──→ [mongo container]                       │
└─────────────────────────────────────────────────────────────────┘
```

### Роли в процессе:

| Этап | Кто участвует | Что делает с Docker |
|------|---------------|---------------------|
| Локальная разработка | Разработчик | `docker run` для запуска зависимостей |
| CI/CD сборка | Jenkins / GitHub Actions | `docker build` + `docker push` |
| Деплой | Ops / DevOps | `docker pull` + `docker run` на сервере |

---

## Шпаргалка по командам

```bash
# ── Образы ──────────────────────────────────────────────────────
docker images                        # список локальных образов
docker pull nginx:1.23               # скачать образ
docker rmi nginx:1.23                # удалить образ
docker build -t my-app:1.0 .         # собрать образ из Dockerfile

# ── Контейнеры ──────────────────────────────────────────────────
docker run nginx:1.23                # запустить контейнер
docker run -d nginx:1.23             # запустить в фоне (detached)
docker run -d -p 8080:80 nginx:1.23  # с привязкой порта
docker run -d --name web nginx:1.23  # с именем контейнера
docker ps                            # запущенные контейнеры
docker ps -a                         # все контейнеры
docker stop <id/name>                # остановить контейнер
docker start <id/name>               # запустить снова
docker rm <id/name>                  # удалить контейнер

# ── Логи и информация ────────────────────────────────────────────
docker logs <id/name>                # логи контейнера
docker inspect <id/name>             # детальная информация

# ── Реестр ──────────────────────────────────────────────────────
docker login                         # войти в Docker Hub
docker push myuser/my-app:1.0        # опубликовать образ
docker tag my-app:1.0 myuser/my-app:1.0  # переименовать/тегировать
```

---

## Полезные ресурсы

- 📖 [Официальная документация Docker](https://docs.docker.com/)
- 🐳 [Docker Hub](https://hub.docker.com/)
- 📚 Docker Compose — для оркестрации нескольких контейнеров
- 📚 Docker Volumes — для персистентного хранения данных
- 📚 Kubernetes — для оркестрации контейнеров в продакшене

