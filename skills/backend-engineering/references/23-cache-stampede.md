# Prevent Cache Stampedes with Coalescing, Stale Responses, and Jitter

## When to use

Read this when a hot cached key expires and backend traffic spikes, when many workers independently regenerate the same expensive result, or when synchronized TTLs cause overload of a database or origin service.

## What the supplied video teaches

**Source:** `backend_mockreps_2026-10-19_DeqVnX_imSE.mp4`, plus its supplied `.txt` description (MockReps, October 19, 2026). Sampled video frames show many requests hitting a warm cache, followed by a synchronized expiration and a rush of duplicate database rebuilds. The description identifies three fixes:

1. **One rebuilder** — request coalescing (also called single-flight) lets one caller populate the missing cache entry while peers wait or use a fallback.
2. **Serve stale while refreshing** — return previously cached data while one background job refreshes it.
3. **Jittered expiry** — spread expiration times to reduce synchronized misses.

This is a conceptual demonstration, **not** a validated throughput benchmark or a verified audio transcription.

## Why it happens (additional engineering guidance)

With `N` simultaneous requests for an expired popular key, a naïve cache-aside implementation can trigger up to `N` recomputations or database queries. Increasing TTL alone postpones the event. A single hot key can still expire, while identical TTLs across many keys can create a fleet-wide expiry wave.

## Strategy A — Single-flight/request coalescing

```python
# Illustrative pseudocode: single-process only.
async def get_hot_item(key):
    cached = await cache.get(key)
    if cached is not None:
        return cached

    async with per_key_lock(key):
        # A peer may have filled the cache while this caller waited.
        cached = await cache.get(key)
        if cached is not None:
            return cached
        value = await load_from_database(key)
        await cache.set(key, value, ttl=ttl_with_jitter())
        return value
```

- Recheck the cache **after** acquiring the lock to avoid redundant loads.
- Scope the lock by cache key; a global lock blocks unrelated keys.
- This example only coalesces requests inside one process. Multiple hosts need distributed coordination or another design; choose bounded lease durations and account for lock-owner crashes. Never assume a Redis lock yields perfect exactly-once execution.
- Bound wait times, in-flight work, connection pools, and concurrency. Avoid waiting forever when the rebuilder fails.

## Strategy B — Stale-while-revalidate

Maintain a **freshness deadline** and a later **hard expiration deadline**:

```text
fresh:   serve immediately
stale:   serve cached value; trigger at most one refresh
missing: coalesce a rebuild; otherwise use bounded fallback
expired beyond hard deadline: do not serve without explicit policy
```

- Serve stale only where acceptable: e.g., public catalog or aggregate counts. Do **not** casually use stale values for authorization, account balances, inventory reservations, or payment state.
- Limit refresh concurrency and ensure refresh failure doesn't flood the origin.
- Record refresh failures, stale age, and the number of callers served stale.
- Decide whether failure should preserve the previous entry or fail closed based on the data's correctness requirements.

## Strategy C — Expiration jitter

```python
import random

def ttl_with_jitter(base_seconds=300, spread_seconds=60):
    return base_seconds + random.uniform(0, spread_seconds)
```

- Randomized TTLs help prevent synchronized expiry across **different keys**. They do **not** prevent a stampede on a **single extremely hot key**; combine with coalescing or stale-while-revalidate.
- Choose bounds aligned to data freshness and capacity targets. Do not extend expiration past mandatory correctness windows.

## Production checklist

1. Identify hot keys and whether misses or rebuild cost drive overload.
2. Measure cache hit ratio, origin request rate, rebuild concurrency, queue depth, p95/p99 latency, and errors.
3. Choose per-key coalescing for concurrent misses; consider a distributed lease if requests span processes.
4. Set lease/wait timeouts, retry backoff, and maximum backend concurrency.
5. Enable stale serving only for data whose freshness policy permits it.
6. Add TTL jitter for expiry waves; pre-warm exceptionally hot entries if justified.
7. Load-test a simultaneous expiration, distributed lock-holder crash, cold start, and unavailable database.
8. Confirm correctness under invalidation and deploy/restart scenarios; protect against poisoned or empty cache values.

## Example failure modes to test

- The lock holder crashes after acquiring the lease.
- Refresh takes longer than the lock duration, allowing multiple rebuilders.
- An invalidation races with an older rebuild, causing stale overwrite; consider generations/version checks.
- Repeated refresh errors cause unlimited retries or permanent stale data.
- All application instances cold-start together and miss their local caches.

## Related references

- [Production performance](05-application-database-performance.md)
- [Consistent hashing](21-consistent-hashing.md)
- [Scaling backend systems](16-scaling-backend-systems.md)
- [Engineering tradeoffs](14-engineering-tradeoffs.md)

## Supplied creator description (verbatim)

> Your cache expires at midnight… and your database melts. Why? 🔥
>
> Tricky interview questions, Part 01: the cache stampede.
>
> The fixes:
> 1. One rebuilder (request coalescing)
> 2. Serve stale while refreshing
> 3. Jittered expiry
>
> Save this for your next system design interview 📌 and send it to a friend who's prepping.
>
> Practise explaining it out loud at mockreps.com (link in bio).
>
> #systemdesign #softwareengineer #techinterview #interviewprep #caching
