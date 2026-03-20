# Engineering Philosophy

These are the core values that drive how I build systems. They are not theoretical; they are hard-won lessons from production.

### Instrument Before You Optimize
Never guess where the bottleneck is. On *Milan*, identifying the ~60-70MB Node.js footprint and managing 4K concurrent users was only possible because we had the metrics. Optimization without measurement is just changing code in the dark. If you can't measure the performance impact of a PR, you shouldn't merge it.

### Diagrams Must Reflect Deployment Reality
An architecture diagram that shows ideal state instead of current state is a lie. Diagrams should map directly to deployed infrastructure. If a component is "planned for Q3," it doesn't exist yet. Draw what runs.

### Document the Decision, Not Just the Outcome
The final code only tells you *what* was built, not *why* the alternatives were rejected. This is why I write Architecture Decision Records (ADRs). Context decays faster than code. When a new engineer joins in six months, they need to know why we chose a specific path despite its obvious flaws.

### Pragmatism Over Premature Abstraction
Complex systems fail in complex ways. Don't introduce a distributed cache, an event bus, or a microservice boundary until the pain of *not* having it outweighs the operational cost of maintaining it. If a single PostgreSQL instance and in-memory state solve the problem today, don't deploy Redis just because it "scales better." Scale it when it breaks.
