# 🚀 Deployment

> **Source Projects:** Gaming (multi-cloud, serverless), Sentinel (Docker Compose, render.yaml, nginx), DevTrace (embedded dashboard)
>
> Deployment is the final gate. A system that works locally but fails in production has not been built — it has been prototyped.

---

## 1️⃣ Deployment Tiers

### Environment Progression

```
Development → Staging → Production
```

| Environment | Purpose | Data | Access |
| :--- | :--- | :--- | :--- |
| **Development** | Local development, debugging | Synthetic or seeded | Developer machines |
| **Staging** | Pre-production validation | Anonymized production copy | QA, stakeholders |
| **Production** | Live user traffic | Real user data | Public / authorized users |

> [!IMPORTANT]
> **Never deploy directly to production.** Staging must be an exact replica of production (same container images, same config schema, same database engine).

---

## 2️⃣ Pattern A: Multi-Cloud Serverless (Gaming)

### Architecture

```mermaid
flowchart TD
    subgraph Vercel["Vercel Edge (Frontend)"]
        NextJS["Next.js App"]
        Middleware["Edge Middleware"]
        API["API Routes"]
    end
    
    subgraph GitHub["GitHub Actions (Worker)"]
        Cron["Cron Schedule"]
        GoWorker["Go Binary"]
    end
    
    subgraph DataCloud["Data Cloud"]
        AstraDB["Astra DB (Cassandra)"]
        Redis["Upstash Redis"]
        QStash["Upstash QStash"]
    end
    
    Middleware -->|rate limit| Redis
    Cron -->|every 5min| GoWorker
    GoWorker -->|ingest| AstraDB
    GoWorker -->|webhook| QStash
    QStash -->|trigger| API
    
    style Vercel fill:#ff6b6b,stroke:#333,stroke-width:2px,color:#fff
    style GitHub fill:#95e1d3,stroke:#333,stroke-width:2px,color:#333
    style DataCloud fill:#ffe66d,stroke:#333,stroke-width:2px,color:#333
```

### Key Principles

| Principle | Implementation |
| :--- | :--- |
| **Stateless Frontend** | Next.js on Vercel Edge, no local disk writes |
| **Background Workers as CI** | GitHub Actions cron jobs replace always-on servers |
| **Edge Rate Limiting** | Upstash Redis in Vercel middleware, blocks traffic before it reaches the app |
| **Cryptographic Webhooks** | QStash signature verification prevents spoofed triggers |

### Deployment Checklist

- [ ] Environment variables configured in Vercel Dashboard (not `.env`)
- [ ] GitHub Secrets configured for worker tokens and database credentials
- [ ] `NEXT_PUBLIC_` prefix only for public keys (never secret keys)
- [ ] Health check endpoint returns `200 OK` with version hash
- [ ] Rollback plan documented (Vercel: one-click revert; Worker: revert git tag)

---

## 3️⃣ Pattern B: Polyglot Container Mesh (Sentinel)

### Architecture

```mermaid
flowchart TD
    subgraph Frontend["Frontend Tier"]
        NextJS["Next.js + WebGL"]
    end
    
    subgraph APIGateway["API Gateway"]
        Nginx["Nginx Reverse Proxy"]
        GoGateway["Go Gateway Service"]
    end
    
    subgraph Services["Service Tier"]
        GoAPI["Go API Services"]
        ScalaStream["Scala Streaming (ZIO)"]
        PythonML["Python ML / RAG"]
    end
    
    subgraph Data["Data Tier"]
        Postgres["PostgreSQL"]
        ClickHouse["ClickHouse"]
        Redis["Redis"]
        Neo4j["Neo4j"]
        Qdrant["Qdrant"]
    end
    
    NextJS -->|HTTPS| Nginx
    Nginx -->|route| GoGateway
    GoGateway -->|REST| GoAPI
    GoGateway -->|gRPC| ScalaStream
    GoGateway -->|gRPC| PythonML
    
    GoAPI --> Postgres
    ScalaStream --> ClickHouse
    ScalaStream --> Kafka
    PythonML --> Qdrant
    PythonML --> Neo4j
    
    style Frontend fill:#ff6b6b,stroke:#333,stroke-width:2px,color:#fff
    style APIGateway fill:#95e1d3,stroke:#333,stroke-width:2px,color:#333
    style Services fill:#ffe66d,stroke:#333,stroke-width:2px,color:#333
    style Data fill:#dda0dd,stroke:#333,stroke-width:2px,color:#333
```

### Docker Compose Strategy

```yaml
# docker-compose.yml
services:
  gateway:
    build: ./gateway
    ports: ["8080:8080"]
    depends_on: [postgres, redis]
    
  ml-optimizer:
    build: ./ml-optimizer
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    environment:
      - CUDA_VISIBLE_DEVICES=0
    
  scala-streaming:
    build: ./scala-streaming
    depends_on: [kafka, clickhouse]
```

### Service Communication

| Protocol | When to Use | Example |
| :--- | :--- | :--- |
| **REST/HTTP** | Public APIs, simple CRUD | Go gateway → Frontend |
| **gRPC** | Internal service-to-service, typed contracts | Go → Scala streaming |
| **Message Broker** | Async, decoupled, event-driven | Scala → ClickHouse sink |
| **WebSocket** | Real-time bidirectional | Dashboard → Gateway |

> [!TIP]
> **Strangler Fig Migration:** When migrating from monolith to mesh, route traffic through a gateway that progressively moves endpoints to new services. The old system handles fallback during migration.

---

## 4️⃣ Pattern C: Embedded Deployment (DevTrace)

### Concept

> [!TIP]
> Ship the dashboard as a static asset embedded in the binary. This eliminates a separate deployment pipeline for the UI.

### Implementation

```rust
// Rust + rust-embed
use rust_embed::Embed;

#[derive(Embed)]
#[folder = "ui/dist/"]
struct UiAssets;

// Serve embedded dashboard
app.route("/", get(|_| async move {
    let index = UiAssets::get("index.html").unwrap();
    Response::builder()
        .header("Content-Type", "text/html")
        .body(Body::from(index.data))
}))
```

### Benefits

- Single binary contains both proxy and dashboard
- No CDN or separate frontend deployment
- Version-locked: dashboard always matches the binary version

---

## 5️⃣ Configuration Management

### Environment Variables

```text
# .env.example
DATABASE_URL=postgres://user:pass@localhost:5432/db
REDIS_URL=redis://localhost:6379/0
API_KEY=sk-...
NODE_ENV=production
PORT=3000
LOG_LEVEL=info
```

> [!WARNING]
> **Never commit `.env` to version control.** Always provide `.env.example` with placeholder values. Use secret managers (Vault, Doppler, AWS Secrets Manager) in production.

### Configuration Validation

```typescript
// config.ts
const required = ["DATABASE_URL", "REDIS_URL", "API_KEY"];
for (const key of required) {
  if (!process.env[key]) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
}
```

```python
# settings.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    redis_url: str
    api_key: str
    
    model_config = SettingsConfigDict(env_file=".env")

settings = Settings()  # Fails fast on missing variables
```

---

## 6️⃣ Health Checks & Readiness

### Liveness Probe

> Is the process running?

```bash
GET /health/live
```

Returns `200 OK` if the process is alive.

### Readiness Probe

> Is the service ready to accept traffic?

```bash
GET /health/ready
```

Returns `200 OK` only if:
- Database connection is healthy
- Cache is reachable
- Dependencies are responding

### Startup Probe

> Has the application finished initializing?

```bash
GET /health/startup
```

Returns `200 OK` when migrations have run, caches are warm, and the server is listening.

> [!IMPORTANT]
> Distinguish between liveness, readiness, and startup probes. A service that is alive but not ready should not receive traffic. A service that is starting up should not be killed by premature liveness checks.

---

## 7️⃣ Rollback Strategy

### Immediate Rollback

| Platform | Method |
| :--- | :--- |
| **Vercel** | Dashboard: Deployments → "Promote" previous deployment |
| **GitHub Actions** | `git revert <commit>` → push to trigger new deployment |
| **Docker / K8s** | `kubectl rollout undo deployment/<name>` |
| **Render** | Dashboard: Manual Deploy → select previous successful deploy |

### Rollback Principles

1. **Tag bad deployments.** `git tag bad-deploy-2026-08-12` creates a reference point.
2. **Revert, don't reset.** `git revert -m 1 <merge-hash>` preserves history.
3. **Postmortem first.** Document the failure before reverting.
4. **Verify after rollback.** Confirm error rates drop and metrics normalize.

---

## 8️⃣ Reference Implementations

| Repo | Pattern | Stack |
| :--- | :--- | :--- |
| **Gaming** | Multi-Cloud Serverless | Vercel, GitHub Actions, Astra DB, Upstash |
| **Sentinel** | Polyglot Container Mesh | Docker Compose, Nginx, Scala ZIO, Go, Python |
| **DevTrace** | Embedded Deployment | Rust binary with rust-embed dashboard |
| **DocsSense** | Full-Stack Platform | Vercel, Railway, Hugging Face Spaces, Docker |
