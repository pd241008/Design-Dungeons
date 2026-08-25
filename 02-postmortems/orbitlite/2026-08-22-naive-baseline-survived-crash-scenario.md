# Postmortem: Naive Baseline Survived Its Own Crash Scenario

**Date:** 2026-08-22
**Status:** Resolved

## Incident Summary
During skeleton bring-up, the first full run of `make bench` showed the naive
fixed-batch baseline completing all 200 steps with zero kills — directly
contradicting the paper's central premise that oversized fixed configurations
get OOM-killed. The demo table read `naive: completed=200, kills=0`,
undermining the crash-rate story the entire evaluation depends on.

## Root Cause
1. **Failure-injection threshold set too generously.** The modeled allocator
   raises `KillError` only when projected bytes exceed `budget × 2`. With a
   256 MB budget the kill line sat at 512 MB, while the naive config's
   projection was `2048 × 64 × 64 × 8 ≈ 67 MB` — comfortably survivable.
2. **Scenario calibrated by intuition, not assertion.** No test pinned the
   invariant "naive must fail"; the crash story lived only in prose
   expectations, so nothing failed when the fixture stopped crashing.

## Fix
Raised the naive baseline's fixed batch to 32,768 (projection ≈ 1 GB > kill
line) so it enters the modeled SIGKILL regime deterministically
(`kills=200, completed=0`). Bench output now matches the paper narrative:
naive dies, tuned-static survives, ours adapts and completes.

## Action Items
- [x] Calibrate failure injection so each baseline sits on the intended side
      of the kill line.
- [ ] Add a bench-level regression test asserting `naive.kills > 0` whenever
      the harness changes (prevents silent re-drift).
- [ ] When real EO data replaces synthetic scenes, re-derive kill thresholds
      from measured per-step allocations rather than byte math guesses.
