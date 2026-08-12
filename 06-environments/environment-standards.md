# 🌍 Environment Standards

> **Configuration is an interface, not an afterthought.** Environment variables are the contract between your code and the infrastructure that runs it. If a new engineer cannot start the project by copying one file and running one command, the environment setup has already failed.

This document defines the `.env` conventions, secret management rules, and environment promotion discipline extracted from actual production repositories (`Sentinel`, `Milan`, `DevTrace`, `ExpressKit`, `Gaming`).

---

## 🧭 The Core Philosophy

Before applying the rules, understand the constraints that govern them:

> [!IMPORTANT]
> 1. **Reproducibility First:** Any engineer must be able to clone the repo, copy `.env.example`, and run the project without asking for secrets.
> 2. **Secrets Are Never in Git:** `.env` files, API keys, and credentials are infrastructure concerns, not code concerns.
> 3. **Validation at Startup:** The application must fail fast with a clear message if a required variable is missing or malformed.

---

## 1️⃣ `.env.example` Standards

_Reference Repos: Sentinel, Milan, DevTrace, ExpressKit, Gaming_

Every repository must ship a `.env.example` at the root. This file is the **single source of truth** for what configuration the application requires.

### Structure & Grouping

Group variables by domain using comment blocks. Never dump a flat list of 40 variables.

```text
# ═══════════════════════════════════════════════════════════════
# Application
# ═══════════════════════════════════════════════════════════════
APP_TITLE=Sentinel
APP_VERSION=1.0.0
APP_ENVIRONMENT=development
APP_DEBUG=true

# ═══════════════════════════════════════════════════════════════
# Database
# ═══════════════════════════════════════════════════════════════
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
DATABASE_POOL_SIZE=5

# ═══════════════════════════════════════════════════════════════
# Security
# ═══════════════════════════════════════════════════════════════
JWT_SECRET=<GENERATE_RANDOM_256_BIT_KEY>
CORS_ORIGINS=["http://localhost:3000"]

# ═══════════════════════════════════════════════════════════════
# External Services
# ═══════════════════════════════════════════════════════════════
REDIS_URL=redis://localhost:6379
KAFKA_BROKER=localhost:9092
```

### Placeholder Conventions

| Placeholder | Meaning | Example |
|-------------|---------|---------|
| `<GENERATE_RANDOM_256_BIT_KEY>` | Must be generated; never hardcode | `openssl rand -hex 32` |
| `<CHANGE_ME>` | Must be replaced with project-specific value | API keys, passwords |
| `localhost` | Default local service endpoint | Databases, queues |

> [!WARNING]
> **NEVER commit real secrets in `.env.example`.** If a secret leaks into `.env.example`, rotate it immediately and purge the git history.

### Required Metadata

Every `.env.example` must include a header comment:

```text
# ═══════════════════════════════════════════════════════════════
# Project Name — Environment Configuration Template
# Copy to .env and customize: cp .env.example .env
# ═══════════════════════════════════════════════════════════════
```

---

## 2️⃣ Environment Tiers

_Reference Repos: Sentinel, Milan, ExpressKit, Gaming_

Applications run in three distinct tiers. Never mix tier behavior in a single environment flag.

| Tier | `APP_ENVIRONMENT` Value | Database | Cache | Logging | Debug | Purpose |
|------|------------------------|----------|-------|---------|-------|---------|
| **Development** | `development` | Local SQLite / PostgreSQL | Local Redis / in-memory | `DEBUG` | `true` | Local feature work |
| **Staging** | `staging` | Shared staging DB | Shared Redis | `INFO` | `false` | Pre-production validation |
| **Production** | `production` | Primary DB (replicated) | Redis cluster / ElastiCache | `WARN` / `ERROR` | `false` | Live traffic |

### Tier-Specific Rules

```text
# Development
APP_DEBUG=true
LOG_LEVEL=debug
DATABASE_URL=postgresql://localhost/dev_db

# Staging
APP_DEBUG=false
LOG_LEVEL=info
DATABASE_URL=postgresql://staging-db.example.com/staging_db

# Production
APP_DEBUG=false
LOG_LEVEL=warn
DATABASE_URL=postgresql://prod-db.example.com/prod_db?sslmode=require
```

> [!IMPORTANT]
> **Production must enforce `sslmode=require`** on PostgreSQL connections. Development and staging may use `sslmode=disable` for local setups, but this must never leak to production.

### The `NEXT_PUBLIC_` Convention (Frontend)

Frontend environment variables are embedded at build time. Prefix them explicitly to avoid accidentally exposing backend secrets.

```text
# Backend only (never exposed to browser)
DATABASE_URL=postgresql://...
JWT_SECRET=...

# Frontend safe (embedded in JS bundle)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_WS_URL=wss://ws.example.com
```

> [!TIP]
> In Next.js, any env var without the `NEXT_PUBLIC_` prefix is server-only. Use this as a safety net against secret leakage.

---

## 3️⃣ Secret Management

_Reference Repos: Sentinel, ExpressKit, Milan_

### The Golden Rule

> [!WARNING]
> **`.env` is gitignored. `.env.example` is committed. Secrets live in a secret manager, not in files.**

### Secret Rotation Policy

| Secret Type | Rotation Frequency | Method |
|-------------|-------------------|--------|
| JWT signing keys | 90 days | Generate new key, support dual-key decode during transition |
| Database passwords | 60 days | Automated via secret manager; app restarts on rotation |
| Third-party API keys | 30 days | Staggered rotation; old key invalidated after new key is confirmed |
| OAuth client secrets | 90 days | Provider dashboard + CI secret update |

### Implementation Pattern

```text
# 1. Developer copies template
cp .env.example .env

# 2. Developer fills local values (gitignored)
# 3. CI/CD injects production secrets via environment variables or secret manager
# 4. Application reads from process.env at runtime
```

> [!NOTE]
> **Never print secrets to logs.** A single leaked log line can compromise the entire system. Use redaction middleware for any structured logging that might capture request headers or payloads.

---

## 4️⃣ Configuration Validation

_Reference Repos: ExpressKit, Sentinel, DocsSense_

The application must validate all required environment variables at startup. Missing or malformed config is a deployment failure, not a runtime surprise.

### Validation Rules

1. **Required variables must be present.** Fail fast with a descriptive error.
2. **Ports must be integers in valid range (1–65535).**
3. **URLs must be parseable.** Use a URL parser, not string matching.
4. **Boolean flags must accept `true`/`false` only.** Never accept `"1"`, `"yes"`, `"on"`.

### Example Validation (ExpressKit Pattern)

ExpressKit centralizes config in `expresskit.config.ts` with environment overrides and runtime validation. This prevents "it works on my machine" failures caused by missing env vars.

```typescript
// Runtime config validation pattern
const required = ["DATABASE_URL", "JWT_SECRET", "PORT"];
const missing = required.filter((key) => !process.env[key]);
if (missing.length > 0) {
  throw new Error(`Missing required environment variables: ${missing.join(", ")}`);
}
```

---

## 5️⃣ `.gitignore` Discipline

_Reference: Engineering Standards_

The `.gitignore` must enforce the boundary between committed templates and local secrets.

```text
# Environment
.env
.env.local
.env.*.local
.env.production
.env.staging

# Keep the template
!.env.example
```

> [!WARNING]
> **If `.env` is accidentally committed, rotate all secrets in that file immediately.** Git history may retain the leaked values even after removal.

---

## 6️⃣ Environment Promotion

_Reference Repos: Gaming, Sentinel, Omega_

Promoting code from development → staging → production must be a deterministic, repeatable process.

### Promotion Rules

1. **Staging is a production clone.** Same Docker images, same environment variables (with tier-specific overrides), same database engine.
2. **No "it works on my machine" promotions.** If it wasn't tested in staging, it doesn't go to production.
3. **Environment variables are promotion artifacts.** They are versioned in the secret manager, not in the repo.
4. **Rollback is an environment change, not a code revert.** To rollback, change the deployed image tag, not the git history.

### Promotion Checklist

```text
[ ] Code merged to main
[ ] CI passed on main
[ ] Staging deployed and smoke tests passed
[ ] Database migrations applied to staging
[ ] Feature flags toggled ON in staging
[ ] Manual QA signed off
[ ] Production deployment scheduled
[ ] Rollback plan documented (image tag to revert to)
```

---

## 🚫 Anti-Patterns

> [!WARNING]
> **DO NOT DO THE FOLLOWING:**
>
> - Committing `.env` files to version control.
> - Hardcoding secrets in source code (even in `config/` files).
> - Using the same JWT secret across staging and production.
> - Disabling SSL in production because "staging doesn't use it."
> - Sharing a single `.env` across multiple services in a monorepo.

---

## 🔄 Revisit When

When introducing a new secret manager (HashiCorp Vault, AWS Secrets Manager, Doppler) or a new deployment target (serverless, edge functions), revisit this section to confirm the validation and rotation policies still align with the new infrastructure.
