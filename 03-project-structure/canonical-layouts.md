# 🏗️ Canonical Project Structures

> **Pragmatism over abstraction.** A folder structure should map directly to the system's architecture, deployment boundaries, and observability needs.

This document defines the canonical monorepo layouts extracted from actual production repositories (`Milan`, `Taskiee`, `DevTrace`, `Adv-Guard`).

---

## 🧭 The Core Philosophy

Before looking at the layouts, understand the rules that govern them:

> [!IMPORTANT]
> 1. **Root-Level Decoupling:** Frontend and Backend are ALWAYS physically separated at the root (e.g., `frontend/` and `backend/`). Mixing build tools, `package.json` files, or dependencies is strictly forbidden. 
> 2. **Pragmatic Monorepos:** Keep microservices or decoupled services in a single repository until deployment friction or team size dictates otherwise.
> 3. **Visibility First:** "Instrument before you optimize." Tooling, telemetry, and observability (like `logger/` or `runner/`) are treated as first-class citizens at the root level, not buried in a `utils/` directory.

---

## 1️⃣ The Standard Full-Stack 

_Reference Repos: Taskiee, Milan, Gram-Sevak_  
_Tech Stack: Next.js + Node.js (Express) / Python (FastAPI)_

This is the default layout for web platforms and products. It enforces a hard line between the client-side rendering application and the backend API server.

```text
project-root/
├── frontend/               # (or nextjs-frontend/)
│   ├── app/                # Next.js App Router
│   │   ├── (auth)/         # Grouped routes (e.g., login, register)
│   │   ├── dashboard/      # Protected routes
│   │   ├── layout.tsx      # Root layout
│   │   └── page.tsx        # Landing page
│   ├── components/         # Reusable UI elements
│   │   ├── ui/             # Base elements (buttons, inputs)
│   │   └── forms/          # Complex form structures
│   ├── types/              # Shared TypeScript definitions
│   │   └── index.d.ts      # Global interfaces
│   ├── utils/              # Client-side helpers
│   │   └── api.ts          # Axios/Fetch wrappers
│   └── public/             # Static assets (images, fonts)
│
├── backend/
│   ├── src/                # (or app/ for Python) Core API logic
│   │   ├── controllers/    # Request validation and parsing
│   │   ├── services/       # Core business logic
│   │   ├── routes/         # Express/FastAPI route definitions
│   │   └── middleware/     # Auth, error handling, rate limiting
│   ├── data/               # Database schemas, migrations, and ORM models
│   │   ├── migrations/     # SQL migration scripts
│   │   └── schema.prisma   # (or models.py) Data definitions
│   └── tests/              # Backend unit/integration tests
│       ├── integration/    # API endpoint tests
│       └── unit/           # Service layer tests
│
├── .gitignore
└── README.md
```

> [!TIP]
> **Why?** It ensures that a vulnerability in a frontend dependency doesn't compromise the backend build, and makes it trivial to containerize the frontend and backend into separate Docker images.

---

## 2️⃣ The Systems / Multi-Binary Monorepo

_Reference Repos: DevTrace, Aegis, OmniStat-Core_  
_Tech Stack: Rust, Go, C++, alongside Web UI_

This layout is for complex, highly concurrent systems where the "backend" actually consists of several independent engines, agents, or binaries written in different languages.

```text
project-root/
├── ui/                     # Web dashboard or visualization layer
│   ├── src/                # Frontend codebase (React/Vue/Svelte)
│   │   ├── components/     # UI components
│   │   └── views/          # Top-level page views
│   └── public/             # Static assets
│
├── logger/                 # Dedicated telemetry/ingestion engine
│   ├── src/                # e.g., Rust event processor
│   │   ├── main.rs         # Binary entrypoint
│   │   ├── ingestion/      # TCP/UDP stream handling
│   │   └── parsing/        # High-performance payload parsers
│   └── database/           # Append-only store bindings
│       └── sqlite.rs       # Interface for local storage
│
├── runner/                 # Execution environment or agent
│   ├── src/                # e.g., Go worker pool
│   │   ├── main.go         # Agent entrypoint
│   │   ├── workers/        # Task executors
│   │   └── sync/           # Concurrency primitives
│   └── config/             # Agent configuration schemas
│
├── packger/                # Multi-platform distribution scripts
│   ├── go/                 # Go build and release scripts
│   ├── npm/                # JS package configuration
│   └── python/             # PyPI wheel generation
│
├── .gitignore
└── README.md
```

> [!TIP]
> **Why?** By making `logger/` and `runner/` root-level domains, we prioritize visibility. This directly mirrors the deployment reality, where each root directory typically compiles to a distinct artifact or container.

---

## 3️⃣ The Research / ML Sandbox

_Reference Repos: Adv-Guard, AQI-Prediction-Model_  
_Tech Stack: Python (PyTorch/TensorFlow), Jupyter, Next.js_

This layout prioritizes reproducibility and experimentation. Research code is inherently messy, so the goal is to isolate the chaos from the stable production layers.

```text
project-root/
├── notebooks/              # Jupyter notebooks for EDA and messy research
│   ├── 01_data_explo.ipynb # Numbered for chronological flow
│   └── 02_model_test.ipynb # Training iterations
│
├── datasets/               # Raw and processed data (often gitignored)
│   ├── raw/                # Immutable source data
│   └── processed/          # Cleaned, tokenized, or normalized data
│
├── models/                 # Serialized weights (e.g., .pt, .onnx)
│   ├── v1_baseline.pt      # Baseline model weights
│   └── v2_optimized.onnx   # Production-ready quantized model
│
├── backend/                # Inference API
│   ├── app/                # Flask/FastAPI server
│   │   ├── main.py         # API entrypoint
│   │   ├── predict.py      # Inference logic
│   │   └── schemas.py      # Pydantic validation models
│   └── data/               # API-specific models
│
├── nextjs-frontend/        # User interface for the model
│   ├── app/                # Next.js routes
│   └── components/         # Interactive UI (e.g., file uploaders)
│
├── requirements.txt        # Strict dependency locking
└── README.md
```

> [!TIP]
> **Why?** Research code changes rapidly. By strictly isolating `notebooks/` and `datasets/` from the `backend/` inference API, you can prototype fearlessly without breaking the application's contract with the frontend.
