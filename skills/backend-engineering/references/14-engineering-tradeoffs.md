# Judge system-design tradeoffs and failure modes

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-13 — https://www.instagram.com/reel/DecFoqVicbm/

### Lessons extracted
- Senior engineering work is not selecting every scaling pattern; it is deciding which failure is most consequential to the actual product and customers.
- The clip recaps previous fixes: load balancing, stateless servers, connection pooling, indexes, replicas, caching, stampede protection, queueing, idempotency, outbox, and sharding. Each solves a particular problem and adds operational or correctness costs.
- Determine what fails first: connection exhaustion, slow checkout, stale data, request latency, duplicate side effects, message loss, or excessive data volume.
- Decide if consistency, staleness, availability, or operational complexity is acceptable *for the specific workflow*. A cached price could be unacceptable at checkout even when cached catalog data is acceptable.

### Implementation guidance (synthesis)
1. Start with a failure scenario, measurable symptom, and customer impact.
2. Specify service objectives (latency, error rate, freshness, recovery objectives) before choosing architecture.
3. Select the smallest change that addresses the bottleneck; write down the cost and the reversal plan.
4. Test failures explicitly: DB outage, worker restart, retry storm, stale cache, and overloaded downstream services.
5. Monitor after release and revise the design when the observed bottleneck changes.
