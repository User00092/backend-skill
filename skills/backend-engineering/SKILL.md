---
name: backend-engineering
description: Use when designing, implementing, debugging, reviewing, or optimizing backend services and related web systems. Covers database performance, distributed messaging, idempotency, API architecture, caching, scalability, HTTP, SEO, security, and user-perceived performance. Consult the topic index and read only the relevant reference documents.
---

# Backend Engineering Skill

Use this skill as a **routing index**, not a command to load every document.

## Workflow

1. Identify the user's concrete engineering problem and its constraints.
2. Pick one or more topics below based on their **When to read** descriptions.
3. Open the corresponding `references/*.md` files and apply the checklists/guidance relevant to the task.
4. Distinguish **creator-supplied claims**, **observed video items**, and **additional engineering synthesis**; do not present measurements or simplified claims as universal guarantees.
5. Cross-check security-critical or version-sensitive choices against current authoritative documentation and the user's environment.
6. Prefer targeted, measurable fixes and explain latency, correctness, reliability, operational, and cost tradeoffs.

## Topic router

| Topic | When to read | Reference |
|---|---|---|
| Debounce search requests and cancel stale requests | Typeahead search, API request bursts, outdated fetch results. | [`references/01-debounce-and-cancel-stale-requests.md`](references/01-debounce-and-cancel-stale-requests.md) |
| Reduce shared-counter contention and false sharing | Atomic counter hot spots, thread contention, cache-line bouncing. | [`references/02-cache-line-contention.md`](references/02-cache-line-contention.md) |
| Decouple services using durable pub/sub and idempotent consumers | Async events, reliable queues, retries, consumer idempotency. | [`references/03-durable-pub-sub.md`](references/03-durable-pub-sub.md) |
| Technical SEO and indexing checklist | Indexing, sitemaps, metadata, structured data, crawler access. | [`references/04-technical-seo.md`](references/04-technical-seo.md) |
| Production application/database performance checklist | Slow APIs, database queries, cache decisions, pagination. | [`references/05-application-database-performance.md`](references/05-application-database-performance.md) |
| CAPTCHA, risk scoring, and false positives | Bot mitigation, CAPTCHA scores, false positives. | [`references/06-captcha-bot-scoring.md`](references/06-captcha-bot-scoring.md) |
| Apply the SOLID principles | Object-oriented module boundaries, interface design, testability. | [`references/07-solid-principles.md`](references/07-solid-principles.md) |
| Ask what breaks first when designing a system | Initial architecture planning, failure mode identification. | [`references/08-failure-first-system-design.md`](references/08-failure-first-system-design.md) |
| Bound PostgreSQL connections with pooling | PostgreSQL connection exhaustion, PgBouncer, pool sizing. | [`references/09-database-connection-pooling.md`](references/09-database-connection-pooling.md) |
| Make side-effecting requests idempotent | Safe retries for payments and side-effecting operations. | [`references/10-idempotency-keys.md`](references/10-idempotency-keys.md) |
| Use a transactional outbox for reliable messages | Database writes and reliable message publication. | [`references/11-transactional-outbox.md`](references/11-transactional-outbox.md) |
| Choose REST or GraphQL based on client needs | Choosing REST/GraphQL, API querying and caching. | [`references/12-rest-vs-graphql.md`](references/12-rest-vs-graphql.md) |
| Shard data with a stable, query-aware key | Large datasets, partition-key design, hot shards. | [`references/13-database-sharding.md`](references/13-database-sharding.md) |
| Judge system-design tradeoffs and failure modes | Choosing reliability/latency/operational tradeoffs. | [`references/14-engineering-tradeoffs.md`](references/14-engineering-tradeoffs.md) |
| Interpret and return HTTP status codes correctly | HTTP response semantics, errors, API monitoring. | [`references/15-http-status-codes.md`](references/15-http-status-codes.md) |
| Scale reads, writes, real-time updates and reliability separately | Scaling read, write, real-time and background workloads. | [`references/16-scaling-backend-systems.md`](references/16-scaling-backend-systems.md) |
| Optimize mobile-app startup on weak connections | Mobile app startup latency and weak networks. | [`references/17-mobile-startup-performance.md`](references/17-mobile-startup-performance.md) |
| Distribute high-volume location updates efficiently | Frequent location writes, spatial indexing, dispatch. | [`references/18-geospatial-updates.md`](references/18-geospatial-updates.md) |
| Secure payment webhooks and make fulfillment retry-safe | Authenticating payment webhooks and safe fulfillment. | [`references/19-payment-webhooks.md`](references/19-payment-webhooks.md) |
| Improve frontend performance and Core Web Vitals | Web loading performance, render blocking, Core Web Vitals. | [`references/20-core-web-vitals.md`](references/20-core-web-vitals.md) |
| Distribute cache keys with consistent hashing | Distributing cache keys and adding/removing cache servers. | [`references/21-consistent-hashing.md`](references/21-consistent-hashing.md) |

| Design modular monolith boundaries and module contracts | Monolith architecture, internal API ownership, enforcing dependency boundaries, and deciding when to extract microservices. | [`references/22-modular-monolith.md`](references/22-modular-monolith.md) |
| Prevent cache stampedes and synchronized expiration | Hot-key cache expiry, many simultaneous misses, stale-while-revalidate, request coalescing, TTL jitter. | [`references/23-cache-stampede.md`](references/23-cache-stampede.md) |

## Source and provenance

These notes were compiled from 23 user-provided educational clips and their available descriptions. Not all videos have complete audio transcripts. The source descriptions are **not verified audio transcriptions**. The initial 21-topic combined source document, including its supplied descriptions and attribution notes, is preserved at `../../archive/backend-skills-original.md` relative to this skill folder. Read that archive only when a precise source quote, attribution, or missing appendix item is needed.

## Maintenance

- Each distinct topic belongs in one focused `references/NN-*.md` file.
- Add or update an entry in the topic router when adding a reference.
- Keep the frontmatter `name` matching the skill directory name.
- Never silently replace source material with externally inferred claims. Label any added guidance.
