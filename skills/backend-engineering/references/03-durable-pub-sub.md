# Decouple services using durable pub/sub and idempotent consumers

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

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
