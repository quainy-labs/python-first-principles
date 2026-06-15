# Chapter 45 — Encapsulation and Managed Attributes

---

# Learning Objectives

By the end of this chapter, you should understand:

* What encapsulation means in Python.
* Why encapsulation is about interface design, not only hiding data.
* How Python treats public attributes.
* What leading underscores mean by convention.
* What double-underscore name mangling does.
* Why Python does not use strict private fields by default.
* When direct attribute access is appropriate.
* When methods are better than attributes.
* What a managed attribute is.
* How `property` lets method logic appear as attribute access.
* How property getters, setters, and deleters work.
* How validation can happen during assignment.
* Why properties help APIs evolve.
* How managed attributes prepare us for descriptors.

Chapter 44 showed how attributes are found and assigned.

Now we ask a design question:

```text
Which attributes should other code access directly,
and which details should stay inside the object?
```

That question is encapsulation.

In some languages, encapsulation is mainly about strict access modifiers:

```text
public
private
protected
```

Python takes a different approach.

Python gives you powerful tools and clear conventions.

It expects programmers to design readable interfaces and respect boundaries.

This chapter explains that style carefully.

---

# Concept Overview

Encapsulation means organizing an object so that its public interface is clear while its internal details can change.

A public interface is what other code is expected to use.

Internal implementation is how the object makes that interface work.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name

    def display_name(self):
        return self.name.title()
```

Public interface:

```text
user.name
user.display_name()
```

Internal implementation:

```text
how display_name formats the name
how name is stored
whether validation is added later
```

Encapsulation does not always mean hiding every attribute.

In Python, simple public attributes are normal.

Example:

```python
user.name
task.done
point.x
point.y
```

These can be perfectly good public interfaces.

The key is not:

```text
never expose attributes
```

The key is:

```text
expose a clear interface and keep implementation details flexible
```

---

# Encapsulation Is Interface Design

A class is not only a container for data.

It is a boundary.

Other code interacts with objects through that boundary.

Example:

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner
        self.balance = balance
```

This exposes:

```python
account.balance
```

Other code can write:

```python
account.balance = -100
```

Maybe that is invalid.

If balance must never be negative, the class needs an interface that protects that rule.

Better:

```python
class BankAccount:
    def __init__(self, owner, balance):
        if balance < 0:
            raise ValueError("balance cannot be negative")

        self.owner = owner
        self._balance = balance

    @property
    def balance(self):
        return self._balance
```

Now other code can read:

```python
account.balance
```

but cannot casually assign:

```python
account.balance = -100
```

unless a setter is provided.

Encapsulation protects meaning.

It helps the object remain valid.

---

# Public Attributes

In Python, public attributes are normal.

Example:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

Use:

```python
point = Point(10, 20)
print(point.x)
point.x = 15
```

This is not bad Python.

It is idiomatic when the attributes are simple data and direct access is part of the object's intended interface.

Good public attributes are:

* Easy to understand.
* Safe to read.
* Safe to assign if assignment is allowed.
* Part of the object's intended usage.

Examples:

```python
point.x
point.y
user.name
task.title
task.done
```

Do not create getters and setters automatically just because another language would.

Python does not require that style.

---

# Python Does Not Require Getter Methods for Simple Data

In some languages, you might write:

```python
user.get_name()
user.set_name("Ada")
```

In Python, this is often unnecessary.

Prefer:

```python
user.name
user.name = "Ada"
```

when direct access is safe and meaningful.

Bad Python style:

```python
class User:
    def __init__(self, name):
        self._name = name

    def get_name(self):
        return self._name

    def set_name(self, name):
        self._name = name
```

If there is no validation, computation, or abstraction, this adds noise.

Better:

```python
class User:
    def __init__(self, name):
        self.name = name
```

If validation becomes necessary later, Python gives us `property`.

That is one reason properties are so important.

They let public attribute syntax remain stable while implementation changes.

---

# Internal Attributes

Python uses a leading underscore to mark internal attributes.

Example:

```python
class User:
    def __init__(self, name):
        self._name = name
```

The name:

```python
_name
```

means:

```text
internal implementation detail
do not use directly from outside unless you have a special reason
```

This is a convention.

Python does not prevent:

```python
user._name
```

But the underscore communicates intent.

It tells readers:

```text
this is not the public API
```

Public:

```python
user.name
```

Internal:

```python
user._name
```

This convention is everywhere in Python.

Respect it.

---

# Convention Is Real

Some beginners think:

```text
If Python does not enforce it, it does not matter.
```

That is not how Python culture works.

Conventions are part of the language ecosystem.

A leading underscore is not a lock.

It is a sign.

Like a sign on a door:

```text
Staff Only
```

You may physically be able to enter.

But the sign tells you that you are crossing a boundary.

When code uses:

```python
obj._cache
obj._connection
obj._raw_data
```

it is saying:

```text
this may change
this may be incomplete
this may require invariants
this is not promised to external callers
```

Python trusts programmers to understand and respect these signals.

That trust makes Python flexible.

It also requires discipline.

---

# Double Underscore Name Mangling

Python has a stronger mechanism called name mangling.

If an attribute starts with two underscores and does not end with two underscores, Python rewrites the name inside the class.

Example:

```python
class User:
    def __init__(self, name):
        self.__name = name
```

The attribute is not stored literally as:

```text
__name
```

It is transformed to something like:

```text
_User__name
```

Try:

```python
user = User("Ada")
print(user.__dict__)
```

Output:

```text
{'_User__name': 'Ada'}
```

This is name mangling.

It is designed mainly to avoid accidental name collisions in subclasses.

It is not absolute privacy.

You can still access:

```python
user._User__name
```

if you insist.

But you usually should not.

---

# Why Name Mangling Exists

Name mangling helps avoid accidental attribute collisions in inheritance.

Example:

```python
class Base:
    def __init__(self):
        self.__value = "base"


class Child(Base):
    def __init__(self):
        super().__init__()
        self.__value = "child"
```

The names become:

```text
Base attribute:
    _Base__value

Child attribute:
    _Child__value
```

They do not overwrite each other.

This matters when base classes want internal attributes that subclasses are unlikely to collide with accidentally.

But do not use double underscores everywhere.

Most internal attributes should use a single underscore:

```python
_value
```

Use double underscores when avoiding subclass collisions is genuinely useful.

---

# Dunder Names Are Different

Names with double underscores on both sides are special method names.

Examples:

```python
__init__
__str__
__len__
__enter__
__exit__
```

These are often called dunder names.

Dunder means:

```text
double underscore
```

Do not invent your own arbitrary dunder names.

Names like:

```python
__my_custom_system__
```

may conflict with future Python features or confuse readers.

Use normal names for your own APIs.

Important distinction:

```text
_name     -> internal by convention
__name    -> name-mangled inside class
__name__  -> Python special protocol name
```

These three patterns mean different things.

---

# Encapsulation Without Strict Privacy

Python does not usually stop you from accessing internal attributes.

Example:

```python
class User:
    def __init__(self, name):
        self._name = name
```

External code can do:

```python
user._name = ""
```

Python allows it.

But that does not mean it is good design.

Python's philosophy is often summarized as:

```text
we are all responsible adults here
```

The idea is:

* The class communicates its intended interface.
* Callers respect that interface.
* Advanced users can reach internals when debugging or extending.
* The language does not force rigid walls everywhere.

This style is flexible.

It rewards clear naming and documentation.

Encapsulation in Python is about boundaries, not barricades.

---

# Public API

A public API is the set of names other code is expected to use.

For a class, public API can include:

* Constructor arguments.
* Public attributes.
* Public methods.
* Properties.
* Exceptions raised.
* Behavior promised by methods.

Example:

```python
class Task:
    def __init__(self, title):
        self.title = title
        self.done = False

    def complete(self):
        self.done = True
```

Public API:

```text
Task(title)
task.title
task.done
task.complete()
```

If other code uses these, changing them later may break callers.

Internal implementation might include:

```python
self._created_at
self._history
self._normalize_title()
```

Public API is a promise.

Internal details are allowed to change.

Encapsulation is how you keep that distinction clear.

---

# Direct Attributes Are Part of the API

If you expose:

```python
user.name
```

callers may rely on it.

Example:

```python
print(user.name)
user.name = "Grace"
```

That becomes part of the object's contract.

Later, if you decide to store:

```python
self.first_name
self.last_name
```

instead of:

```python
self.name
```

you may break callers.

Properties help here.

You can keep:

```python
user.name
```

as the public interface while changing the internal storage.

That is one reason Python allows direct attributes without forcing getter methods from day one.

You can start simple.

Then add management later.

---

# Managed Attributes

A managed attribute looks like normal attribute access but runs code behind the scenes.

Example:

```python
user.name
```

might simply read:

```python
self.name
```

from the instance dictionary.

Or it might run:

```python
def name(self):
    return self._first_name + " " + self._last_name
```

Properties make this possible.

Managed attributes let you keep pleasant syntax:

```python
user.name
```

while adding:

* Validation.
* Computation.
* Read-only behavior.
* Logging.
* Lazy loading.
* Compatibility with old APIs.

The caller sees an attribute.

The class controls what happens.

---

# `property`

`property` is a built-in tool for creating managed attributes.

Example:

```python
class User:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        return self._name
```

Use:

```python
user = User("Ada")
print(user.name)
```

Output:

```text
Ada
```

Notice:

```python
user.name
```

not:

```python
user.name()
```

The method is called behind the scenes by attribute lookup.

The `@property` decorator turns a method into a managed attribute getter.

This is one of the most important object-oriented tools in Python.

---

# Read-Only Properties

If you define only a getter, the property is read-only from the outside.

Example:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius * self.radius
```

Use:

```python
circle = Circle(10)
print(circle.area)
```

Output:

```text
314.159
```

Try:

```python
circle.area = 500
```

This raises:

```text
AttributeError
```

because `area` has no setter.

This makes sense.

`area` is derived from `radius`.

It should not be assigned independently.

Property design:

```text
radius -> stored state
area   -> computed read-only attribute
```

---

# Property Setters

A property can define a setter.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        if value.strip() == "":
            raise ValueError("name cannot be empty")

        self._name = value.strip()
```

Use:

```python
user = User("  Ada  ")
print(user.name)

user.name = "Grace"
print(user.name)
```

Output:

```text
Ada
Grace
```

Try:

```python
user.name = ""
```

This raises:

```text
ValueError
```

The public interface is:

```python
user.name
```

The internal storage is:

```python
self._name
```

The setter controls assignment.

---

# Why `__init__` Uses the Property

In the previous example:

```python
def __init__(self, name):
    self.name = name
```

This calls the property setter.

That means initialization uses the same validation as later assignment.

Good:

```python
self.name = name
```

uses:

```python
@name.setter
```

Bad:

```python
self._name = name
```

inside `__init__` would bypass validation.

Sometimes bypassing is intentional.

But often, using the property inside `__init__` keeps rules consistent.

Design question:

```text
Should construction and later assignment follow the same rule?
```

If yes, call the property setter from `__init__`.

---

# Property Deleters

Properties can also define deleters.

Example:

```python
class Session:
    def __init__(self, token):
        self._token = token

    @property
    def token(self):
        return self._token

    @token.deleter
    def token(self):
        self._token = None
```

Use:

```python
session = Session("abc")
print(session.token)

del session.token
print(session.token)
```

Output:

```text
abc
None
```

Deleters are less common than getters and setters.

They are useful when deleting an attribute should perform controlled cleanup or state change.

Most classes do not need property deleters.

But knowing they exist completes the property model:

```text
getter -> read
setter -> assign
deleter -> delete
```

---

# Property Syntax

Full property pattern:

```python
class Example:
    def __init__(self, value):
        self.value = value

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, value):
        self._value = value

    @value.deleter
    def value(self):
        del self._value
```

The names must line up:

```python
@property
def value(...)

@value.setter
def value(...)

@value.deleter
def value(...)
```

The property object is stored on the class under the name `value`.

When you access:

```python
obj.value
```

the property controls what happens.

This is descriptor behavior.

We will study descriptors deeply later.

For now:

```text
property is a managed class attribute that controls instance attribute access
```

---

# Internal Storage Name

Properties often use an internal storage attribute.

Example:

```python
class User:
    @property
    def name(self):
        return self._name
```

The public property is:

```python
name
```

The internal storage is:

```python
_name
```

Why not store in `self.name` inside the getter?

Bad:

```python
@property
def name(self):
    return self.name
```

This calls the property again.

It recurses forever until Python raises:

```text
RecursionError
```

Use a different internal name:

```python
_name
```

Common pattern:

```text
public property: name
internal storage: _name
```

---

# Avoid Recursive Setters

Bad:

```python
class User:
    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self.name = value
```

The setter assigns to:

```python
self.name
```

That calls the setter again.

And again.

And again.

Eventually:

```text
RecursionError
```

Correct:

```python
@name.setter
def name(self, value):
    self._name = value
```

Inside property methods, use the internal storage attribute unless you intentionally want to invoke another managed attribute.

Rule:

```text
property name controls public access
underscore name stores internal value
```

---

# Validation With Properties

Properties are excellent for validation.

Example:

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("price cannot be negative")

        self._price = value
```

Use:

```python
product = Product("Book", 30)
product.price = 40
```

Works.

Try:

```python
product.price = -10
```

Raises:

```text
ValueError
```

The object protects its invariant:

```text
price is never negative
```

An invariant is a rule that should remain true for a valid object.

Encapsulation often exists to protect invariants.

---

# Computed Properties

Properties can compute values.

Example:

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height
```

Use:

```python
rectangle = Rectangle(10, 5)
print(rectangle.area)
```

Output:

```text
50
```

If width changes:

```python
rectangle.width = 20
print(rectangle.area)
```

Output:

```text
100
```

`area` is not stored.

It is computed each time.

This is useful when a value is derived from other state.

It keeps data consistent:

```text
do not store area separately if it can be computed from width and height
```

---

# Cached Computed Values

Sometimes computation is expensive.

You may want to compute once and cache.

Simple example:

```python
class Report:
    def __init__(self, rows):
        self.rows = rows
        self._summary = None

    @property
    def summary(self):
        if self._summary is None:
            self._summary = self._build_summary()

        return self._summary

    def _build_summary(self):
        return {"row_count": len(self.rows)}
```

Use:

```python
report = Report([1, 2, 3])
print(report.summary)
print(report.summary)
```

The summary is built the first time.

Later access reuses it.

This is called lazy computation.

Be careful with caching when underlying data can change.

If `rows` changes, should `_summary` be invalidated?

Encapsulation forces you to answer that design question.

---

# Read-Only Public Data

Sometimes callers should read a value but not assign it.

Example:

```python
class User:
    def __init__(self, user_id, name):
        self._user_id = user_id
        self.name = name

    @property
    def user_id(self):
        return self._user_id
```

Use:

```python
user = User(1, "Ada")
print(user.user_id)
```

Try:

```python
user.user_id = 2
```

This raises:

```text
AttributeError
```

The public interface says:

```text
you can read user_id
you cannot replace it
```

This is often useful for identifiers, derived values, or values that should only be set during construction.

---

# Attributes vs Methods

Should something be an attribute or a method?

Use an attribute when it feels like a property of the object:

```python
user.name
rectangle.area
account.balance
```

Use a method when it feels like an action or operation:

```python
task.complete()
account.deposit(100)
report.generate()
```

But there is judgment involved.

Example:

```python
rectangle.area
```

could be a property because area is a value-like fact about the rectangle.

Example:

```python
report.generate()
```

should probably be a method because generation sounds like an action that may do work.

Properties should not hide surprising expensive operations or side effects.

If accessing:

```python
obj.value
```

sends an email or writes to a database, that is bad design.

Attribute access should feel calm.

---

# Properties Should Feel Like Attributes

A property should behave like an attribute from the caller's point of view.

Good property:

```python
rectangle.area
```

It computes a value.

It has no surprising side effects.

Questionable property:

```python
report.download_latest_data
```

if accessing it makes a network call.

Better:

```python
report.download_latest_data()
```

because the method call signals an action.

Guideline:

```text
properties are for value-like access
methods are for actions
```

Properties can do validation or computation.

But they should not surprise the reader.

Good APIs match expectation.

---

# Evolving From Attribute to Property

One of Python's strengths is that you can start simple.

Version one:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Callers use:

```python
user.name
```

Later, you need validation.

Version two:

```python
class User:
    def __init__(self, name):
        self.name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        if value.strip() == "":
            raise ValueError("name cannot be empty")

        self._name = value.strip()
```

Callers still use:

```python
user.name
```

The public API did not change.

The implementation became managed.

This is why Python does not require getter/setter methods up front.

You can evolve.

---

# Encapsulation and Invariants

An invariant is a rule that must remain true for an object to be valid.

Examples:

```text
balance cannot be negative
name cannot be empty
percentage must be between 0 and 100
end date cannot be before start date
cart total must match item prices
```

Encapsulation helps protect invariants.

Example:

```python
class Percentage:
    def __init__(self, value):
        self.value = value

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, value):
        if not 0 <= value <= 100:
            raise ValueError("percentage must be between 0 and 100")

        self._value = value
```

Now:

```python
percentage.value = 120
```

fails.

The object guards its own validity.

That is a major purpose of managed attributes.

---

# Encapsulation and Representation

Sometimes the public interface should not reveal internal representation.

Example:

```python
class FullName:
    def __init__(self, first, last):
        self._first = first
        self._last = last

    @property
    def display(self):
        return f"{self._first} {self._last}"
```

Callers use:

```python
name.display
```

They do not need to know whether the object stores:

```text
first and last separately
```

or:

```text
one full string
```

Later, implementation can change:

```python
class FullName:
    def __init__(self, full_name):
        self._full_name = full_name

    @property
    def display(self):
        return self._full_name
```

Public interface:

```python
name.display
```

can remain stable.

Encapsulation separates what an object promises from how it stores data.

---

# Encapsulation and Mutation

Be careful when returning mutable internal objects.

Example:

```python
class Team:
    def __init__(self):
        self._members = []

    @property
    def members(self):
        return self._members
```

Callers can do:

```python
team.members.append("Ada")
```

That mutates the internal list directly.

Maybe that is acceptable.

Maybe it breaks rules.

If you want more control, provide methods:

```python
class Team:
    def __init__(self):
        self._members = []

    def add_member(self, member):
        if member not in self._members:
            self._members.append(member)

    @property
    def members(self):
        return tuple(self._members)
```

Now callers can read members but cannot mutate the internal list directly.

Returning a tuple gives a safe snapshot-like view.

Encapsulation often means controlling mutation paths.

---

# Defensive Copies

Sometimes you want to return a copy of internal mutable data.

Example:

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def add_song(self, song):
        self._songs.append(song)

    @property
    def songs(self):
        return list(self._songs)
```

Use:

```python
playlist = Playlist()
playlist.add_song("Song A")

songs = playlist.songs
songs.append("Song B")

print(playlist.songs)
```

Output:

```text
['Song A']
```

The external list was a copy.

The internal list was not mutated.

This can protect object invariants.

Tradeoff:

```text
copying costs memory and time
```

Use defensive copies when protection matters.

Do not use them blindly for every object.

---

# Methods for State Changes

When changing state requires rules, use methods.

Example:

```python
class BankAccount:
    def __init__(self, balance):
        if balance < 0:
            raise ValueError("balance cannot be negative")

        self._balance = balance

    @property
    def balance(self):
        return self._balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("deposit must be positive")

        self._balance += amount

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("withdrawal must be positive")

        if amount > self._balance:
            raise ValueError("insufficient funds")

        self._balance -= amount
```

Why not allow:

```python
account.balance = account.balance - amount
```

Because withdrawal has rules.

Methods express operations.

Properties expose state.

---

# Internal Helper Methods

Methods can also be internal.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self._name = self._normalize_name(value)

    def _normalize_name(self, value):
        return value.strip().title()
```

The method:

```python
_normalize_name
```

is internal.

External code should use:

```python
user.name
```

not:

```python
user._normalize_name(...)
```

The underscore helps keep the public API small.

Small public APIs are easier to maintain.

Internal helpers can change freely.

---

# Double Underscore for Subclass Safety

Use double underscore when a base class wants to avoid accidental subclass collisions.

Example:

```python
class BaseCounter:
    def __init__(self):
        self.__count = 0

    def increment(self):
        self.__count += 1
        return self.__count
```

The internal name becomes:

```text
_BaseCounter__count
```

If a subclass also uses:

```python
self.__count
```

it becomes:

```text
_SubclassName__count
```

The two names do not collide.

This is useful in library base classes.

For ordinary application code, a single underscore is usually enough.

Rule:

```text
_name for internal
__name when avoiding subclass name collision matters
```

---

# Encapsulation and Inheritance Preview

Inheritance complicates internal details.

If a base class exposes too many internals, subclasses may depend on them.

Example:

```python
class BaseReport:
    def __init__(self):
        self._rows = []
```

A subclass might write:

```python
class CsvReport(BaseReport):
    def add_header(self):
        self._rows.insert(0, "header")
```

Now `_rows` is no longer purely internal.

The subclass depends on it.

This may be fine if intended.

But if not, changing `_rows` later can break subclasses.

Encapsulation is not only about outside users.

It also affects subclass contracts.

We will revisit this when we study inheritance and method overriding.

---

# Encapsulation and Composition Preview

Composition means building objects out of other objects.

Example:

```python
class Engine:
    def start(self):
        print("starting")


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

It hides:

```python
car._engine
```

Callers do not need to know how the car starts internally.

Maybe later it uses:

```text
ElectricMotor
```

instead of:

```text
Engine
```

If the public API stays:

```python
car.start()
```

callers do not break.

Composition and encapsulation often work together.

Chapter 46 will focus on composition.

---

# Encapsulation and Descriptors Preview

Properties are built on the descriptor protocol.

Descriptor means:

```text
an object that controls attribute access through special methods
```

Properties are the friendly, common version.

Descriptors are the general mechanism.

Example:

```python
@property
def name(self):
    return self._name
```

Behind the scenes, `property` creates an object stored on the class.

That object participates in attribute lookup.

This is why:

```python
user.name
```

can call code.

Later, Chapter 54 will explain descriptors directly.

This chapter gives the practical foundation.

---

# A Complete Example: Product

Class:

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        value = value.strip()

        if value == "":
            raise ValueError("name cannot be empty")

        self._name = value

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("price cannot be negative")

        self._price = value
```

Use:

```python
book = Product("  Book  ", 30)

print(book.name)
print(book.price)
```

Output:

```text
Book
30
```

Try:

```python
book.price = -5
```

Raises:

```text
ValueError
```

The object controls its invariants.

Public interface:

```python
book.name
book.price
```

Internal storage:

```python
book._name
book._price
```

---

# A Complete Example: Temperature

Class:

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("temperature cannot be below absolute zero")

        self._celsius = value

    @property
    def fahrenheit(self):
        return self.celsius * 9 / 5 + 32
```

Use:

```python
temperature = Temperature(0)

print(temperature.celsius)
print(temperature.fahrenheit)
```

Output:

```text
0
32.0
```

Try:

```python
temperature.fahrenheit = 100
```

This raises:

```text
AttributeError
```

because `fahrenheit` is read-only.

If setting Fahrenheit should be supported, you can add a setter.

API design question:

```text
Should callers be allowed to assign this value?
```

Properties make that decision explicit.

---

# A Complete Example: Team

Class:

```python
class Team:
    def __init__(self, name):
        self.name = name
        self._members = []

    def add_member(self, member):
        if member not in self._members:
            self._members.append(member)

    def remove_member(self, member):
        self._members.remove(member)

    @property
    def members(self):
        return tuple(self._members)
```

Use:

```python
team = Team("Core")
team.add_member("Ada")
team.add_member("Ada")

print(team.members)
```

Output:

```text
('Ada',)
```

Why not expose `_members` directly?

Because the class wants rules:

```text
no duplicate members
controlled removal
read-only external view
```

Public API:

```python
team.add_member(...)
team.remove_member(...)
team.members
```

Internal data:

```python
team._members
```

That is encapsulation.

---

# Common Mistake: Writing Java-Style Getters Everywhere

Unnecessary:

```python
class User:
    def __init__(self, name):
        self._name = name

    def get_name(self):
        return self._name

    def set_name(self, name):
        self._name = name
```

Better when no logic is needed:

```python
class User:
    def __init__(self, name):
        self.name = name
```

Better when logic is needed:

```python
class User:
    def __init__(self, name):
        self.name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self._name = value.strip()
```

Python style prefers attribute access for value-like data.

Use methods when the operation is action-like.

Use properties when attribute access needs management.

---

# Common Mistake: Treating One Underscore as Security

This:

```python
self._token = token
```

does not make `_token` inaccessible.

External code can do:

```python
obj._token
```

The underscore is a convention.

It says:

```text
internal, do not depend on this
```

If you are storing secrets, a leading underscore is not security.

Security requires different tools and designs.

Encapsulation is about interface boundaries.

It is not a cryptographic protection mechanism.

Do not confuse internal naming with secure storage.

---

# Common Mistake: Overusing Double Underscores

Double underscores can make code harder to read and test.

Example:

```python
class User:
    def __init__(self, name):
        self.__name = name
```

This becomes:

```text
_User__name
```

If there is no inheritance collision concern, this may be unnecessary.

Single underscore is usually better:

```python
self._name = name
```

Use double underscores when:

* You are writing a base class.
* Subclasses might accidentally reuse the same internal name.
* Avoiding that collision matters.

Do not use double underscores just to make attributes look private.

Python code usually favors clarity.

---

# Common Mistake: Property Recursion

Bad getter:

```python
class User:
    @property
    def name(self):
        return self.name
```

This calls itself.

Bad setter:

```python
class User:
    @name.setter
    def name(self, value):
        self.name = value
```

This calls itself too.

Correct:

```python
class User:
    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self._name = value
```

Property name:

```text
name
```

Internal storage:

```text
_name
```

Keep them distinct.

---

# Common Mistake: Hiding Expensive Work Behind a Property

Questionable:

```python
class Report:
    @property
    def latest_data(self):
        return download_from_network()
```

Callers see:

```python
report.latest_data
```

This looks like simple attribute access.

But it performs network work.

Better:

```python
class Report:
    def download_latest_data(self):
        return download_from_network()
```

The method call signals an action.

Properties should usually be:

* Quick enough.
* Side-effect-light.
* Value-like.
* Safe to access repeatedly.

If an operation is expensive or surprising, make it a method.

---

# Common Mistake: Exposing Mutable Internals

Problem:

```python
class Team:
    def __init__(self):
        self._members = []

    @property
    def members(self):
        return self._members
```

External code:

```python
team.members.clear()
```

This bypasses class rules.

Better:

```python
@property
def members(self):
    return tuple(self._members)
```

or:

```python
def get_members(self):
    return list(self._members)
```

depending on the API style.

If callers should mutate, provide controlled methods:

```python
team.add_member(member)
team.remove_member(member)
```

Do not accidentally expose internal mutable state.

---

# Common Mistake: Validating in Only One Path

Bad:

```python
class Product:
    def __init__(self, price):
        if price < 0:
            raise ValueError("price cannot be negative")

        self.price = price
```

Later:

```python
product.price = -10
```

works, because assignment has no validation.

Better:

```python
class Product:
    def __init__(self, price):
        self.price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("price cannot be negative")

        self._price = value
```

Now construction and later assignment share the same rule.

Protect invariants at every public mutation path.

---

# Design Guidance

When designing encapsulation, ask:

```text
What should callers be allowed to read?
What should callers be allowed to change?
What rules must always remain true?
Which details might change later?
Which names are public promises?
Which names are internal implementation?
Should this be an attribute or a method?
Should this value be computed?
Should this mutable object be exposed directly?
Would a property keep the API stable?
```

General guidance:

* Use public attributes for simple, safe data.
* Use leading underscores for internal details.
* Use double underscores sparingly for subclass collision avoidance.
* Use properties when access needs logic.
* Use methods for actions.
* Avoid exposing mutable internals accidentally.
* Keep property behavior value-like and unsurprising.
* Protect invariants consistently.

Encapsulation is not ceremony.

It is care for the object boundary.

---

# Exercises

1. Write a `User` class with a public `name` attribute.

Then explain why a direct public attribute is fine if no validation is needed.

---

2. Rewrite `User` so `name` cannot be empty.

Use a property with a setter.

Make sure `__init__` uses the setter.

---

3. Explain the meaning of these three names:

```python
_name
__name
__name__
```

When should each style be used?

---

4. Create a class:

```python
class Example:
    def __init__(self):
        self.__value = 10
```

Print:

```python
vars(example)
```

What name do you see?

Why?

---

5. Create a read-only property:

```python
class Circle:
    ...
```

with:

```python
circle.area
```

computed from radius.

Try assigning to `circle.area`.

What happens?

---

6. Write a `Product` class where `price` cannot be negative.

Use a property setter.

Test both construction and later assignment.

---

7. Explain why this property is dangerous:

```python
@property
def data(self):
    return download_large_file()
```

Should this be a property or a method?

---

8. Create a `Team` class that stores `_members` internally.

Expose `members` as a tuple.

Provide `add_member()` and `remove_member()` methods.

Explain how this protects the internal list.

---

9. Fix this recursive property:

```python
class User:
    @property
    def name(self):
        return self.name
```

Why does the original recurse?

---

10. In your own words, explain:

```text
encapsulation in Python is about clear interfaces, not strict walls
```

Use public attributes, underscores, and properties in your explanation.

---

# Summary

In this chapter we learned:

* Encapsulation is about clear public interfaces and protected implementation details.
* Python commonly uses public attributes for simple data.
* Python does not require getter and setter methods for every field.
* A single leading underscore marks an internal name by convention.
* Double-leading-underscore names are name-mangled to avoid accidental subclass collisions.
* Dunder names such as `__init__` are reserved for Python protocols.
* Python does not enforce strict private fields by default.
* A public API is a promise other code may depend on.
* Managed attributes let attribute access run code.
* `property` creates managed attributes.
* Property getters control reads.
* Property setters control assignments.
* Property deleters control deletion.
* Properties are useful for validation, computed values, read-only values, and API evolution.
* Properties should feel like attributes, not surprising actions.
* Methods are better for operations with side effects or expensive work.
* Encapsulation helps protect invariants.
* Avoid exposing mutable internal objects accidentally.

Core model:

```text
public attribute:
    obj.name

internal storage:
    obj._name

managed attribute:
    @property
    def name(self): ...

setter:
    @name.setter
    def name(self, value): ...
```

Design model:

```text
simple safe data -> public attribute
internal detail -> leading underscore
validated assignment -> property setter
derived value -> read-only property
action or side effect -> method
```

Python encapsulation is practical.

It gives you conventions for communicating boundaries and tools for enforcing rules when rules matter.

---

# Preview of Chapter 46

Next we study composition.

Encapsulation showed how an object can hide internal details behind a clear interface.

Composition shows how objects can be built from other objects.

Chapter 46 will study:

* What composition means.
* How objects can own or use other objects.
* Why composition often beats inheritance.
* How encapsulation and composition work together.
* How to delegate behavior to contained objects.
* How object relationships affect lifetime and ownership.
* How composition prepares us for larger object-oriented designs.

The transition is direct:

```text
encapsulation defines object boundaries
composition connects objects across those boundaries
```

Once composition is clear, inheritance will be easier to evaluate because we will know when not to use it.
