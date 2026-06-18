# 🧠 Engineering Philosophy

> **Core Tenet**: These are the values that drive how systems are built. They are not theoretical; they are hard-won lessons from production scale (e.g. 5,400+ passes sold on Milan, distributed architectures in Project Omega).

---

## 1. Instrument Before You Optimize

Never guess where the bottleneck is. Optimization without measurement is just changing code in the dark. 

> [!IMPORTANT]  
> **If you can't measure the performance impact of a PR, you shouldn't merge it.**

**The Reality Check (Milan Case Study):**
Identifying the ~60-70MB Node.js footprint and successfully handling the load of 5,400+ passes sold during festival ticket drops was *only* possible because we had active telemetry. We didn't guess the bottleneck; we measured it and patched it.

---

## 2. Diagrams Must Reflect Deployment Reality

An architecture diagram that shows an ideal future state instead of the current state is a lie. Diagrams should map directly to deployed infrastructure. 

> [!WARNING]  
> **If a component is "planned for Q3," it doesn't exist yet. Draw what runs.**

**The Reality Check (IntelliDoc):**
When diagramming the `IntelliDoc Query Engine`, the architecture reflects the *actual* Redis queue operating in production, not the hypothetical Kafka stream we might want next year.

---

## 3. Document the Decision, Not Just the Outcome

The final code only tells you *what* was built, not *why* the alternatives were rejected. Context decays faster than code. 

> [!TIP]  
> **This is why we rigorously write Architecture Decision Records (ADRs).** 

**The Reality Check:**
When a new engineer joins in six months, they need to know *why* we chose a specific, seemingly flawed path. By documenting the constraints (budget, time, scale) at the exact moment the decision was made, we prevent repetitive debates and reverse-engineering.

---

## 4. Pragmatism Over Premature Abstraction

Complex systems fail in complex ways. Don't introduce a distributed cache, an event bus, or a microservice boundary until the pain of *not* having it outweighs the operational cost of maintaining it. 

> [!NOTE]  
> **Scale it when it breaks, not before.**

**The Reality Check (OmniStat):**
If a single PostgreSQL instance and in-memory state solve the problem today, don't deploy Redis just because it "scales better." We actively dropped Redis from OmniStat-Core because at sub-1,000 events/day, the operational overhead was just a tax on velocity.
