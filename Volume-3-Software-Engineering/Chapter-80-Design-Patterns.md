# Chapter 80 — Design Patterns

Design patterns are named responses to recurring design problems.

They are not magic.

They are not rules.

They are not decorations.

They are not proof that code is professional.

A design pattern is useful when it gives a team a shared name for a design pressure:

```text
we need to choose an algorithm at runtime
we need to adapt one interface to another
we need to isolate database access
we need to represent an action as an object
we need to publish events to many listeners
we need to create objects without spreading construction logic everywhere
```

The name helps conversation.

The structure helps code.

But patterns can be overused.

If a pattern makes code harder to understand without solving a real problem, it is not helping.

Python makes this especially important.

Because Python has first-class functions, duck typing, context managers, decorators, dynamic dispatch, protocols, dataclasses, modules, and rich standard-library tools, many patterns are simpler in Python than in more rigid languages.

This chapter teaches design patterns as engineering vocabulary.

The goal is not to memorize a catalog.

The goal is to recognize recurring forces and choose a shape that keeps the code understandable as it changes.

---

# What a Pattern Is

A pattern has three parts:

```text
problem
context
solution shape
```

The problem is what keeps recurring.

The context is when the pattern makes sense.

The solution shape is the structure that addresses the problem.

For example:

```text
Problem: multiple pricing rules need to be swapped.
Context: checkout should not know every rule.
Pattern: Strategy.
```

The pattern says:

```text
represent each pricing rule behind a common interface
choose the rule from outside the checkout code
```

It does not say:

```text
create five abstract classes and three factories before writing the first feature
```

Patterns are not ceremony.

They are pressure-release valves for real complexity.

---

# Why Patterns Matter

Patterns matter because software changes.

When change is small, direct code is often best.

When change repeats in a predictable direction, structure helps.

Examples:

* more payment providers will be added
* more export formats will be supported
* more notification channels will be introduced
* database access must be tested independently
* external service APIs change frequently
* business rules vary by customer
* commands need undo or retry
* many parts of the system need to react to an event

Patterns help when they make future change local.

Without structure, change spreads.

With appropriate structure, change has a place to go.

That is the real value:

```text
patterns organize change
```

---

# The Danger of Pattern Worship

Patterns can become harmful when developers force them into simple code.

Simple code:

```python
def tax(amount: Decimal) -> Decimal:
    return amount * Decimal("0.18")
```

Overdesigned code:

```python
class AbstractTaxStrategyFactoryProvider:
    ...
```

The second version may look more architectural.

It may simply be worse.

A pattern should pay rent.

It should reduce duplication, isolate change, clarify boundaries, improve testability, or express a real domain concept.

If it only adds names, files, and indirection, it is noise.

Python rewards simple designs.

Start direct.

Add structure when change demands it.

---

# Pythonic Patterns

Many classic patterns look different in Python.

In Python:

* functions are objects
* classes are objects
* modules are objects
* behavior can be passed directly
* protocols can express duck-typed interfaces
* decorators can wrap behavior
* context managers manage setup and cleanup
* dictionaries can replace simple dispatch classes
* dataclasses can model simple records

For example, in some languages Strategy requires classes.

In Python, a strategy can be a function:

```python
def percentage_discount(total: Decimal) -> Decimal:
    return total * Decimal("0.10")


def apply_discount(total: Decimal, discount_rule) -> Decimal:
    return total - discount_rule(total)
```

That may be enough.

If the strategy needs state, configuration, or multiple methods, a class may be better.

Python gives you a range.

Use the lightest shape that communicates the design.

---

# Factory

The Factory pattern centralizes object creation.

Use it when construction logic is complex or should not be scattered.

Without a factory:

```python
if provider == "smtp":
    sender = SmtpEmailSender(host, port, username, password)
elif provider == "console":
    sender = ConsoleEmailSender()
elif provider == "api":
    sender = ApiEmailSender(api_key)
else:
    raise ValueError(f"unknown provider: {provider}")
```

If this appears in many places, the code becomes hard to change.

A factory gives construction one home:

```python
def create_email_sender(config: EmailConfig) -> EmailSender:
    if config.provider == "smtp":
        return SmtpEmailSender(
            config.host,
            config.port,
            config.username,
            config.password,
        )
    if config.provider == "console":
        return ConsoleEmailSender()
    if config.provider == "api":
        return ApiEmailSender(config.api_key)
    raise ValueError(f"unknown provider: {config.provider}")
```

Now application code asks for an email sender.

It does not know every construction detail.

---

# When to Use a Factory

Use a factory when:

* construction has branching logic
* construction requires configuration
* callers should not know concrete classes
* object creation is repeated
* tests need to replace construction
* external providers vary

Do not use a factory when ordinary construction is clear:

```python
user = User(email=email)
```

This does not need a factory.

Factories are useful when construction itself becomes a design concern.

They are not needed for every class.

---

# Factory as Function

In Python, a factory is often just a function.

Example:

```python
def create_storage(settings: Settings) -> Storage:
    match settings.storage_backend:
        case "local":
            return LocalStorage(settings.local_path)
        case "s3":
            return S3Storage(settings.bucket_name)
        case _:
            raise ValueError(f"unknown storage backend: {settings.storage_backend}")
```

This is simple.

It is easy to test.

It does not require a factory class.

Factory classes are useful when the factory has state, dependencies, or multiple related creation methods.

Start with a function unless a class has a real reason to exist.

---

# Strategy

The Strategy pattern represents interchangeable behavior.

Use it when an algorithm or rule varies independently from the code that uses it.

Example:

```python
def no_discount(total: Decimal) -> Decimal:
    return Decimal("0")


def percentage_discount(total: Decimal) -> Decimal:
    return total * Decimal("0.10")


def fixed_discount(total: Decimal) -> Decimal:
    return Decimal("100")
```

Checkout can accept the strategy:

```python
def checkout(cart: Cart, discount_rule: Callable[[Decimal], Decimal]) -> Receipt:
    total = cart.total()
    discount = discount_rule(total)
    return Receipt(total=total - discount)
```

The checkout logic does not know every discount type.

It only knows how to use a discount rule.

This keeps variation outside the core workflow.

---

# Strategy as Class

Sometimes a strategy needs state.

Example:

```python
class PercentageDiscount:
    def __init__(self, percent: Decimal):
        self.percent = percent

    def __call__(self, total: Decimal) -> Decimal:
        return total * self.percent / Decimal("100")
```

Use:

```python
discount = PercentageDiscount(15)
receipt = checkout(cart, discount)
```

A strategy class is useful when behavior has configuration.

If behavior is a simple calculation, a function may be enough.

Python's callable objects let you choose either shape.

---

# Strategy With Protocol

For richer strategies, define a protocol.

```python
from typing import Protocol


class PricingStrategy(Protocol):
    def price(self, cart: Cart) -> Money:
        ...
```

Implementations:

```python
class RetailPricing:
    def price(self, cart: Cart) -> Money:
        ...


class WholesalePricing:
    def price(self, cart: Cart) -> Money:
        ...
```

Usage:

```python
def checkout(cart: Cart, pricing: PricingStrategy) -> Receipt:
    total = pricing.price(cart)
    return Receipt(total=total)
```

The checkout code depends on the capability, not the concrete class.

This is strategy in a Pythonic typed style.

---

# Adapter

The Adapter pattern translates one interface into another.

Use it when code you have does not match the interface you want.

Suppose your application wants:

```python
class EmailSender(Protocol):
    def send(self, message: EmailMessage) -> None:
        ...
```

But a third-party client works like this:

```python
third_party.send_email(
    to_address="a@example.com",
    subject_line="Hello",
    html_body="<p>Hello</p>",
)
```

Create an adapter:

```python
class ThirdPartyEmailAdapter:
    def __init__(self, client):
        self.client = client

    def send(self, message: EmailMessage) -> None:
        self.client.send_email(
            to_address=message.to,
            subject_line=message.subject,
            html_body=message.html,
        )
```

Now the rest of the application uses `EmailSender`.

Only the adapter knows the third-party shape.

---

# Why Adapters Matter

Adapters protect your code from external interfaces.

Third-party APIs change.

SDKs have awkward names.

External response objects may be messy.

Your domain should not be forced to speak every vendor dialect.

Adapter boundary:

```text
application interface -> adapter -> external service
```

This helps:

* testing
* replacement
* migration
* readability
* error handling
* security review

Adapters are especially useful for:

* payment providers
* email services
* storage systems
* analytics clients
* AI APIs
* CRM systems
* legacy systems

Do not let vendor details leak everywhere.

Put them behind an adapter.

---

# Facade

The Facade pattern provides a simpler interface over a complex subsystem.

Example:

```python
class ReportFacade:
    def __init__(self, loader, calculator, renderer, storage):
        self.loader = loader
        self.calculator = calculator
        self.renderer = renderer
        self.storage = storage

    def generate_monthly_report(self, customer_id: str, month: Month) -> ReportId:
        data = self.loader.load(customer_id, month)
        summary = self.calculator.summarize(data)
        pdf = self.renderer.render(summary)
        return self.storage.save(pdf)
```

Callers do not need to know the whole workflow.

They call:

```python
report_id = reports.generate_monthly_report(customer_id, month)
```

The facade hides subsystem complexity.

It does not eliminate complexity.

It gives it a boundary.

---

# Facade vs God Object

A facade can become too large.

Bad sign:

```python
class ApplicationFacade:
    def do_everything(...):
        ...
```

If one facade knows every subsystem and every workflow, it becomes a god object.

A good facade is focused.

Examples:

* `ReportService`
* `CheckoutService`
* `ImportService`
* `NotificationService`

It should simplify a coherent area.

It should not become the place where all logic goes to hide.

---

# Repository

The Repository pattern isolates data access behind a collection-like interface.

Instead of spreading database queries everywhere:

```python
user = session.query(UserModel).filter_by(email=email).one_or_none()
```

create a repository:

```python
class UserRepository:
    def __init__(self, session):
        self.session = session

    def find_by_email(self, email: str) -> User | None:
        row = self.session.query(UserRow).filter_by(email=email).one_or_none()
        if row is None:
            return None
        return User(id=row.id, email=row.email)
```

Application code:

```python
user = users.find_by_email(email)
```

The repository hides persistence details.

It can convert between database rows and domain objects.

---

# When Repository Helps

Repository helps when:

* database access is repeated
* domain code should not know SQL or ORM details
* tests need fake persistence
* data mapping is nontrivial
* storage may change
* transaction boundaries need clarity

Repository may be overkill when:

* the app is tiny
* ORM models are the domain model
* queries are simple and local
* abstraction only repeats ORM methods

Bad repository:

```python
class UserRepository:
    def add(self, user):
        self.session.add(user)

    def delete(self, user):
        self.session.delete(user)

    def query(self):
        return self.session.query(User)
```

This may not add much.

A repository should express domain persistence operations, not merely rename the ORM.

---

# Unit of Work

The Unit of Work pattern groups changes into a transaction.

It answers:

```text
which changes commit together?
```

Example:

```python
class UnitOfWork:
    def __init__(self, session_factory):
        self.session_factory = session_factory

    def __enter__(self):
        self.session = self.session_factory()
        self.users = UserRepository(self.session)
        self.orders = OrderRepository(self.session)
        return self

    def __exit__(self, exc_type, exc, traceback):
        if exc_type is None:
            self.session.commit()
        else:
            self.session.rollback()
        self.session.close()
```

Usage:

```python
with unit_of_work as uow:
    user = uow.users.find_by_email(email)
    order = Order(user_id=user.id)
    uow.orders.add(order)
```

If the block succeeds, changes commit.

If it fails, changes roll back.

This pattern connects naturally to Python context managers.

---

# Command

The Command pattern represents an action as an object.

Use it when actions need to be queued, logged, retried, undone, authorized, or dispatched.

Example:

```python
@dataclass
class CreateUserCommand:
    email: str
    name: str
```

Handler:

```python
class CreateUserHandler:
    def __init__(self, users, email_sender):
        self.users = users
        self.email_sender = email_sender

    def handle(self, command: CreateUserCommand) -> UserId:
        user = self.users.create(command.email, command.name)
        self.email_sender.send_welcome(user.email)
        return user.id
```

Now the request can be represented independently from the handler.

This is useful for:

* background jobs
* message queues
* audit logs
* permission checks
* retries
* CLI operations

---

# Command as Function

Not every command needs a class.

In Python, a command can be a function:

```python
def create_user(command: CreateUserCommand, users, email_sender) -> UserId:
    user = users.create(command.email, command.name)
    email_sender.send_welcome(user.email)
    return user.id
```

This may be enough.

Use a handler class when dependencies are shared or the command belongs to a larger dispatch system.

Use a function when the shape is simple.

The pattern is about representing actions clearly.

The implementation can stay lightweight.

---

# Observer

The Observer pattern lets one event notify multiple listeners.

Example:

```python
class EventBus:
    def __init__(self):
        self._subscribers = {}

    def subscribe(self, event_type: type, handler):
        self._subscribers.setdefault(event_type, []).append(handler)

    def publish(self, event):
        for handler in self._subscribers.get(type(event), []):
            handler(event)
```

Event:

```python
@dataclass
class UserCreated:
    user_id: UserId
    email: str
```

Usage:

```python
bus.subscribe(UserCreated, send_welcome_email)
bus.subscribe(UserCreated, record_analytics_event)

bus.publish(UserCreated(user.id, user.email))
```

The user creation code does not need to know every side effect.

It publishes an event.

Listeners react.

---

# Observer Tradeoffs

Observer reduces direct coupling.

It can also make behavior harder to trace.

Direct call:

```python
create_user()
send_email()
record_metric()
```

Event-based call:

```python
publish(UserCreated(...))
```

Now side effects happen elsewhere.

This can be good when the system has many independent reactions.

It can be bad when the flow becomes invisible.

Use observer when decoupling is valuable.

Keep events named, logged, and testable.

Do not use events to hide ordinary control flow.

---

# Dependency Injection

Dependency injection means giving an object or function its dependencies from outside.

Harder to test:

```python
class SignupService:
    def signup(self, email: str) -> None:
        sender = SmtpEmailSender()
        sender.send(email, "Welcome", "Hello")
```

Easier to test:

```python
class SignupService:
    def __init__(self, sender: EmailSender):
        self.sender = sender

    def signup(self, email: str) -> None:
        self.sender.send(email, "Welcome", "Hello")
```

Now tests can pass a fake sender.

Dependency injection is not only for tests.

It also makes configuration explicit.

It makes boundaries visible.

It reduces hidden global coupling.

---

# Dependency Injection Without Frameworks

Python does not require a dependency injection framework.

You can wire objects manually:

```python
def build_app(settings: Settings) -> App:
    storage = create_storage(settings)
    email_sender = create_email_sender(settings)
    users = UserRepository(storage.session)
    signup = SignupService(users, email_sender)
    return App(signup=signup)
```

This is called composition root in some design discussions.

It means:

```text
construct the object graph near application startup
use explicit dependencies elsewhere
```

Frameworks can help in large systems.

But explicit construction is often clearer than magic.

---

# Context Object

A context object groups information that travels together.

Example:

```python
@dataclass
class RequestContext:
    request_id: str
    user_id: UserId | None
    tenant_id: TenantId
    permissions: set[str]
```

Instead of passing many values:

```python
def create_order(request_id, user_id, tenant_id, permissions, payload):
    ...
```

pass context:

```python
def create_order(context: RequestContext, payload: OrderPayload) -> Order:
    ...
```

This can improve readability when values are truly related.

But context objects can become dumping grounds.

Do not put everything into `context`.

Keep it coherent.

---

# Template Method

Template Method defines a fixed workflow with customizable steps.

Example:

```python
class Importer:
    def run(self, path: Path) -> ImportResult:
        rows = self.read(path)
        records = [self.parse(row) for row in rows]
        return self.save(records)

    def read(self, path: Path):
        raise NotImplementedError

    def parse(self, row):
        raise NotImplementedError

    def save(self, records):
        raise NotImplementedError
```

Subclasses customize steps.

In Python, composition is often preferable:

```python
def run_import(reader, parser, saver, path: Path) -> ImportResult:
    rows = reader.read(path)
    records = [parser.parse(row) for row in rows]
    return saver.save(records)
```

Template Method can be useful.

But inheritance-heavy designs can become rigid.

Prefer composition when it keeps variation clearer.

---

# Decorator Pattern

The Decorator pattern wraps behavior with additional behavior.

Python has language-level decorators:

```python
def traced(func):
    def wrapper(*args, **kwargs):
        logger.info("calling %s", func.__name__)
        return func(*args, **kwargs)
    return wrapper
```

Use:

```python
@traced
def create_order(...):
    ...
```

Decorators are useful for cross-cutting behavior:

* logging
* timing
* caching
* authorization
* retrying
* validation
* instrumentation

But decorators can hide control flow.

Use `functools.wraps`.

Keep decorators focused.

Avoid stacking many decorators until function behavior becomes hard to see.

---

# Adapter vs Decorator

Adapter changes an interface.

Decorator preserves an interface while adding behavior.

Adapter:

```text
third-party email client -> EmailSender interface
```

Decorator:

```text
EmailSender -> LoggingEmailSender -> still EmailSender
```

Example decorator object:

```python
class LoggingEmailSender:
    def __init__(self, sender: EmailSender):
        self.sender = sender

    def send(self, message: EmailMessage) -> None:
        logger.info("email_sending to=%s", message.to)
        self.sender.send(message)
```

It wraps another sender and adds logging.

The interface remains `send`.

That is the distinction.

---

# Singleton

Singleton ensures only one instance exists.

In Python, this pattern is often unnecessary.

Modules already provide a simple shared namespace.

Example:

```python
# settings.py
config = load_config()
```

Import:

```python
from settings import config
```

This is module-level shared state.

But be careful.

Global shared state can make tests and concurrency harder.

Many things people call singleton are better handled by:

* explicit dependency injection
* application startup wiring
* module-level constants
* cached factory functions
* context objects

Use singleton-like state sparingly.

Shared mutable global state is a sharp tool.

---

# Registry

A registry maps names to objects or handlers.

Example:

```python
EXPORTERS: dict[str, Exporter] = {}


def register_exporter(name: str, exporter: Exporter) -> None:
    EXPORTERS[name] = exporter


def export(name: str, data: Data) -> bytes:
    return EXPORTERS[name].export(data)
```

Registries are useful for:

* plugin systems
* serializers
* command dispatch
* file format handlers
* test factories

They can also hide dependencies if overused.

If registration happens at import time, import order can matter.

Prefer explicit registration during application setup when possible.

---

# Dispatch Table

A dispatch table maps keys to functions.

Example:

```python
def export_json(data: Data) -> bytes:
    ...


def export_csv(data: Data) -> bytes:
    ...


EXPORTERS = {
    "json": export_json,
    "csv": export_csv,
}


def export(format_name: str, data: Data) -> bytes:
    try:
        exporter = EXPORTERS[format_name]
    except KeyError:
        raise ValueError(f"unknown format: {format_name}")
    return exporter(data)
```

This is often simpler than a class hierarchy.

Use dispatch tables when behavior can be represented as functions.

Move to objects when behavior needs state, lifecycle, or multiple methods.

---

# State Pattern

The State pattern represents behavior that changes with an object's state.

Example states:

```text
draft
submitted
approved
rejected
cancelled
```

Naive code:

```python
def approve(document):
    if document.status == "draft":
        raise ValueError("draft cannot be approved")
    if document.status == "submitted":
        document.status = "approved"
    if document.status == "rejected":
        raise ValueError("rejected document cannot be approved")
```

For small state machines, conditionals are fine.

For complex state behavior, explicit transition tables or state objects may help.

Example transition table:

```python
ALLOWED_TRANSITIONS = {
    "draft": {"submitted"},
    "submitted": {"approved", "rejected"},
    "approved": set(),
    "rejected": {"submitted"},
}
```

Python often expresses state machines well with data tables.

Use state classes only when each state has substantial behavior.

---

# Builder

The Builder pattern constructs complex objects step by step.

In Python, many builder use cases are better served by:

* dataclasses
* keyword arguments
* factory functions
* configuration objects
* test factories

Example:

```python
@dataclass
class EmailMessage:
    to: str
    subject: str
    body: str
    cc: list[str] = field(default_factory=list)
```

Construction:

```python
message = EmailMessage(
    to="a@example.com",
    subject="Welcome",
    body="Hello",
)
```

This may be clearer than a builder.

Use a builder when construction has many ordered steps, validation phases, or fluent test setup needs.

Do not create builders just to avoid keyword arguments.

Python already has good keyword arguments.

---

# Builder for Tests

Builders can be useful in tests.

Example:

```python
class UserBuilder:
    def __init__(self):
        self.email = "user@example.com"
        self.active = True

    def inactive(self):
        self.active = False
        return self

    def with_email(self, email: str):
        self.email = email
        return self

    def build(self) -> User:
        return User(email=self.email, active=self.active)
```

Use:

```python
user = UserBuilder().inactive().build()
```

This can make test setup readable.

But a simple factory function may be enough:

```python
def make_user(**overrides) -> User:
    data = {"email": "user@example.com", "active": True}
    data.update(overrides)
    return User(**data)
```

Choose clarity.

---

# Proxy

The Proxy pattern controls access to another object.

Examples:

* lazy loading
* access control
* caching
* remote service access
* logging

Example caching proxy:

```python
class CachedUserClient:
    def __init__(self, client):
        self.client = client
        self.cache = {}

    def get_user(self, user_id: str) -> User:
        if user_id not in self.cache:
            self.cache[user_id] = self.client.get_user(user_id)
        return self.cache[user_id]
```

The proxy exposes the same method as the real client.

It adds caching.

Proxy is similar to decorator, but often emphasizes controlled access, laziness, or remote interaction.

The boundaries overlap.

Pattern names are vocabulary, not prison bars.

---

# Chain of Responsibility

Chain of Responsibility passes a request through handlers until one handles it.

Example:

```python
class Handler(Protocol):
    def handle(self, request: Request) -> Response | None:
        ...
```

Chain:

```python
def dispatch(request: Request, handlers: list[Handler]) -> Response:
    for handler in handlers:
        response = handler.handle(request)
        if response is not None:
            return response
    raise LookupError("no handler")
```

This is useful for:

* middleware
* validation chains
* command routing
* import parsers
* authentication strategies

Be careful with invisible flow.

If a chain becomes long and order-sensitive, debugging can become difficult.

Log important decisions.

Keep handlers focused.

---

# Middleware

Middleware is a common form of layered processing.

In web applications, middleware may handle:

* authentication
* logging
* request IDs
* error handling
* compression
* CORS
* sessions
* metrics

Conceptually:

```text
request -> middleware -> middleware -> handler -> middleware -> response
```

Middleware is powerful because it centralizes cross-cutting concerns.

It is risky when business logic leaks into it.

Use middleware for request-level infrastructure.

Keep domain decisions in domain or application services.

---

# Null Object

Null Object replaces `None` with an object that does nothing.

Example:

```python
class NullEmailSender:
    def send(self, message: EmailMessage) -> None:
        pass
```

Instead of:

```python
if email_sender is not None:
    email_sender.send(message)
```

you can always call:

```python
email_sender.send(message)
```

Null objects can simplify code when "do nothing" is a valid behavior.

But they can hide configuration mistakes.

If email must be sent in production, a null sender would be dangerous.

Use null objects when absence is intentional.

Do not use them to hide errors.

---

# Specification

The Specification pattern represents business rules as composable objects.

Example:

```python
class Specification(Protocol):
    def is_satisfied_by(self, candidate) -> bool:
        ...
```

Rule:

```python
class ActiveUser:
    def is_satisfied_by(self, user: User) -> bool:
        return user.active
```

Another rule:

```python
class HasMinimumPoints:
    def __init__(self, points: int):
        self.points = points

    def is_satisfied_by(self, user: User) -> bool:
        return user.points >= self.points
```

This can help when business rules are named, reused, combined, or explained.

For simple conditions, plain functions or direct `if` statements are clearer.

Do not turn every boolean into a class.

---

# Service Layer

A service layer coordinates application actions.

Example:

```python
class CheckoutService:
    def __init__(self, orders, payments, email_sender):
        self.orders = orders
        self.payments = payments
        self.email_sender = email_sender

    def checkout(self, order_id: OrderId) -> Receipt:
        order = self.orders.get(order_id)
        payment = self.payments.charge(order)
        order.mark_paid(payment.id)
        self.orders.save(order)
        self.email_sender.send_receipt(order)
        return Receipt(order.id, payment.id)
```

The service coordinates repositories, gateways, and domain objects.

It should not become a dumping ground for all business logic.

Domain rules that belong on domain objects should stay there.

Service layer is useful when workflows cross multiple components.

---

# Domain Model

A domain model represents business concepts directly in code.

Example:

```python
@dataclass
class Order:
    id: OrderId
    status: str
    total: Money

    def mark_paid(self, payment_id: PaymentId) -> None:
        if self.status != "pending":
            raise ValueError("only pending orders can be paid")
        self.status = "paid"
        self.payment_id = payment_id
```

The rule lives with the concept.

This is often better than putting all rules in external functions that manipulate passive data.

But simple data containers are fine when there is no behavior.

A domain model is useful when business rules have identity, lifecycle, and invariants.

---

# Data Mapper

Data Mapper separates domain objects from database representation.

Domain:

```python
@dataclass
class User:
    id: UserId
    email: str
```

Database row:

```python
class UserRow:
    ...
```

Mapper:

```python
def row_to_user(row: UserRow) -> User:
    return User(id=UserId(row.id), email=row.email)
```

This pattern helps when domain models should not depend on ORM details.

It adds complexity.

For many applications, using ORM models directly may be acceptable.

Use data mapping when the separation pays for itself.

---

# Choosing a Pattern

Choose a pattern by asking:

* what changes often?
* what should stay stable?
* what dependency should point where?
* what should be easy to test?
* what should be hidden?
* what should be explicit?
* what vocabulary helps the team?

Do not start with:

```text
which pattern can I use here?
```

Start with:

```text
what design pressure exists here?
```

Then choose the simplest shape that handles that pressure.

Sometimes the answer is a pattern.

Sometimes the answer is a function.

Sometimes the answer is deleting code.

---

# Patterns and Testing

Good patterns often improve testing.

Examples:

* Strategy lets tests pass a simple rule.
* Adapter isolates third-party APIs.
* Repository can be replaced with a fake.
* Unit of Work makes transaction behavior explicit.
* Dependency Injection exposes boundaries.
* Command objects are easy to queue and inspect.

But patterns can also hurt tests if overdone.

If a test needs to construct ten design-pattern objects before checking one rule, the design may be too heavy.

Testing pressure is feedback.

Hard-to-test code may need better boundaries.

Over-abstracted code may need fewer boundaries.

---

# Patterns and Type Hints

Type hints make patterns clearer.

Protocols are especially useful.

Example:

```python
class PaymentGateway(Protocol):
    def charge(self, order: Order) -> PaymentResult:
        ...
```

Now `CheckoutService` depends on a capability:

```python
class CheckoutService:
    def __init__(self, payments: PaymentGateway):
        self.payments = payments
```

The concrete implementation can be:

```python
class StripePaymentGateway:
    ...
```

or:

```python
class FakePaymentGateway:
    ...
```

Type hints make the boundary visible without requiring inheritance.

This is very Pythonic.

---

# Patterns and Performance

Patterns can affect performance.

Indirection has cost.

Usually the cost is tiny compared with I/O, database queries, or real business work.

But in hot loops, abstraction can matter.

Profiling should guide performance decisions.

Do not remove good boundaries because they "might be slow."

Do not add layers in performance-critical paths without reason.

The relationship is:

```text
design for clarity
profile when performance matters
adjust based on evidence
```

Chapter 79 exists for this reason.

---

# Refactoring Toward Patterns

Patterns often emerge through refactoring.

Start:

```python
def export(data, format_name):
    if format_name == "json":
        ...
    elif format_name == "csv":
        ...
```

Then more formats appear.

The condition grows.

Tests become repetitive.

Now a dispatch table or strategy may help.

Refactor:

```python
EXPORTERS = {
    "json": export_json,
    "csv": export_csv,
    "xml": export_xml,
}
```

Later, exporters need state and metadata.

Now classes may help.

Patterns do not need to appear on day one.

They can emerge when the code shows the need.

---

# Common Pattern Mistakes

Common mistakes include:

* adding patterns before the problem exists
* confusing complexity with professionalism
* using inheritance when composition is simpler
* creating abstract base classes for one implementation
* hiding ordinary control flow behind events
* making factories for simple constructors
* creating repositories that only rename ORM methods
* using singleton global state everywhere
* turning every condition into a specification object
* using patterns to avoid understanding the domain
* copying patterns from another language without adapting to Python

Patterns should clarify.

If they obscure, pause.

---

# A Pattern Checklist

Before introducing a pattern, ask:

* What problem does this solve?
* What change does it make easier?
* What complexity does it add?
* Is there a simpler Python feature that solves it?
* Can a function handle this instead of a class?
* Does this improve testability?
* Does this hide important control flow?
* Does the team understand this vocabulary?
* Is the abstraction based on real variation or imagined variation?
* Would direct code be clearer today?

The most important question is:

```text
does this pattern make the code easier to change safely?
```

If not, it may be premature.

---

# Chapter Summary

Design patterns are named solution shapes for recurring design problems.

They are vocabulary for organizing change.

They are not mandatory recipes.

In Python, patterns are often lighter than in more rigid languages because functions, modules, protocols, decorators, context managers, dataclasses, and dictionaries can express many designs directly.

Factories centralize construction logic.

Strategies represent interchangeable behavior.

Adapters translate external interfaces into application interfaces.

Facades simplify access to complex subsystems.

Repositories isolate persistence operations.

Unit of Work groups changes into transaction boundaries.

Commands represent actions as data or objects.

Observers notify multiple listeners about events.

Dependency Injection makes dependencies explicit and testable.

Context objects group related request or workflow information.

Template Method defines a fixed workflow with customizable steps, though composition is often preferable in Python.

Decorators wrap behavior while preserving the interface.

Singletons are often unnecessary in Python and should be treated carefully.

Registries and dispatch tables map names to behavior.

State patterns help when behavior varies substantially by state.

Builders are useful only when construction complexity justifies them.

Proxies control access to another object.

Chain of Responsibility passes work through ordered handlers.

Null Objects replace absence with intentional do-nothing behavior.

Specifications represent named business rules when those rules need reuse or composition.

Service layers coordinate workflows across components.

Domain models put business rules near business concepts.

Data mappers separate persistence representation from domain objects.

The central lesson is:

```text
patterns are useful when they make real change easier
```

Use them with taste.

Let Python stay Python.

---

# Exercises

1. Replace a long `if`/`elif` export format block with a dispatch table.

2. Create a factory function that builds a storage implementation from configuration.

3. Implement a discount strategy first as a function, then as a class with state.

4. Write an adapter around a fake third-party email client.

5. Create a repository for user lookup and test it with a fake in-memory implementation.

6. Implement a Unit of Work using a context manager.

7. Represent a background job request as a command dataclass and write a handler for it.

8. Build a small event bus and publish a `UserCreated` event to two handlers.

9. Refactor hidden construction inside a service into dependency injection.

10. Take one over-designed pattern-heavy example and simplify it using ordinary Python functions or data structures.

---

# Preview of Chapter 81

Chapter 80 studied design patterns.

We saw how recurring design pressures can be handled with factories, strategies, adapters, facades, repositories, unit of work, commands, observers, dependency injection, context objects, decorators, dispatch tables, and other Pythonic structures.

Next we study SOLID principles.

SOLID is another design vocabulary.

It focuses on object-oriented design principles:

* Single Responsibility Principle
* Open/Closed Principle
* Liskov Substitution Principle
* Interface Segregation Principle
* Dependency Inversion Principle

Like patterns, SOLID can help or harm depending on how it is used.

The goal is not to create abstract architecture for its own sake.

The goal is to manage change, dependencies, and substitutability.

The transition is:

```text
design patterns name recurring structures
SOLID names design pressures inside those structures
```

Chapter 81 will explain SOLID in Python terms, with practical examples and without turning principles into rigid slogans.
