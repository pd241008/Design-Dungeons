# 📖 README Patterns

> **The `README.md` is the front door of your project.** It must answer _what this is, why it exists, and how to run it_ within the first scroll.

---

## 🎯 The Project Abstract

Do not write a tech stack list masked as an abstract.

> [!WARNING]  
> **Bad:** _"This is a Next.js and Prisma application that uses PostgreSQL to store user data."_

> [!IMPORTANT]  
> **Good:** _"Milan is an event-driven platform that sold 5,400+ passes, built to provide real-time coordination with dual authentication. (Next.js / PostgreSQL)."_

Focus on the _domain problem_ being solved, then list the tech.

---

## 🏷️ Badge Conventions

Badges signal the health and state of a project at a glance:

- **`[Status: Live]`** 🟢 Production-ready, currently serving traffic.
- **`[Status: Building]`** 🟡 Active development, APIs will change.
- **`[Status: Research]`** 🟣 Experimental/ML repo, expect messy notebooks, optimized for reproducibility rather than scale.

---

## 🤝 CONTRIBUTING.md vs. Inline

> [!NOTE]
>
> - If a project is open-source or has **>3 active maintainers**, write a dedicated `CONTRIBUTING.md`.
> - If the project is **internal or solo**, put a brief "Local Setup" section directly in the README and don't bother with a separate file.

---

## 🏗️ Variant Templates

> [!TIP]
> **Four variants below, one per project archetype.**

### 1️⃣ Variant A — Systems / Backend (Rust · Go · C++)

_Use for: DevTrace, NEURO, Aegis, OmniStat-Core, TASCP, SystemsLab projects_
_Focus heavily on architecture diagrams, environment variables, and dependency requirements._

**[View Backend Template](./backend-template.md)**

---

### 2️⃣ Variant B — Machine Learning / Research (Python)

_Use for: AdvGuard, IntelliDoc, AQI Prediction_
_Focus on reproducibility. Include exactly how to get the dataset, the exact conda environment setup, and the evaluation script._

**[View ML Template](./ml-template.md)**

---

### 3️⃣ Variant C — CLI Tools / Published Package

_Use for: ExpressKit, NeoUI, DevTrace (multi-platform)_
_Focus on usage examples. Provide a `--help` snippet and 3-4 concrete examples of what the tool actually does._

**[View CLI Template](./cli-template.md)**

---

### 4️⃣ Variant D — Full-Stack App / Platform

_Use for: Milan, Taskiee, Gram Sevak, NeuroTrack_
_Focus on the local dev loop. Where is the frontend? Where is the backend? Give them the one-liner (`npm run dev:all`) that starts everything._

**[View Fullstack Template](./fullstack-template.md)**

---

## 💡 Common Rules Across All Variants

> [!IMPORTANT]
>
> 1. **First sentence = what it does, not what it's built with.** Tech stack goes in a table, not the headline.
> 2. **Lead with the non-obvious thing.** If you solved a hard problem, say so in the first 5 lines.
> 3. **Quickstart before architecture.** Let someone run it first, understand it second.
> 4. **Status table > vague "in progress" text.** Recruiters and collaborators can scan a table in 3 seconds.
> 5. **Keep the footer consistent** — your handle + portfolio link on every repo.
