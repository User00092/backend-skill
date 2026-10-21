# Make side-effecting requests idempotent

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-10 — https://www.instagram.com/reel/DeUi98AAM7E/ — EP10.

### Source-derived skills

- A charge may succeed while the response is lost. A timeout therefore means **outcome unknown**, not necessarily failure.
- Retrying a payment without deduplication risks charging twice.
- The video shows an **idempotency key** shared across attempts, enabling the provider or server to recognize repeats.
- Duplicate user taps, client retries, and redelivered queue messages are related sources of duplicates.

### Practical guidelines (synthesis)

1. Have the client supply a stable key for one logical operation and reuse it across retries.
2. Scope keys to the authenticated account and operation; also bind/compare the request payload to reject incompatible key reuse.
3. Atomically record operation state and result. Concurrent same-key requests must not start independent charges.
4. Return or reconstruct the existing result for successful duplicate requests; handle in-progress attempts explicitly.
5. Use provider-supported idempotency features and reconcile unknown outcomes against the provider before issuing a new charge. Define key retention policy.
6. For queue consumers, use durable deduplication and database constraints around the side effect.
