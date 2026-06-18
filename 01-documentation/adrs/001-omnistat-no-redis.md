# 📜 ADR-001: Dropping Redis from OmniStat-Core

> **Status:** `Decided`
> **Date:** `June, 2026`

---

## 🌎 Context

OmniStat-Core processes and aggregates git event data. Initially, the architecture included Redis as a caching layer to store frequently accessed analytics computations and manage temporary session state, anticipating high volume read operations.

However, in reality, the system is currently processing **sub-1,000 events/day**. The read volume is low enough that querying the primary database directly, combined with simple in-process memory caching, is instantaneous for the user.

## 🛤️ Options Considered

1. **Keep Redis** as the caching layer to remain "future-proof."
2. **Drop Redis entirely**, use in-memory state for ephemeral data and SQLite/PostgreSQL directly for queries.

---

## 🎯 Decision

> [!IMPORTANT]  
> **Drop Redis entirely from the OmniStat-Core stack.**

## 🧠 Reasoning

At sub-1,000 events/day, the operational overhead of running, monitoring, and maintaining a Redis instance far outweighed any negligible cache hit benefit. Managing another infrastructure dependency (and its connection pooling, error handling, and memory limits) was slowing down development without providing actual value at our current scale.

> [!NOTE]
> **Premature abstraction is a tax; we chose to optimize for simplicity and speed of iteration instead.**

This decision reverses at roughly ~50k events/day (a limit observed during local load tests) when database load becomes a real concern.

## ⚖️ Consequences

- **Good:** 🟢 Simpler infrastructure, one fewer moving part to monitor, significantly easier local development setup for new engineers.
- **Bad:** 🔴 We give up centralized, persistent caching. If traffic spikes unpredictably, the database takes the direct hit, which could degrade read performance.

## 🔄 Revisit When

When ingestion consistently exceeds **50,000 events/day**, or when we notice slow query performance degrading the user experience on the analytics dashboard.
