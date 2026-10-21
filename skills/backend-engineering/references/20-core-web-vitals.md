# Improve frontend performance and Core Web Vitals

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

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
