# Variant D — Full-Stack App / Platform

_Use for: Milan, Taskiee, Gram Sevak, NeuroTrack_  
_Focus on the local dev loop. Where is the frontend? Where is the backend? Give them the one-liner (`npm run dev:all`) that starts everything._

---

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

| Layer    | Technology            | Why      |
| -------- | --------------------- | -------- |
| Frontend | Next.js 14 + Tailwind | [reason] |
| Backend  | Node / Express        | [reason] |
| Auth     | Google OAuth + OTP    | [reason] |
| DB       | PostgreSQL + Prisma   | [reason] |
| Infra    | AWS EC2 + PM2         | [reason] |

---

## Local Setup

```bash
git clone https://github.com/pd241008/[repo]
cd [repo]
cp .env.example .env   # fill in secrets

npm install
npm run dev            # or docker-compose up
```

---

## Architecture Notes

[1–3 bullet points on non-obvious decisions: why async queue, why dual auth, why PM2 over container orchestration at this scale, etc.]

---

_[pd241008](https://github.com/pd241008) · [ct-os-dev-portfolio.vercel.app](https://ct-os-dev-portfolio.vercel.app)_
