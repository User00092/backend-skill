# Ask what breaks first when designing a system

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-04 — https://www.instagram.com/reel/DeFTopCBAjy/ — EP01 of *How Senior Engineers Actually Think About System Design*.

### Video-derived skills

- The scenario uses a limited-time sneaker drop with approximately **50,000 users** trying to buy a product at the same time.
- A junior design is illustrated with a server and database; additional options shown include cache, queue, replica, and load balancer.
- The central question presented is **“What breaks FIRST?”** rather than simply **“What should I add?”**
- The video depicts queue backlog and an early-morning alert as failure examples; it does not establish measured capacity limits for a specific technology stack.

### Practical guidelines (synthesis)

1. Specify traffic shape, reads/writes, consistency, latency, and availability goals.
2. Map the critical path: ingress → app → cache/queue → database → outside services.
3. Calculate likely bottlenecks (connection limits, write contention, hot keys, queue consumption, inventory races).
4. Simulate spike/failure scenarios and establish backpressure, admission control, safe retries, and graceful degradation before adding infrastructure.
5. Instrument the path; optimize the bottleneck you actually measure.
