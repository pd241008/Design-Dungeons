# 🤯 Negative-Result Engineering

> **Source:** Project Archimedes — TASCP (Trajectory-Aware Spiral Constraint
> Projection), a stateful moving-target adversarial defense rigorously shown
> **not** to work.

> [!IMPORTANT]
> **A rigorous, honest negative result is worth more to the field than a
> fabricated positive one.** Project Archimedes is a full empirical autopsy:
> a two-loop stateful Givens-rotation defense with proven theoretical bounds that
> nevertheless fails in practice — and the write-up isolates *exactly why*.

TASCP tracks the spatial momentum of a query trajectory and applies a
momentum-conditioned Givens rotation to the constraint manifold. The theoretical
results hold: **Lemma 2** proves a rotation by `Δθ` reduces gradient correlation
to exactly `cos(Δθ)`, and **Proposition 1** bounds query complexity. Yet the
defense fails. The value is in how the negative result is dissected:

```
Attack Success Rate (2,000 samples, max 20 queries):
  Undefended          10.50%  (baseline)
  DACM (static)       10.50%  (no improvement — stateless bound)
  TASCP (naive)       22.95%  (rotation acts as adversarial perturbation!)
  TASCP (adaptive)    53.60%  (fully bypassed via straight-through BPDA)
```

---

## The Root-Cause Discipline

Isolated through rigorous ablation, not hand-waving:

1. **Basis coupling** — the rotation basis vectors are data-dependent; the
   attacker gets perfect analytic gradients through them.
2. **Gradient masking** — the rotation magnitude creates a gradient penalty that
   *masks* the vulnerability rather than removing it.
3. **The control experiment proves it** — with basis coupling removed, the
   protective effect drops to statistical zero (10.30% ASR vs 10.50% baseline,
   `p=0.836`).

---

## The Honest Numbers

- **Latency:** target `<1ms`; actual `7.08ms` mean (p50 6.69 / p95 12.09 / p99
  20.79) — an order-of-magnitude miss under load.
- **False positives:** rotation trigger `1.21%`, FP `0.00%` — the mechanism adds
  no benign-traffic overhead, which *doesn't rescue it*.

> [!WARNING]
> **Provable gradient degradation ≠ empirical robustness.** Theoretical bounds
> held mathematically, but the defense still failed. Never let a proof substitute
> for measuring actual attack success against a fully adaptive (BPDA-style)
> adversary. Report the adaptive result, not just the naive one.

---

## Why Negative Results Belong in the Playbook

A cautionary result teaches what *not* to do: **dynamic constraint manifolds
that depend deterministically on the attacker's trajectory do not mask
gradients.** A stateful rotation that perturbs samples toward adversarial
regions, plus a fully adaptive attacker, yields *higher* success with *fewer*
queries. This is the canonical counter-example to the "moving target" intuition
and directly strengthens the honesty-first policy and the independent
verification gate in [`research-patterns.md`](./research-patterns.md) §1.

> [!TIP]
> When you ship a negative result, treat it like a first-class artifact:
> report every measured number (including latency and false positives), keep the
> superseded designs under `diagnostics/`, and publish the control experiment
> that isolates the root cause. That is what turns a "this failed" into a
> transferable lesson.