# Modular Monolith: Boundaries, Contracts, and Extraction

## When to use

Read this when designing a new backend, untangling a tightly coupled monolith, deciding whether to adopt microservices, or defining dependencies among modules such as identity, orders, and payments.

## What the supplied video teaches

**Source:** `backend_medbouhezza_2026-10-20_DeuqLfoRAmp.mp4` (Mohamed Bouhezza, October 20, 2026). These are observations from sampled video frames, **not** a verbatim audio transcript.

- A normal monolith places login, orders, and payments inside one application; poorly controlled dependencies let modules access each other's internals.
- A **modular monolith** is still deployed as one application but has meaningful internal boundaries.
- Each module exposes a small **front door** or public API while keeping implementation details private.
- The video shows orders interacting with payments by **function call** or by **event**, depending on the contract.
- It contrasts an internal module boundary with extracting payments into a separate, separately deployed microservice.
- Illustrated examples mention Java/ArchUnit, .NET's `internal` visibility, Go's `internal/` packages, and Shopify's Packwerk. These are examples, not guarantees of enforcement in every configuration.

## Recommended architecture (additional engineering guidance)

```text
app/
  modules/
    identity/
      public/        # exported contracts and DTOs
      internal/      # business logic, persistence, integrations
    orders/
      public/
      internal/
    payments/
      public/
      internal/
  platform/          # logging, metrics, transport, infrastructure adapters
  composition/       # bootstrap and dependency wiring
```

- Treat modules as **business capabilities**, not merely folders for controllers and repositories.
- Public contracts should be intentionally small and versionable. Keep private classes, tables, and repositories inaccessible to other modules by convention **and** automated architectural tests.
- Prefer dependency injection and explicit interfaces. Avoid a global service locator, shared mutable state, or a generic `utils` module that allows bypassing boundaries.
- Document allowed dependency directions and enforce **no dependency cycles** in CI.
- A single physical database is compatible with a modular monolith, but do not let modules write directly to each other's tables. Prefer module-owned repository/service contracts and explicit cross-module workflows. Separate schemas may help enforcement, but are not automatically required.
- **Synchronous calls** fit immediate answers or invariants needing confirmation; **events** fit deferred reactions and loosely coupled workflows. In-process events are not durable by themselves. For reliable events spanning commits/crashes, use the [transactional outbox](11-transactional-outbox.md) and [durable pub/sub](03-durable-pub-sub.md) guidance.
- Avoid distributed transactions and microservices until independent scaling, deployment cadence, ownership, or isolation justifies their operational cost.

## Implementation checklist

1. Define capabilities and module ownership (including data and side effects).
2. List each module's public commands, queries, domain events, and error contracts.
3. Deny imports of another module's internals; test dependency rules in CI.
4. Require all writes to owned data to pass through the owning module.
5. Make cross-module actions observable with metrics, tracing, and structured logs.
6. Specify which events are best-effort versus durable and retryable.
7. Add contract tests, integration tests, and failure-path tests (especially payments).
8. Benchmark real bottlenecks before extracting a service.

## When to extract a microservice

Consider extraction when a module has a stable public boundary and a concrete need for independently managed scaling, deployment, resilience, compliance, or team ownership. Confirm the team can support network timeouts, distributed tracing, separate data ownership, backward-compatible interfaces, and incident response. Extraction trades local calls for distributed failure modes.

## Review red flags

- `orders` directly imports `payments/internal/*` or writes payments tables.
- A change to a private module detail forces edits throughout the repository.
- In-process events are treated as guaranteed delivery after a crash.
- The plan claims a modular monolith provides independent deployments or hard fault isolation; it does not.
- Splitting services is proposed before module contracts and data ownership are clear.

## Related references

- [Failure-first design](08-failure-first-system-design.md)
- [Engineering tradeoffs](14-engineering-tradeoffs.md)
- [Transactional outbox](11-transactional-outbox.md)
- [Durable pub/sub](03-durable-pub-sub.md)
