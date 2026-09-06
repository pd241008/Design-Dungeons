# ⏳ Changelog

> All notable additions to this playbook will be documented in this file.

---

### September 6, 2026

- 📚 **Refactored** `01-documentation/adrs/` from a bulk project dump (28 files) into a
  **template + curated examples + catalog** model: one canonical template,
  [`adr-template.md`](01-documentation/adr-template.md), six curated example ADRs in
  `01-documentation/adrs/examples/`, and a
  [`catalog`](01-documentation/adrs/README.md) routing every decision to its canonical
  home in the source project repo (BlackIce 4, ChaosSeal 4, Helios 5, Pissta 9,
  OrbitLite 3, AI-Agent 1, DevTrace 1, Milan 1, OmniStat 1).
- 🎁 **Added** three ML research focus docs under `13-ml-and-research/`:
  [`artifact-design.md`](13-ml-and-research/artifact-design.md) (reviewer-centric
  reproducibility artifact design — claim maps, typed R1–R4, review paths),
  [`negative-results.md`](13-ml-and-research/negative-results.md) (TASCP negative-result
  autopsy), and [`ffi-validation.md`](13-ml-and-research/ffi-validation.md) (Midas
  cross-language FFI validation).

---

### September 6, 2026

- 📝 **Added** `chaosseal/` ADRs (4): Benettin tangent update fix, Jacobian
  inertia placement, 256-bit/epoch claim reframe on metastability, and the
  bounded wrapped-coupling redesign (`01-documentation/adrs/chaosseal/`).
- 🧠 **Added** `chaosseal/` postmortem: the metastable pendulum + fixed-point
  saturation "phantom attractor" (`02-postmortems/chaosseal/`).
- 🔢 **Total inventory:** 28 ADRs, 19 postmortems across 7 projects.
- 🧬 **Extended** `13-ml-and-research/research-patterns.md` §1 (ChaosSeal) with
  the independent cross-simulator verification gate.
- 🧭 **Updated** `blackice/001-canonical-exhaustive-mixed-norm-evaluation`: ADR
  condensed to its canonical decision form (pros/cons preserved); the detailed
  Section-III reproduction + scaling evidence is retained in
  `03-audit-reports/004` within the BlackIce project repo.

### June 18, 2026

- ✨ **Added** `00-foundations/engineering-standards.md` translated from the legacy Design Doc PDF.
- 💅 **Styled** all core markdown files to adhere to the new premium formatting standard.
- 📖 **Added** `01-documentation/readme-patterns.md` covering project abstract and variant structures.
- ⚖️ **Added** `01-documentation/adr-template.md` as the standard decision format.
- 📝 **Added** `ADR-003: Dual Auth for Milan` (`01-documentation/adrs/003-milan-dual-auth.md`).
- 📝 **Added** `ADR-002: CQRS & Append-Only Event Store for DevTrace`.
- 📝 **Added** `ADR-001: Dropping Redis from OmniStat-Core` (`01-documentation/adrs/001-omnistat-no-redis.md`).
- 🧠 **Added** `00-foundations/philosophy.md` detailing engineering values.
- 🎉 **Initialized** the playbook.

### August 16, 2026

- ⚡ **Added** `08-async-and-queues/queue-architectures.md` covering conveyor belt ingestion (DevTrace), Celery task queues (DocsSense), cron-triggered serverless workers (Gaming), and event-driven streaming (Sentinel).
- 🔭 **Added** `09-observability/observability-patterns.md` covering the three pillars (logs, metrics, traces), embedded observability (DevTrace), edge defense (Gaming), infrastructure monitoring (Sentinel), and alerting philosophy.
- 🚀 **Added** `12-deployment/deployment-patterns.md` covering multi-cloud serverless (Gaming), polyglot container mesh (Sentinel), embedded deployment (DevTrace), configuration management, health checks, and rollback strategy.
- 🧪 **Added** `10-testing/testing-architecture.md` covering three-layer test strategy, 100% mocked infrastructure, async task testing, frontend testing stack (Vitest, RTL, Playwright, MSW), test organization, and execution commands.
- 🧬 **Added** `13-ml-and-research/research-patterns.md` covering reproducibility discipline (ChaosSeal), experiment tracking, prototype vs. production separation (Midas), RAG pipeline architecture (DocsSense), adversarial ML patterns (Midas), and LSH/vector search (PrimeVector).
- 🏗️ **Added** `04-reference-code/03-strangler-fig-migration.md` documenting the progressive monolith-to-mesh migration pattern proven in Sentinel's Python → Go/Scala migration.

### August 25, 2026

- 📦 **Consolidated** 21 ADRs and 15 postmortems from all local projects into project-scoped subdirectories (`blackice/`, `helios/`, `pissta/`, `orbitlite/`, `ai-agent/`, `sentinalmesh/`).
- 🔢 **Total inventory:** 24 ADRs, 17 postmortems across 6 projects.
- 🧹 **Fixed** placeholder dates (`[Month, Year]`) in `001-why-we-removed-redis.md` and `002-sentinalmesh-simulator-bugs.md`.
- 📝 **Extended** `adr-template.md` with missing `Consequences` and `Revisit When` sections to match actual ADR structure.
- 📖 **Updated** README with project subdirectory navigation.

### August 12, 2026

- ⚙️ **Added** `05-git-and-versioning/` covering Conventional Commits, branch naming, commit body/footers, linking commits to documentation, decision framework, anti-patterns, and collaboration guidelines (merge strategies, conflict resolution, rollback procedures).
- 🌍 **Added** `06-environments/environment-standards.md` covering `.env.example` structure, environment tiers (development/staging/production), secret management and rotation policies, configuration validation, `.gitignore` discipline, and environment promotion rules.
- 🕵️ **Added** `02-postmortems/002-sentinalmesh-simulator-bugs.md` documenting the 8-bug retrospective from SentinelMesh (oracle bug, string mismatch, inverse scoring, uncalibrated thresholds, EWMA tie-break, unbounded window regression, spurious quorum conflation, latency escapement).
