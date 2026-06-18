# 📖 README Patterns

> **The `README.md` is the front door of your project.** It must answer *what this is, why it exists, and how to run it* within the first scroll.

---

## 🎯 The Project Abstract

Do not write a tech stack list masked as an abstract. 

> [!WARNING]  
> **Bad:** *"This is a Next.js and Prisma application that uses PostgreSQL to store user data."*

> [!IMPORTANT]  
> **Good:** *"Milan is an event-driven platform handling 5400+ passes sold, built to provide real-time coordination with dual authentication. (Next.js / PostgreSQL)."*

Focus on the *domain problem* being solved, then list the tech.

---

## 🏷️ Badge Conventions

Badges signal the health and state of a project at a glance:

- **`[Status: Live]`** 🟢 Production-ready, currently serving traffic.
- **`[Status: Building]`** 🟡 Active development, APIs will change.
- **`[Status: Research]`** 🟣 Experimental/ML repo, expect messy notebooks, optimized for reproducibility rather than scale.

---

## 🤝 CONTRIBUTING.md vs. Inline

> [!NOTE]
> - If a project is open-source or has **>3 active maintainers**, write a dedicated `CONTRIBUTING.md`.
> - If the project is **internal or solo**, put a brief "Local Setup" section directly in the README and don't bother with a separate file.

---

## 🏗️ Variant Templates

> [!TIP]
> **Four variants below, one per project archetype.** Copy the relevant one into your repo, fill in the bracketed fields, and delete unused sections.

### 1️⃣ Variant A — Systems / Backend (Rust · Go · C++)
*Use for: DevTrace, NEURO, Aegis, OmniStat-Core, TASCP, SystemsLab projects*
*Focus heavily on architecture diagrams, environment variables, and dependency requirements.*

```markdown
<div align="center">

# [Project Name]

**[One-line description — what it does, not what it is made of]**

[![Rust](https://img.shields.io/badge/rust-black?style=flat-square&logo=rust)](https://www.rust-lang.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-[active|building|experimental]-orange?style=flat-square)]()

</div>

---

## What This Is

[2–3 sentences. What problem does this solve? Who would use it? What is the core architectural decision that makes it interesting?]

> **Design constraint:** [One sentence on the hardest technical tradeoff — e.g. "Zero-copy ingestion over Tokio MPSC channels with a bounded 10k-event buffer."]

---

## Architecture

```text
[ASCII diagram or short description of the pipeline / data flow]

e.g.
Ingestion Layer (Tokio MPSC)
    └── Event Store (SQLite / append-only)
        └── Query Engine (CQRS read path)
            └── Exporter (JSON / OpenTelemetry)
```
---
... (Standard README sections like Quickstart, Configuration, Performance, Status) ...
```

---

### 2️⃣ Variant B — Machine Learning / Research (Python)
*Use for: AdvGuard, IntelliDoc, AQI Prediction*
*Focus on reproducibility. Include exactly how to get the dataset, the exact conda environment setup, and the evaluation script.*

```markdown
<div align="center">

# [Project Name]

**[One-line research statement — what the system defends against / learns from / predicts]**

[![Python](https://img.shields.io/badge/python-3670A0?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Status](https://img.shields.io/badge/status-[research|production]-green?style=flat-square)]()

</div>

---

## Abstract

[3–5 sentences. What is the problem, what is your approach, what do the results show. Write this like an abstract you'd submit — it doubles as the README summary and recruiters can parse it fast.]

---

## Results

| Attack / Metric | Baseline | This Work | Notes |
|---|---|---|---|
| PGD (ε=0.15) | [X%] | **[Y%]** | [Config] |
| FGSM | [X%] | **[Y%]** | |
| Clean accuracy | [X%] | [X%] | No degradation |

---
... (Standard research sections like Algorithm, Datasets, Quickstart, Citation) ...
```

---

### 3️⃣ Variant C — CLI Tools / Published Package
*Use for: ExpressKit, NeoUI, DevTrace (multi-platform)*
*Focus on usage examples. Provide a `--help` snippet and 3-4 concrete examples of what the tool actually does.*

```markdown
<div align="center">

# [package-name]

**[What you get when you install this — one sentence, user-facing]**

[![npm](https://img.shields.io/npm/v/@pd241008/[package]?style=flat-square&logo=npm)](https://npmjs.com/package/@pd241008/[package])
[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

</div>

---

## Usage

```bash
# Most common command — show this first
[binary] [verb] [target]

# Example
expresskit init my-api --template rest
```

### Options

```text
[binary] [verb] --help

  --option-a    [what it does]   (default: X)
  --option-b    [what it does]
  --flag        [what it does]
```
---
... (Standard package sections like Install, What It Does, Generated Structure) ...
```

---

### 4️⃣ Variant D — Full-Stack App / Platform
*Use for: Milan, Taskiee, Gram Sevak, NeuroTrack*
*Focus on the local dev loop. Where is the frontend? Where is the backend? Give them the one-liner (`npm run dev:all`) that starts everything.*

```markdown
<div align="center">

# [Project Name]

**[What the product does for a user — one sentence]**

[![Live](https://img.shields.io/badge/live-[domain]-brightgreen?style=flat-square)](https://[domain])
[![Next.js](https://img.shields.io/badge/Next.js-black?style=flat-square&logo=next.js)](https://nextjs.org)

</div>

---

## What This Is

[2–3 sentences. Who uses it, what core action do they perform, what makes the implementation non-trivial.]

> **Production stat (if applicable):** [e.g. "5,400+ tickets sold across Milan '25 & '26 · 99.9% uptime · ₹0 payment failures"]

---

## Stack

| Layer | Technology | Why |
|---|---|---|
| Frontend | Next.js 14 + Tailwind | [reason] |
| Backend | Node / Express | [reason] |
| Auth | Google OAuth + OTP | [reason] |
| DB | PostgreSQL + Prisma | [reason] |
| Infra | AWS EC2 + PM2 | [reason] |

---
... (Standard app sections like Local Setup, Env Variables, Architecture Notes) ...
```

---

## 💡 Common Rules Across All Variants

> [!IMPORTANT]
> 1. **First sentence = what it does, not what it's built with.** Tech stack goes in a table, not the headline.
> 2. **Lead with the non-obvious thing.** If you solved a hard problem, say so in the first 5 lines.
> 3. **Quickstart before architecture.** Let someone run it first, understand it second.
> 4. **Status table > vague "in progress" text.** Recruiters and collaborators can scan a table in 3 seconds.
> 5. **Keep the footer consistent** — your handle + portfolio link on every repo.