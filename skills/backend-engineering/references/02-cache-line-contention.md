# Reduce shared-counter contention and false sharing

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

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
