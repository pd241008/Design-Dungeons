# ADR-001: Canonical Exhaustive Mixed-Norm Evaluation

> **Model example:** How to replace a flawed evaluation methodology with a
> canonical one and preserve the superseded implementation for provenance.
> Demonstrates the **methodology-replacement + canonical-vs-legacy** pattern.
> Source: BlackIce.

## Status

Accepted

## Context

The original AdvGuard evaluation used a one-shot gradient-snapped attack:
run continuous PGD, then snap categorical features to argmax once at the
end. This approach has two failure modes:

1. **Locally optimal categorical flips**: The argmax selects a categorical
   state that is locally optimal for the current continuous perturbation but
   may not be the worst-case state overall.
2. **Invalid intermediate states**: During continuous optimization, the
   categorical features may take invalid values (non-one-hot), which can
   produce misleading gradients.

The original AdvGuard paper reported 29.10% robust accuracy for the hardened
NSL-KDD model. This figure was later retracted.

## Decision

Replace one-shot gradient-snapped evaluation with **exhaustive enumeration**
of valid categorical states.

For categorical budget $K=1$:
- Evaluate K=0 (continuous-only attack)
- Evaluate every categorical group $G$ individually: flip all features in $G$
  to every valid one-hot state, run continuous PGD-40, pick worst outcome
- A sample survives K=1 only if it survives K=0 AND all group evaluations

For $K=2$, enumerate all pairs of groups.

## Consequences

### Positive
- Eliminates invalid-state artifacts
- Guarantees coverage of the discrete state space within budget $K$
- Produces deterministic discrete-state coverage for a fixed attack initialization; stochastic continuous optimization remains subject to random-start variance
- Faithful reproduction yields 40.36% robust accuracy (vs. 29.10%)

### Negative
- Runtime scales combinatorially with $K$ and number of categorical groups
- For UNSW-NB15 with 5 categorical groups, K=2 requires 667 states per sample
- Memory bandwidth becomes the bottleneck; GPU parallelism is essential for
  large datasets

### Neutral
- Continuous inner optimization remains PGD-based and does not provide a
  global certificate
- Random-start PGD introduces run-to-run variance; tolerances are
  pre-registered in `verification/compare_exh_fresh.py`

## Superseded Implementation

The legacy one-shot evaluator is preserved in `canonical/eval_unified.py`
and `diagnostics/train_pgd_robust.py` for provenance. It should not be used
for canonical claims.