# Real-time Chat / Мессенджер в реальном времени (Go)

[English](#english-version) | [Русская версия](#русская-версия)

---

## English version

### Overview

Real-time Chat / Messaging Platform is a production-style backend project written in Go, focused on real-time messaging.

Users can:

- register and authenticate;
- create and manage chat rooms (channels);
- connect to rooms via WebSocket;
- send and receive messages in real time.

The main goal is to practice a **full backend stack**:

- Go (REST + WebSocket, concurrency, networking);
- PostgreSQL as the primary datastore;
- Redis as a cache and pub/sub mechanism;
- Docker & Kubernetes for containerization and orchestration;
- Prometheus & Grafana for metrics and monitoring;
- CI/CD (GitLab CI or GitHub Actions) for automated build & deploy.

The project is designed as a **pet project for learning and interviews**: you can demonstrate it in your portfolio and discuss design decisions in detail.

---

### Features

- User registration and login (JWT-based auth).
- CRUD operations for chat rooms.
- WebSocket connections per room.
- Real-time message broadcasting within a room.
- Message history persisted in PostgreSQL.
- Redis for:
  - caching recent messages / presence;
  - pub/sub between multiple instances of the chat service.
- Basic metrics (via Prometheus):
  - active WebSocket connections;
  - REST API RPS;
  - number of sent/received messages;
  - error rates and latency.
- Ready to be deployed into a Kubernetes cluster.

---

### High-level Architecture

**Components:**

- **API / Gateway (Go)**  
  Handles HTTP REST requests (auth, rooms management) and WebSocket connections (chat).  
  Responsible for:
  - authentication & authorization (JWT);
  - REST API for users/rooms;
  - WebSocket upgrade and message frames handling.

- **Chat Hub (Go)**  
  Internal module that:
  - manages rooms and room members;
  - tracks connected clients;
  - broadcasts messages to room participants;
  - can be extended with direct messages and presence.

- **PostgreSQL**  
  Main persistent storage:
  - `users` — accounts and credentials;
  - `rooms` — chat rooms (channels);
  - `messages` — message history (text, sender, timestamps).

- **Redis**  
  Used for:
  - caching frequently accessed data (e.g. last messages, online status);
  - pub/sub across multiple instances of the chat service (so that messages are delivered regardless of which pod a user is connected to).

- **Prometheus & Grafana**  
  - `/metrics` endpoint exported by the Go service (Prometheus client);
  - Prometheus scrapes metrics;
  - Grafana visualizes dashboards (RPS, WS connections, errors, latency, etc.).

- **Kubernetes**  
  Base deployment model:
  - `Deployment` for the Go API/Chat service;
  - `Service` (ClusterIP/LoadBalancer) for internal/external access;
  - `ConfigMap`/`Secret` for configuration (DB connection string, JWT secret, etc.);
  - optional `Ingress` + HPA for routing and autoscaling.

---

### Tech Stack

**Language & Libraries**

- Go 1.21+ / 1.22
- HTTP router: `github.com/go-chi/chi/v5`
- WebSocket: `github.com/gorilla/websocket`
- JWT: `github.com/golang-jwt/jwt/v5` (or similar)
- Configuration: environment variables (with optional helper library)

**Data & Cache**

- PostgreSQL
- Redis (`github.com/redis/go-redis/v9`)
- DB migrations: `golang-migrate` or any other migration tool

**Infrastructure**

- Docker, docker-compose
- Kubernetes (minikube, kind, k3s or any managed K8s)
- (Optional) Helm charts

**Observability**

- Prometheus (`github.com/prometheus/client_golang`)
- Grafana
- Logs written to stdout (integratable with Loki/Elastic stack)

**CI/CD**

- GitLab CI (`.gitlab-ci.yml`) or
- GitHub Actions (`.github/workflows/ci.yml`)

---

### Project Structure (suggested)

```text
realtime-chat/
  cmd/
    api/
      main.go              # entry point for HTTP + WebSocket server
  internal/
    http/
      router.go            # chi router and middlewares
      handlers.go          # REST endpoints (auth, rooms, etc.)
      ws_handler.go        # WebSocket upgrade and message handling
    chat/
      hub.go               # central hub for rooms and clients
      room.go              # room logic (broadcast, join/leave)
      client.go            # client structure (connection, send/recv)
    storage/
      postgres/
        repo.go            # Postgres repository (users, rooms, messages)
      redis/
        cache.go           # cache and pub/sub logic
    auth/
      jwt.go               # JWT generation and validation
      middleware.go        # HTTP auth middleware
    config/
      config.go            # configuration loader (env, files)
    metrics/
      metrics.go           # Prometheus metrics and /metrics handler
    logger/
      logger.go            # logging wrapper (zap/logrus/slog)
  deployments/
    docker/
      Dockerfile
      docker-compose.yml
    k8s/
      configmap.yaml
      deployment.yaml
      service.yaml
      ingress.yaml
  .gitlab-ci.yml
  go.mod
  go.sum
  README.md
Quick Start (local via docker-compose)
Commands are illustrative; adjust to your implementation.

Clone the repository:

bash
Copy code
git clone https://github.com/<your-name>/realtime-chat.git
cd realtime-chat
Build and start services:

bash
Copy code
docker-compose up --build
This should start:

Go API / Chat service;

PostgreSQL;

Redis;

(optionally) Prometheus;

(optionally) Grafana.

Verify the service:

Healthcheck:
GET http://localhost:8080/healthz

Example API:
GET http://localhost:8080/api/v1/rooms

Example WebSocket route:
ws://localhost:8080/ws/rooms/{roomID}

Open Grafana (if enabled):

http://localhost:3000
Default credentials: admin/admin (change in production).

Example API (conceptual)
Routes are conceptual; the actual implementation may differ.

Registration
POST /api/v1/auth/register

json
Copy code
{
  "email": "user@example.com",
  "password": "secret123"
}
Login
POST /api/v1/auth/login

json
Copy code
{
  "email": "user@example.com",
  "password": "secret123"
}
Response: JWT access/refresh tokens.

Create Room
POST /api/v1/rooms

json
Copy code
{
  "name": "general"
}
Connect to Room via WebSocket
GET /ws/rooms/{roomID}

Header: Authorization: Bearer <access_token>.

Example message payload:

json
Copy code
{
  "type": "message",
  "payload": {
    "text": "Hello, world!"
  }
}
Metrics & Monitoring
The service exposes metrics at:

GET /metrics

Prometheus scrapes this endpoint and Grafana visualizes dashboards, e.g.:

requests per second (RPS) per endpoint;

number of active WebSocket connections;

messages per second;

error rates and latency distributions.

Roadmap
Planned or possible improvements:

Direct messages (user-to-user).

Room permissions (public/private/invite-only).

Typing indicators and presence (online/offline).

Splitting the system into multiple microservices (auth, chat, notifications).

Introducing Kafka / RabbitMQ for event-driven workflows.

Adding distributed tracing (OpenTelemetry + Jaeger).

Building a simple frontend client (React/Vue/Svelte) as a reference implementation.

Русская версия
Обзор
Real-time Chat / Мессенджер в реальном времени — это backend-проект на Go, архитектурно близкий к production и предназначенный для отработки навыков разработки real-time сервисов.

Пользователи могут:

регистрироваться и проходить авторизацию;

создавать и управлять чат-комнатами (каналами);

подключаться к комнатам по WebSocket;

отправлять и получать сообщения в реальном времени.

Цель проекта — отработать полноценный backend-стек:

Go (REST + WebSocket, конкурентность, работа с сетью);

PostgreSQL как основное хранилище данных;

Redis как кэш и механизм pub/sub;

Docker и Kubernetes для контейнеризации и оркестрации;

Prometheus и Grafana для метрик и мониторинга;

CI/CD (GitLab CI или GitHub Actions) для автоматизации сборки и деплоя.

Проект задуман как pet-проект для портфолио и собеседований: его можно показывать как пример продуманной архитектуры и обсуждать технические решения.

Основные возможности
Регистрация и авторизация пользователей (JWT).

CRUD-операции с чат-комнатами.

Подключение к комнатам по WebSocket.

Обмен сообщениями в реальном времени (broadcast внутри комнаты).

Сохранение истории сообщений в PostgreSQL.

Redis:

кэширование последних сообщений / статуса присутствия;

pub/sub для работы нескольких инстансов сервиса в Kubernetes.

Базовые метрики (через Prometheus):

количество активных WebSocket-подключений;

RPS по REST-эндпоинтам;

число отправленных/полученных сообщений;

частота ошибок и задержки (latency).

Готовность к развёртыванию в Kubernetes.

Архитектура (в общих чертах)
Компоненты:

API / Gateway (Go)
Обрабатывает HTTP REST-запросы (регистрация, логин, управление комнатами) и WebSocket-подключения (чат).
Отвечает за:

аутентификацию и авторизацию (JWT);

REST API для пользователей и комнат;

upgrade до WebSocket и обработку сообщений.

Chat Hub (Go)
Внутренний модуль, который:

управляет комнатами и их участниками;

ведёт список подключённых клиентов;

рассылает сообщения всем участникам комнаты;

может быть расширен до личных сообщений и presence.

PostgreSQL
Основное хранилище:

users — пользователи и их данные;

rooms — чат-комнаты;

messages — история сообщений (текст, отправитель, время).

Redis
Используется для:

кэширования часто используемых данных (последние сообщения, статусы);

pub/sub между несколькими репликами сервиса в Kubernetes (чтобы сообщения доходили независимо от конкретного pod).

Prometheus и Grafana

сервис экспортирует /metrics (через Prometheus-клиент);

Prometheus собирает метрики;

Grafana показывает дашборды (RPS, активные подключения, ошибки, задержки и т.п.).

Kubernetes
Основная модель развёртывания:

Deployment для Go-сервиса;

Service (ClusterIP/LoadBalancer) для доступа;

ConfigMap/Secret для конфигурации (строка подключения к БД, JWT секрет и т.п.);

опционально Ingress + HPA для маршрутизации и автоскейлинга.

Стек технологий
Язык и библиотеки

Go 1.21+ / 1.22

HTTP роутер: github.com/go-chi/chi/v5

WebSocket: github.com/gorilla/websocket

JWT: github.com/golang-jwt/jwt/v5 (или аналог)

Конфигурация: переменные окружения (при желании — отдельная библиотека)

Данные и кэш

PostgreSQL

Redis (github.com/redis/go-redis/v9)

Миграции БД: golang-migrate или любая другая утилита

Инфраструктура

Docker, docker-compose

Kubernetes (minikube, kind, k3s или managed-кластер)

(Опционально) Helm charts

Observability

Prometheus (github.com/prometheus/client_golang)

Grafana

Логи в stdout (можно интегрировать с Loki/Elastic стэком)

CI/CD

GitLab CI (.gitlab-ci.yml) или

GitHub Actions (.github/workflows/ci.yml)

Структура проекта (рекомендуемая)
text
Copy code
realtime-chat/
  cmd/
    api/
      main.go              # входная точка HTTP + WebSocket сервера
  internal/
    http/
      router.go            # chi router, middlewares
      handlers.go          # REST-эндпоинты (auth, rooms, etc.)
      ws_handler.go        # upgrade до WebSocket, обработка сообщений
    chat/
      hub.go               # центральный "hub" для комнат и клиентов
      room.go              # логика комнаты (broadcast, join/leave)
      client.go            # структура клиента (connection, send/recv)
    storage/
      postgres/
        repo.go            # работа с Postgres (users, rooms, messages)
      redis/
        cache.go           # кэш и pub/sub
    auth/
      jwt.go               # генерация/валидация JWT
      middleware.go        # HTTP auth middleware
    config/
      config.go            # чтение конфигурации (env/файлы)
    metrics/
      metrics.go           # Prometheus метрики и /metrics handler
    logger/
      logger.go            # логгер (zap/logrus/slog)
  deployments/
    docker/
      Dockerfile
      docker-compose.yml
    k8s/
      configmap.yaml
      deployment.yaml
      service.yaml
      ingress.yaml
  .gitlab-ci.yml
  go.mod
  go.sum
  README.md
Быстрый старт (локально через docker-compose)
Команды примерные, подстрой под свою реализацию.

Клонируй репозиторий:

bash
Copy code
git clone https://github.com/<your-name>/realtime-chat.git
cd realtime-chat
Собери и подними сервисы:

bash
Copy code
docker-compose up --build
Обычно поднимаются:

Go API / чат-сервис;

PostgreSQL;

Redis;

(опционально) Prometheus;

(опционально) Grafana.

Проверь доступность сервиса:

Healthcheck:
GET http://localhost:8080/healthz

Пример API:
GET http://localhost:8080/api/v1/rooms

Пример WebSocket-маршрута:
ws://localhost:8080/ws/rooms/{roomID}

Grafana (если включена):

http://localhost:3000
Логин/пароль по умолчанию: admin/admin (обязательно поменять).

Примеры API (концептуально)
Маршруты примерные, реализация может отличаться.

Регистрация
POST /api/v1/auth/register

json
Copy code
{
  "email": "user@example.com",
  "password": "secret123"
}
Логин
POST /api/v1/auth/login

json
Copy code
{
  "email": "user@example.com",
  "password": "secret123"
}
Ответ: JWT access/refresh токены.

Создание комнаты
POST /api/v1/rooms

json
Copy code
{
  "name": "general"
}
Подключение к комнате по WebSocket
GET /ws/rooms/{roomID}

Заголовок: Authorization: Bearer <access_token>.

Пример формата сообщения:

json
Copy code
{
  "type": "message",
  "payload": {
    "text": "Hello, world!"
  }
}
Метрики и мониторинг
Сервис отдаёт метрики по маршруту:

GET /metrics

Prometheus настраивается на scrape этого эндпоинта, а Grafana строит дашборды, например:

RPS по эндпоинтам;

количество активных WebSocket-подключений;

сообщений в секунду;

частота ошибок и распределение задержек.

Планы развития
Потенциальные улучшения:

Личные сообщения (direct messages).

Права доступа к комнатам (public/private/invite-only).

Индикатор набора текста и presence (online/offline).

Разделение на несколько микросервисов (auth, chat, notifications).

Kafka / RabbitMQ для event-driven архитектуры.

Distributed tracing (OpenTelemetry + Jaeger).

Простой фронтенд-клиент (React/Vue/Svelte) как пример клиента.
