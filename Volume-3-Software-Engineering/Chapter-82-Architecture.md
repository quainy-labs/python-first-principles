# Chapter 82 — Architecture

Architecture is design at system scale.

Design patterns help organize recurring structures.

SOLID helps reason about local design pressure.

Architecture asks a larger question:

```text
how should the whole system be shaped so it can change, run, and be understood?
```

Architecture is not only folders.

It is not only diagrams.

It is not only frameworks.

It is not only cloud infrastructure.

Architecture is the set of important decisions that shape a system:

* what belongs where
* what depends on what
* how data flows
* where business rules live
* where external systems are isolated
* where transactions begin and end
* how configuration enters
* how errors are handled
* how modules are deployed
* how the system is tested
* how the system is observed
* how change is contained

Good architecture makes important change easier.

Bad architecture makes every change feel like surgery with the lights off.

The goal of this chapter is not to sell one architecture.

The goal is to teach architectural judgment in Python.

---

# Architecture Is About Tradeoffs

There is no architecture that is best for every system.

A small CLI tool does not need the same structure as a payment platform.

A notebook experiment does not need the same boundaries as a public library.

A single-service internal app does not need the same deployment shape as a distributed system with hundreds of teams.

Architecture is tradeoff management.

Every choice gives something and costs something.

Layering improves separation, but adds indirection.

Microservices improve independent deployment, but add network and operational complexity.

Strong domain boundaries improve maintainability, but require more upfront modeling.

Simple scripts are fast to write, but can become tangled if they grow unchecked.

Architectural maturity means asking:

```text
what complexity does this system actually have?
what complexity are we introducing to manage it?
is the cure smaller than the disease?
```

---

# Architecture Starts With Change

A useful architectural question is:

```text
what changes independently?
```

Examples:

* business rules change
* database schema changes
* API contracts change
* UI changes
* payment provider changes
* deployment environment changes
* authentication rules change
* reporting needs change
* performance needs change
* team ownership changes

Architecture should separate things that change for different reasons.

This is SRP at system scale.

If payment-provider code is mixed with checkout policy, changing providers risks breaking checkout rules.

If API request parsing is mixed with domain logic, changing the API shape risks changing core behavior.

If database rows are used everywhere as domain objects, schema changes ripple through the system.

Architecture tries to make important change local.

---

# Architecture Is Not Folder Decoration

A project can have impressive folders and poor architecture.

Example:

```text
app/
    domain/
    services/
    repositories/
    infrastructure/
```

This looks architectural.

But if every module imports every other module freely, the folders are only decoration.

Architecture is dependency direction, not just directory names.

Ask:

```text
can domain logic run without the web framework?
can services be tested without the database?
does infrastructure depend on application policy, or does policy depend on infrastructure details?
can I replace an external provider without editing core rules?
```

If the answer is no, the architecture may not match the folder structure.

Folders should reveal boundaries that actually exist.

---

# The Simplest Architecture

The simplest useful architecture is direct code.

For a tiny script:

```text
script.py
```

This can be enough.

For a small CLI:

```text
file_organizer/
    __init__.py
    cli.py
    scanner.py
    rules.py
```

This can be enough.

Do not start every project with enterprise layers.

Start with clarity.

Add boundaries when the code shows real pressure.

The danger is not simplicity.

The danger is accidental growth without structure.

Simple architecture is good when it is intentional.

Tangled architecture is bad when it is accidental.

---

# A Common Python Application Shape

A medium-sized Python application may use a structure like:

```text
shop/
    __init__.py
    main.py
    config.py
    api/
        orders.py
        users.py
    application/
        checkout.py
        register_user.py
    domain/
        orders.py
        users.py
        money.py
    infrastructure/
        database.py
        email.py
        payments.py
    tests/
```

This is not mandatory.

It expresses one common idea:

```text
API layer receives requests
application layer coordinates use cases
domain layer holds business concepts and rules
infrastructure layer talks to outside systems
```

The important part is not the exact names.

The important part is the dependency direction.

Core rules should not be trapped inside framework and infrastructure details.

---

# Layers

Layering separates responsibilities by level.

A common layered model:

```text
presentation/API layer
application/service layer
domain layer
infrastructure layer
```

The API layer handles transport:

* HTTP requests
* CLI arguments
* message payloads
* serialization
* authentication context

The application layer handles workflows:

* create user
* checkout order
* import file
* generate report

The domain layer handles business concepts:

* order
* invoice
* payment
* user
* permission
* money

The infrastructure layer handles details:

* databases
* email providers
* payment SDKs
* filesystems
* queues
* caches
* external APIs

Layers help when each layer has a clear reason to exist.

They hurt when they only pass data through without adding meaning.

---

# Dependency Direction

Dependency direction is central to architecture.

One useful direction:

```text
API -> application -> domain
infrastructure -> domain
application -> abstractions
infrastructure implements abstractions
```

The domain should usually not import the web framework.

The domain should usually not know about database sessions.

The domain should usually not know about HTTP response objects.

Example bad domain:

```python
from flask import request


class Order:
    def apply_current_user_discount(self):
        user_id = request.headers["X-User-ID"]
        ...
```

Now `Order` depends on Flask request state.

That makes domain logic hard to test and reuse.

Better:

```python
class Order:
    def apply_discount(self, discount: Discount) -> None:
        ...
```

The API layer extracts request information.

The application layer decides what discount applies.

The domain object applies the rule.

---

# Domain Layer

The domain layer contains the language and rules of the problem.

Examples:

* `Order`
* `Invoice`
* `Payment`
* `Subscription`
* `Money`
* `Permission`
* `User`
* `Policy`

Domain code should be understandable without knowing how HTTP, SQL, or queues work.

Example:

```python
@dataclass
class Order:
    id: OrderId
    status: OrderStatus
    total: Money
    payment_id: PaymentId | None = None

    def mark_paid(self, payment_id: PaymentId) -> None:
        if self.status != OrderStatus.PENDING:
            raise InvalidOrderState("only pending orders can be paid")
        self.status = OrderStatus.PAID
        self.payment_id = payment_id
```

This rule is domain logic.

It should not be hidden in a route handler or database trigger unless there is a strong reason.

Domain logic is the heart of many applications.

Protect it from accidental infrastructure coupling.

---

# Application Layer

The application layer coordinates use cases.

Example:

```python
class Checkout:
    def __init__(self, orders, payments, unit_of_work):
        self.orders = orders
        self.payments = payments
        self.unit_of_work = unit_of_work

    def execute(self, order_id: OrderId) -> Receipt:
        with self.unit_of_work:
            order = self.orders.get(order_id)
            payment = self.payments.charge(order)
            order.mark_paid(payment.id)
            self.orders.save(order)
            return Receipt(order_id=order.id, payment_id=payment.id)
```

The application layer does not implement the payment SDK.

It does not parse HTTP JSON.

It coordinates the workflow:

```text
load order
charge payment
change domain state
save changes
return result
```

This layer is often the best place for transaction boundaries, authorization checks, idempotency decisions, and orchestration.

---

# Infrastructure Layer

Infrastructure code talks to the outside world.

Examples:

* SQL repositories
* Redis caches
* S3 storage
* SMTP email sender
* Stripe payment adapter
* HTTP clients
* message queue publishers

Infrastructure code is allowed to know ugly details.

Example:

```python
class StripePaymentGateway:
    def __init__(self, client):
        self.client = client

    def charge(self, order: Order) -> PaymentResult:
        response = self.client.PaymentIntent.create(
            amount=order.total.cents,
            currency=order.total.currency.lower(),
            metadata={"order_id": str(order.id)},
        )
        return PaymentResult(id=PaymentId(response.id))
```

The adapter translates between your application concepts and the provider's API.

The ugliness stays here.

That is the point.

---

# API Layer

The API layer translates between external requests and application use cases.

Example:

```python
def checkout_endpoint(request):
    payload = request.json()
    order_id = OrderId(payload["order_id"])

    receipt = checkout.execute(order_id)

    return {
        "order_id": str(receipt.order_id),
        "payment_id": str(receipt.payment_id),
    }
```

The API layer handles:

* request parsing
* response formatting
* status codes
* authentication extraction
* input validation
* transport-specific errors

It should not contain complex business rules.

If a route handler becomes long and full of domain decisions, the architecture is drifting.

Move workflow into application services and rules into domain code.

---

# Data Flow

Architecture should make data flow understandable.

Example HTTP flow:

```text
HTTP request
    -> API schema validation
    -> application command
    -> domain objects
    -> repositories/gateways
    -> result object
    -> HTTP response
```

Each boundary may transform data.

Raw JSON is not the same as a domain object.

A database row is not always the same as a domain object.

An external API response is not always trusted internal data.

Good architecture makes transformations explicit.

Bad architecture lets raw dictionaries, ORM rows, and provider responses flow everywhere.

That creates confusion.

Ask:

```text
what shape is this data here?
is it trusted?
who owns this representation?
```

---

# Boundaries

A boundary separates two parts of a system.

Useful boundaries include:

* web framework vs application
* application vs domain
* domain vs persistence
* application vs external service
* internal model vs public API model
* command-line interface vs core logic
* synchronous code vs asynchronous code
* trusted code vs untrusted input

Boundaries are where translation happens.

They are also where validation often happens.

Example:

```text
external JSON -> validated command object -> domain operation
```

Do not let untrusted data cross deeply into the system.

Validate and translate near the boundary.

Then use typed, meaningful internal objects.

---

# Transaction Boundaries

Architecture should define transaction boundaries.

Example:

```text
create order
reserve inventory
charge payment
mark order paid
send receipt
```

Which of these happen in one database transaction?

Which happen after commit?

What happens if payment succeeds but saving fails?

What happens if email fails?

These are architectural questions.

A Unit of Work can help:

```python
with uow:
    order = orders.get(order_id)
    payment = payments.charge(order)
    order.mark_paid(payment.id)
    orders.save(order)
```

But external calls inside transactions require care.

Holding a database transaction while waiting for a payment API may be risky.

Architecture must consider consistency, failure, and time.

---

# Consistency

Consistency means the system's data obeys expected rules.

Examples:

* paid orders have payment IDs
* inventory cannot go below zero
* users cannot belong to another tenant's account
* invoice totals match invoice lines
* deleted records cannot be active

Some consistency is enforced in code.

Some belongs in the database.

Some spans multiple systems and cannot be immediate.

Architecture should decide where invariants are protected.

Database constraints are excellent for data integrity.

Domain logic is excellent for business rules.

Application workflows are excellent for coordinating multi-step use cases.

Do not rely on only one layer for every kind of consistency.

---

# Configuration

Configuration enters the system from the outside.

Examples:

* environment variables
* config files
* command-line flags
* secret managers
* feature flags
* deployment settings

Bad architecture lets configuration access spread everywhere:

```python
def charge(order):
    api_key = os.environ["STRIPE_KEY"]
    ...
```

Better:

```python
@dataclass(frozen=True)
class Settings:
    stripe_key: str
    database_url: str
    log_level: str
```

Load settings at startup.

Pass configuration into factories and services.

This makes configuration visible, testable, and safer to log in redacted form.

---

# Composition Root

The composition root is where the application wires concrete objects together.

Example:

```python
def build_app(settings: Settings) -> App:
    session_factory = create_session_factory(settings.database_url)
    payments = StripePaymentGateway(create_stripe_client(settings.stripe_key))
    orders = SqlOrderRepository(session_factory)
    checkout = Checkout(orders=orders, payments=payments)
    return App(checkout=checkout)
```

This function knows concrete details.

That is fine.

The rest of the system can depend on abstractions or narrow capabilities.

The composition root is the controlled place where infrastructure meets application policy.

Without a composition root, construction logic often leaks everywhere.

---

# Modular Monolith

A modular monolith is one deployable application with strong internal module boundaries.

It may look like:

```text
shop/
    orders/
    payments/
    inventory/
    users/
    notifications/
```

Each module owns its concepts and exposes clear APIs to other modules.

The whole system deploys together.

This can be a very strong architecture.

It avoids distributed-system complexity while still keeping code organized.

A modular monolith is not a failure.

For many teams, it is the best starting point.

Microservices can come later if independent deployment and scaling needs are real.

---

# Layered Architecture

Layered architecture organizes code by technical responsibility.

Example:

```text
api/
services/
domain/
repositories/
infrastructure/
```

This is easy to understand at first.

It works well when the application is small or the domain is not too complex.

But as features grow, purely technical layers can scatter one business feature across many folders.

To understand checkout, you may need:

```text
api/orders.py
services/checkout.py
domain/order.py
repositories/orders.py
infrastructure/payments.py
```

That may be fine.

But if feature cohesion matters more, package-by-feature may be better.

Architecture is choice.

---

# Package by Feature

Package-by-feature organizes around business areas.

Example:

```text
shop/
    orders/
        api.py
        service.py
        domain.py
        repository.py
    payments/
        api.py
        service.py
        gateway.py
    users/
        api.py
        service.py
        domain.py
```

This keeps related code closer.

It can improve ownership and navigation.

The risk is duplication of infrastructure patterns across features.

Package-by-feature works well when business domains are strong and teams think in feature areas.

Package-by-layer works well when technical separation matters more.

Many real systems mix both.

Do not argue from aesthetics alone.

Choose based on change patterns.

---

# Hexagonal Architecture

Hexagonal architecture is also called ports and adapters.

The idea:

```text
core application defines ports
adapters connect external systems to those ports
```

Port:

```python
class PaymentGateway(Protocol):
    def charge(self, order: Order) -> PaymentResult:
        ...
```

Adapter:

```python
class StripePaymentGateway:
    def charge(self, order: Order) -> PaymentResult:
        ...
```

The application depends on the port.

Infrastructure implements the port.

This supports testing, replacement, and clearer boundaries.

Hexagonal architecture is powerful when external systems are important and changeable.

It can be overkill for tiny apps.

---

# Clean Architecture

Clean Architecture emphasizes dependency direction toward inner policy.

Outer layers contain details:

* web
* database
* frameworks
* external services

Inner layers contain policy:

* use cases
* domain rules
* entities

Dependency direction points inward.

Python can express this without heavy ceremony.

Example:

```text
api -> application -> domain
infrastructure -> application ports
```

The useful idea is:

```text
business policy should not depend on replaceable details
```

Do not obsess over diagrams.

Use the dependency rule where it improves testability and change.

---

# Event-Driven Architecture

Event-driven architecture uses events to communicate that something happened.

Example events:

```text
UserCreated
OrderPaid
InvoiceGenerated
PasswordResetRequested
InventoryLow
```

An event can trigger multiple reactions.

Example:

```text
OrderPaid
    -> send receipt
    -> update analytics
    -> schedule shipment
    -> notify warehouse
```

Events decouple producers from consumers.

They also make flow harder to follow.

Event-driven design is useful when reactions are independent or asynchronous.

It is harmful when it hides straightforward workflows.

Use events for real decoupling.

Do not use events to avoid function calls.

---

# Architecture and Testing

Good architecture makes testing easier.

Examples:

* domain rules test without database
* application services test with fakes
* infrastructure adapters test with integration tests
* API layer tests focus on request/response behavior
* contract tests verify fake and real implementations

If every test requires the full application, database, network, and framework, boundaries are weak.

If every unit test mocks ten internal functions, boundaries may be wrong or overcomplicated.

Testing pressure reveals architecture.

Ask:

```text
what makes this hard to test?
hidden dependencies?
mixed responsibilities?
external systems?
unclear data shapes?
```

The answer often points to architectural improvement.

---

# Architecture and Observability

Architecture should include observability.

Not as an afterthought.

Systems need:

* logs
* metrics
* traces
* health checks
* error reporting
* audit events
* correlation IDs

Where do request IDs enter?

How are they passed?

Where are errors logged?

Which layer records business events?

Which layer records infrastructure failures?

If observability is not designed, teams add random logs during incidents.

That creates noise.

Good architecture gives operational evidence a place.

---

# Architecture and Error Handling

Error handling is architectural.

Decide:

* which exceptions are domain errors?
* which are infrastructure errors?
* where are errors translated to HTTP responses?
* where are retries allowed?
* where are errors logged?
* which errors are expected business outcomes?
* which errors should fail fast?

Example:

```python
class PaymentDeclined(Exception):
    ...


class PaymentProviderUnavailable(Exception):
    ...
```

These are different.

A declined card may be a normal business result.

A provider outage may require retry, alerting, or fallback.

Architecture should preserve these distinctions.

Do not collapse every failure into `Exception`.

---

# Architecture and Security

Security belongs in architecture.

Questions:

* where is authentication handled?
* where is authorization enforced?
* how is tenant isolation protected?
* where are secrets loaded?
* who can access logs?
* what data crosses boundaries?
* where is input validated?
* where are audit events recorded?
* how are external calls authenticated?

Security is not one decorator on a route.

It is a set of boundaries and invariants.

For example, tenant checks should not depend only on UI behavior.

The backend must enforce them.

Architecture should make dangerous bypasses difficult.

---

# Architecture and Performance

Performance architecture includes:

* caching boundaries
* database access patterns
* async vs sync decisions
* batching
* queueing
* data streaming
* read/write separation
* background jobs
* service boundaries

Do not design for imaginary scale.

But do avoid obvious traps.

Example:

```text
API endpoint loops over 1,000 rows and calls external API for each one
```

This is an architectural performance issue.

No small syntax optimization will fix it.

Profiling identifies bottlenecks.

Architecture determines which solutions are available.

---

# Architecture and Deployment

Deployment shape affects architecture.

Questions:

* is this a CLI, web app, worker, library, or service?
* does one process run everything?
* are workers separate from web servers?
* how are migrations run?
* how are background jobs scheduled?
* how are secrets supplied?
* how are logs collected?
* how are health checks exposed?
* how is configuration changed?

A Python project may contain multiple entry points:

```text
web server
background worker
CLI management command
scheduled job
```

They can share domain and application code.

They may have different infrastructure wiring.

Architecture should make entry points explicit.

---

# Architecture and Teams

Architecture is partly social.

Module boundaries often reflect team boundaries.

If many people work in the same files, conflicts increase.

If ownership is unclear, changes become risky.

A good architecture helps teams answer:

* who owns this module?
* who can change this API?
* where should a new feature go?
* what contracts exist between teams?
* how are shared models versioned?

Technical structure and collaboration structure influence each other.

This matters more as teams grow.

For a solo project, architecture optimizes comprehension.

For a team project, it also optimizes coordination.

---

# Avoid Circular Dependencies

Circular dependencies are often architecture smells.

Example:

```text
orders imports payments
payments imports orders
```

Sometimes circular imports are technical accidents.

Often they reveal unclear ownership.

Ask:

* which module owns the concept?
* should shared types move to another module?
* should communication happen through events?
* should one dependency be inverted through a protocol?
* is a service layer needed?

Python circular imports can fail at runtime because modules are partially initialized.

But even when they work, circular dependencies make systems harder to reason about.

Prefer directed dependencies.

---

# Public Module APIs

Modules should expose intentional APIs.

Example:

```python
# orders/__init__.py
from .service import Checkout
from .models import Order, OrderId


__all__ = ["Checkout", "Order", "OrderId"]
```

This tells users:

```text
these are the intended public names
```

Internal modules can still exist:

```text
orders/_pricing.py
orders/_mapping.py
```

Python cannot fully enforce privacy.

But naming and exports communicate boundaries.

Architecture depends on communication.

If everything is public, boundaries blur.

---

# Shared Kernel

A shared kernel is a small set of shared concepts used across modules.

Examples:

* `Money`
* `UserId`
* `TenantId`
* common error types
* date/time utilities

Shared code is useful.

Shared code is also coupling.

Do not create a giant `common` package that becomes a dumping ground.

Bad:

```text
common/
    utils.py
    helpers.py
    stuff.py
```

Better:

```text
shared/
    money.py
    identifiers.py
    time.py
```

Shared concepts should be stable and genuinely shared.

If only one module uses it, keep it there.

---

# Anti-Corruption Layer

An anti-corruption layer protects your model from an external or legacy model.

Suppose a legacy API returns:

```json
{
  "cust_no": "123",
  "stat": "A",
  "bal": "100.50"
}
```

Do not let these names spread everywhere.

Translate at the boundary:

```python
def legacy_customer_to_domain(payload: dict[str, str]) -> Customer:
    return Customer(
        id=CustomerId(payload["cust_no"]),
        status=parse_status(payload["stat"]),
        balance=Money(payload["bal"]),
    )
```

Now the rest of your system speaks your language.

External systems have their own models.

Architecture should prevent them from corrupting yours.

---

# Architecture Smells

Common architecture smells include:

* route handlers with business logic
* domain objects importing web frameworks
* database sessions used everywhere
* external SDK objects passed deep into domain code
* raw dictionaries flowing through the whole system
* circular imports
* `utils.py` growing endlessly
* tests requiring too much infrastructure
* every change touching many modules
* configuration read from environment everywhere
* errors collapsed into generic exceptions
* no clear transaction boundaries
* logs without correlation across layers

Smells are not automatic crimes.

They are signals.

Investigate the pressure behind them.

---

# Architecture Documentation

Architecture should be documented lightly but clearly.

Useful documents include:

* project structure overview
* dependency rules
* system context diagram
* key flows
* external integrations
* deployment shape
* important decisions
* tradeoffs

Architecture Decision Records, or ADRs, are one way to record important decisions.

An ADR usually states:

* context
* decision
* consequences

Example:

```text
Decision: Use a modular monolith rather than microservices.
Context: Team is small, domain boundaries still evolving, deployment simplicity matters.
Consequences: Modules must maintain clear internal boundaries; services can be extracted later if needed.
```

Documentation should help people make future changes.

It should not become a museum.

---

# Architecture Evolves

Architecture is not frozen.

A project may start as:

```text
single script
```

then become:

```text
package with modules
```

then:

```text
web app with services
```

then:

```text
modular monolith
```

then maybe:

```text
multiple services
```

Each step should respond to real pressure.

Do not jump to the final imagined architecture too early.

Also do not ignore growth until the system becomes unchangeable.

Architecture evolves through refactoring, not prophecy.

---

# Choosing Architecture by Size

For a small script:

```text
clear functions
simple CLI
tests for core logic
```

For a small package:

```text
src layout
public API
tests
minimal internal modules
```

For a medium application:

```text
API/application/domain/infrastructure boundaries
configuration object
explicit dependencies
integration tests
logging and metrics
```

For a large system:

```text
module ownership
strict dependency rules
well-defined contracts
observability
deployment strategy
data migration strategy
security boundaries
possibly service boundaries
```

Match architecture to the system you have and the system you are realistically becoming.

---

# Architecture Checklist

When reviewing architecture, ask:

* Where does business logic live?
* What depends on what?
* Are external systems isolated?
* Are transaction boundaries clear?
* Are data transformations explicit?
* Are configuration and secrets centralized?
* Can core logic be tested without infrastructure?
* Are module boundaries meaningful?
* Are public APIs intentional?
* Are errors represented clearly?
* Is observability designed?
* Are security boundaries enforced?
* Does deployment shape match code structure?
* Can the system grow without every change touching everything?

This checklist does not produce architecture automatically.

It starts the right conversations.

---

# Chapter Summary

Architecture is design at system scale.

It decides how modules, layers, boundaries, dependencies, data flow, transactions, configuration, deployment, observability, security, and ownership fit together.

Architecture is not folder decoration.

Real architecture is dependency direction and boundary discipline.

The simplest architecture that supports the current system is usually best.

Add structure when change pressure justifies it.

Common Python architectures include simple scripts, layered applications, package-by-feature layouts, modular monoliths, hexagonal architecture, clean architecture, and event-driven systems.

API layers should translate external requests.

Application layers should coordinate use cases.

Domain layers should hold business concepts and rules.

Infrastructure layers should isolate databases, external services, filesystems, queues, caches, and provider SDKs.

Data should be validated and translated at boundaries.

Transaction boundaries should be explicit.

Configuration should be loaded centrally and passed into construction.

Composition roots wire concrete dependencies together.

Architecture should support testing, observability, security, performance, deployment, and team coordination.

Circular dependencies, giant utility modules, raw dictionaries everywhere, hidden configuration, and framework-coupled domain code are common smells.

Architecture evolves as systems grow.

The central lesson is:

```text
architecture makes change local and system behavior understandable
```

A good architecture does not make code impressive.

It makes code livable.

---

# Exercises

1. Draw the current module structure of a Python project and mark dependency directions.

2. Identify where business logic lives in a web endpoint and move it into an application service.

3. Create a small domain object that has no dependency on a web framework or database.

4. Write a repository protocol and one infrastructure implementation.

5. Create a composition root that wires settings, repositories, gateways, and services.

6. Refactor raw external API data into a domain object at the boundary.

7. Find a circular import and redesign the dependency direction.

8. Compare package-by-layer and package-by-feature for one project.

9. Write an ADR for one architectural decision.

10. Add a request ID flow through API, application, and logging layers.

---

# Preview of Chapter 83

Chapter 82 studied architecture.

We zoomed out from local design principles to system structure: layers, dependency direction, domain boundaries, application services, infrastructure adapters, data flow, transactions, configuration, composition roots, modular monoliths, event-driven systems, observability, security, deployment, and team ownership.

Next we study APIs.

APIs are contracts between software components.

They can be:

* function APIs
* class APIs
* module APIs
* package APIs
* HTTP APIs
* CLI APIs
* event APIs
* internal service APIs

Architecture defines boundaries.

APIs define how code crosses those boundaries.

The transition is:

```text
architecture decides where boundaries exist
APIs decide how those boundaries are used
```

Chapter 83 will explain how to design clear, stable, versioned, documented, and testable APIs in Python systems.
