# Use a transactional outbox for reliable messages

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

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
