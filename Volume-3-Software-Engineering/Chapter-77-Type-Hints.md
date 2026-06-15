# Chapter 77 — Type Hints

Type hints let Python code describe expected shapes.

They can say:

```text
this function expects a string
this function returns an integer
this list contains users
this dictionary maps names to scores
this value may be None
this object only needs a send method
```

Python remains a dynamically typed language.

That does not change when you add type hints.

This function:

```python
def add(a: int, b: int) -> int:
    return a + b
```

still runs at runtime like ordinary Python.

If you call:

```python
add("hello", "world")
```

Python itself does not reject the call because of the annotations.

It returns:

```text
helloworld
```

unless some other tool or runtime validation layer intervenes.

This is the most important idea in the chapter:

```text
type hints are not automatic runtime enforcement
```

Type hints are communication.

They communicate with:

* human readers
* editors
* IDEs
* documentation tools
* static type checkers
* linters
* runtime libraries that choose to inspect annotations

Used well, type hints make Python code easier to understand, refactor, and maintain.

Used poorly, they add noise and false confidence.

This chapter teaches the language of type hints.

Chapter 78 will teach static type checking as a workflow.

---

# Why Type Hints Exist

Python's dynamic typing is powerful.

You can write:

```python
def double(value):
    return value * 2
```

This works for many values:

```python
double(3)
double("ha")
double([1, 2])
```

The same function can work with different types as long as those objects support the required operation.

That flexibility is part of Python's beauty.

But it also creates questions:

```text
what does this function expect?
what does it return?
can this be None?
is this list homogeneous?
what keys are in this dictionary?
does this function mutate its argument?
what methods must this object provide?
```

Without type hints, readers infer answers from:

* names
* docstrings
* tests
* implementation
* examples
* runtime failures

Type hints make some of those answers explicit.

They do not replace tests.

They do not replace documentation.

They do not replace good design.

They add another layer of clarity.

---

# Dynamic Typing and Static Typing

In dynamic typing, type behavior is checked as the program runs.

Example:

```python
value = "hello"
print(value.upper())
```

This works because at runtime `value` refers to a string object with an `upper` method.

If later:

```python
value = 123
print(value.upper())
```

Python raises:

```text
AttributeError
```

because an integer does not have `upper`.

Static typing tries to detect some type problems before the program runs.

In Python, type hints provide information that static tools can analyze.

The runtime remains dynamic.

The type checker is an additional tool.

The relationship is:

```text
Python executes dynamically
type checkers analyze annotations statically
```

That combination is called gradual typing.

You can add types gradually instead of all at once.

---

# Gradual Typing

Gradual typing means typed and untyped code can coexist.

This is important for Python.

Existing projects may have thousands of lines without annotations.

You do not need to stop everything and annotate the whole codebase.

You can start with:

* public functions
* important domain models
* risky boundaries
* newly written code
* code that often breaks
* library APIs
* complex data transformations

Untyped code remains valid Python.

Typed code adds clarity where it helps.

For example:

```python
def normalize_email(email: str) -> str:
    return email.strip().lower()
```

This function is now clearer to readers and tools.

Gradual typing lets teams improve code over time.

That is why type hints fit Python's culture better than a forced all-or-nothing system.

---

# The Basic Syntax

Function parameters are annotated after the parameter name:

```python
def greet(name: str):
    return f"Hello, {name}"
```

Return types are annotated after `->`:

```python
def greet(name: str) -> str:
    return f"Hello, {name}"
```

Variables can be annotated:

```python
age: int = 30
name: str = "Narendra"
active: bool = True
```

A variable can be annotated before assignment:

```python
result: int

if condition:
    result = 1
else:
    result = 2
```

The annotation says what type the variable is expected to hold.

It does not create the value.

If you write:

```python
count: int
print(count)
```

you still get a runtime error because `count` was never assigned.

Annotations describe.

They do not initialize.

---

# Simple Built-In Types

Common simple annotations use ordinary type names:

```python
def repeat(text: str, times: int) -> str:
    return text * times
```

Examples:

```python
name: str = "Asha"
age: int = 32
price: float = 99.5
active: bool = True
data: bytes = b"hello"
```

Use the type that matches the domain.

For money, `float` is often the wrong type.

You might use:

```python
from decimal import Decimal


price: Decimal = Decimal("99.50")
```

Type hints should express design, not just syntax.

If the domain cares about exact decimal values, annotate exact decimal values.

---

# Return Type None

Functions that return nothing explicitly should be annotated with `None`.

Example:

```python
def log_message(message: str) -> None:
    print(message)
```

This means:

```text
this function is called for its effect, not for a useful return value
```

It still returns `None` at runtime because every Python function returns something.

If no return value is specified, Python returns `None`.

This is especially helpful for methods:

```python
class Cart:
    def add(self, item: str) -> None:
        ...
```

The annotation tells readers:

```text
call this method to mutate the cart
do not expect a returned cart
```

---

# Collection Types

Modern Python allows built-in collection types to be parameterized.

Examples:

```python
names: list[str] = ["Asha", "Ravi"]
scores: dict[str, int] = {"Asha": 95, "Ravi": 88}
tags: set[str] = {"python", "testing"}
point: tuple[int, int] = (10, 20)
```

The part inside brackets describes contained types.

`list[str]` means:

```text
a list whose items are strings
```

`dict[str, int]` means:

```text
a dictionary whose keys are strings and values are integers
```

This makes data shape visible.

Compare:

```python
def average(scores):
    ...
```

with:

```python
def average(scores: list[int]) -> float:
    ...
```

The second version communicates much more.

---

# Tuple Types

Tuples can be fixed-length or variable-length.

Fixed-length tuple:

```python
point: tuple[int, int] = (10, 20)
```

This means exactly two items:

```text
first item is int
second item is int
```

Mixed tuple:

```python
user_row: tuple[int, str, bool] = (1, "asha@example.com", True)
```

Variable-length tuple:

```python
values: tuple[int, ...] = (1, 2, 3, 4)
```

This means:

```text
zero or more integers
```

Do not use tuple types when a named structure would be clearer.

This:

```python
tuple[int, str, bool]
```

may be less readable than a dataclass:

```python
@dataclass
class User:
    id: int
    email: str
    active: bool
```

Type hints should improve clarity.

---

# list vs Sequence

Sometimes a function does not need a list specifically.

It only needs something it can iterate over or index.

Example:

```python
def first_name(names: list[str]) -> str:
    return names[0]
```

This annotation says callers should pass a list.

But the function also works with a tuple:

```python
first_name(("Asha", "Ravi"))
```

If you want to accept any sequence, use `Sequence`:

```python
from collections.abc import Sequence


def first_name(names: Sequence[str]) -> str:
    return names[0]
```

Use concrete types when you need concrete behavior.

Use abstract collection types when you only need an interface.

This is type-hinting version of duck typing.

---

# Iterable

If a function only loops over values, it may accept `Iterable`.

Example:

```python
from collections.abc import Iterable


def total(values: Iterable[int]) -> int:
    result = 0
    for value in values:
        result += value
    return result
```

This accepts:

* list
* tuple
* set
* generator
* custom iterable

That is more flexible than:

```python
def total(values: list[int]) -> int:
    ...
```

But flexibility has tradeoffs.

An iterable may be consumed only once.

If your function needs length or indexing, `Iterable` is too weak.

Choose the annotation based on what the function actually needs.

---

# Mapping

If a function reads dictionary-like data but does not mutate it, use `Mapping`.

Example:

```python
from collections.abc import Mapping


def format_user(user: Mapping[str, str]) -> str:
    return f"{user['name']} <{user['email']}>"
```

If a function mutates the dictionary, use `MutableMapping`:

```python
from collections.abc import MutableMapping


def normalize_user(user: MutableMapping[str, str]) -> None:
    user["email"] = user["email"].strip().lower()
```

Type hints can express mutability expectations.

That matters because mutation is part of a function's contract.

---

# Optional Values

Some values may be absent.

In Python, absence is often represented by `None`.

Modern union syntax:

```python
def find_user(email: str) -> User | None:
    ...
```

This means:

```text
the function returns a User or None
```

Older spelling:

```python
from typing import Optional


def find_user(email: str) -> Optional[User]:
    ...
```

Both express the same idea.

The `User | None` spelling is now common in modern Python.

When you see `None` in a type, the caller must handle absence.

Example:

```python
user = find_user("a@example.com")

if user is None:
    raise LookupError("user not found")

send_email(user.email)
```

Type hints make absence visible.

That helps prevent `NoneType` errors.

---

# Union Types

A union means a value may be one of several types.

Example:

```python
def parse_id(value: int | str) -> int:
    return int(value)
```

This accepts either an integer or a string.

Use unions when the domain genuinely accepts multiple shapes.

Do not use unions to avoid modeling data clearly.

This may be a smell:

```python
def process(value: str | int | dict[str, object] | list[object]) -> None:
    ...
```

If a function accepts many unrelated types, ask whether it does too much.

Type hints can reveal design confusion.

---

# Any

`Any` means the type checker should allow anything.

Example:

```python
from typing import Any


def load_json(text: str) -> Any:
    ...
```

`Any` is sometimes necessary.

It is common at dynamic boundaries:

* JSON
* untyped third-party libraries
* plugin systems
* dynamic imports
* framework magic
* gradual migration from untyped code

But `Any` is also an escape hatch.

Once a value is `Any`, type checkers can no longer help much with it.

This may pass static checks:

```python
value: Any = get_value()
value.this.does.not.exist()
```

Use `Any` deliberately.

When possible, convert unknown data into known shapes at boundaries.

---

# object

`object` is different from `Any`.

Every Python value is an object.

So:

```python
def accept_anything(value: object) -> None:
    ...
```

can accept anything.

But inside the function, you cannot assume much about `value`.

This is safe:

```python
def display(value: object) -> str:
    return str(value)
```

This is not safe:

```python
def uppercase(value: object) -> str:
    return value.upper()
```

The type checker should complain because not every object has `upper`.

Use `object` when you truly accept any object but will treat it generically.

Use `Any` when you intentionally opt out of checking.

They are not the same.

---

# Type Aliases

A type alias gives a name to a type expression.

Example:

```python
UserId = int
Email = str
```

Now:

```python
def send_email(user_id: UserId, email: Email) -> None:
    ...
```

This improves readability.

For complex types, aliases help even more:

```python
JsonObject = dict[str, object]
Headers = dict[str, str]
```

Then:

```python
def request(url: str, headers: Headers) -> JsonObject:
    ...
```

Use aliases to name concepts.

Do not create aliases for every simple type unless it helps the domain.

---

# NewType

`NewType` creates a distinct static type with the same runtime representation.

Example:

```python
from typing import NewType


UserId = NewType("UserId", int)
OrderId = NewType("OrderId", int)
```

Now a type checker can distinguish:

```python
def get_user(user_id: UserId) -> User:
    ...
```

Calling with an `OrderId` can be flagged statically.

At runtime, `UserId(123)` is essentially still an integer-like value.

`NewType` helps prevent mixing values that share representation but differ in meaning.

Examples:

* user ID vs order ID
* meters vs seconds
* customer ID vs account ID
* internal ID vs external ID

Use it for domain safety.

---

# Literal

`Literal` restricts a value to specific literal choices.

Example:

```python
from typing import Literal


LogLevel = Literal["debug", "info", "warning", "error"]


def set_log_level(level: LogLevel) -> None:
    ...
```

Now the intended values are visible:

```python
set_log_level("info")
set_log_level("verbose")
```

A type checker can reject `"verbose"` if it is not allowed.

`Literal` is useful for:

* modes
* small command sets
* configuration flags
* fixed string protocols
* state labels

If choices grow large or need behavior, consider `Enum`.

---

# Final

`Final` marks a name as not intended to be reassigned.

Example:

```python
from typing import Final


MAX_RETRIES: Final = 3
```

A type checker can flag:

```python
MAX_RETRIES = 5
```

At runtime, Python still allows reassignment unless something else prevents it.

`Final` communicates intent to tools and readers.

It is useful for constants and API values that should not change.

---

# ClassVar

`ClassVar` marks a class variable, not an instance field.

Example:

```python
from typing import ClassVar


class User:
    table_name: ClassVar[str] = "users"

    def __init__(self, email: str):
        self.email = email
```

This tells type checkers and dataclass-like tools:

```text
table_name belongs to the class
email belongs to the instance
```

This matters especially with dataclasses:

```python
from dataclasses import dataclass
from typing import ClassVar


@dataclass
class User:
    table_name: ClassVar[str] = "users"
    email: str
```

`table_name` is not treated as a dataclass field.

---

# Callable

`Callable` describes a function-like object.

Example:

```python
from collections.abc import Callable


def apply(value: int, func: Callable[[int], str]) -> str:
    return func(value)
```

This means:

```text
func accepts one int and returns str
```

Example use:

```python
def format_number(value: int) -> str:
    return f"#{value}"


apply(10, format_number)
```

For simple callbacks, `Callable` is enough.

For callbacks with keyword-only parameters, overloads, attributes, or more complex behavior, a protocol may be clearer.

---

# Protocols

Protocols express structural typing.

They say:

```text
an object is acceptable if it has these members
```

Example:

```python
from typing import Protocol


class EmailSender(Protocol):
    def send(self, to: str, subject: str, body: str) -> None:
        ...
```

Now:

```python
def send_welcome_email(sender: EmailSender, email: str) -> None:
    sender.send(email, "Welcome", "Hello")
```

Any object with a compatible `send` method can be accepted by a static type checker.

It does not need to inherit from `EmailSender`.

This matches Python's duck typing philosophy:

```text
if it behaves like the needed thing, it can be used
```

Protocols are type-hinted duck typing.

---

# Protocols vs ABCs

An abstract base class defines an inheritance relationship.

A protocol defines a structural expectation.

ABC style:

```python
from abc import ABC, abstractmethod


class EmailSender(ABC):
    @abstractmethod
    def send(self, to: str, subject: str, body: str) -> None:
        ...
```

Implementations inherit:

```python
class SmtpEmailSender(EmailSender):
    def send(self, to: str, subject: str, body: str) -> None:
        ...
```

Protocol style:

```python
class EmailSender(Protocol):
    def send(self, to: str, subject: str, body: str) -> None:
        ...
```

Implementations do not have to inherit:

```python
class SmtpEmailSender:
    def send(self, to: str, subject: str, body: str) -> None:
        ...
```

Use ABCs when inheritance and runtime abstraction matter.

Use protocols when structural capability matters.

---

# runtime_checkable

Protocols are mainly for static checking.

If you want limited runtime `isinstance` support, use `runtime_checkable`.

Example:

```python
from typing import Protocol, runtime_checkable


@runtime_checkable
class HasClose(Protocol):
    def close(self) -> None:
        ...
```

Then:

```python
if isinstance(resource, HasClose):
    resource.close()
```

Runtime protocol checks are limited.

They check presence of members, not full type signatures.

Do not confuse runtime protocol checks with full static type analysis.

They are useful in narrow cases.

---

# TypedDict

`TypedDict` describes dictionaries with specific keys and value types.

Example:

```python
from typing import TypedDict


class UserPayload(TypedDict):
    id: int
    email: str
    active: bool
```

Now:

```python
def format_user(user: UserPayload) -> str:
    return f"{user['email']} ({user['id']})"
```

This is useful for dictionary-shaped data:

* JSON payloads
* API responses
* configuration dictionaries
* parsed records
* event payloads

At runtime, a `TypedDict` value is still a plain dictionary.

The type information helps static tools and readers.

If the data has behavior, consider a dataclass or class instead.

---

# Required and Optional Keys

Some dictionary keys may be optional.

Modern typing supports required and non-required keys.

Example:

```python
from typing import NotRequired, TypedDict


class UserPayload(TypedDict):
    id: int
    email: str
    display_name: NotRequired[str]
```

Here `id` and `email` are required.

`display_name` may be absent.

This is different from:

```python
display_name: str | None
```

That means the key exists, but the value may be `None`.

Distinguish:

```text
missing key
present key with None value
```

They are different data shapes.

---

# NamedTuple

`NamedTuple` gives tuple-like data named fields.

Example:

```python
from typing import NamedTuple


class Point(NamedTuple):
    x: int
    y: int
```

Use:

```python
point = Point(10, 20)
print(point.x)
```

Named tuples are immutable and tuple-compatible.

They can be useful for lightweight records.

For richer models, dataclasses are often clearer.

The choice depends on whether tuple behavior matters.

---

# Dataclasses and Type Hints

Dataclasses use type annotations to define fields.

Example:

```python
from dataclasses import dataclass


@dataclass
class User:
    id: int
    email: str
    active: bool = True
```

The annotations tell `dataclass` what fields to generate.

This is one of the places where annotations do affect runtime behavior indirectly because the `dataclass` decorator reads them.

But Python itself is not enforcing the types.

This still runs unless you add validation:

```python
user = User(id="not an int", email=123)
```

Dataclasses use annotations structurally.

Type checkers use them semantically.

Runtime validation is separate.

---

# Generic Functions

Generics let a function preserve relationships between input and output types.

Example:

```python
def first[T](items: list[T]) -> T:
    return items[0]
```

This means:

```text
if items is list[int], return int
if items is list[str], return str
```

Older spelling uses `TypeVar`:

```python
from typing import TypeVar


T = TypeVar("T")


def first(items: list[T]) -> T:
    return items[0]
```

Generics are useful when types are connected.

Without generics, you might write:

```python
def first(items: list[object]) -> object:
    return items[0]
```

That loses information.

Generics preserve it.

---

# Generic Classes

Classes can be generic too.

Example:

```python
class Box[T]:
    def __init__(self, value: T):
        self.value = value

    def get(self) -> T:
        return self.value
```

Use:

```python
int_box = Box(123)
str_box = Box("hello")
```

Older spelling:

```python
from typing import Generic, TypeVar


T = TypeVar("T")


class Box(Generic[T]):
    def __init__(self, value: T):
        self.value = value

    def get(self) -> T:
        return self.value
```

Generic classes are common in:

* repositories
* containers
* result types
* API clients
* caches
* parsers

Use them when the class stores or returns values whose type should be preserved.

---

# Self

`Self` represents the current class type.

It is useful for methods that return `self`.

Example:

```python
from typing import Self


class Query:
    def filter(self, expression: str) -> Self:
        self.filters.append(expression)
        return self
```

This helps fluent APIs:

```python
query.filter("active = true").filter("role = 'admin'")
```

`Self` is better than hard-coding the class name when subclasses should preserve their type.

For example, if `AdminQuery` inherits `Query`, methods returning `Self` can be understood as returning `AdminQuery` when called on `AdminQuery`.

---

# Type of a Class

Sometimes a function accepts a class, not an instance.

Use `type[T]`.

Example:

```python
def create_user[T](cls: type[T]) -> T:
    return cls()
```

Simpler example:

```python
def make_exception(cls: type[Exception], message: str) -> Exception:
    return cls(message)
```

This means `cls` should be an exception class, not an exception instance.

The distinction matters:

```python
ValueError
```

is a class.

```python
ValueError("bad")
```

is an instance.

Type hints can express that difference.

---

# Overload

`overload` describes functions whose return type depends on input types.

Example:

```python
from typing import overload


@overload
def parse(value: int) -> int:
    ...


@overload
def parse(value: str) -> str:
    ...


def parse(value: int | str) -> int | str:
    return value
```

The overload definitions are for type checkers.

The final implementation is what runs.

Overloads are useful for APIs where a simple union loses information.

But overloads add complexity.

Use them when they clarify real caller behavior.

Do not use them to make an already confusing API look precise.

---

# Annotated

`Annotated` attaches metadata to a type.

Example:

```python
from typing import Annotated


UserId = Annotated[int, "database primary key"]
```

The base type is still `int`.

The metadata may be used by frameworks, validation libraries, documentation tools, or custom code.

Another example:

```python
PositiveInt = Annotated[int, "must be greater than zero"]
```

By itself, this does not enforce positivity.

It describes extra information.

Runtime libraries may choose to inspect and enforce metadata.

Again, annotation is not automatic validation.

---

# cast

`cast` tells a type checker to treat a value as a certain type.

Example:

```python
from typing import cast


value = load_from_cache("user")
user = cast(User, value)
```

At runtime, `cast` returns the value unchanged.

It does not check the type.

This is important.

`cast` is not conversion.

It is a statement to the type checker:

```text
trust me, I know this value is a User
```

Use it sparingly.

Too many casts may mean the code is not modeling types clearly.

---

# TYPE_CHECKING

`TYPE_CHECKING` is a constant that is `False` at runtime but treated as `True` by type checkers.

Example:

```python
from typing import TYPE_CHECKING


if TYPE_CHECKING:
    from expensive_module import ExpensiveType
```

This is useful to avoid runtime imports needed only for annotations.

It can help with:

* circular imports
* expensive imports
* optional dependencies
* type-only references

Example:

```python
if TYPE_CHECKING:
    from .models import User


def send_email(user: "User") -> None:
    ...
```

Modern Python has better annotation handling than older versions, but type-only imports still matter in real projects.

---

# Forward References

Sometimes a type is referenced before it is defined.

Example:

```python
class Node:
    def __init__(self, parent: "Node | None" = None):
        self.parent = parent
```

The quotes create a forward reference.

They delay evaluation of the annotation.

Forward references are useful for:

* recursive data structures
* circular type references
* classes that refer to themselves

Modern Python versions continue to evolve how annotations are evaluated.

When writing library code, be aware that runtime annotation behavior can differ across Python versions.

If you inspect annotations at runtime, test that behavior on the Python versions you support.

---

# Runtime Access to Annotations

Annotations are stored in `__annotations__`.

Example:

```python
def add(a: int, b: int) -> int:
    return a + b


print(add.__annotations__)
```

You may see:

```python
{"a": int, "b": int, "return": int}
```

Classes and modules can also have annotations.

Libraries can inspect annotations to build:

* validators
* serializers
* API schemas
* command-line parsers
* dependency injection systems
* documentation

But this is library behavior.

Python itself does not enforce the annotated types for ordinary functions.

---

# Runtime Validation

Type hints and runtime validation are different.

Type hint:

```python
def create_user(email: str) -> User:
    ...
```

Runtime validation:

```python
def create_user(email: str) -> User:
    if not isinstance(email, str):
        raise TypeError("email must be a string")
    ...
```

Many frameworks use annotations for validation.

That is useful.

But it is not automatic Python behavior.

If your system needs runtime validation, choose and design that explicitly.

Do not assume annotations protect runtime boundaries.

External input should still be validated.

Examples:

* HTTP JSON
* CLI arguments
* environment variables
* files
* database rows
* messages from queues

Type hints help inside the program.

Validation protects boundaries.

---

# Type Hints and Documentation

Type hints are executable-looking documentation.

Compare:

```python
def send_email(to, subject, body):
    ...
```

with:

```python
def send_email(to: str, subject: str, body: str) -> None:
    ...
```

The second function answers questions immediately.

Type hints also make documentation tools more useful.

They can show:

* parameter types
* return types
* class attributes
* generic parameters
* optional values

However, type hints are not a substitute for explaining behavior.

This tells shape:

```python
def refund(order_id: OrderId, amount: Decimal) -> Refund:
    ...
```

It does not explain policy:

```text
refunds are allowed only for captured payments within 30 days
```

Use docstrings for behavior that types cannot express clearly.

---

# Type Hints and Tests

Type hints do not replace tests.

They catch different problems.

Type hints can help detect:

```python
send_email(user_id=123)
```

when `send_email` expects an email string.

Tests can verify:

```text
the email body contains the reset link
the token expires after one hour
the function retries after timeout
the database row is created
```

Types describe shapes.

Tests verify behavior.

Good software uses both.

---

# Type Hints and Refactoring

Type hints make refactoring safer.

If you change a function signature:

```python
def calculate_total(items: list[Item], currency: str) -> Money:
    ...
```

tools can help find callers that still use the old shape.

If you rename a field on a dataclass, type-aware editors can often update usages.

If you change a return type, type checkers can reveal places that assumed the old type.

Type hints create a map of expectations.

Refactoring changes the map.

Static analysis helps find mismatches before runtime.

---

# Type Hints in Public APIs

Public APIs benefit strongly from type hints.

For a library function:

```python
def parse_invoice(data: bytes) -> Invoice:
    ...
```

users can immediately see:

* input must be bytes
* output is an Invoice

For a class:

```python
class Client:
    def get_user(self, user_id: UserId) -> User:
        ...
```

the API is easier to explore in editors.

If your package is distributed, type hints become part of the user experience.

Chapter 76 mentioned `py.typed`.

If a package wants to expose inline type hints to type checkers, packaging must include that typing information.

---

# Type Hints in Private Code

Private code can also benefit from hints, but the tradeoff is different.

You may not need to annotate every tiny helper.

Type inference can often understand local variables.

Useful private annotations include:

* complex function signatures
* values initialized later
* empty containers
* important domain objects
* callbacks
* dictionaries with known shape
* functions that return optional values

Example:

```python
users_by_email: dict[str, User] = {}
```

Without the annotation, an empty dictionary has no obvious intended key and value types.

Annotate where it helps humans or tools.

Do not annotate mechanically when it adds no information.

---

# Empty Collections

Empty collections often need annotations.

Example:

```python
users = []
```

What kind of users?

Strings?

User objects?

Dictionaries?

Better:

```python
users: list[User] = []
```

Similarly:

```python
scores: dict[str, int] = {}
seen_ids: set[UserId] = set()
```

The annotation tells the reader and type checker what will be stored later.

This is one of the most practical uses of variable annotations.

---

# Avoid Redundant Noise

This annotation is usually redundant:

```python
name: str = "Asha"
```

The value already makes the type obvious.

Sometimes redundancy is acceptable for consistency or emphasis.

But avoid turning code into clutter:

```python
count: int = 1
enabled: bool = True
message: str = "ok"
```

inside a tiny local scope may not help.

Better places to annotate:

* function boundaries
* class attributes
* empty collections
* complex values
* values from dynamic sources
* public APIs

Type hints should increase signal.

They should not become wallpaper.

---

# Bad Type Hints

Bad type hints are worse than no type hints because they mislead readers.

Example:

```python
def find_user(email: str) -> User:
    return database.get(email)
```

If `database.get` can return `None`, the annotation is wrong.

It should be:

```python
def find_user(email: str) -> User | None:
    return database.get(email)
```

Or the function should raise when missing:

```python
def find_user(email: str) -> User:
    user = database.get(email)
    if user is None:
        raise LookupError(email)
    return user
```

Types should match behavior.

Do not write the type you wish were true.

Write the type your function actually promises.

---

# Type Narrowing

Type narrowing happens when code checks a value and tools infer a more specific type.

Example:

```python
def send(user: User | None) -> None:
    if user is None:
        return

    reveal = user.email
```

After the `None` check, `user` is known to be `User`.

Other narrowing checks include:

```python
isinstance(value, str)
value is not None
key in dictionary
```

Example:

```python
def normalize(value: str | int) -> str:
    if isinstance(value, int):
        return str(value)
    return value.strip().lower()
```

In the `else` path, `value` is understood as `str`.

Type narrowing is how type checkers follow ordinary Python control flow.

---

# assert for Narrowing

Assertions can narrow types.

Example:

```python
def process(user: User | None) -> None:
    assert user is not None
    send_email(user.email)
```

After the assertion, `user` is treated as `User`.

But remember:

Python can remove assertions with optimization mode.

Do not use `assert` for required runtime validation at external boundaries.

For internal invariants, it can be fine.

For user input, raise explicit exceptions.

---

# TypeGuard and TypeIs

Sometimes a helper function narrows a type.

Typing provides tools for expressing that.

Example concept:

```python
from typing import TypeGuard


def is_str_list(value: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(item, str) for item in value)
```

If this function returns `True`, a type checker can treat the value as `list[str]`.

Newer typing features also include `TypeIs` for specific narrowing behavior.

These tools are useful for validation helpers and dynamic data.

Use them when ordinary `isinstance` checks are not expressive enough.

---

# Stub Files

Stub files contain type information without implementation.

They use the `.pyi` extension.

Example:

```python
# library.pyi
def parse(text: str) -> dict[str, object]: ...
```

Stubs are useful when:

* a package is implemented in C
* a package is dynamically generated
* adding annotations to source is not practical
* external type information is maintained separately

For most modern Python code, inline annotations are simpler.

But stub files remain important in the ecosystem.

They help type checkers understand code that cannot easily annotate itself.

---

# Type Comments

Before modern annotation syntax, code sometimes used type comments:

```python
count = 0  # type: int
```

Function type comments also existed.

Modern code should usually prefer annotations:

```python
count: int = 0
```

You may still encounter type comments in older codebases or compatibility-focused projects.

Be able to read them.

Do not introduce them unless you have a compatibility reason.

---

# Type Hints and Imports

Type hints can create import problems.

Example:

```python
from orders import Order
from users import User
```

If `orders` also imports `users`, a circular import may appear.

Solutions include:

* restructure modules
* use `TYPE_CHECKING`
* use forward references
* move shared types to another module
* reduce runtime import coupling

Type hints should not force bad architecture.

If annotations create circular imports, the design may need clearer boundaries.

---

# Type Hints and Performance

Ordinary type hints usually have little runtime cost.

But annotations may affect:

* import time
* runtime introspection
* frameworks that inspect annotations
* large complex type expressions
* circular imports

If annotations import heavy modules at runtime, startup can slow down.

Use type-only imports when needed:

```python
from typing import TYPE_CHECKING


if TYPE_CHECKING:
    from heavy_module import HeavyType
```

Do not prematurely optimize annotations.

But be aware that annotations are still part of Python code and import behavior.

---

# Type Hints and Frameworks

Many frameworks inspect annotations.

Examples:

* data validation libraries
* web frameworks
* dependency injection systems
* serializers
* ORMs
* CLI frameworks

In such frameworks, annotations may influence runtime behavior.

Example concept:

```python
def endpoint(limit: int = 10) -> list[User]:
    ...
```

A framework may use `int` to parse a query parameter.

This is framework behavior.

It does not mean Python generally enforces annotations.

When using a framework, learn that framework's annotation semantics.

The same annotation can be documentation in one context and runtime configuration in another.

---

# Type Hints and Dataclass Defaults

Type hints do not fix mutable default problems.

Bad:

```python
from dataclasses import dataclass


@dataclass
class Cart:
    items: list[str] = []
```

This is wrong because the list would be shared.

Dataclasses reject some mutable defaults, but the deeper lesson matters.

Use:

```python
from dataclasses import dataclass, field


@dataclass
class Cart:
    items: list[str] = field(default_factory=list)
```

Type hints describe the field type.

They do not replace correct object lifecycle design.

---

# Type Hints and Exceptions

Python type hints do not generally annotate raised exceptions in function signatures.

You do not write:

```python
def read_file(path: str) -> str raises FileNotFoundError:
    ...
```

Some languages have checked exceptions.

Python does not.

Document important exceptions in docstrings.

Test important error behavior.

Use meaningful exception types.

Type hints describe values.

They do not fully describe all control flow.

---

# Type Hints and Async

Async functions are annotated by their awaited result.

Example:

```python
async def fetch_user(user_id: str) -> User:
    ...
```

When called, this function returns a coroutine object.

When awaited, it produces a `User`.

So the annotation means:

```text
await fetch_user(...) gives User
```

Async generators can be annotated with async collection interfaces:

```python
from collections.abc import AsyncIterator


async def stream_users() -> AsyncIterator[User]:
    ...
```

Async typing is another place where runtime behavior and annotation meaning must be understood carefully.

---

# Type Hints and Generators

For simple generators, use `Iterator` or `Iterable`.

Example:

```python
from collections.abc import Iterator


def count_up_to(limit: int) -> Iterator[int]:
    current = 0
    while current < limit:
        yield current
        current += 1
```

For advanced generator behavior involving `send` and return values, `typing.Generator` can express yield type, send type, and return type.

Most application code does not need that complexity.

Use the simplest annotation that accurately communicates behavior.

---

# Public and Private Symbols

Type hints help define public APIs.

If a module exports:

```python
__all__ = ["Client", "RequestError"]
```

then those names form part of the public surface.

Public functions and classes deserve careful annotations.

Private helpers may be annotated where useful.

For libraries, changing public annotations can affect users.

If users rely on type checking, annotations are part of the developer experience.

Treat public type hints with the same care as public function names.

---

# Distributing Type Information

For a typed package, include type information when packaging.

Inline typed packages often include:

```text
py.typed
```

inside the import package.

Example:

```text
src/
    greetings/
        __init__.py
        py.typed
```

The marker tells type checkers that inline annotations should be used.

If you publish a package with type hints but omit the marker, users' type checkers may not treat it as typed.

This is where Chapter 76 and Chapter 77 meet.

Packaging carries the type information to users.

---

# Good Type Hint Style

Good type hints are:

* accurate
* readable
* useful
* as general as the function needs
* as specific as the contract promises
* consistent with runtime behavior
* helpful at boundaries

Example:

```python
from collections.abc import Iterable


def total(values: Iterable[int]) -> int:
    return sum(values)
```

This is good because the function only needs iteration.

Example:

```python
def get_user(email: str) -> User | None:
    ...
```

This is good because absence is explicit.

Example:

```python
def send(sender: EmailSender, message: Message) -> None:
    ...
```

This is good because the boundary is named.

---

# Common Type Hint Mistakes

Common mistakes include:

* assuming hints enforce runtime behavior
* using `Any` everywhere
* forgetting `None` in return types
* annotating everything mechanically
* using concrete collections when abstract ones fit better
* using abstract collections when mutation is required
* writing inaccurate return types
* hiding unclear design behind huge unions
* importing heavy modules only for annotations
* confusing `object` and `Any`
* using `cast` to silence real problems
* forgetting `py.typed` for distributed typed packages
* treating type hints as a replacement for validation
* treating type hints as a replacement for tests

Most mistakes come from forgetting the role of type hints.

They describe expectations.

They do not magically make the expectations true.

---

# A Type Hinting Checklist

When adding type hints, ask:

* Does this annotation match runtime behavior?
* Does this function ever return `None`?
* Is this collection mutable?
* Do I need a concrete type or an abstract interface?
* Is `Any` truly necessary?
* Would a protocol express the dependency better?
* Would a dataclass be clearer than a dictionary?
* Is this public API annotation stable?
* Will this import cause runtime or circular import problems?
* Does this package need to distribute `py.typed`?
* Is runtime validation needed separately?

This checklist keeps type hints honest.

Honest types are useful.

Decorative types are a liability.

---

# Chapter Summary

Type hints describe expected shapes in Python code.

They can annotate function parameters, return values, variables, collections, callbacks, class attributes, dictionaries, protocols, and generic relationships.

Python remains dynamically typed.

Annotations do not automatically enforce runtime behavior.

Type hints communicate with humans, editors, static type checkers, documentation tools, and frameworks that choose to inspect them.

Gradual typing lets typed and untyped Python coexist.

Use basic annotations for simple values.

Use collection annotations such as `list[str]`, `dict[str, int]`, `tuple[int, int]`, `Sequence[T]`, `Iterable[T]`, and `Mapping[K, V]` to express container shapes.

Use `T | None` when absence is possible.

Use unions when a value genuinely may have multiple types.

Use `Any` carefully because it disables much checking.

Use `object` when any value is accepted but only generic object behavior is allowed.

Use type aliases and `NewType` to name domain concepts.

Use `Literal` for fixed choices.

Use `Final` and `ClassVar` to communicate special variable roles.

Use `Callable` for simple callbacks.

Use protocols to express duck-typed interfaces.

Use `TypedDict` for dictionary-shaped data with known keys.

Use generics to preserve relationships between input and output types.

Use `Self` for methods that return the current instance type.

Use overloads only when they clarify real API behavior.

Use `TYPE_CHECKING` and forward references to manage type-only imports and circular references.

Runtime validation is separate from type hints.

Tests are separate from type hints.

Packaging must include type information when distributing typed libraries.

The central lesson is:

```text
type hints make expectations visible
```

Visible expectations make code easier to read, use, refactor, and check.

---

# Exercises

1. Add parameter and return type hints to five small functions.

2. Annotate an empty list, empty dictionary, and empty set with intended element types.

3. Rewrite a function returning `None` sometimes so its return type says `T | None`.

4. Replace a concrete `list` parameter with `Sequence` or `Iterable` where appropriate.

5. Create a `TypedDict` for an API response dictionary.

6. Create a `Protocol` for an email sender and write two classes that satisfy it without inheriting from it.

7. Use `NewType` to distinguish `UserId` and `OrderId`.

8. Write a generic `first` function that preserves item type.

9. Add `Self` to a fluent class method that returns `self`.

10. Find one inaccurate type hint in existing code and correct the behavior or the annotation.

---

# Preview of Chapter 78

Chapter 77 introduced type hints as a language for making expectations visible.

We studied basic annotations, collections, optional values, unions, `Any`, `object`, type aliases, `NewType`, `Literal`, `Final`, `ClassVar`, `Callable`, protocols, `TypedDict`, generics, `Self`, overloads, `Annotated`, `cast`, `TYPE_CHECKING`, forward references, and the boundary between annotations and runtime behavior.

Next we study static type checking.

Type hints by themselves are only information.

Static type checkers turn that information into feedback.

They can report:

* incompatible argument types
* missing `None` handling
* impossible attribute access
* incorrect return values
* mismatched generic types
* untyped boundaries
* invalid overrides
* unreachable branches

Static type checking is where annotations become part of the engineering workflow.

The transition is:

```text
type hints express expectations
static type checking verifies those expectations before runtime
```

Chapter 78 will show how to use static type checking gradually, practically, and without turning Python into a language it is not.
