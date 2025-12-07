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
