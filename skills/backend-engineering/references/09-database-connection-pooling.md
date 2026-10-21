# Bound PostgreSQL connections with pooling

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-06 — https://www.instagram.com/reel/DeKQJykieLS/ — EP04.

### Source-derived skills

- As stateless app instances scale, their independent database connection pools can jointly exceed PostgreSQL's connection capacity.
- Bursty serverless invocations can make the problem more severe, even when each instance appears lightweight.
- A connection pool reuses a bounded set of established DB connections. The video shows ten app instances and a smaller shared connection pool.
- A dedicated pooler such as **PgBouncer** can provide a shared front door to PostgreSQL.
- The video also shows a **pool-exhaustion** failure: a slow payment integration ties up checked-out connections and checkout requests wait.

### Practical guidelines (synthesis)

- Budget globally: `max app replicas × maximum pool connections`, plus workers, migrations, administrative sessions, and headroom.
- Set acquire timeouts, query/transaction timeouts, and concurrency limits. Do not hold a DB transaction open across network calls to payment or email services.
- Choose PgBouncer session/transaction pooling with awareness of session state, prepared statements, and ORM compatibility.
- Track active/idle/waiting connections, pool wait time, DB CPU, locks, and query latency. Pooling limits connections; it does **not** make inefficient SQL fast.
