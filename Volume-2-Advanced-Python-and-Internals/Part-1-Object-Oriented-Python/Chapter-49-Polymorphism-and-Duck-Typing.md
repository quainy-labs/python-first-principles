# Chapter 49 — Polymorphism and Duck Typing

---

# Learning Objectives

By the end of this chapter, you should understand:

* What polymorphism means.
* How inheritance can support polymorphism.
* What duck typing means.
* Why Python often cares more about behavior than exact class.
* How objects can share an interface without sharing a parent class.
* How composition and duck typing work together.
* What informal protocols are.
* Why `isinstance()` can be useful but also limiting.
* How EAFP and LBYL relate to duck typing.
* How duck typing appears in built-in Python tools.
* How polymorphism prepares us for ABCs, mixins, protocols, and static typing.

Chapter 47 taught inheritance.

Chapter 48 taught MRO.

Those chapters showed how classes can share and specialize behavior through class relationships.

Now we study a more Pythonic idea:

```text
Sometimes the exact class matters less than the behavior an object provides.
```

If an object has the method your code needs, your code may not care whether the object inherits from a specific base class.

This is duck typing.

It is one of the most important ideas in Python design.

---

# Concept Overview

Polymorphism means many forms.

In programming, it means the same operation can work with different kinds of objects.

Example:

```python
class Dog:
    def speak(self):
        return "woof"


class Cat:
    def speak(self):
        return "meow"
```

Use:

```python
animals = [Dog(), Cat()]

for animal in animals:
    print(animal.speak())
```

Output:

```text
woof
meow
```

The code calls:

```python
animal.speak()
```

for every object.

Each object responds in its own way.

That is polymorphism.

The caller uses a common behavior.

The objects provide different implementations.

---

# Polymorphism Through Inheritance

Inheritance can create polymorphism.

Example:

```python
class Animal:
    def speak(self):
        raise NotImplementedError


class Dog(Animal):
    def speak(self):
        return "woof"


class Cat(Animal):
    def speak(self):
        return "meow"
```

Use:

```python
animals: list[Animal] = [Dog(), Cat()]

for animal in animals:
    print(animal.speak())
```

Output:

```text
woof
meow
```

Here, `Dog` and `Cat` both inherit from `Animal`.

They share a conceptual parent.

The parent defines the expected behavior:

```python
speak()
```

The children implement it differently.

This is inheritance-based polymorphism.

It is useful when there is a real family of related types.

---

# Polymorphism Without Inheritance

Python does not require a shared parent class for polymorphism.

Example:

```python
class Dog:
    def speak(self):
        return "woof"


class Robot:
    def speak(self):
        return "beep"
```

Use:

```python
speakers = [Dog(), Robot()]

for speaker in speakers:
    print(speaker.speak())
```

Output:

```text
woof
beep
```

`Dog` and `Robot` do not inherit from the same custom base class.

But both provide:

```python
speak()
```

The loop only needs that behavior.

This is where Python's style becomes clear:

```text
If the object can do what the code needs, the exact class may not matter.
```

That is duck typing.

---

# Duck Typing

Duck typing comes from the phrase:

```text
If it walks like a duck and quacks like a duck, treat it like a duck.
```

In Python terms:

```text
If an object has the behavior needed here, use it.
```

Example:

```python
def make_it_speak(obj):
    print(obj.speak())
```

This function does not ask:

```python
isinstance(obj, Animal)
```

It simply calls:

```python
obj.speak()
```

Any object with a compatible `speak` method works.

Example:

```python
make_it_speak(Dog())
make_it_speak(Robot())
```

Duck typing focuses on behavior.

It says:

```text
The method matters more than the declared family.
```

This is a core Python idea.

---

# Behavior Over Exact Class

Suppose we write:

```python
def save_text(destination, text):
    destination.write(text)
```

This works with many objects:

```python
class ConsoleWriter:
    def write(self, text):
        print(text)


class ListWriter:
    def __init__(self):
        self.items = []

    def write(self, text):
        self.items.append(text)
```

Use:

```python
save_text(ConsoleWriter(), "hello")

writer = ListWriter()
save_text(writer, "hello")
print(writer.items)
```

The function does not care whether the destination is a file, console writer, memory writer, or network writer.

It only cares that the object supports:

```python
write(text)
```

That expected behavior is the informal interface.

Duck typing lets code stay flexible.

---

# Informal Protocols

An informal protocol is a set of behavior an object is expected to provide.

Example:

```text
writable object:
    has write(text)
```

Function:

```python
def save_text(destination, text):
    destination.write(text)
```

This function expects the writable protocol.

It does not require a base class named `Writable`.

It does not require registration.

It simply uses the expected behavior.

Other informal protocols:

```text
iterable:
    can be used in a for loop

context manager:
    has __enter__ and __exit__

sequence-like:
    supports len() and indexing

file-like:
    supports read() or write()
```

Python has many behavior-based expectations.

Some are formalized by special methods.

Some are simply conventions.

---

# Built-In Duck Typing

Python's built-ins often use duck typing.

Example:

```python
for item in obj:
    ...
```

The `for` loop does not require `obj` to inherit from a class named `Iterable`.

It needs `obj` to be iterable.

Example:

```python
len(obj)
```

Python does not ask:

```text
is obj a List?
```

It asks whether the object supports the length protocol through `__len__`.

Example:

```python
with obj:
    ...
```

Python expects context manager behavior:

```text
__enter__
__exit__
```

Python syntax itself is deeply protocol-oriented.

That is why duck typing is not a side feature.

It is woven into the language.

---

# File-Like Objects

A classic duck typing example is file-like objects.

Suppose:

```python
def count_lines(file_obj):
    count = 0

    for line in file_obj:
        count += 1

    return count
```

This can work with a real file:

```python
with open("notes.txt") as file:
    print(count_lines(file))
```

It can also work with another object that behaves like an iterable of lines:

```python
class MemoryLines:
    def __init__(self, lines):
        self._lines = lines

    def __iter__(self):
        return iter(self._lines)
```

Use:

```python
lines = MemoryLines(["one\n", "two\n", "three\n"])
print(count_lines(lines))
```

Output:

```text
3
```

The function cares about behavior:

```text
can be iterated line by line
```

not exact type.

---

# Formatter Example

Composition and duck typing work well together.

Example:

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
    def __init__(self, formatter):
        self._formatter = formatter

    def render(self, rows):
        return self._formatter.format(rows)
```

Use:

```python
rows = ["one", "two"]

print(Report(TextFormatter()).render(rows))
print(Report(HtmlFormatter()).render(rows))
```

`Report` expects:

```text
formatter has format(rows)
```

It does not require a shared formatter base class.

The collaborator only needs the right behavior.

That is duck typing supporting composition.

---

# Repository Example

Duck typing is useful for storage abstractions.

Example:

```python
class InMemoryRepository:
    def __init__(self):
        self._items = {}

    def save(self, key, value):
        self._items[key] = value

    def get(self, key):
        return self._items.get(key)
```

Service:

```python
class UserService:
    def __init__(self, repository):
        self._repository = repository

    def register(self, user_id, name):
        user = {"id": user_id, "name": name}
        self._repository.save(user_id, user)
        return user

    def find(self, user_id):
        return self._repository.get(user_id)
```

The service expects:

```text
repository.save(key, value)
repository.get(key)
```

A database repository, fake repository, or memory repository can work if it provides those methods.

Inheritance is not required.

Behavior is enough.

---

# Fake Objects in Tests

Duck typing makes test doubles easy.

Example:

```python
class FakeSender:
    def __init__(self):
        self.messages = []

    def send(self, message):
        self.messages.append(message)
```

Production service:

```python
class NotificationService:
    def __init__(self, sender):
        self._sender = sender

    def welcome(self, name):
        self._sender.send(f"Welcome {name}")
```

Test:

```python
sender = FakeSender()
service = NotificationService(sender)

service.welcome("Ada")

assert sender.messages == ["Welcome Ada"]
```

`FakeSender` does not inherit from `EmailSender`.

It provides the needed behavior:

```python
send(message)
```

That is enough.

Duck typing supports simple, focused tests.

---

# Inheritance-Based Interface vs Duck-Typed Interface

Inheritance-based:

```python
class Sender:
    def send(self, message):
        raise NotImplementedError


class EmailSender(Sender):
    def send(self, message):
        print("email:", message)
```

Function:

```python
def notify(sender: Sender, message):
    sender.send(message)
```

Duck-typed:

```python
class EmailSender:
    def send(self, message):
        print("email:", message)


class SmsSender:
    def send(self, message):
        print("sms:", message)
```

Function:

```python
def notify(sender, message):
    sender.send(message)
```

Both can be valid.

Inheritance is useful when you want a formal family.

Duck typing is useful when behavior is enough.

Python lets you choose.

---

# When Inheritance Helps

Duck typing does not mean inheritance is useless.

Inheritance helps when:

* There is a real is-a relationship.
* Shared behavior belongs in a parent class.
* You need a common implementation.
* You want `isinstance()` checks to mean something.
* You want formal abstract base classes.
* You are working with a framework that expects subclasses.

Example:

```python
class Shape:
    def area(self):
        raise NotImplementedError


class Rectangle(Shape):
    def area(self):
        return self.width * self.height
```

This is reasonable if your design has a family of shapes.

Duck typing simply reminds us:

```text
inheritance is not required for polymorphism
```

It is one tool among several.

---

# When Duck Typing Helps

Duck typing helps when:

* You only need a small behavior.
* You want to avoid unnecessary base classes.
* You want to accept existing objects.
* You want test fakes.
* You want flexible composition.
* You care about capability more than ancestry.

Example:

```python
def print_all(items):
    for item in items:
        print(item)
```

This works with:

* Lists.
* Tuples.
* Sets.
* Generators.
* Files.
* Custom iterable objects.

The function does not need:

```python
isinstance(items, list)
```

It only needs:

```text
items is iterable
```

That flexibility is Pythonic.

---

# `isinstance()` Can Help

Duck typing does not mean:

```text
never use isinstance()
```

`isinstance()` can be useful when:

* You truly need different behavior for different types.
* You are validating public API input.
* You are distinguishing strings from other iterables.
* You are working with numeric types.
* You are protecting against ambiguous input.
* You are debugging.

Example:

```python
def repeat(value, times):
    if not isinstance(times, int):
        raise TypeError("times must be an integer")

    return value * times
```

This check may be reasonable.

The problem is not `isinstance()` itself.

The problem is using it where behavior would be more flexible.

---

# `isinstance()` Can Be Too Rigid

Rigid:

```python
def total(values):
    if not isinstance(values, list):
        raise TypeError("values must be a list")

    return sum(values)
```

This rejects:

```python
total((1, 2, 3))
total({1, 2, 3})
total(x for x in [1, 2, 3])
```

But `sum()` can handle many iterables.

Better:

```python
def total(values):
    return sum(values)
```

Now the function accepts any object `sum` can iterate over.

This is more flexible.

Ask:

```text
Do I need a list specifically?
Or do I need something iterable?
```

Duck typing often makes APIs more useful by accepting the behavior, not one concrete class.

---

# EAFP

EAFP means:

```text
Easier to Ask Forgiveness than Permission
```

Python code often tries the operation and handles failure.

Example:

```python
def save(destination, text):
    try:
        destination.write(text)
    except AttributeError:
        raise TypeError("destination must support write(text)")
```

This style says:

```text
try to use the object as needed
handle the error if it cannot
```

EAFP fits duck typing.

Instead of checking:

```python
hasattr(destination, "write")
```

then calling it, we try:

```python
destination.write(text)
```

and handle failure.

EAFP is common in Python, especially when race conditions or dynamic behavior make pre-checks unreliable.

---

# LBYL

LBYL means:

```text
Look Before You Leap
```

Example:

```python
def save(destination, text):
    if not hasattr(destination, "write"):
        raise TypeError("destination must support write(text)")

    destination.write(text)
```

This checks before acting.

LBYL can be readable for simple validation.

But it can be less reliable in some situations.

For example, between:

```python
hasattr(destination, "write")
```

and:

```python
destination.write(text)
```

the object could change in a concurrent or dynamic system.

Also, `hasattr()` only checks that lookup succeeds.

It does not guarantee the attribute is callable or accepts the right arguments.

Both EAFP and LBYL are tools.

Duck typing often pairs naturally with EAFP.

---

# Errors Are Part of Duck Typing

If an object does not provide needed behavior, Python raises an error.

Example:

```python
def notify(sender, message):
    sender.send(message)
```

Call:

```python
notify(object(), "hello")
```

Error:

```text
AttributeError
```

This means:

```text
object does not support the expected protocol
```

You can let the error happen.

Or you can catch it and raise a clearer error:

```python
def notify(sender, message):
    try:
        sender.send(message)
    except AttributeError as exc:
        raise TypeError("sender must provide send(message)") from exc
```

Design choice:

```text
internal helper -> raw AttributeError may be fine
public API -> clearer TypeError may be better
```

Good errors are part of good interfaces.

---

# Asking for Capability

Duck typing asks:

```text
Can this object do what I need?
```

not:

```text
Is this object exactly the class I expected?
```

Example:

```python
def render(formatter, rows):
    return formatter.format(rows)
```

Capability:

```text
has format(rows)
```

Class:

```text
TextFormatter
HtmlFormatter
JsonFormatter
```

If the function checks:

```python
isinstance(formatter, TextFormatter)
```

it rejects useful alternatives.

Better:

```python
formatter.format(rows)
```

This is capability-oriented design.

It supports extension.

New formatter classes can work without changing `render`.

---

# Polymorphism and Composition

Composition becomes more flexible with polymorphism.

Example:

```python
class Report:
    def __init__(self, formatter):
        self._formatter = formatter

    def render(self, rows):
        return self._formatter.format(rows)
```

Any formatter object can work.

The report is composed with a collaborator.

Polymorphism lets that collaborator vary.

This combination is powerful:

```text
composition:
    object has collaborator

duck typing:
    collaborator only needs expected behavior
```

Result:

```text
flexible design without unnecessary inheritance
```

This is one reason Python object-oriented design often prefers small collaborators over large class hierarchies.

---

# Polymorphism and Standard Library Style

Many Python standard library functions accept file-like objects or iterable objects.

Example idea:

```python
def process_lines(lines):
    for line in lines:
        ...
```

This can work with:

* A file object.
* A list of strings.
* A tuple of strings.
* A generator.
* A custom object with `__iter__`.

That style is intentional.

It makes functions broadly useful.

Instead of demanding one concrete type, Python APIs often accept behavior.

When writing your own functions, copy this style when appropriate.

Ask:

```text
What behavior do I need?
```

Then design for that behavior.

---

# String Is Also Iterable

Duck typing can surprise you.

Example:

```python
def print_items(items):
    for item in items:
        print(item)
```

Call:

```python
print_items("abc")
```

Output:

```text
a
b
c
```

This happens because strings are iterable.

If your function wants a collection of items but should reject a single string, you may need a type check.

Example:

```python
def print_items(items):
    if isinstance(items, str):
        raise TypeError("items must be an iterable of items, not a string")

    for item in items:
        print(item)
```

This is a good example of balanced design.

Duck typing is powerful.

But sometimes a concrete type check prevents ambiguity.

---

# Mapping-Like Objects

Some functions need mapping behavior.

Example:

```python
def display_user(user):
    return f"{user['name']} <{user['email']}>"
```

This function does not require `user` to be a `dict`.

It requires:

```text
supports key access for 'name' and 'email'
```

This can work with:

* A dictionary.
* A custom mapping.
* A wrapper object implementing `__getitem__`.

If you write:

```python
if not isinstance(user, dict):
    raise TypeError
```

you reject mapping-like objects.

Ask:

```text
Do I need an actual dict?
Or do I need mapping behavior?
```

This question leads to better APIs.

---

# Sequence-Like Objects

Suppose:

```python
def first(items):
    return items[0]
```

This needs:

```text
indexing with 0
```

It works with:

```python
first([1, 2, 3])
first((1, 2, 3))
first("abc")
```

It may also work with custom sequence-like objects.

But it will not work with a generator:

```python
first(x for x in [1, 2, 3])
```

because generators are iterable but not indexable.

Duck typing requires precision.

Do not say:

```text
any collection
```

when you mean:

```text
indexable sequence
```

Behavior-based design still needs clear expectations.

---

# Iterable vs Iterator Preview

An iterable is something you can loop over.

An iterator is the object that produces values one at a time.

Example:

```python
values = [1, 2, 3]
iterator = iter(values)
```

The list is iterable.

The iterator is produced by `iter(values)`.

Functions often need only an iterable:

```python
def total(values):
    amount = 0

    for value in values:
        amount += value

    return amount
```

This works with many object types.

We will study iterators and generators in Part III.

For now, notice:

```text
iterability is a behavior
```

Duck typing is everywhere.

---

# Runtime Protocols vs Static Protocols

At runtime, duck typing can be informal:

```python
obj.write(text)
```

If it works, it works.

Static typing later gives us a way to describe behavior expectations formally.

Example idea:

```python
class Writable(Protocol):
    def write(self, text: str) -> None:
        ...
```

Then a type checker can understand:

```text
any object with write(str) is acceptable
```

We will study this later in the type system chapters.

For now, remember:

```text
duck typing is the runtime idea
protocols are a way to describe that idea more formally
```

This is why duck typing comes before static typing in the book.

---

# ABCs Preview

ABCs are abstract base classes.

They can define formal interfaces and support `isinstance()` checks.

Example idea:

```python
from collections.abc import Iterable

isinstance(obj, Iterable)
```

ABCs can be useful when:

* You want a formal interface.
* You want runtime checking.
* You want shared behavior.
* You want to document subclass responsibilities.

But ABCs are not always necessary.

If a function simply calls:

```python
obj.save(value)
```

duck typing may be enough.

Chapter 50 will study ABCs and mixins.

This chapter gives the reason they exist:

```text
sometimes informal behavior expectations need more structure
```

---

# Polymorphism and Special Methods

Special methods make Python syntax polymorphic.

Example:

```python
len(obj)
```

calls:

```python
obj.__len__()
```

conceptually.

Example:

```python
obj + other
```

uses special methods such as:

```python
__add__
```

Example:

```python
for item in obj:
```

uses iteration protocol methods.

This means Python syntax works with any object that implements the expected protocol.

That is deep polymorphism.

Part II will study dunder methods and operator overloading.

For now, see the connection:

```text
duck typing is not only about ordinary methods
it also powers Python syntax
```

---

# A Complete Example: Exporters

Exporters:

```python
class CsvExporter:
    def export(self, rows):
        return "\n".join(",".join(row) for row in rows)


class JsonExporter:
    def export(self, rows):
        return str(rows)
```

Function:

```python
def export_report(exporter, rows):
    return exporter.export(rows)
```

Use:

```python
rows = [["name", "email"], ["Ada", "ada@example.com"]]

print(export_report(CsvExporter(), rows))
print(export_report(JsonExporter(), rows))
```

The function expects:

```text
exporter has export(rows)
```

It does not require:

```text
exporter is subclass of Exporter
```

If later you add:

```python
class XmlExporter:
    def export(self, rows):
        return "<rows>...</rows>"
```

it works without changing `export_report`.

That is open-ended polymorphism.

---

# A Complete Example: Payment Processors

Processors:

```python
class CardProcessor:
    def pay(self, amount):
        return f"card payment: {amount}"


class WalletProcessor:
    def pay(self, amount):
        return f"wallet payment: {amount}"
```

Checkout:

```python
class Checkout:
    def __init__(self, processor):
        self._processor = processor

    def complete(self, amount):
        if amount <= 0:
            raise ValueError("amount must be positive")

        return self._processor.pay(amount)
```

Use:

```python
checkout = Checkout(CardProcessor())
print(checkout.complete(100))

checkout = Checkout(WalletProcessor())
print(checkout.complete(100))
```

`Checkout` does not need inheritance from processors.

It needs a collaborator with:

```python
pay(amount)
```

Composition plus duck typing keeps checkout logic independent from payment method details.

---

# A Complete Example: Sorting Custom Objects

Python's sorting can use a key function.

Example:

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

Use:

```python
users = [
    User("Ada", 36),
    User("Grace", 30),
]

users.sort(key=lambda user: user.age)
```

The sort operation does not require `User` to inherit from a sortable base class.

It needs a way to compare key values.

Alternative:

```python
users.sort(key=lambda user: user.name)
```

Behavior is supplied through a callable.

This is another form of polymorphism:

```text
the sort algorithm works with many object types because key extraction is configurable
```

Python often solves flexibility with functions and protocols, not only class hierarchies.

---

# Common Mistake: Checking Too Specific a Type

Bad:

```python
def average(values):
    if not isinstance(values, list):
        raise TypeError("values must be a list")

    return sum(values) / len(values)
```

This rejects tuples and other sequences.

If the function only needs:

```text
sum support and len support
```

then write:

```python
def average(values):
    return sum(values) / len(values)
```

Now it accepts:

```python
average([1, 2, 3])
average((1, 2, 3))
```

Maybe it should reject generators because `len()` does not work.

That is fine.

The requirement is:

```text
iterable and sized
```

not:

```text
list
```

Name the behavior precisely.

---

# Common Mistake: Duck Typing Without Clear Errors

Too vague:

```python
def save(destination, text):
    destination.write(text)
```

If wrong input is common, callers may see:

```text
AttributeError: 'NoneType' object has no attribute 'write'
```

For an internal helper, that may be fine.

For a public API, improve the error:

```python
def save(destination, text):
    try:
        write = destination.write
    except AttributeError as exc:
        raise TypeError("destination must provide write(text)") from exc

    write(text)
```

This still uses duck typing.

It simply gives a clearer failure message.

Duck typing does not mean careless errors.

It means behavior-based expectations.

Good APIs communicate those expectations.

---

# Common Mistake: Assuming Same Method Name Means Same Meaning

Two objects can have the same method name but incompatible meaning.

Example:

```python
class FileWriter:
    def write(self, text):
        ...


class DatabaseTransaction:
    def write(self, record):
        ...
```

Both have `write`.

But one expects text.

The other expects a record.

Duck typing requires compatible behavior, not only matching names.

Ask:

```text
Does this method mean the same kind of operation?
Does it accept compatible arguments?
Does it return compatible results?
Does it raise compatible errors?
```

An informal protocol is more than a method name.

It is a behavioral contract.

---

# Common Mistake: Over-Abstracting Too Early

Do not create an abstract interface for every tiny variation.

Maybe this is enough:

```python
def send_email(message):
    ...
```

You do not always need:

```python
class AbstractMessageSender:
    ...

class EmailMessageSender(AbstractMessageSender):
    ...
```

Start simple.

Introduce abstractions when:

* You have multiple implementations.
* Tests need a fake.
* The dependency is external or expensive.
* The interface is stable enough to name.
* The design becomes clearer with a boundary.

Duck typing lets you delay formal abstraction.

You can depend on behavior first and formalize later when the pattern is real.

---

# Common Mistake: Ignoring Existing Protocols

Python already has many established protocols.

Examples:

```text
iteration
context management
callable objects
sequence behavior
mapping behavior
numeric operations
comparison
```

Before inventing a custom method name, consider whether Python already has a protocol.

Example:

Instead of:

```python
obj.next_item()
```

maybe implement iteration:

```python
for item in obj:
    ...
```

Instead of:

```python
obj.close_after_use()
```

maybe implement a context manager:

```python
with obj:
    ...
```

Pythonic design often means fitting into existing protocols.

Part II and Part III will deepen this.

---

# Common Mistake: Thinking Duck Typing Means No Design

Duck typing is not:

```text
anything goes
```

It still requires clear expectations.

Bad:

```python
def process(obj):
    obj.do_the_thing()
```

if nobody knows what `do_the_thing` means.

Better:

```python
def render_report(formatter, rows):
    return formatter.format(rows)
```

The expected behavior is clear:

```text
formatter.format(rows)
```

Duck typing works best when the required behavior is small, explicit, and meaningful.

Informal does not mean vague.

Good duck-typed code still has a contract.

It is just a behavioral contract rather than necessarily an inheritance contract.

---

# Design Guidance

When designing polymorphic code, ask:

```text
What behavior does this code actually need?
Do I need a specific class or only a capability?
Would accepting a broader protocol make the function more useful?
Would a type check prevent valid objects?
Would no type check produce confusing errors?
Should this expectation be informal, an ABC, or a static Protocol later?
Does composition make the varying behavior easier to swap?
Is the method name enough, or does the behavior need clearer documentation?
```

General guidance:

* Prefer behavior-based design when possible.
* Use inheritance when it expresses a real type family.
* Use composition to inject collaborators.
* Use duck typing for flexible collaborator behavior.
* Use `isinstance()` when exact type distinctions truly matter.
* Avoid checking for concrete types when a protocol would be enough.
* Provide clear errors for public APIs.
* Keep informal protocols small and meaningful.

Pythonic design often begins with:

```text
What can this object do?
```

not:

```text
What exact class is this object?
```

---

# Exercises

1. Create two classes:

```text
Dog
Cat
```

Both should define:

```python
speak()
```

Put instances in a list and call `speak` on each.

Explain why this is polymorphism.

---

2. Repeat exercise 1 with a shared `Animal` parent class.

Then remove the parent class.

Does the loop still work?

Why?

---

3. Write a function:

```python
def save(destination, text):
    destination.write(text)
```

Create two destination classes with `write`.

Use both with `save`.

---

4. Create a fake sender for a notification service.

The fake should collect messages in a list.

Explain why it does not need to inherit from the real sender.

---

5. Rewrite this function to be less rigid:

```python
def total(values):
    if not isinstance(values, list):
        raise TypeError
    return sum(values)
```

What behavior does the function actually need?

---

6. Write an example where `isinstance()` is useful.

Then write an example where `isinstance()` makes code unnecessarily rigid.

---

7. Explain EAFP and LBYL using a `write(text)` example.

Which style fits duck typing naturally?

---

8. Create two formatter classes:

```text
TextFormatter
HtmlFormatter
```

Both should implement:

```python
format(rows)
```

Inject each into a `Report` class.

---

9. Explain why matching method names are not enough for duck typing.

What else must be compatible?

---

10. In your own words, explain:

```text
inheritance says what an object is
duck typing cares what an object can do
```

Use at least two examples.

---

# Summary

In this chapter we learned:

* Polymorphism means the same operation can work with different forms of object.
* Inheritance can support polymorphism through shared parent classes.
* Python also supports polymorphism without inheritance.
* Duck typing focuses on behavior rather than exact class.
* An object can be accepted if it provides the methods or protocol the code needs.
* Informal protocols are behavior expectations without formal base classes.
* Built-in Python features such as iteration, `len`, context managers, and operators rely on protocols.
* Composition and duck typing work well together.
* Fake objects in tests often rely on duck typing.
* `isinstance()` is useful when exact type distinctions matter, but it can make code too rigid.
* EAFP tries the operation and handles failure.
* LBYL checks before performing the operation.
* Duck typing still requires clear behavioral contracts.
* ABCs, mixins, static protocols, and dunder methods build on these ideas.

Core model:

```text
inheritance polymorphism:
    shared parent class
    child classes override behavior

duck typing:
    no required shared parent
    object provides expected behavior
```

Design model:

```text
need a family of related types -> inheritance or ABC
need a swappable collaborator -> composition + duck typing
need broad input support -> accept behavior, not concrete class
need clear runtime validation -> use careful checks or clear errors
```

Duck typing is not loose thinking.

It is precise behavior-based thinking.

It is one of the reasons Python code can be both simple and flexible.

---

# Preview of Chapter 50

Next we study ABCs and mixins.

Duck typing showed how informal behavior expectations can be enough.

But sometimes we want more structure.

Chapter 50 explains two tools for that structure:

* Abstract base classes.
* Mixins.

We will study:

* What an ABC is.
* How abstract methods define required behavior.
* How ABCs differ from informal duck typing.
* When runtime `isinstance()` checks are useful.
* What mixins are.
* Why mixins should be small and focused.
* How mixins rely on MRO.
* How ABCs, mixins, composition, and duck typing fit together.

The transition is direct:

```text
duck typing gives informal behavior contracts
ABCs and mixins give more explicit reusable structure
```

This completes the object-oriented design arc before we move into dataclasses.
