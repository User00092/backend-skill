# Production application/database performance checklist

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @joshtheaiguy — 2026-09-14 — https://www.instagram.com/reel/DdP1ySxAsxT/

**Source note:** The supplied text attachment contains a short general caption; the specific technical items were read from the video's on-screen checklist.

### On-screen items
- ✅ Cache API responses.
- ✅ Index the database.
- ✅ Compress images.
- ✅ Split code into chunks.
- ✅ Add a CDN.
- ✅ Add lazy loading.
- ✅ Minify JS and CSS.
- ❌ Unnecessary re-renders.
- ✅ Debounce input handlers.
- ✅ Paginate large lists.
- ❌ Unused dependencies.
- ✅ Defer non-critical scripts.
- ✅ Loading skeletons.
- ✅ Load balancer.
- ✅ Cache expensive queries.
- ❌ N+1 database queries.
- ✅ Database connection pooling.
- ✅ Server-side caching.
- ✅ Compress API payloads.

### Practical interpretation
**API and caching**
- Define cache keys, scope/tenancy, authorization boundaries, TTL, invalidation, and consistency. Never cache private responses in a shared public cache by accident.
- Apply compression selectively based on content type/size and existing edge compression.
- Measure cache hit rate and tail latency; caching without observability can hide correctness problems.

**Databases**
- Add indexes that support actual query filters, joins, and sort patterns; verify with `EXPLAIN`/execution plans and observe write/storage costs.
- Detect and remove N+1 round trips with appropriate batch loading/joins.
- Paginate potentially large result sets; consider keyset pagination where offset scans become costly.
- Bound connection pools to avoid overwhelming a limited DB, especially across multiple app instances.
- Cache expensive query results when invalidation and correctness are understood.

**Frontend and delivery (included because the video covers full-stack speed)**
- Use image compression, CDN caching, lazy loading, code splitting, small bundles, non-blocking scripts, and reduced needless re-renders.
- Skeletons improve **perceived** performance but do not make the underlying service faster.
- Load balancers improve distribution and fault tolerance when multiple healthy instances exist; they are not automatic speed multipliers.

### Engineering checklist
- [ ] Establish baselines for p50/p95/p99 latency, error rate, throughput, and DB load.
- [ ] Profile and fix high-cost queries; add indexes and control N+1 behavior.
- [ ] Implement authorized caching with deliberate invalidation.
- [ ] Use pagination, request compression, and appropriately sized connection pools.
- [ ] Improve bundles, images, CDN delivery, and client rendering bottlenecks.
- [ ] Load test the system and verify measurable gains rather than applying every optimization blindly.
