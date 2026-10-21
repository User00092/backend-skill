# Debounce search requests and cancel stale requests

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

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
