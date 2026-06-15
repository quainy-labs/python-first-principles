# Chapter 56 — `__slots__`

---

# Learning Objectives

By the end of this chapter, you should understand:

* What `__slots__` does.
* How normal instances store attributes in `__dict__`.
* Why slotted instances may not have `__dict__`.
* How slots restrict arbitrary new attributes.
* How slots can reduce memory usage.
* How slots can affect attribute lookup.
* Why slots are implemented with descriptors.
* How slots interact with inheritance.
* Why subclasses may regain `__dict__` unless they define slots too.
* How to allow dynamic attributes with `"__dict__"`.
* How to allow weak references with `"__weakref__"`.
* Why slot names cannot safely be reused in subclasses.
* How dataclasses support slots.
* When slots are worth using.
* When slots make code unnecessarily rigid.

Chapter 55 studied properties, static methods, and class methods.

Properties manage attribute access.

Slots change attribute storage.

That distinction matters.

When you write:

```python
obj.name = "Maya"
```

Python usually stores the attribute in the instance dictionary:

```python
obj.__dict__
```

But not all objects have an instance dictionary.

Slots let a class declare a fixed set of instance attributes.

Example:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

Now each `Point` instance has space for `x` and `y`.

It does not automatically have a normal `__dict__`.

That changes what attributes can exist and how they are stored.

Slots are useful.

Slots are also easy to overuse.

This chapter is about understanding the mechanism before reaching for it.

---

# Normal Instance Attribute Storage

Start with an ordinary class:

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
```

Create an instance:

```python
user = User("Maya", "maya@example.com")
```

Most ordinary instances store attributes in a dictionary:

```python
print(user.__dict__)
```

Output:

```python
{'name': 'Maya', 'email': 'maya@example.com'}
```

This dictionary is flexible.

You can add new attributes at runtime:

```python
user.active = True
print(user.__dict__)
```

Output:

```python
{'name': 'Maya', 'email': 'maya@example.com', 'active': True}
```

This flexibility is one of Python's strengths.

But it has costs:

* each instance needs dictionary storage
* attribute names are stored dynamically
* typos can create new attributes accidentally
* memory overhead can matter for millions of objects

Slots trade some flexibility for a fixed layout.

---

# A First Slotted Class

Define slots with a class variable named `__slots__`:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

Now create:

```python
point = Point(10, 20)
```

These assignments work:

```python
point.x = 10
point.y = 20
```

But this fails:

```python
point.z = 30
```

because `z` is not listed in `__slots__`.

You will typically see:

```python
AttributeError
```

Also:

```python
point.__dict__
```

usually fails because the instance does not have a dictionary.

The class declared:

```text
instances of this class have these named slots
```

not:

```text
instances can accept arbitrary attributes
```

---

# Slots Are About Storage, Not Privacy

Slots do not make attributes private.

This works:

```python
point.x
point.x = 99
```

Slots do not prevent users from reading or writing declared attributes.

They prevent arbitrary undeclared attribute names when no `__dict__` exists.

Slots are not an encapsulation feature.

They are a storage layout feature.

If you need validation, use:

* properties
* descriptors
* `__setattr__`
* constructor validation
* domain methods

If you need memory savings or fixed attributes, consider slots.

The mental model:

```text
property -> controls access behavior
__slots__ -> controls instance storage layout
```

---

# Why Slots Can Save Memory

A dictionary is flexible, but it costs memory.

If you create a handful of objects, that cost rarely matters.

If you create millions, it may matter a lot.

Example:

```python
class NormalPoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y


class SlottedPoint:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

`NormalPoint` instances usually have per-instance dictionaries.

`SlottedPoint` instances do not unless requested.

That can reduce memory.

But exact memory savings depend on:

* Python implementation
* Python version
* object shape
* number of attributes
* number of instances
* inheritance
* whether `__dict__` or `__weakref__` is included

Do not assume a fixed percentage.

Measure in the target environment.

Slots are an optimization tool.

Optimization should be guided by evidence.

---

# Attribute Lookup and Slots

Slots can also improve attribute lookup speed.

Instead of looking up a name in an instance dictionary, Python can use slot descriptors that know where the value lives in the object layout.

But the speed difference is usually not the first reason to use slots.

For most application code, readability and design clarity matter more.

Use slots for speed only when:

* profiling shows attribute access matters
* many objects are involved
* the object layout is stable
* reduced flexibility is acceptable

Do not sprinkle slots everywhere hoping for free performance.

Slots are a design constraint.

Treat them as one.

---

# Slots Are Implemented with Descriptors

Chapter 54 now pays off.

When you write:

```python
class Point:
    __slots__ = ("x", "y")
```

Python creates descriptor objects on the class for `x` and `y`.

Those descriptors manage access to slot storage.

This is why you should not define class attributes with the same names as slots.

This is wrong:

```python
class Point:
    __slots__ = ("x", "y")
    x = 0
```

The class attribute `x` would conflict with the slot descriptor for `x`.

Slot names need their descriptor entries in the class.

If you want defaults, assign them in `__init__`:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y
```

Slots are not just a list of allowed names.

They are implemented by class-level machinery.

---

# No Automatic `__dict__`

A slotted class usually prevents automatic instance dictionaries.

Example:

```python
class Point:
    __slots__ = ("x", "y")
```

Then:

```python
point = Point()
point.x = 1
point.y = 2
```

works.

But:

```python
point.label = "origin"
```

fails.

And:

```python
point.__dict__
```

fails.

If you want slots but also dynamic attributes, include `"__dict__"`:

```python
class FlexiblePoint:
    __slots__ = ("x", "y", "__dict__")
```

Now:

```python
point = FlexiblePoint()
point.x = 1
point.y = 2
point.label = "origin"
```

works.

But adding `__dict__` reduces the memory benefit.

You are asking for both:

```text
fixed slots for some attributes
dynamic dictionary for others
```

That can be useful.

But if you need full dynamic flexibility, slots may not be buying much.

---

# No Automatic Weak References

In Chapter 37, we studied weak references.

Slotted classes do not automatically support weak references unless a parent provides support or the class includes `"__weakref__"`.

Example:

```python
import weakref


class Node:
    __slots__ = ("name",)

    def __init__(self, name):
        self.name = name
```

This may fail:

```python
node = Node("root")
reference = weakref.ref(node)
```

To support weak references:

```python
class Node:
    __slots__ = ("name", "__weakref__")

    def __init__(self, name):
        self.name = name
```

Now weak references can work:

```python
reference = weakref.ref(Node("root"))
```

Use `"__weakref__"` when instances need to participate in weak-reference-based structures.

Examples:

* caches
* observer systems
* parent-child graphs
* external registries

If you do not need weak references, leave it out.

---

# Slot Declarations Can Be Strings or Iterables

For one slot, you might see:

```python
class User:
    __slots__ = "name"
```

This works because a string is allowed.

But it can be confusing.

Prefer a tuple:

```python
class User:
    __slots__ = ("name",)
```

The trailing comma matters.

Without it:

```python
("name")
```

is just a string in parentheses.

With it:

```python
("name",)
```

is a one-item tuple.

For multiple slots:

```python
class User:
    __slots__ = ("name", "email")
```

You may also see lists:

```python
__slots__ = ["name", "email"]
```

Tuples are common because the slot declaration itself is not meant to change.

---

# Slots Do Not Initialize Values

Declaring a slot does not assign a value.

Example:

```python
class Point:
    __slots__ = ("x", "y")
```

Create:

```python
point = Point()
```

This fails:

```python
print(point.x)
```

because `x` has not been assigned yet.

Slots reserve space for attributes.

They do not fill that space with defaults.

Initialize in `__init__`:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y
```

Now:

```python
Point().x
```

works.

This is another reason slot names should not be confused with default values.

---

# Slots and Typing

Slots do not declare types.

This:

```python
class Point:
    __slots__ = ("x", "y")
```

does not say that `x` and `y` must be numbers.

You can still write:

```python
point = Point()
point.x = "left"
point.y = []
```

Slots restrict names.

They do not validate values.

If you want type hints, write annotations:

```python
class Point:
    __slots__ = ("x", "y")

    x: float
    y: float
```

If you want runtime validation, use properties, descriptors, or explicit checks.

Slots answer:

```text
which attributes may exist?
```

They do not answer:

```text
what values are valid?
```

---

# Slots and Properties

Slots and properties can work together.

Example:

```python
class Product:
    __slots__ = ("_price",)

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

Here the public property is:

```python
price
```

The stored slot is:

```python
_price
```

Do not put both the property name and backing name in slots:

```python
__slots__ = ("price", "_price")
```

if `price` is also a property.

The property lives as a class attribute named `price`.

A slot with the same name would conflict with that class attribute.

Use a backing slot:

```python
__slots__ = ("_price",)
```

This pattern is common when you want both validation and slotted storage.

---

# Slots and Descriptors

Slots create descriptors.

Custom descriptors can also be used in slotted classes, but storage must be planned.

This descriptor stores in a private attribute:

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("must be positive")
        setattr(obj, self.private_name, value)
```

Use with slots:

```python
class Product:
    __slots__ = ("_price",)

    price = PositiveNumber()

    def __init__(self, price):
        self.price = price
```

The descriptor manages public `price`.

The slot stores `_price`.

If you forget `_price` in `__slots__`, assignment fails because the slotted instance has nowhere to store it.

Descriptors and slots are compatible, but the storage name must exist.

---

# Slots and Inheritance

Inheritance is where slots become tricky.

Start with a slotted base:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

Subclass without slots:

```python
class ColoredPoint(Point):
    def __init__(self, x, y, color):
        super().__init__(x, y)
        self.color = color
```

This may work.

Why?

Because a subclass without `__slots__` usually gets an instance dictionary.

That dictionary allows `color`.

It also means the subclass instances no longer have the same fixed-layout benefit.

If you want the subclass to remain slotted, define slots there too:

```python
class ColoredPoint(Point):
    __slots__ = ("color",)

    def __init__(self, x, y, color):
        super().__init__(x, y)
        self.color = color
```

Now `ColoredPoint` instances have slots for:

* `x`
* `y`
* `color`

and no automatic `__dict__`, assuming no parent adds one.

---

# Empty Slots in Subclasses

Sometimes a subclass adds methods but no new attributes.

Example:

```python
class Point:
    __slots__ = ("x", "y")


class PrintablePoint(Point):
    __slots__ = ()

    def __repr__(self):
        return f"PrintablePoint({self.x!r}, {self.y!r})"
```

Why define:

```python
__slots__ = ()
```

Because it tells Python:

```text
this subclass adds no new instance attributes and should not get a __dict__
```

Without it, the subclass may regain a dictionary.

That defeats part of the purpose of the slotted base.

For slotted hierarchies, every subclass should make an explicit decision:

```text
new slots
empty slots
or dynamic dictionary
```

Silence is often not what you want.

---

# Inheriting from a Non-Slotted Base

If a class inherits from a normal base class, slots cannot remove the base's dictionary behavior.

Example:

```python
class Base:
    pass


class Child(Base):
    __slots__ = ("x",)
```

Instances of `Child` may still have `__dict__` because `Base` provides normal instance dictionary support.

Slots in the child add slot storage for `x`.

They do not erase storage layout inherited from the base.

This matters when trying to optimize an existing hierarchy.

You cannot always add slots at the leaf and expect full memory savings.

The whole inheritance chain matters.

---

# Do Not Repeat Slot Names in Subclasses

If a base class defines a slot:

```python
class Base:
    __slots__ = ("x",)
```

do not redefine the same slot in a subclass:

```python
class Child(Base):
    __slots__ = ("x",)
```

This can make the base slot inaccessible in normal ways and leads to undefined or confusing behavior.

Slots declared in parents are already available in children.

Only list additional slot names in the subclass:

```python
class Child(Base):
    __slots__ = ("y",)
```

A slotted hierarchy is cumulative:

```text
base slots + child slots + grandchild slots
```

Do not redeclare names already owned by a parent slot.

---

# Multiple Inheritance and Slots

Multiple inheritance with slots is possible, but constrained.

The hard part is object layout.

Python must know how to lay out all the slotted storage in memory.

If multiple base classes define non-empty slot layouts, conflicts can occur.

Example:

```python
class A:
    __slots__ = ("a",)


class B:
    __slots__ = ("b",)


class C(A, B):
    __slots__ = ()
```

This kind of design can raise `TypeError` because multiple bases may contribute incompatible instance layouts.

In real code, avoid complex multiple inheritance with non-empty slots unless you deeply understand the layout constraints.

Mixins that do not add instance storage can use:

```python
__slots__ = ()
```

Example:

```python
class ReprMixin:
    __slots__ = ()

    def __repr__(self):
        return f"{type(self).__name__}()"
```

Empty slots tell Python that the mixin does not require instance storage.

That makes it friendlier in slotted hierarchies.

---

# Slots and Mixins

If a mixin assumes instances have a dictionary, it may break with slots.

Example:

```python
class DictMixin:
    def as_dict(self):
        return self.__dict__.copy()
```

This fails for slotted objects without `__dict__`.

Better:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def as_dict(self):
        return {
            name: getattr(self, name)
            for name in self.__slots__
        }
```

But even this is simplistic because inherited slots may live in base classes.

A more robust slot-aware utility has to walk the MRO:

```python
def slot_names(cls):
    names = []
    for base in cls.__mro__:
        slots = getattr(base, "__slots__", ())
        if isinstance(slots, str):
            slots = (slots,)
        names.extend(
            name for name in slots
            if name not in {"__dict__", "__weakref__"}
        )
    return names
```

Slots change assumptions.

Any code that assumes `obj.__dict__` may fail.

---

# Slots and Pickling

Serialization tools may need to know how to handle slotted objects.

Many built-in mechanisms understand slots, but custom serialization code often assumes:

```python
obj.__dict__
```

That fails for slotted instances without dictionaries.

If you write serializers, think about:

* normal instance dictionaries
* slotted attributes
* inherited slots
* properties
* descriptors
* private backing attributes

For simple slotted classes, explicit conversion is often best:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def to_dict(self):
        return {"x": self.x, "y": self.y}
```

Explicit serialization is boring in the best way.

It states the public data shape clearly.

---

# Slots and Dataclasses

Dataclasses support slots:

```python
from dataclasses import dataclass


@dataclass(slots=True)
class Point:
    x: float
    y: float
```

This generates a slotted dataclass.

You still get dataclass features:

* generated `__init__`
* generated `__repr__`
* generated `__eq__`
* field support

But instances use slots for fields.

Example:

```python
point = Point(1, 2)
point.z = 3
```

fails because `z` is not a declared field slot.

This is a convenient way to get dataclass ergonomics with slotted storage.

Use it when:

* the data shape is fixed
* many instances may be created
* dynamic attributes are not needed
* the reduced flexibility is acceptable

---

# Dataclasses and `weakref_slot`

Slotted dataclasses can support weak references with:

```python
@dataclass(slots=True, weakref_slot=True)
class Node:
    name: str
```

This adds weak-reference support.

It is an error to request `weakref_slot=True` without `slots=True`.

Why?

Because `weakref_slot` is specifically about adding the weak-reference slot to a slotted layout.

For normal dataclasses with instance dictionaries, weak-reference behavior follows the normal object layout.

Use `weakref_slot=True` when:

* the dataclass is slotted
* instances must be weak-referenceable

Most dataclasses do not need it.

---

# Dataclasses and Inherited Slots

Dataclass slot behavior has version-specific details.

Modern Python avoids duplicating inherited slot names when generating slots for dataclasses.

The practical rule:

```text
do not use __slots__ to discover dataclass fields
```

Use:

```python
from dataclasses import fields
```

Example:

```python
for field in fields(Point):
    print(field.name)
```

Slots are an implementation/storage detail.

Dataclass fields are the data model declared by the dataclass.

They overlap, but they are not the same API.

If you want dataclass field names, ask the dataclass module.

---

# Slots and Defaults

You cannot set slot defaults by writing class attributes with the same names.

Bad:

```python
class Point:
    __slots__ = ("x", "y")
    x = 0
    y = 0
```

The class attributes conflict with the slot descriptors.

Use `__init__`:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y
```

Or use a dataclass:

```python
@dataclass(slots=True)
class Point:
    x: float = 0
    y: float = 0
```

The dataclass machinery knows how to combine field defaults with generated initialization.

Manual slots require manual initialization.

---

# Slots and `__class__` Assignment

Python can sometimes let an object change its class by assigning:

```python
obj.__class__ = OtherClass
```

With slots, this is heavily constrained.

The object layouts must be compatible.

If two classes have different slot layouts, assignment can fail.

This is an advanced corner.

Most programs should not assign `__class__` directly.

The slots lesson is simple:

```text
slots affect object layout, and object layout affects advanced runtime behavior
```

This is another reason slots should be used deliberately.

---

# Measuring Slots

If you use slots for memory, measure.

A simple starting point:

```python
import sys


normal = NormalPoint(1, 2)
slotted = SlottedPoint(1, 2)

print(sys.getsizeof(normal))
print(sys.getsizeof(normal.__dict__))
print(sys.getsizeof(slotted))
```

Be careful interpreting this.

`sys.getsizeof` reports the size of one object, not necessarily the full reachable object graph.

For normal instances, the dictionary is separate.

For contained values, those values have their own memory too.

For serious analysis, use profiling tools and realistic workloads.

Still, simple measurement is better than guessing.

Slots are often most useful when a small class has many instances.

Example:

```text
millions of points, tokens, nodes, events, cells, records
```

For a few dozen service objects, slots rarely matter.

---

# Slots Are Not Always Faster in Meaningful Ways

It is easy to hear:

```text
slots are faster
```

and overapply them.

Attribute lookup may improve.

Memory may improve.

But your program may spend time elsewhere:

* database calls
* network requests
* parsing
* rendering
* algorithmic work
* file I/O
* logging

If slots save 2 percent of attribute lookup time in code that consumes 1 percent of runtime, the program will not feel faster.

Use slots for performance only after you understand the bottleneck.

Otherwise, use them for design only when fixed attributes are genuinely useful.

---

# Slots for Typo Prevention

Slots can catch accidental attribute creation.

Normal class:

```python
class User:
    def __init__(self, email):
        self.email = email
```

Typo:

```python
user.emial = "wrong"
```

This creates a new attribute.

Slotted class:

```python
class User:
    __slots__ = ("email",)

    def __init__(self, email):
        self.email = email
```

Typo:

```python
user.emial = "wrong"
```

raises `AttributeError`.

This can be helpful.

But do not use slots only as a typo checker.

Static type checkers, tests, linters, and good design also help.

Slots impose runtime layout constraints.

Typo prevention is a side benefit, not always enough justification by itself.

---

# Slots and Dynamic Features

Some Python patterns expect dynamic attributes:

```python
obj.debug_info = ...
obj.cache = ...
obj.request_id = ...
```

Some libraries attach attributes to objects.

Some tests monkey patch instance attributes.

Some debugging tools inspect `__dict__`.

Slots can break those patterns.

This is not a reason to avoid slots forever.

It is a reason to know your object usage.

Before adding slots to a public class, ask:

```text
Do users attach attributes to instances?
Do tools inspect __dict__?
Do tests monkey patch instances?
Do subclasses need flexibility?
Is this a stable data object or a dynamic framework object?
```

Slots are friendlier for small, stable value-like objects than for highly dynamic framework objects.

---

# Slots and Public APIs

Adding slots to a public class can be a breaking change.

Previously, users may have written:

```python
obj.extra = value
```

After slots, that may fail.

Previously, users may have read:

```python
obj.__dict__
```

After slots, that may fail.

Even if those users were relying on internals, public ecosystem code often does.

For internal classes, you can change more freely.

For published libraries, treat slots as part of the public behavior.

Document the change.

Consider including `"__dict__"` if dynamic attributes must remain supported.

---

# Slots and Immutable-Looking Objects

Slots do not make objects immutable.

Example:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

This is still mutable:

```python
point = Point(1, 2)
point.x = 99
```

Slots restrict names.

They do not prevent assignment to declared names.

If you want immutability, use another design:

```python
@dataclass(frozen=True, slots=True)
class Point:
    x: float
    y: float
```

Even then, remember frozen dataclasses are not deep immutability if fields reference mutable objects.

Slots and immutability are separate concepts.

---

# Slots and `__setattr__`

You can combine slots with `__setattr__`, but be careful.

Example:

```python
class FrozenAfterInit:
    __slots__ = ("name", "_locked")

    def __init__(self, name):
        self._locked = False
        self.name = name
        self._locked = True

    def __setattr__(self, name, value):
        if getattr(self, "_locked", False):
            raise AttributeError("object is locked")
        super().__setattr__(name, value)
```

This makes assignment fail after initialization.

But this is advanced.

It is easy to create initialization bugs or inheritance problems.

For ordinary immutable data, prefer:

```python
@dataclass(frozen=True, slots=True)
```

or a simpler explicit design.

Use `__setattr__` only when you need class-wide assignment control.

---

# Slots and Representation

If you write a generic `__repr__` that depends on `__dict__`, it will fail:

```python
class ReprFromDict:
    def __repr__(self):
        return f"{type(self).__name__}({self.__dict__!r})"
```

Slotted objects without dictionaries do not work with that.

Write explicit representations:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point({self.x!r}, {self.y!r})"
```

Or use dataclasses:

```python
@dataclass(slots=True)
class Point:
    x: float
    y: float
```

The generated `__repr__` handles the fields.

---

# A Practical Example: Many Points

Suppose you are processing millions of points:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def translate(self, dx, dy):
        self.x += dx
        self.y += dy
```

This can be a good slots use case:

* small fixed shape
* many instances
* no dynamic attributes needed
* performance and memory may matter

If the class grows:

```python
point.color = "red"
point.label = "start"
point.metadata = {}
```

then slots become less attractive.

The right design depends on object role.

Small numeric record?

Slots may help.

Flexible user-facing object?

Maybe not.

---

# A Practical Example: Tree Nodes

Tree nodes are common candidates:

```python
class Node:
    __slots__ = ("value", "children", "__weakref__")

    def __init__(self, value):
        self.value = value
        self.children = []
```

Why include `"__weakref__"`?

Maybe parent pointers or caches use weak references.

If not, leave it out.

If you add parent references:

```python
class Node:
    __slots__ = ("value", "children", "parent", "__weakref__")
```

Think about reference cycles.

Slots do not eliminate cycles.

If `parent` and `children` point to each other, objects can still form cycles.

Garbage collection rules still matter.

Slots change storage.

They do not change graph semantics.

---

# A Practical Example: Tokens

Compilers, parsers, and text processors often create many token objects.

Example:

```python
class Token:
    __slots__ = ("kind", "text", "line", "column")

    def __init__(self, kind, text, line, column):
        self.kind = kind
        self.text = text
        self.line = line
        self.column = column
```

This is a strong slots candidate.

Token shape is fixed.

Many token instances may exist.

Dynamic attributes are rarely needed.

A dataclass version:

```python
@dataclass(frozen=True, slots=True)
class Token:
    kind: str
    text: str
    line: int
    column: int
```

This is often even better:

* concise
* immutable
* slotted
* readable representation
* value equality

This shows how modern Python combines tools.

Slots are not isolated.

They work with dataclasses, typing, and object design.

---

# A Practical Example: Avoid Slots

Consider a request context object:

```python
class RequestContext:
    def __init__(self, request_id):
        self.request_id = request_id
```

During debugging, middleware may attach:

```python
context.user = user
context.trace_id = trace_id
context.feature_flags = flags
context.timings = {}
```

This object is dynamic by nature.

Slots might fight the design.

You could list many slots, but the class may keep growing.

Here, a normal instance dictionary may be more appropriate.

Optimization should not erase useful flexibility.

---

# Common Mistake: Thinking Slots Are Just Documentation

This is not merely documentation:

```python
__slots__ = ("x", "y")
```

It changes runtime behavior.

It affects:

* instance dictionaries
* dynamic attributes
* weak references
* inheritance
* descriptors
* serialization assumptions
* memory layout

If you only want to document fields, use:

* annotations
* dataclasses
* docstrings
* type aliases
* comments when necessary

Slots are executable class configuration.

They are not just a list for readers.

---

# Common Mistake: Forgetting Slots in Subclasses

Base:

```python
class Base:
    __slots__ = ("x",)
```

Subclass:

```python
class Child(Base):
    pass
```

This may give `Child` instances a dictionary.

If you want no new attributes:

```python
class Child(Base):
    __slots__ = ()
```

If you want one new attribute:

```python
class Child(Base):
    __slots__ = ("y",)
```

In slotted hierarchies, every subclass should be explicit.

---

# Common Mistake: Using Slots with Dynamic Libraries

Some libraries expect objects to have `__dict__`.

Examples may include:

* serializers
* ORMs
* test tools
* debug tools
* monkey-patching utilities
* template systems

Modern libraries may handle slots well.

But not all code does.

Before making a class slotted, check how it is used.

If the object crosses framework boundaries, slots may cause compatibility surprises.

---

# Common Mistake: Expecting Slots to Validate Types

This class:

```python
class User:
    __slots__ = ("age",)
```

does not prevent:

```python
user.age = "old"
```

Slots only allow the name `age`.

They do not validate the value assigned to it.

Use a property:

```python
class User:
    __slots__ = ("_age",)

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if not isinstance(value, int):
            raise TypeError("age must be an integer")
        self._age = value
```

or a descriptor if the validation should be reused.

---

# Common Mistake: Using Slots Everywhere

Slots can make classes less flexible.

They can complicate:

* inheritance
* debugging
* serialization
* monkey patching
* weak references
* dynamic extension
* framework integration

For many classes, a normal instance dictionary is better.

Use slots when the benefits are clear:

* many instances
* stable fixed attributes
* memory pressure
* typo prevention is valuable
* dynamic attributes are unwanted

Do not use slots as a style preference alone.

The best Python code is not the code with the most special mechanisms.

It is the code whose mechanisms match the problem.

---

# Design Checklist

Before adding slots, ask:

```text
Will many instances of this class exist?
```

If no, memory savings may not matter.

Ask:

```text
Is the attribute set stable?
```

If no, slots may be painful.

Ask:

```text
Do users or tools rely on __dict__?
```

If yes, avoid slots or include `"__dict__"`.

Ask:

```text
Do instances need weak references?
```

If yes, include `"__weakref__"`.

Ask:

```text
Is this class part of an inheritance hierarchy?
```

If yes, plan slots across the hierarchy.

Ask:

```text
Do I need validation?
```

If yes, slots alone are not enough.

Ask:

```text
Would a slotted dataclass express this better?
```

For simple data objects, it often will.

---

# Practice: Create a Slotted Point

Create a slotted `Point` class.

Requirements:

* attributes `x` and `y`
* no arbitrary new attributes
* useful representation

Solution:

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point({self.x!r}, {self.y!r})"
```

Test:

```python
point = Point(1, 2)
assert point.x == 1
assert point.y == 2
```

Then:

```python
point.z = 3
```

should raise `AttributeError`.

---

# Practice: Add Weak Reference Support

Create a slotted `Node` class that supports weak references.

Solution:

```python
class Node:
    __slots__ = ("name", "__weakref__")

    def __init__(self, name):
        self.name = name
```

Test:

```python
import weakref

node = Node("root")
reference = weakref.ref(node)

assert reference() is node
```

Without `"__weakref__"`, this may fail.

---

# Practice: Slotted Subclass

Given:

```python
class Point:
    __slots__ = ("x", "y")
```

Create a subclass that adds `color` but does not allow arbitrary attributes.

Solution:

```python
class ColoredPoint(Point):
    __slots__ = ("color",)

    def __init__(self, x, y, color):
        self.x = x
        self.y = y
        self.color = color
```

Test:

```python
point = ColoredPoint(1, 2, "red")
point.label = "start"
```

The last line should raise `AttributeError`.

---

# Practice: Empty Slots in a Mixin

Create a mixin that adds a method but no instance attributes.

Solution:

```python
class AsTupleMixin:
    __slots__ = ()

    def as_tuple(self):
        return tuple(
            getattr(self, name)
            for name in self.__slots__
        )
```

This specific implementation is too simplistic for inherited slots, but the important piece is:

```python
__slots__ = ()
```

The mixin declares that it adds no instance storage.

In real slotted hierarchies, slot-aware mixins need careful design.

---

# Practice: Dataclass with Slots

Create a slotted dataclass for a token.

Solution:

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class Token:
    kind: str
    text: str
    line: int
    column: int
```

Test:

```python
token = Token("NAME", "value", 1, 5)
assert token.kind == "NAME"
```

This should fail:

```python
token.extra = "anything"
```

because the dataclass is slotted and frozen.

---

# Practice: Choose Slots or Not

For each class, decide whether slots are likely useful:

```text
Point with x and y, millions of instances
RequestContext where middleware attaches attributes
Token in a parser
DatabaseConnection
TreeNode in a large tree
User model used by an ORM
Small one-off service object
Frozen dataclass representing coordinates
```

Likely answers:

```text
Point with many instances -> yes
RequestContext -> probably no
Token -> yes
DatabaseConnection -> probably no
TreeNode -> maybe yes
User model used by ORM -> depends on ORM support
Small service object -> probably no
Frozen coordinate dataclass -> yes, if many instances or fixed layout matters
```

The decision depends on object role, not fashion.

---

# Summary

`__slots__` lets a class declare fixed instance attribute names.

Slotted instances usually do not get automatic `__dict__` or `__weakref__` storage unless explicitly requested or inherited.

Without `__dict__`, instances cannot receive arbitrary new attributes.

Without `__weakref__`, instances may not support weak references.

Slots can reduce memory usage when many instances exist.

Slots can sometimes improve attribute lookup speed.

Slots are implemented using class-level descriptors.

That is why slot names conflict with class attributes of the same name.

Slots do not initialize values.

Slots do not validate types.

Slots do not make objects immutable.

Slots interact carefully with inheritance.

Subclasses of slotted classes should define their own slots or `__slots__ = ()` if they add no storage.

Dataclasses support slots with `@dataclass(slots=True)`.

Slotted dataclasses can support weak references with `weakref_slot=True`.

Use slots when the class has a stable shape and many instances, or when preventing arbitrary attributes is valuable.

Avoid slots when dynamic attributes, framework integration, easy debugging, or flexible extension matter more.

The design principle is:

```text
slots are a storage-layout decision, not a general-purpose quality upgrade
```

Use them when the layout constraint is worth the benefits.

---

# Preview of Chapter 57

Chapter 56 studied `__slots__`, which changes how instance attributes are stored.

Next we study metaclasses.

Metaclasses change how classes themselves are created.

So far, we have customized:

* instances
* attributes
* methods
* operators
* storage

Metaclasses move one level up.

They let Python customize class creation itself.

Chapter 57 will explain:

* why classes are objects
* what `type` does
* how class creation works
* what a metaclass is
* how metaclasses relate to inheritance
* when `__init_subclass__` is simpler than a metaclass
* why most code should avoid custom metaclasses
* where metaclasses appear in frameworks

The transition is:

```text
slots customize instance layout
metaclasses customize class creation
```

Metaclasses are powerful, but they should be rare.

The goal is not to make them feel magical.

The goal is to know what they are, recognize them in advanced code, and choose simpler tools when possible.

