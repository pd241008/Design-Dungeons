# 🎁 Reproducibility Artifact Design

> **Source:** Research-submission artifact (adversarial ML, mixed-norm
> intrusion-detection robustness) + its companion position paper,
> *"The Artifact as Interface: Toward a Reviewer-Centric Approach to Scientific
> Artifact Design."*

> [!IMPORTANT]
> **A research artifact is an interface between a scientific claim and its
> verification — not a container for code that happens to reproduce a result.**
> Design it for a time-constrained, skeptical reviewer, the same way you would
> design any interface for a user under pressure.

Existing artifact-evaluation guidance (USENIX Security, NSDI, ACM badging) says
an artifact should be *available, functional, reproducible* — an outcome, not a
design. This page goes one step further: it designs the *interaction* so a
reviewer can actually find and check a claim without reverse-engineering the
repo. It rests on six principles, each stated as a problem it addresses rather
than a prescription.

---

## The Six Principles

| # | Principle | Problem it addresses |
| :- | :--- | :--- |
| **P1** | **Claims are navigable** | A reviewer should not have to reverse-engineer code to find the evidence for a specific claim. A single stable *claim map* maps each paper claim → script → manifest → output file. |
| **P2** | **Verification is cheaper than reproduction** | Not every check needs a full pipeline rerun. Provide a fast, hash-verified verification path separate from a slow "reproduce everything" path. |
| **P3** | **Historical ≠ canonical methodology** | Research accumulates superseded evaluators and abandoned diagnostics. Don't make a reviewer guess which version produced the reported numbers. |
| **P4** | **Reproducibility claims are typed** | "Reproducible" collapses several guarantees. Label each result's level exactly / deterministic / statistical / archival (R1–R4). |
| **P5** | **Limitations are discoverable before evaluation** | Let a reviewer find a known boundary in 2 minutes up front, not mid-verification by accident. |
| **P6** | **Artifacts accommodate different time budgets** | A 15-minute sanity check and a 2-hour full reproduction are both legitimate; support both explicitly. |

---

## Typed Reproducibility (P4 — the R1–R4 scale)

Labeling every result with its guarantee level prevents the worst failure mode:
a reviewer expecting exact reproduction receiving a statistical result and
reading it as a defect.

| Level | Meaning | Verification method |
| :--- | :--- | :--- |
| **R1** | Exact byte-identical | Hash comparison against archived output |
| **R2** | Deterministic numerical | Exact numerical comparison; formatting may vary |
| **R3** | Statistical / tolerance-based | Comparison script with a stated, pre-registered tolerance |
| **R4** | Archival verification | Re-derive summary statistics from archived canonical results |

> [!WARNING]
> **For anything R3, state the tolerance in advance — before validation — and
> give it a name.** In the case-study artifact, the EXH K=1 sweeps got
> `k0/k1_survivors ±0.75% relative` and `k1_pct ±0.5 pp absolute`, pre-registered
> in the comparator script. Pre-registering tolerances converts a fuzzy "close
> enough" into a pass/fail check.

---

## The Claim Map (P1)

One row per scientific claim: a stable claim ID (C1, C2, …), a plain-language
statement, the manifest that specifies it, the generating script, and the output
file that holds the evidence. Also record dependencies so a reviewer checking a
downstream claim can trace back to the observation that motivated it.

```markdown
| ID | Paper Claim | Manifest | Evidence | Script | Output |
|----|-------------|----------|----------|--------|--------|
| C1 | Legacy one-shot evaluation is unreliable | E-C1-...json | Faithful run gives 16% / 40% vs retracted 29% | canonical/section3_faithful_diagnostic.py | results/section3/...json |
| C3 | Canonical exhaustive K=1 is accurate | E-C3-...json | Multi-seed EXH K=1 sweeps, 3 datasets | canonical/eval_deepfool_k1.py | results/foolbox/exh_k1_*.json |
```

```
C1 (faithful diagnostic)
    └── motivates C2 (exhaustive enumeration)
        └── enables C3 (accurate measurement)
            ├── validated by C4 (JSMA divergence)
            ├── validated by C6 (independent attacks)
            └── scaled by C8 (tractability)
```

---

## Canonical vs. Diagnostics Separation (P3)

```
canonical/      # Code that produced the numbers actually reported in the paper
diagnostics/    # Superseded / exploratory code kept for provenance
docs/adr/       # Lightweight ADRs recording the rationale for the split
```

Keep the historical analyzer (e.g. `eval_unified.py`) under `diagnostics/` so a
reviewer never has to guess which evaluator is authoritative. Document the
separation as a short ADR sequence.

---

## The Reviewer Documentation Suite

A top-level set of small markdown files, each with one job:

| File | Job |
| :--- | :--- |
| `REVIEWER_GUIDE.md` | "If you're reviewing X, start here" quick-navigation table + explicit review paths (15-min / 30-min / 2-hr / provenance) |
| `CLAIM_MAP.md` | P1: claim → evidence navigation |
| `REVIEW_CHECKLIST.md` | Bounded verification checklist, one subsection per claim |
| `VERIFY.md` | P2: fast verification without full reproduction (`make smoke`, ~2 min) |
| `REPRODUCE.md` | P2: full reproduction instructions |
| `REPRODUCIBILITY_LEVELS.md` | P4: the R1–R4 classification |
| `LIMITATIONS.md` | P5: boundaries discoverable up front |
| `PROVENANCE.md` | P3: methodological history and superseded code |

> [!TIP]
> Ship a `Makefile` with named targets (`make verify`, `make smoke`,
> `make reproduce-table3`). Fresh outputs go to `*.fresh.*` / `results_fresh/`
> paths and never overwrite the archive; `make clean` removes them. This keeps
> shipped results authoritative while still proving they regenerate.

---

## What Not to Over-Document

> [!WARNING]
> **Be candid about what you don't know.** The case-study authors admit there is
> no evidence yet that this scaffolding helps, and that a reviewer might open the
> claim map once and never return. For a paper with 2–3 simple claims, this suite
> is overkill — a Docker image + README is the more honest artifact. Match the
> documentation to the number and difficulty of the claims.

---

_Tied to the honesty-first reproducibility discipline in
[`research-patterns.md`](./research-patterns.md) §1 and the negative-result
discipline in [`negative-results.md`](./negative-results.md)._
