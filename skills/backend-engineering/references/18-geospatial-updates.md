# Distribute high-volume location updates efficiently

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-17 — https://www.instagram.com/reel/DenRPl4kgVY/

### On-screen items from sampled frames
- The example is a ride-hailing system with **10,000 riders** and thousands of moving cars.
- **Keep only the latest location** rather than appending every movement to the current-position view.
- **Divide the map into tiles** to limit relevant nearby-driver candidates.
- **One offer at a time** appears in the dispatch flow.
- The clip hints that real geospatial partitioning does not necessarily use literal square tiles and references a free, open-source Uber geospatial technology from 2018 (consistent with H3; verify specifics before assuming a deployment uses it).

### Implementation guidance (synthesis)
- Use a geospatial index or cell identifier (e.g., H3) to narrow candidate-driver lookup by area, expanding search rings if needed.
- Store current coordinates with update time and expiration; historical routes, if required, need a distinct retention policy.
- Rate-limit/coalesce noisy device GPS updates, and reject stale or out-of-order position versions.
- Dispatch must be concurrency-safe: use assignment state transitions or compare-and-set so one driver cannot accidentally accept multiple conflicting rides.
- Account for GPS error, privacy/retention, location spoofing, rapid boundary crossings, and driver availability.
