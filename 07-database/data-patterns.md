# 🗄️ Database & Event-Driven Data

> **Source Projects:** Omega (CQRS + Event Sourcing + Raft consensus +
> Kafka choreography), OmniStat (removing Redis — see
> [`01-documentation/adrs/examples/migration-removing-dependency.md`](../01-documentation/adrs/examples/migration-removing-dependency.md)).
>
> Financial-correctness systems model state as an **append-only event log**.
> The write path (commands) and read path (queries) scale independently, and
> ordering/consistency is enforced by distributed consensus rather than a
> single database lock.

---

## 1️⃣ Event Sourcing & CQRS (Omega)

### The Append-Only Event Log

> [!IMPORTANT]
> **System state is strictly derived from events — never stored as mutable rows.**
> The event store is the source of truth; projections/materialized views are
> derived and rebuildable.

Omega models every micro-transaction as an immutable event appended to
PostgreSQL. The event log is the authoritative record; read models are
denormalized projections of it:

```
Client → Envoy (API Gateway)
            ├── POST /transactions  → Command API   (write model)
            └── GET /transactions   → Query API     (read model)

Command API  → publishes domain events  → Kafka (per-aggregate partitioning)
Event Processor (Go) → appends events   → PostgreSQL (event store)
                    → updates views     → Redis (materialized views)
Query API    → fetches views            → Redis
```

### Why CQRS Here

| Concern | Write side (Commands) | Read side (Queries) |
| :--- | :--- | :--- |
| **Traffic profile** | Low volume, high correctness | High volume, low latency |
| **Scaling** | Scale with event throughput | Scale independently (caching layers) |
| **Failure mode** | Duplicate/conflict detection | Stale-but-eventually-consistent views |

> [!TIP]
> CQRS is justified when the write and read paths have **different SLAs and
> scaling profiles** (financial correctness vs. fast reads). For a CRUD app
> with symmetric load, it's added complexity. DevTrace made the same call for
> its capture-vs-visualize split (see
> [`examples/cqrs-append-only-event-store.md`](../01-documentation/adrs/examples/cqrs-append-only-event-store.md)).

---

## 2️⃣ Distributed Consensus (Raft)

For transactions that must be **strictly ordered across nodes**, Omega layers
Raft leader election on top of the event log:

- A **Raft node** replicates the ordered command log across the cluster.
- The **leader** orders transactions; followers replicate and apply them.
- Taken together with Kafka's **per-aggregate partitioning**, event order is
  deterministic per aggregate (per-account, per-wallet).

> [!NOTE]
> Raft gives you **ordered, replicated logs with leader election** — it is not
> the same job as the Kafka event bus. In Omega, Kafka moves domain events
> between services (choreography), while Raft enforces *definitive* order for
> the money-path aggregates. Don't conflate the two.

---

## 3️⃣ Event Choreography & the Saga Orchestrator

Omega does **not** use a central orchestrator for every flow — it uses
**event choreography** over Kafka with partitioned topics, plus a Go **saga
orchestrator** for flows that genuinely span aggregates (multi-wallet
transfers, refunds):

- **Choreography:** each service reacts to domain events it subscribes to —
  no central coordinator for the happy path.
- **Saga:** for multi-step flows with compensating actions, the orchestrator
  tracks state and issues compensating events on failure.

```
Event Processor (Saga Orchestrator)
    ├── consume transfer-initiated
    ├── debit source wallet   (compensation: reverse debit)
    ├── credit sink wallet    (compensation: reverse credit)
    └── publish transfer-completed
```

> [!IMPORTANT]
> **Every saga step must have an explicit compensating action.** A saga without
> an inverse is a distributed transaction waiting to deadlock on retries.

---

## 4️⃣ Streaming Fraud Detection (Velocity Analysis)

The fraud engine runs **sliding-window velocity checks** as events stream
through Kafka — no batch scoring:

- Configurable thresholds (e.g. max transactions per window per account).
- Operates on the same event stream as the ledger (single read, two consumers).
- Scales as a separate Go service.

---

## 5️⃣ Lessons & Anti-Patterns

> [!WARNING]
> **Read models are not the source of truth.** If a projection drifts or is lost,
> rebuild it from the event log. If a bug writes directly to a materialized
> view instead of appending an event, you've broken event sourcing — hard to
> detect and expensive to repair.

> [!WARNING]
> **Removing a caching dependency is a real decision worth an ADR.**
> OmniStat dropped Redis entirely (see
> [`examples/migration-removing-dependency.md`](../01-documentation/adrs/examples/migration-removing-dependency.md))
> after the vector-search layer made it redundant. Before adding CQRS + event
> sourcing + Raft, ask whether your system is a financial-ledger use case — a
> plain relational DB with transactions may be the more honest architecture.

---

## 6️⃣ Reference

| Repo | Pattern | Highlights |
| :--- | :--- | :--- |
| **Omega** | CQRS + Event Sourcing + Raft | Append-only event log in PostgreSQL, per-aggregate Kafka partitioning, Go saga orchestrator, streaming velocity fraud |
| **DevTrace** | CQRS + append-only event store | Capture (write) / visualize (read) split at Rust→Node boundary |

_This fills the previously-empty **07. Database** navigation slot (migration
discipline, indexing, and schema-change patterns to be added as projects
accumulate)._