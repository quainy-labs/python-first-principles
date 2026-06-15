# Chapter 84 — Microservices

Microservices are an architectural style.

They split a system into independently deployable services that communicate through APIs.

That definition sounds simple.

The consequences are not simple.

A microservice is not just a small web app.

A microservice is a service with its own boundary:

* code
* API
* data ownership
* deployment
* monitoring
* failure modes
* scaling behavior
* operational responsibility

Microservices are powerful when the system needs independent deployment, independent scaling, fault isolation, and clear ownership across teams.

They are costly when used too early.

They turn local function calls into network calls.

They turn simple transactions into distributed coordination.

They turn ordinary debugging into cross-service investigation.

They make API design, logging, testing, observability, packaging, deployment, and architecture more important.

Microservices are not the next level after monoliths.

They are a tradeoff.

This chapter explains that tradeoff honestly.

---

# What a Microservice Is

A microservice is an independently deployable service focused around a business capability.

Examples:

* user service
* payment service
* inventory service
* notification service
* search service
* recommendation service
* billing service
* reporting service

Each service usually has:

* its own codebase or deployable unit
* its own runtime process
* its own API
* its own data ownership
* its own logs and metrics
* its own release cycle
* its own operational responsibilities

The important phrase is:

```text
business capability
```

Microservices should not be split only by technical layer.

This is usually not a good microservice split:

```text
controller service
business logic service
database service
```

That creates distributed layers.

It often increases latency and coupling without improving ownership.

Good services own meaningful capabilities.

---

# Microservices vs Monolith

A monolith is an application deployed as one unit.

That does not mean it must be messy.

A monolith can be modular, well-tested, observable, and maintainable.

A microservice system is deployed as multiple services.

That does not mean it is automatically clean.

A distributed mess is still a mess.

The real contrast is not:

```text
monolith bad
microservices good
```

The real contrast is:

```text
one deployable unit
multiple deployable units
```

Multiple deployable units create flexibility.

They also create coordination cost.

Do not choose microservices because they sound modern.

Choose them when their benefits outweigh their costs.

---

# The Modular Monolith

A modular monolith is one deployable application with strong internal module boundaries.

Example:

```text
shop/
    orders/
    payments/
    inventory/
    users/
    notifications/
```

Each module owns its concepts.

The system deploys together.

This is often the best architecture before microservices.

Why?

Because it gives many benefits of good design:

* clear boundaries
* cohesive modules
* easier testing
* simpler deployment
* easier local development
* simpler transactions
* easier debugging

without distributed-system costs.

If your modular monolith is tangled, splitting it into services will not automatically fix it.

It may distribute the tangles over the network.

Good microservices often begin as good modules.

---

# Why Teams Choose Microservices

Microservices can help when a system needs:

* independent deployment
* independent scaling
* independent team ownership
* fault isolation
* technology diversity
* clearer business boundaries
* high release velocity in different areas
* different data storage needs
* different performance profiles

Example:

The search system may need different scaling, indexing, and deployment behavior than checkout.

The notification system may process background messages independently from the web application.

The payment system may require stricter security and audit boundaries.

These are real reasons.

"The codebase is messy" is not enough by itself.

Messy code split into services often becomes messy services.

---

# The Cost of Microservices

Microservices add costs:

* network latency
* partial failure
* distributed tracing
* service discovery
* deployment orchestration
* API versioning
* data consistency problems
* duplicated infrastructure
* operational overhead
* local development complexity
* cross-service testing complexity
* security between services
* schema and contract coordination
* more logs, metrics, and alerts

In a monolith, this call:

```python
payment = payments.charge(order)
```

may be a function call.

In a microservice system, it may become:

```text
HTTP request or message publish
authentication
serialization
network transport
timeout handling
retry policy
remote service execution
response parsing
error mapping
logging
metrics
tracing
```

That is not free.

Distribution magnifies engineering discipline.

---

# Service Boundaries

The hardest microservice question is:

```text
where should service boundaries be?
```

Bad boundaries create constant coordination.

Good boundaries align with business capabilities and ownership.

Useful questions:

* who owns this data?
* who changes this behavior?
* who needs to deploy independently?
* which operations must be strongly consistent?
* which teams own which capabilities?
* what API would this service expose?
* can this service be useful without direct database access to another service?
* does this boundary reduce coupling or only move it?

A service boundary should reduce coordination inside a capability.

It should not create a remote call for every line of business logic.

---

# Data Ownership

In microservices, each service should own its data.

This is one of the biggest differences from modular monoliths.

Bad:

```text
orders service reads payments database tables directly
payments service reads orders database tables directly
```

This creates database-level coupling.

If the payments team changes its schema, orders may break.

Better:

```text
payments service owns payment data
orders service asks payments through an API or consumes payment events
```

Data ownership means:

```text
the service that owns the data controls how others access it
```

This keeps storage details private.

It also creates consistency challenges.

That is the tradeoff.

---

# Database Per Service

Microservice architecture often says each service should have its own database.

This does not always mean different database servers.

It means ownership boundary.

Possibilities:

* separate database server
* separate database
* separate schema
* separate tables with strict ownership

The principle:

```text
services should not casually share mutable storage
```

Shared databases create hidden coupling.

But separate databases make joins and transactions harder.

If your system still needs many cross-service joins to answer basic questions, your boundaries may be wrong.

Microservice data design is architectural design.

---

# Communication Styles

Microservices communicate in different ways.

Common styles:

* synchronous HTTP
* synchronous RPC
* asynchronous messaging
* event streams
* queues
* scheduled batch exchange

Synchronous request:

```text
orders -> payments -> response now
```

Asynchronous event:

```text
payments publishes PaymentCaptured
orders consumes it later
```

Synchronous calls are easier to reason about locally.

They create runtime coupling.

Asynchronous events reduce direct coupling.

They introduce eventual consistency and harder debugging.

Choose communication style based on workflow needs.

---

# Synchronous APIs

Synchronous APIs return a response immediately.

HTTP is common:

```http
POST /payments
```

Synchronous APIs are useful when the caller needs an immediate answer.

Examples:

* validate credentials
* authorize payment
* fetch user profile
* check inventory availability

But synchronous calls can fail:

* service unavailable
* timeout
* network error
* rate limit
* invalid response

Callers need timeouts.

Callers need error handling.

Callers need retry policies only when retrying is safe.

Every synchronous service call is a failure point.

---

# Asynchronous Messaging

Asynchronous messaging decouples producer and consumer in time.

Example:

```text
orders publishes OrderCreated
inventory reserves stock
notifications sends confirmation
analytics records event
```

The order service does not wait for every consumer.

This improves resilience and responsiveness.

But it introduces:

* eventual consistency
* duplicate messages
* out-of-order messages
* delayed processing
* harder tracing
* dead-letter queues
* idempotent consumers

Async messaging is excellent for side effects and workflows that do not require immediate response.

It is not a way to avoid thinking about consistency.

---

# Events

Events say that something happened.

Good event name:

```text
OrderPaid
```

Weak event name:

```text
OrderUpdated
```

`OrderUpdated` is vague.

What changed?

Was payment captured?

Was the address changed?

Was the status cancelled?

Specific events are easier for consumers to understand.

Event payload example:

```json
{
  "event_id": "evt_123",
  "type": "OrderPaid",
  "version": 1,
  "occurred_at": "2026-01-01T10:00:00Z",
  "order_id": "ord_123",
  "payment_id": "pay_456"
}
```

Events are APIs.

Version them and test them.

---

# Eventual Consistency

In a monolith with one database transaction, changes can be immediately consistent.

In microservices, consistency often becomes eventual.

Example:

```text
payment service captures payment
payment service publishes PaymentCaptured
order service consumes event
order service marks order paid
```

There may be a delay between payment capture and order update.

During that delay, different services may show different states.

This is eventual consistency.

It is not a bug by itself.

It is a design choice.

But users and systems must understand it.

If the business requires immediate consistency, a microservice boundary may be expensive or wrong for that workflow.

---

# Distributed Transactions

Transactions across services are hard.

In a monolith:

```text
begin transaction
update order
update payment
commit
```

In microservices:

```text
orders database
payments database
inventory database
```

There may be no simple single transaction.

Distributed transactions exist, but they add complexity and are often avoided in typical microservice designs.

Instead, systems use:

* sagas
* compensating actions
* idempotency
* retries
* events
* reconciliation jobs

This is one of the main costs of microservices.

Do not split services casually across workflows that require strong atomic consistency.

---

# Sagas

A saga coordinates a multi-step business process across services.

Example order flow:

```text
create order
reserve inventory
authorize payment
confirm order
send notification
```

If payment authorization fails, inventory reservation may need to be released.

That release is a compensating action.

Saga styles:

* orchestration
* choreography

Orchestration uses a coordinator.

Choreography uses services reacting to events.

Both have tradeoffs.

Sagas are business workflows, not just technical retries.

They require careful design and testing.

---

# Idempotency

Idempotency is critical in distributed systems.

Messages may be delivered more than once.

HTTP calls may be retried.

Workers may crash after completing work but before acknowledging a message.

An idempotent operation can safely be repeated.

Example:

```text
mark order paid with payment_id pay_123
```

If the order is already marked paid with the same payment ID, repeating the operation should not create a second payment.

Payment APIs often use idempotency keys:

```text
Idempotency-Key: checkout-ord_123
```

Idempotency is not an implementation detail.

It is part of the API contract.

---

# Timeouts

Every network call needs a timeout.

Without timeouts, a caller can wait forever.

Bad:

```python
response = requests.get(url)
```

Better:

```python
response = requests.get(url, timeout=3)
```

Timeouts should be chosen intentionally.

Too short creates false failures.

Too long ties up resources and worsens cascading failure.

Timeouts are architectural settings.

They should align with user expectations, service-level objectives, and retry policies.

---

# Retries

Retries can help with transient failures.

They can also make outages worse.

Retry when:

* the operation is safe to retry
* the error is likely transient
* there is backoff
* there is a maximum attempt limit
* idempotency is handled

Do not blindly retry:

* non-idempotent operations
* validation errors
* authorization failures
* permanent business failures

Use exponential backoff and jitter where appropriate.

Retries are not a substitute for reliability.

They are a controlled response to temporary failure.

---

# Circuit Breakers

A circuit breaker prevents repeated calls to a failing service.

Conceptually:

```text
closed: calls allowed
open: calls blocked quickly
half-open: limited trial calls
```

If a dependency is failing, continuing to call it may waste resources and slow your system.

A circuit breaker can fail fast and give the dependency time to recover.

Circuit breakers are useful in larger distributed systems.

They add state and complexity.

Use them when dependency failure patterns justify them.

---

# Service Discovery

Services need to find each other.

In a simple setup, configuration may contain URLs:

```text
PAYMENTS_URL=https://payments.internal
```

In larger systems, service discovery may be handled by:

* orchestration platforms
* DNS
* service meshes
* load balancers
* configuration systems

Service discovery is not just lookup.

It affects:

* load balancing
* health checking
* deployment rollout
* failure handling
* local development

Microservices require operational infrastructure.

That infrastructure becomes part of the architecture.

---

# API Gateways

An API gateway sits in front of services.

It may handle:

* routing
* authentication
* rate limiting
* request aggregation
* TLS termination
* logging
* version routing

Gateways can simplify clients.

They can also become bottlenecks or hidden monoliths.

Do not put all business logic into the gateway.

Use gateways for cross-cutting edge concerns.

Keep service ownership clear.

---

# Observability

Microservices require strong observability.

You need:

* structured logs
* correlation IDs
* distributed traces
* metrics
* health checks
* dashboards
* alerts

In a monolith, one request may produce one local call stack.

In microservices, one request may cross:

```text
gateway -> orders -> payments -> inventory -> notifications
```

Without correlation IDs and tracing, debugging becomes guesswork.

Every service should include request or trace context in logs.

Observability is not optional in distributed systems.

---

# Distributed Tracing

Distributed tracing follows a request across services.

A trace contains spans.

Example:

```text
checkout request
    orders service
        database query
        payments service call
            payment provider call
        inventory service call
```

Tracing helps answer:

* where did time go?
* which service failed?
* which dependency was slow?
* how did the request flow?

Logs show events.

Metrics show aggregates.

Traces show cross-service paths.

Microservice systems usually need all three.

---

# Logging in Microservices

Microservice logs should include:

* service name
* version
* environment
* request ID or trace ID
* operation
* entity IDs
* status
* duration
* error type

Example:

```text
service=orders trace_id=abc123 event=payment_requested order_id=ord_1
```

Avoid logs that cannot be correlated:

```text
payment failed
```

Which payment?

Which service?

Which request?

Which version?

Chapter 75's logging lessons become more important in microservices.

Distributed logs without correlation are fragments.

---

# Metrics in Microservices

Each service should expose useful metrics.

Common metrics:

* request count
* error count
* latency
* queue depth
* retry count
* timeout count
* dependency latency
* saturation
* resource usage

Metrics help detect system health.

Logs help investigate details.

Traces help connect paths.

Do not rely only on logs for service health.

Alerts usually work better from metrics.

---

# Health Checks

Services need health checks.

Common endpoints:

```text
/health
/ready
/live
```

Liveness asks:

```text
is the process alive?
```

Readiness asks:

```text
can this service receive traffic?
```

A service may be alive but not ready.

For example, it may be starting up, missing configuration, or unable to connect to a required dependency.

Health checks influence deployment, load balancing, and recovery.

Design them carefully.

Do not make health checks so heavy that they become a source of load.

---

# Testing Microservices

Microservice testing needs layers:

* unit tests
* service tests
* contract tests
* integration tests
* end-to-end tests
* smoke tests

Unit tests check local behavior.

Service tests check one service with controlled dependencies.

Contract tests verify APIs between services.

Integration tests check real boundaries.

End-to-end tests check critical flows across services.

Do not rely only on end-to-end tests.

They are slow, brittle, and harder to diagnose.

Use contract tests to protect service boundaries.

---

# Contract Testing

Contract testing is essential when services evolve independently.

If orders service calls payments service, both need agreement on:

* endpoint
* request shape
* response shape
* errors
* status codes
* version
* semantics

Consumer-driven contract testing records what consumers expect.

Providers verify they satisfy those expectations.

This reduces the chance that one service deploys a breaking change for another.

APIs are contracts.

Microservices multiply contracts.

Testing must reflect that.

---

# Local Development

Microservices complicate local development.

Questions:

* do developers run all services locally?
* do they use containers?
* are dependencies mocked?
* is there a shared development environment?
* how are databases seeded?
* how are messages tested?
* how are service URLs configured?

If local development becomes too hard, productivity suffers.

A modular monolith often wins here.

Microservice architecture must include developer experience.

If nobody can run the system, the architecture is failing humans.

---

# Deployment

Microservices require deployment discipline.

Each service may have:

* build pipeline
* tests
* container image
* configuration
* secrets
* migrations
* rollout strategy
* monitoring
* rollback path

Independent deployment is a benefit only if the organization can operate it.

If every deployment still requires coordinating all services manually, the system may have microservice costs without microservice benefits.

Deployment automation is part of the architecture.

---

# Data Migrations

Data migrations are harder with microservices.

If one service owns data, other services should not modify it directly.

Schema changes must be compatible with deployed code.

A safe migration often uses expand-and-contract:

1. add new schema without breaking old code
2. deploy code that writes both old and new forms if needed
3. migrate existing data
4. deploy code that reads new form
5. remove old schema later

This avoids requiring all code to update at the exact same moment.

Microservice migrations require patience.

Breaking changes are expensive.

---

# Security Between Services

Microservices need service-to-service security.

Questions:

* how do services authenticate each other?
* how are requests authorized?
* how are secrets stored?
* is traffic encrypted?
* who can call internal endpoints?
* how are audit logs collected?
* how are tokens rotated?

Internal network does not mean trusted network.

Assume service boundaries need protection.

A bug in one service should not grant unlimited access to every other service.

Security architecture matters more as service count grows.

---

# Versioning Services

Services evolve independently.

APIs must be versioned or evolved compatibly.

Good practice:

* add fields before requiring them
* support old and new consumers during migration
* avoid sudden response shape changes
* version event schemas
* communicate deprecations
* monitor old API usage

The hardest part is not adding version numbers.

The hardest part is running multiple versions long enough for clients to migrate.

Microservices require patience with compatibility.

---

# Ownership

Microservices work best when ownership is clear.

Each service should have owners who understand:

* code
* data
* API
* alerts
* deployment
* dependencies
* operational behavior

If no team owns a service, it decays.

If many teams casually change a service without coordination, its boundary weakens.

Service ownership is not only organizational.

It is technical.

Ownership defines who protects the contract.

---

# Python and Microservices

Python is commonly used for microservices.

Strengths:

* fast development
* strong web frameworks
* good API tooling
* rich ecosystem
* good scripting and automation
* readable service code
* excellent integration capabilities

Challenges:

* CPU-bound performance under CPython
* dependency management across services
* packaging discipline
* async vs sync design choices
* runtime type enforcement needs at boundaries
* deployment and environment consistency

Python can work very well for microservices.

But the architecture still matters.

A bad service boundary is bad in any language.

---

# FastAPI, Flask, and Django Services

Python services may be built with:

* Flask
* Django
* FastAPI
* other frameworks

Framework choice affects:

* request handling
* validation
* dependency injection style
* async support
* ORM patterns
* documentation generation
* testing style

But framework choice is not service architecture by itself.

A FastAPI app can be tangled.

A Flask app can be clean.

A Django app can be modular.

Frameworks provide tools.

Architecture is how you use them.

Volume IV will study these frameworks directly.

---

# When Microservices Help

Microservices may help when:

* multiple teams need independent ownership
* parts of the system scale very differently
* deployment cadence differs by capability
* a capability needs strong isolation
* data ownership boundaries are clear
* operational maturity exists
* APIs are well-defined
* observability is strong
* the organization can support distributed systems

Example:

Search, payments, and notifications may have very different scaling and operational needs.

Separate services may make sense.

But only if the boundaries are clear and the team can run them.

---

# When Microservices Hurt

Microservices hurt when:

* the team is small
* the domain boundaries are unclear
* the monolith is not modular yet
* deployment automation is weak
* observability is weak
* services need constant synchronous calls to each other
* data consistency must be immediate everywhere
* local development becomes painful
* APIs are unstable
* teams cannot own services independently

In these cases, a modular monolith may be better.

Do not use microservices to avoid refactoring.

Refactor boundaries first.

Then split when the boundary is strong enough to survive distribution.

---

# Extracting a Service

A practical extraction path:

1. Identify a module with clear responsibility.
2. Define its public API inside the monolith.
3. Stop other modules from reaching into its internals.
4. Give it clear data ownership.
5. Add contract tests.
6. Add observability around its operations.
7. Replace direct calls with an internal interface.
8. Move implementation behind a process boundary.
9. Keep compatibility during migration.
10. Remove old paths after consumers migrate.

Do not start by creating a new repository.

Start by creating a real boundary.

Process boundaries should come after logical boundaries.

---

# The Distributed Monolith

A distributed monolith has multiple services but behaves like one tightly coupled application.

Symptoms:

* every deployment requires all services
* services share databases casually
* one request requires many synchronous calls
* API changes require coordinated releases
* local development requires running everything
* failures cascade easily
* teams cannot work independently

This architecture has the costs of microservices without the benefits.

It is a common failure mode.

Avoid it by protecting boundaries, data ownership, compatibility, and service autonomy.

---

# Microservices Checklist

Before choosing microservices, ask:

* Are service boundaries clear?
* Does each service own its data?
* Do teams need independent deployment?
* Can the organization operate multiple services?
* Is observability mature enough?
* Are APIs stable and tested?
* How will local development work?
* How will data consistency be handled?
* How will failures be isolated?
* How will migrations happen?
* Who owns each service?
* Would a modular monolith solve the current problem?

The last question matters.

Microservices are not the only way to build serious systems.

---

# Chapter Summary

Microservices are independently deployable services organized around business capabilities.

They communicate through APIs and usually own their data.

They can support independent deployment, scaling, team ownership, and fault isolation.

They also introduce distributed-system costs: latency, partial failure, consistency challenges, observability needs, deployment complexity, API versioning, and local development friction.

A modular monolith is often the best step before microservices.

Good services usually begin as good modules.

Service boundaries should align with business capabilities and data ownership.

Services should not casually share mutable databases.

Communication may be synchronous or asynchronous.

Synchronous calls are simpler but create runtime coupling.

Asynchronous messaging decouples time but introduces eventual consistency, duplicates, ordering concerns, and harder debugging.

Events are APIs and should be versioned, documented, and tested.

Distributed transactions are hard, so systems often use sagas, compensating actions, retries, idempotency, and reconciliation.

Every network call needs timeouts and thoughtful retry behavior.

Microservices require strong logging, metrics, health checks, tracing, and contract testing.

Python is a strong language for many services, but framework choice does not replace architectural judgment.

Microservices help when independent ownership and deployment are real needs.

They hurt when used before boundaries, operations, and APIs are mature.

The central lesson is:

```text
microservices distribute architecture decisions across processes
```

If the decisions are unclear inside one process, distribution will not clarify them.

It will amplify them.

---

# Exercises

1. Take a modular monolith and identify one module that might eventually become a service. Explain why or why not.

2. Define the data owned by an imaginary payments service and list what other services may access through APIs only.

3. Design an `OrderPaid` event with idempotency and versioning fields.

4. Write a timeout and retry policy for a safe read-only service call.

5. Identify a workflow that needs strong consistency and explain why splitting it across services may be hard.

6. Draw a trace for a checkout request crossing three services.

7. Write contract tests for a consumer and provider API shape.

8. Design health and readiness checks for a Python service.

9. List the observability fields every service log should include.

10. Compare a modular monolith and microservices for a team of three developers building an early product.

---

# Volume III Closing

Volume III began with testing because professional engineering starts with confidence.

From there, it moved through the practices that make Python software reliable, maintainable, observable, distributable, understandable, and evolvable.

We studied:

* testing
* mocking and monkey patching
* debugging
* logging
* packaging
* type hints
* static type checking
* profiling
* design patterns
* SOLID principles
* architecture
* APIs
* microservices

Together, these chapters moved from code that works on one machine to software that can be changed, shipped, operated, and trusted by teams.

The deep lesson of Volume III is:

```text
professional engineering is not one technique
it is a system of feedback loops
```

Tests give behavioral feedback.

Type checkers give structural feedback.

Logs, metrics, and traces give operational feedback.

Profiling gives performance feedback.

Architecture gives change feedback.

APIs give contract feedback.

Microservices show what happens when those contracts become distributed.

Volume I taught the foundations and core language.

Volume II opened Python's advanced language features and internals.

Volume III turned that knowledge into engineering practice.

You now have the tools to read, write, test, debug, package, type, profile, design, and reason about serious Python systems.

---

# Preview of Volume IV

Volume IV moves from engineering foundations into the Python ecosystem and professional paths.

The next volume studies the frameworks, libraries, domains, and career directions where Python is used every day.

It begins with web development:

* Flask
* Django
* FastAPI

Then it moves into data and scientific computing:

* NumPy
* Pandas
* Polars

Then machine learning and AI engineering:

* PyTorch
* TensorFlow
* AI engineering
* RAG systems
* agents

Then automation, integration, and professional specialization:

* backend engineering
* data engineering
* machine learning
* DevOps
* cybersecurity

The transition is:

```text
Volume III teaches how to engineer Python systems
Volume IV shows where those systems live in the real world
```

The next step is Flask, a small web framework that makes the request-response model visible without hiding too much machinery.
