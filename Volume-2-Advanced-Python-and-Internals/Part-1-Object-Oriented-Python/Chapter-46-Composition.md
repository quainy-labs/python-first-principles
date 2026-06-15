# Chapter 46 — Composition

---

# Learning Objectives

By the end of this chapter, you should understand:

* What composition means in object-oriented programming.
* How objects can be built from other objects.
* The difference between "has-a" and "is-a" relationships.
* Why composition is often simpler than inheritance.
* How encapsulation and composition work together.
* How delegation works.
* How composed objects affect ownership and lifetime.
* How dependency injection supports composition.
* How composition improves testing.
* How to avoid over-composition.
* How composition prepares us to evaluate inheritance properly.

Chapter 45 focused on object boundaries.

Composition focuses on object relationships.

Encapsulation asks:

```text
What should this object expose?
What should it keep internal?
```

Composition asks:

```text
What other objects should this object use to do its job?
```

This is one of the most important design ideas in object-oriented programming.

Before we study inheritance, we need composition.

Why?

Because many problems that beginners try to solve with inheritance are better solved by connecting objects together.

---

# Concept Overview

Composition means building an object out of other objects.

Example:

```python
class Engine:
    def start(self):
        print("engine starting")


class Car:
    def __init__(self):
        self.engine = Engine()

    def start(self):
        self.engine.start()
```

Use:

```python
car = Car()
car.start()
```

Output:

```text
engine starting
```

The `Car` object contains an `Engine` object.

Relationship:

```text
Car has an Engine
```

This is composition.

The car is not an engine.

The car has an engine.

That distinction matters.

---

# Has-A vs Is-A

Two common object relationships:

```text
has-a
is-a
```

Composition models has-a relationships.

Inheritance models is-a relationships.

Examples:

```text
Car has an Engine.
House has Rooms.
Order has LineItems.
Playlist has Songs.
Report has a Formatter.
Service has a Repository.
```

Inheritance examples:

```text
Dog is an Animal.
SavingsAccount is an Account.
CsvReport is a Report.
```

Beginners often overuse inheritance because it feels like the main object-oriented tool.

But in real Python design, composition is often the first tool to consider.

Question:

```text
Is this object a kind of that object?
Or does this object use that object?
```

If it uses or owns another object, composition is probably the better model.

---

# Why Composition Matters

Composition gives programs flexible structure.

Example:

```python
class EmailSender:
    def send(self, message):
        print("email:", message)


class NotificationService:
    def __init__(self):
        self.sender = EmailSender()

    def notify(self, message):
        self.sender.send(message)
```

The notification service uses a sender.

Later, we can change the sender:

```python
class SmsSender:
    def send(self, message):
        print("sms:", message)
```

If `NotificationService` is designed well, it can use either sender.

Composition lets behavior vary by swapping collaborator objects.

This is powerful.

Instead of building one huge class that does everything, we can build small objects that cooperate.

---

# Objects as Collaborators

In composition, objects collaborate.

Example:

```python
class Formatter:
    def format(self, task):
        marker = "x" if task["done"] else " "
        return f"[{marker}] {task['title']}"


class TaskPrinter:
    def __init__(self, formatter):
        self.formatter = formatter

    def print_task(self, task):
        print(self.formatter.format(task))
```

Use:

```python
formatter = Formatter()
printer = TaskPrinter(formatter)

task = {"title": "learn composition", "done": False}
printer.print_task(task)
```

Output:

```text
[ ] learn composition
```

The `TaskPrinter` does not know formatting details.

It delegates formatting to `Formatter`.

Relationship:

```text
TaskPrinter has a Formatter
TaskPrinter uses Formatter to do part of its job
```

This keeps responsibilities separate.

---

# Composition and Encapsulation

Composition works best with encapsulation.

Example:

```python
class Engine:
    def start(self):
        print("engine starting")


class Car:
    def __init__(self):
        self._engine = Engine()

    def start(self):
        self._engine.start()
```

The car exposes:

```python
car.start()
```

It keeps internal:

```python
car._engine
```

Callers do not need to know exactly how the car starts.

Maybe today it uses an `Engine`.

Tomorrow it might use an `ElectricMotor`.

If the public interface stays:

```python
car.start()
```

callers do not need to change.

Encapsulation defines the boundary.

Composition fills the inside of that boundary with collaborating objects.

---

# Direct Composition

Direct composition means an object creates the object it uses.

Example:

```python
class FileStorage:
    def save(self, text):
        print("saving to file:", text)


class NotesApp:
    def __init__(self):
        self._storage = FileStorage()

    def save_note(self, text):
        self._storage.save(text)
```

Use:

```python
app = NotesApp()
app.save_note("learn Python")
```

This is simple.

`NotesApp` owns its storage object.

Direct composition is fine when:

* The dependency is simple.
* There is only one sensible implementation.
* You do not need to swap it in tests.
* The owning object should control creation.

But direct composition can become rigid.

If `NotesApp` must support file storage, database storage, and memory storage, we need more flexibility.

That leads to dependency injection.

---

# Dependency Injection

Dependency injection means giving an object its dependencies from the outside.

Instead of:

```python
class NotesApp:
    def __init__(self):
        self._storage = FileStorage()
```

write:

```python
class NotesApp:
    def __init__(self, storage):
        self._storage = storage

    def save_note(self, text):
        self._storage.save(text)
```

Use:

```python
storage = FileStorage()
app = NotesApp(storage)
```

Now `NotesApp` does not care which storage object it receives, as long as it supports:

```python
save(text)
```

This is composition with external wiring.

Dependency injection improves:

* Flexibility.
* Testability.
* Separation of responsibilities.
* Reuse.

The object receives collaborators instead of creating all of them internally.

---

# Swapping Collaborators

With dependency injection, we can swap collaborators.

Example:

```python
class FileStorage:
    def save(self, text):
        print("file:", text)


class MemoryStorage:
    def __init__(self):
        self.items = []

    def save(self, text):
        self.items.append(text)
```

Application:

```python
class NotesApp:
    def __init__(self, storage):
        self._storage = storage

    def save_note(self, text):
        self._storage.save(text)
```

Use file storage:

```python
app = NotesApp(FileStorage())
app.save_note("hello")
```

Use memory storage:

```python
memory = MemoryStorage()
app = NotesApp(memory)
app.save_note("hello")

print(memory.items)
```

Output:

```text
['hello']
```

`NotesApp` did not change.

Only the collaborator changed.

That is composition doing real design work.

---

# Delegation

Delegation means one object passes part of its work to another object.

Example:

```python
class JsonFormatter:
    def format(self, data):
        return str(data)


class Report:
    def __init__(self, formatter):
        self._formatter = formatter

    def render(self):
        data = {"status": "ok"}
        return self._formatter.format(data)
```

The report handles report logic.

The formatter handles formatting.

`Report.render()` delegates formatting to:

```python
self._formatter.format(data)
```

Delegation is common in composed designs.

It lets one class stay focused.

Instead of one object doing everything, each collaborator handles its own responsibility.

Core idea:

```text
I know when formatting is needed.
My formatter knows how formatting works.
```

That division is clean.

---

# Ownership

Composition raises an ownership question:

```text
Who owns the collaborator object?
```

Example:

```python
class Car:
    def __init__(self):
        self._engine = Engine()
```

The car creates the engine.

It likely owns it.

If the car disappears, the engine likely disappears too.

But:

```python
engine = Engine()
car = Car(engine)
```

Now the engine was created outside.

Maybe the car uses it but does not fully own it.

Ownership affects:

* Object lifetime.
* Cleanup responsibility.
* Mutation responsibility.
* Sharing.
* Testing.

Ask:

```text
Does this object create the dependency?
Can other objects share the dependency?
Who is responsible for closing or cleaning it up?
```

Composition is not only structure.

It is ownership design.

---

# Lifetime

Object relationships affect lifetime.

Example:

```python
class Service:
    def __init__(self, database):
        self._database = database
```

As long as the `Service` object is alive, it keeps a strong reference to the database object.

Conceptually:

```text
service ──▶ database
```

If no other references exist, the database still stays alive through the service.

This is ordinary reference behavior from Volume I.

Composition creates object graphs.

Those graphs affect memory and cleanup.

If the collaborator manages an external resource, such as a file or socket, lifetime matters even more.

The owner must know who closes it.

Good composition makes ownership clear.

Bad composition leaves cleanup ambiguous.

---

# Shared Collaborators

Sometimes many objects share one collaborator.

Example:

```python
class Logger:
    def log(self, message):
        print(message)


class UserService:
    def __init__(self, logger):
        self._logger = logger


class OrderService:
    def __init__(self, logger):
        self._logger = logger
```

Use:

```python
logger = Logger()
users = UserService(logger)
orders = OrderService(logger)
```

Graph:

```text
users  ──▶ logger
orders ──▶ logger
```

This can be good.

One logger may be shared across the application.

But shared mutable collaborators need care.

If many objects mutate the same collaborator, behavior can become hard to reason about.

Ask:

```text
Is sharing intentional?
Is the collaborator safe to share?
Does it hold mutable state?
Who configures it?
```

---

# Composition vs Inheritance

Inheritance says:

```text
this class is a kind of another class
```

Composition says:

```text
this class uses another object
```

Suppose we need a report that can output JSON.

Inheritance approach:

```python
class JsonReport(Report):
    ...
```

Composition approach:

```python
class Report:
    def __init__(self, formatter):
        self._formatter = formatter
```

If formatting is a behavior that can vary, composition is often cleaner.

You can create:

```python
Report(JsonFormatter())
Report(HtmlFormatter())
Report(TextFormatter())
```

without building a subclass for every format.

Inheritance can be powerful.

But composition often gives flexibility with less hierarchy.

Before using inheritance, ask:

```text
Is this truly an is-a relationship?
Or am I just trying to reuse behavior?
```

---

# Reuse Through Composition

Composition reuses behavior by holding an object that provides that behavior.

Example:

```python
class Slugifier:
    def slugify(self, text):
        return text.lower().replace(" ", "-")


class ArticleService:
    def __init__(self, slugifier):
        self._slugifier = slugifier

    def create_slug(self, title):
        return self._slugifier.slugify(title)
```

Use:

```python
service = ArticleService(Slugifier())
print(service.create_slug("Hello World"))
```

Output:

```text
hello-world
```

`ArticleService` reuses slugification behavior.

It does not need to inherit from `Slugifier`.

This is often better than:

```python
class ArticleService(Slugifier):
    ...
```

because an article service is not a kind of slugifier.

It uses a slugifier.

That is composition.

---

# Avoiding Inheritance Just for Code Sharing

Bad inheritance:

```python
class EmailSender:
    def send_email(self, message):
        print("email:", message)


class UserService(EmailSender):
    def welcome_user(self, user):
        self.send_email(f"Welcome {user}")
```

Problem:

```text
UserService is not an EmailSender
UserService uses email sending
```

Better composition:

```python
class UserService:
    def __init__(self, email_sender):
        self._email_sender = email_sender

    def welcome_user(self, user):
        self._email_sender.send_email(f"Welcome {user}")
```

Now the relationship is accurate:

```text
UserService has an EmailSender
```

This avoids confusing type relationships.

Inheritance should model meaning, not just convenience.

---

# Testing With Composition

Composition makes testing easier.

Example:

```python
class EmailSender:
    def send(self, message):
        print("sending real email:", message)


class UserService:
    def __init__(self, sender):
        self._sender = sender

    def welcome(self, name):
        self._sender.send(f"Welcome {name}")
```

In production:

```python
service = UserService(EmailSender())
```

In tests:

```python
class FakeSender:
    def __init__(self):
        self.messages = []

    def send(self, message):
        self.messages.append(message)


fake = FakeSender()
service = UserService(fake)
service.welcome("Ada")

assert fake.messages == ["Welcome Ada"]
```

No real email is sent.

The service is easy to test because the sender is injected.

Composition helps separate business behavior from external effects.

---

# Fakes and Test Doubles

A fake object is a simple replacement used in tests.

Example:

```python
class FakeStorage:
    def __init__(self):
        self.saved = []

    def save(self, value):
        self.saved.append(value)
```

Use:

```python
storage = FakeStorage()
app = NotesApp(storage)

app.save_note("test")

assert storage.saved == ["test"]
```

The fake has the same method the app needs:

```python
save(value)
```

It does not need to be a subclass of real storage.

It only needs to behave the same for the required operation.

This connects to duck typing, which we will study later.

Composition and duck typing work beautifully together.

The object cares about what the collaborator can do, not necessarily its exact class.

---

# Protocol Thinking Without Formal Protocols

When using composition, you often depend on behavior.

Example:

```python
class NotesApp:
    def __init__(self, storage):
        self._storage = storage

    def save_note(self, text):
        self._storage.save(text)
```

`NotesApp` expects:

```text
storage has a save(text) method
```

That expectation is a protocol in the informal sense.

The storage object can be:

* `FileStorage`
* `MemoryStorage`
* `DatabaseStorage`
* `FakeStorage`

as long as it supports:

```python
save(text)
```

Later, we will study formal protocols and static typing.

For now, this is enough:

```text
composition often depends on expected behavior, not exact class
```

---

# Delegating a Public Method

Sometimes an object exposes a method that simply delegates.

Example:

```python
class Engine:
    def start(self):
        print("engine started")


class Car:
    def __init__(self, engine):
        self._engine = engine

    def start(self):
        self._engine.start()
```

Why not expose:

```python
car.engine.start()
```

Maybe because the car wants to keep the engine internal.

Public API:

```python
car.start()
```

Internal collaborator:

```python
car._engine
```

The car controls what starting means.

Maybe later:

```python
def start(self):
    self._battery.check()
    self._engine.start()
    self._dashboard.show_ready()
```

Callers still use:

```python
car.start()
```

Delegation preserves encapsulation.

---

# Delegation With Added Behavior

Delegation does not mean doing nothing except forwarding.

Example:

```python
class Storage:
    def save(self, text):
        print("saved:", text)


class NotesApp:
    def __init__(self, storage):
        self._storage = storage

    def save_note(self, text):
        text = text.strip()

        if text == "":
            raise ValueError("note cannot be empty")

        self._storage.save(text)
```

`NotesApp` handles note rules.

`Storage` handles saving.

This is better than putting every rule in storage or every storage detail in the app.

Good composition separates responsibilities:

```text
NotesApp -> note validation and workflow
Storage  -> persistence
```

Delegation often includes preparation, validation, transformation, or coordination.

---

# Object Graphs

Composition creates object graphs.

Example:

```python
class App:
    def __init__(self, users, orders):
        self.users = users
        self.orders = orders
```

Conceptually:

```text
app ──▶ user service ──▶ user repository ──▶ database
 │
 └──▶ order service ──▶ order repository ──▶ database
```

Object graphs matter because:

* They show dependencies.
* They show ownership.
* They show lifetimes.
* They show possible cycles.
* They show test boundaries.

In small examples, graphs are tiny.

In real applications, graphs can be large.

Good composition keeps the graph understandable.

Bad composition creates tangled dependency webs.

---

# Composition and Cycles

Composition can create cycles.

Example:

```python
class Parent:
    def __init__(self):
        self.children = []

    def add_child(self, child):
        child.parent = self
        self.children.append(child)
```

Use:

```python
parent = Parent()
child = Child()
parent.add_child(child)
```

Graph:

```text
parent ──▶ children list ──▶ child
   ▲                         │
   │                         ▼
   └──────── parent ◀────────┘
```

This cycle may be perfectly reasonable.

Python's garbage collector can handle many unreachable cycles.

But cycles affect ownership thinking.

If child should not keep parent alive, use a weak reference.

If parent owns child and child merely observes parent, weak references may fit.

Composition connects directly to memory management.

---

# Composition and Resource Cleanup

If a composed object owns a resource-owning object, cleanup matters.

Example:

```python
class FileWriter:
    def __init__(self, path):
        self._file = open(path, "w")

    def write(self, text):
        self._file.write(text)

    def close(self):
        self._file.close()
```

Composed owner:

```python
class ReportWriter:
    def __init__(self, file_writer):
        self._file_writer = file_writer

    def write_report(self, text):
        self._file_writer.write(text)
```

Who closes the file writer?

Possible answers:

* The creator closes it.
* The `ReportWriter` owns it and closes it.
* A context manager handles it.

The design must be clear.

Composition without cleanup responsibility can leak resources.

Ownership is not academic.

It affects correctness.

---

# Composition With Context Managers Preview

Context managers can express ownership and cleanup.

Example:

```python
class ReportWriter:
    def __init__(self, path):
        self._path = path
        self._file = None

    def __enter__(self):
        self._file = open(self._path, "w")
        return self

    def __exit__(self, exc_type, exc, tb):
        self._file.close()

    def write(self, text):
        self._file.write(text)
```

Use:

```python
with ReportWriter("report.txt") as writer:
    writer.write("hello")
```

We will study context managers deeply later.

For now, see the design connection:

```text
composition creates ownership
ownership may require cleanup
context managers express cleanup boundaries
```

Object design, memory management, and resource management are connected.

---

# Composition and Configuration

Composition is often used for configuration.

Example:

```python
class HtmlFormatter:
    def format(self, text):
        return f"<p>{text}</p>"


class MarkdownFormatter:
    def format(self, text):
        return f"**{text}**"


class Renderer:
    def __init__(self, formatter):
        self._formatter = formatter

    def render(self, text):
        return self._formatter.format(text)
```

Use:

```python
html_renderer = Renderer(HtmlFormatter())
markdown_renderer = Renderer(MarkdownFormatter())
```

Same `Renderer` class.

Different behavior through different collaborators.

Composition lets you configure behavior without creating many subclasses.

This is a major reason composition scales well.

---

# Composition and Small Classes

Composition works best when classes have focused responsibilities.

Bad:

```python
class Application:
    def parse_input(self): ...
    def validate_user(self): ...
    def save_database(self): ...
    def send_email(self): ...
    def render_html(self): ...
    def log_event(self): ...
```

This class does too much.

Better:

```text
Application
    uses Parser
    uses Validator
    uses Repository
    uses EmailSender
    uses Renderer
    uses Logger
```

Each collaborator can be understood and tested separately.

But do not overdo it.

One-method classes everywhere can make code fragmented.

Balance:

```text
split responsibilities when it reduces real complexity
keep things together when they naturally belong together
```

Composition should make code easier to understand, not more ceremonious.

---

# Composition and Modules

Sometimes a module is enough.

Example:

```python
# formatting.py

def format_task(task):
    marker = "x" if task.done else " "
    return f"[{marker}] {task.title}"
```

Using a class:

```python
class TaskFormatter:
    def format(self, task):
        marker = "x" if task.done else " "
        return f"[{marker}] {task.title}"
```

Which is better?

It depends.

Use a module function when:

* There is no state.
* There is no need to swap implementations.
* There is no configuration.
* The behavior is simple.

Use a composed object when:

* It holds configuration.
* It has multiple related methods.
* You want to swap behavior.
* You want test fakes.
* It represents a meaningful collaborator.

Composition is powerful, but functions and modules are still excellent tools.

---

# Composition and Data Structures

Sometimes a composed object wraps a data structure.

Example:

```python
class History:
    def __init__(self):
        self._items = []

    def record(self, item):
        self._items.append(item)

    def latest(self):
        if not self._items:
            return None

        return self._items[-1]

    @property
    def items(self):
        return tuple(self._items)
```

The class uses a list internally.

Public interface:

```python
history.record(item)
history.latest()
history.items
```

Internal detail:

```python
history._items
```

The list is composed inside the object.

This adds rules and meaning around a raw data structure.

Use this when the rules matter.

Do not wrap every list just because you can.

---

# Composition and APIs

Composition helps create stable APIs.

Example:

```python
class ReportService:
    def __init__(self, repository, formatter):
        self._repository = repository
        self._formatter = formatter

    def report_for_user(self, user_id):
        data = self._repository.load_user_data(user_id)
        return self._formatter.format(data)
```

Public API:

```python
service.report_for_user(user_id)
```

Internal collaborators:

```python
repository
formatter
```

Later, you can replace:

```text
DatabaseRepository with ApiRepository
JsonFormatter with HtmlFormatter
```

The public API can remain stable.

Composition lets internals vary behind an interface.

This is encapsulation at object-network scale.

---

# A Complete Example: Notifications

Collaborators:

```python
class EmailSender:
    def send(self, recipient, message):
        print(f"email to {recipient}: {message}")


class SmsSender:
    def send(self, recipient, message):
        print(f"sms to {recipient}: {message}")
```

Service:

```python
class NotificationService:
    def __init__(self, sender):
        self._sender = sender

    def notify(self, user, message):
        if user.get("disabled"):
            return

        self._sender.send(user["contact"], message)
```

Use:

```python
email_service = NotificationService(EmailSender())
sms_service = NotificationService(SmsSender())

user = {"contact": "ada@example.com", "disabled": False}

email_service.notify(user, "Welcome")
```

Same service logic.

Different sending strategy.

Composition avoids:

```text
EmailNotificationService
SmsNotificationService
PushNotificationService
```

unless those subclasses truly add meaningful type-specific behavior.

---

# A Complete Example: Order Total

Classes:

```python
class LineItem:
    def __init__(self, name, quantity, price):
        self.name = name
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.quantity * self.price


class Order:
    def __init__(self):
        self._items = []

    def add_item(self, item):
        self._items.append(item)

    def total(self):
        amount = 0

        for item in self._items:
            amount += item.total()

        return amount
```

Use:

```python
order = Order()
order.add_item(LineItem("Book", 2, 30))
order.add_item(LineItem("Pen", 3, 5))

print(order.total())
```

Output:

```text
75
```

Relationship:

```text
Order has LineItems
```

The order delegates item subtotal calculation to each line item.

This is natural composition.

---

# A Complete Example: Report Rendering

Formatters:

```python
class TextFormatter:
    def format(self, rows):
        return "\n".join(rows)


class HtmlFormatter:
    def format(self, rows):
        items = "".join(f"<li>{row}</li>" for row in rows)
        return f"<ul>{items}</ul>"
```

Report:

```python
class Report:
    def __init__(self, rows, formatter):
        self._rows = rows
        self._formatter = formatter

    def render(self):
        return self._formatter.format(self._rows)
```

Use:

```python
rows = ["one", "two"]

text_report = Report(rows, TextFormatter())
html_report = Report(rows, HtmlFormatter())

print(text_report.render())
print(html_report.render())
```

The report owns the workflow.

The formatter owns representation.

This is cleaner than putting every formatting style into one large `Report` class.

---

# A Complete Example: Repository and Service

Repository:

```python
class InMemoryUserRepository:
    def __init__(self):
        self._users = {}

    def save(self, user_id, user):
        self._users[user_id] = user

    def get(self, user_id):
        return self._users.get(user_id)
```

Service:

```python
class UserService:
    def __init__(self, repository):
        self._repository = repository

    def register(self, user_id, name):
        if name.strip() == "":
            raise ValueError("name cannot be empty")

        user = {"id": user_id, "name": name.strip()}
        self._repository.save(user_id, user)
        return user

    def find(self, user_id):
        return self._repository.get(user_id)
```

Use:

```python
repository = InMemoryUserRepository()
service = UserService(repository)

service.register(1, "Ada")
print(service.find(1))
```

Composition separates:

```text
UserService -> business rules
Repository  -> storage
```

This is a common professional pattern.

---

# Common Mistake: Using Inheritance for Has-A

Bad:

```python
class Engine:
    def start(self):
        print("engine")


class Car(Engine):
    pass
```

This says:

```text
Car is an Engine
```

But that is not true.

A car has an engine.

Better:

```python
class Car:
    def __init__(self, engine):
        self._engine = engine

    def start(self):
        self._engine.start()
```

Use inheritance for genuine is-a relationships.

Use composition for has-a relationships.

This simple distinction prevents many design problems.

---

# Common Mistake: Creating a Huge Manager Object

Composition can go wrong if one object simply collects everything.

Example:

```python
class AppManager:
    def __init__(self):
        self.users = UserService()
        self.orders = OrderService()
        self.emails = EmailSender()
        self.reports = ReportGenerator()
        self.cache = Cache()
        self.logger = Logger()
```

Maybe this is useful as an application container.

Maybe it is a sign that one class has become a global dumping ground.

Ask:

```text
Does this object have a clear responsibility?
Or is it just holding unrelated things?
```

Composition should organize responsibilities.

It should not create a vague object that owns the whole world.

---

# Common Mistake: Too Many Tiny Collaborators

Over-composition can make code harder to follow.

Example:

```text
UserNameNormalizer
UserNameTrimmer
UserNameTitleCaser
UserNameEmptyChecker
UserNameValidator
```

Maybe this is needed in a complex system.

Often, it is too much.

Simpler:

```python
class UserNameValidator:
    def normalize(self, name):
        return name.strip().title()

    def validate(self, name):
        if name == "":
            raise ValueError("name cannot be empty")
```

Composition should reduce complexity.

If it multiplies files and objects without improving clarity, step back.

Design is balance.

---

# Common Mistake: Hidden Construction

Rigid:

```python
class UserService:
    def __init__(self):
        self._repository = DatabaseUserRepository()
```

This hides the dependency.

It makes testing harder.

It forces every `UserService` to use the database repository.

More flexible:

```python
class UserService:
    def __init__(self, repository):
        self._repository = repository
```

Now the dependency is visible.

Production can pass:

```python
DatabaseUserRepository()
```

Tests can pass:

```python
InMemoryUserRepository()
```

Constructor arguments reveal what an object needs.

That is a good thing.

---

# Common Mistake: Leaking Internal Collaborators

Suppose:

```python
class Car:
    def __init__(self, engine):
        self.engine = engine
```

Now callers may do:

```python
car.engine.start()
car.engine.replace_part(...)
car.engine.internal_state = ...
```

Maybe that is intended.

Maybe it leaks internals.

If callers should only start the car, expose:

```python
class Car:
    def __init__(self, engine):
        self._engine = engine

    def start(self):
        self._engine.start()
```

This keeps the engine internal.

Expose collaborators directly only when that is part of the public API.

Otherwise, delegate through clear methods.

---

# Common Mistake: Unclear Cleanup Responsibility

Example:

```python
file = open("report.txt", "w")
writer = ReportWriter(file)
```

Who closes `file`?

The caller?

The writer?

Both?

Neither?

This ambiguity causes bugs.

Make ownership explicit.

Possible design:

```python
with open("report.txt", "w") as file:
    writer = ReportWriter(file)
    writer.write("hello")
```

Here the caller owns the file lifetime.

Another design:

```python
with ReportWriter("report.txt") as writer:
    writer.write("hello")
```

Here `ReportWriter` owns the file lifetime.

Both can be valid.

The danger is not choosing.

---

# Common Mistake: Depending on Concrete Classes Too Early

Rigid:

```python
class ReportService:
    def __init__(self, formatter: HtmlFormatter):
        self._formatter = formatter
```

If the service only needs:

```python
formatter.format(data)
```

then the exact class may not matter.

At runtime, Python only needs an object with the right behavior.

This allows:

```python
ReportService(TextFormatter())
ReportService(HtmlFormatter())
ReportService(FakeFormatter())
```

Static typing can express this later with protocols.

At the design level, think:

```text
What behavior do I need from this collaborator?
```

not always:

```text
What exact class must it be?
```

---

# Design Guidance

When using composition, ask:

```text
What responsibility does this object have?
What collaborators does it need?
Does it create them or receive them?
Who owns each collaborator?
Can collaborators be shared?
Who cleans up resources?
Should a collaborator be public or internal?
Should this object delegate or expose the collaborator directly?
Would a function or module be simpler?
Would inheritance claim a false is-a relationship?
Will this design be easy to test?
```

General guidance:

* Use composition for has-a relationships.
* Prefer composition when behavior needs to vary.
* Inject dependencies when flexibility or testing matters.
* Keep collaborators focused.
* Do not expose internal collaborators unnecessarily.
* Make ownership and cleanup clear.
* Do not split objects so finely that the design becomes noisy.
* Use inheritance only when the relationship truly is-a.

Composition is not just a pattern.

It is how object-oriented programs are assembled.

---

# Exercises

1. Create an `Engine` class and a `Car` class.

Make `Car` use an `Engine` through composition.

Explain why `Car` should not inherit from `Engine`.

---

2. Write a `NotesApp` class that receives a `storage` object.

Create two storage classes:

```text
FileStorage
MemoryStorage
```

Both should support:

```python
save(text)
```

Show that `NotesApp` works with either one.

---

3. Write a fake storage object for testing.

Use it to test that `NotesApp.save_note()` saves the expected text.

Explain why composition makes this easy.

---

4. Create a `Report` class that receives a formatter.

Create:

```text
TextFormatter
HtmlFormatter
```

Render the same report with both formatters.

---

5. Explain the difference between:

```text
has-a
```

and:

```text
is-a
```

Give three examples of each.

---

6. Create an `Order` class composed of `LineItem` objects.

Each line item should know its own total.

The order should compute the full total by delegating to line items.

---

7. Consider this design:

```python
class UserService(Database):
    ...
```

Why might this be a bad inheritance relationship?

Rewrite it using composition.

---

8. Draw an object graph for:

```text
App -> UserService -> UserRepository -> Database
App -> EmailService -> EmailSender
```

Which objects might be shared?

Who might own cleanup?

---

9. Explain why exposing:

```python
car.engine
```

might be a bad idea.

When might it be acceptable?

---

10. In your own words, explain:

```text
composition connects object boundaries
```

Use encapsulation, delegation, and ownership in your answer.

---

# Summary

In this chapter we learned:

* Composition means building objects from other objects.
* Composition models has-a relationships.
* Inheritance models is-a relationships.
* Objects can collaborate by delegating work to contained objects.
* Encapsulation and composition work together.
* Direct composition means an object creates its own collaborator.
* Dependency injection means collaborators are provided from outside.
* Dependency injection improves flexibility and testing.
* Composition creates object graphs.
* Object graphs affect ownership, lifetime, sharing, and cleanup.
* Shared collaborators should be intentional.
* Composition often avoids unnecessary inheritance.
* Composition can reuse behavior without claiming a false type relationship.
* Fake collaborators make testing easier.
* Composition often depends on behavior rather than exact class.
* Resource-owning collaborators require clear cleanup responsibility.
* Composition should reduce complexity, not create ceremony.

Core model:

```text
composition:
    object A has object B
    object A uses object B
    object A delegates work to object B
```

Design model:

```text
has-a relationship -> composition
is-a relationship  -> inheritance
variable behavior  -> composition with swappable collaborator
external effect    -> injected collaborator for testability
resource ownership -> explicit cleanup plan
```

Composition gives object-oriented programs shape.

It lets us build larger systems from focused objects that cooperate through clear interfaces.

---

# Preview of Chapter 47

Next we study inheritance and method overriding.

Composition showed how objects can use other objects.

Inheritance shows how classes can specialize other classes.

Chapter 47 will study:

* What inheritance means.
* How child classes inherit attributes and methods.
* What method overriding is.
* How inherited behavior is reused.
* How subclass behavior can specialize parent behavior.
* Why inheritance should model is-a relationships.
* How inheritance differs from composition.
* Common inheritance mistakes.

The transition is intentional:

```text
composition handles has-a relationships
inheritance handles is-a relationships
```

Now that composition is clear, we can study inheritance with better judgment.
