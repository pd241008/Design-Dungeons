# 🗂️ Architecture Decision Records

> **This is the catalog.** A single template plus curated examples live here.
> Every decision is linked to its **canonical home** in the source project repo;
> this file does not duplicate decision bodies — it routes you to them and
> highlights the curated examples worth studying.

---

## ✍️ How to Write an ADR

- Use the single canonical [`adr-template.md`](../adr-template.md).
- One decision per ADR. Record: **Context** (constraints, facts, no bias) →
  **Options Considered** → **Decision** (one-sentence why) → **Consequences**
  (good / bad) → **Revisit When**.
- The body lives in the **project repo** it decides. This catalog routes there.
- Only decisions with **no** source file elsewhere live inline (marked below).

---

## ⭐ Curated Examples (study these)

Pick the example that matches your decision type, then follow
[`adr-template.md`](../adr-template.md).

| # | Pattern | Example file | What it demonstrates |
| :- | :--- | :--- | :--- |
| 1 | Honesty & verify-before-trust | [`examples/honesty-verification-reframe.md`](./examples/honesty-verification-reframe.md) | Reframing an over-claimed headline result when the model is found non-integrable (ChaosSeal) |
| 2 | Evaluation integrity & data provenance | [`examples/evaluation-integrity-split.md`](./examples/evaluation-integrity-split.md) | Fixing a stale evaluation split with per-city policy + split provenance (Helios) |
| 3 | Methodology replacement & canonical-vs-legacy | [`examples/methodology-replacement-exhaustive.md`](./examples/methodology-replacement-exhaustive.md) | Replacing a flawed evaluator with a canonical one, keeping the legacy for provenance (BlackIce) |
| 4 | CQRS & append-only event store | [`examples/cqrs-append-only-event-store.md`](./examples/cqrs-append-only-event-store.md) | Splitting write path from read path (DevTrace) — Design_Doc-original |
| 5 | Authentication / delegated authority | [`examples/dual-auth-oauth-session.md`](./examples/dual-auth-oauth-session.md) | Dual auth (OAuth + OTP) (Milan) — Design_Doc-original |
| 6 | Dependency removal / simplify-then-verify | [`examples/migration-removing-dependency.md`](./examples/migration-removing-dependency.md) | Dropping Redis (OmniStat) — Design_Doc-original |

---

## 📚 Full Catalog (project-scoped)

Each entry links to the ADR's **canonical home** in the sibling project repo.
Only the 3 "Design_Doc-original" decisions above are duplicated inline because
they have no other source file.

### BlackIce — adversarial-ML defense framework
Canonical: `Legacy/BlackIce/docs/01-documentation/adrs/`
- [001 Canonical exhaustive mixed-norm evaluation](https://github.com/pd241008) (see `examples/methodology-replacement-exhaustive.md`)
- [002 Unified adversarial training](https://github.com/pd241008)
- [003 Multi-seed validation](https://github.com/pd241008)
- [004 Streaming data loader](https://github.com/pd241008)

### ChaosSeal — cryptographic protocol for LEO swarm
Canonical: `Legacy/ChaosSeal/docs/01-documentation/adrs/`
- [ADR-001 Benettin tangent Jacobian flow](https://github.com/pd241008)
- [ADR-002 Jacobian inertia placement](https://github.com/pd241008)
- [ADR-003 Entropy-claim reframe on metastability](https://github.com/pd241008) (see `examples/honesty-verification-reframe.md`)
- [ADR-004 Bounded wrapped-coupling redesign](https://github.com/pd241008)

### Helios — land-surface-temperature ML pipeline
Canonical: `Legacy/Helios/docs/adrs/`
- [001 Target leakage guard](https://github.com/pd241008)
- [002 STAC pagination limit](https://github.com/pd241008)
- [003 SHAP silent-failure logging](https://github.com/pd241008)
- [004 Full-resolution ensemble memory strategy](https://github.com/pd241008)
- [005 Per-city temporal split policy](https://github.com/pd241008) (see `examples/evaluation-integrity-split.md`)

### Pissta — VLSI statistical static timing analysis
Canonical: `Legacy/Pissta/docs/adrs/`
- [ADR-001 Stage 3 MC reference](https://github.com/pd241008)
- [ADR-002 Clark MAX](https://github.com/pd241008)
- [ADR-003 Analytical SSTA](https://github.com/pd241008)
- [ADR-004 Monorepo structure](https://github.com/pd241008)
- [ADR-005 Stage 5 pooled reporting](https://github.com/pd241008)
- [ADR-006 Stage 6A training-data generation](https://github.com/pd241008)
- [ADR-007 Stage 6B vanilla DAG-GNN baseline](https://github.com/pd241008)
- [ADR-008 Stage 6C physics-feature injection](https://github.com/pd241008)
- [ADR-009 Stage 7 split conformal calibration](https://github.com/pd241008) (newest — calibrated uncertainty)

### OrbitLite — memory-aware adaptive training
Canonical: `Legacy/OrbitLite/docs/adrs/`
- [001 Closed-loop AIMD over static profiling](https://github.com/pd241008)
- [002 Host-RAM-first, VRAM-second](https://github.com/pd241008)
- [003 Synthetic scenes before real EO data](https://github.com/pd241008)

### AI Agent (OmniTrace) — autonomous polyglot agent
Canonical: `Legacy/AI Agent/docs/adr/`
- [0001 Use BLAKE3 for CAS addressing](https://github.com/pd241008)

### Design_Doc-original (no source file — kept inline)
- [`examples/cqrs-append-only-event-store.md`](./examples/cqrs-append-only-event-store.md) — DevTrace CQRS (ADR-002)
- [`examples/dual-auth-oauth-session.md`](./examples/dual-auth-oauth-session.md) — Milan dual auth (ADR-003)
- [`examples/migration-removing-dependency.md`](./examples/migration-removing-dependency.md) — OmniStat no-Redis (ADR-001)

---

## ⚖️ Why Not Duplicate Every ADR Here?

The playbook's job is to teach the **decision pattern**, not to host every
project's full decision history (which lives and evolves in its repo). Keeping
one template + six curated examples + a routable catalog:
- avoids stale duplicate bodies drifting out of sync with the source,
- keeps the playbook readable (29 → 1 template + 6 examples + links),
- still surface the ADR-009 count so nothing is silently missing.

_See the [main README](../../README.md) for the full navigation._
