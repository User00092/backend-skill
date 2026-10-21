# Shard data with a stable, query-aware key

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-12 — https://www.instagram.com/reel/DeZfb1GilY3/ — EP12.

### Video-derived skills

- A scenario depicts roughly **4 TB of orders** and a similarly sized replica, motivating discussion of sharding; these are scenario values, not established cutoffs for sharding.
- **Sharding** distributes records across multiple independent data partitions/nodes.
- A `hash(customer_id) → shard` example keeps a given customer's records together, aiding customer-specific queries.
- A global query such as **total sales today** may need fan-out/aggregation across all shards.
- The video contrasts shard-key choices and shows a **hot-shard** trap: uneven traffic/load.
- It also illustrates the cost of moving data when adding shards with a naïve `id % N` mapping; changing the modulus remaps many keys.
- The video explicitly raises less invasive options first, including **indexes**, **read replicas**, **caching**, and **archiving old data**.

### Practical guidelines (synthesis)

1. Verify that indexing, partitioning, lifecycle policies, storage scaling, and read replicas cannot solve the current problem more simply.
2. Pick a shard key informed by real query patterns, cardinality, tenant locality, and distribution—not just an easy modulo operation.
3. Use a stable routing scheme (directory service, consistent/rendezvous hashing, or carefully managed virtual shards) if gradual rebalancing matters.
4. Design cross-shard queries, global uniqueness, joins, transactions, backups/restores, migrations, and failure isolation before rollout.
5. Monitor hot shards, skew, data size, per-shard latency, migration lag, and operational overhead.
