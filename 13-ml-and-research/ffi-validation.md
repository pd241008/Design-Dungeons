# 🔀 Cross-Language FFI Validation

> **Source:** Midas (EDGE) — budget-gated Givens-rotation adversarial defense on
> ARM, where the trained PyTorch model must run inside a Rust engine via the
> TensorFlow Lite C API.

> [!TIP]
> When a production engine (Rust/C++/TFLite) replaces a Python-trained model,
> prove the port is faithful with **reference vectors** — run the same inputs
> through every implementation and record the max discrepancy.

Midas validates its Rust FFI bindings to the TensorFlow Lite C API against three
implementations (PyTorch shadow, Python interpreter, Rust FFI):

```
max |Δp| = 1.9e-6   (float32 / dynamic-int8), zero label flips
max |Δp| = 1.9e-3   (fp16), zero label flips
```

---

## The Validation Workflow

```
shadow/gen_ffi_vectors.py          → reference vectors from PyTorch
cargo run -p edge --bin tflite_validate --features tflite \
    models/classifier_fp16.tflite  \
    results/ffi_vectors.json \
    results/ffi_validation.json
```

The binary is gated behind a `tflite` cargo feature so a default build still
compiles without vendor'd `.so` files. Output is a machine-readable
`results/ffi_validation.json` artifact plus a `results/latency_reproducibility.json`
record.

---

## Distribution vs. SLA Validation

> [!NOTE]
> **Watch for the p50/p99 swap across platforms.** Midas found the median latency
> is governed by FP arithmetic (slower on the Pi than x86_64), while the tail
> (p99/max) reflects warmup + noise (lower and tighter on the Pi). A naive
> "compare single median" check would mislead; validate both the distribution
> shape and the edge SLA (50 ms) separately per target.

Validation checks three things on each deployment target:
1. **Numerical fidelity** — max `|Δp|` against reference vectors.
2. **Distribution shape** — p50/p99 reported separately, not a single median.
3. **SLA headroom** — latency far inside the 50 ms edge SLA on both platforms.

---

_Tied to the shadow-model / prototype-vs-production separation in
[`research-patterns.md`](./research-patterns.md) §3._