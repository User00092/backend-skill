# Distribute cache keys with consistent hashing

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @arjay_the_dev — 2026-09-21 — https://www.instagram.com/reel/Ddjy_ZEpJJ0/

### Lessons extracted from supplied description and sampled video frames

- A naive cache assignment such as `hash(key) % number_of_servers` often remaps a large fraction of keys when the server count changes. The video contrasts **three cache servers** with **four**, showing previously cached keys being assigned to different servers and a resulting rise in database load from cache misses.
- **Consistent hashing** positions both cache nodes and keys on a logical hash ring. A key is assigned to the next node clockwise on the ring, wrapping around at the end.
- Adding a server changes ownership mainly for keys in the ring interval claimed by that server, rather than reshuffling nearly the whole cache. Removing a server similarly reassigns its share to successor nodes.
- This limits cold-cache traffic when scaling out and is particularly useful for a distributed cache client that chooses the node for each key.
- The source illustrates the concept with named cache nodes and an example key such as `user:42`.

### Implementation guidance (additional synthesis; not a creator quote)

1. Choose a stable hash algorithm and a deterministic, versioned mapping so every client calculates the same owner.
2. Add **virtual nodes** (or a weighted variant such as rendezvous hashing) to balance uneven placement, especially with small physical clusters or differently sized cache machines.
3. Plan graceful handoff or prewarming: consistent hashing *reduces* movement but cannot prevent cache misses for reassigned keys. Monitor database request spikes during topology changes.
4. Account for node failures and replication. A single-owner hash ring by itself does not provide redundancy or durable cache state.
5. Keep membership changes consistent across clients; diverging views of the ring can create misses or inconsistent reads.
6. Validate the actual cache implementation. Systems such as Redis Cluster have their own hash-slot and rebalancing mechanisms; do **not** assume adding consistent hashing client-side is appropriate.
7. Measure remapped-key fraction, hit rate, origin/database QPS, eviction, and p95/p99 latency before and after adding/removing nodes.

**Numerical caveat:** The creator says a modulo scheme changing from 3 to 4 nodes *can* remap roughly 75% of keys. This is an illustrative estimate, not a universal constant; the exact fraction depends on hashing and server assignment. A simple contiguous uniform ring with one token per server can also be unbalanced without virtual nodes.

### Source text (verbatim supplied `.txt` description; not verified audio transcription)

```text
How to add a cache server safely
Consistent hashing is a way to distribute data across multiple servers without reshuffling almost everything every time you add or remove one.
With a simple hash(key) % servers setup, going from 3 servers to 4 can remap roughly 75% of your keys, causing a huge spike in cache misses and database traffic.
With consistent hashing, only a small portion of the keyspace moves to the new server, so the rest of your cache stays hot.
#systemdesign #coding #programming
```
