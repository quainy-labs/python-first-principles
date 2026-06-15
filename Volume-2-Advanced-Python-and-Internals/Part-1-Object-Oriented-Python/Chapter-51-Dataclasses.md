# Chapter 51 — Dataclasses

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a dataclass is.
* Why dataclasses exist.
* How `@dataclass` changes a class.
* Why dataclasses depend on type annotations.
* Which methods Python can generate automatically.
* How dataclass fields are discovered.
* How defaults and `default_factory` work.
* Why mutable defaults are dangerous.
* How `field()` gives you per-field control.
* How `__post_init__` supports validation and derived state.
* How frozen dataclasses model value objects.
* How equality, ordering, and hashing work.
* How dataclasses interact with inheritance.
* How `ClassVar` and `InitVar` change field behavior.
* How `asdict`, `astuple`, and `replace` should be used.
* How dataclasses compare with normal classes, tuples, dictionaries, and named tuples.
* When dataclasses are a good fit.
* When dataclasses are the wrong tool.

Chapter 50 studied ABCs and mixins.

Those tools help us express behavior contracts and reusable behavior.

Dataclasses solve a different problem.

Many classes are not mainly about behavior.

They are mainly about structured data.

For example:

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

This class is not complicated.

But it is repetitive.

The important idea is:

```text
a point has x and y
```

The code spends most of its space saying:

```text
copy constructor arguments into attributes
print the attributes
compare the attributes
```

Dataclasses let you write the idea directly:

```python
from dataclasses import dataclass


@dataclass
class Point:
    x: int
    y: int
```

That is still a real class.

It still has instances.

It still has attributes.

It can still have methods.

But Python generates common object methods for you.

Dataclasses are not magic replacements for object-oriented design.

They are a tool for classes whose shape is mostly data.

---

# The Problem Dataclasses Solve

Consider a small program that tracks books:

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages
```

This works.

But the moment you print a `Book`, the output is not useful:

```python
book = Book("Fluent Python", "Luciano Ramalho", 1014)
print(book)
```

You may see something like:

```text
<__main__.Book object at 0x104ac7190>
```

So you add `__repr__`:

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def __repr__(self):
        return (
            f"Book(title={self.title!r}, "
            f"author={self.author!r}, "
            f"pages={self.pages!r})"
        )
```

Then you compare two books:

```python
first = Book("Python", "A", 300)
second = Book("Python", "A", 300)

print(first == second)
```

Without `__eq__`, this is usually `False`, because the two variables refer to different objects.

So you add equality:

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def __repr__(self):
        return (
            f"Book(title={self.title!r}, "
            f"author={self.author!r}, "
            f"pages={self.pages!r})"
        )

    def __eq__(self, other):
        if type(other) is not Book:
            return NotImplemented
        return (
            self.title == other.title
            and self.author == other.author
            and self.pages == other.pages
        )
```

This is correct enough.

But now three fields created a lot of mechanical code.

If you add a fourth field, you must update multiple methods.

If you forget one, the class becomes inconsistent.

Dataclasses reduce that repetition.

```python
from dataclasses import dataclass


@dataclass
class Book:
    title: str
    author: str
    pages: int
```

This generates useful methods based on the fields.

You still wrote the class.

Python filled in common object behavior.

---

# A Dataclass Is Still a Class

This point matters.

A dataclass is not a dictionary.

It is not a tuple.

It is not a special record object from another language.

It is an ordinary Python class processed by the `@dataclass` decorator.

```python
from dataclasses import dataclass


@dataclass
class User:
    name: str
    email: str
```

You create instances normally:

```python
user = User("Maya", "maya@example.com")
```

You access attributes normally:

```python
print(user.name)
print(user.email)
```

You can mutate attributes unless the dataclass is frozen:

```python
user.email = "maya@company.com"
```

You can define methods:

```python
@dataclass
class User:
    name: str
    email: str

    def display_name(self):
        return self.name.title()
```

You can use inheritance.

You can use composition.

You can implement ABCs.

You can mix in behavior.

The dataclass decorator does not remove the class model.

It adds generated methods to the class model.

---

# The Smallest Useful Dataclass

Here is a tiny dataclass:

```python
from dataclasses import dataclass


@dataclass
class Point:
    x: int
    y: int
```

This says:

```text
Point has two fields: x and y
```

The annotations are important:

```python
x: int
y: int
```

Dataclasses discover fields from annotations.

If you write:

```python
@dataclass
class Point:
    x = 0
    y = 0
```

then `x` and `y` are not dataclass fields.

They are ordinary class attributes.

The dataclass decorator uses `__annotations__` to decide which attributes are fields.

This is why dataclasses and type hints often appear together.

But the annotations are not runtime validators by default.

This class:

```python
@dataclass
class Point:
    x: int
    y: int
```

does not automatically reject strings:

```python
point = Point("left", "top")
```

Python will create the object unless you add validation yourself or use a separate validation library.

The annotation tells tools and readers what should be there.

The dataclass decorator uses the annotation to identify the field.

---

# What `@dataclass` Generates

By default, `@dataclass` generates:

* `__init__`
* `__repr__`
* `__eq__`

It can also generate ordering methods and hashing behavior depending on options.

Start with:

```python
from dataclasses import dataclass


@dataclass
class Point:
    x: int
    y: int
```

Python behaves as if you had written an initializer like:

```python
def __init__(self, x: int, y: int):
    self.x = x
    self.y = y
```

It behaves as if you had a representation like:

```python
Point(x=10, y=20)
```

It behaves as if equality compares field values:

```python
Point(1, 2) == Point(1, 2)
```

returns:

```python
True
```

But:

```python
Point(1, 2) == Point(2, 1)
```

returns:

```python
False
```

The generated methods use the fields in the order they appear in the class body.

Field order matters for:

* constructor parameter order
* representation order
* comparison order
* ordering behavior when enabled
* pattern matching support

So this:

```python
@dataclass
class User:
    id: int
    name: str
```

is not identical in generated constructor shape to this:

```python
@dataclass
class User:
    name: str
    id: int
```

Same fields.

Different order.

---

# The Generated Initializer

The generated `__init__` accepts one argument for each field that has `init=True`.

Example:

```python
@dataclass
class Rectangle:
    width: float
    height: float
```

You can create:

```python
box = Rectangle(10.0, 5.0)
```

or:

```python
box = Rectangle(width=10.0, height=5.0)
```

The generated initializer assigns:

```python
self.width = width
self.height = height
```

It does not call property setters unless the fields are designed through descriptors or properties in a more advanced pattern.

For ordinary dataclasses, fields become ordinary instance attributes.

You can add methods that use those attributes:

```python
@dataclass
class Rectangle:
    width: float
    height: float

    def area(self):
        return self.width * self.height
```

The method is your code.

The initializer is generated.

That is the common dataclass shape:

```text
generated data plumbing + handwritten domain behavior
```

---

# Defaults

Dataclass fields can have default values:

```python
@dataclass
class User:
    name: str
    active: bool = True
```

Now `active` is optional:

```python
user = User("Maya")
print(user.active)
```

Output:

```python
True
```

This is similar to default function arguments.

The generated initializer acts like:

```python
def __init__(self, name: str, active: bool = True):
    self.name = name
    self.active = active
```

Fields without defaults must come before fields with defaults.

This is valid:

```python
@dataclass
class Task:
    title: str
    done: bool = False
```

This is not valid:

```python
@dataclass
class Task:
    done: bool = False
    title: str
```

The reason is the generated initializer would need to look like:

```python
def __init__(self, done=False, title):
    ...
```

Python function syntax does not allow a required positional parameter after a default positional parameter.

The rule also matters across inheritance.

If a base dataclass has default fields, a subclass cannot casually add required positional fields after them.

We will return to inheritance later in this chapter.

---

# The Mutable Default Trap

Mutable defaults are one of the most important dataclass lessons.

Suppose we want a shopping cart:

```python
from dataclasses import dataclass


@dataclass
class Cart:
    items: list[str] = []
```

This is wrong.

A list is mutable.

If a default list were shared across instances, changing one cart could affect another cart.

Python's dataclasses protect against common mutable defaults by raising an error for many built-in mutable cases.

The correct version uses `default_factory`:

```python
from dataclasses import dataclass, field


@dataclass
class Cart:
    items: list[str] = field(default_factory=list)
```

Now each `Cart` gets its own new list:

```python
first = Cart()
second = Cart()

first.items.append("book")

print(first.items)
print(second.items)
```

Output:

```python
['book']
[]
```

Think of it this way:

```text
default value -> one object written in the class body
default_factory -> function called for each new instance
```

For immutable defaults, use plain defaults:

```python
@dataclass
class Config:
    retries: int = 3
    debug: bool = False
```

For mutable defaults, use factories:

```python
@dataclass
class Inbox:
    messages: list[str] = field(default_factory=list)
    labels: set[str] = field(default_factory=set)
    metadata: dict[str, str] = field(default_factory=dict)
```

This is not just dataclass trivia.

It is the same object model from Volume I.

Lists, dictionaries, and sets are objects.

If you reuse one mutable object as a default, you reuse the same state.

`default_factory` asks Python to create fresh state per instance.

---

# `field()`

The `field()` function lets you configure one field.

Example:

```python
from dataclasses import dataclass, field


@dataclass
class User:
    username: str
    password_hash: str = field(repr=False)
```

Now printing the user will not expose the password hash:

```python
user = User("maya", "hash-value")
print(user)
```

Output:

```python
User(username='maya')
```

The field still exists:

```python
print(user.password_hash)
```

But it is excluded from `repr`.

Common `field()` options include:

* `default`
* `default_factory`
* `init`
* `repr`
* `compare`
* `hash`
* `metadata`
* `kw_only`

Recent Python versions also support a field-level documentation value through `doc`.

You will not use every option every day.

But you should understand the most common ones.

---

# Excluding a Field from `__init__`

Sometimes a field should exist on the instance but should not be accepted directly by the constructor.

Example:

```python
from dataclasses import dataclass, field


@dataclass
class Invoice:
    subtotal: float
    tax_rate: float
    total: float = field(init=False)

    def __post_init__(self):
        self.total = self.subtotal * (1 + self.tax_rate)
```

Now callers pass:

```python
invoice = Invoice(100.0, 0.18)
```

They do not pass `total`.

The object computes it.

This can be useful, but it should be used carefully.

If the computed value must always match other fields, ask whether it should be stored at all.

Sometimes this is better:

```python
@dataclass
class Invoice:
    subtotal: float
    tax_rate: float

    @property
    def total(self):
        return self.subtotal * (1 + self.tax_rate)
```

The stored version captures a value at initialization time.

The property version computes from current state every time.

Choose based on the meaning you want.

---

# `__post_init__`

The generated `__init__` assigns fields.

After it assigns fields, it calls `__post_init__` if the class defines one.

This gives you a hook for:

* validation
* normalization
* derived attributes
* setup that depends on multiple fields

Example:

```python
from dataclasses import dataclass


@dataclass
class Percentage:
    value: float

    def __post_init__(self):
        if not 0 <= self.value <= 100:
            raise ValueError("percentage must be between 0 and 100")
```

Now invalid objects are rejected:

```python
Percentage(50)
Percentage(120)  # ValueError
```

Dataclasses do not validate types automatically.

But `__post_init__` can validate values.

Example:

```python
@dataclass
class User:
    email: str

    def __post_init__(self):
        if "@" not in self.email:
            raise ValueError("email must contain @")
        self.email = self.email.strip().lower()
```

This normalizes email addresses after initialization.

Be careful with normalization.

Changing user input during construction can be helpful.

It can also hide bugs if done too aggressively.

Use `__post_init__` when the object has an invariant.

An invariant is a rule that should always be true for a valid object.

For `Percentage`, the invariant is:

```text
value is between 0 and 100
```

For `DateRange`, it might be:

```text
start is not after end
```

Example:

```python
from dataclasses import dataclass
from datetime import date


@dataclass
class DateRange:
    start: date
    end: date

    def __post_init__(self):
        if self.start > self.end:
            raise ValueError("start date cannot be after end date")
```

The purpose of `__post_init__` is not to stuff random setup into an object.

Its best use is protecting the object's meaning.

---

# Dataclasses and Type Hints

Dataclasses use annotations to find fields.

This does not mean Python enforces the annotation at runtime.

Example:

```python
@dataclass
class Product:
    name: str
    price: float
```

This says `price` should be a float.

But this code still runs by default:

```python
product = Product("Keyboard", "expensive")
```

The generated initializer does not check:

```python
isinstance(price, float)
```

Why not?

Because Python's type hints are primarily hints.

They help:

* readers
* editors
* static type checkers
* documentation tools
* dataclass field discovery

If you need runtime validation, you must write it:

```python
@dataclass
class Product:
    name: str
    price: float

    def __post_init__(self):
        if not isinstance(self.name, str):
            raise TypeError("name must be a string")
        if not isinstance(self.price, int | float):
            raise TypeError("price must be numeric")
```

Even this is incomplete for a full production validation system.

For example, `bool` is a subclass of `int`, decimals may matter, and coercion rules may differ by application.

Dataclasses are not validation frameworks.

They are class-generation tools.

Static type checking appears later in the book.

For now, remember:

```text
annotation identifies intent
dataclass uses it to find fields
runtime type enforcement is separate
```

---

# `repr` and Debuggability

The generated `__repr__` is one of the immediate joys of dataclasses.

Example:

```python
@dataclass
class Coordinate:
    lat: float
    lon: float
```

Printing gives:

```python
Coordinate(lat=12.9716, lon=77.5946)
```

That is much better than a memory address.

Good representations help when:

* debugging
* writing tests
* reading logs
* inspecting objects in a REPL
* comparing expected and actual values

But not every field belongs in `repr`.

Exclude sensitive or noisy fields:

```python
@dataclass
class ApiToken:
    name: str
    token: str = field(repr=False)
```

Exclude fields that are too large:

```python
@dataclass
class Dataset:
    name: str
    rows: list[dict[str, object]] = field(repr=False)
```

Do not blindly expose secrets because dataclasses make representation convenient.

A generated representation is useful only if it is safe and readable.

---

# Equality

By default, dataclasses compare by field values.

```python
@dataclass
class Point:
    x: int
    y: int
```

Then:

```python
Point(1, 2) == Point(1, 2)
```

is:

```python
True
```

This is value equality.

It connects directly to Volume I's identity and equality chapter.

Identity asks:

```text
are these the same object?
```

Equality asks:

```text
are these objects considered equal?
```

Dataclasses choose field-based equality by default.

This is excellent for value-like objects:

```python
@dataclass
class Money:
    amount: int
    currency: str
```

Two `Money(100, "INR")` objects should probably compare equal.

But field equality is not always correct.

Consider:

```python
@dataclass
class UserAccount:
    id: int
    email: str
    last_login_ip: str
```

Should two accounts be equal if all fields match?

Maybe.

Should they stop being equal when `last_login_ip` changes?

Probably not.

For entity objects, equality often depends on identity fields, not every field.

You can exclude fields from comparison:

```python
@dataclass
class UserAccount:
    id: int
    email: str
    last_login_ip: str = field(compare=False)
```

Now equality ignores `last_login_ip`.

You can also disable generated equality entirely:

```python
@dataclass(eq=False)
class UserAccount:
    id: int
    email: str
```

Then Python falls back to normal object equality unless you define your own `__eq__`.

The main design question is:

```text
is this object a value or an entity?
```

A value object is defined by its contents.

An entity is defined by identity, lifecycle, or external meaning.

Dataclasses are often wonderful for value objects.

They must be used more carefully for entities.

---

# Ordering

Dataclasses do not generate ordering methods by default.

This means:

```python
@dataclass
class Score:
    points: int
    player: str
```

This works:

```python
Score(10, "A") == Score(10, "A")
```

But this does not work by default:

```python
Score(10, "A") < Score(20, "B")
```

To generate ordering methods, use `order=True`:

```python
@dataclass(order=True)
class Score:
    points: int
    player: str
```

Now ordering compares fields in definition order.

That means:

```python
Score(10, "Zara") < Score(20, "Asha")
```

is true because `10 < 20`.

If points match, it compares `player` next.

This tuple-like ordering can be useful.

But it can also be misleading.

For a scoreboard, you may want highest points first.

For a task list, you may want priority first, then due date.

Field order becomes sorting logic.

Sometimes that is too implicit.

A common pattern is to create a hidden sort key:

```python
from dataclasses import dataclass, field


@dataclass(order=True)
class Task:
    sort_index: tuple[int, str] = field(init=False, repr=False)
    priority: int
    title: str

    def __post_init__(self):
        self.sort_index = (self.priority, self.title)
```

But do not reach for this too quickly.

Often an explicit `key` function is clearer:

```python
tasks.sort(key=lambda task: (task.priority, task.title))
```

Use `order=True` when the type has one natural ordering.

Use sort keys when the ordering depends on context.

---

# Hashing

Hashing is where dataclass behavior becomes subtle.

Objects used as dictionary keys or set elements need a hash.

But hashable objects should not change in ways that affect equality.

Example:

```python
@dataclass
class Point:
    x: int
    y: int
```

This dataclass is mutable by default.

It has generated equality.

So Python does not make it hashable by default.

This usually fails:

```python
points = {Point(1, 2)}
```

Why?

Because if a mutable object could be placed in a set and then changed, the set's internal hash table could become inconsistent.

Frozen dataclasses are different:

```python
@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

Now `Point(1, 2)` can usually be hashed if all fields are hashable:

```python
points = {Point(1, 2)}
```

The broad rule is:

```text
mutable + value equality -> usually unhashable
frozen + value equality -> usually hashable
```

There is an `unsafe_hash=True` option.

The name is honest.

Use it only when you truly understand the consequences.

If an object can change after being used as a dictionary key, you can create bugs that are difficult to see.

Hashing is not just a method-generation detail.

It is a promise about object stability.

---

# Frozen Dataclasses

A frozen dataclass prevents normal attribute assignment after creation.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Money:
    amount: int
    currency: str
```

Now:

```python
price = Money(100, "INR")
price.amount = 200
```

raises an error.

Frozen dataclasses are useful for value objects:

```python
@dataclass(frozen=True)
class Coordinate:
    lat: float
    lon: float
```

They express:

```text
this object should not change after construction
```

But frozen does not mean deep immutability.

This is still risky:

```python
@dataclass(frozen=True)
class FrozenCart:
    items: list[str]
```

You cannot assign a new list:

```python
cart.items = []
```

But you can mutate the existing list:

```python
cart.items.append("book")
```

The dataclass is frozen at the attribute-assignment level.

It does not recursively freeze every object referenced by its fields.

If you want a deeply immutable shape, use immutable field types:

```python
@dataclass(frozen=True)
class FrozenCart:
    items: tuple[str, ...]
```

This is much safer.

The object model lesson is familiar:

```text
an attribute stores a reference
freezing the attribute does not freeze every object behind the reference
```

---

# Working Inside Frozen Dataclasses

Sometimes a frozen dataclass needs derived state.

Normal assignment in `__post_init__` does not work:

```python
@dataclass(frozen=True)
class Slug:
    text: str
    value: str = field(init=False)

    def __post_init__(self):
        self.value = self.text.lower().replace(" ", "-")  # error
```

Because the instance is frozen.

The escape hatch is `object.__setattr__`:

```python
@dataclass(frozen=True)
class Slug:
    text: str
    value: str = field(init=False)

    def __post_init__(self):
        slug = self.text.strip().lower().replace(" ", "-")
        object.__setattr__(self, "value", slug)
```

Use this sparingly.

It is appropriate during initialization.

It should not become a back door for random mutation.

The meaning of frozen should remain clear:

```text
after construction, this object behaves as immutable
```

---

# Keyword-Only Fields

Dataclasses can support keyword-only fields.

This is useful when positional arguments would become unclear.

Example:

```python
@dataclass(kw_only=True)
class Email:
    to: str
    subject: str
    body: str
    urgent: bool = False
```

Now callers must use keywords:

```python
message = Email(
    to="maya@example.com",
    subject="Welcome",
    body="Hello",
    urgent=True,
)
```

This avoids unclear calls like:

```python
Email("maya@example.com", "Welcome", "Hello", True)
```

For small mathematical objects, positional arguments are fine:

```python
Point(10, 20)
```

For business objects, keyword-only construction is often clearer:

```python
UserProfile(
    username="maya",
    display_name="Maya Rao",
    newsletter_opt_in=True,
)
```

You can also mark individual fields keyword-only with `field(kw_only=True)`.

Use keyword-only fields when:

* there are many fields
* several fields have the same type
* booleans appear in the constructor
* call sites should read like declarations

Boolean positional arguments are especially hard to read:

```python
Account("maya", True, False, True)
```

This tells the reader almost nothing.

Keyword arguments tell the story:

```python
Account(
    username="maya",
    active=True,
    verified=False,
    staff=True,
)
```

---

# `ClassVar`

Sometimes a class attribute should not become a dataclass field.

Use `ClassVar` for that.

```python
from dataclasses import dataclass
from typing import ClassVar


@dataclass
class User:
    username: str
    table_name: ClassVar[str] = "users"
```

Here `username` is an instance field.

`table_name` is a class variable.

It is not included in the generated initializer.

It is not included in equality.

It is not returned by dataclass field inspection.

This matters because dataclasses use annotations.

Without `ClassVar`, this:

```python
@dataclass
class User:
    username: str
    table_name: str = "users"
```

would treat `table_name` as an instance field.

That may not be what you mean.

Use `ClassVar` when the value belongs to the class as a whole.

Examples:

```python
@dataclass
class Currency:
    code: str
    minor_units: int
    registry_name: ClassVar[str] = "iso-4217"
```

The registry name describes the class concept.

It is not separate data for each instance.

---

# `InitVar`

`InitVar` creates an initialization-only parameter.

It is accepted by `__init__`.

It is passed to `__post_init__`.

It is not stored as a normal dataclass field.

Example:

```python
from dataclasses import dataclass, InitVar


@dataclass
class User:
    username: str
    raw_password: InitVar[str]
    password_hash: str = ""

    def __post_init__(self, raw_password):
        self.password_hash = f"hashed:{raw_password}"
```

You construct:

```python
user = User("maya", "secret")
```

The raw password is used during initialization.

It is not stored as `user.raw_password`.

This is useful for values needed only to build the object.

Another example:

```python
@dataclass
class Report:
    title: str
    data: list[int]
    normalize: InitVar[bool] = False

    def __post_init__(self, normalize):
        if normalize:
            total = sum(self.data)
            if total:
                self.data = [value / total for value in self.data]
```

The `normalize` parameter changes construction behavior.

It is not part of the final object's data.

Use `InitVar` when a constructor input is not an instance attribute.

Do not overuse it.

If many initialization-only inputs are needed, your construction process may deserve a separate factory function or builder.

---

# Dataclasses with Methods

A common beginner mistake is thinking dataclasses should not have methods.

They absolutely can.

Example:

```python
@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def add(self, other):
        if self.currency != other.currency:
            raise ValueError("cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)
```

This is a strong dataclass.

It has:

* simple data
* clear invariants
* meaningful behavior
* value semantics

A dataclass is not merely a passive bag of attributes.

It can be a real domain object.

Another example:

```python
@dataclass
class Rectangle:
    width: float
    height: float

    def area(self):
        return self.width * self.height

    def scale(self, factor):
        self.width *= factor
        self.height *= factor
```

The class stores state and defines behavior related to that state.

That is ordinary object-oriented design.

The dataclass decorator only removed repetitive method definitions.

---

# Dataclasses Are Not Dictionaries

A dictionary is flexible:

```python
user = {
    "name": "Maya",
    "email": "maya@example.com",
}
```

You can add any key:

```python
user["last_login"] = "today"
```

That flexibility is useful.

But it also means typos can become runtime bugs:

```python
print(user["emial"])
```

A dataclass has named attributes:

```python
@dataclass
class User:
    name: str
    email: str
```

Then:

```python
user = User("Maya", "maya@example.com")
```

Attribute names are visible in the class definition.

Editors and type checkers can help.

Refactoring is easier.

The object can have behavior.

But dictionaries are still better for:

* truly dynamic keys
* JSON-like data with unknown shape
* quick accumulation
* flexible mappings

Dataclasses are better for:

* known fields
* domain concepts
* structured inputs and outputs
* readable object representations
* value objects

Do not replace every dictionary with a dataclass.

Use dataclasses when the structure deserves a name.

---

# Dataclasses Are Not Tuples

A tuple is positional:

```python
point = (10, 20)
```

This is compact.

But the meaning of each element lives outside the tuple.

You must remember:

```text
index 0 is x
index 1 is y
```

A dataclass names the parts:

```python
@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

Now:

```python
point.x
point.y
```

carry meaning.

Tuples are good for:

* small ad hoc groupings
* returning multiple values locally
* fixed positional protocols
* immutable sequences

Dataclasses are good when:

* fields need names
* behavior may grow
* equality should be value-based
* the concept deserves a type

This function is fine:

```python
def min_max(values):
    return min(values), max(values)
```

But this result probably deserves a dataclass:

```python
@dataclass(frozen=True)
class SearchResult:
    title: str
    url: str
    snippet: str
    score: float
```

The more meaning the grouped data has, the more useful a named class becomes.

---

# Dataclasses and Named Tuples

Named tuples also provide named fields.

Example:

```python
from typing import NamedTuple


class Point(NamedTuple):
    x: int
    y: int
```

Named tuples are immutable and tuple-like.

They support indexing:

```python
point = Point(10, 20)
print(point[0])
```

Dataclasses are class-like by default.

They do not behave like tuples unless you add such behavior yourself.

Use a named tuple when tuple compatibility matters.

Use a dataclass when object modeling matters.

Example:

```python
@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

This is usually better when you want a domain object.

Named tuples are excellent for lightweight immutable records.

Dataclasses are more flexible as your model grows.

---

# `asdict`

The `asdict()` function converts a dataclass instance into a dictionary.

```python
from dataclasses import dataclass, asdict


@dataclass
class User:
    name: str
    email: str


user = User("Maya", "maya@example.com")
print(asdict(user))
```

Output:

```python
{'name': 'Maya', 'email': 'maya@example.com'}
```

Nested dataclasses are converted recursively:

```python
@dataclass
class Address:
    city: str
    country: str


@dataclass
class Customer:
    name: str
    address: Address
```

Then:

```python
customer = Customer("Maya", Address("Bengaluru", "India"))
print(asdict(customer))
```

Output:

```python
{'name': 'Maya', 'address': {'city': 'Bengaluru', 'country': 'India'}}
```

This is useful for serialization boundaries.

But there is an important detail:

`asdict()` recursively copies dataclass structures and uses deep-copy behavior for many contained objects.

That can be more expensive than expected.

For shallow conversion, use field inspection:

```python
from dataclasses import fields


def shallow_asdict(obj):
    return {field.name: getattr(obj, field.name) for field in fields(obj)}
```

Use `asdict()` when you want recursive conversion.

Use a custom function when you need exact control.

---

# `astuple`

The `astuple()` function converts a dataclass instance into a tuple.

```python
from dataclasses import dataclass, astuple


@dataclass
class Point:
    x: int
    y: int


point = Point(10, 20)
print(astuple(point))
```

Output:

```python
(10, 20)
```

Like `asdict()`, it recursively handles nested dataclasses.

This can be useful when:

* interoperating with tuple-based code
* writing tests
* converting to simple structures

But do not use `astuple()` as a habit.

If the names matter, a dictionary or the dataclass itself may be clearer.

Tuple conversion removes field names.

That can make data less readable.

---

# `replace`

The `replace()` function creates a new dataclass instance with some fields changed.

```python
from dataclasses import dataclass, replace


@dataclass(frozen=True)
class Settings:
    theme: str
    notifications: bool


settings = Settings("light", True)
dark_settings = replace(settings, theme="dark")
```

The original remains:

```python
Settings(theme='light', notifications=True)
```

The new one is:

```python
Settings(theme='dark', notifications=True)
```

This is especially useful for frozen dataclasses.

Instead of mutating:

```python
settings.theme = "dark"
```

you create a modified copy:

```python
settings = replace(settings, theme="dark")
```

This style is common in functional programming and immutable object design.

It can make state transitions easier to reason about.

But remember that `replace()` creates the new object through the dataclass initializer.

That means `__post_init__` runs again.

This is usually good.

It preserves validation.

But it can surprise you if `__post_init__` has side effects.

Another reason to keep `__post_init__` focused on object invariants.

---

# Inspecting Fields

The `fields()` function returns information about dataclass fields.

```python
from dataclasses import dataclass, fields


@dataclass
class User:
    name: str
    email: str


for field in fields(User):
    print(field.name, field.type)
```

This can power:

* serializers
* form generators
* schema tools
* test helpers
* debugging utilities

Each field object includes information such as:

* name
* type
* default
* default factory
* whether it appears in `init`
* whether it appears in `repr`
* whether it participates in comparison
* metadata

You should use `fields()` instead of poking at private dataclass internals.

If you want field names, ask the dataclass system.

Do not rely on generated `__slots__`, private attributes, or implementation details.

---

# Metadata

Dataclass fields can carry metadata.

Dataclasses themselves do not use this metadata.

It is provided for your tools and frameworks.

Example:

```python
from dataclasses import dataclass, field, fields


@dataclass
class Product:
    name: str = field(metadata={"label": "Product name"})
    price: float = field(metadata={"label": "Unit price"})
```

You can inspect it:

```python
for item in fields(Product):
    print(item.name, item.metadata.get("label"))
```

Metadata can support:

* form labels
* serialization names
* validation hints
* database mapping hints
* documentation generation

But be careful.

Too much metadata can turn a simple dataclass into a mini framework.

If metadata begins to carry core business logic, consider a clearer design.

---

# Slots in Dataclasses

Dataclasses can generate `__slots__` by using `slots=True`:

```python
@dataclass(slots=True)
class Point:
    x: int
    y: int
```

This can reduce per-instance memory overhead and prevent arbitrary new attributes.

Example:

```python
point = Point(1, 2)
point.z = 3  # error
```

Slots are useful when:

* you create many instances
* memory matters
* you want a fixed attribute layout

But slots also have tradeoffs.

They affect weak references, inheritance, attribute storage, and some dynamic patterns.

Because `__slots__` deserves its own careful treatment, Chapter 56 studies it in depth.

For now, know that dataclasses can participate in slot-based design.

Do not turn on `slots=True` everywhere just because it looks efficient.

Measure first.

Design second.

Optimize when needed.

---

# Weak References and `weakref_slot`

In Chapter 37, we studied weak references.

Slot-based classes need special support if their instances should be weak-referenceable.

Dataclasses provide `weakref_slot=True` for this case:

```python
@dataclass(slots=True, weakref_slot=True)
class Node:
    name: str
```

This adds the weak-reference slot needed by the object layout.

You only use this when:

* the dataclass uses `slots=True`
* instances need to be targets of weak references

Most dataclasses do not need this option.

It exists for advanced memory-sensitive designs.

The important connection is that dataclasses are not isolated from Python internals.

They sit directly on top of the same object model, attribute model, and memory model we have been building.

---

# Pattern Matching and `__match_args__`

Python's structural pattern matching can work with dataclasses.

Example:

```python
@dataclass
class Point:
    x: int
    y: int
```

You can match by position:

```python
def describe(point):
    match point:
        case Point(0, 0):
            return "origin"
        case Point(x, 0):
            return f"x-axis at {x}"
        case Point(0, y):
            return f"y-axis at {y}"
        case Point(x, y):
            return f"point at {x}, {y}"
```

Dataclasses generate `__match_args__` by default.

That tells pattern matching which fields correspond to positional patterns.

Keyword patterns are often clearer:

```python
match point:
    case Point(x=0, y=0):
        print("origin")
```

For large dataclasses, prefer keyword patterns.

They are more readable and less fragile if field order changes.

You can disable generated match args:

```python
@dataclass(match_args=False)
class Event:
    type: str
    payload: dict[str, object]
```

Most code can leave the default alone.

But it is useful to know that dataclasses participate in the broader data model.

---

# Inheritance with Dataclasses

Dataclasses can inherit from dataclasses.

Example:

```python
@dataclass
class Person:
    name: str


@dataclass
class Employee(Person):
    employee_id: int
```

Now:

```python
employee = Employee("Maya", 101)
```

The generated initializer includes fields from the base dataclass and the subclass.

Inheritance is useful when the relationship is genuinely an `is-a` relationship.

But inheritance with dataclasses can become tricky because:

* field order matters
* default fields affect subclass constructor rules
* generated methods must combine fields correctly
* multiple inheritance can become difficult to reason about

Example of a potential problem:

```python
@dataclass
class Base:
    active: bool = True


@dataclass
class User(Base):
    username: str
```

This creates a required field after a default field through inheritance.

The generated initializer shape would be invalid.

A common fix is keyword-only fields:

```python
@dataclass(kw_only=True)
class Base:
    active: bool = True


@dataclass
class User(Base):
    username: str
```

Another fix is redesigning the hierarchy.

Do not force inheritance just to reuse fields.

Composition is often clearer:

```python
@dataclass
class AuditInfo:
    created_by: str
    active: bool = True


@dataclass
class User:
    username: str
    audit: AuditInfo
```

Dataclass inheritance is powerful.

But like all inheritance, it should represent a real conceptual relationship.

---

# Calling Base Initializers

Generated dataclass initializers do not automatically call every non-dataclass base class initializer in the way beginners might expect.

Suppose:

```python
class Resource:
    def __init__(self):
        self.closed = False
```

And:

```python
@dataclass
class FileResource(Resource):
    path: str
```

The generated initializer for `FileResource` focuses on dataclass fields.

If the base initializer must run, call it from `__post_init__`:

```python
@dataclass
class FileResource(Resource):
    path: str

    def __post_init__(self):
        super().__init__()
```

This is one reason to be cautious with dataclasses that inherit from complex non-dataclass bases.

Dataclasses are simplest when the class hierarchy is simple.

When base classes perform important setup, read the constructor flow carefully.

The MRO chapter is relevant here.

Generated methods are still methods.

Inheritance still follows Python's method resolution rules.

---

# Dataclasses and ABCs

A dataclass can implement an abstract base class.

Example:

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass


class Shape(ABC):
    @abstractmethod
    def area(self):
        raise NotImplementedError


@dataclass
class Rectangle(Shape):
    width: float
    height: float

    def area(self):
        return self.width * self.height
```

This is a natural combination.

The ABC defines required behavior.

The dataclass stores the data needed for that behavior.

Another example:

```python
class Command(ABC):
    @abstractmethod
    def execute(self):
        raise NotImplementedError


@dataclass
class SendEmail(Command):
    to: str
    subject: str
    body: str

    def execute(self):
        print(f"sending email to {self.to}")
```

This is often cleaner than writing repetitive constructors for every command class.

The design question remains:

```text
what is generated data plumbing, and what behavior must I write?
```

Dataclasses handle the plumbing.

ABCs define behavioral obligations.

Your methods implement the domain meaning.

---

# Dataclasses and Mixins

Dataclasses can use mixins too.

Example:

```python
class JsonMixin:
    def to_json_dict(self):
        return self.__dict__.copy()


@dataclass
class User(JsonMixin):
    name: str
    email: str
```

This works for simple non-slotted dataclasses.

But be careful with mixins that assume `__dict__`.

If you later use `slots=True`, instances may not have a normal `__dict__`.

A better mixin can use dataclass inspection:

```python
from dataclasses import fields


class DataclassDictMixin:
    def to_dict(self):
        return {
            field.name: getattr(self, field.name)
            for field in fields(self)
        }
```

Then:

```python
@dataclass
class User(DataclassDictMixin):
    name: str
    email: str
```

This mixin understands dataclasses.

But it should probably check that `self` is a dataclass instance if used broadly.

Mixin design still follows the rules from Chapter 50:

* keep it small
* avoid hidden assumptions
* document what it expects
* cooperate with the object model

---

# Dataclasses and Properties

Dataclasses work well with properties, but you must separate stored fields from computed attributes.

Example:

```python
@dataclass
class Rectangle:
    width: float
    height: float

    @property
    def area(self):
        return self.width * self.height
```

Here `width` and `height` are fields.

`area` is computed.

It is not a dataclass field.

This is usually the right design when the value can be derived cheaply.

Avoid storing redundant data unless needed.

Less ideal:

```python
@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)

    def __post_init__(self):
        self.area = self.width * self.height
```

If `width` changes later, `area` becomes stale.

You can prevent that with a frozen dataclass:

```python
@dataclass(frozen=True)
class Rectangle:
    width: float
    height: float

    @property
    def area(self):
        return self.width * self.height
```

Now the derived value always reflects the immutable fields.

Chapter 55 studies properties, static methods, and class methods in more detail.

For dataclasses, the basic rule is:

```text
store what must be stored
compute what can be computed clearly
```

---

# Custom `__init__`

Sometimes you do not want a generated initializer.

You can disable it:

```python
@dataclass(init=False)
class User:
    name: str
    email: str

    def __init__(self, name):
        self.name = name
        self.email = f"{name.lower()}@example.com"
```

This is valid.

But if you write most methods yourself, ask whether `@dataclass` is still helping.

Another approach is to keep the generated initializer and add class methods:

```python
@dataclass
class User:
    name: str
    email: str

    @classmethod
    def from_username(cls, username):
        return cls(
            name=username,
            email=f"{username.lower()}@example.com",
        )
```

This often keeps the primary object shape simple.

Use factory methods when:

* there are multiple construction paths
* input needs parsing
* construction has a name
* you want to preserve the generated initializer

Use custom `__init__` when:

* the generated initializer fundamentally does not match the object
* initialization must be completely controlled
* field assignment is unusual

Dataclasses are most valuable when you let them generate common behavior.

If you disable everything, you may be using the wrong tool.

---

# Field-Level Comparison Control

Suppose we want tasks sorted by priority, but equality should care only about an ID.

This is a design smell if pushed too far, but it is useful to understand.

```python
from dataclasses import dataclass, field


@dataclass
class Task:
    id: int
    title: str
    priority: int = field(compare=False)
```

Now `priority` is ignored by equality.

```python
Task(1, "Write", 5) == Task(1, "Write", 1)
```

is:

```python
True
```

because `priority` is excluded.

You can exclude noisy or operational fields:

```python
@dataclass
class Job:
    id: str
    payload: dict[str, object]
    attempts: int = field(default=0, compare=False)
    last_error: str | None = field(default=None, compare=False)
```

This means:

```text
same id and payload -> same job identity for comparison
attempt history does not affect equality
```

Be explicit.

If comparison behavior is not obvious, write tests.

Generated equality is convenient, but equality is part of your domain model.

Treat it with care.

---

# Value Objects

Dataclasses are excellent for value objects.

A value object is defined by its values rather than a separate identity.

Examples:

* `Money(100, "INR")`
* `Point(10, 20)`
* `DateRange(start, end)`
* `EmailAddress("maya@example.com")`
* `Version(1, 4, 2)`

A strong value object is often:

* small
* validated
* immutable
* comparable by value
* easy to print

Dataclasses support this shape:

```python
@dataclass(frozen=True)
class Version:
    major: int
    minor: int
    patch: int = 0

    def __post_init__(self):
        if self.major < 0 or self.minor < 0 or self.patch < 0:
            raise ValueError("version numbers cannot be negative")

    def __str__(self):
        return f"{self.major}.{self.minor}.{self.patch}"
```

This is a good dataclass.

It has clear data.

It has a clear invariant.

It has useful behavior.

It should not change after creation.

---

# Entity Objects

Entities are different.

An entity has identity and lifecycle.

Example:

```python
@dataclass
class User:
    id: int
    email: str
    display_name: str
```

This may be fine.

But ask:

```text
when are two User objects equal?
```

If equality should be based only on `id`, the generated equality may be wrong.

Two objects representing the same database user might have different display names because one is stale.

Should they compare unequal?

Maybe not.

For entities, consider:

```python
@dataclass(eq=False)
class User:
    id: int
    email: str
    display_name: str

    def __eq__(self, other):
        if type(other) is not User:
            return NotImplemented
        return self.id == other.id

    def __hash__(self):
        return hash(self.id)
```

Or avoid custom equality and compare IDs explicitly.

The dataclass can still be useful for constructor and representation generation.

But the default value semantics may not fit every entity.

The design principle is:

```text
do not let generated methods quietly define your domain semantics by accident
```

---

# Configuration Objects

Dataclasses are excellent for configuration.

Example:

```python
@dataclass(frozen=True)
class ServerConfig:
    host: str
    port: int
    debug: bool = False
    workers: int = 4

    def __post_init__(self):
        if not 1 <= self.port <= 65535:
            raise ValueError("port must be between 1 and 65535")
        if self.workers < 1:
            raise ValueError("workers must be positive")
```

Configuration objects benefit from:

* named fields
* defaults
* validation
* readable representation
* immutability
* easy replacement

Example:

```python
production = ServerConfig("0.0.0.0", 8000)
debug = replace(production, debug=True)
```

This is much clearer than passing dictionaries through the program.

It gives configuration a shape.

It also makes invalid configuration fail early.

---

# Request and Response Models

Dataclasses can model internal request and response objects.

Example:

```python
@dataclass(frozen=True)
class CreateUserRequest:
    email: str
    display_name: str


@dataclass(frozen=True)
class CreateUserResponse:
    user_id: int
    email: str
```

This can make service boundaries clear:

```python
def create_user(request: CreateUserRequest) -> CreateUserResponse:
    ...
```

But remember:

Dataclasses do not parse JSON automatically.

They do not validate untrusted data automatically.

At external boundaries, you may need explicit parsing or a validation library.

Inside your application, dataclasses can be excellent structured messages.

At the boundary, be careful.

Never assume that external dictionaries are valid just because you can unpack them:

```python
request = CreateUserRequest(**payload)
```

That may fail.

Or worse, it may succeed with values of the wrong runtime type.

External input deserves explicit validation.

---

# Serialization Boundaries

Dataclasses often sit near serialization code.

Example:

```python
@dataclass
class Event:
    type: str
    payload: dict[str, object]
```

You might convert it:

```python
data = asdict(event)
```

But serialization is not always simple.

What about:

* dates
* decimals
* enums
* bytes
* nested custom classes
* private fields
* renamed fields
* optional fields
* versioned schemas

`asdict()` is a structural conversion.

It is not a complete serialization policy.

For serious APIs, design serialization deliberately.

Example:

```python
@dataclass(frozen=True)
class Event:
    type: str
    created_at: datetime

    def to_json_dict(self):
        return {
            "type": self.type,
            "created_at": self.created_at.isoformat(),
        }
```

This method states the external representation.

That is often better than exposing internal object shape automatically.

---

# Dataclasses and Validation Libraries

Dataclasses are in the standard library.

They are lightweight.

They do not perform rich runtime validation.

Other tools can offer more:

* runtime type validation
* parsing
* coercion
* schema generation
* error reporting
* JSON integration

Those tools are useful, but they are not the same thing as dataclasses.

Dataclasses are best understood as:

```text
standard-library support for generating common class methods from annotated fields
```

If your problem is:

```text
I want less boilerplate in simple classes
```

dataclasses are a good fit.

If your problem is:

```text
I need robust validation of untrusted nested JSON
```

you may need more than dataclasses.

This distinction prevents disappointment.

Use the right tool at the right boundary.

---

# Dataclasses and `__dict__`

Most regular dataclass instances have a `__dict__`.

Example:

```python
@dataclass
class User:
    name: str
    email: str
```

Then:

```python
user = User("Maya", "maya@example.com")
print(user.__dict__)
```

Output:

```python
{'name': 'Maya', 'email': 'maya@example.com'}
```

This is the same attribute storage model we studied earlier.

But do not build important code by assuming every dataclass has `__dict__`.

With `slots=True`, the instance may not have one.

With inheritance or descriptors, attribute storage may differ.

Use dataclass tools when you mean dataclass fields:

```python
from dataclasses import fields


def extract(obj):
    return {
        field.name: getattr(obj, field.name)
        for field in fields(obj)
    }
```

This says what you mean.

You want dataclass fields, not arbitrary instance storage.

---

# Generated Methods and Existing Methods

If you define a method that dataclasses would otherwise generate, your method can take precedence depending on the option.

Example:

```python
@dataclass
class User:
    name: str

    def __repr__(self):
        return f"<User {self.name}>"
```

The dataclass decorator will not replace this `__repr__`.

You wrote it.

Python keeps it.

This gives you control.

But be careful when combining generated and custom methods.

If you customize `__eq__`, consider hashing.

If you customize `__init__`, consider `__post_init__`.

If you customize `__repr__`, consider whether sensitive fields are handled.

Generated methods are meant to be coherent as a group.

When you replace one, make sure the rest still make sense.

---

# The Decorator Options

The full `@dataclass` decorator has several options.

You do not need to memorize them all.

But you should recognize what each controls.

Common options:

```python
@dataclass(
    init=True,
    repr=True,
    eq=True,
    order=False,
    unsafe_hash=False,
    frozen=False,
    match_args=True,
    kw_only=False,
    slots=False,
    weakref_slot=False,
)
class Example:
    ...
```

The practical meanings:

```text
init         generate __init__
repr         generate __repr__
eq           generate __eq__
order        generate ordering methods
unsafe_hash  force hash generation
frozen       block normal assignment after creation
match_args   support positional pattern matching
kw_only      make fields keyword-only
slots        generate slots
weakref_slot support weak references with slots
```

Most dataclasses use the default:

```python
@dataclass
class Example:
    ...
```

Some use:

```python
@dataclass(frozen=True)
```

or:

```python
@dataclass(order=True)
```

or:

```python
@dataclass(kw_only=True)
```

Use options to express design.

Do not turn options on because they exist.

---

# A Practical Example: Money

Let us design a small `Money` type.

First attempt:

```python
@dataclass
class Money:
    amount: int
    currency: str
```

This works.

But money values should probably not mutate casually.

Use `frozen=True`:

```python
@dataclass(frozen=True)
class Money:
    amount: int
    currency: str
```

Now add validation:

```python
@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def __post_init__(self):
        if not isinstance(self.amount, int):
            raise TypeError("amount must be stored in minor units as an integer")
        if len(self.currency) != 3:
            raise ValueError("currency must be a 3-letter code")
```

Now add behavior:

```python
@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def __post_init__(self):
        if not isinstance(self.amount, int):
            raise TypeError("amount must be stored in minor units as an integer")
        if len(self.currency) != 3:
            raise ValueError("currency must be a 3-letter code")

    def add(self, other):
        if self.currency != other.currency:
            raise ValueError("cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)
```

This is no longer just data storage.

It is a small domain type.

Dataclasses support this style well.

---

# A Practical Example: Search Results

Suppose a search system returns results:

```python
@dataclass(frozen=True)
class SearchResult:
    title: str
    url: str
    snippet: str
    score: float
```

This is a clean data container.

It has a clear shape.

It prints well.

It compares by value.

It can be converted to a dictionary.

But suppose you want to sort by score descending.

Do not necessarily use `order=True`.

This is clearer:

```python
results.sort(key=lambda result: result.score, reverse=True)
```

Sorting by score is a use-case-specific operation.

The type itself does not need a universal ordering.

This is a common design judgment.

Generated ordering is powerful, but explicit sorting often communicates intent better.

---

# A Practical Example: Commands

Dataclasses are useful for command objects.

```python
@dataclass(frozen=True)
class CreateProject:
    name: str
    owner_id: int
```

The command describes an action.

It can be passed through handlers:

```python
def handle_create_project(command: CreateProject):
    ...
```

This is clearer than passing several loose parameters through multiple layers:

```python
handle_create_project(name, owner_id)
```

The dataclass gives the bundle a name.

It improves readability.

It also makes future changes easier:

```python
@dataclass(frozen=True)
class CreateProject:
    name: str
    owner_id: int
    template_id: int | None = None
```

Now the command can evolve without changing every function signature in the same way.

Do not overdo this for tiny local functions.

But for application boundaries, named request and command objects can make code easier to understand.

---

# Common Mistake: Treating Dataclasses as Validation

This is wrong:

```python
@dataclass
class Age:
    value: int


age = Age("old")
```

The annotation does not stop this.

If your object must be valid, write validation:

```python
@dataclass(frozen=True)
class Age:
    value: int

    def __post_init__(self):
        if not isinstance(self.value, int):
            raise TypeError("age must be an integer")
        if self.value < 0:
            raise ValueError("age cannot be negative")
```

Or validate before constructing the object.

Or use a runtime validation tool.

The key is not to assume.

Dataclass annotations are not runtime guards by default.

---

# Common Mistake: Too Many Fields

A dataclass with many fields can become hard to use:

```python
@dataclass
class UserProfile:
    first_name: str
    last_name: str
    email: str
    phone: str
    address_line_1: str
    address_line_2: str
    city: str
    state: str
    country: str
    postal_code: str
    newsletter: bool
    sms_alerts: bool
    dark_mode: bool
```

This may be acceptable in some contexts.

But often it signals missing structure.

Break related concepts apart:

```python
@dataclass
class Address:
    line_1: str
    line_2: str
    city: str
    state: str
    country: str
    postal_code: str


@dataclass
class Preferences:
    newsletter: bool
    sms_alerts: bool
    dark_mode: bool


@dataclass
class UserProfile:
    first_name: str
    last_name: str
    email: str
    phone: str
    address: Address
    preferences: Preferences
```

This uses composition.

The object becomes easier to read.

The constructor becomes more meaningful.

The structure reflects the domain.

Dataclasses make large data shapes easy to write.

They do not make large data shapes automatically good.

---

# Common Mistake: Hiding Behavior Outside the Class

Suppose you have:

```python
@dataclass
class Order:
    subtotal: float
    discount: float
    tax_rate: float
```

And then many functions:

```python
def order_total(order):
    return (order.subtotal - order.discount) * (1 + order.tax_rate)


def order_is_discounted(order):
    return order.discount > 0
```

This may be fine for simple procedural code.

But if these operations define what an order means, methods may be clearer:

```python
@dataclass
class Order:
    subtotal: float
    discount: float
    tax_rate: float

    def total(self):
        return (self.subtotal - self.discount) * (1 + self.tax_rate)

    def is_discounted(self):
        return self.discount > 0
```

Dataclasses should not make you afraid of methods.

If behavior belongs to the data, keep it near the data.

---

# Common Mistake: Using Dataclasses for Everything

Dataclasses are comfortable.

That makes them easy to overuse.

Do not use a dataclass when:

* the object has complex lifecycle behavior
* identity matters more than field values
* fields are highly dynamic
* construction must perform significant work
* validation and parsing dominate the problem
* a plain tuple or dictionary is clearer
* a normal class better expresses the design

Example where a normal class may be better:

```python
class DatabaseConnection:
    def __init__(self, dsn):
        self.dsn = dsn
        self._connection = None

    def connect(self):
        ...

    def close(self):
        ...
```

This is not mainly structured data.

It is a resource with lifecycle.

A dataclass would not add much.

It might even make the class seem more value-like than it really is.

The right question is:

```text
does this class primarily model structured state?
```

If yes, consider a dataclass.

If no, use a normal class.

---

# Common Mistake: Accidental Positional APIs

This dataclass:

```python
@dataclass
class Payment:
    amount: int
    currency: str
    captured: bool
    refunded: bool
```

allows:

```python
Payment(1000, "INR", True, False)
```

That call is not very readable.

For business objects, prefer keyword-only:

```python
@dataclass(kw_only=True)
class Payment:
    amount: int
    currency: str
    captured: bool
    refunded: bool
```

Then:

```python
Payment(
    amount=1000,
    currency="INR",
    captured=True,
    refunded=False,
)
```

The call is longer.

But it is safer and clearer.

The constructor is part of your API.

Design it deliberately.

---

# Common Mistake: Stale Derived Fields

This is dangerous:

```python
@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)

    def __post_init__(self):
        self.area = self.width * self.height
```

Why?

Because:

```python
rectangle = Rectangle(10, 20)
rectangle.width = 100
print(rectangle.area)
```

The area is now stale.

Better options:

Use a property:

```python
@dataclass
class Rectangle:
    width: float
    height: float

    @property
    def area(self):
        return self.width * self.height
```

Or freeze the object:

```python
@dataclass(frozen=True)
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)

    def __post_init__(self):
        object.__setattr__(self, "area", self.width * self.height)
```

The property is usually simpler.

Store derived values only when:

* computation is expensive
* the object is immutable
* the cached value has clear invalidation rules

---

# Common Mistake: Confusing `field(default=...)` and `field(default_factory=...)`

Use `default` when the default value is already the value you want:

```python
@dataclass
class RetryPolicy:
    attempts: int = field(default=3)
```

This is equivalent to:

```python
@dataclass
class RetryPolicy:
    attempts: int = 3
```

Use `default_factory` when you need a new value per instance:

```python
@dataclass
class Buffer:
    items: list[str] = field(default_factory=list)
```

Do not write:

```python
field(default_factory=list())
```

That calls `list()` immediately.

You want to pass the callable:

```python
field(default_factory=list)
```

The factory must take no arguments.

If you need arguments, wrap it:

```python
def default_tags():
    return ["new"]


@dataclass
class Article:
    tags: list[str] = field(default_factory=default_tags)
```

Or use a lambda carefully:

```python
@dataclass
class Article:
    tags: list[str] = field(default_factory=lambda: ["new"])
```

Named functions are often clearer than lambdas when defaults have meaning.

---

# Common Mistake: Leaking Secrets in `repr`

This is risky:

```python
@dataclass
class Credentials:
    username: str
    password: str
```

Printing:

```python
Credentials("maya", "secret")
```

shows the password.

Use:

```python
@dataclass
class Credentials:
    username: str
    password: str = field(repr=False)
```

Also consider whether the password should be stored at all.

Maybe only a password hash belongs in memory.

Maybe even the hash should be excluded from `repr`.

Generated methods are convenient.

Security still requires judgment.

---

# Dataclass Design Checklist

When you design a dataclass, ask:

```text
What concept does this class name?
```

If you cannot name the concept clearly, you may just need a local tuple or dictionary.

Ask:

```text
Are these fields stable and known?
```

Dataclasses like known structure.

Ask:

```text
Should instances be mutable?
```

If not, use `frozen=True`.

Ask:

```text
Should equality compare all fields?
```

If not, configure fields or define equality yourself.

Ask:

```text
Should construction be positional or keyword-only?
```

For business objects, keyword-only is often better.

Ask:

```text
Are any defaults mutable?
```

Use `default_factory`.

Ask:

```text
Are there invariants?
```

Use `__post_init__`.

Ask:

```text
Are there secrets or huge fields?
```

Exclude them from `repr`.

Ask:

```text
Am I using generated methods deliberately?
```

Generated methods define object behavior.

They are not just convenience.

---

# A Full Example: A Small Todo Model

Let us build a small todo model.

Start with a value object for priority:

```python
from dataclasses import dataclass, field, replace
from datetime import date


@dataclass(frozen=True, order=True)
class Priority:
    level: int

    def __post_init__(self):
        if not 1 <= self.level <= 5:
            raise ValueError("priority must be between 1 and 5")
```

This is small, immutable, and ordered.

Now define a task:

```python
@dataclass(kw_only=True)
class Task:
    title: str
    priority: Priority
    due: date | None = None
    done: bool = False
    tags: list[str] = field(default_factory=list)
```

We use keyword-only construction because the object has several fields:

```python
task = Task(
    title="Write chapter",
    priority=Priority(2),
    tags=["book", "python"],
)
```

Add behavior:

```python
@dataclass(kw_only=True)
class Task:
    title: str
    priority: Priority
    due: date | None = None
    done: bool = False
    tags: list[str] = field(default_factory=list)

    def mark_done(self):
        self.done = True

    def is_overdue(self, today: date):
        return self.due is not None and self.due < today and not self.done
```

This is a mutable entity-like task.

It changes over time.

Now suppose we want immutable task updates instead:

```python
@dataclass(frozen=True, kw_only=True)
class Task:
    title: str
    priority: Priority
    due: date | None = None
    done: bool = False
    tags: tuple[str, ...] = ()

    def mark_done(self):
        return replace(self, done=True)

    def is_overdue(self, today: date):
        return self.due is not None and self.due < today and not self.done
```

Now:

```python
task = task.mark_done()
```

returns a new task.

Neither version is universally better.

The mutable version is simple.

The frozen version is easier to reason about in some workflows.

Dataclasses support both.

Your design chooses the meaning.

---

# A Full Example: Parsing Configuration Carefully

Suppose external configuration arrives as a dictionary:

```python
raw = {
    "host": "127.0.0.1",
    "port": "8000",
    "debug": "false",
}
```

Do not blindly do:

```python
config = ServerConfig(**raw)
```

because `port` and `debug` are strings.

Instead, parse at the boundary:

```python
@dataclass(frozen=True)
class ServerConfig:
    host: str
    port: int
    debug: bool = False

    def __post_init__(self):
        if not 1 <= self.port <= 65535:
            raise ValueError("invalid port")


def parse_bool(value: str) -> bool:
    normalized = value.strip().lower()
    if normalized in {"true", "1", "yes"}:
        return True
    if normalized in {"false", "0", "no"}:
        return False
    raise ValueError(f"invalid boolean: {value!r}")


def parse_config(raw: dict[str, str]) -> ServerConfig:
    return ServerConfig(
        host=raw["host"],
        port=int(raw["port"]),
        debug=parse_bool(raw.get("debug", "false")),
    )
```

The dataclass represents valid configuration.

The parsing function handles messy external input.

This separation is important:

```text
external data is messy
internal objects should be meaningful
```

Dataclasses help with the internal object.

They do not remove the need for boundary handling.

---

# A Full Example: Domain Events

Dataclasses are useful for events:

```python
from dataclasses import dataclass, field
from datetime import datetime, timezone
from uuid import UUID, uuid4


@dataclass(frozen=True, kw_only=True)
class UserRegistered:
    user_id: UUID
    email: str
    occurred_at: datetime = field(default_factory=lambda: datetime.now(timezone.utc))
    event_id: UUID = field(default_factory=uuid4)
```

This class has:

* required fields
* generated defaults
* immutability
* readable representation
* clear event identity

But think about equality.

Should two events with the same `user_id` and `email` compare equal if they have different `event_id` values?

Probably not.

The generated equality includes all comparable fields by default.

That may be exactly right here.

But if `occurred_at` should not affect equality, configure it:

```python
occurred_at: datetime = field(
    default_factory=lambda: datetime.now(timezone.utc),
    compare=False,
)
```

This is a domain decision.

The dataclass gives you the mechanism.

You provide the meaning.

---

# Performance Considerations

Dataclasses are ordinary Python classes with generated methods.

They are not automatically faster than handwritten classes.

They are usually chosen for clarity and maintainability.

Performance questions include:

* How many instances will exist?
* How often are they created?
* Are they mutable or frozen?
* Do they use `slots=True`?
* Do conversions like `asdict()` create deep copies?
* Are large fields included in `repr` or comparison?

For most programs, the performance difference is not the deciding factor.

Use dataclasses for clearer code.

Measure before optimizing.

If millions of instances are created, investigate `slots=True`, memory layout, and alternatives.

If large nested structures are converted frequently, avoid casual `asdict()`.

If comparisons are expensive, exclude fields that do not belong in comparison.

Performance follows design.

A clean object shape is easier to optimize later.

---

# Dataclasses and Static Type Checking

Static type checkers understand dataclasses.

They can often infer:

* generated initializer parameters
* field types
* missing arguments
* wrong argument types
* frozen assignment errors
* default value mismatches

Example:

```python
@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

A static checker can warn about:

```python
Point("left", "top")
```

even though Python may run it.

This is one reason dataclasses and type hints pair well.

Runtime Python stays dynamic.

Static tools add an optional checking layer.

Later in the book, static type checking gets its own treatment.

For now, see dataclasses as a bridge:

```text
runtime classes + annotated structure + generated methods + tool support
```

---

# Dataclasses and the Python Data Model

Dataclasses are a perfect transition into the next part of Volume II.

Why?

Because `@dataclass` is mostly about generating data model methods.

It can generate:

* `__init__`
* `__repr__`
* `__eq__`
* `__lt__`
* `__le__`
* `__gt__`
* `__ge__`
* `__hash__`
* `__match_args__`

These methods are not random names.

They are part of Python's data model.

They determine how objects:

* construct
* display
* compare
* sort
* hash
* match

Dataclasses are convenient because they generate these methods.

But to use dataclasses deeply, you must understand what those methods mean.

That is exactly where the book goes next.

---

# Practice: Convert a Handwritten Class

Start with:

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def __repr__(self):
        return f"Book(title={self.title!r}, author={self.author!r}, pages={self.pages!r})"
```

Convert it to a dataclass.

Then add:

* a default value for `pages`
* validation that pages cannot be negative
* a method called `is_long`

One possible solution:

```python
@dataclass
class Book:
    title: str
    author: str
    pages: int = 0

    def __post_init__(self):
        if self.pages < 0:
            raise ValueError("pages cannot be negative")

    def is_long(self):
        return self.pages >= 500
```

Notice that the dataclass handles plumbing.

You still write the meaningful behavior.

---

# Practice: Fix a Mutable Default

This class is wrong:

```python
@dataclass
class Playlist:
    name: str
    songs: list[str] = []
```

Fix it.

Solution:

```python
@dataclass
class Playlist:
    name: str
    songs: list[str] = field(default_factory=list)
```

Then test:

```python
first = Playlist("Morning")
second = Playlist("Evening")

first.songs.append("Song A")

assert second.songs == []
```

The test proves each instance gets a separate list.

---

# Practice: Design a Value Object

Create an immutable `EmailAddress` dataclass.

Requirements:

* it stores one string field called `value`
* it strips whitespace
* it lowercases the address
* it rejects values without `@`
* it is frozen

One possible solution:

```python
@dataclass(frozen=True)
class EmailAddress:
    value: str

    def __post_init__(self):
        normalized = self.value.strip().lower()
        if "@" not in normalized:
            raise ValueError("invalid email address")
        object.__setattr__(self, "value", normalized)
```

This example shows a frozen object with normalization.

The assignment must use `object.__setattr__` because the dataclass is frozen.

---

# Practice: Choose Equality

Suppose you have:

```python
@dataclass
class Session:
    session_id: str
    user_id: int
    last_seen_at: datetime
```

Ask:

```text
Should two sessions compare equal if last_seen_at differs?
```

If the answer is no, keep the default.

If the answer is yes, exclude `last_seen_at`:

```python
@dataclass
class Session:
    session_id: str
    user_id: int
    last_seen_at: datetime = field(compare=False)
```

Then write tests that express the chosen behavior.

Equality is not only technical.

It is semantic.

---

# Practice: Keyword-Only Construction

Design a dataclass for a payment:

```python
@dataclass(kw_only=True)
class Payment:
    amount: int
    currency: str
    captured: bool = False
    refunded: bool = False
```

Create it with keyword arguments:

```python
payment = Payment(
    amount=1000,
    currency="INR",
    captured=True,
)
```

Try to create it positionally:

```python
Payment(1000, "INR", True, False)
```

It should fail.

That failure is useful.

It protects call sites from unclear positional booleans.

---

# Practice: Dataclass or Normal Class?

For each concept, decide whether a dataclass is a good fit:

```text
Point(x, y)
DatabaseConnection
UserRegistrationCommand
Logger
Money(amount, currency)
Cache
SearchResult
ThreadPool
```

Likely choices:

```text
Point -> dataclass
DatabaseConnection -> normal class
UserRegistrationCommand -> dataclass
Logger -> normal class or existing logging object
Money -> frozen dataclass
Cache -> normal class
SearchResult -> dataclass
ThreadPool -> normal class
```

The pattern:

```text
structured data -> dataclass
lifecycle/resource/behavior-heavy object -> normal class
```

---

# Summary

Dataclasses reduce boilerplate for classes that primarily model structured data.

They are ordinary Python classes processed by the `@dataclass` decorator.

Fields are discovered from type annotations.

By default, dataclasses generate `__init__`, `__repr__`, and `__eq__`.

They can also generate ordering, hashing, pattern matching support, and slot-based layouts.

Defaults work like constructor defaults.

Mutable defaults require `default_factory`.

The `field()` function gives per-field control over initialization, representation, comparison, hashing, metadata, and keyword-only behavior.

`__post_init__` is the right place for validation, normalization, and derived setup after generated initialization.

Frozen dataclasses are useful for value objects, but freezing is not deep immutability.

`ClassVar` marks class-level data that should not become a field.

`InitVar` accepts initialization-only values that are passed to `__post_init__` but not stored as fields.

`asdict()`, `astuple()`, `replace()`, and `fields()` provide useful helper behavior, but each has design implications.

Dataclasses work with inheritance, ABCs, mixins, properties, and static type checking.

They are powerful because they sit directly on Python's class and data model.

Use dataclasses when a named concept has known structured fields.

Use normal classes when lifecycle, identity, dynamic behavior, or complex construction dominates.

The most important design lesson is:

```text
dataclasses generate mechanics, but you still own meaning
```

---

# Preview of Chapter 52

Chapter 51 completes Volume II Part I.

We have now built a complete object-oriented foundation:

* classes
* instances
* attributes
* methods
* encapsulation
* composition
* inheritance
* overriding
* MRO
* `super()`
* polymorphism
* duck typing
* ABCs
* mixins
* dataclasses

Next we begin Part II: the Python data model.

Chapter 52 studies dunder methods.

Dataclasses have already shown why these methods matter.

`@dataclass` can generate methods such as `__init__`, `__repr__`, `__eq__`, and `__hash__`.

But Python's data model is much larger than dataclasses.

It explains how objects participate in:

* construction
* display
* comparison
* arithmetic
* containers
* iteration
* function calls
* context management
* attribute access

The transition is natural:

```text
dataclasses generate selected data model methods
Chapter 52 teaches the data model itself
```

Once you understand dunder methods directly, Python objects become much less mysterious.

