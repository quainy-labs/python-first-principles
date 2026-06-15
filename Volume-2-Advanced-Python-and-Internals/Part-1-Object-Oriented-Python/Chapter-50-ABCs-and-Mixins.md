# Chapter 50 — ABCs and Mixins

---

# Learning Objectives

By the end of this chapter, you should understand:

* What an abstract base class is.
* Why ABCs exist.
* How `ABC` and `abstractmethod` work.
* How abstract methods define required behavior.
* Why an abstract class cannot be instantiated until required methods are implemented.
* How ABCs differ from informal duck typing.
* When ABCs are useful and when they are unnecessary.
* What virtual subclasses are at a practical level.
* What `collections.abc` provides.
* What a mixin is.
* Why mixins should be small and focused.
* How mixins rely on MRO.
* How ABCs, mixins, composition, and duck typing fit together.

Chapter 49 taught duck typing.

Duck typing says:

```text
if an object provides the behavior we need, use it
```

That is often enough.

But sometimes we want more structure.

We may want to say:

```text
Any concrete subclass must implement these methods.
```

Or:

```text
This object should be recognized as part of a formal interface.
```

Or:

```text
This small reusable behavior should be mixed into multiple classes.
```

That is where ABCs and mixins enter.

---

# Concept Overview

An ABC is an abstract base class.

It defines an interface that subclasses are expected to implement.

Example:

```python
from abc import ABC, abstractmethod


class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
```

`Shape` is abstract because it has an abstract method.

You cannot create a plain `Shape`:

```python
shape = Shape()
```

This raises:

```text
TypeError
```

because `area` has not been implemented.

Concrete subclass:

```python
class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height
```

Now:

```python
rectangle = Rectangle(10, 5)
print(rectangle.area())
```

works.

The ABC defines the requirement.

The concrete subclass fulfills it.

---

# Why ABCs Exist

Duck typing is flexible.

Example:

```python
def render(shape):
    return shape.area()
```

Any object with `area()` works.

But sometimes you want a formal contract:

```text
All shapes must implement area.
Do not allow incomplete shape classes to be instantiated.
```

ABCs help with that.

They are useful when:

* You are defining a family of related types.
* Subclasses must implement certain methods.
* You want runtime `isinstance()` or `issubclass()` checks.
* You are designing a framework or library.
* You want to document expected methods clearly.
* You want a base class to provide some shared behavior and require other behavior.

ABCs make expectations explicit.

They turn an informal protocol into a formal class-level contract.

---

# Importing ABC Tools

ABCs usually use the `abc` module.

Import:

```python
from abc import ABC, abstractmethod
```

`ABC` is a helper base class.

`abstractmethod` is a decorator.

Together:

```python
from abc import ABC, abstractmethod


class Exporter(ABC):
    @abstractmethod
    def export(self, rows):
        pass
```

This says:

```text
Exporter is an abstract base class.
Concrete subclasses must implement export(rows).
```

The `abc` module also exposes lower-level machinery such as `ABCMeta`.

Most code should use:

```python
class Name(ABC):
```

rather than manually specifying `ABCMeta`.

The helper form is clear and idiomatic.

---

# Abstract Methods

An abstract method is a method that subclasses are required to implement.

Example:

```python
class Storage(ABC):
    @abstractmethod
    def save(self, key, value):
        pass
```

Incomplete subclass:

```python
class IncompleteStorage(Storage):
    pass
```

Try:

```python
storage = IncompleteStorage()
```

Python raises:

```text
TypeError
```

because `save` is still abstract.

Complete subclass:

```python
class MemoryStorage(Storage):
    def __init__(self):
        self._items = {}

    def save(self, key, value):
        self._items[key] = value
```

Now:

```python
storage = MemoryStorage()
storage.save("a", 1)
```

works.

The subclass became concrete by implementing all abstract methods.

---

# Abstract Class vs Concrete Class

An abstract class is incomplete by design.

It defines expectations.

It may also provide shared behavior.

A concrete class can be instantiated.

Example:

```python
class Formatter(ABC):
    @abstractmethod
    def format(self, data):
        pass
```

`Formatter` is abstract.

Concrete:

```python
class TextFormatter(Formatter):
    def format(self, data):
        return str(data)
```

Use:

```python
formatter = TextFormatter()
print(formatter.format({"status": "ok"}))
```

Key distinction:

```text
abstract class:
    defines required behavior
    may be incomplete
    cannot be instantiated while abstract methods remain

concrete class:
    implements required behavior
    can be instantiated
```

This helps prevent accidentally creating objects that cannot do what the interface promises.

---

# ABCs Can Provide Shared Behavior

An ABC does not have to contain only abstract methods.

It can also provide concrete methods.

Example:

```python
class Exporter(ABC):
    @abstractmethod
    def export_row(self, row):
        pass

    def export_all(self, rows):
        return [self.export_row(row) for row in rows]
```

Concrete subclass:

```python
class TextExporter(Exporter):
    def export_row(self, row):
        return ", ".join(row)
```

Use:

```python
exporter = TextExporter()
print(exporter.export_all([["Ada", "Python"], ["Grace", "Compiler"]]))
```

The subclass implements:

```python
export_row
```

The base class provides:

```python
export_all
```

This is a useful pattern:

```text
ABC defines required primitive behavior
ABC provides shared higher-level behavior
```

---

# Abstract Methods Can Have Implementations

This surprises many learners.

An abstract method can contain implementation.

Example:

```python
class Processor(ABC):
    @abstractmethod
    def process(self, value):
        print("common processing step")
```

Subclass:

```python
class TextProcessor(Processor):
    def process(self, value):
        super().process(value)
        print(value.upper())
```

Even though `Processor.process` is abstract, its implementation can be called with `super()`.

Why make it abstract then?

Because the base class wants to require subclasses to explicitly participate.

It says:

```text
There is common behavior here,
but every concrete subclass must still define its own process method.
```

This is an advanced but important point.

Abstract does not always mean empty.

It means:

```text
must be overridden before instantiation
```

---

# Abstract Properties

Properties can be abstract too.

Example:

```python
class UserLike(ABC):
    @property
    @abstractmethod
    def name(self):
        pass
```

Concrete subclass:

```python
class User(UserLike):
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

This says:

```text
Concrete UserLike objects must expose a name property.
```

Notice decorator order:

```python
@property
@abstractmethod
def name(self):
```

For abstract properties, `abstractmethod` is applied to the function, and `property` wraps it.

We will see more descriptor-related details later.

For now, remember that ABCs can require properties, not only ordinary methods.

---

# Abstract Class Methods and Static Methods

ABCs can require class methods.

Example:

```python
class Parser(ABC):
    @classmethod
    @abstractmethod
    def from_text(cls, text):
        pass
```

Concrete:

```python
class CsvParser(Parser):
    @classmethod
    def from_text(cls, text):
        rows = [line.split(",") for line in text.splitlines()]
        return cls(rows)

    def __init__(self, rows):
        self.rows = rows
```

ABCs can require static methods too:

```python
class Validator(ABC):
    @staticmethod
    @abstractmethod
    def is_valid(value):
        pass
```

The decorator order matters:

```python
@classmethod
@abstractmethod
def method(...)
```

and:

```python
@staticmethod
@abstractmethod
def method(...)
```

This is the modern style.

Legacy decorators such as `abstractclassmethod` are no longer the preferred approach.

---

# ABCs and `isinstance()`

ABCs can make runtime checks meaningful.

Example:

```python
class Storage(ABC):
    @abstractmethod
    def save(self, key, value):
        pass


class MemoryStorage(Storage):
    def save(self, key, value):
        pass
```

Check:

```python
storage = MemoryStorage()

print(isinstance(storage, Storage))
```

Output:

```text
True
```

This can be useful in public APIs:

```python
def configure_storage(storage):
    if not isinstance(storage, Storage):
        raise TypeError("storage must be a Storage")
```

But use runtime checks thoughtfully.

If duck typing is enough, you may not need the ABC check.

If a formal family matters, the ABC check can clarify intent.

---

# ABCs vs Duck Typing

Duck typing:

```python
def save(storage, key, value):
    storage.save(key, value)
```

ABC-based:

```python
class Storage(ABC):
    @abstractmethod
    def save(self, key, value):
        pass
```

Duck typing says:

```text
anything with save(key, value) works
```

ABCs say:

```text
this formal family of classes promises save(key, value)
```

Use duck typing when:

* The expected behavior is small.
* You want flexibility.
* Existing objects should work without subclassing.
* Formal runtime type checks are not needed.

Use ABCs when:

* You are designing a type family.
* You want to prevent incomplete subclasses.
* You want runtime `isinstance()` checks.
* You want shared base behavior plus required methods.

ABCs and duck typing are not enemies.

They solve different levels of structure.

---

# Virtual Subclasses

ABCs can register virtual subclasses.

Example:

```python
class MyIterable(ABC):
    @abstractmethod
    def __iter__(self):
        pass
```

You can register an unrelated class:

```python
MyIterable.register(tuple)
```

Then:

```python
print(issubclass(tuple, MyIterable))
print(isinstance((), MyIterable))
```

can be true.

Important:

```text
registration affects issubclass/isinstance
registration does not add methods
registration does not put the ABC in the class MRO
```

Virtual subclassing is advanced.

It is useful when you want runtime recognition without inheritance.

Most beginner application code does not need it.

But it explains part of how ABCs can support behavior-based recognition.

---

# `collections.abc`

Python provides many useful ABCs in `collections.abc`.

Examples include:

```python
from collections.abc import Iterable, Iterator, Sequence, Mapping, MutableMapping, Callable
```

These ABCs represent common protocols.

Example:

```python
from collections.abc import Iterable


def print_items(items):
    if not isinstance(items, Iterable):
        raise TypeError("items must be iterable")

    for item in items:
        print(item)
```

`collections.abc` helps when you need runtime checks for common behaviors.

Examples:

```python
isinstance([], Iterable)
isinstance({}, Mapping)
isinstance(lambda x: x, Callable)
```

These checks can be useful.

But remember Chapter 49:

```text
do not over-check when simply using the behavior is clearer
```

---

# ABCs and Static Type Hints

ABCs can appear in type hints.

Example:

```python
from collections.abc import Iterable


def total(values: Iterable[int]) -> int:
    amount = 0

    for value in values:
        amount += value

    return amount
```

This tells readers and type checkers:

```text
values should be iterable and produce integers
```

This is broader than:

```python
def total(values: list[int]) -> int:
```

because it accepts lists, tuples, generators, and other iterables.

Runtime ABCs, static type hints, and duck typing can support the same design idea:

```text
accept the behavior you need, not a narrower concrete type
```

Static typing will be studied later.

This chapter focuses on the runtime object model.

---

# What Mixins Are

A mixin is a class designed to add a small, focused piece of reusable behavior.

Example:

```python
class JsonMixin:
    def to_json(self):
        return str(self.__dict__)
```

Use:

```python
class User(JsonMixin):
    def __init__(self, name):
        self.name = name
```

Now:

```python
user = User("Ada")
print(user.to_json())
```

Output:

```text
{'name': 'Ada'}
```

`JsonMixin` is not usually meant to be instantiated by itself.

It is meant to be mixed into other classes.

Mixin idea:

```text
small reusable behavior
added through inheritance
not a complete standalone type
```

---

# Why Mixins Exist

Suppose several unrelated classes need the same helper behavior.

Example:

```text
User needs to_json
Product needs to_json
Order needs to_json
```

Option one:

```text
copy the method into every class
```

Bad.

Option two:

```text
create a common parent class
```

Maybe awkward if these classes are not the same kind of thing.

Option three:

```text
use a mixin
```

Example:

```python
class JsonMixin:
    def to_json(self):
        return str(self.__dict__)


class User(JsonMixin):
    ...


class Product(JsonMixin):
    ...
```

The mixin adds one focused capability.

It does not claim:

```text
User is a JsonMixin
```

in a conceptual domain sense.

It says:

```text
User includes JSON behavior
```

---

# Mixins and MRO

Mixins rely on MRO.

Example:

```python
class SaveMixin:
    def save(self):
        print("saving")


class Model:
    pass


class User(SaveMixin, Model):
    pass
```

MRO:

```python
print(User.mro())
```

Conceptually:

```text
User
SaveMixin
Model
object
```

When you call:

```python
User().save()
```

Python finds `save` in `SaveMixin`.

The mixin works because it appears in the MRO.

This is why Chapter 48 came before this one.

Mixin behavior is MRO behavior.

---

# Mixin Naming

Mixin classes are often named with `Mixin`.

Examples:

```python
JsonMixin
LoggingMixin
TimestampMixin
ValidationMixin
ReprMixin
```

This naming convention tells readers:

```text
this class adds behavior
it is probably not meant to stand alone
it is intended for multiple inheritance
```

Not every mixin must end with `Mixin`, but the suffix is useful in teaching and application code.

Clear names matter.

Compare:

```python
class Json:
```

with:

```python
class JsonMixin:
```

The second is clearer.

It communicates design intent.

---

# Small Focused Mixins

Mixins should be small and focused.

Good:

```python
class TimestampMixin:
    def touch(self):
        self.updated_at = "now"
```

Less good:

```python
class EverythingMixin:
    def save(self): ...
    def send_email(self): ...
    def render_html(self): ...
    def validate_user(self): ...
```

A mixin should usually add one coherent capability.

Good mixin question:

```text
What single behavior does this mixin add?
```

If the answer is unclear, the mixin is too broad.

Mixins are for reusable behavior slices.

They are not storage containers for random methods.

---

# Mixins Should Have Minimal Assumptions

A mixin often depends on the class it is mixed into.

Example:

```python
class DisplayNameMixin:
    def display_name(self):
        return self.name.title()
```

This mixin assumes:

```text
the class has a name attribute
```

That can be fine.

But the assumption should be documented or obvious.

Use:

```python
class User(DisplayNameMixin):
    def __init__(self, name):
        self.name = name
```

If mixed into a class without `name`:

```python
class Product(DisplayNameMixin):
    pass
```

then:

```python
Product().display_name()
```

raises:

```text
AttributeError
```

Mixin assumptions are informal contracts.

Keep them small and clear.

---

# Mixins and `super()`

Mixins can cooperate with `super()`.

Example:

```python
class SaveMixin:
    def save(self):
        print("SaveMixin before")
        result = super().save()
        print("SaveMixin after")
        return result
```

This assumes some later class in the MRO also has:

```python
save()
```

Complete example:

```python
class BaseModel:
    def save(self):
        print("Base save")


class AuditMixin:
    def save(self):
        print("Audit before")
        result = super().save()
        print("Audit after")
        return result


class User(AuditMixin, BaseModel):
    pass
```

Call:

```python
User().save()
```

Output:

```text
Audit before
Base save
Audit after
```

The mixin participates in the MRO chain.

---

# Mixin Order Matters

Parent order affects MRO.

Example:

```python
class Base:
    def process(self):
        print("Base")


class ValidateMixin:
    def process(self):
        print("Validate")
        super().process()


class AuditMixin:
    def process(self):
        print("Audit")
        super().process()
```

Class:

```python
class A(ValidateMixin, AuditMixin, Base):
    pass
```

Output:

```text
Validate
Audit
Base
```

Different order:

```python
class B(AuditMixin, ValidateMixin, Base):
    pass
```

Output:

```text
Audit
Validate
Base
```

Mixin order is not cosmetic.

It controls behavior order.

Always inspect MRO when combining mixins.

---

# ABCs and Mixins Together

ABCs and mixins can be combined.

Example:

```python
class Serializer(ABC):
    @abstractmethod
    def to_dict(self):
        pass


class JsonMixin:
    def to_json(self):
        return str(self.to_dict())


class User(JsonMixin, Serializer):
    def __init__(self, name):
        self.name = name

    def to_dict(self):
        return {"name": self.name}
```

Use:

```python
user = User("Ada")
print(user.to_json())
```

Here:

```text
Serializer requires to_dict
JsonMixin provides to_json using to_dict
User implements to_dict
```

This can be elegant.

But it requires clear contracts.

The mixin assumes `to_dict` exists.

The ABC makes that requirement explicit.

---

# ABCs vs Mixins

ABCs and mixins are different tools.

ABC:

```text
defines required behavior
may prevent incomplete classes from instantiating
supports runtime type checks
may provide shared behavior
```

Mixin:

```text
adds reusable behavior
is usually small and focused
often assumes certain methods or attributes exist
relies on MRO
```

They can overlap.

An ABC can provide concrete helper methods.

A mixin can be abstract.

But conceptually:

```text
ABC asks: what must subclasses provide?
Mixin asks: what reusable behavior can I add?
```

Keeping that distinction clear helps design.

---

# ABCs vs Protocols Preview

Later we will study static type protocols.

For now:

```text
ABC:
    runtime class-based interface
    can affect isinstance()
    can block instantiation

Protocol:
    static behavior-based interface
    type checker can recognize matching structure
```

Duck typing is runtime behavior.

ABCs are runtime formal class contracts.

Protocols are static structural contracts.

Example idea:

```python
class Writable(Protocol):
    def write(self, text: str) -> None:
        ...
```

An object does not need to inherit from `Writable`.

It only needs a compatible `write`.

This will matter later.

For now, ABCs are our formal runtime tool.

---

# A Complete Example: Exporters

ABC:

```python
from abc import ABC, abstractmethod


class Exporter(ABC):
    @abstractmethod
    def export(self, rows):
        pass
```

Concrete exporters:

```python
class CsvExporter(Exporter):
    def export(self, rows):
        return "\n".join(",".join(row) for row in rows)


class JsonExporter(Exporter):
    def export(self, rows):
        return str(rows)
```

Use:

```python
def export_report(exporter, rows):
    if not isinstance(exporter, Exporter):
        raise TypeError("exporter must be an Exporter")

    return exporter.export(rows)
```

This formal check may be useful in some public APIs.

But if flexibility matters more, you could omit the check and rely on duck typing:

```python
def export_report(exporter, rows):
    return exporter.export(rows)
```

Design decides.

---

# A Complete Example: Repository ABC

ABC:

```python
class UserRepository(ABC):
    @abstractmethod
    def save(self, user_id, user):
        pass

    @abstractmethod
    def get(self, user_id):
        pass
```

Concrete:

```python
class InMemoryUserRepository(UserRepository):
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
        user = {"id": user_id, "name": name}
        self._repository.save(user_id, user)
        return user
```

The ABC documents:

```text
repository must support save and get
```

The service may or may not check `isinstance`.

Often, type hints and tests are enough.

Runtime enforcement is a design choice.

---

# A Complete Example: Mixins

Mixin:

```python
class ReprMixin:
    def __repr__(self):
        values = ", ".join(
            f"{key}={value!r}"
            for key, value in vars(self).items()
        )
        return f"{type(self).__name__}({values})"
```

Class:

```python
class User(ReprMixin):
    def __init__(self, name, email):
        self.name = name
        self.email = email
```

Use:

```python
user = User("Ada", "ada@example.com")
print(user)
```

Output:

```text
User(name='Ada', email='ada@example.com')
```

The mixin adds representation behavior.

It assumes:

```text
vars(self) works
```

That is okay for ordinary classes.

But for classes using `__slots__`, this mixin may fail.

Mixin assumptions matter.

---

# A Complete Example: ABC Plus Mixin

ABC:

```python
class DictSerializable(ABC):
    @abstractmethod
    def to_dict(self):
        pass
```

Mixin:

```python
class JsonSerializableMixin:
    def to_json(self):
        return str(self.to_dict())
```

Class:

```python
class Product(JsonSerializableMixin, DictSerializable):
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def to_dict(self):
        return {"name": self.name, "price": self.price}
```

Use:

```python
product = Product("Book", 30)
print(product.to_json())
```

Design:

```text
DictSerializable requires to_dict
JsonSerializableMixin provides to_json using to_dict
Product supplies concrete data
```

This is a clean combination when the contract is clear.

---

# Common Mistake: Making ABCs Too Early

Do not create an ABC for every possible variation.

Too early:

```python
class NameFormatter(ABC):
    @abstractmethod
    def format_name(self, name):
        pass
```

if your program has one simple function:

```python
def format_name(name):
    return name.title()
```

ABCs add structure.

Structure is useful when it solves a real problem.

Use ABCs when:

* Multiple implementations exist or are expected.
* The required methods are stable.
* A formal family improves clarity.
* Incomplete subclasses should be blocked.

Do not use ABCs merely to make code look advanced.

Professional design uses abstraction when it earns its keep.

---

# Common Mistake: Using ABCs Instead of Duck Typing Everywhere

Rigid:

```python
def print_items(items):
    if not isinstance(items, MyIterableABC):
        raise TypeError

    for item in items:
        print(item)
```

This rejects many valid iterables unless they inherit or register.

Often better:

```python
def print_items(items):
    for item in items:
        print(item)
```

If you need a runtime check, use existing ABCs:

```python
from collections.abc import Iterable
```

But even then, ask:

```text
Is checking necessary?
Or can I simply iterate and let errors happen naturally?
```

ABCs should not erase Python's flexibility.

They should add clarity where useful.

---

# Common Mistake: Confusing Mixin With Base Type

Mixin:

```python
class JsonMixin:
    def to_json(self):
        ...
```

A user object with `JsonMixin` is not conceptually a kind of `JsonMixin`.

The mixin is not the domain identity.

It is an added capability.

Bad thinking:

```text
User is a JsonMixin
```

Better:

```text
User has JSON serialization behavior mixed in
```

This is why mixins should be small.

If a mixin starts to represent the object's identity, maybe it should be a real base class or a collaborator.

Use mixins for capability slices, not domain taxonomy.

---

# Common Mistake: Broad Mixins

Bad:

```python
class UtilityMixin:
    def save(self): ...
    def validate(self): ...
    def send_email(self): ...
    def to_json(self): ...
```

This mixin has no focused meaning.

Better:

```python
class JsonMixin:
    def to_json(self): ...


class ValidationMixin:
    def validate(self): ...
```

Even then, ask whether composition would be clearer.

Broad mixins create hidden dependencies and method collisions.

Small mixins keep MRO understandable.

Mixin rule:

```text
one focused capability
clear assumptions
minimal state
```

---

# Common Mistake: Mixins With Heavy State

Mixins that require lots of state can be fragile.

Example:

```python
class EmailMixin:
    def send_email(self):
        self.smtp_client.send(self.email_address, self.message)
```

This assumes:

```text
self.smtp_client
self.email_address
self.message
```

exist.

That may be too much implicit contract.

Composition may be clearer:

```python
class EmailSender:
    def send(self, address, message):
        ...
```

Then:

```python
class UserService:
    def __init__(self, sender):
        self._sender = sender
```

Use mixins when assumptions are small and natural.

Use composition when a collaborator with state and behavior is clearer.

---

# Common Mistake: Ignoring MRO in Mixins

If two mixins define the same method, order matters.

Example:

```python
class A:
    def process(self):
        print("A")


class B:
    def process(self):
        print("B")


class C(A, B):
    pass
```

`C().process()` uses `A.process`.

If:

```python
class C(B, A):
    pass
```

it uses `B.process`.

With mixins, method collisions can be accidental.

Always inspect:

```python
ClassName.mro()
```

when combining mixins.

MRO is the truth.

---

# Common Mistake: Forgetting to Implement Abstract Methods

ABC:

```python
class Storage(ABC):
    @abstractmethod
    def save(self, key, value):
        pass
```

Subclass:

```python
class MemoryStorage(Storage):
    pass
```

Try:

```python
MemoryStorage()
```

This raises:

```text
TypeError
```

because `save` remains abstract.

This is the ABC doing its job.

The class is incomplete.

Implement the method:

```python
class MemoryStorage(Storage):
    def save(self, key, value):
        ...
```

Now the class can be instantiated.

---

# Common Mistake: Wrong Method Signature

ABC:

```python
class Sender(ABC):
    @abstractmethod
    def send(self, message):
        pass
```

Subclass:

```python
class EmailSender(Sender):
    def send(self):
        print("sending")
```

Python considers the abstract method implemented because a method named `send` exists.

But the signature is incompatible.

Calling:

```python
sender.send("hello")
```

fails.

ABCs enforce method presence by name.

They do not fully verify behavioral compatibility or all signature expectations at runtime.

Static type checkers can help with that later.

Design still matters.

---

# Design Guidance

When deciding between duck typing, ABCs, mixins, and composition, ask:

```text
Do I need a formal runtime contract?
Do I need to prevent incomplete subclasses?
Do I need runtime isinstance checks?
Is behavior small enough for duck typing?
Is reusable behavior small enough for a mixin?
Does this mixin assume too much state?
Would composition make dependencies clearer?
Does parent order and MRO remain understandable?
Are method names likely to collide?
```

General guidance:

* Start with simple duck typing when behavior expectations are small.
* Use ABCs when a formal family or runtime enforcement helps.
* Use `collections.abc` for common runtime protocol checks.
* Use mixins for small reusable behavior slices.
* Keep mixins focused and low-state.
* Use composition for dependencies and stateful collaborators.
* Inspect MRO when using mixins.
* Do not create abstraction just for decoration.

The right tool depends on the design pressure.

Python gives you several layers.

Use the lightest one that makes the code clear.

---

# Exercises

1. Create an ABC named `Shape` with an abstract method:

```python
area()
```

Try to instantiate `Shape`.

What happens?

---

2. Create `Rectangle(Shape)` and implement `area`.

Instantiate `Rectangle`.

Why does it work now?

---

3. Create an ABC named `Storage` with abstract methods:

```python
save(key, value)
get(key)
```

Create an incomplete subclass and observe the error.

Then complete it.

---

4. Write an abstract property:

```python
name
```

on an ABC.

Implement it in a concrete subclass.

---

5. Use `collections.abc.Iterable` to check whether a few objects are iterable:

```python
[]
{}
"abc"
123
```

Which are iterable?

---

6. Create a `JsonMixin` with:

```python
to_json()
```

Use it in two unrelated classes.

What assumption does your mixin make?

---

7. Create two mixins that both define:

```python
process()
```

Use them in different parent orders.

How does MRO change the result?

---

8. Combine an ABC and a mixin:

* ABC requires `to_dict`.
* Mixin provides `to_json` using `to_dict`.
* Concrete class implements `to_dict`.

Explain why this combination works.

---

9. Explain why a mixin with many required attributes may be worse than composition.

Give an example.

---

10. In your own words, compare:

```text
duck typing
ABC
mixin
composition
```

Use one practical example for each.

---

# Summary

In this chapter we learned:

* An ABC is an abstract base class.
* ABCs define formal runtime contracts for subclasses.
* `ABC` is the common helper base class for defining ABCs.
* `abstractmethod` marks methods that concrete subclasses must implement.
* Classes with unimplemented abstract methods cannot be instantiated.
* Abstract methods can still contain implementation.
* Properties, class methods, and static methods can be abstract.
* ABCs can provide shared concrete behavior.
* ABCs can support meaningful `isinstance()` and `issubclass()` checks.
* Virtual subclass registration can affect runtime subclass checks without changing MRO or adding methods.
* `collections.abc` provides ABCs for common container and callable protocols.
* Mixins add small, focused reusable behavior.
* Mixins rely on MRO.
* Mixin order matters.
* Mixins should have clear assumptions and minimal state.
* ABCs and mixins can work together.
* Composition is often better for stateful collaborators and dependencies.

Core model:

```text
ABC:
    defines required behavior
    blocks incomplete concrete subclasses

mixin:
    adds focused reusable behavior
    relies on MRO

duck typing:
    accepts behavior without formal inheritance

composition:
    connects objects as collaborators
```

Design model:

```text
small informal expectation -> duck typing
formal runtime contract -> ABC
small reusable behavior -> mixin
stateful dependency -> composition
static behavior contract -> Protocol later
```

ABCs and mixins add structure to Python's flexible object model.

Used carefully, they make designs clearer.

Used too early or too broadly, they create ceremony and confusion.

---

# Preview of Chapter 51

Next we study dataclasses.

Part I has shown how classes model behavior, state, relationships, inheritance, MRO, duck typing, ABCs, and mixins.

But many classes mainly represent structured data.

Writing those classes by hand can be repetitive.

Chapter 51 introduces dataclasses as a practical tool for structured objects.

We will study:

* What dataclasses are.
* Why they reduce boilerplate.
* How field definitions work.
* How generated `__init__`, `__repr__`, and comparison behavior work.
* When dataclasses are better than handwritten classes.
* When normal classes are still better.
* How dataclasses connect to type hints.
* Common dataclass mistakes.

The transition is direct:

```text
ABCs and mixins structure behavior
dataclasses streamline structured state
```

Dataclasses will complete Volume II Part I before we move into Python's data model.
