# Apply the SOLID principles

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

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
