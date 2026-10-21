# Backend Skills — Video-Derived Engineering Reference

> Compiled from twenty-one supplied Instagram clips and their available accompanying text files. The **Source text** sections reproduce supplied `.txt` descriptions verbatim where provided; they are not independently verified word-for-word audio transcripts. Videos lacking supplied descriptions are represented by their observed visual content, not a complete spoken transcript. **On-screen items** were extracted from sampled video frames. **Implementation guidance** is my synthesis, not a quote from a creator. Treat the creators’ measurements, defaults, and generalizations as claims requiring verification in your own stack.

## Quick index

| # | Skill | Source | Primary use |
|---|---|---|---|
| 1 | Debounce requests and cancel stale fetches | @codewithdeekeej | Typeahead search and API efficiency |
| 2 | Minimize cache-line contention | @codezup | Highly concurrent counters |
| 3 | Design reliable asynchronous pub/sub | @goatengineerhq | Decoupled backend jobs |
| 4 | Make websites crawlable and indexable | @joshtheaiguy | SEO implementation |
| 5 | Audit application and database performance | @joshtheaiguy | Production scalability |
| 6 | Understand CAPTCHA risk scoring | @krishnachaytanyaa | Bot mitigation design |
| 7 | Apply SOLID object-oriented design principles | @kulstech | Maintainable modules and contracts |
| 8 | Design systems by identifying failure modes first | @medbouhezza | Architecture and resilience planning |
| 9 | Bound database connections through pooling | @medbouhezza | PostgreSQL and horizontally scaled apps |
| 10 | Make retries safe with idempotency keys | @medbouhezza | Payments and side-effecting APIs |
| 11 | Publish reliably with a transactional outbox | @medbouhezza | DB-to-queue consistency |
| 12 | Choose between REST and GraphQL | @medbouhezza | API design and caching |
| 13 | Shard data using deliberate partition keys | @medbouhezza | Large-dataset scaling |
| 14 | Judge system-design tradeoffs and failure modes | @medbouhezza | Architecture decisions |
| 15 | Use meaningful HTTP status codes | @medbouhezza | API responses and troubleshooting |
| 16 | Scale reads, writes, real-time updates and reliability | @medbouhezza | Distributed application scaling |
| 17 | Improve slow-network mobile startup | @medbouhezza | Mobile API and UX performance |
| 18 | Handle high-volume geospatial updates | @medbouhezza | Ride-hailing and location services |
| 19 | Process payment webhooks securely and idempotently | @peterstewiestartup | Payment fulfillment |
| 20 | Improve web performance and Core Web Vitals | @yatesvids | Frontend loading and interactivity |
| 21 | Distribute cache keys with consistent hashing | @arjay_the_dev | Scale cache nodes without remapping most keys |


## 1. Debounce search requests and cancel stale requests

**Source:** @codewithdeekeej — 2026-08-11 — https://www.instagram.com/reel/Db6UeOQuWZv/

### Lessons extracted
- An input event can trigger on every text change; coupling every event to a network call creates unnecessary calls.
- **Trailing-edge debounce** invokes work only after there have been no further events for the selected wait period. New typing resets the wait.
- Debounce does **not** guarantee only one call: a pause longer than the wait can launch multiple searches during the same typing session.
- Choose and measure the wait period for your users and latency requirements. The source highlights a 300 ms example and mentions Livewire `wire:model.live` and `.debounce.400ms`.
- **Throttle** is different: it rate-limits execution frequency during continuous activity, often useful for scroll/resize callbacks.
- Cancel or reject obsolete in-flight responses to prevent older search results overwriting newer ones.

### Practical implementation pattern
```javascript
let timer;
let controller;
let generation = 0;

function searchOnInput(query) {
  clearTimeout(timer);
  controller?.abort();
  const requestId = ++generation;
  timer = setTimeout(async () => {
    const current = new AbortController();
    controller = current;
    try {
      const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`, {
        signal: current.signal,
      });
      if (!response.ok) throw new Error(`Search failed: ${response.status}`);
      const data = await response.json();
      if (requestId === generation) renderResults(data);
    } catch (error) {
      if (error.name !== 'AbortError' && requestId === generation) showSearchError(error);
    }
  }, 400);
}
```
The sample is illustrative. Production code should also handle component teardown, blank queries, minimum character thresholds, accessibility, and loading states. Debouncing is a client optimization, **not a substitute for server-side rate limiting**.

### Engineering checklist
- [ ] Avoid issuing a request for every keystroke without a deliberate reason.
- [ ] Tune debounce using observed typing patterns, request latency, and UX requirements.
- [ ] Cancel in-flight requests and guard against stale responses.
- [ ] Add server-side rate limits, minimum query lengths where appropriate, and caching when useful.
- [ ] Measure request volume and end-to-end search latency before/after.

## 2. Reduce shared-counter contention and false sharing

**Source:** @codezup — 2026-08-17 — https://www.instagram.com/reel/DcJGGWNABg9/

### Lessons extracted
- A logically correct counter can scale poorly when several threads or cores repeatedly modify a **shared cache line**.
- Cache coherence forces ownership transfers for writes. Contention can bottleneck atomic increments even without an explicit application mutex.
- Spread counter writes across **striped or per-CPU/per-thread slots**, aggregate them on read, and pad/alignment-isolate hot slots if needed.
- The clip uses Java `LongAdder` as an example of striped accumulation and compares it with Linux counters.
- The clip claims approximately **85 ns between core clusters**, **17 ns within one cluster** on an AMD Ryzen 9 5950X, and a near **12M increments/sec** illustrative limit. These figures are not independently verified and are hardware/workload-dependent.

### Engineering checklist
- [ ] Identify high-contention writes with profiling before changing the data model.
- [ ] Prefer striped counters (`LongAdder` in Java, or an appropriate equivalent) for high-update, lower-frequency-read telemetry.
- [ ] Pad separate frequently written values to avoid unintended cache-line sharing when profiling justifies it.
- [ ] Decide whether reads must provide an atomic exact snapshot; striped counters may return a non-atomic aggregate during concurrent updates.
- [ ] Benchmark on deployment hardware at realistic concurrency.

## 3. Decouple services using durable pub/sub and idempotent consumers

**Source:** @goatengineerhq — 2026-09-25 — https://www.instagram.com/reel/DdtqrOSMAor/

### Lessons extracted
- Publish an event once, then let independently deployed consumers handle work such as email, analytics, or fraud screening without extending the checkout request path.
- Durable brokers/queues allow subscribers to recover and process backlog after an outage. Plain **Redis Pub/Sub is ephemeral** and does not retain messages for offline subscribers.
- Many messaging systems offer **at-least-once delivery**, meaning duplicate deliveries are possible. Make handlers idempotent (e.g., persist a unique processed event/order identifier).
- Kafka preserves ordering **within a partition**, not globally across partitions. Key related events consistently where order matters.
- Retry temporary failures with bounded backoff; redirect poison messages to a **dead-letter queue** after policy-defined retries.
- Use request/response calls for workflows needing an immediate result (e.g., authorization/confirmation of a payment), rather than pretending eventual events satisfy synchronous requirements.
- Mentioned technologies: Kafka, Google Cloud Pub/Sub, SNS→SQS, Redis Streams, Redis Pub/Sub.

### Engineering checklist
- [ ] Publish durable events when eventual execution is acceptable.
- [ ] Use an **outbox** or transactionally safe pattern when a DB change and event publication must not drift.
- [ ] Design consumers to survive duplicate deliveries and restarts.
- [ ] Define ordering keys, retries, visibility/acknowledgment behavior, DLQ handling, and replay policy.
- [ ] Monitor lag, queue depth, redelivery rates, DLQs, and end-to-end completion times.
- [ ] Keep critical synchronous confirmations on a correctly designed synchronous path.

## 4. Technical SEO and indexing checklist

**Source:** @joshtheaiguy — 2026-09-12 — https://www.instagram.com/reel/DdMjkxiyaJ/

**Source note:** The text attachment has only a brief caption. The actionable details below were read from **on-screen checklists** in the video, not from a speech transcript.

### On-screen items
- ✅ Build a `sitemap.xml`.
- ✅ Build a `robots.txt`.
- ✅ Meta titles.
- ✅ Meta descriptions.
- ✅ Canonical tags.
- ✅ Only one H1 tag per page (presented as a recommended rule in the clip).
- ✅ Fix header hierarchy.
- ✅ Add alt text to images.
- ✅ Improve Core Web Vitals.
- ✅ Internal links.
- ✅ Add schema markup.
- ✅ Compress images.
- ✅ Fix mobile responsiveness.
- ✅ Add an `og:image`.
- ✅ Build a backlink strategy.
- ✅ Clean up URL slugs.
- ✅ Verify Search Console.
- ❌ `noindex` tags.
- ❌ Broken links.

### Interpretation and cautions
- Check `noindex` on pages intended for public indexing; **do not** remove it indiscriminately from staging, private, filtered, or duplicate pages.
- A sitemap helps discovery but does not guarantee indexing. `robots.txt` controls crawler access; it is **not** a secrecy mechanism or a guaranteed de-indexing tool.
- A single H1 can be a useful editorial convention, but multiple H1 elements are not inherently an indexing prohibition.
- Alt text should convey meaningful image information, not keyword stuffing; decorative images can have empty alt text.
- Structured data must represent real page content and comply with search engine policies.
- Backlinks should be earned and relevant, not purchased or artificially manufactured.

### Engineering checklist
- [ ] Return indexable, canonical, accessible HTML for intended public pages.
- [ ] Generate valid `robots.txt` and XML sitemaps.
- [ ] Inspect `noindex`, canonical URLs, redirects, broken links, and HTTP status codes.
- [ ] Create descriptive metadata, headings, internal linking, and image text.
- [ ] Improve loading speed, mobile layout, structured data, and OG previews.
- [ ] Validate with actual crawl data and search console rather than assuming ranking improvements.

## 5. Production application/database performance checklist

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

## 6. CAPTCHA, risk scoring, and false positives

**Source:** @krishnachaytanyaa — 2026-08-19 — https://www.instagram.com/reel/DcOPmONtnQB/

### Lessons extracted
- CAPTCHA/bot defenses can consider session/cookie context, browser attributes, network reputation, and interaction signals; implementation varies by vendor/product.
- Bot detection is probabilistic and can generate both false positives (legitimate users challenged) and false negatives (automation accepted).
- The source describes reCAPTCHA v3 as producing scores from **0.0 to 1.0** and cites **0.5** as a default threshold example.

### Important qualification
The video blends concepts from multiple reCAPTCHA generations. **reCAPTCHA v3 is score-based and does not itself automatically show image challenges to users below 0.5**; an application may choose further verification or other actions based on the score. Avoid treating claimed mouse-tracking behavior or a universal threshold as an established public specification for every CAPTCHA system.

### Engineering checklist
- [ ] Validate verification tokens on the server, including expected site key, hostname/action, freshness, and other vendor-required checks.
- [ ] Tune risk thresholds using actual abuse and false-positive rates.
- [ ] Provide accessible alternatives and recovery routes for legitimate users.
- [ ] Layer anti-abuse controls: rate limits, replay prevention, monitoring, and account/session rules as appropriate.
- [ ] Avoid assuming a checkbox click proves human identity or that a score proves malicious intent.

## Shared engineering principles

1. **Measure before and after.** Compare p95/p99 latency, DB query counts, CPU contention, cache hit rates, request volume, and failure rates.
2. **Distinguish optimization from correctness.** Debounce does not enforce quotas; caches can expose data; striped counters can sacrifice snapshot precision.
3. **Design for failures.** Cancel stale requests, retry transient events, dead-letter poison events, tolerate subscriber downtime.
4. **Respect semantics.** Use synchronous APIs for immediate answers; asynchronous events when eventual completion is acceptable.
5. **Avoid cargo-cult configuration.** All intervals, indexes, pool sizes, cache TTLs, and risk thresholds depend on the workload.

## Exact supplied source text

The blocks below preserve the uploaded `.txt` text for each clip. These text files may be descriptions/captions rather than full transcripts of the spoken audio. Not all spoken words are verified here.

### Source 1: codewithdeekeej — Input debouncing and request cancellation

Source file: `backend_codewithdeekeej_2026-08-11_Db6UeOQuWZv.txt`

```text
Your search box is DDoSing your own API.

Twenty characters. “prague weather today”. Your endpoint got twenty requests for one search — and nineteen of them were for a query nobody meant to run.

It’s not a bug you introduced. It’s the default. MDN: the input event “is fired every time the value of the element changes.” One character, one handler run, one fetch.

Debounce fixes it, but not the way most people describe it. It doesn’t run your function on a timer. lodash: it “delays invoking func until after wait milliseconds have elapsed since the LAST time the debounced function was invoked.” Every new keystroke throws the timer away and starts over. It fires on the silence, not on the clock.

That’s why the bar in the video keeps getting knocked back to zero.

The honest tradeoffs:

→ It is not “always 1 request”. Aalto measured the gap between keystrokes across 136 million of them: 239ms on average, 122ms for the fastest 10%, 481ms for the slowest. A 300ms window swallows a whole word for an average typist — but someone at 481ms clears the window between characters and gets a request for almost every one. That’s correct behaviour, not a leak.

→ On Livewire? Go look at your number. wire:model.live debounces at 150ms by default — that’s below the 239ms average gap, so for a typical typist the window expires between characters and you’re back to a request per keystroke. wire:model.live.debounce.400ms.

→ Debounce is not throttle. Debounce waits for you to stop. Throttle fires on a fixed interval no matter what. Search box wants debounce. Scroll handler wants throttle.

→ Still cancel the in-flight request. Otherwise an older response lands after a newer one and you render results for “prague weath”.

The word is borrowed from hardware — press a mechanical switch and the contacts physically bounce, firing several times. Ganssle scoped 18 of them, most around 1.5ms. We’ve been throwing away duplicate signals since long before search boxes.

Go look at your own input handler. One line, and your API log reads differently tomorrow.

Save this for your next search box.

#debounce #devtips #frontendtips #apiperformanc
```

### Source 2: codezup — Cache coherence and contention

Source file: `backend_codezup_2026-08-17_DcJGGWNABg9.txt`

```text
Eight cores. One counter. No locks anywhere in the code, and it still gets slower every time you add a thread.

The counter is correct. The problem is that the number lives inside one cache line, and only one core can own that line at a time. Every increment issued from a different core has to drag it across the chip first. Measured on an AMD Ryzen 9 5950X, that trip costs about 85 nanoseconds between core clusters against 17 inside one. One line, one trip at a time works out to a ceiling near 12 million increments a second, total, no matter how many cores you own.

The fix is to stop having one number: give every core its own padded slot, hash each thread to one, and add them up only when someone reads. That is exactly what Java's LongAdder does, and it stops growing at one slot per CPU. ConcurrentHashMap already counts its own size this way. So does the Linux kernel.

Comment "SOURCE" and I will send you the full write-up with every measurement and the source it came from.

Follow @codezup for more.

Comment "SOURCE" and I'll send you the full written breakdown.

#concurrency #cpu #cachecoherence #multithreading #performance
```

### Source 3: goatengineerhq — Durable pub/sub messaging

Source file: `backend_goatengineerhq_2026-09-25_DdtqrOSMAor.txt`

```text
One broken email service shouldn't be able to stop people from checking out.

Pub/sub is how most large systems avoid that. A service publishes a message to a topic once and moves on. Every service that subscribes gets its own copy in its own queue and reads it when it's ready. The publisher never knows who is listening, so you can add a new subscriber, a fraud check for example, without changing a line of checkout code.

The queues are where the resilience comes from. If a subscriber is down, its messages wait for it, and when it comes back it catches up. Nothing upstream had to wait.

Most brokers deliver at least once, so after a retry a subscriber can see the same message twice. Make the handler safe to repeat, for example by skipping an order ID you've already emailed.

Ordering is only guaranteed within a partition. Kafka keeps messages with the same key in order, but makes no promise across keys.

A message that keeps failing should move to a dead-letter queue after a few retries. Otherwise it keeps coming back, and in an ordered queue it holds up everything behind it.

Redis Pub/Sub stores nothing, so a subscriber that is offline simply misses the message. When messages have to survive an outage, people reach for Kafka, Google Cloud Pub/Sub, SNS feeding SQS queues, or Redis Streams.

Pub/sub is the wrong tool when the caller needs an answer before it can continue, like confirming a payment went through. That still needs a direct call.

#systemdesign #softwareengineering #backend #distributedsystems #pubsub #kafka #microservices
```

### Source 4: joshtheaiguy — SEO and crawlability

Source file: `backend_joshtheaiguy_2026-09-12_DdMjkxiyaJ.txt`

```text
Google isn’t showing your vibe coded website because you didn’t prompt Claude to add all this to your site.
#vibecoding #claudecode #grokbot #chatgpt #aiseo
```

### Source 5: joshtheaiguy — Production performance checklist

Source file: `backend_joshtheaiguy_2026-09-14_DdP1ySxAsxT.txt`

```text
Vibe-coding is all fun and games until you have actual user data running. Claude does not optimize for scale out of the box. Especially when using tools like Supabase, speed is a critical element to prompt around.

#claudecode #grokbot #vibecoding #codex #cursor
```

### Source 6: krishnachaytanyaa — CAPTCHA and bot assessment

Source file: `backend_krishnachaytanyaa_2026-08-19_DcOPmONtnQB.txt`

```text
CAPTCHA, explained : how a checkbox knows you're human.

The "I'm not a robot" box is not a test you pass by clicking it. From the moment the page loads it is already scoring you, and it looks at four kinds of thing: your cookies and session history, what your browser itself looks like, your network, and how your pointer actually moved on the way over.

Real hands wander, hesitate and overshoot. A script goes in a straight line, at a constant speed, into the exact centre.

That score is decided before you ever click. reCAPTCHA v3 hands the site a number from 0.0 to 1.0, and the documented default line is 0.5. Clear it and you are waved through and never see a thing. Fall short and you get the pictures.

Which is why a real person on a private browser with no history gets the traffic lights too. The score is a guess, and it is wrong in both directions.

Ever been called a robot by a website? Save this for the next time you are clicking fire hydrants.

Follow @krishnachaytanyaa for system design explained visually.

#trending #vibecoding #systemdesign #ai #softwarearchitecture #productionready #aitools
```


---

# Additional video-derived skills (sources 7–13)

These sections derive from the seven additional supplied clips and their `.txt` descriptions. **Video observations** come from sampled on-screen frames, which are not a complete audio transcription. **Practical guidelines** are implementation synthesis, not quotations. The original six sections above are retained.

## 7. Apply the SOLID principles

**Source:** @kulstech — 2026-07-14 — https://www.instagram.com/reel/DaxrQVmgKn0/

### Source-derived skills

- **S — Single Responsibility Principle:** Separate responsibilities that change for different reasons. The video decomposes a `UserService` handling `saveUser()`, `sendEmail()`, and `writeLog()` into a repository, email service, and logger.
- **O — Open/Closed Principle:** Make components extensible without modifying the established core for every new variant. The video depicts a `PaymentProcessor` extended by `CreditCard` and `PayPal` implementations.
- **L — Liskov Substitution Principle:** Subtypes must honor the behavioral expectations of their base type. The video illustrates a square/rectangle inheritance problem where changing width or height breaks expected behavior.
- **I — Interface Segregation Principle:** Prefer narrowly focused capabilities instead of requiring implementers to provide unrelated methods. The video splits an oversized machine interface into printer, scanner, and fax interfaces.
- **D — Dependency Inversion Principle:** High-level application logic depends on contracts/abstractions, not directly on a particular database implementation. The video shows a report generator using a database interface implemented by MySQL.

### Practical guidelines (synthesis)

- Split code along cohesive responsibilities, not automatically into one class per method.
- Use small, behaviorally meaningful interfaces when multiple interchangeable implementations or testing boundaries justify them.
- Test substitutability by verifying behavior and invariants, not merely matching method signatures.
- Inject database, queue, HTTP, and clock dependencies at boundaries; avoid elaborate abstraction layers when they add no flexibility.

## 8. Ask what breaks first when designing a system

**Source:** @medbouhezza — 2026-10-04 — https://www.instagram.com/reel/DeFTopCBAjy/ — EP01 of *How Senior Engineers Actually Think About System Design*.

### Video-derived skills

- The scenario uses a limited-time sneaker drop with approximately **50,000 users** trying to buy a product at the same time.
- A junior design is illustrated with a server and database; additional options shown include cache, queue, replica, and load balancer.
- The central question presented is **“What breaks FIRST?”** rather than simply **“What should I add?”**
- The video depicts queue backlog and an early-morning alert as failure examples; it does not establish measured capacity limits for a specific technology stack.

### Practical guidelines (synthesis)

1. Specify traffic shape, reads/writes, consistency, latency, and availability goals.
2. Map the critical path: ingress → app → cache/queue → database → outside services.
3. Calculate likely bottlenecks (connection limits, write contention, hot keys, queue consumption, inventory races).
4. Simulate spike/failure scenarios and establish backpressure, admission control, safe retries, and graceful degradation before adding infrastructure.
5. Instrument the path; optimize the bottleneck you actually measure.

## 9. Bound PostgreSQL connections with pooling

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

## 10. Make side-effecting requests idempotent

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

## 11. Use a transactional outbox for reliable messages

**Source:** @medbouhezza — 2026-10-11 — https://www.instagram.com/reel/DeWzzSlnQrM/ — EP11.

### Source-derived skills

- Saving an order and publishing a queue message as two separate actions creates a crash window: the order persists but the event does not publish.
- Simply reversing their order creates a different crash window: an event may publish for an order that never commits.
- The **transactional outbox** stores the business row and corresponding pending event in one database transaction.
- A relay worker publishes pending outbox records; the video shows subsequent delivery and a uniqueness check on `order_id` for the consuming handler.

### Practical guidelines (synthesis)

```sql
BEGIN;
INSERT INTO orders (id, status) VALUES (:order_id, 'placed');
INSERT INTO outbox (event_id, event_type, aggregate_id, payload, status)
VALUES (:event_id, 'order.placed', :order_id, :payload, 'pending');
COMMIT;
```

- Poll or stream pending outbox events with a worker, publish to a durable broker, then mark them delivered. Use safe concurrency coordination such as `FOR UPDATE SKIP LOCKED` where appropriate.
- The relay can crash after publishing but before updating the outbox: **duplicate delivery is still possible**. Consumers must be idempotent.
- Track outbox age, backlog, attempts, poison events, and dead-letter handling. Maintain unique message identifiers and a retention strategy.

## 12. Choose REST or GraphQL based on client needs

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

## 13. Shard data with a stable, query-aware key

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

## Source text supplied for clips 7–13 (verbatim descriptions, not audio transcripts)

### Source 7: `backend_kulstech_2026-07-14_DaxrQVmgKn0.txt`

```text
Five letters that separate messy code from clean code. 🧱 S-O-L-I-D.

Here's SOLID explained the simplest, most visual way — five principles 
that keep your code flexible and easy to change: Single Responsibility, 
Open/Closed, Liskov Substitution, Interface Segregation, and Dependency 
Inversion. Follow them, and big projects stop turning into spaghetti.

No jargon. Just "oh, now I get it." 💡

🔔 Follow Kuls Tech - complex tech, made simple and visual.

#SOLID #CleanCode #OOP #DesignPatterns #TechExplained
```

### Source 8: `backend_medbouhezza_2026-10-04_DeFTopCBAjy.txt`

```text
EP01: how senior engineers actually think about system design?
#system_design #series #edge_cases
#software_engineering
```

### Source 9: `backend_medbouhezza_2026-10-06_DeKQJykieLS.txt`

```text
You scaled to 10 servers… and your database started refusing connections. More servers, more errors. Why? 👀
In EP04 of How Senior Engineers Think About System Design, we break down how database connections multiply as your app scales—and how connection pooling keeps them under control.
We cover PostgreSQL connection limits, serverless traffic spikes, and how poolers like PgBouncer let requests reuse connections.
Ever hit “too many connections” in production? Tell me in the comments. 👇
Next: connections are under control, but a page still takes 4 seconds to load. Enter EP05.
Save this for your next system design interview and follow @medbouhezza.
#SystemDesign #PostgreSQL #ConnectionPooling #BackendDevelopment #SoftwareEngineering
```

### Source 10: `backend_medbouhezza_2026-10-10_DeUi98AAM7E.txt`

```text
One click. Two charges. How does that happen?
A payment succeeds, but its response gets lost. Your app retries—and charges the customer again.
Episode 10 explains how idempotency keys make retries safe, why duplicate messages matter, and why “no response” doesn’t mean “failed.”
Would you stop retrying—or make retries safe? Comment below.
Next: the order saves, but the server crashes before sending the email. Episode 11 tackles that gap.
#SystemDesign #Idempotency #BackendDevelopment #SoftwareEngineering #DistributedSystems
```

### Source 11: `backend_medbouhezza_2026-10-11_DeWzzSlnQrM.txt`

```text
Order saved. Email never sent. Where did it go?

Your server saves the order, then crashes before telling the queue to send the confirmation email. Switching the order of those steps only creates a different failure.

In Episode 11 of How Senior Engineers Actually Think About System Design, Mohamed explains the transactional outbox: save the order and the message together in one database transaction, then let a worker deliver the message. Since that worker can retry, consumers still need to handle duplicates safely.

Would you have sent the message first? Comment below.

Next: four terabytes of orders—and one database can’t hold them anymore. Episode 12 explores sharding.

#SystemDesign #TransactionalOutbox #BackendDevelopment #DistributedSystems #SoftwareEngineering
```

### Source 12: `backend_medbouhezza_2026-10-11_DeXFeXjjYsu.txt`

```text
The difference between REST and GraphQl
```

### Source 13: `backend_medbouhezza_2026-10-12_DeZfb1GilY3.txt`

```text
Sharding 🤔
```


## 14. Judge system-design tradeoffs and failure modes

**Source:** @medbouhezza — 2026-10-13 — https://www.instagram.com/reel/DecFoqVicbm/

### Lessons extracted
- Senior engineering work is not selecting every scaling pattern; it is deciding which failure is most consequential to the actual product and customers.
- The clip recaps previous fixes: load balancing, stateless servers, connection pooling, indexes, replicas, caching, stampede protection, queueing, idempotency, outbox, and sharding. Each solves a particular problem and adds operational or correctness costs.
- Determine what fails first: connection exhaustion, slow checkout, stale data, request latency, duplicate side effects, message loss, or excessive data volume.
- Decide if consistency, staleness, availability, or operational complexity is acceptable *for the specific workflow*. A cached price could be unacceptable at checkout even when cached catalog data is acceptable.

### Implementation guidance (synthesis)
1. Start with a failure scenario, measurable symptom, and customer impact.
2. Specify service objectives (latency, error rate, freshness, recovery objectives) before choosing architecture.
3. Select the smallest change that addresses the bottleneck; write down the cost and the reversal plan.
4. Test failures explicitly: DB outage, worker restart, retry storm, stale cache, and overloaded downstream services.
5. Monitor after release and revise the design when the observed bottleneck changes.

## 15. Interpret and return HTTP status codes correctly

**Source:** @medbouhezza — 2026-10-14 — https://www.instagram.com/reel/DeehBXKERyM/

### On-screen items from sampled frames
- `2xx`: success; example `200 OK` and `201 Created`.
- `3xx`: redirection; `301 Moved Permanently` appears in the clip.
- `4xx`: client-side request problems; `401 Unauthorized` (authentication needed), `403 Forbidden`, and `404 Not Found` are illustrated.
- The clip contrasts a `404` response with a successful `200` response, and recommends monitoring the resulting status codes rather than hiding errors.

### Implementation guidance (synthesis)
- `400`: invalid request syntax/parameters; `401`: unauthenticated; `403`: authenticated but not permitted; `404`: absent/hidden resource; `409`: state conflict; `422`: semantically invalid content when your API uses that convention; `429`: too many requests.
- `500`: unexpected server failure; `502`: invalid upstream response; `503`: temporary service unavailability; `504`: upstream timeout.
- Prefer accurate status codes and structured error bodies. Do not respond `200` with an `error` string merely to keep success dashboards green.
- Use `Retry-After` for applicable rate limits or temporary unavailability; treat redirects and unsafe methods carefully.
- Track rates by endpoint/status class; do not assume every `4xx` indicates a server outage.

## 16. Scale reads, writes, real-time updates and reliability separately

**Source:** @medbouhezza — 2026-10-15 — https://www.instagram.com/reel/Deh5HkkCsPu/

### On-screen items from sampled frames
- Six problems are introduced: **scale reads**, **scale writes**, **real-time**, **background jobs**, **reliability**, and **split reads & writes**.
- The visuals show **batching** for writes, **WebSockets** and **server-sent events (SSE)** for live updates, and separate paths for reads and writes.
- The clip frames scaling as choosing a solution based on the traffic and data pattern, not adopting every trendy infrastructure component.

### Implementation guidance (synthesis)
- **Read pressure:** profile queries, indexes, caching and read replicas; budget for replica lag.
- **Write pressure:** batching, queues, partitioning and reducing unnecessary writes; preserve integrity and latency requirements.
- **Real-time:** WebSockets for bidirectional communication; SSE for one-way server push; design authorization, heartbeats and reconnect behavior.
- **Background jobs:** use durable queues when task completion must survive restarts; make workers idempotent.
- **Reliability:** bound timeouts, retries with jitter, circuit breakers where appropriate, and backpressure.
- **Separate reads/writes:** recognize CQRS-style separation can increase synchronization complexity and stale-read risk.

## 17. Optimize mobile-app startup on weak connections

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

## 18. Distribute high-volume location updates efficiently

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

## 19. Secure payment webhooks and make fulfillment retry-safe

**Source:** @peterstewiestartup — 2026-08-27 — https://www.instagram.com/reel/DcjNj5ZijOI/

### Lessons extracted
- A successful browser checkout or redirect is **not** reliable confirmation that the backend recorded the payment: the browser can close, disconnect, or visit a success URL without paying.
- The provider posts a server-to-server **webhook** to notify your backend of relevant payment events.
- **Verify the webhook signature** with the provider's specified signing method and secret; do not trust arbitrary HTTP POSTs.
- Retries and duplicates are normal. Persist a unique provider event identifier and make the resulting business effect idempotent.
- Record the event and associated authoritative state change atomically when they share a database; use an outbox or recoverable state machine for downstream work.
- Acknowledge quickly **after durable acceptance**; do not return `200` when the database failed before the event was recorded.
- The thank-you page can report **processing** and poll your own backend for the verified payment status, but must not grant access itself.

### Implementation guidance (synthesis)
1. Validate authentic signature against the **raw request body**, respecting the payment provider's specification and timestamp tolerance.
2. Record event ID, payment intent/object ID and processing state with unique constraints; protect state transitions against out-of-order events.
3. Process entitlements using authoritative provider status, amount, currency and order identity; reconcile periodically against the provider.
4. On durable acceptance, return the provider-expected success status; retry worker processing when needed.
5. Fail with a retryable HTTP error if the event was not durably received; alert on repeated failures and dead-lettered events.

**Caution:** Provider delivery is generally *at least once*, not truly exactly-once. Idempotent application effects are the objective.

## 20. Improve frontend performance and Core Web Vitals

**Source:** @yatesvids — 2026-10-08 — https://www.instagram.com/reel/DeNoh27h-j8/

### On-screen checklist from sampled frames
- Compress images and convert to modern formats.
- Lazy-load below-the-fold media.
- Cache static files.
- Remove unused code and stop loading unused scripts.
- Reduce shipped JavaScript and optimize fonts.
- Improve largest contentful paint (**LCP**).
- Reduce interaction delays and fix layout shifts.
- Run Lighthouse checks, fix measured issues, and test with a slow connection.

### Implementation guidance (synthesis)
- Serve right-sized AVIF/WebP with responsive `srcset`; retain useful format fallbacks.
- Do **not** lazy-load the primary above-the-fold/LCP image; prioritize it when appropriate.
- Fingerprint static assets for long-lived immutable caches while setting short/controlled policies for HTML.
- Split large bundles, tree-shake unused modules, defer noncritical scripts, and reduce main-thread long tasks.
- Set explicit image dimensions to mitigate cumulative layout shift (**CLS**); preload critical fonts carefully and use appropriate `font-display`.
- Measure field metrics (LCP, **INP**, CLS), not Lighthouse scores alone; test under slow mobile CPU/network conditions.

## 21. Distribute cache keys with consistent hashing

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

## Appendix: New source descriptions and transcription notes

The following additions are based on sampled *visual frames* and/or the supplied `.txt` files. **They should not be mistaken for complete voice transcriptions**; no speech-to-text transcript was supplied for the 2026-10-15, -10-16 or -10-17 clips. The 2026-10-14 clip supplies only the caption “Status code 🤔”. The 2026-10-08 performance clip's `.txt` file is only a brief promotional caption. Where source content is thin, the practical guidance is explicitly labeled synthesis.

### Source 14: `backend_medbouhezza_2026-10-13_DecFoqVicbm.txt`

```text
AI can write the code. Who decides which tradeoffs your customers can live with?

Across this series, one app grew through real failures: traffic spikes, slow queries, duplicate charges, lost messages, and too much data. Each fix solved a problem—and introduced a cost.

In the finale of How Senior Engineers Actually Think About System Design, Mohamed brings those pieces together. Senior engineering means understanding what breaks, choosing the next fix, and judging the effects on latency, reliability, operations, and the customer experience.

Look at your current project: what breaks first? Drop your answer below—and tell me what the next series should cover.

Save this series and follow for more practical software engineering.

#SystemDesign #SoftwareEngineering #EngineeringTradeoffs #BackendDevelopment #AI
```

### Source 15: `backend_medbouhezza_2026-10-14_DeehBXKERyM.txt`

```text
Status code 🤔
```

### Source 19: `backend_peterstewiestartup_2026-08-27_DcjNj5ZijOI.txt`

```text
A customer pays. Your server has no idea.

That's not a bug. That's how checkout actually works.

The money moves between the browser and Stripe's servers. Your backend isn't part of that conversation. It only finds out afterwards, when the browser redirects back to your site.

Two separate steps. And anything can break between them.

Tab closed. Connection dropped. Redirect failed.

They're charged either way. Your server just never finds out.

So the provider sends your backend a webhook instead. Provider to server. Directly. No browser involved.

But receiving an HTTP request is the easy part. Here's what actually makes it work:

→ Don't trust it. Anyone on the internet can POST to that URL. Verify the signature with your webhook secret. If it fails, reject it.

→ Be idempotent. Providers send the same event more than once. Every event has a unique ID — already in your database? Return success, do nothing.

→ Store that ID in the same transaction as the work. Crash between the two and you've lost the guarantee.

→ Respond fast. Verify, save what matters, queue the heavy work, return 200. Background workers handle the rest.

→ But never return 200 for work you didn't do. If your database was down and nothing got recorded, the provider will think it worked and stop retrying. Return an error and let it retry.

Final rule: the webhook is the source of truth. Not the redirect. Not the front end.

Because the browser belongs to the user — and that cuts both ways. The redirect can fail when the payment succeeded. And anyone can visit your success URL when no payment happened at all.

So what does the success page do? Show "processing" and poll your own API.

The webhook grants access. The page just reports it.

The hard part isn't receiving the request. It's making sure it's authentic, processed exactly once, and never leaves your payment state out of sync.

Which one of these has caught you out? 👇

#systemdesign #backenddevelopment #webdev #stripe #softwareengineering
```

### Source 20: `backend_yatesvids_2026-10-08_DeNoh27h-j8.txt`

```text
Hey Claude, my website is taking forever to load!

#vibecoding #vibecode #claude
```
