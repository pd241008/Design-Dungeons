# Postmortem: Why We Removed Redis

> **Date:** [Month, Year]  
> **Project:** OmniStat-Core

---

## 💥 What we expected

When designing the ingestion pipeline for OmniStat, we originally provisioned a Redis cluster to act as a buffer and high-speed cache between the API gateway and our PostgreSQL datastore. The expectation was that Redis would absorb write spikes during high-load periods, preventing the relational database from becoming a bottleneck.

We expected:

1. Improved ingestion latency.
2. Protection against database locks during bulk writes.
3. A foundation for future real-time pub/sub event routing.

## 📉 What actually happened

In production, the system rarely exceeded sub-1,000 events per day. Because of this low volume:

- The Redis cluster was massively underutilized.
- We introduced complex, asynchronous worker logic to drain the Redis queues and write to PostgreSQL, increasing the surface area for bugs and making local development difficult.
- We experienced occasional synchronization issues where the state in Redis did not immediately match the persistent state in PostgreSQL.

Essentially, we paid the operational and cognitive cost of a distributed cache without reaping any of the performance benefits.

## 🧠 What we learned

**Premature abstraction is a tax on velocity.**

We learned that:

- **Scale it when it breaks:** A single PostgreSQL instance can easily handle tens of thousands of writes per day without a caching layer.
- **Infrastructure is a liability:** Every new piece of infrastructure must justify its existence. If the problem can be solved by an in-memory state or a simpler database write, choose the simpler option.
- We established a new rule: This decision to re-introduce a caching layer will only reverse at roughly ~50k events/day (a limit observed during local load tests), when database load becomes a statistically significant concern.

_See the corresponding ADR: [ADR-001: Removing Redis from the Core Ingestion Path](../01-documentation/adrs/001-omnistat-no-redis.md)_
