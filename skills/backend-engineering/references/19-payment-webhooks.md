# Secure payment webhooks and make fulfillment retry-safe

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

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
