# Optimize mobile-app startup on weak connections

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-16 — https://www.instagram.com/reel/DekjJ-vAJuN/

### On-screen items from sampled frames
- The example mobile application starts by requesting **15 endpoints** before presenting useful UI; a spinning loader is shown despite no visible content.
- **Fix 1 — Trust the phone:** restore a local signed-in state while the server validates or refreshes in the background; avoid waiting for redundant round-trips before drawing UI.
- **Fix 2 — Cache the home screen:** show a saved screen or data immediately and refresh it asynchronously.
- **Fix 3 — Run in parallel:** execute independent API requests concurrently instead of building a serial waterfall.
- The example also shows an **expired subscription** state, illustrating that some server-authoritative entitlements must be rechecked.

### Implementation guidance (synthesis)
- Render a usable skeleton or cached state as early as possible, while marking stale or pending data appropriately.
- Fetch only above-the-fold/critical data first; batch endpoints when it reduces overhead without creating excessively large responses.
- Deduplicate requests, use bounded parallelism, and avoid dozens of competing calls on weak connections.
- Cached UI is not proof of authorization: the backend must independently check authentication and entitlements for every protected action.
- Measure cold start, time to first meaningful paint, percentile latency, payload sizes, and behavior on emulated low-bandwidth/high-latency networks.
