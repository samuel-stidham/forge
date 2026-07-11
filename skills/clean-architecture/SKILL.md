---
name: clean-architecture
description: Reference for Clean Architecture layering, dependency direction, naming conventions, and per-layer testing strategy. Use when scaffolding new projects, reviewing architecture for layer violations, or designing/refactoring a layered system.
---

# Clean Architecture Reference

This skill is the shared source of truth for `/forge:scaffold` and `/forge:review`. It is language-agnostic and framework-agnostic. Apply the principles, not a specific folder name, when the target language's idioms call for something different.

## The four layers

### Domain

Entities, value objects, and business rules that stay true no matter how the software is delivered or stored. The domain layer has **zero external dependencies**. It has no framework imports, no database client, no HTTP library, and no logging framework. It depends on nothing but the language's standard library, plus maybe a tiny utility library used sparingly.

Importing a framework type, an ORM model, or an HTTP request object into domain code is a violation. The domain should not know these things exist.

### Application

Use cases, also called interactors or services, orchestrate domain objects to fulfill a specific action the system supports. Examples include "place an order" or "deactivate a user." This layer also defines **ports**. Ports are interfaces describing what the use case needs from the outside world, such as a repository, a notifier, or a clock. The use case does not know how those needs get fulfilled.

The application layer may depend on the domain layer. It must not depend on infrastructure or presentation. It defines interfaces. It does not implement them.

### Infrastructure

Concrete adapters that implement the ports the application layer defines. Examples include a Postgres repository implementing a `UserRepository` port and an SMTP client implementing a `Notifier` port. Another example is an HTTP client implementing a `PaymentGateway` port. This is where frameworks, drivers, SDKs, and external service clients live.

Infrastructure depends on application, to know which interfaces to implement. It also depends on domain, to know which types it is persisting, sending, or fetching. It is the most replaceable layer. Swapping Postgres for MySQL, or SES for SendGrid, should only touch this layer.

### Presentation

The entry point and composition root. This includes HTTP handlers and controllers, CLI command definitions, message consumers, and scheduled job triggers. This layer translates requests from the outside world into calls on application use cases. It also wires concrete infrastructure implementations into the use cases that need them. Dependency injection happens here, as close to the program's entry point as possible.

Presentation may depend on all three other layers, since it is where everything gets assembled. Business logic does not belong here. If a handler does more than parse input, call a use case, and format output, logic has leaked into the wrong layer.

## The dependency rule

Dependencies point **inward, always**. Presentation depends on Application, which depends on Domain. Infrastructure also depends on Application, which depends on Domain. Domain depends on nothing. Application depends only on Domain. Nothing inside Domain or Application may import from Infrastructure or Presentation.

This is enforced through **ports and adapters**. Inner layers define interfaces, called ports, for what they need. Outer layers provide implementations, called adapters, that satisfy those interfaces. The inner layer never imports the outer layer's concrete type. It imports its own interface instead, and the concrete type gets wired in at the composition root.

When you review a diff, the fastest way to spot a violation is to check imports. Does a file in `domain/` or `application/` import anything from `infrastructure/` or a framework package? That is the violation, regardless of what the code does.

## Interfaces vs. concrete types, and when to use each

Use an interface, or port, when any of the following apply.

- The application layer needs something from outside its boundary, such as persistence, network calls, time, randomness, or file I/O. These always need a port. The application layer cannot depend on the concrete implementation.
- You genuinely expect more than one implementation, such as swappable storage backends or multiple notification channels. You may also need a test double for unit testing a use case without hitting real infrastructure.

Use a concrete type when any of the following apply.

- The thing is internal to a single layer and has exactly one shape. Examples include a domain entity, a value object, or a DTO. Do not wrap a domain entity in an interface "for flexibility" it will never need. That is indirection without payoff.
- You are inside infrastructure or presentation, gluing things together. These layers can use concrete types from their own dependencies freely.

A common mistake is over-abstracting the domain layer, for example giving every entity an interface. Another is under-abstracting the application layer. An example is letting a use case call a concrete database client directly instead of through a port. Watch for both.

## Naming conventions

- Ports are named for the capability they provide, not the technology behind them. Use `UserRepository`, not `PostgresUserStore`. Use `Notifier`, not `SmtpClient`. The interface name should still make sense if the implementation changes.
- Use cases are named as verbs or actions, such as `PlaceOrder`, `DeactivateUser`, or `SendPasswordReset`. Avoid generic CRUD names like `UserService`, since they invite scope creep.
- Adapters are named for what they implement, plus the technology. Examples include `PostgresUserRepository`, `SendgridNotifier`, and `StripePaymentGateway`.
- Domain entities are named for the real-world concept they model, without technical suffixes. Use `Order`, `User`, or `Invoice`, not `OrderEntity`, `UserModel`, or `InvoiceDTO`. The exception is when the language or framework needs a suffix to tell two types apart.

## Testing strategy per layer

- **Domain**: pure unit tests. No mocks or test doubles are needed. Construct objects, call methods, and assert on the results or the state. If a domain test needs a mock, that is a sign domain logic has taken on a dependency it should not have.
- **Application**: unit tests with fake or mock implementations of the ports the use case depends on. Examples include an in-memory repository, a fake clock, or a spy notifier. Assert on the use case's behavior and its calls to ports, not on infrastructure details.
- **Infrastructure**: integration tests against the real thing where practical. Examples include a real test database run through Docker, or a sandbox API. This layer's entire job is talking correctly to that external system. Mocking the thing infrastructure is supposed to integrate with defeats the purpose of testing it.
- **Presentation**: integration or end-to-end tests that exercise the full request path. This means an HTTP request in and a response out, or a CLI invocation in and output out. Use real or test-double infrastructure, depending on the tradeoff between speed and cost.

A healthy test suite is mostly fast domain and application unit tests. It has a smaller number of slower infrastructure and presentation integration tests covering the seams.
