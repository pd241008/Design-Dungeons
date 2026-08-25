# ADR 002: Host RAM Is the Managed Resource, VRAM Second

## Status
Accepted (2026-08-22)

## Context
Prior adaptive systems manage GPU memory because their failure signal (CUDA
OOM exception) lives there. Our contribution targets students and independent
researchers on ordinary laptops: 8–16 GB system RAM, weak or absent discrete
GPU. In that population the binding constraint is host RAM, and the fatal
signal is SIGKILL from the kernel OOM killer, not an allocator exception.
Designing primarily for VRAM would inherit the exact assumption this project
exists to remove.

## Decision
The controller's primary telemetry, headroom target `h*` and all descent
triggers operate on **system RAM** (`/proc/meminfo`, psutil). GPU memory is
treated as an optional secondary budget on tier T3 only, folded into the same
headroom abstraction when present. Framework baselines that require catchable
CUDA OOMs (Lightning BatchSizeFinder, Composer) are therefore evaluated
exclusively on T3, never on T1/T2 where they cannot run.

## Consequences
**Positive:**
- The evaluation axis (T1 → T3) becomes a differentiating feature of the
  paper rather than a limitation: no prior baseline covers T1/T2.
- `/proc`-based sampling needs no CUDA, keeping tests hermetic on CPU-only CI.

**Negative:**
- On T3 the single headroom abstraction must blend two allocators with very
  different reclaim semantics (cached blocks vs page cache), which may need a
  per-resource split later.
- Reviewers may ask for GPU-memory experiments we deliberately scope out;
  the paper must state the T3-only baseline policy explicitly.
