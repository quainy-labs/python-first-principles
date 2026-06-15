# Chapter 68 — Runtime Type System

Python is dynamically typed.

That sentence is true.

It is also easy to misunderstand.

Many beginners hear "dynamically typed" and think:

```text
Python does not really have types
```

That is wrong.

Python has types everywhere.

Every object has a type.

Types control behavior.

Types define what operations an object supports.

Types participate in attribute lookup.

Types determine method resolution.

Types affect `isinstance()`, `issubclass()`, protocols, abstract base classes, descriptors, metaclasses, annotations, and static type checking.

Python is not typeless.

Python is dynamically typed because type checks and method dispatch happen at runtime, and names do not have fixed declared types enforced by the interpreter.

This distinction matters:

```text
names are not statically bound to one type
objects always have runtime types
```

Volume I taught:

```text
object = identity + type + value
```

Chapter 68 returns to that model at a deeper level.

Now we ask:

* What is a type at runtime?
* What is the relationship between a class and a type?
* What does `type(x)` return?
* Why is `isinstance()` usually better than `type(x) is SomeClass`?
* What does `issubclass()` ask?
* How do runtime checks differ from static type hints?
* What are nominal and structural type ideas?
* What do annotations do at runtime?
* Why does Python's type system feel flexible but still coherent?

This chapter prepares us for the rest of Part VI.

Before bytecode internals, CPython architecture, and C extensions, we need to understand the runtime objects those systems operate on.

---

# The Foundation: Objects Have Types

Every Python object has:

```text
identity
type
value
```

Example:

```python
value = 42

print(id(value))
print(type(value))
print(value)
```

The type is not a label attached to a variable.

The type belongs to the object.

```python
x = 42
x = "hello"
```

The name `x` first refers to an `int` object.

Then it refers to a `str` object.

The name did not change type.

The name was rebound to a different object.

This is why saying "Python variables have types" is imprecise.

Better:

```text
Python names refer to objects, and objects have types
```

That model explains dynamic typing cleanly.

---

# Runtime Typing

Runtime typing means type information exists and affects behavior while the program runs.

Example:

```python
print(len("hello"))
print(len([1, 2, 3]))
```

Both calls use `len()`.

But the behavior depends on the object.

String length:

```text
number of characters
```

List length:

```text
number of elements
```

The runtime asks the object whether its type supports the length protocol.

This is not a compile-time decision.

It happens while the program is running.

Example:

```python
def describe(value):
    print(type(value))
    print(len(value))
```

This function does not declare that `value` must be a string or list.

It accepts whatever object is passed.

If the object supports `len()`, the call works.

If not, Python raises `TypeError`.

```python
describe(10)
```

This fails because an integer does not support the length operation.

Dynamic typing does not mean all operations are allowed.

It means operation validity is discovered at runtime.

---

# type()

The built-in `type()` function returns an object's type.

```python
print(type(42))
print(type("hello"))
print(type([1, 2, 3]))
```

Output:

```text
<class 'int'>
<class 'str'>
<class 'list'>
```

Notice the word `class`.

In Python, types and classes are unified.

The type of `42` is `int`.

`int` is a class.

The type of `"hello"` is `str`.

`str` is a class.

The type of `[1, 2, 3]` is `list`.

`list` is a class.

When you define your own class:

```python
class User:
    pass

user = User()

print(type(user))
```

Output:

```text
<class '__main__.User'>
```

Your class participates in the same runtime type system as built-in classes.

There is not one type system for built-ins and another for user code.

Python's object model is unified.

---

# Classes Are Objects Too

Classes are objects.

This is not a metaphor.

```python
class User:
    pass

print(type(User))
```

Output:

```text
<class 'type'>
```

The class object `User` has type `type`.

This means:

```text
user is an instance of User
User is an instance of type
```

Diagram:

```text
user  --type-->  User
User  --type-->  type
type  --type-->  type
```

The last line is strange at first:

```python
type(type) is type
```

But it reflects Python's self-describing runtime.

Classes are objects that create objects.

`type` is the default metaclass that creates classes.

We studied metaclasses earlier.

Now we place them inside the runtime type system:

```text
instances are created by classes
classes are created by metaclasses
type is the usual metaclass
```

---

# A Class Defines Object Behavior

A type determines what operations an object supports.

Example:

```python
class Cart:
    def __init__(self):
        self.items = []

    def add(self, item):
        self.items.append(item)

cart = Cart()
cart.add("book")
```

The `Cart` class defines:

* how instances are initialized
* which attributes they usually have
* which methods can be called
* how they participate in protocols if dunder methods exist

If you add:

```python
def __len__(self):
    return len(self.items)
```

then `Cart` supports `len(cart)`.

```python
class Cart:
    def __init__(self):
        self.items = []

    def add(self, item):
        self.items.append(item)

    def __len__(self):
        return len(self.items)
```

Now:

```python
len(cart)
```

works because the type provides the length protocol.

Types are not only names.

They are behavior definitions.

---

# __class__

Objects expose their type through `__class__`.

```python
value = "hello"

print(value.__class__)
print(type(value))
```

Both refer to the object's class.

For most ordinary code, use `type(value)` when you want to inspect an object's exact type.

`__class__` matters more when understanding internals, descriptors, proxies, metaclasses, and dynamic behavior.

The important idea:

```text
an object knows its class at runtime
```

That class is itself an object.

---

# Exact Type Checks

You can check exact type identity:

```python
type(value) is str
```

This asks:

```text
is the object's exact runtime type str?
```

Sometimes exact type checks are appropriate.

Example:

```python
def serialize_scalar(value):
    if type(value) is bool:
        return "true" if value else "false"
    if type(value) is int:
        return str(value)
    raise TypeError("unsupported scalar")
```

Here exact distinction may matter because `bool` is a subclass of `int`.

```python
isinstance(True, int)
```

returns `True`.

But often exact type checks are too strict.

If a subclass behaves correctly, exact checks reject it unnecessarily.

This is why `isinstance()` is usually preferred for normal runtime type checks.

---

# isinstance()

`isinstance(object, classinfo)` asks whether an object is an instance of a class or of a subclass.

```python
isinstance("hello", str)
```

returns:

```python
True
```

Subclass example:

```python
class Animal:
    pass

class Dog(Animal):
    pass

dog = Dog()

print(isinstance(dog, Dog))
print(isinstance(dog, Animal))
```

Both are true.

The object is directly an instance of `Dog`.

It is also an instance of `Animal` through inheritance.

This is why `isinstance()` respects polymorphism.

If a function accepts any animal, it should not reject dogs just because their exact type is `Dog`.

```python
def feed(animal):
    if not isinstance(animal, Animal):
        raise TypeError("expected Animal")
```

This accepts subclasses.

Exact type checks would not.

---

# isinstance() with Tuples and Unions

`isinstance()` can accept a tuple of types.

```python
isinstance(value, (str, bytes))
```

This asks:

```text
is value a str or bytes?
```

Example:

```python
def normalize_text(value):
    if isinstance(value, bytes):
        return value.decode("utf-8")
    if isinstance(value, str):
        return value
    raise TypeError("expected str or bytes")
```

Modern Python also supports union type syntax in some `isinstance()` contexts:

```python
isinstance(value, str | bytes)
```

But be careful not to confuse runtime checks with static type hint syntax in every situation.

Some typing constructs are for type checkers, not direct runtime use.

For simple runtime checks, normal classes and tuples of classes remain clear.

---

# issubclass()

`issubclass()` asks whether one class is a subclass of another.

```python
class Animal:
    pass

class Dog(Animal):
    pass

print(issubclass(Dog, Animal))
print(issubclass(Animal, Dog))
```

Output:

```text
True
False
```

`isinstance()` asks about an object.

`issubclass()` asks about classes.

Compare:

```python
isinstance(dog, Animal)
issubclass(Dog, Animal)
```

The first asks:

```text
is this object an Animal?
```

The second asks:

```text
is this class a subclass of Animal?
```

This distinction matters when writing frameworks, serializers, registries, plugins, and class-based APIs.

---

# Nominal Typing

Nominal typing means type relationships are based on declared names and inheritance.

Example:

```python
class FileStorage:
    def save(self, data):
        ...

class S3Storage(FileStorage):
    def save(self, data):
        ...
```

`S3Storage` is a subclass of `FileStorage` because the class definition says so.

```python
issubclass(S3Storage, FileStorage)
```

This is nominal.

The relationship is based on the class hierarchy.

ABCs also participate in nominal thinking, though they can add virtual subclass behavior.

Nominal checks are useful when:

* class hierarchy is meaningful
* a framework owns the base class
* behavior requires more than method names
* registration matters
* identity of a domain concept matters

But Python often prefers a more flexible idea too.

That idea is structural typing.

---

# Structural Typing

Structural typing asks whether an object has the required structure or behavior.

It is less concerned with declared inheritance.

Example:

```python
class FileWriter:
    def write(self, text):
        print(text)

class NetworkWriter:
    def write(self, text):
        send(text)

def log_message(writer, message):
    writer.write(message)
```

`log_message()` does not care whether `writer` inherits from a specific base class.

It cares that `writer` has a usable `write()` method.

This is duck typing:

```text
if it supports the behavior needed here, use it
```

Structural thinking is natural in Python.

It appears in:

* iteration
* context managers
* file-like objects
* callables
* protocols
* static type checking

The runtime often discovers behavior by trying operations.

Static tools can describe the expected structure ahead of time.

---

# Duck Typing

Duck typing is Python's traditional flexible type style.

Instead of asking:

```text
is this object exactly this class?
```

duck typing asks:

```text
does this object support the operation I need?
```

Example:

```python
def count_items(container):
    return len(container)
```

This works for:

* strings
* lists
* tuples
* dictionaries
* sets
* custom objects with `__len__`

The function does not need to check:

```python
isinstance(container, list)
```

It needs an object that supports `len()`.

This is powerful.

It is also why overly narrow runtime checks can make Python code less useful.

Prefer checking behavior when behavior is what matters.

Prefer checking type when the exact type or hierarchy is part of the contract.

---

# EAFP and Runtime Types

Python often favors EAFP:

```text
easier to ask forgiveness than permission
```

Instead of checking a type first:

```python
if isinstance(value, Sized):
    return len(value)
```

you may simply try:

```python
try:
    return len(value)
except TypeError:
    raise TypeError("value must support len()")
```

This is useful when the behavior matters more than the class.

But do not overuse EAFP to hide unclear contracts.

If an API truly requires a specific domain type, say so.

Example:

```python
def charge(account, amount):
    if not isinstance(account, Account):
        raise TypeError("expected Account")
```

Here the domain concept may matter.

Runtime type design is not about one rule.

It is about matching the check to the contract.

---

# ABCs at Runtime

Abstract base classes can express runtime contracts.

Python's standard library includes ABCs in `collections.abc`.

Example:

```python
from collections.abc import Iterable

def consume(values):
    if not isinstance(values, Iterable):
        raise TypeError("expected iterable")
```

This is broader than checking for `list`.

It accepts lists, tuples, sets, generators, and custom iterables.

ABCs are useful when you want runtime checks for broad behavior categories:

* `Iterable`
* `Iterator`
* `Sequence`
* `Mapping`
* `MutableMapping`
* `Callable`

But remember:

```text
ABCs are still runtime objects
```

They participate in `isinstance()` and `issubclass()`.

They are not merely documentation.

---

# Protocols at Runtime

Static type protocols come from the `typing` module.

They describe structural contracts.

Example:

```python
from typing import Protocol

class SupportsClose(Protocol):
    def close(self) -> None:
        ...
```

Static type checkers can use this to verify that an object has a compatible `close()` method.

At runtime, ordinary protocols are not automatically usable with `isinstance()`.

To allow runtime checks, use `@runtime_checkable`:

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class SupportsClose(Protocol):
    def close(self) -> None:
        ...

print(isinstance(open("example.txt", "w"), SupportsClose))
```

Runtime-checkable protocols perform a relatively simple structural check.

They check for required attributes.

They do not fully verify type signatures at runtime.

This distinction is critical:

```text
static protocol checking can reason about signatures
runtime protocol checking is much shallower
```

Do not assume a runtime protocol check proves everything a static type checker would prove.

---

# Annotations

Annotations attach metadata to functions, variables, and classes.

Example:

```python
def add(a: int, b: int) -> int:
    return a + b
```

The annotations are visible at runtime:

```python
print(add.__annotations__)
```

Output:

```python
{'a': <class 'int'>, 'b': <class 'int'>, 'return': <class 'int'>}
```

But Python does not enforce them by default.

This call runs:

```python
add("a", "b")
```

It returns:

```python
"ab"
```

The annotation says `int`.

The runtime does not automatically reject strings.

Annotations are metadata.

They are used by:

* static type checkers
* IDEs
* documentation tools
* runtime validation libraries
* frameworks
* dataclass-like systems

But the interpreter itself does not turn annotations into automatic runtime checks.

---

# Type Hints Are Not Runtime Enforcement

This is one of the most important rules in modern Python:

```text
type hints describe intent; they do not enforce it by themselves
```

Example:

```python
def repeat(text: str, times: int) -> str:
    return text * times
```

This works:

```python
repeat("ha", 3)
```

This may also work:

```python
repeat(4, 3)
```

It returns:

```python
12
```

The annotation did not stop it.

A static type checker may warn before runtime.

The runtime follows the actual operation:

```python
4 * 3
```

That operation is valid.

Annotations and runtime behavior are connected, but not identical.

---

# Runtime Validation

If runtime validation matters, write validation or use a validation library.

Example:

```python
def repeat(text: str, times: int) -> str:
    if not isinstance(text, str):
        raise TypeError("text must be str")
    if not isinstance(times, int):
        raise TypeError("times must be int")
    return text * times
```

This enforces types at runtime.

But runtime validation has costs:

* more code
* runtime overhead
* decisions about subclasses
* decisions about coercion
* decisions about error messages
* decisions about nested structures

For application boundaries, validation is often worth it.

For internal functions, static checking and tests may be enough.

Common validation boundaries:

* API input
* CLI input
* config files
* JSON payloads
* database records
* user-submitted data
* plugin interfaces

Inside trusted internal code, repeated runtime checks can become noise.

Use the right tool at the right boundary.

---

# Generic Runtime Objects

Modern Python lets you write type hints such as:

```python
list[str]
dict[str, int]
tuple[int, str]
```

These are useful for annotations.

But they are not ordinary runtime classes in the same way `list` is.

This is not valid:

```python
isinstance(["a", "b"], list[str])
```

Runtime `isinstance()` checks an object's class relationship.

It does not inspect every element and verify generic type arguments.

If you need to validate a list of strings at runtime, you must check it:

```python
def validate_strings(values):
    if not isinstance(values, list):
        raise TypeError("expected list")
    if not all(isinstance(value, str) for value in values):
        raise TypeError("expected list of strings")
```

Static type hints can express:

```python
list[str]
```

Runtime validation must decide how deep to check.

That is a separate design problem.

---

# Type Aliases

Type aliases give names to type expressions.

Example:

```python
UserId = int

def load_user(user_id: UserId):
    ...
```

At runtime, `UserId` is just another name bound to `int`.

```python
print(UserId is int)
```

Output:

```python
True
```

Type aliases help readers and type checkers.

They do not create new runtime types by themselves.

If you need a distinct runtime type, use a class.

Example:

```python
class UserId:
    def __init__(self, value: int):
        self.value = value
```

A type alias says:

```text
treat this type expression as having a domain name
```

A class creates:

```text
a new runtime type
```

Do not confuse the two.

---

# NewType

`typing.NewType` creates a distinct static type for type checkers with almost no runtime overhead.

Example:

```python
from typing import NewType

UserId = NewType("UserId", int)
```

Static type checkers can distinguish `UserId` from plain `int`.

At runtime, calling `UserId(42)` returns the underlying value.

It does not create a rich wrapper object like a custom class would.

Use `NewType` when:

* you want static distinction
* runtime representation should stay cheap
* behavior is the same as the underlying type

Use a class when:

* runtime distinction matters
* behavior differs
* validation belongs with the value
* methods are needed

Again:

```text
static distinction and runtime distinction are not always the same
```

---

# isinstance() and bool

One surprising runtime relationship:

```python
isinstance(True, int)
```

returns:

```python
True
```

Why?

`bool` is a subclass of `int`.

```python
issubclass(bool, int)
```

returns:

```python
True
```

This is historically and practically important.

It means:

```python
True + True
```

returns:

```python
2
```

But semantically, booleans should usually represent truth values, not integers.

When validating numbers, decide whether booleans are acceptable.

Example:

```python
def require_int(value):
    if type(value) is not int:
        raise TypeError("expected int")
```

This rejects `True`.

Using:

```python
isinstance(value, int)
```

would accept it.

This is one case where exact type checking may be intentional.

---

# Runtime Type Checks and Inheritance

Consider:

```python
class PathLikeString(str):
    pass
```

An instance is a string:

```python
path = PathLikeString("/tmp/data.txt")

isinstance(path, str)
```

returns:

```python
True
```

Exact type check:

```python
type(path) is str
```

returns:

```python
False
```

Which is correct?

It depends on the contract.

If the function can handle any string-like subclass, use `isinstance()`.

If the function truly requires a plain `str`, use exact type checking.

Most of the time, accepting subclasses is more Pythonic.

But sometimes exact type distinction matters.

The runtime gives you both tools.

You must choose deliberately.

---

# Runtime Type Information Is Introspectable

Python lets you inspect runtime objects.

Examples:

```python
type(obj)
obj.__class__
obj.__dict__
dir(obj)
vars(obj)
callable(obj)
hasattr(obj, "name")
getattr(obj, "name")
```

This introspection powers many Python patterns:

* debuggers
* serializers
* ORMs
* web frameworks
* plugin systems
* documentation generators
* test frameworks
* dataclass-like libraries

But introspection should not become a habit of over-control.

Sometimes the cleanest code simply calls the expected operation and lets Python dispatch.

Example:

```python
def write_message(stream, message):
    stream.write(message)
```

This is often better than checking many possible stream types.

Introspection is powerful.

Use it when it clarifies a boundary or framework mechanism.

Avoid it when duck typing is simpler.

---

# Classes, Instances, and Attribute Lookup

Runtime types matter because attribute lookup uses the class.

Example:

```python
class Greeter:
    def greet(self):
        return "hello"

obj = Greeter()
```

When you call:

```python
obj.greet()
```

Python looks at the instance and its class.

The method lives on the class object.

The instance's type participates in resolving that method.

This connects:

* classes and instances
* descriptors
* methods
* MRO
* inheritance
* properties
* dunder methods

The type system is not a separate feature from object behavior.

It is the structure that makes object behavior possible.

---

# Runtime Types and Dunder Protocols

Many operations are syntax over runtime type behavior.

Examples:

```python
len(x)       -> type(x).__len__(x)
x + y        -> type(x).__add__(x, y)
for item in x -> iteration protocol
with x       -> context manager protocol
x[key]       -> item access protocol
x()          -> callable protocol
```

The exact lookup rules have details, especially for special methods.

But the broad idea is:

```text
syntax asks the object's type how to behave
```

This is why adding a dunder method to a class changes what syntax its instances support.

Example:

```python
class Box:
    def __len__(self):
        return 3

box = Box()

print(len(box))
```

The runtime type `Box` defines `__len__`.

So `box` supports `len()`.

Types are behavior providers.

---

# Runtime Type System vs Static Type System

Python's runtime type system always exists.

Static type checking is optional tooling layered on top.

Runtime:

```text
objects have actual types while code runs
operations dispatch dynamically
errors happen if an operation is unsupported
```

Static checking:

```text
tools analyze code before execution
annotations describe expected types
errors can be reported before runtime
```

Example:

```python
def shout(text: str) -> str:
    return text.upper()
```

Runtime asks:

```text
does the object passed as text have an upper method?
```

Static checker asks:

```text
does the code appear to pass a str where str is expected?
```

Both are useful.

They answer different questions at different times.

---

# Why Static Type Checking Helps Dynamic Python

Static type checking helps because many bugs are predictable before runtime.

Example:

```python
def total(values: list[int]) -> int:
    return sum(values)

total(["a", "b"])
```

At runtime, this fails when `sum()` tries to add strings to an integer start value.

A static checker can warn earlier.

Static checking helps with:

* wrong argument types
* missing attributes
* wrong return values
* `None` handling
* unreachable branches
* refactoring
* documentation
* IDE support
* large-team coordination

But static type checking does not replace runtime validation at untrusted boundaries.

External input is still runtime data.

JSON does not become safe because your function has annotations.

---

# The Boundary Rule

Use runtime validation at boundaries.

Use static typing inside trusted code.

Boundary examples:

* HTTP request payload
* config file
* CSV import
* CLI argument
* environment variable
* message queue payload
* plugin input
* database record

At these boundaries, data comes from outside your trusted Python object graph.

Validate it.

Once validated, internal code can rely more on types, tests, and static checking.

This design avoids two extremes:

```text
checking everything everywhere
trusting unvalidated external data
```

Professional Python usually uses both runtime validation and static type checking, but at different layers.

---

# Common Mistakes

Do not say Python has no types.

Python has runtime types on every object.

Do not say variables have fixed runtime types.

Names can be rebound to objects of different types.

Do not use `type(x) is T` when `isinstance(x, T)` better expresses the contract.

Do not use `isinstance()` when exact type distinction matters.

Do not assume annotations enforce runtime behavior.

They are metadata unless something explicitly uses them.

Do not use `list[str]` directly with `isinstance()`.

Runtime generic validation requires explicit checking.

Do not confuse a type alias with a new runtime type.

An alias is another name.

Do not assume runtime-checkable protocols verify signatures deeply.

They mostly check attribute presence.

Do not overuse runtime type checks when duck typing would be clearer.

If the operation is the contract, calling the operation may be enough.

Do not skip validation for external data.

Static hints do not validate JSON, CSV, environment variables, or user input.

---

# Mental Checklist

When thinking about runtime types, ask:

* Which object is being checked?
* Does the type belong to the object or the name?
* Do I need exact type or subclass acceptance?
* Is this a nominal contract or a structural contract?
* Would duck typing be clearer?
* Is this runtime validation or static documentation?
* Are annotations being inspected by a tool or library?
* Is this data trusted internal data or external input?
* Do generic type arguments need runtime validation?
* Would an ABC or protocol express the contract better?

The goal is not to check types everywhere.

The goal is to understand what kind of contract your code needs.

---

# Exercises

1. Create several objects and print `type(obj)` and `obj.__class__`. Compare the results.

2. Define a class `Animal` and subclass `Dog`. Test `isinstance()` and `issubclass()` relationships.

3. Show a case where `type(x) is int` and `isinstance(x, int)` behave differently.

4. Write a function that accepts any object supporting `len()` without checking for `list`.

5. Write the same function with an EAFP style using `try` and `except TypeError`.

6. Use `collections.abc.Iterable` to validate an iterable input.

7. Define a `Protocol` for an object with a `close()` method.

8. Add `@runtime_checkable` to the protocol and test it with a file object.

9. Inspect `__annotations__` on a function with annotated parameters and return type.

10. Call an annotated function with the wrong runtime type and observe that Python does not enforce the annotation.

11. Try `isinstance(["a"], list[str])` and explain the result.

12. Create a type alias and compare it with creating a new class.

13. Use `NewType` and inspect what happens at runtime.

14. Choose a real API boundary and write runtime validation for its input.

---

# Summary

Python is dynamically typed, not typeless.

Names refer to objects.

Objects have runtime types.

An object's type determines supported operations and behavior.

`type(obj)` returns the object's type.

Classes are objects too.

Most classes are instances of `type`.

`isinstance()` checks whether an object is an instance of a class or subclass.

`issubclass()` checks class hierarchy relationships.

Exact type checks are sometimes useful, but `isinstance()` is usually better when subclasses should be accepted.

Nominal typing is based on declared class relationships.

Structural typing is based on required behavior or shape.

Duck typing focuses on whether an object supports the operation needed.

ABCs provide runtime-checkable behavior categories.

Protocols describe structural contracts for static type checkers and can be made runtime-checkable in a limited way.

Annotations are runtime metadata, but Python does not enforce them by default.

Type hints help static tools, IDEs, documentation, and readers.

Runtime validation is still necessary at untrusted boundaries.

Generic annotations such as `list[str]` are not deep runtime checks.

The deep lesson is:

```text
Python's flexibility comes from runtime object behavior, not from the absence of types
```

Once you understand that, static typing, protocols, ABCs, metaclasses, descriptors, and bytecode all feel less separate.

They are different layers over the same runtime object model.

---

# Preview of Chapter 69

Chapter 68 studied Python's runtime type system.

We connected dynamic typing to the object model:

* objects have types
* classes are runtime objects
* `type` creates and describes classes
* `isinstance()` and `issubclass()` ask runtime type questions
* ABCs and protocols express contracts in different ways
* annotations are metadata unless tools or libraries use them

Next we return to execution internals with bytecode.

Volume I introduced bytecode and the Python Virtual Machine.

Chapter 69 goes deeper.

We will study:

* code objects
* bytecode instructions
* frames
* local variable access
* globals and builtins
* function calls
* attribute access
* control flow
* exception tables
* how bytecode changes across Python versions
* why bytecode is an implementation detail but still useful to understand

The transition is:

```text
runtime types explain what objects are
bytecode internals explain how operations over those objects are executed
```

The next chapter brings us closer to CPython's execution engine.
