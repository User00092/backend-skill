# Scale reads, writes, real-time updates and reliability separately

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-15 — https://www.instagram.com/reel/Deh5HkkCsPu/

### On-screen items from sampled frames
- Six problems are introduced: **scale reads**, **scale writes**, **real-time**, **background jobs**, **reliability**, and **split reads & writes**.
- The visuals show **batching** for writes, **WebSockets** and **server-sent events (SSE)** for live updates, and separate paths for reads and writes.
- The clip frames scaling as choosing a solution based on the traffic and data pattern, not adopting every trendy infrastructure component.

### Implementation guidance (synthesis)
- **Read pressure:** profile queries, indexes, caching and read replicas; budget for replica lag.
- **Write pressure:** batching, queues, partitioning and reducing unnecessary writes; preserve integrity and latency requirements.
- **Real-time:** WebSockets for bidirectional communication; SSE for one-way server push; design authorization, heartbeats and reconnect behavior.
- **Background jobs:** use durable queues when task completion must survive restarts; make workers idempotent.
- **Reliability:** bound timeouts, retries with jitter, circuit breakers where appropriate, and backpressure.
- **Separate reads/writes:** recognize CQRS-style separation can increase synchronization complexity and stale-read risk.
