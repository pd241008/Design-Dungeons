# 🧪 Testing Architecture

> **Source Projects:** DocsSense (228+ tests), DevTrace (18 unit tests), SentinalMesh (90 tests)
>
> Testing is not a phase. It is a continuous validation layer that proves the system behaves correctly under real constraints, mocked dependencies, and edge cases.

---

## 1️⃣ Philosophy

### The Three-Layer Test Strategy

Every project should have three isolated, independently executable test suites:

| Layer | Responsibility | Example |
| :--- | :--- | :--- |
| **Unit** | Pure functions, business logic, validators | Auth service hashing, payload validation |
| **Integration** | Database queries, API contracts, message queues | `/api/auth/otp` → MongoDB upsert → SMTP delivery |
| **E2E** | Critical user journeys across the full stack | Upload document → chunk → embed → query → answer |

> [!IMPORTANT]
> Tests must be **deterministic**. If a test fails intermittently, it is not a flaky test — it is a hidden race condition or shared state bug.

---

## 2️⃣ 100% Mocked Infrastructure

### Core Principle

> [!TIP]
> **No live external dependencies in CI.** Tests must run in a sandboxed environment with no network calls to real databases, APIs, or cloud services.

### Implementation Patterns

#### In-Memory Databases

Replace real databases with ephemeral in-memory instances for test isolation:

```typescript
// Jest + mongodb-memory-server
beforeAll(async () => {
  const mongod = await { MongoMemoryServer }.create();
  const uri = mongod.getUri();
  mongoose.connect(uri);
});
afterAll(async () => {
  await mongoose.disconnect();
  await mongod.stop();
});
```

```python
# pytest + fakeredis
import fakeredis
redis_client = fakeredis.FakeRedis()
```

#### Pre-emptive Monkey-Patching

Intercept side effects *before* the application module loads:

```python
# conftest.py
import sys
from unittest.mock import MagicMock

# Patch Redis before importing the FastAPI app
sys.modules["redis"] = MagicMock()
sys.modules["redis.asyncio"] = MagicMock()

# Now the app initializes with fake Redis
from app.main import app
```

#### Mock Service Worker (MSW) for Frontend

Intercept all `fetch` calls at the network layer:

```typescript
// mocks/handlers.ts
export const handlers = [
  rest.get("/api/auth/session", (req, res, ctx) => {
    return res(ctx.json({ user: mockUser }));
  }),
];

// test-setup.ts
beforeAll(() => { server.use(...handlers) });
afterAll(() => { server.close() });
```

---

## 3️⃣ Async Task Testing

### The Problem

Celery, Kafka consumers, and background workers are asynchronous by nature. Testing them requires turning async pipelines into synchronous, deterministic flows.

### The Solution

```python
# pytest + celery
@pytest.fixture
def celery_app():
    app.conf.task_always_eager = True  # Execute synchronously
    app.conf.task_eager_propagates = True  # Raise exceptions immediately
    return app

def test_document_ingestion(celery_app):
    result = ingest_document.delay(file_path="test.pdf")
    assert result.get() == {"status": "completed", "chunks": 42}
```

> [!IMPORTANT]
> `task_always_eager=True` is the only safe way to test Celery pipelines. Without it, tests depend on a running broker and worker process, which introduces network flakiness.

---

## 4️⃣ Frontend Testing Stack

### Component Unit Tests

| Framework | Purpose | Example |
| :--- | :--- | :--- |
| **Vitest** | Fast unit tests with React Testing Library | Button click → state update |
| **React Testing Library** | DOM assertions, user-centric queries | `screen.getByRole("button", { name: /upload/i })` |
| **MSW** | API mocking without backend | Spoof degraded/down server states |

### E2E Browser Tests

| Framework | Purpose |
| :--- | :--- |
| **Playwright** | Cross-browser journey specs against live dev server |

```typescript
// e2e/upload-journey.spec.ts
test("user uploads document and receives answer", async ({ page }) => {
  await page.goto("http://localhost:3000");
  await page.getByLabel("Email").fill("test@example.com");
  await page.getByRole("button", { name: "Sign In" }).click();
  await page.getByLabel("Upload Document").setInputFiles("test.pdf");
  await expect(page.getByText("Processing complete")).toBeVisible();
});
```

### Complex Mocking

```typescript
// Mock Next.js internals
vi.mock("next/navigation", () => ({
  useRouter: () => ({ push: vi.fn(), replace: vi.fn() }),
  useSearchParams: () => new URLSearchParams(),
}));

// Mock Canvas for Chart.js
HTMLCanvasElement.prototype.getContext = vi.fn(() => ({
  fillRect: vi.fn(),
  clearRect: vi.fn(),
  getImageData: vi.fn(() => ({ data: [] })),
}));
```

---

## 5️⃣ Test Organization

### Directory Convention

```
project-root/
├── __tests__/
│   ├── unit/
│   │   ├── auth.service.test.ts
│   │   └── validators.test.ts
│   ├── integration/
│   │   ├── auth.api.test.ts
│   │   └── database.test.ts
│   └── e2e/
│       ├── upload-journey.spec.ts
│       └── query-journey.spec.ts
├── conftest.py
├── mocks/
│   ├── handlers.ts
│   └── fixtures.ts
└── test-result-*.md
```

### Naming Conventions

| Pattern | Purpose | Example |
| :--- | :--- | :--- |
| `*.test.ts` | Unit / integration tests | `auth.service.test.ts` |
| `*.spec.ts` | E2E journey specs | `upload-journey.spec.ts` |
| `conftest.py` | Shared pytest fixtures | Database connection, mock Redis |
| `fixtures/*.json` | Test data payloads | `otp_request.json`, `document_payload.json` |

---

## 6️⃣ Execution Commands

```bash
# Node.js (Jest / Vitest)
npm run test              # All suites
npm run test:unit         # Unit + integration only
npm run test:e2e          # Playwright browser suite

# Python (pytest)
pytest -v                 # All tests
pytest tests/unit/        # Unit only
pytest -k "integration"   # Filter by keyword

# Rust (cargo test)
cargo test                # All tests
cargo test --lib          # Library only (no integration)
```

---

## 7️⃣ The Test Checklist

Before merging a PR, verify:

- [ ] **Unit tests** cover all new business logic paths
- [ ] **Integration tests** validate API contracts against the real schema
- [ ] **E2E tests** cover the critical user journey affected by the change
- [ ] All tests run **offline** with no external network dependencies
- [ ] Test execution time is **under 60 seconds** for unit + integration
- [ ] No test relies on **shared mutable state** between test blocks
- [ ] Mocks are **realistic** — they should mimic real failure modes, not just happy paths

---

## 8️⃣ Reference Implementations

| Repo | Tests | Highlights |
| :--- | :--- | :--- |
| **DocsSense** | 228+ | mongodb-memory-server, pre-emptive monkey-patching, MSW, Playwright |
| **DevTrace** | 18 | Rust unit tests for momentum, penetration epsilon, Givens rotation |
| **SentinalMesh** | 90 | 41 Go + 49 Python, deterministic simulator validation |
| **Gamify Pipeline** | 100% mocked | Offline automated testing with zero live API calls |
