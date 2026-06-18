# 🧠 Engineering Philosophy

> **Core Tenet**: These are the values that drive how systems are built. They are not theoretical; they are hard-won lessons from production scale (e.g. 5,400+ passes sold on [Milan](https://github.com/orgs/TechTeam-Official/repositories), distributed architectures in Project Omega 🔒).

---

## 1. Instrument Before You Optimize

Never guess where the bottleneck is. Optimization without measurement is just changing code in the dark.

> [!IMPORTANT]  
> **If you can't measure the performance impact of a PR, you shouldn't merge it.**

**Production Context ([Milan](https://github.com/orgs/TechTeam-Official/repositories) Case Study):**
Identifying the ~60-70MB Node.js footprint and successfully handling the load of 5,400+ passes sold during festival ticket drops was _only_ possible because we had active telemetry. We didn't guess the bottleneck; we measured it and patched it.

---

## 2. Diagrams Must Reflect Deployment Reality

An architecture diagram that shows an ideal future state instead of the current state is a lie. Diagrams should map directly to deployed infrastructure.

> [!WARNING]  
> **If a component is "planned for Q3," it doesn't exist yet. Draw what runs.**

> [!NOTE]  
> **Tooling Constraint:** All diagrams must use **Mermaid.js**. This prevents documentation from becoming an unmaintainable mix of Excalidraw screenshots, ASCII text, and Draw.io exports over time.

**Production Context ([IntelliDoc](https://github.com/pd241008/IntelliDoc-Query)):**
When diagramming the `IntelliDoc Query Engine`, the architecture reflects the _actual_ Redis queue operating in production, not the hypothetical Kafka stream we might want next year.

---

## 3. Document the Decision, Not Just the Outcome

The final code only tells you _what_ was built, not _why_ the alternatives were rejected. Context decays faster than code.

> [!TIP]  
> **This is why we rigorously write Architecture Decision Records (ADRs).**

**Production Context:**
When a new engineer joins in six months, they need to know _why_ we chose a specific, seemingly flawed path. By documenting the constraints (budget, time, scale) at the exact moment the decision was made, we prevent repetitive debates and reverse-engineering.

---

## 4. Pragmatism Over Premature Abstraction

Complex systems fail in complex ways. Don't introduce a distributed cache, an event bus, or a microservice boundary until the pain of _not_ having it outweighs the operational cost of maintaining it.

> [!NOTE]  
> **Scale it when it breaks, not before.**

**The Reality Check (OmniStat 🔒):**
If a single PostgreSQL instance and in-memory state solve the problem today, don't deploy Redis just because it "scales better." We actively dropped Redis from OmniStat-Core 🔒 because at sub-1,000 events/day, the operational overhead was just a tax on velocity.
