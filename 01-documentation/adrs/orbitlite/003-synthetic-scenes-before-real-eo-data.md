# ADR 003: Synthetic Scenes Before Real EO Data

## Status
Accepted (2026-08-22)

## Context
Real benchmarks (LCC segmentation, EuroSAT) require torch, rasterio and
multi-gigabyte downloads. The deadline is 2026-09-20 and the development
machines include the exact low-RAM hosts the project claims to serve.
Blocking all controller/scoring work on dataset plumbing would leave the
paper's core mechanisms untested until the final week.

## Decision
Build the testing environment on **synthetic scenes**: a seeded NumPy
composition of ocean (low entropy), textured land (high entropy) and a bright
cloud patch. All tests, the benchmark harness, and the entropy-skip
calibration run against these scenes hermetically. Real loaders plug into the
same `WindowReader` interface later; the torch adapter wraps `StepFn`.

Entropy uses a fixed value range (0–255), not per-tile min/max normalization:
per-tile ranges erase exactly the ocean-vs-land contrast the skipper exists
to detect. This was caught by tests during skeleton bring-up (2026-08-22).

## Consequences
**Positive:**
- Deterministic, fast (<1 s) CI on any hardware including Tier T1.
- Controller behavior is verifiable independently of dataset quirks.
- Failure injection (KillError thresholds) can be calibrated precisely.

**Negative:**
- Synthetic results say nothing about real raster I/O bottlenecks or true
  cloud-mask fidelity; the paper's Tables 2–3 cannot be filled from this
  harness alone.
- Risk of over-fitting τ_e defaults to synthetic distributions; real-scene
  recalibration is mandatory before submission.
