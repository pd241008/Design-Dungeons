# Reference: Strangler Fig Migration

> **Concept Proven:** "Progressive monolith-to-mesh migration by intercepting traffic at the gateway and routing endpoints to new services incrementally."
> **Source Project:** Sentinel (Python → Go/Scala polyglot migration)

---

## The Problem

Legacy systems accumulate technical debt, but rewriting them all at once is high-risk. The Strangler Fig pattern allows you to migrate incrementally while keeping the old system running.

```
┌─────────────────────────────────────┐
│         Old Monolith (Python)       │
│  ┌─────────┐ ┌─────────┐ ┌──────┐ │
│  │  Auth   │ │  Users  │ │ Files │ │
│  └─────────┘ └─────────┘ └──────┘ │
└─────────────────────────────────────┘
           ▲              ▲
           │              │
     All traffic      All traffic
```

## The Solution

Route all traffic through a gateway. New functionality is built in new services. The gateway progressively redirects endpoints from the monolith to the new services. The monolith shrinks until it can be decommissioned.

```
                        ┌─────────────────────────────┐
                        │   API Gateway (Go)           │
                        │   ┌─────────────────────┐    │
                        │   │ Auth  → New Service  │    │
                        │   │ Users → Monolith    │    │
                        │   │ Files → New Service │    │
                        │   └─────────────────────┘    │
                        └─────────────────────────────┘
                                     ▲
                        ┌────────────┴────────────┐
                        ▼                         ▼
                ┌──────────────┐          ┌──────────────┐
                │ New Services │          │  Monolith    │
                │ (Go/Scala)   │          │  (Python)    │
                └──────────────┘          └──────────────┘
```

---

## Implementation

### Phase 1: Deploy the Gateway

The gateway accepts all traffic and proxies everything to the monolith.

```go
// gateway/router.go
func (g *Gateway) Route(ctx context.Context, req *http.Request) (*http.Response, error) {
    // All traffic goes to monolith initially
    return g.monolith.Forward(ctx, req)
}
```

### Phase 2: Intercept One Endpoint

Move a single endpoint to a new service.

```go
func (g *Gateway) Route(ctx context.Context, req *http.Request) (*http.Response, error) {
    switch req.URL.Path {
    case "/api/auth":
        // New service handles auth
        return g.authService.Forward(ctx, req)
    default:
        // Everything else still goes to monolith
        return g.monolith.Forward(ctx, req)
    }
}
```

### Phase 3: Incremental Migration

Move endpoints one by one, validating each migration.

```go
var routes = map[string]Service{
    "/api/auth":        authService,    // ✅ Migrated
    "/api/users":       monolith,       // ⏳ Pending
    "/api/files":       fileService,    // ✅ Migrated
    "/api/notifications": monolith,    // ⏳ Pending
}

func (g *Gateway) Route(ctx context.Context, req *http.Request) (*http.Response, error) {
    if service, ok := routes[req.URL.Path]; ok {
        return service.Forward(ctx, req)
    }
    return g.monolith.Forward(ctx, req)
}
```

### Phase 4: Decommission the Monolith

Once all endpoints are migrated, the monolith is decommissioned.

```go
func (g *Gateway) Route(ctx context.Context, req *http.Request) (*http.Response, error) {
    if service, ok := routes[req.URL.Path]; ok {
        return service.Forward(ctx, req)
    }
    return nil, fmt.Errorf("endpoint not found: %s", req.URL.Path)
}
```

---

## Key Decisions

| Decision | Rationale |
| :--- | :--- |
| **Gateway is the single entry point** | No client-side changes needed; transparent migration |
| **One endpoint at a time** | Limits blast radius; each migration is independently testable |
| **Keep monolith running during migration** | Zero downtime; rollback is instant by reverting gateway config |
| **Data migration last** | Services initially share the monolith database; dual-write until cutover |

---

## Anti-Patterns

> [!WARNING]
> **DO NOT DO THE FOLLOWING:**
>
> - **The Big Bang Rewrite:** Rewriting the entire system at once. This almost always fails.
> - **Client-side routing:** Requiring frontend changes to point to new services breaks the migration contract.
> - **Skipping the gateway:** Direct service-to-monolith communication creates hidden coupling.
> - **Parallel data models:** Maintaining two schemas simultaneously without a clear cutover plan creates data drift.

---

## Reference: Sentinel Migration

Sentinel migrated from a Python monolith to a polyglot mesh:

| Layer | Before | After |
| :--- | :--- | :--- |
| **API** | Python Flask | Go gateway + services |
| **Streaming** | Python | Scala ZIO 2 |
| **ML/AI** | Python | Python (only ML remains) |
| **Database** | SQLite | PostgreSQL + ClickHouse + Neo4j + Qdrant |

**Result:** ~47 redundant Python files deleted. Platform services now run in Go (memory-safe, compiled); streaming runs in Scala (ZIO, circe, Kafka); ML retains Python (PyTorch, LangChain).

---

## When to Use

| Scenario | Strangler Fig | Big Bang |
| :--- | :--- | :--- |
| **Production system with users** | ✅ | ❌ |
| **Multiple teams working in parallel** | ✅ | ❌ |
| **Need to validate architecture before full commitment** | ✅ | ❌ |
| **Greenfield project with no users** | ❌ | ✅ |
