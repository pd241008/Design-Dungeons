# 🏰 Design Dungeons

### 📓 The Engineering Playbook

This repository is a living document of my engineering decisions, patterns, and conventions. It is written from real project experience, not theory. It serves as a continuous record of how I build systems, designed for senior engineers to understand the "why" behind the "what," and for junior engineers as a practical reference guide.

---

## 🧭 How to Read This

> [!IMPORTANT]  
> **Every pattern here comes from a real decision made under real constraints. If it links to a repo, that's where I actually used it.**

This is not a generic list of "best practices." If a pattern or architecture choice is documented here, it means I have built it, supported it, and dealt with its consequences in production.

---

## 🗺️ Navigation

| Section                                      | Description                                                        |
| -------------------------------------------- | ------------------------------------------------------------------ |
| **[00. Foundations](./00-foundations/)**     | Core philosophy and engineering standards.                         |
| **[01. Documentation](./01-documentation/)** | README patterns and Architecture Decision Records (ADRs).          |
| ├─ [ADRs](./01-documentation/adrs/) | `adr-template.md` + [6 curated examples](./01-documentation/adrs/#-curated-examples-study-these) + [full catalog](./01-documentation/adrs/README.md) linking every decision to its source repo. Covers BlackIce (4), ChaosSeal (4), Helios (5), Pissta (9), OrbitLite (3), AI-Agent (1), DevTrace (1), Milan (1), OmniStat (1). |
| **[02. Postmortems](./02-postmortems/)**     | Failure stories, mistakes, and what we learned.                    |
| ├─ [Postmortems](./02-postmortems/) | `omnistat/` (1) `blackice/` (3) `helios/` (3) `pissta/` (8) `orbitlite/` (1) `sentinalmesh/` (2) `chaosseal/` (1) — **19 total** |
| **[03. Project Structure](./03-project-structure/canonical-layouts.md)** | Canonical folder layouts for full-stack, systems, and ML repos. |
| **[04. Reference Code](./04-reference-code/)** | Actual code snippets proving the architecture in production.       |
| **[05. Git & Versioning](./05-git-and-versioning/05-git-and-versioning.md)** | Commit formats, PR templates, branch strategies, and collaboration. |
| **[06. Environments](./06-environments/environment-standards.md)** | `.env` standards, secret management, and environment promotion. |
| **[07. Database](./07-database/data-patterns.md)** | Event sourcing, CQRS, distributed consensus, and streaming-variance patterns (Omega). |
| **[08. Async & Queues](./08-async-and-queues/queue-architectures.md)** | Task idempotency, retries, and queue architectures. |
| **[09. Observability](./09-observability/observability-patterns.md)** | Metrics baselines, logging shapes, and alerting. |
| **[10. Testing](./10-testing/testing-architecture.md)** | What to test, async job testing, and full-stack test suites. |
| **[12. Deployment](./12-deployment/deployment-patterns.md)** | PM2, containers, and GitHub actions. |
| **[13. ML & Research](./13-ml-and-research/research-patterns.md)** | Experiment tracking and reproducible repos. |
| _11. Performance_                            | (Coming soon) Profiling, caching, and concurrency.                 |
| _14. Code Review_                            | (Coming soon) Checklists and what _not_ to block on.               |

---

_See [CHANGELOG.md](./CHANGELOG.md) for recent updates._
