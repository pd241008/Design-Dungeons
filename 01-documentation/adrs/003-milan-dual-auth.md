# 📜 ADR-003: Dual Auth (Google OAuth + OTP) for Milan

> **Status:** `Decided`
> **Date:** `March, 2026`

---

## 🌎 Context
Milan is an event-driven ticketing platform that handles extreme load spikes during festival ticket drops, having successfully sold over 5,400 passes. 

For authentication, we needed a system that is incredibly low-friction to onboard users quickly during a high-stress drop, but also highly secure to prevent ticket scalping and account takeovers. A single authentication method presented tradeoffs:
- **Only OAuth:** Great for speed, but limits users who don't want to use Google or don't have it as their primary festival email.
- **Only OTP:** Slower flow, reliant on email/SMS delivery speeds which can lag under extreme load.

## 🛤️ Options Considered
1. **Single SSO Provider (Google-only):** Fastest implementation, but alienates a subset of users and relies completely on Google's uptime.
2. **Standard Email/Password:** Too much friction during high-stress ticket drops. Forgotten passwords lead to dropped conversions.
3. **Dual Auth (Google OAuth + Custom OTP):** Support one-click OAuth for speed, with a fallback custom OTP system for flexibility and resilience.

---

## 🎯 Decision
> [!IMPORTANT]  
> **We will implement a Dual Authentication system: Google OAuth as the primary path, backed by a custom OTP flow.**

## 🧠 Reasoning
In a ticketing platform, conversion speed is everything. Google OAuth allows users to authenticate in a single click, which is absolutely crucial when tickets sell out in minutes. 

However, relying *solely* on Google OAuth is a single point of failure and creates friction for users purchasing tickets on behalf of others (or using non-Google emails). By building a custom OTP system alongside it, we ensure that anyone can access the platform without ever maintaining a password database. 

During a high-traffic drop, if the OTP email provider lags, users can instantly switch to OAuth. If OAuth has an issue, OTP is the fallback. 

## ⚖️ Consequences
- **Good:** 🟢 Maximum conversion rate. Users never get stuck on a "forgot password" screen during a ticket drop. High resilience against auth provider outages.
- **Bad:** 🔴 Increased implementation complexity. We have to maintain two distinct authentication flows, handle edge cases where a user switches between methods, and manage OTP token expiration securely.
