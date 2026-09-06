# 🧬 ML & Research

> **Source Projects:** ChaosSeal (reproducibility discipline), PrimeVector (LSH research), DocsSense (RAG pipeline), Midas (adversarial ML), Artifact A / TASCP (reproducibility artifact & negative-result design)
>
> Research code is inherently messy. The goal is to isolate the chaos from stable production layers while maintaining reproducibility, experiment tracking, and clear separation between prototype and production.

> [!TIP]
> **Focus docs in this section:**
> - [`artifact-design.md`](./artifact-design.md) — reviewer-centric reproducibility artifact design (claim maps, typed R1–R4 reproducibility, review paths)
> - [`negative-results.md`](./negative-results.md) — how to run and report an honest negative-result autopsy (TASCP)
> - [`ffi-validation.md`](./ffi-validation.md) — cross-language FFI validation for prototype→production model ports (Midas)

---

## 1️⃣ Reproducibility Discipline (ChaosSeal)

### The Honesty-First Policy

> [!IMPORTANT]
> **Every number in the paper must trace back to a real file produced by a real run.** No fabricated results, no hand-picked examples.

ChaosSeal implements this through a rigid traceability chain:

```
results/*.json ──► stats.py ──► figures/*.pdf
     ▲                │
     │                ▼
run.sh ◄─── Makefile ◄─── README instructions
```

### Implementation

```makefile
# Makefile
.PHONY: all clean run paper

all: run paper

run:
	./run.sh 2>&1 | tee results/run.log

paper: results/*.json
	python3 stats.py
	python3 figures.py
```

### Key Rules

| Rule | Implementation |
| :--- | :--- |
| **Fixed seeds** | All RNG seeds logged to `results/seed.log` |
| **Deterministic builds** | Rust tests enforce `cargo test` reproducibility |
| **JSON artifacts** | Every simulation writes exact same fields |
| **No floats in hot path** | ChaosSeal uses Q32.32 fixed-point arithmetic |

### The Independent Verification Gate (Claim-Level)

> [!IMPORTANT]
> **A headline number derived from a simulator is not a measurement until a
> second, independent implementation reproduces the model's *bounds* — not just
> its output.**

After a cross-check exposed that the pendulum's elastic coupling was unbounded
(energy escape ~18-127 s) and the fixed-point "long-horizon attractor" was a
±2^31 saturation artifact, ChaosSeal adopted a claim-level gate:

1. **Verify the model, then the rate.** Run an independent, first-principles
   script (`verify_metastability.py`) that checks bounds across multiple
   integrators (RK4 dt=1e-2/2e-3, symplectic Verlet, DOP853) *before* any
   Lyapunov/KS rate is committed.
2. **The float64 replicator is the referee.** Every Rust Q32.32 exponent is
   gated against an exactly-replicated float64 Benettin
   (`validate_benettin.py`, non-unit-inertia configs included so the Jacobian
   split is actually exercised).
3. **Tag finite windows honestly.** T=100 s data from an unbounded model is a
   *transient* statistic, permanently labeled as such, never extrapolated to
   the protocol epoch.
4. **Kill the headline, then redesign.** The 256-bit/epoch claim was reframed
   to a conditioner, the coupling was made bounded by construction (wrapped
   `atan2` spring, default c=1.0), and the claim was re-earned with measured
   KS ≈ 1.0-1.3 nats/s → 256-bit dt ≈ 136-176 s — still verifier-gated.

> [!WARNING]
> **Fixed-point ceilings lie in both directions.** Q32.32 guarantees
> determinism, not physical boundedness: a saturating i64 ceiling produces a
> bounded-looking trajectory whose converged spectrum is the exponent of the
> *saturated* system. Any long-horizon claim must first bound the
> discretization floor.

> [!WARNING]
> **Do not commit generated figures or data to version control.** Commit the scripts and seeds, then generate artifacts in CI. This prevents accidental "result drift" where old figures no longer match the code.

---

## 2️⃣ Experiment Tracking

### Minimal Tracking

For research projects, a lightweight tracking system beats a heavy ML platform.

```python
# experiment.py
import json
from datetime import datetime

def log_experiment(config: dict, metrics: dict, artifacts: list[str]):
    record = {
        "timestamp": datetime.utcnow().isoformat(),
        "config": config,
        "metrics": metrics,
        "artifacts": artifacts,
        "git_commit": subprocess.check_output(["git", "rev-parse", "HEAD"]).strip(),
    }
    with open(f"results/experiment_{timestamp}.json", "w") as f:
        json.dump(record, f, indent=2)
```

### What to Track

| Field | Purpose | Example |
| :--- | :--- | :--- |
| **config** | Reproduce the exact setup | `{"lr": 0.001, "batch_size": 32, "seed": 42}` |
| **metrics** | Quantitative results | `{"accuracy": 0.94, "latency_ms": 120}` |
| **artifacts** | File paths to model weights, figures | `["models/v2.pt", "figures/loss.pdf"]` |
| **git_commit** | Exact code version | `"b83d195..."` |

> [!TIP]
> Store experiments as JSON files in `results/`, not in a database. JSON is human-readable, git-friendly, and works offline.

---

## 3️⃣ Prototype vs. Production Separation (Midas)

### The Shadow Model Pattern

> [!TIP]
> Use a high-level prototype (Python) for rapid iteration, then port validated algorithms to a production engine (Rust/C++).

Midas implements this through a dual-language architecture:

| Layer | Language | Purpose |
| :--- | :--- | :--- |
| **Shadow Model** | Python | Rapid prototyping, gradient testing, diagnostic scripts |
| **Production Engine** | Rust | ARM Cortex-A76 deployment, real-time guarantees |

### Porting Checklist

When moving from prototype to production:

1. **Validate mathematically first.** Ensure the prototype produces correct outputs on known inputs.
2. **Write tests in the prototype.** The Python tests define the expected behavior.
3. **Port incrementally.** Move one function at a time, running Python tests against the Rust implementation.
4. **Match floating-point behavior.** Be aware of IEEE 754 vs. fixed-point differences.

---

## 4️⃣ RAG Pipeline Architecture (DocsSense)

### The Dual-Pipeline Pattern

> [!IMPORTANT]
> Separate the async ingestion pipeline from the low-latency query pipeline. These have fundamentally different SLAs and scaling profiles.

```mermaid
flowchart LR
    subgraph Ingestion["Async Ingestion Pipeline"]
        Upload["Upload"] --> Storage["S3/GCS"]
        Storage --> Celery["Celery Worker"]
        Celery --> Chunking["Chunking"]
        Chunking --> Embedding["Embeddings"]
        Embedding --> VectorDB[("Vector DB")]
    end
    
    subgraph Query["Low-Latency Query Pipeline"]
        Query["Query"] --> QEmbed["Embedding"]
        QEmbed --> VSearch["Vector Search"]
        VSearch --> Context["Context Assembly"]
        Context --> LLM["LLM"]
        LLM --> Answer["Answer"]
    end
    
    style Ingestion fill:#ff6b6b,stroke:#333,stroke-width:2px,color:#fff
    style Query fill:#4ecdc4,stroke:#333,stroke-width:2px,color:#fff
```

### Key Decisions

| Decision | Rationale |
| :--- | :--- |
| **Async ingestion** | Document processing (OCR, chunking) is CPU-heavy and slow |
| **Sync query** | Users expect answers in < 2 seconds |
| **Triple database** | MongoDB (auth), Vector DB (search), S3/GCS (storage) |
| **Pre-emptive mocking** | Monkey-patch external services before app load for fast tests |

---

## 5️⃣ Adversarial ML Patterns (Midas)

### Budget-Gated Fallback

> [!TIP]
> In real-time systems, if a defense mechanism exceeds its time budget, fall back to the undefended result rather than dropping the request.

```rust
struct DefenseResult {
    defended: Option<Vector>,
    fallback: Vector,
    budget_exceeded: bool,
}

fn defend_with_budget(input: Vector, budget_ms: u64) -> DefenseResult {
    let start = Instant::now();
    let defended = apply_givens_rotation(input);
    
    if start.elapsed().as_millis() > budget_ms {
        return DefenseResult {
            defended: None,
            fallback: input,  // Project onto unrotated manifold
            budget_exceeded: true,
        };
    }
    
    DefenseResult {
        defended: Some(defended),
        fallback: input,
        budget_exmitted: false,
    }
}
```

### Trait-Based Hardware Abstraction

```rust
trait InferenceModel {
    fn predict(&self, input: &[f32]) -> Result<Vec<f32>, InferenceError>;
    fn latency_budget(&self) -> Duration;
}

trait SensorSource {
    fn read(&mut self) -> Result<Vec<f32>, SensorError>;
}

// Swappable backends
struct TFLiteModel { /* ... */ }
struct OnnxModel { /* ... */ }
```

---

## 6️⃣ LSH & Vector Search (PrimeVector)

### Pipeline Architecture

```mermaid
flowchart LR
    Input["Byte Stream"] --> Padder["InputPadder"]
    Padder --> Encoder["ByteNgramEncoder"]
    Encoder --> Projector["LSHProjector"]
    Projector --> Normalizer["VectorNormalizer"]
    Normalizer --> Kafka["KafkaDispatcher"]
    Kafka --> Scala["Scala Analytics"]
    
    style Projector fill:#95e1d3,stroke:#333,stroke-width:2px,color:#333
```

### LSH Guarantees

PrimeVector satisfies formal (r1, r2, p1, p2)-sensitivity using seeded random projection onto Gaussian hyperplanes.

| Parameter | Meaning |
| :--- | :--- |
| **r1, r2** | Distance thresholds for near / far neighbor decisions |
| **p1** | Probability similar items hash to same bucket |
| **p2** | Probability distant items hash to same bucket |

### Protobuf Contracts

```protobuf
message VectorPayload {
  string source_id = 1;
  repeated float embedding = 2;
  int64 timestamp_ms = 3;
  string category = 4;
}
```

> [!NOTE]
> Protobuf enforces strict contracts between Go (producer) and Scala (consumer). Version the `.proto` file and never change field numbers.

---

## 7️⃣ Research Directory Structure

```
project-root/
├── notebooks/              # Jupyter notebooks for EDA and messy research
│   ├── 01_data_explo.ipynb
│   └── 02_model_test.ipynb
├── datasets/               # Raw and processed data
│   ├── raw/                # Immutable source data
│   └── processed/          # Cleaned, tokenized, or normalized data
├── models/                 # Serialized weights
│   ├── v1_baseline.pt
│   └── v2_optimized.onnx
├── results/                # JSON artifacts, figures, logs
│   ├── experiment_001.json
│   └── figures/
├── proto/                  # Protobuf definitions (if applicable)
├── paper/                  # LaTeX sources for publications
├── backend/                # Inference API
├── frontend/               # User interface
├── requirements.txt        # Strict dependency locking
└── README.md
```

---

## 8️⃣ Reference Implementations

| Repo | Pattern | Highlights |
| :--- | :--- | :--- |
| **ChaosSeal** | Reproducibility | Honesty-first policy, JSON artifacts, fixed seeds, Makefile |
| **PrimeVector** | LSH Research | Formal sensitivity guarantees, Protobuf contracts, Scala analytics |
| **DocsSense** | RAG Pipeline | Dual-pipeline architecture, 100% mocked tests, triple database |
| **Midas** | Adversarial ML | Budget-gated fallback, shadow model, trait-based hardware abstraction |
