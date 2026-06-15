# Chapter 52 — Dunder Methods

---

# Learning Objectives

By the end of this chapter, you should understand:

* What dunder methods are.
* Why Python calls them special methods.
* How dunder methods connect syntax to object behavior.
* Why `len(x)` calls `x.__len__()` through Python's protocol machinery.
* Why `x + y` is not just syntax, but a method dispatch.
* How object construction uses `__new__` and `__init__`.
* How representation uses `__repr__`, `__str__`, and `__format__`.
* How truthiness uses `__bool__` and `__len__`.
* How equality and hashing use `__eq__` and `__hash__`.
* Why returning `NotImplemented` is different from raising `NotImplementedError`.
* How containers use `__len__`, `__getitem__`, `__setitem__`, `__delitem__`, and `__contains__`.
* How iteration uses `__iter__` and `__next__`.
* How callable objects use `__call__`.
* How context managers use `__enter__` and `__exit__`.
* Why special method lookup is unusual.
* How dunder methods help you design Pythonic objects.
* When not to implement a dunder method.

Chapter 51 introduced dataclasses.

Dataclasses can generate methods such as:

```python
__init__
__repr__
__eq__
__hash__
```

Those method names are not accidental.

They are part of Python's data model.

The data model is the set of rules that explains how objects participate in the language.

When you write:

```python
len(items)
```

Python asks the object for length behavior.

When you write:

```python
a + b
```

Python asks the operands for addition behavior.

When you write:

```python
for item in collection:
    ...
```

Python asks the object for iteration behavior.

When you write:

```python
with resource:
    ...
```

Python asks the object for context management behavior.

Dunder methods are the hooks behind those behaviors.

The word "dunder" means:

```text
double underscore
```

So `__len__` is pronounced:

```text
dunder len
```

Python's official term is special method.

Programmers commonly say dunder method.

This chapter uses both.

---

# Why This Chapter Matters

Many languages separate objects from language syntax.

Python does not.

Python lets objects define how they behave with the language's own operations.

That is why this works:

```python
len("hello")
```

and this works:

```python
len([1, 2, 3])
```

and this can work for your own class:

```python
len(my_collection)
```

if your class implements `__len__`.

This is one reason Python feels consistent.

Built-in types and user-defined types can participate in the same protocols.

The goal is not to memorize every dunder method.

The goal is to see the pattern:

```text
Python syntax or built-in function -> special method protocol -> object behavior
```

Once you understand that pattern, Python becomes much less mysterious.

---

# A First Example: `__len__`

Suppose we build a simple bookshelf:

```python
class Bookshelf:
    def __init__(self):
        self.books = []
```

We can add books:

```python
class Bookshelf:
    def __init__(self):
        self.books = []

    def add(self, title):
        self.books.append(title)
```

Now:

```python
shelf = Bookshelf()
shelf.add("Python")
shelf.add("Algorithms")
```

How should we ask how many books are on the shelf?

We could write:

```python
shelf.count()
```

But Python already has a standard length operation:

```python
len(shelf)
```

To support it, implement `__len__`:

```python
class Bookshelf:
    def __init__(self):
        self.books = []

    def add(self, title):
        self.books.append(title)

    def __len__(self):
        return len(self.books)
```

Now:

```python
print(len(shelf))
```

works.

Python calls the length protocol.

Your object participates in that protocol by defining `__len__`.

This is the central idea of dunder methods.

They are not meant to be called by users most of the time.

They are meant to be called by Python.

---

# Dunder Methods Are Protocol Hooks

A protocol is an expected set of behavior.

The length protocol says:

```text
if an object has length, it should provide __len__
```

The iteration protocol says:

```text
if an object can be iterated, it should provide __iter__
```

The context manager protocol says:

```text
if an object can be used with with, it should provide __enter__ and __exit__
```

Dunder methods let objects join protocols.

Example:

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        current = self.start
        while current > 0:
            yield current
            current -= 1
```

Now:

```python
for number in Countdown(3):
    print(number)
```

Output:

```text
3
2
1
```

The `for` loop did not need to know about `Countdown`.

It only needed the iteration protocol.

This is duck typing at the language level.

If an object implements the right protocol, Python can use it.

---

# Special Names Are Reserved by Convention

Dunder names are special.

Names like:

```python
__init__
__repr__
__len__
__iter__
```

are reserved for Python's data model.

You should not invent your own random dunder names.

This is bad design:

```python
class Report:
    def __export__(self):
        ...
```

Python has no built-in protocol for `__export__`.

A normal method is better:

```python
class Report:
    def export(self):
        ...
```

Use dunder names only when you are implementing a Python-recognized protocol.

This keeps your code readable.

It also avoids future conflicts if Python later defines a special name.

---

# Calling Dunder Methods Directly

You can call many dunder methods directly:

```python
items = [1, 2, 3]
print(items.__len__())
```

Output:

```python
3
```

But the Pythonic call is:

```python
len(items)
```

The built-in operation communicates intent.

It also lets Python handle special lookup rules and fallbacks.

Prefer:

```python
len(x)
str(x)
iter(x)
next(iterator)
hash(x)
bool(x)
```

over direct calls like:

```python
x.__len__()
x.__str__()
x.__iter__()
iterator.__next__()
x.__hash__()
x.__bool__()
```

Direct dunder calls are useful when teaching, debugging, or implementing related protocols.

In normal application code, use the language operation.

---

# Object Creation: `__new__` and `__init__`

Object creation has two major steps:

```text
allocate/create the object
initialize the object
```

Python separates these steps:

```python
__new__
__init__
```

`__new__` creates and returns a new instance.

`__init__` initializes that instance.

Most classes only need `__init__`.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name
```

When you write:

```python
user = User("Maya")
```

Python roughly does:

```text
instance = User.__new__(User)
User.__init__(instance, "Maya")
user = instance
```

This is simplified, but the mental model is useful.

`__new__` is especially important for:

* immutable types
* singleton-like behavior
* subclassing built-in immutable types
* metaclass-level object creation patterns

Most day-to-day classes should not override `__new__`.

Use `__init__` unless you truly need to control object creation before initialization.

---

# `__init__` Does Not Return the Object

`__init__` must return `None`.

This is wrong:

```python
class User:
    def __init__(self, name):
        self.name = name
        return self
```

Python creates the object before `__init__` finishes.

The job of `__init__` is to initialize that object, not return it.

This is correct:

```python
class User:
    def __init__(self, name):
        self.name = name
```

If you need a custom construction path, use a class method:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    @classmethod
    def from_username(cls, username):
        return cls(username, f"{username}@example.com")
```

This keeps object initialization clear.

---

# `__new__` with an Immutable Type

Here is a small example using `__new__`.

Suppose we want a string subclass that always stores lowercase text:

```python
class LowercaseString(str):
    def __new__(cls, value):
        return super().__new__(cls, value.lower())
```

Now:

```python
name = LowercaseString("MAYA")
print(name)
```

Output:

```text
maya
```

Why use `__new__`?

Because strings are immutable.

By the time `__init__` runs, the string value already exists.

To control the value of an immutable object, you usually need `__new__`.

This is advanced.

Most classes in this book use `__init__`.

But knowing `__new__` exists makes class creation less mysterious.

---

# Object Destruction: `__del__`

Python has a finalizer method:

```python
__del__
```

It can run when an object is about to be destroyed.

Example:

```python
class Resource:
    def __del__(self):
        print("cleaning up")
```

You should be careful with `__del__`.

It is not a general-purpose cleanup mechanism.

Garbage collection timing can vary.

Reference cycles can complicate finalization.

Interpreter shutdown can make global variables unavailable.

Exceptions in finalizers are awkward.

For external resources, prefer explicit cleanup:

```python
resource.close()
```

or context managers:

```python
with open("data.txt") as file:
    data = file.read()
```

Later chapters cover context managers in depth.

For now, remember:

```text
__del__ is not a substitute for clear resource management
```

---

# Representation: `__repr__`

`__repr__` defines the official string representation of an object.

It is used by:

* `repr(obj)`
* interactive sessions
* containers when displaying contained objects
* debugging output
* many assertion messages

Example:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point(x={self.x!r}, y={self.y!r})"
```

Now:

```python
point = Point(10, 20)
print(repr(point))
```

Output:

```python
Point(x=10, y=20)
```

A good `__repr__` is:

* unambiguous
* useful for debugging
* honest about the object's state
* safe to display

For simple value objects, a common ideal is that `repr(obj)` looks like code that could recreate the object.

Example:

```python
Point(x=10, y=20)
```

That is not always possible.

For a database connection, a recreation-style representation may be impossible or unsafe.

Still, `__repr__` should help a developer understand what the object is.

---

# Representation: `__str__`

`__str__` defines the informal, human-facing string representation.

It is used by:

* `str(obj)`
* `print(obj)`
* f-strings without conversion flags

Example:

```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency

    def __repr__(self):
        return f"Money(amount={self.amount!r}, currency={self.currency!r})"

    def __str__(self):
        return f"{self.amount} {self.currency}"
```

Now:

```python
price = Money(100, "INR")

print(repr(price))
print(str(price))
print(price)
```

Output:

```python
Money(amount=100, currency='INR')
100 INR
100 INR
```

Use `__repr__` for developers.

Use `__str__` for users.

If `__str__` is not defined, Python falls back to `__repr__`.

That is why defining only `__repr__` is often enough for small internal objects.

---

# Formatting: `__format__`

The `__format__` method supports the `format()` function and f-string format specifications.

Example:

```python
value = 12.3456
print(f"{value:.2f}")
```

The `.2f` format specification is handled by the float formatting protocol.

You can define formatting for your own type:

```python
class Money:
    def __init__(self, cents, currency):
        self.cents = cents
        self.currency = currency

    def __format__(self, spec):
        amount = self.cents / 100
        if spec == "symbol":
            return f"{amount:.2f} {self.currency}"
        return f"{amount:{spec}} {self.currency}"
```

Usage:

```python
price = Money(12345, "INR")
print(f"{price:.2f}")
```

Output:

```text
123.45 INR
```

Most classes do not need custom `__format__`.

Implement it when your object has meaningful formatting variations.

For many classes, `__str__` is enough.

---

# Truthiness: `__bool__`

Python asks for truthiness in contexts such as:

```python
if obj:
    ...
```

and:

```python
while obj:
    ...
```

To define truthiness directly, implement `__bool__`:

```python
class Cart:
    def __init__(self):
        self.items = []

    def __bool__(self):
        return bool(self.items)
```

Now an empty cart is falsey:

```python
cart = Cart()

if cart:
    print("has items")
else:
    print("empty")
```

Output:

```text
empty
```

`__bool__` must return a Boolean value.

Use it when truthiness has a clear meaning.

Examples:

```text
empty collection -> False
closed connection -> False perhaps
successful result -> True perhaps
```

Be careful.

Truthiness should not surprise readers.

If the condition is domain-specific, a named method may be clearer:

```python
if result.is_success():
    ...
```

instead of:

```python
if result:
    ...
```

---

# Truthiness Fallback: `__len__`

If a class does not define `__bool__`, Python can use `__len__`.

If length is zero, the object is falsey.

If length is nonzero, the object is truthy.

Example:

```python
class Playlist:
    def __init__(self):
        self.songs = []

    def __len__(self):
        return len(self.songs)
```

Now:

```python
playlist = Playlist()
print(bool(playlist))
```

Output:

```python
False
```

After adding a song:

```python
playlist.songs.append("Song A")
print(bool(playlist))
```

Output:

```python
True
```

This is why built-in empty lists, dictionaries, sets, strings, and tuples are falsey.

They have length zero.

---

# Equality: `__eq__`

`__eq__` defines equality behavior.

It powers:

```python
a == b
```

Example:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        if type(other) is not Point:
            return NotImplemented
        return self.x == other.x and self.y == other.y
```

Now:

```python
Point(1, 2) == Point(1, 2)
```

is:

```python
True
```

The `type(other) is not Point` check is deliberate.

If the other object is not a `Point`, this method does not know how to compare.

It returns `NotImplemented`.

That gives Python a chance to try the other operand's comparison behavior or fall back appropriately.

---

# `NotImplemented` Is a Value

`NotImplemented` is a special singleton value.

It means:

```text
this operation is not implemented for these operands
```

It is not the same as `NotImplementedError`.

`NotImplementedError` is an exception often used in unfinished abstract-ish methods:

```python
def save(self):
    raise NotImplementedError
```

`NotImplemented` is returned from binary special methods:

```python
def __eq__(self, other):
    if type(other) is not Point:
        return NotImplemented
    return self.x == other.x and self.y == other.y
```

Do not write:

```python
return NotImplementedError
```

That returns the exception class as a value.

Do not write:

```python
raise NotImplemented
```

That tries to raise a value that is not an exception.

Use:

```python
return NotImplemented
```

inside binary protocol methods when the operation does not support the other operand.

Also avoid using `NotImplemented` in Boolean contexts.

Modern Python treats that as an error.

It is a protocol signal, not a truth value.

---

# Hashing: `__hash__`

`__hash__` supports:

```python
hash(obj)
```

and hashed collections such as:

* dictionary keys
* set elements
* frozenset elements

Hashing must be consistent with equality.

The rule is:

```text
if a == b, then hash(a) must equal hash(b)
```

The reverse is not required.

Two unequal objects can have the same hash.

That is called a collision.

Example:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        if type(other) is not Point:
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))
```

Now points can be used in sets:

```python
points = {Point(1, 2), Point(1, 2)}
print(len(points))
```

Output:

```python
1
```

But this is safe only if the fields used for equality and hashing do not change while the object is in a set or dictionary.

Mutable hashable objects are dangerous.

If `x` changes after the point is inserted into a set, the set may no longer be able to find the object correctly.

This is why dataclasses avoid generating a hash for mutable value-equality classes by default.

---

# Ordering Methods

Ordering operators map to special methods:

```text
x < y   -> __lt__
x <= y  -> __le__
x > y   -> __gt__
x >= y  -> __ge__
```

Example:

```python
class Version:
    def __init__(self, major, minor, patch=0):
        self.major = major
        self.minor = minor
        self.patch = patch

    def _parts(self):
        return (self.major, self.minor, self.patch)

    def __lt__(self, other):
        if type(other) is not Version:
            return NotImplemented
        return self._parts() < other._parts()

    def __eq__(self, other):
        if type(other) is not Version:
            return NotImplemented
        return self._parts() == other._parts()
```

Now:

```python
Version(1, 2) < Version(1, 3)
```

is:

```python
True
```

You do not always need to define every ordering method manually.

The `functools.total_ordering` decorator can fill in missing ordering methods if you define `__eq__` and one ordering method.

Dataclasses can also generate ordering methods with `order=True`.

But ordering should express a natural ordering.

Do not add ordering just because sorting might happen once.

For one-off sorting, use a key function:

```python
versions.sort(key=lambda version: version._parts())
```

Implement ordering when the type itself has a stable ordering meaning.

---

# Container Length: `__len__`

We already saw `__len__`, but it deserves its own place in the container family.

`__len__` should return a non-negative integer.

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __len__(self):
        return len(self._members)
```

Now:

```python
team = Team(["Asha", "Maya"])
print(len(team))
```

Output:

```python
2
```

If your object does not have a meaningful size, do not implement `__len__`.

For example, a `DatabaseConnection` does not naturally have a length.

Adding protocol methods without clear meaning makes objects confusing.

---

# Item Access: `__getitem__`

`__getitem__` supports indexing:

```python
obj[key]
```

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __getitem__(self, index):
        return self._members[index]
```

Now:

```python
team = Team(["Asha", "Maya", "Dev"])

print(team[0])
print(team[-1])
print(team[1:])
```

Output:

```text
Asha
Dev
['Maya', 'Dev']
```

Because this implementation delegates to a list, it automatically supports:

* positive indexes
* negative indexes
* slices

But that is because `self._members[index]` supports them.

If you implement `__getitem__` yourself, you decide what keys mean.

For a sequence, keys are usually integer indexes and slices.

For a mapping, keys can be arbitrary hashable objects.

Example mapping-like object:

```python
class Settings:
    def __init__(self, values):
        self._values = dict(values)

    def __getitem__(self, key):
        return self._values[key]
```

Then:

```python
settings["theme"]
```

works.

The same syntax can mean sequence indexing or mapping lookup.

The object's protocol determines the meaning.

---

# Item Assignment and Deletion

`__setitem__` supports:

```python
obj[key] = value
```

`__delitem__` supports:

```python
del obj[key]
```

Example:

```python
class Settings:
    def __init__(self):
        self._values = {}

    def __getitem__(self, key):
        return self._values[key]

    def __setitem__(self, key, value):
        self._values[key] = value

    def __delitem__(self, key):
        del self._values[key]
```

Usage:

```python
settings = Settings()
settings["theme"] = "dark"
print(settings["theme"])
del settings["theme"]
```

Only implement mutation methods if mutation belongs to the object.

An immutable collection should not implement `__setitem__`.

Trying to mimic a built-in type without honoring its expectations creates unpleasant surprises.

---

# Membership: `__contains__`

The `in` operator can use `__contains__`:

```python
item in obj
```

Example:

```python
class Team:
    def __init__(self, members):
        self._members = set(members)

    def __contains__(self, member):
        return member in self._members
```

Now:

```python
team = Team(["Asha", "Maya"])
print("Maya" in team)
print("Dev" in team)
```

Output:

```python
True
False
```

If `__contains__` is absent, Python may fall back to iteration for membership testing.

That means this can work:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)
```

Then:

```python
"Maya" in team
```

can still work by iterating.

Implement `__contains__` when you can provide clearer or faster membership behavior.

---

# Iteration: `__iter__`

The iteration protocol powers:

* `for` loops
* comprehensions
* `iter(obj)`
* unpacking
* many built-in functions

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)
```

Now:

```python
team = Team(["Asha", "Maya", "Dev"])

for member in team:
    print(member)
```

Output:

```text
Asha
Maya
Dev
```

This is often the simplest pattern.

If your object contains an internal iterable, delegate:

```python
return iter(self._members)
```

You can also make `__iter__` a generator:

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        current = self.start
        while current > 0:
            yield current
            current -= 1
```

This produces a fresh iterator each time.

That is usually what container-like objects should do.

---

# Iterators: `__next__`

An iterator is an object that returns values one at a time.

It implements:

```python
__iter__
__next__
```

Example:

```python
class CountdownIterator:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value
```

Usage:

```python
for number in CountdownIterator(3):
    print(number)
```

Output:

```text
3
2
1
```

The iterator returns itself from `__iter__`.

Its `__next__` either returns the next value or raises `StopIteration`.

This is different from a container.

A container can usually create a fresh iterator each time.

An iterator is consumed as it runs.

This distinction becomes central in the iterator and generator chapters.

---

# Callable Objects: `__call__`

`__call__` lets an object behave like a function.

Example:

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, value):
        return value * self.factor
```

Now:

```python
double = Multiplier(2)
print(double(10))
```

Output:

```python
20
```

Why use a callable object instead of a function?

Because an object can hold state:

```python
class Counter:
    def __init__(self):
        self.count = 0

    def __call__(self):
        self.count += 1
        return self.count
```

Usage:

```python
counter = Counter()
print(counter())
print(counter())
```

Output:

```python
1
2
```

Callable objects are useful for:

* configurable functions
* stateful callbacks
* decorators
* strategy objects
* small command objects

But do not make an object callable if a named method would be clearer.

This:

```python
validator(value)
```

is nice when the object's purpose is validation.

This:

```python
report()
```

may be unclear if the object has many possible actions.

Use `__call__` when calling is the object's main behavior.

---

# Context Managers: `__enter__` and `__exit__`

The `with` statement uses the context manager protocol.

Example:

```python
with open("data.txt") as file:
    text = file.read()
```

The object returned by `open()` implements context management.

It enters the context, provides a file object, and ensures cleanup.

You can implement your own:

```python
class ManagedList:
    def __init__(self):
        self.items = []

    def __enter__(self):
        print("entering")
        return self.items

    def __exit__(self, exc_type, exc, traceback):
        print("leaving")
```

Usage:

```python
with ManagedList() as items:
    items.append("a")
    items.append("b")
```

Output:

```text
entering
leaving
```

`__enter__` returns the value bound after `as`.

`__exit__` receives exception information if an exception happened inside the block.

If `__exit__` returns a true value, it suppresses the exception.

Most context managers should not suppress exceptions unless that is their explicit purpose.

Context managers are important enough to receive a full chapter later.

For now, understand the protocol:

```text
with obj as value:
    ...
```

maps to:

```text
enter using __enter__
run block
exit using __exit__
```

---

# Attribute Access Dunders

Python also provides special methods for attribute access:

```python
__getattribute__
__getattr__
__setattr__
__delattr__
```

These are powerful and easy to misuse.

Example:

```python
class LoggingObject:
    def __setattr__(self, name, value):
        print(f"setting {name} = {value!r}")
        super().__setattr__(name, value)
```

Usage:

```python
obj = LoggingObject()
obj.name = "Maya"
```

Output:

```text
setting name = 'Maya'
```

This works because assignment to an attribute goes through `__setattr__`.

But overriding attribute access can create recursion bugs.

Example of a dangerous idea:

```python
def __getattribute__(self, name):
    return self.name
```

Accessing `self.name` inside `__getattribute__` calls `__getattribute__` again.

Then again.

Then again.

Until recursion fails.

Because attribute access is deep and subtle, this chapter only introduces the family.

Descriptors and managed attributes get focused treatment in Chapter 54 and Chapter 55.

---

# Numeric Dunders

Arithmetic operators are also protocol calls.

Examples:

```text
x + y   -> __add__
x - y   -> __sub__
x * y   -> __mul__
x / y   -> __truediv__
x // y  -> __floordiv__
x % y   -> __mod__
x ** y  -> __pow__
```

Example:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"
```

Usage:

```python
print(Vector(1, 2) + Vector(3, 4))
```

Output:

```python
Vector(4, 6)
```

This is operator overloading.

Chapter 53 is dedicated to it.

Here, the key idea is simple:

```text
operators are syntax over special methods
```

Implement numeric dunders only when the operation is meaningful for your type.

Vector addition makes sense.

Adding two database connections does not.

---

# Reflected and In-Place Operations

Binary operations have more than one possible method.

For addition:

```text
x + y
```

Python may try:

```python
x.__add__(y)
```

If that returns `NotImplemented`, it may try the reflected operation:

```python
y.__radd__(x)
```

For in-place addition:

```python
x += y
```

Python may use:

```python
x.__iadd__(y)
```

These details matter for custom numeric and collection types.

Example:

```python
class Bag:
    def __init__(self, items):
        self.items = list(items)

    def __add__(self, other):
        if type(other) is not Bag:
            return NotImplemented
        return Bag(self.items + other.items)
```

This supports:

```python
Bag(["a"]) + Bag(["b"])
```

But it does not necessarily support:

```python
["a"] + Bag(["b"])
```

unless the appropriate reflected behavior exists and makes sense.

Chapter 53 explores these operator relationships carefully.

---

# Conversion Dunders

Some special methods support conversion:

```python
__int__
__float__
__complex__
__bytes__
__index__
```

Example:

```python
class Quantity:
    def __init__(self, value):
        self.value = value

    def __int__(self):
        return int(self.value)
```

Now:

```python
quantity = Quantity(12.8)
print(int(quantity))
```

Output:

```python
12
```

Use conversion dunders carefully.

Conversion should be unsurprising and loss rules should be acceptable.

`__index__` is more specific than `__int__`.

It means the object can be used where Python needs an exact integer index, such as slicing or certain numeric operations.

Do not implement conversions just to make random code pass.

Implement them when your type truly has a natural conversion.

---

# The Difference Between Protocol and Convenience

Suppose you have:

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius
```

Should you implement `__float__`?

Maybe.

If the natural numeric representation is Celsius, this can be reasonable:

```python
def __float__(self):
    return float(self.celsius)
```

But it may be ambiguous.

What if users expect Fahrenheit?

In that case, named methods are clearer:

```python
def as_celsius(self):
    return self.celsius

def as_fahrenheit(self):
    return self.celsius * 9 / 5 + 32
```

Dunder methods should represent obvious, general protocol behavior.

Named methods should represent domain-specific choices.

This distinction is one of the most important design instincts in Python.

---

# Special Method Lookup

Special methods have unusual lookup behavior.

For many operations, Python looks up the special method on the type, not just on the instance dictionary.

This surprises people.

Example:

```python
class Box:
    pass


box = Box()
box.__len__ = lambda: 10
```

You might expect:

```python
len(box)
```

to return `10`.

But it does not.

Python does not normally find special methods for built-in operations by looking at random instance attributes.

The method must be on the class:

```python
class Box:
    def __len__(self):
        return 10
```

Now:

```python
len(Box())
```

returns:

```python
10
```

This behavior exists for performance and correctness reasons.

It also means monkey patching dunder methods onto a single instance usually will not affect syntax.

If you want an object to support a protocol, define the special method on its class.

---

# Why `len(x)` Instead of `x.len()`?

Python uses built-in functions for some protocols:

```python
len(x)
iter(x)
next(x)
repr(x)
str(x)
hash(x)
bool(x)
```

This can feel unusual if you come from languages where everything is method-call syntax.

The advantage is that protocols are uniform.

`len(x)` means:

```text
ask Python for the length of x
```

not:

```text
call whatever method named len happens to exist
```

The object provides `__len__`.

The language operation is `len`.

This separates user-facing syntax from protocol implementation.

That separation is a major part of Python's style.

---

# Dunders and Duck Typing

Dunder methods are duck typing made concrete.

An object is iterable if it can produce an iterator.

It does not need to inherit from `list`.

It does not need to inherit from a special base class.

It needs to implement the iteration protocol.

Example:

```python
class Squares:
    def __init__(self, limit):
        self.limit = limit

    def __iter__(self):
        for number in range(self.limit):
            yield number * number
```

Now this works:

```python
for square in Squares(5):
    print(square)
```

The `for` loop only cares that the object is iterable.

This is why protocols matter more than inheritance for many Python APIs.

ABCs can formalize protocols.

Dunder methods implement many of them.

---

# Dunders and Dataclasses

Dataclasses generate selected dunder methods for you.

Example:

```python
from dataclasses import dataclass


@dataclass
class Point:
    x: int
    y: int
```

This creates behavior similar to:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point(x={self.x!r}, y={self.y!r})"

    def __eq__(self, other):
        if type(other) is not Point:
            return NotImplemented
        return self.x == other.x and self.y == other.y
```

Dataclasses are convenient because they understand common data model methods.

But the generated methods still have meaning.

If you write:

```python
@dataclass(order=True)
```

you are not merely asking for sorting convenience.

You are defining ordering behavior for the type.

If you write:

```python
@dataclass(frozen=True)
```

you are affecting assignment and hashing behavior.

Dataclasses are not outside the data model.

They are built on it.

---

# Dunders and Inheritance

Dunder methods are inherited like other methods, but protocol meaning can make inheritance choices more sensitive.

Example:

```python
class SizedCollection:
    def __len__(self):
        return len(self._items)
```

A subclass can inherit this if it stores `_items`:

```python
class Queue(SizedCollection):
    def __init__(self):
        self._items = []
```

Now:

```python
len(Queue())
```

works.

But this inheritance assumes the subclass has `_items`.

That is a hidden contract.

A mixin can make this explicit by naming the expectation:

```python
class SizedByItemsMixin:
    def __len__(self):
        return len(self._items)
```

Even then, document it.

Dunder methods in base classes can be useful.

They can also create surprising behavior in subclasses.

When a base class implements a protocol, every subclass may appear to support that protocol too.

Make sure that is true.

---

# Dunders and Exceptions

Special methods should raise the exceptions that match the protocol.

For item access:

```python
__getitem__
```

sequence indexes should raise `IndexError` when out of range.

mapping keys should raise `KeyError` when missing.

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __getitem__(self, index):
        return self._members[index]
```

Delegating to a list automatically raises `IndexError` for invalid indexes.

For iteration, `__next__` must raise `StopIteration` when finished.

For unsupported operand types, binary dunder methods should often return `NotImplemented`.

These details matter because Python's surrounding machinery expects them.

If you raise the wrong exception, your object may feel unlike a normal Python object.

Protocol correctness is not only about method names.

It is also about behavior.

---

# A Custom Collection Example

Let us design a small collection type.

We want a `Library` that:

* stores book titles
* has a length
* supports iteration
* supports membership testing
* supports indexing
* has a useful representation

Implementation:

```python
class Library:
    def __init__(self, books=None):
        self._books = list(books or [])

    def add(self, title):
        self._books.append(title)

    def __len__(self):
        return len(self._books)

    def __iter__(self):
        return iter(self._books)

    def __contains__(self, title):
        return title in self._books

    def __getitem__(self, index):
        return self._books[index]

    def __repr__(self):
        return f"Library({self._books!r})"
```

Usage:

```python
library = Library(["Python", "Algorithms"])

print(len(library))
print("Python" in library)
print(library[0])

for book in library:
    print(book)
```

This object feels Pythonic because it supports familiar operations.

The implementation is also simple because it delegates to a list.

The class does not expose the list directly.

It exposes a controlled object interface.

This is a good use of dunder methods.

The object has collection semantics, so collection protocols make sense.

---

# A Value Object Example

Now design a small value object:

```python
class Money:
    def __init__(self, cents, currency):
        self.cents = cents
        self.currency = currency

    def __repr__(self):
        return f"Money({self.cents!r}, {self.currency!r})"

    def __str__(self):
        return f"{self.cents / 100:.2f} {self.currency}"

    def __eq__(self, other):
        if type(other) is not Money:
            return NotImplemented
        return (
            self.cents == other.cents
            and self.currency == other.currency
        )

    def __hash__(self):
        return hash((self.cents, self.currency))
```

This supports:

```python
price = Money(1250, "INR")

print(price)
print(repr(price))
print(price == Money(1250, "INR"))
print({price})
```

But this class is mutable because attributes can be changed:

```python
price.cents = 2000
```

That makes `__hash__` risky.

A better version might prevent mutation:

```python
class Money:
    def __init__(self, cents, currency):
        self._cents = cents
        self._currency = currency

    @property
    def cents(self):
        return self._cents

    @property
    def currency(self):
        return self._currency

    def __repr__(self):
        return f"Money({self.cents!r}, {self.currency!r})"

    def __eq__(self, other):
        if type(other) is not Money:
            return NotImplemented
        return (
            self.cents == other.cents
            and self.currency == other.currency
        )

    def __hash__(self):
        return hash((self.cents, self.currency))
```

Or use a frozen dataclass:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Money:
    cents: int
    currency: str
```

The lesson is the same:

```text
dunder methods define object semantics
```

Do not implement them mechanically.

Make sure the object's state model supports them.

---

# A Callable Strategy Example

Callable objects are useful when a function needs configuration.

Example:

```python
class MinLength:
    def __init__(self, length):
        self.length = length

    def __call__(self, value):
        return len(value) >= self.length
```

Usage:

```python
password_rule = MinLength(8)

print(password_rule("short"))
print(password_rule("long-enough"))
```

Output:

```python
False
True
```

This reads well because the object is a rule.

Calling the rule checks the value.

But if the object has many actions, named methods are better:

```python
rule.validate(value)
rule.describe()
rule.error_message()
```

`__call__` should not hide complexity.

It should express the object's natural action.

---

# A Context Manager Example

Here is a simple timer context manager:

```python
from time import perf_counter


class Timer:
    def __enter__(self):
        self.start = perf_counter()
        return self

    def __exit__(self, exc_type, exc, traceback):
        self.end = perf_counter()
        self.elapsed = self.end - self.start
```

Usage:

```python
with Timer() as timer:
    total = sum(range(1_000_000))

print(timer.elapsed)
```

This object uses the context manager protocol to mark a region of execution.

Notice that `__exit__` does not return `True`.

So if an exception happens inside the block, it is not suppressed.

That is usually what you want.

The object provides a clean syntax for setup and teardown.

This is exactly what context managers are for.

---

# Designing Dunder Methods

Before implementing a dunder method, ask:

```text
Does this object naturally support this operation?
```

If yes, ask:

```text
Will users correctly predict what this operation means?
```

If yes, implementing the dunder may be a good idea.

Example:

```python
len(team)
```

is predictable.

Example:

```python
team[0]
```

is predictable if the team has an order.

Example:

```python
team + team
```

may be less obvious.

Does it merge members?

Does it preserve duplicates?

Does it combine roles?

If the meaning is not obvious, use a named method:

```python
team.merge(other)
```

Pythonic does not mean adding every possible dunder.

Pythonic means making objects fit the language where the fit is natural.

---

# Common Mistake: Implementing Dunders for Cleverness

This is clever but bad:

```python
class EmailSender:
    def __add__(self, message):
        self.send(message)
```

Then:

```python
sender + message
```

sends an email.

That is surprising.

Addition should feel like addition.

If you want to send an email, write:

```python
sender.send(message)
```

Dunder methods should reduce surprise.

They should not create private jokes in the API.

Code is read more often than it is written.

Make operations mean what readers expect them to mean.

---

# Common Mistake: Returning the Wrong Type

Special methods often have expected return types.

`__len__` should return an integer:

```python
def __len__(self):
    return "many"  # wrong
```

`__bool__` should return a Boolean:

```python
def __bool__(self):
    return 1  # wrong
```

`__iter__` should return an iterator:

```python
def __iter__(self):
    return [1, 2, 3]  # wrong
```

A list is iterable, but it is not itself an iterator.

Use:

```python
def __iter__(self):
    return iter([1, 2, 3])
```

or:

```python
def __iter__(self):
    yield 1
    yield 2
    yield 3
```

Special methods are contracts.

The method name alone is not enough.

The behavior must match the contract.

---

# Common Mistake: Raising Instead of Returning `NotImplemented`

For unsupported binary operations, beginners often write:

```python
def __eq__(self, other):
    if type(other) is not Point:
        raise TypeError("cannot compare")
    return self.x == other.x and self.y == other.y
```

Sometimes raising is appropriate.

But for many binary protocol methods, returning `NotImplemented` is better:

```python
def __eq__(self, other):
    if type(other) is not Point:
        return NotImplemented
    return self.x == other.x and self.y == other.y
```

This lets Python handle fallback behavior.

For arithmetic:

```python
def __add__(self, other):
    if type(other) is not Vector:
        return NotImplemented
    return Vector(self.x + other.x, self.y + other.y)
```

Returning `NotImplemented` does not mean your method is unfinished.

It means this operation is not supported for this particular other operand.

That is an important distinction.

---

# Common Mistake: Breaking Hash Invariants

This class is dangerous:

```python
class User:
    def __init__(self, email):
        self.email = email

    def __eq__(self, other):
        if type(other) is not User:
            return NotImplemented
        return self.email == other.email

    def __hash__(self):
        return hash(self.email)
```

It looks fine.

But:

```python
user = User("a@example.com")
users = {user}

user.email = "b@example.com"
```

Now the hash changed after insertion.

The set is in trouble.

If equality depends on mutable state, avoid hashing.

Or make the object immutable.

Hashing is not a decorative method.

It is part of a collection's internal correctness.

---

# Common Mistake: Confusing `__repr__` and `__str__`

This is weak:

```python
def __repr__(self):
    return "nice user"
```

It may be friendly, but it is not useful for debugging.

Better:

```python
def __repr__(self):
    return f"User(id={self.id!r}, email={self.email!r})"
```

Then optionally:

```python
def __str__(self):
    return self.email
```

Use:

```text
__repr__ -> developer-facing, unambiguous
__str__  -> human-facing, readable
```

If you define only one, define `__repr__` first for internal objects.

---

# Common Mistake: Making Truthiness Ambiguous

Suppose:

```python
class ApiResponse:
    def __init__(self, status_code, body):
        self.status_code = status_code
        self.body = body

    def __bool__(self):
        return bool(self.body)
```

Now:

```python
if response:
    ...
```

means:

```text
body is non-empty
```

But many readers may assume it means:

```text
request succeeded
```

A clearer design might be:

```python
def is_success(self):
    return 200 <= self.status_code < 300
```

Then:

```python
if response.is_success():
    ...
```

Truthiness should be obvious.

When it is not, use a named method.

---

# Common Mistake: Exposing Internal Mutability

Suppose:

```python
class Team:
    def __iter__(self):
        return iter(self._members)
```

That is usually fine.

But if you expose the internal list directly elsewhere:

```python
def members(self):
    return self._members
```

callers can mutate it:

```python
team.members().clear()
```

Dunder methods often make objects feel like collections.

That does not mean you must expose all internals.

If mutation should be controlled, return copies or immutable views:

```python
def members(self):
    return tuple(self._members)
```

A Pythonic object can still protect its invariants.

---

# Protocol Completeness

If you implement part of a protocol, ask what else users will expect.

If an object supports `len(obj)`, should it also support iteration?

Maybe.

If it supports `obj[index]`, should it support slicing?

Maybe.

If it supports equality, should it support hashing?

Only if it is immutable enough.

If it supports ordering, does it have a clear total ordering?

Maybe not.

Python does not force you to implement complete protocols.

But users bring expectations from built-in types.

If your object feels like a sequence, people may expect:

* `len(obj)`
* `obj[0]`
* `for item in obj`
* `item in obj`
* slicing

If your object feels like a mapping, people may expect:

* `obj[key]`
* `key in obj`
* iteration over keys
* `.items()`
* `.get()`

You do not need to implement everything.

But you should be aware of the shape you are suggesting.

---

# Dunders and ABCs from `collections.abc`

The `collections.abc` module contains abstract base classes for common protocols.

For example:

* `Iterable`
* `Iterator`
* `Sized`
* `Container`
* `Sequence`
* `MutableSequence`
* `Mapping`
* `MutableMapping`
* `Callable`

These names correspond to protocol ideas.

Example:

```python
from collections.abc import Sized


class Team:
    def __len__(self):
        return 0


print(isinstance(Team(), Sized))
```

Many collection ABCs use special methods to determine behavior.

This connects Chapter 50 to this chapter.

ABCs can name a protocol.

Dunder methods implement the protocol.

Duck typing can use the behavior without requiring explicit inheritance.

These are not separate worlds.

They are different ways to talk about object capability.

---

# A Mental Map of Common Dunder Families

Here is a compact map.

Construction and lifecycle:

```text
__new__
__init__
__del__
```

Representation:

```text
__repr__
__str__
__format__
```

Truth and size:

```text
__bool__
__len__
```

Comparison and hashing:

```text
__eq__
__ne__
__lt__
__le__
__gt__
__ge__
__hash__
```

Containers:

```text
__getitem__
__setitem__
__delitem__
__contains__
```

Iteration:

```text
__iter__
__next__
```

Calling:

```text
__call__
```

Context management:

```text
__enter__
__exit__
```

Attribute access:

```text
__getattribute__
__getattr__
__setattr__
__delattr__
```

Descriptors:

```text
__get__
__set__
__delete__
__set_name__
```

Class creation:

```text
__init_subclass__
__class_getitem__
```

Metaclass-related behavior goes even deeper.

You do not need to memorize this map.

Use it as a way to orient yourself.

When you see a dunder method, ask:

```text
which protocol family does this belong to?
```

---

# How to Read Dunder Documentation

When reading Python documentation for a dunder method, look for four things.

First, identify the operation:

```text
Which syntax or built-in function calls this method?
```

Second, identify the expected return:

```text
What must this method return?
```

Third, identify fallback behavior:

```text
What happens if the method is absent or returns NotImplemented?
```

Fourth, identify invariants:

```text
What rules must remain true?
```

For `__hash__`, the invariant is equality consistency.

For `__iter__`, the return must be an iterator.

For `__next__`, exhaustion must raise `StopIteration`.

For `__repr__`, the result must be a string.

This reading habit matters more than memorization.

Python has many special methods.

Professional Python is not about knowing all of them by heart.

It is about understanding how to learn and apply the protocol correctly.

---

# Practice: Add Length and Iteration

Start with:

```python
class Playlist:
    def __init__(self, songs):
        self._songs = list(songs)
```

Add:

* `__len__`
* `__iter__`
* `__contains__`
* `__repr__`

One possible solution:

```python
class Playlist:
    def __init__(self, songs):
        self._songs = list(songs)

    def __len__(self):
        return len(self._songs)

    def __iter__(self):
        return iter(self._songs)

    def __contains__(self, song):
        return song in self._songs

    def __repr__(self):
        return f"Playlist({self._songs!r})"
```

Test:

```python
playlist = Playlist(["A", "B"])

assert len(playlist) == 2
assert "A" in playlist
assert list(playlist) == ["A", "B"]
assert repr(playlist) == "Playlist(['A', 'B'])"
```

---

# Practice: Implement Equality Safely

Create a `Point` class with:

* `x`
* `y`
* `__eq__`
* `__repr__`

Solution:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        if type(other) is not Point:
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __repr__(self):
        return f"Point({self.x!r}, {self.y!r})"
```

Test:

```python
assert Point(1, 2) == Point(1, 2)
assert Point(1, 2) != Point(2, 1)
assert Point(1, 2) != (1, 2)
```

Notice the use of `NotImplemented`.

The class does not claim to know how to compare itself to every possible object.

---

# Practice: Make a Callable Rule

Create a callable object that checks whether a string contains a required word.

Solution:

```python
class ContainsWord:
    def __init__(self, word):
        self.word = word

    def __call__(self, text):
        return self.word in text
```

Usage:

```python
has_python = ContainsWord("Python")

assert has_python("Python is fun")
assert not has_python("Java is fun")
```

Ask yourself:

```text
Is __call__ clear here?
```

Yes.

The object is a rule.

Calling the rule applies it.

---

# Practice: Avoid a Bad Dunder

Imagine this class:

```python
class EmailSender:
    def __call__(self, message):
        self.send(message)

    def send(self, message):
        ...
```

Is `__call__` a good idea?

Maybe.

If the object's entire purpose is to send messages, `sender(message)` might be acceptable.

But in application code, this may be clearer:

```python
sender.send(message)
```

Now imagine:

```python
class EmailSender:
    def __add__(self, message):
        self.send(message)
```

This is a bad idea.

Addition does not mean sending.

Use dunders only where the operation meaning is natural.

---

# Practice: Context Manager Skeleton

Fill in a context manager that prints messages before and after a block:

```python
class Announce:
    def __enter__(self):
        ...

    def __exit__(self, exc_type, exc, traceback):
        ...
```

Solution:

```python
class Announce:
    def __enter__(self):
        print("start")
        return self

    def __exit__(self, exc_type, exc, traceback):
        print("end")
```

Usage:

```python
with Announce():
    print("inside")
```

Output:

```text
start
inside
end
```

This demonstrates the protocol.

Real context managers usually manage resources, state, or temporary conditions.

---

# Practice: Special Method Lookup

Predict what happens:

```python
class Thing:
    pass


thing = Thing()
thing.__len__ = lambda: 5

print(thing.__len__())
print(len(thing))
```

The direct call works:

```python
5
```

But `len(thing)` does not use that instance attribute as the length protocol.

To make `len(thing)` work, define `__len__` on the class:

```python
class Thing:
    def __len__(self):
        return 5
```

This is one of the most important differences between ordinary method access and special method lookup.

---

# Summary

Dunder methods are Python's special methods.

They connect language syntax and built-in functions to object behavior.

They are protocol hooks.

`len(x)` uses the length protocol.

`x + y` uses numeric operator protocols.

`for item in x` uses the iteration protocol.

`x[key]` uses item access protocols.

`with x` uses the context manager protocol.

`x()` uses the callable protocol.

`repr(x)` and `str(x)` use representation protocols.

Dataclasses are built on this machinery because they generate selected data model methods.

Dunder methods should be implemented only when the operation naturally belongs to the object.

Special methods are not decoration.

They define how the object participates in Python itself.

Good dunder methods make custom objects feel ordinary in the best way.

Bad dunder methods make code surprising.

The design instinct is:

```text
use dunder methods to honor Python protocols, not to be clever
```

Once you see dunder methods as protocol hooks, much of Python's elegance becomes visible.

---

# Preview of Chapter 53

Chapter 52 introduced dunder methods as the general mechanism behind Python's data model.

Next we focus on one important family: operator overloading.

Chapter 53 studies how custom objects work with operators such as:

* `+`
* `-`
* `*`
* `/`
* `==`
* `<`
* `+=`
* unary operators

We will examine:

* normal binary methods such as `__add__`
* reflected methods such as `__radd__`
* in-place methods such as `__iadd__`
* comparison methods
* when to return `NotImplemented`
* how to avoid surprising operator behavior
* how to design numeric and collection-like objects responsibly

The transition is direct:

```text
dunder methods explain the protocol system
operator overloading studies one protocol family in depth
```

Operators are powerful because they are compact.

That also makes them dangerous when misused.

Chapter 53 is about using them with taste, clarity, and correctness.

