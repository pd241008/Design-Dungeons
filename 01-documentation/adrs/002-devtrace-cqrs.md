# 📜 ADR-002: CQRS & Append-Only Event Store for DevTrace

> **Status:** `Decided`
> **Date:** `March, 2026`

---

## 🌎 Context

DevTrace intercepts and monitors system-level events (syscalls, network I/O, file access) in high-throughput environments using a Rust agent.

Initially, the architecture attempted to directly map incoming telemetry streams into normalized relational tables to make querying easy. However, telemetry generation drastically outpaces analysis. A single node can produce 10,000+ events per second. Trying to perform relational writes (with index updates, foreign key checks, and locks) at ingestion time caused severe backpressure, leading to dropped events and agent memory bloat.

## 🛤️ Options Considered

1. **Relational Ingestion:** Write events directly to normalized PostgreSQL tables.
2. **Document Store:** Dump events into MongoDB or Elasticsearch for schema-less ingestion.
3. **CQRS with Append-Only Store:** Separate the write path (append-only event store via SQLite/Disk) from the read path (projection views for querying).

---

## 🎯 Decision

> [!IMPORTANT]  
> **We will implement Command Query Responsibility Segregation (CQRS) with an append-only event store for DevTrace ingestion.**

## 🧠 Reasoning

Telemetry is immutable. An event that occurred cannot "un-occur."

By treating the ingestion layer strictly as an append-only log (the Command side), we remove all write contention. Rust's Tokio MPSC channels can flush serialized events to an append-only file or raw SQLite table with zero-copy overhead, trivially absorbing 10k+ events/sec.

The analysis dashboard (the Query side) operates asynchronously. Background projections read the immutable event log and build indexed relational views.

> [!NOTE]  
> **This intentionally decouples write throughput from read complexity.** If the query engine goes down, the agent continues buffering events to disk without data loss.

## ⚖️ Consequences

- **Good:** 🟢 Infinite ingestion scaling limited only by disk I/O. Zero write-locks or complex transactions during the critical interception phase.
- **Bad:** 🔴 Eventual consistency on the dashboard. Projections may be slightly delayed behind real-time ingestion. Increased architectural complexity due to managing two distinct models.
