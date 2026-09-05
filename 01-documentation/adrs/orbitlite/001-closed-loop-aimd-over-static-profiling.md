# ADR 001: Closed-Loop AIMD Control Over Static Profiling and Retry-on-OOM

## Status
Accepted (2026-08-22)

## Context
Existing adaptive-training systems (MosaicML Composer auto gradient
accumulation, PyTorch Lightning BatchSizeFinder, `toma`) all share one
assumption: memory over-allocation surfaces as a *catchable* exception from
the CUDA allocator, which the framework intercepts and retries with a smaller
configuration. Our target regime is different. On CPU-only laptops training
against system RAM, Linux's OOM killer sends SIGKILL: no exception is raised,
no stack is unwound, hours of unsaved optimizer state vanish.

Reacting to failures is impossible when failures are fatal, so the control
strategy must be *predictive*. Additionally, peak step memory drifts during a
run (cache growth, allocator fragmentation, variable window content), which
makes any offline-profiled static configuration intermittently fatal.

## Decision
Implement the training loop as a closed-loop controller in the style of TCP
congestion avoidance (Jacobson 1988; Chiu & Jain 1989):

- A telemetry layer samples headroom `h = M_free / M_tot` plus smoothed drift
  velocity every step.
- An AIMD policy adjusts batch size: additive increase (+α) only after W
  consecutive above-target steps; multiplicative decrease (÷2) on pressure;
  deadband δ suppresses noise-driven oscillation.
- Entropy-based window skipping acts as a fast-relief lever under pressure.
- Kill-and-resume is modeled explicitly (`KillError` → `on_kill_resume`
  re-entering at β·b) so recovery semantics are testable.

## Consequences
**Positive:**
- Runs complete under budgets where static configs die intermittently.
- The controller is unit-testable against scripted samplers — no real OOM
  needed to verify descent behavior.
- Clear differentiation from prior art for the paper: host-RAM-first,
  SIGKILL-safe, data-space levers.

**Negative:**
- Controller parameters (δ, ε_v, W) are new hyperparameters someone must
  sanity-check per workload class.
- Proactive control trades some throughput: headroom held at h* means the
  last ~25% of RAM is deliberately unused.
