# 🤯 Negative-Result Engineering

> Reusable discipline for running and reporting an empirical autopsy of a
> system that was designed to do something, and rigorously shown **not** to
> work.

> [!IMPORTANT]
> **A rigorous, honest negative result is worth more to the field than a
> fabricated positive one.** The discipline below comes from a full empirical
> autopsy of a mechanism that had proven theoretical bounds yet failed in
> practice — and the write-up isolates *exactly why*. The names don't matter;
> the structure of the autopsy does.

A classic shape: a defense that theoretically guarantees the failure of its
attacker (a provable correlation-decay bound, a bounded query-complexity
argument) still ships a worse success rate than the undefended baseline. The
value is in how the negative result is dissected:

```
Attack Success Rate:
  Undefended          10.5%   (baseline)
  Static defense      10.5%   (no improvement — stateless bound)
  Naive dynamic       22.9%   (defense acts as adversarial perturbation!)
  Adaptive attack     53.6%   (fully bypassed via straight-through gradient)
```

---

## The Root-Cause Discipline

Isolated through rigorous ablation, not hand-waving:

1. **Input/basis coupling** — the defense's steering parameters are
   input-dependent; the attacker gets perfect analytic gradients through them.
2. **Gradient masking** — the defense's magnitude creates a gradient penalty
   that *masks* the vulnerability rather than removing it.
3. **The control experiment proves it** — with the coupling removed, the
   protective effect drops to statistical zero (ASR ≈ baseline, `p ≈ 0.84`).

---

## The Honest Numbers

- **Latency:** target `1ms`; actual `7ms` mean (p50 6.7 / p95 12.1 / p99 20.8) —
  an order-of-magnitude miss under load.
- **False positives:** mechanism trigger `1.2%`, FP `0.0%` — the mechanism adds
  no benign-traffic overhead, which *doesn't rescue it*.

> [!WARNING]
> **Provable gradient degradation ≠ empirical robustness.** Theoretical bounds
> can hold mathematically while the system still fails. Never let a proof
> substitute for measuring actual outcomes against a fully adaptive (e.g.
> BPDA-style) adversary. Report the adaptive result, not just the naive one.

---

## Why Negative Results Belong in the Playbook

A cautionary result teaches what *not* to do: **dynamic constraints that depend
deterministically on the attacker's trajectory do not hide gradients.** A
steering mechanism that perturbs samples toward adversarial regions, plus a
fully adaptive attacker, yields *higher* success with *fewer* queries. This is
the canonical counter-example to the "moving target" intuition and directly
strengthens the honesty-first policy and the independent verification gate in
[`research-patterns.md`](./research-patterns.md) §1.

> [!TIP]
> When you ship a negative result, treat it like a first-class artifact:
> report every measured number (including latency and false positives), keep the
> superseded designs under `diagnostics/`, and publish the control experiment
> that isolates the root cause. That is what turns a "this failed" into a
> transferable lesson.