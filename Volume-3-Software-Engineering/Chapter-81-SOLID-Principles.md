# Chapter 81 — SOLID Principles

SOLID is a set of five object-oriented design principles.

The letters stand for:

```text
S - Single Responsibility Principle
O - Open/Closed Principle
L - Liskov Substitution Principle
I - Interface Segregation Principle
D - Dependency Inversion Principle
```

These principles are often taught as if they are laws.

They are not laws.

They are design lenses.

They help you notice when code is becoming hard to change, hard to substitute, hard to test, or hard to understand.

Like design patterns, SOLID can help or harm.

Used with judgment, SOLID gives useful vocabulary:

```text
this class has too many reasons to change
this subclass cannot safely replace its parent
this interface forces callers to depend on methods they do not use
this high-level policy depends directly on low-level infrastructure
```

Used without judgment, SOLID can produce abstract, fragmented code that feels engineered but is difficult to read.

Python needs a Pythonic reading of SOLID.

Python has:

* duck typing
* first-class functions
* protocols
* dataclasses
* modules
* decorators
* context managers
* dynamic dispatch
* simple composition

So the goal is not to copy Java-style designs into Python.

The goal is to understand the design pressure behind each principle and choose a shape that fits.

---

# Why SOLID Exists

SOLID exists because software changes.

When code is small, almost any structure can work.

As code grows, some shapes become painful.

Pain appears as:

* one change requires editing many unrelated places
* tests need huge setup
* subclasses break callers
* abstractions are too large
* low-level details leak into business logic
* new behavior requires changing stable code
* simple features require risky edits
* code cannot be reused without dragging a large dependency graph

SOLID principles help name these pains.

They do not automatically solve them.

They help you ask better questions.

Good design starts with better questions.

---

# SOLID and Python

Python is not a statically compiled class-only language.

This matters.

In Python, you can often replace a class hierarchy with:

* a function
* a protocol
* a small dataclass
* a dictionary dispatch table
* a module-level function
* a context manager
* explicit dependency injection

For example, dependency inversion does not require a large interface hierarchy.

It may only require passing a callable:

```python
def send_welcome_email(email: str, send: Callable[[str, str], None]) -> None:
    send(email, "Welcome")
```

This can be enough.

Pythonic SOLID is not about maximizing classes.

It is about minimizing harmful coupling.

---

# Single Responsibility Principle

The Single Responsibility Principle, or SRP, says:

```text
a module, class, or function should have one reason to change
```

This phrase is often misunderstood.

It does not mean:

```text
each class should have only one method
```

It does not mean:

```text
split every function into tiny fragments
```

It means that unrelated change pressures should not be forced into the same unit.

For example, if one class changes when pricing rules change and also when PDF formatting changes and also when database schema changes, it has multiple reasons to change.

That class is likely doing too much.

---

# SRP Example

Consider:

```python
class InvoiceService:
    def create_invoice(self, order):
        total = sum(item.price * item.quantity for item in order.items)
        tax = total * Decimal("0.18")
        invoice = Invoice(order.id, total + tax)

        database.save(invoice)

        html = f"<h1>Invoice {invoice.id}</h1>"
        pdf = pdf_renderer.render(html)

        email.send(
            to=order.customer_email,
            subject="Your invoice",
            attachment=pdf,
        )

        return invoice
```

This code has many responsibilities:

* calculate invoice totals
* apply tax rules
* create invoice objects
* save to database
* render PDF
* send email

These responsibilities change for different reasons.

Tax rules may change because finance policy changes.

PDF rendering may change because branding changes.

Database saving may change because persistence changes.

Email sending may change because notification infrastructure changes.

SRP says these change pressures should not all be tangled in one class.

---

# Applying SRP

One possible refactor:

```python
class InvoiceCalculator:
    def calculate(self, order: Order) -> Money:
        subtotal = sum(item.price * item.quantity for item in order.items)
        tax = subtotal * Decimal("0.18")
        return subtotal + tax


class InvoiceRenderer:
    def render_pdf(self, invoice: Invoice) -> bytes:
        html = f"<h1>Invoice {invoice.id}</h1>"
        return pdf_renderer.render(html)


class InvoiceService:
    def __init__(self, calculator, invoices, renderer, email_sender):
        self.calculator = calculator
        self.invoices = invoices
        self.renderer = renderer
        self.email_sender = email_sender

    def create_invoice(self, order: Order) -> Invoice:
        total = self.calculator.calculate(order)
        invoice = Invoice(order.id, total)
        self.invoices.save(invoice)
        pdf = self.renderer.render_pdf(invoice)
        self.email_sender.send_invoice(order.customer_email, pdf)
        return invoice
```

Now responsibilities are separated.

The service coordinates.

The calculator calculates.

The renderer renders.

The repository saves.

The sender sends.

This may be better when the workflow is important and each part changes independently.

But do not split code merely because you can.

If a script has five lines and one user, direct code may be clearer.

SRP is about change pressure.

---

# SRP and Functions

SRP applies to functions too.

Problematic function:

```python
def import_users(path):
    text = path.read_text()
    rows = csv.loads(text)
    users = []
    for row in rows:
        email = row["email"].strip().lower()
        if "@" not in email:
            raise ValueError("invalid email")
        users.append(User(email=email))
    database.save_all(users)
    email_admin(f"Imported {len(users)} users")
```

This function:

* reads files
* parses CSV
* validates rows
* creates domain objects
* saves to database
* sends notification

Refactor around stages:

```python
def parse_user_rows(text: str) -> list[UserRow]:
    ...


def row_to_user(row: UserRow) -> User:
    ...


def import_users(path: Path, users: UserRepository, notifier: Notifier) -> int:
    rows = parse_user_rows(path.read_text())
    imported = [row_to_user(row) for row in rows]
    users.save_all(imported)
    notifier.import_finished(len(imported))
    return len(imported)
```

Now pure transformation can be tested separately from I/O.

That is SRP producing practical testability.

---

# SRP Overcorrection

SRP can be over-applied.

Bad split:

```python
class EmailStringStripper:
    def strip(self, email: str) -> str:
        return email.strip()


class EmailLowercaser:
    def lowercase(self, email: str) -> str:
        return email.lower()


class EmailNormalizer:
    ...
```

This may be absurd.

The responsibility is not:

```text
one line of code
```

The responsibility is:

```text
normalize an email address
```

Good SRP finds meaningful cohesion.

It does not atomize code into dust.

---

# Open/Closed Principle

The Open/Closed Principle, or OCP, says:

```text
software entities should be open for extension but closed for modification
```

This means code should allow new behavior to be added without constantly editing stable, tested code.

It does not mean code can never be modified.

That would be impossible.

It means that when variation is expected, design should give variation a place to go.

Classic problem:

```python
def calculate_shipping(order, method):
    if method == "standard":
        return Decimal("50")
    if method == "express":
        return Decimal("150")
    if method == "international":
        return Decimal("500")
    raise ValueError(f"unknown shipping method: {method}")
```

Every new method edits this function.

If shipping methods change often, this function becomes unstable.

---

# Applying OCP With Strategy

Use strategies:

```python
class ShippingMethod(Protocol):
    def cost(self, order: Order) -> Money:
        ...


class StandardShipping:
    def cost(self, order: Order) -> Money:
        return Money("50")


class ExpressShipping:
    def cost(self, order: Order) -> Money:
        return Money("150")
```

Then:

```python
def calculate_shipping(order: Order, method: ShippingMethod) -> Money:
    return method.cost(order)
```

Adding a new method means adding a new class.

The calculation function stays stable.

This is useful if shipping behavior changes often or is configured externally.

But if there are only two methods and they never change, the conditional may be perfectly fine.

OCP is valuable when extension is real.

Imaginary extension creates unnecessary abstraction.

---

# OCP With Dispatch Tables

Python can support OCP with data structures.

Example:

```python
SHIPPING_METHODS = {
    "standard": standard_shipping_cost,
    "express": express_shipping_cost,
}


def calculate_shipping(order: Order, method_name: str) -> Money:
    try:
        method = SHIPPING_METHODS[method_name]
    except KeyError:
        raise ValueError(f"unknown shipping method: {method_name}")
    return method(order)
```

New methods can be registered:

```python
SHIPPING_METHODS["pickup"] = pickup_shipping_cost
```

This may be simpler than class-based strategy.

Python gives lightweight extension mechanisms.

Use them.

---

# OCP and Plugins

OCP appears strongly in plugin systems.

Core application:

```python
class Exporter(Protocol):
    def export(self, data: Data) -> bytes:
        ...
```

Plugin:

```python
class CsvExporter:
    def export(self, data: Data) -> bytes:
        ...
```

The core knows the interface.

Plugins provide implementations.

This allows extension without changing core code.

Packaging entry points, discussed in Chapter 76, can advertise plugin implementations.

OCP often connects to packaging, protocols, factories, registries, and adapters.

Principles do not live alone.

They reinforce one another.

---

# OCP Overcorrection

OCP can be abused by making everything extensible before extension exists.

Bad:

```python
class UsernameNormalizationStrategy(Protocol):
    ...
```

for:

```python
def normalize_username(username: str) -> str:
    return username.strip().lower()
```

Maybe the function is enough.

Premature extensibility adds:

* interfaces
* factories
* configuration
* tests for wiring
* more files
* more names

All before the variation exists.

The better question:

```text
is this part likely to vary independently?
```

If yes, design for extension.

If no, keep it direct.

---

# Liskov Substitution Principle

The Liskov Substitution Principle, or LSP, says:

```text
objects of a subtype should be usable wherever objects of the base type are expected
```

In simpler terms:

```text
a subclass should not break the promises of its parent
```

If code expects a `Bird`, and you pass a `Penguin`, the program should still behave correctly according to the `Bird` contract.

The classic mistake:

```python
class Bird:
    def fly(self) -> None:
        ...


class Penguin(Bird):
    def fly(self) -> None:
        raise NotImplementedError("penguins cannot fly")
```

The subclass violates the parent contract.

If `Bird` promises `fly`, every `Bird` must be safely flyable.

The problem is not the penguin.

The problem is the abstraction.

---

# Fixing LSP Problems

Better model:

```python
class Bird:
    ...


class FlyingBird(Bird):
    def fly(self) -> None:
        ...


class Penguin(Bird):
    def swim(self) -> None:
        ...
```

Now only flying birds promise `fly`.

Another Pythonic option is a protocol:

```python
class Flyable(Protocol):
    def fly(self) -> None:
        ...
```

Functions that need flying behavior ask for `Flyable`:

```python
def migrate(bird: Flyable) -> None:
    bird.fly()
```

This avoids forcing all birds into one inheritance tree.

LSP often tells you that inheritance is modeling the wrong relationship.

---

# LSP and Exceptions

A subtype should not surprise callers with stricter requirements or unexpected failures.

Base class:

```python
class Storage:
    def save(self, name: str, content: bytes) -> None:
        ...
```

Bad subtype:

```python
class ImageStorage(Storage):
    def save(self, name: str, content: bytes) -> None:
        if not name.endswith(".png"):
            raise ValueError("only png files allowed")
        ...
```

If callers expect general `Storage`, they may save `"report.pdf"`.

`ImageStorage` is not substitutable for general `Storage`.

Better:

```python
class ImageStorage:
    def save_image(self, name: str, content: bytes) -> None:
        ...
```

or define a narrower protocol:

```python
class PngStorage(Protocol):
    def save_png(self, name: str, content: bytes) -> None:
        ...
```

Subtypes should not narrow the contract in ways callers do not expect.

---

# LSP and Return Values

Subtypes should preserve return expectations too.

Base:

```python
class UserRepository:
    def find(self, email: str) -> User | None:
        ...
```

Subtype:

```python
class RaisingUserRepository(UserRepository):
    def find(self, email: str) -> User:
        raise LookupError(email)
```

This changes behavior.

Maybe returning `User` is type-compatible in a narrow static sense, but semantically the method no longer follows the base contract.

The base says missing users return `None`.

The subtype raises.

Callers written for the base may break.

LSP is not only about type signatures.

It is about behavioral promises.

---

# LSP and Mutability

Mutability can break substitutability.

Suppose:

```python
class ReadOnlyList:
    def get(self, index: int) -> str:
        ...
```

and:

```python
class MutableList(ReadOnlyList):
    def append(self, value: str) -> None:
        ...
```

This can be fine because mutable list supports read-only behavior.

But the reverse is unsafe.

A read-only object cannot replace a mutable object if callers expect mutation.

This connects to type variance from Chapter 78.

If callers mutate, substitutability is stricter.

Interfaces should express what callers really need.

---

# LSP and Tests

You can test substitutability with shared tests.

Example:

```python
def storage_contract(storage: Storage) -> None:
    storage.save("hello.txt", b"hello")
    assert storage.load("hello.txt") == b"hello"
```

Run it against implementations:

```python
def test_local_storage_contract(tmp_path):
    storage_contract(LocalStorage(tmp_path))


def test_memory_storage_contract():
    storage_contract(MemoryStorage())
```

Contract tests make LSP practical.

They verify that different implementations keep the same promise.

This is especially useful for repositories, caches, storage systems, and external adapters.

---

# Interface Segregation Principle

The Interface Segregation Principle, or ISP, says:

```text
clients should not be forced to depend on methods they do not use
```

In Python terms:

```text
ask for the capability you need, not a giant object
```

Bad interface:

```python
class Worker(Protocol):
    def work(self) -> None:
        ...

    def eat(self) -> None:
        ...

    def sleep(self) -> None:
        ...
```

If a robot worker can work but cannot eat or sleep, the interface is too broad.

Better:

```python
class Workable(Protocol):
    def work(self) -> None:
        ...
```

Functions that only need work should depend on `Workable`.

---

# ISP in Application Code

Suppose a function only needs to publish events.

Bad:

```python
def create_user(app: Application, email: str) -> User:
    user = app.database.users.create(email)
    app.email.send_welcome(email)
    app.metrics.increment("user.created")
    app.event_bus.publish(UserCreated(user.id))
    return user
```

This function depends on the entire `Application`.

Better:

```python
class UserCreatorDependencies(Protocol):
    users: UserRepository
    email: EmailSender
    metrics: Metrics
    events: EventBus
```

or even more explicit:

```python
def create_user(
    email: str,
    users: UserRepository,
    email_sender: EmailSender,
    metrics: Metrics,
    events: EventBus,
) -> User:
    ...
```

The function should depend only on what it uses.

This improves tests and reduces coupling.

---

# ISP With Protocols

Protocols make ISP natural in Python.

Instead of passing a large concrete client:

```python
class CloudClient:
    def upload(self, ...): ...
    def download(self, ...): ...
    def delete(self, ...): ...
    def list_buckets(self, ...): ...
    def create_bucket(self, ...): ...
```

Define the narrow capability:

```python
class Uploader(Protocol):
    def upload(self, name: str, content: bytes) -> None:
        ...
```

Function:

```python
def publish_report(report: bytes, storage: Uploader) -> None:
    storage.upload("report.pdf", report)
```

Tests can pass a tiny fake uploader.

Production can pass a full cloud client if it has a compatible `upload`.

The caller does not care about the rest.

---

# ISP Overcorrection

ISP can be overdone.

Do not create a separate protocol for every single method unless it helps.

This may be too much:

```python
class HasEmail(Protocol):
    email: str


class HasName(Protocol):
    name: str


class HasId(Protocol):
    id: str
```

for code that simply needs a `User`.

Use narrow interfaces when they reduce coupling.

Use concrete domain types when they express the concept clearly.

Again, the principle is a lens.

Not a command to fragment everything.

---

# Dependency Inversion Principle

The Dependency Inversion Principle, or DIP, says:

```text
high-level policy should not depend directly on low-level details
both should depend on abstractions
```

High-level policy is business logic.

Low-level details are things like:

* databases
* email providers
* payment SDKs
* file systems
* HTTP clients
* queues
* caches

Bad:

```python
class CheckoutService:
    def checkout(self, order: Order) -> Receipt:
        stripe = stripe_sdk.Client(api_key=settings.STRIPE_KEY)
        payment = stripe.PaymentIntent.create(
            amount=order.total.cents,
            currency="inr",
        )
        order.mark_paid(payment.id)
        database.save(order)
        return Receipt(order.id, payment.id)
```

Checkout policy depends directly on Stripe SDK and database globals.

This is hard to test and hard to change.

---

# Applying DIP

Define capabilities:

```python
class PaymentGateway(Protocol):
    def charge(self, order: Order) -> PaymentResult:
        ...


class OrderRepository(Protocol):
    def save(self, order: Order) -> None:
        ...
```

High-level service:

```python
class CheckoutService:
    def __init__(self, payments: PaymentGateway, orders: OrderRepository):
        self.payments = payments
        self.orders = orders

    def checkout(self, order: Order) -> Receipt:
        payment = self.payments.charge(order)
        order.mark_paid(payment.id)
        self.orders.save(order)
        return Receipt(order.id, payment.id)
```

Low-level implementation:

```python
class StripePaymentGateway:
    def __init__(self, client):
        self.client = client

    def charge(self, order: Order) -> PaymentResult:
        response = self.client.PaymentIntent.create(
            amount=order.total.cents,
            currency="inr",
        )
        return PaymentResult(id=response.id)
```

Now checkout depends on a payment capability.

Stripe details live in the adapter.

This is dependency inversion.

---

# DIP Is Not Just Interfaces

DIP is often mistaken for:

```text
make an interface for every class
```

That is not the point.

The point is dependency direction.

High-level code should not be forced to know low-level details.

In Python, the abstraction can be:

* a protocol
* a callable
* a simple object with the needed method
* a dataclass configuration
* a function parameter

Example with callable:

```python
def retry(operation: Callable[[], T], attempts: int) -> T:
    ...
```

`retry` does not depend on any specific operation.

It depends on a callable abstraction.

That is enough.

---

# DIP and Testing

DIP improves testing because dependencies can be replaced.

Test:

```python
class FakePaymentGateway:
    def __init__(self):
        self.charges = []

    def charge(self, order: Order) -> PaymentResult:
        self.charges.append(order.id)
        return PaymentResult(id=PaymentId("pay_test"))
```

Use:

```python
def test_checkout_marks_order_paid():
    payments = FakePaymentGateway()
    orders = FakeOrderRepository()
    service = CheckoutService(payments, orders)

    receipt = service.checkout(order)

    assert order.status == "paid"
    assert receipt.payment_id == PaymentId("pay_test")
```

No Stripe.

No real database.

No network.

The high-level behavior is testable.

This is not accidental.

It is a design result.

---

# DIP and Application Wiring

If high-level code receives abstractions, something must create concrete objects.

That place is often application startup.

Example:

```python
def build_checkout_service(settings: Settings) -> CheckoutService:
    stripe_client = create_stripe_client(settings)
    payments = StripePaymentGateway(stripe_client)
    orders = SqlOrderRepository(create_session_factory(settings))
    return CheckoutService(payments, orders)
```

This wiring code knows details.

That is fine.

Details must exist somewhere.

DIP does not eliminate details.

It keeps details from invading high-level policy.

The composition root is allowed to know concrete implementations.

---

# SOLID Principles Together

The principles overlap.

SRP says:

```text
separate unrelated reasons to change
```

OCP says:

```text
give expected variation an extension point
```

LSP says:

```text
substitutes must keep promises
```

ISP says:

```text
depend on the capability you actually need
```

DIP says:

```text
high-level policy should not depend on low-level details
```

Together, they push toward:

* cohesive units
* explicit boundaries
* substitutable implementations
* narrow interfaces
* stable high-level code
* testable dependencies

But they can also push toward over-abstraction if applied mechanically.

Balance matters.

---

# SOLID and Design Patterns

SOLID and design patterns often reinforce each other.

Examples:

* Strategy supports OCP.
* Adapter supports DIP.
* Repository supports SRP and DIP.
* Unit of Work supports SRP around transactions.
* Decorator can support OCP.
* Protocols support ISP and DIP.
* Factory can isolate construction details.

But a pattern does not automatically mean SOLID.

You can create a repository that violates SRP.

You can create a strategy hierarchy that violates LSP.

You can create an adapter that leaks vendor details everywhere.

The pattern is structure.

The principle is pressure.

Use both thoughtfully.

---

# SOLID and Duck Typing

Python's duck typing changes how SOLID feels.

You do not always need explicit inheritance.

If a function needs:

```python
sender.send(message)
```

then any object with a compatible `send` method works.

With type hints, a protocol can make that visible:

```python
class Sender(Protocol):
    def send(self, message: Message) -> None:
        ...
```

This supports ISP and DIP without forcing inheritance.

Python often prefers:

```text
small capability protocols
composition
plain objects
functions
```

over deep class hierarchies.

This is SOLID in Python's accent.

---

# SOLID and Modules

SOLID is usually discussed with classes.

But modules also have responsibilities and dependencies.

A module can have too many reasons to change.

Example:

```text
orders.py
```

containing:

* ORM models
* API handlers
* payment logic
* email templates
* report generation
* CLI commands

This module violates SRP at module level.

Better structure:

```text
orders/
    models.py
    service.py
    repository.py
    api.py
    events.py
```

Do not split too early.

But as change pressure grows, module boundaries matter.

Architecture begins with module organization.

Chapter 82 will continue that discussion.

---

# SOLID and Dataclasses

Dataclasses are often used as simple data carriers.

That is fine.

But a dataclass can also hold behavior when behavior belongs with the data.

An anemic design:

```python
@dataclass
class Order:
    status: str
    total: Money
```

All rules elsewhere:

```python
def mark_paid(order: Order, payment_id: PaymentId) -> None:
    if order.status != "pending":
        raise ValueError
    order.status = "paid"
```

Maybe better:

```python
@dataclass
class Order:
    status: str
    total: Money
    payment_id: PaymentId | None = None

    def mark_paid(self, payment_id: PaymentId) -> None:
        if self.status != "pending":
            raise ValueError("only pending orders can be paid")
        self.status = "paid"
        self.payment_id = payment_id
```

SRP does not mean data and behavior must always be separated.

It means behavior should live where it has cohesive reasons to change.

---

# SOLID and Frameworks

Frameworks often shape design.

Web frameworks may encourage:

* route handlers
* models
* serializers
* dependency injection
* middleware
* service layers

ORMs may encourage active record models or data mapper models.

SOLID should be applied within the framework's reality.

Do not fight a framework so hard that your code becomes unnatural.

But do not let framework convenience put all business logic in route handlers.

Practical boundary:

```text
framework layer translates requests
application layer coordinates use cases
domain layer holds core rules
infrastructure layer handles external details
```

This is not mandatory for every app.

It is a useful direction as complexity grows.

---

# SOLID and Small Scripts

Small scripts do not need enterprise structure.

This is fine:

```python
def main():
    rows = read_csv("users.csv")
    users = normalize(rows)
    write_json("users.json", users)
```

Do not create:

```text
CsvReaderFactory
UserNormalizationStrategyProvider
JsonWriterAdapter
```

unless the script is growing into a system.

SOLID principles become more valuable as:

* code changes often
* multiple people work on it
* testing matters
* dependencies vary
* behavior grows
* integration boundaries multiply

For small one-off code, clarity beats abstraction.

---

# A Practical SOLID Refactor

Start with:

```python
def send_invoice(order_id: str) -> None:
    order = database.get_order(order_id)
    total = sum(item.price * item.quantity for item in order.items)
    pdf = render_invoice_pdf(order, total)
    smtp = SmtpClient(settings.SMTP_HOST)
    smtp.send(order.customer_email, "Invoice", pdf)
    database.mark_invoice_sent(order_id)
```

Problems:

* database access is global
* SMTP construction is hidden
* total calculation is embedded
* rendering is mixed with workflow
* testing requires database and SMTP unless patched

Refactor:

```python
class InvoiceSender:
    def __init__(self, orders, calculator, renderer, email_sender):
        self.orders = orders
        self.calculator = calculator
        self.renderer = renderer
        self.email_sender = email_sender

    def send_invoice(self, order_id: OrderId) -> None:
        order = self.orders.get(order_id)
        total = self.calculator.total(order)
        pdf = self.renderer.render(order, total)
        self.email_sender.send(order.customer_email, "Invoice", pdf)
        self.orders.mark_invoice_sent(order_id)
```

This applies:

* SRP: calculation, rendering, sending, persistence separated
* DIP: service depends on capabilities
* ISP: each dependency can be narrow
* OCP: renderer or sender can vary
* LSP: fake and real dependencies should satisfy the same contract

The result is not abstract for its own sake.

It is easier to test and change.

---

# When SOLID Hurts

SOLID hurts when it becomes mechanical.

Warning signs:

* every class has an interface with one implementation
* every simple function becomes a strategy
* code is split so much that behavior is hard to follow
* abstractions are named after technical patterns, not domain concepts
* tests mostly verify wiring
* adding a feature requires touching many tiny files
* developers cannot explain why an abstraction exists

The cure is to return to design pressure.

Ask:

```text
what change is this abstraction protecting?
```

If nobody knows, the abstraction may not be earning its place.

---

# SOLID Checklist

When reviewing design, ask:

* Does this unit have one coherent reason to change?
* Is expected variation isolated?
* Can substitutes keep the same behavioral promise?
* Are callers depending only on capabilities they need?
* Does high-level policy avoid low-level infrastructure details?
* Is this abstraction based on real change pressure?
* Would a simpler Python construct work?
* Are tests easier or harder because of this design?
* Do names reflect domain concepts?
* Can a new developer follow the flow?

SOLID is not a checklist to satisfy blindly.

It is a way to notice design risk.

---

# Chapter Summary

SOLID is a set of five object-oriented design principles.

The Single Responsibility Principle says a unit should have one coherent reason to change.

It is about change pressure, not tiny classes.

The Open/Closed Principle says code should be open for extension and closed for modification when variation is expected.

It is useful for real extension points, not imaginary future flexibility.

The Liskov Substitution Principle says subtypes should be safely substitutable for their base types.

It is about behavioral promises, not only matching method names.

The Interface Segregation Principle says clients should not depend on methods they do not use.

In Python, protocols and duck typing make narrow capability interfaces natural.

The Dependency Inversion Principle says high-level policy should not depend directly on low-level details.

In Python, dependency inversion can use protocols, callables, simple objects, or explicit parameters.

SOLID principles overlap with design patterns.

They also apply to modules, functions, services, and package boundaries, not only classes.

Pythonic SOLID favors composition, protocols, functions, explicit dependencies, and cohesive modules over unnecessary inheritance hierarchies.

The central lesson is:

```text
SOLID is useful when it helps code change safely
```

It should make code clearer, more testable, and more adaptable.

It should not turn simple Python into ceremonial architecture.

---

# Exercises

1. Find a function with multiple responsibilities and split it into cohesive pieces.

2. Refactor a conditional that selects behavior into a dispatch table or strategy.

3. Create a base class/subclass example that violates LSP, then fix the abstraction.

4. Replace a broad dependency with a narrow protocol that exposes only the method a function needs.

5. Refactor a service that constructs its own external client so the dependency is injected.

6. Write contract tests for two implementations of the same repository or storage protocol.

7. Identify an over-engineered abstraction and simplify it.

8. Take a route handler that contains business logic and move the workflow into an application service.

9. Use a fake implementation to test a service that follows DIP.

10. Review a module and list its reasons to change.

---

# Preview of Chapter 82

Chapter 81 studied SOLID principles.

We learned how SRP, OCP, LSP, ISP, and DIP describe design pressures around responsibility, extension, substitutability, interface size, and dependency direction.

Next we study architecture.

Architecture is design at a larger scale.

It asks how modules, packages, layers, boundaries, dependencies, data flow, deployment shape, and operational concerns fit together.

Architecture includes questions such as:

* where should business logic live?
* what should depend on what?
* how should external systems be isolated?
* how should modules be organized?
* where are transaction boundaries?
* how does data move through the system?
* how does the system remain testable?
* how does the system evolve without collapsing?

The transition is:

```text
SOLID improves local design pressure
architecture organizes the whole system
```

Chapter 82 will zoom out from classes and services to the structure of Python applications as complete systems.
