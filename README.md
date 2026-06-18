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
| **[02. Postmortems](./02-postmortems/)**     | Failure stories, mistakes, and what we learned.                    |
| **[03. Project Structure](./03-project-structure/canonical-layouts.md)** | Canonical folder layouts for full-stack, systems, and ML repos. |
| _04. Git & Versioning_                       | (Coming soon) Commit formats, PR templates, and branch strategies. |
| _05. API Design_                             | (Coming soon) REST conventions, error schemas, and async handling. |
| _06. Database_                               | (Coming soon) Migration discipline, indexing, and schema changes.  |
| _07. Async & Queues_                         | (Coming soon) Task idempotency, retries, and queue architectures.  |
| _08. Observability_                          | (Coming soon) Metrics baselines, logging shapes, and alerting.     |
| _09. Testing_                                | (Coming soon) What to test, async job testing, and ML pipelines.   |
| _10. Performance_                            | (Coming soon) Profiling, caching, and concurrency.                 |
| _11. Deployment_                             | (Coming soon) PM2, containers, and GitHub actions.                 |
| _12. ML & Research_                          | (Coming soon) Experiment tracking and reproducible repos.          |
| _13. Code Review_                            | (Coming soon) Checklists and what _not_ to block on.               |

---

_See [CHANGELOG.md](./CHANGELOG.md) for recent updates._
