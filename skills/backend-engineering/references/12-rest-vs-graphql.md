# Choose REST or GraphQL based on client needs

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-11 — https://www.instagram.com/reel/DeXFeXjjYsu/

### Video-derived skills

- A sneaker product page is depicted making multiple REST calls (`/product`, `/sizes`, `/stock`, `/reviews`, `/sales`, `/images`).
- The GraphQL alternative depicts one `POST /graphql` query selecting product fields such as name, price, and sizes.
- Video highlights REST's simpler URL-oriented caching and the need to deliberately build caching behavior for GraphQL.
- A query depth example illustrates the need to restrict GraphQL query complexity and resource usage.

### Practical guidelines (synthesis)

- Prefer **REST** when resources, cacheability, CDN behavior, and a straightforward HTTP contract are primary.
- Consider **GraphQL** when clients need diverse, nested, changing views of related data and can benefit from explicit field selection.
- Fewer HTTP requests do **not** automatically mean less DB work or lower latency. Avoid resolver N+1 problems with batching/loaders, query planning, and profiling.
- With GraphQL, enforce authorization at resolver/data boundaries, maximum depth/complexity, pagination, timeouts, and sensible persisted-query/caching policies.
- REST can aggregate data into purpose-built endpoints, and GraphQL can be cached; neither architecture guarantees performance by itself.
