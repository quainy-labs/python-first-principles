# Chapter 53 — Operator Overloading

---

# Learning Objectives

By the end of this chapter, you should understand:

* What operator overloading means in Python.
* How operators connect to dunder methods.
* Why `a + b` may call `__add__`, `__radd__`, or `__iadd__`.
* How unary operators such as `-x` and `+x` work.
* How comparison operators use rich comparison methods.
* Why `NotImplemented` is central to binary operators.
* How reflected operations support mixed-type expressions.
* How in-place operators differ from normal binary operators.
* Why mutable and immutable objects should handle `+=` differently.
* How equality and hashing interact with overloaded operators.
* How to design numeric-like types responsibly.
* How to design collection-like operator behavior responsibly.
* When operators make code clearer.
* When named methods are better than operators.
* Which common operator-overloading mistakes to avoid.

Chapter 52 introduced dunder methods as protocol hooks.

This chapter focuses on one family of those hooks: operators.

When you write:

```python
1 + 2
```

Python knows how to add integers.

When you write:

```python
"py" + "thon"
```

Python knows how to concatenate strings.

When you write:

```python
[1, 2] + [3, 4]
```

Python knows how to concatenate lists.

The same operator symbol can mean different things for different types.

That is operator overloading.

The operator is overloaded because its meaning depends on the operand types.

Python lets your own classes participate in this system.

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

Now:

```python
Vector(1, 2) + Vector(3, 4)
```

returns:

```python
Vector(4, 6)
```

This is powerful.

It can make custom objects feel natural.

It can also make code confusing if operators are used for surprising behavior.

The rule for this chapter is:

```text
overload operators only when the meaning is natural, predictable, and useful
```

---

# Operators Are Syntax Over Protocols

Operators look like syntax.

But in Python, most operators dispatch to special methods.

For example:

```python
a + b
```

is connected to:

```python
a.__add__(b)
```

This is simplified because Python also considers reflected methods and type relationships.

But the mental model is right:

```text
operator syntax asks objects how to perform an operation
```

Examples:

```text
a + b      -> addition protocol
a - b      -> subtraction protocol
a * b      -> multiplication protocol
a / b      -> true division protocol
a // b     -> floor division protocol
a % b      -> modulo protocol
a ** b     -> power protocol
a == b     -> equality protocol
a < b      -> ordering protocol
a += b     -> in-place addition protocol
```

Objects implement these protocols through dunder methods.

This lets user-defined classes behave like built-in types when the behavior makes sense.

---

# Why Operator Overloading Exists

Operator overloading exists because some concepts are naturally expressed with operators.

Vectors:

```python
velocity + acceleration
```

Money:

```python
subtotal + tax
```

Dates and durations:

```python
deadline + duration
```

Sets:

```python
allowed & requested
```

Paths:

```python
base_path / "chapter.md"
```

Matrices:

```python
a @ b
```

In these cases, operators can make code more readable.

But operators are compact.

Compact syntax carries less explanation.

This is good when the meaning is obvious.

It is bad when the meaning is private to the author.

This is clear:

```python
Vector(1, 2) + Vector(3, 4)
```

This is not:

```python
email_sender + message
```

If `+` sends an email, the operator is hiding behavior.

Named methods are better:

```python
email_sender.send(message)
```

Operator overloading should make code feel more like the domain.

It should not turn code into a puzzle.

---

# The Main Binary Operator Methods

Binary operators work with two operands.

Common binary operator methods include:

```text
a + b    -> __add__
a - b    -> __sub__
a * b    -> __mul__
a / b    -> __truediv__
a // b   -> __floordiv__
a % b    -> __mod__
a ** b   -> __pow__
a @ b    -> __matmul__
a << b   -> __lshift__
a >> b   -> __rshift__
a & b    -> __and__
a ^ b    -> __xor__
a | b    -> __or__
```

These are called binary because each operation has a left operand and a right operand.

Example:

```python
a + b
```

`a` is the left operand.

`b` is the right operand.

The left operand gets the first chance to handle the operation.

For:

```python
a + b
```

Python may ask:

```python
a.__add__(b)
```

If that cannot handle the operation, Python may ask the right operand through a reflected method.

We will study that soon.

---

# A Small Vector Type

A vector is a good teaching example because vector addition has a natural meaning.

Start with:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

Add a useful representation:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"
```

Now add vector addition:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"

    def __add__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)
```

Usage:

```python
first = Vector(1, 2)
second = Vector(3, 4)

print(first + second)
```

Output:

```python
Vector(4, 6)
```

This is a good overload.

The meaning of `+` is obvious.

It returns a new vector.

It does not mutate either operand.

---

# Return a New Object for Value-Like Operations

For immutable or value-like objects, binary operators usually return a new object.

Example:

```python
Vector(1, 2) + Vector(3, 4)
```

should not modify either original vector.

It should create:

```python
Vector(4, 6)
```

This matches how numbers work:

```python
a = 10
b = a + 5
```

`a` remains `10`.

`b` becomes `15`.

Strings also return new objects:

```python
name = "py"
full = name + "thon"
```

`name` remains `"py"`.

`full` becomes `"python"`.

For value-like custom objects, follow that expectation.

When users see:

```python
c = a + b
```

they usually expect `a` and `b` to survive unchanged.

---

# `NotImplemented` in Binary Operators

Suppose someone writes:

```python
Vector(1, 2) + 10
```

Our vector does not know how to add an integer.

Inside `__add__`, we should not return `False`.

We should not return `None`.

We should not raise `NotImplementedError`.

We should usually return `NotImplemented`:

```python
def __add__(self, other):
    if type(other) is not Vector:
        return NotImplemented
    return Vector(self.x + other.x, self.y + other.y)
```

`NotImplemented` tells Python:

```text
this method does not support this operand combination
```

Then Python can try another method or raise a suitable `TypeError`.

If no supported operation is found, the user may see:

```text
TypeError: unsupported operand type(s) for +: 'Vector' and 'int'
```

That is good.

It matches Python's normal behavior.

Returning `NotImplemented` is part of cooperating with Python's operator machinery.

---

# Reflected Methods

Now consider:

```python
10 + Vector(1, 2)
```

The left operand is an integer.

Python asks the integer first.

Conceptually:

```python
int.__add__(10, Vector(1, 2))
```

The integer does not know how to add a vector.

If the left operand cannot handle the operation, Python may ask the right operand through a reflected method.

For addition, the reflected method is:

```python
__radd__
```

So Python may try:

```python
Vector.__radd__(Vector(1, 2), 10)
```

Reflected methods let the right operand participate when the left operand does not know what to do.

Common reflected methods include:

```text
__radd__
__rsub__
__rmul__
__rtruediv__
__rfloordiv__
__rmod__
__rpow__
__rmatmul__
__rlshift__
__rrshift__
__rand__
__rxor__
__ror__
```

They matter most in mixed-type expressions.

---

# Supporting Scalar Multiplication

Vector addition works only between vectors.

But scalar multiplication is natural:

```python
Vector(2, 3) * 10
```

should produce:

```python
Vector(20, 30)
```

Implement `__mul__`:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"

    def __mul__(self, scalar):
        if not isinstance(scalar, int | float):
            return NotImplemented
        return Vector(self.x * scalar, self.y * scalar)
```

Now:

```python
Vector(2, 3) * 10
```

works.

But:

```python
10 * Vector(2, 3)
```

does not necessarily work yet.

The integer gets the first chance.

The integer does not know your vector class.

So implement `__rmul__`:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"

    def __mul__(self, scalar):
        if not isinstance(scalar, int | float):
            return NotImplemented
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        return self.__mul__(scalar)
```

Now both forms work:

```python
Vector(2, 3) * 10
10 * Vector(2, 3)
```

This is appropriate because scalar multiplication is commutative in this case.

The order does not change the result.

---

# Reflected Methods Are Not Always Identical

Do not blindly implement reflected methods by delegating.

For subtraction, order matters:

```python
a - b
```

is not the same as:

```python
b - a
```

Example:

```python
class NumberBox:
    def __init__(self, value):
        self.value = value

    def __sub__(self, other):
        if isinstance(other, int | float):
            return NumberBox(self.value - other)
        return NotImplemented

    def __rsub__(self, other):
        if isinstance(other, int | float):
            return NumberBox(other - self.value)
        return NotImplemented

    def __repr__(self):
        return f"NumberBox({self.value!r})"
```

Now:

```python
NumberBox(10) - 3
```

is:

```python
NumberBox(7)
```

But:

```python
3 - NumberBox(10)
```

is:

```python
NumberBox(-7)
```

The reflected method must respect operand order.

For commutative operations like some additions and multiplications, delegation may be fine.

For non-commutative operations, write the reflected method carefully.

---

# In-Place Operators

In-place operators look like this:

```python
x += y
x -= y
x *= y
x /= y
```

They are connected to methods such as:

```text
__iadd__
__isub__
__imul__
__itruediv__
```

The `i` means in-place.

But the behavior depends on the object.

For mutable objects, in-place operations often mutate the object.

Example with lists:

```python
items = [1, 2]
same = items

items += [3]

print(items)
print(same)
```

Output:

```python
[1, 2, 3]
[1, 2, 3]
```

The list was mutated.

For immutable objects, in-place syntax usually creates a new object and rebinds the variable.

Example with integers:

```python
number = 10
number += 5
```

The integer object `10` is not mutated.

The name `number` is rebound to another integer object.

This distinction matters when you implement `__iadd__`.

---

# Implementing `__iadd__` for a Mutable Object

Suppose we build a bag:

```python
class Bag:
    def __init__(self, items=None):
        self.items = list(items or [])

    def __repr__(self):
        return f"Bag({self.items!r})"
```

Normal addition can return a new bag:

```python
class Bag:
    def __init__(self, items=None):
        self.items = list(items or [])

    def __repr__(self):
        return f"Bag({self.items!r})"

    def __add__(self, other):
        if type(other) is not Bag:
            return NotImplemented
        return Bag(self.items + other.items)
```

Now implement in-place addition:

```python
class Bag:
    def __init__(self, items=None):
        self.items = list(items or [])

    def __repr__(self):
        return f"Bag({self.items!r})"

    def __add__(self, other):
        if type(other) is not Bag:
            return NotImplemented
        return Bag(self.items + other.items)

    def __iadd__(self, other):
        if type(other) is not Bag:
            return NotImplemented
        self.items.extend(other.items)
        return self
```

Usage:

```python
bag = Bag(["a"])
alias = bag

bag += Bag(["b"])

print(bag)
print(alias)
```

Both names refer to the same mutated object:

```python
Bag(['a', 'b'])
Bag(['a', 'b'])
```

If `__iadd__` mutates, it should return `self`.

That is the standard pattern.

---

# In-Place Fallback

If `__iadd__` is not defined, Python can fall back to normal addition and assignment.

This:

```python
x += y
```

can behave like:

```python
x = x + y
```

if in-place addition is unavailable.

For immutable value objects, that fallback is often fine.

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
```

Then:

```python
v = Vector(1, 2)
v += Vector(3, 4)
```

can rebind `v` to the result of `v + Vector(3, 4)`.

The original vector does not mutate.

This can be exactly what you want for value objects.

Do not implement `__iadd__` unless you want to customize in-place behavior.

---

# The Tuple Surprise

Python has a famous in-place-operation surprise involving mutable objects inside immutable containers.

Consider:

```python
t = ([1, 2],)
t[0] += [3]
```

This can both mutate the list and raise an error.

Why?

The list inside the tuple is mutable.

The tuple itself is immutable.

The `+=` operation tries to mutate the list in place and then assign the result back into the tuple slot.

The mutation can happen before the tuple assignment fails.

The lesson is not that `+=` is bad.

The lesson is that in-place operators combine:

* object mutation
* assignment behavior
* container rules

When designing your own `__iadd__`, be aware that it participates in assignment contexts.

Mutation has consequences when aliases exist.

---

# Unary Operators

Unary operators work with one operand.

Common unary operator methods:

```text
-x      -> __neg__
+x      -> __pos__
abs(x)  -> __abs__
~x      -> __invert__
```

Example:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __neg__(self):
        return Vector(-self.x, -self.y)

    def __pos__(self):
        return Vector(+self.x, +self.y)

    def __abs__(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"
```

Usage:

```python
v = Vector(3, 4)

print(-v)
print(+v)
print(abs(v))
```

Output:

```python
Vector(-3, -4)
Vector(3, 4)
5.0
```

Unary operators should also have natural meanings.

`-vector` means the opposite vector.

`abs(vector)` means magnitude.

Those are reasonable.

If there is no clear meaning, do not implement the operator.

---

# Comparison Operators

Comparison operators use rich comparison methods:

```text
a == b   -> __eq__
a != b   -> __ne__
a < b    -> __lt__
a <= b   -> __le__
a > b    -> __gt__
a >= b   -> __ge__
```

You often define `__eq__`.

You define ordering methods only when the type has a natural ordering.

Example:

```python
class Version:
    def __init__(self, major, minor, patch=0):
        self.major = major
        self.minor = minor
        self.patch = patch

    def _parts(self):
        return (self.major, self.minor, self.patch)

    def __eq__(self, other):
        if type(other) is not Version:
            return NotImplemented
        return self._parts() == other._parts()

    def __lt__(self, other):
        if type(other) is not Version:
            return NotImplemented
        return self._parts() < other._parts()

    def __repr__(self):
        return f"Version({self.major!r}, {self.minor!r}, {self.patch!r})"
```

Now:

```python
Version(1, 2) < Version(1, 3)
```

is true.

This ordering is natural.

Version numbers have an expected comparison rule.

But many objects do not.

Is one user less than another user?

Maybe by name.

Maybe by ID.

Maybe by creation date.

If there are multiple plausible orderings, prefer explicit sort keys:

```python
users.sort(key=lambda user: user.created_at)
```

Do not overload `<` unless the ordering is part of the type's meaning.

---

# `functools.total_ordering`

Writing all ordering methods can be repetitive.

The `functools.total_ordering` decorator can help.

If you define `__eq__` and one ordering method, it can fill in the rest.

Example:

```python
from functools import total_ordering


@total_ordering
class Version:
    def __init__(self, major, minor, patch=0):
        self.major = major
        self.minor = minor
        self.patch = patch

    def _parts(self):
        return (self.major, self.minor, self.patch)

    def __eq__(self, other):
        if type(other) is not Version:
            return NotImplemented
        return self._parts() == other._parts()

    def __lt__(self, other):
        if type(other) is not Version:
            return NotImplemented
        return self._parts() < other._parts()
```

Now Python can derive:

* `<=`
* `>`
* `>=`

This is convenient.

Dataclasses can also generate ordering with:

```python
@dataclass(order=True)
```

The same design rule applies:

Only provide ordering when the ordering is meaningful for the type.

---

# Equality and Hashing

Operator overloading includes equality.

If you implement `__eq__`, think about `__hash__`.

The hash rule is:

```text
if a == b, then hash(a) == hash(b)
```

Example:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))
```

This is dangerous if `x` and `y` can change:

```python
v = Vector(1, 2)
items = {v}
v.x = 99
```

Now the set may not behave correctly.

For mutable objects with value equality, avoid `__hash__`.

For immutable value objects, hashing is often appropriate.

Dataclasses encode this principle:

```python
@dataclass(frozen=True)
class Vector:
    x: int
    y: int
```

A frozen dataclass with equality can usually be hashable if its fields are hashable.

The broader lesson:

```text
operators are tied to object invariants
```

You cannot design equality, hashing, and mutation separately.

---

# Numeric Conversion and Operators

Some types should convert naturally to numbers.

Special methods include:

```text
int(x)    -> __int__
float(x)  -> __float__
complex(x)-> __complex__
```

There is also:

```text
__index__
```

`__index__` means the object can be used as an exact integer in places like indexing and slicing.

Example:

```python
class Count:
    def __init__(self, value):
        self.value = value

    def __index__(self):
        return self.value
```

Usage:

```python
items = ["a", "b", "c"]
print(items[Count(1)])
```

Output:

```python
b
```

Do not implement numeric conversion casually.

If a type can be represented in multiple numeric ways, named methods may be better.

Example:

```python
temperature.as_celsius()
temperature.as_fahrenheit()
```

is clearer than:

```python
float(temperature)
```

when the unit is ambiguous.

---

# Collection Operators

Operators are not only numeric.

Built-in collections use operators too.

Lists use `+` for concatenation:

```python
[1, 2] + [3, 4]
```

Sets use operators for set algebra:

```python
first | second   # union
first & second   # intersection
first - second   # difference
first ^ second   # symmetric difference
```

Dictionaries support merge with `|`:

```python
combined = defaults | overrides
```

Paths use `/` in `pathlib`:

```python
from pathlib import Path

path = Path("book") / "chapter.md"
```

These examples are instructive because the operator meanings are domain-friendly.

`set_a | set_b` resembles mathematical union.

`path / child` resembles path joining.

When designing collection-like types, ask:

```text
which existing Python or mathematical convention am I following?
```

If you cannot answer, use a named method.

---

# A Set-Like Permission Example

Suppose we model permissions:

```python
class Permissions:
    def __init__(self, names):
        self._names = frozenset(names)

    def __or__(self, other):
        if type(other) is not Permissions:
            return NotImplemented
        return Permissions(self._names | other._names)

    def __and__(self, other):
        if type(other) is not Permissions:
            return NotImplemented
        return Permissions(self._names & other._names)

    def __sub__(self, other):
        if type(other) is not Permissions:
            return NotImplemented
        return Permissions(self._names - other._names)

    def __contains__(self, name):
        return name in self._names

    def __repr__(self):
        return f"Permissions({sorted(self._names)!r})"
```

Usage:

```python
read = Permissions(["read"])
write = Permissions(["write"])
admin = Permissions(["read", "write", "delete"])

print(read | write)
print(admin - write)
print("delete" in admin)
```

This is reasonable because permissions behave like sets.

The operators follow set conventions.

The implementation uses `frozenset`, making the object value-like.

This is a good sign.

Operator overloading works best when the domain already has operator-like concepts.

---

# A Path-Like Example

Path joining with `/` is familiar because of `pathlib`.

You can build a tiny teaching example:

```python
class SimplePath:
    def __init__(self, parts):
        if isinstance(parts, str):
            self.parts = tuple(part for part in parts.split("/") if part)
        else:
            self.parts = tuple(parts)

    def __truediv__(self, child):
        if not isinstance(child, str):
            return NotImplemented
        return SimplePath(self.parts + (child,))

    def __str__(self):
        return "/" + "/".join(self.parts)

    def __repr__(self):
        return f"SimplePath({str(self)!r})"
```

Usage:

```python
path = SimplePath("/book") / "volume-2" / "chapter-53.md"
print(path)
```

Output:

```text
/book/volume-2/chapter-53.md
```

This is understandable because `/` already visually resembles path separators.

But for real code, use `pathlib.Path`.

The purpose here is to understand the operator protocol.

---

# Matrix Multiplication: `@`

Python has a matrix multiplication operator:

```python
@
```

It maps to:

```text
__matmul__
__rmatmul__
__imatmul__
```

This operator exists because matrix multiplication is common in numerical computing.

Example teaching sketch:

```python
class Matrix2x2:
    def __init__(self, a, b, c, d):
        self.a = a
        self.b = b
        self.c = c
        self.d = d

    def __matmul__(self, other):
        if type(other) is not Matrix2x2:
            return NotImplemented
        return Matrix2x2(
            self.a * other.a + self.b * other.c,
            self.a * other.b + self.b * other.d,
            self.c * other.a + self.d * other.c,
            self.c * other.b + self.d * other.d,
        )

    def __repr__(self):
        return f"Matrix2x2({self.a}, {self.b}, {self.c}, {self.d})"
```

Usage:

```python
first = Matrix2x2(1, 2, 3, 4)
second = Matrix2x2(5, 6, 7, 8)

print(first @ second)
```

This operator should not be used casually.

Use it for matrix-like or composition-like domains where `@` has an established meaning.

---

# Augmented Assignment and Aliasing

Aliasing makes in-place operators important.

Consider:

```python
first = [1, 2]
second = first

first += [3]
```

Both names see the change:

```python
print(second)
```

Output:

```python
[1, 2, 3]
```

Now compare tuples:

```python
first = (1, 2)
second = first

first += (3,)
```

`first` now refers to a new tuple:

```python
print(first)
print(second)
```

Output:

```python
(1, 2, 3)
(1, 2)
```

Your custom objects should follow the same intuition:

* mutable collection-like objects may mutate for `+=`
* immutable value-like objects should return a new object

Do not surprise users by mutating a value-like object behind their back.

Do not surprise users by making a mutable collection's `+=` behave unlike other mutable collections without a good reason.

---

# Operator Overloading and Mutability

Mutability is a design choice.

It affects operator meaning.

For a mutable `Playlist`, `+=` might add songs in place:

```python
playlist += other_playlist
```

For an immutable `Playlist`, `+=` might create a new playlist and rebind the name:

```python
playlist = playlist + other_playlist
```

Both can be valid.

The important part is consistency.

If `+` returns a new object, then `+=` may either:

* mutate and return `self`, for mutable objects
* return a new object, for immutable objects

But it should not do something surprising like returning a plain list when the operands are playlists.

Operators should preserve the abstraction.

If users add two `Playlist` objects, they probably expect a `Playlist`.

---

# Designing Mixed-Type Operations

Mixed-type operations need special care.

Example:

```python
Money(100, "INR") + Money(50, "INR")
```

is natural.

But what about:

```python
Money(100, "INR") + 50
```

Does `50` mean 50 rupees?

50 paise?

50 in the same currency?

Is that safe?

Maybe not.

A strict design rejects it:

```python
def __add__(self, other):
    if type(other) is not Money:
        return NotImplemented
    if self.currency != other.currency:
        raise ValueError("cannot add money in different currencies")
    return Money(self.amount + other.amount, self.currency)
```

Now users must write:

```python
Money(100, "INR") + Money(50, "INR")
```

This is more explicit.

Operator overloading should not encourage ambiguous shortcuts.

Mixed-type support is good when the meaning is clear.

Example:

```python
Vector(1, 2) * 3
```

Scalar multiplication is clear.

Example:

```python
Path("book") / "chapter.md"
```

Joining a path with a string segment is clear.

When meaning is ambiguous, reject the operation or use a named method.

---

# Raising Exceptions Inside Operators

When should an operator return `NotImplemented`, and when should it raise?

Use `NotImplemented` when the operand type is unsupported.

Example:

```python
def __add__(self, other):
    if type(other) is not Money:
        return NotImplemented
    ...
```

Raise an exception when the operand type is supported but the values are invalid for the operation.

Example:

```python
def __add__(self, other):
    if type(other) is not Money:
        return NotImplemented
    if self.currency != other.currency:
        raise ValueError("cannot add different currencies")
    return Money(self.amount + other.amount, self.currency)
```

Here, `Money + Money` is a supported operation.

But different currencies violate a domain rule.

So `ValueError` is appropriate.

The distinction:

```text
unsupported operand type -> NotImplemented
supported type but invalid value -> exception
```

This makes your objects cooperate with Python while still protecting domain invariants.

---

# A Money Example

Let us build a careful `Money` type.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def __post_init__(self):
        if not isinstance(self.amount, int):
            raise TypeError("amount must be stored as an integer minor unit")
        if len(self.currency) != 3:
            raise ValueError("currency must be a 3-letter code")
```

Add representation for display:

```python
@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def __post_init__(self):
        if not isinstance(self.amount, int):
            raise TypeError("amount must be stored as an integer minor unit")
        if len(self.currency) != 3:
            raise ValueError("currency must be a 3-letter code")

    def __str__(self):
        return f"{self.amount / 100:.2f} {self.currency}"
```

Now add addition:

```python
@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def __post_init__(self):
        if not isinstance(self.amount, int):
            raise TypeError("amount must be stored as an integer minor unit")
        if len(self.currency) != 3:
            raise ValueError("currency must be a 3-letter code")

    def __str__(self):
        return f"{self.amount / 100:.2f} {self.currency}"

    def __add__(self, other):
        if type(other) is not Money:
            return NotImplemented
        if self.currency != other.currency:
            raise ValueError("cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)
```

Usage:

```python
subtotal = Money(10_000, "INR")
tax = Money(1_800, "INR")

print(subtotal + tax)
```

Output:

```text
118.00 INR
```

This operator is natural.

It preserves currency rules.

It returns a new immutable value.

This is strong operator overloading.

---

# Should Money Support Multiplication?

What about:

```python
Money(1000, "INR") * 3
```

This can be reasonable.

It means three units of the same money amount.

Implementation:

```python
def __mul__(self, multiplier):
    if not isinstance(multiplier, int):
        return NotImplemented
    return Money(self.amount * multiplier, self.currency)

def __rmul__(self, multiplier):
    return self.__mul__(multiplier)
```

Now:

```python
price * 3
3 * price
```

both work.

Should money support division?

Maybe:

```python
total / 3
```

But then rounding becomes a domain question.

If money cannot divide evenly, what happens?

Do you round?

Do you return a remainder?

Do you raise?

Sometimes a named method is better:

```python
total.split(3)
```

because splitting money is not merely arithmetic.

It has business rules.

This is the kind of judgment operator overloading requires.

---

# A Polynomial Example

Some domains are naturally algebraic.

Polynomials are a good example.

Represent a polynomial by coefficients:

```python
class Polynomial:
    def __init__(self, coefficients):
        self.coefficients = tuple(coefficients)

    def __repr__(self):
        return f"Polynomial({self.coefficients!r})"
```

Add equality:

```python
class Polynomial:
    def __init__(self, coefficients):
        self.coefficients = tuple(coefficients)

    def __repr__(self):
        return f"Polynomial({self.coefficients!r})"

    def __eq__(self, other):
        if type(other) is not Polynomial:
            return NotImplemented
        return self.coefficients == other.coefficients
```

Add polynomial addition:

```python
from itertools import zip_longest


class Polynomial:
    def __init__(self, coefficients):
        self.coefficients = tuple(coefficients)

    def __repr__(self):
        return f"Polynomial({self.coefficients!r})"

    def __eq__(self, other):
        if type(other) is not Polynomial:
            return NotImplemented
        return self.coefficients == other.coefficients

    def __add__(self, other):
        if type(other) is not Polynomial:
            return NotImplemented
        coefficients = [
            a + b
            for a, b in zip_longest(
                self.coefficients,
                other.coefficients,
                fillvalue=0,
            )
        ]
        return Polynomial(coefficients)
```

Usage:

```python
first = Polynomial([1, 2, 3])
second = Polynomial([10, 20])

print(first + second)
```

Output:

```python
Polynomial((11, 22, 3))
```

This is a natural use of operators because the domain is mathematical.

---

# A Bad Operator Example

Suppose we build:

```python
class User:
    def __init__(self, email):
        self.email = email

    def __mul__(self, other):
        send_email(self.email, other)
```

Then:

```python
user * message
```

sends an email.

This is bad.

Multiplication does not mean sending.

The code is surprising.

Use:

```python
user.send(message)
```

or:

```python
email_service.send(user, message)
```

Operator overloading should match common mathematical, collection, or language expectations.

If the operator meaning must be explained every time, it is probably the wrong operator.

---

# Operator Overloading and Readability

Compare:

```python
invoice.total = subtotal + tax - discount
```

If these are `Money` objects, this is readable.

Compare:

```python
pipeline = load >> clean >> transform >> save
```

This might be readable in a framework that clearly defines pipeline composition.

But outside that context, it may be mysterious.

Compare:

```python
user @ permission
```

What does that mean?

Assign permission?

Check permission?

Send notification?

Matrix multiply user and permission?

This is a sign that the operator is doing too much hidden communication.

Prefer:

```python
user.has_permission(permission)
```

or:

```python
permissions.grant(user)
```

Readability depends on shared convention.

Operators are readable only when the convention is strong.

---

# Operator Families Should Be Coherent

If you implement one operator, consider related operators.

If you implement `__eq__`, think about:

* `__hash__`
* `__ne__`

If you implement `__lt__`, think about:

* `__le__`
* `__gt__`
* `__ge__`

If you implement `__add__`, think about:

* `__radd__`
* `__iadd__`

If you implement `__mul__`, think about:

* scalar support
* reflected multiplication
* in-place multiplication

You do not always need every related method.

But you should choose intentionally.

An object that supports:

```python
vector * 3
```

but not:

```python
3 * vector
```

may frustrate users if scalar multiplication is expected to be symmetric.

An object that supports:

```python
a < b
```

but not:

```python
a <= b
```

may feel incomplete.

Protocol design is not only implementation.

It is expectation management.

---

# The Role of `__ne__`

In modern Python, if you define `__eq__` and do not define `__ne__`, Python can usually derive `!=` by negating equality.

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

Then:

```python
Point(1, 2) != Point(3, 4)
```

works as expected.

You usually do not need to define `__ne__`.

Define it only when inequality has special behavior.

That is rare.

For most classes:

```text
define __eq__
let Python handle !=
```

---

# Bitwise Operators and Domain Meaning

Bitwise operators include:

```text
&  -> __and__
|  -> __or__
^  -> __xor__
~  -> __invert__
```

For integers, these operate on bits.

For sets, some of them represent set algebra.

For custom types, use them only when the meaning is strong.

Good examples:

```python
permissions_a | permissions_b
```

for permission union.

```python
query_a & query_b
```

for combining query filters, if a framework clearly establishes that convention.

Risky examples:

```python
user | email
```

unless there is an extremely clear domain convention.

Bitwise operators are visually compact and not always familiar to beginners.

Use them with extra restraint.

---

# The `@` Operator

The `@` operator was added for matrix multiplication.

It should usually be reserved for matrix-like, tensor-like, or composition-like operations where the convention is clear.

Example in numerical code:

```python
result = matrix_a @ matrix_b
```

That reads naturally to people who know linear algebra.

Outside such domains, `@` is often mysterious.

If you are tempted to use `@` for a custom business action, pause.

A named method will probably be clearer.

Operators are not a limited resource you need to use.

They are a language affordance to use only when they improve clarity.

---

# `operator` Module

Python's `operator` module provides function forms of many operators.

Example:

```python
import operator

print(operator.add(2, 3))
```

Output:

```python
5
```

This is equivalent to:

```python
2 + 3
```

The module includes functions such as:

```text
operator.add
operator.sub
operator.mul
operator.truediv
operator.eq
operator.lt
operator.itemgetter
operator.attrgetter
operator.methodcaller
```

These are useful when a function is needed.

Example:

```python
from operator import attrgetter

users.sort(key=attrgetter("last_name"))
```

The `operator` module does not replace dunder methods.

It exposes operator behavior as callables.

Those callables still use the same underlying protocols.

---

# Testing Operator Overloads

Operator overloads deserve tests because they define core object behavior.

For `Vector.__add__`, test:

```python
def test_vector_addition():
    assert Vector(1, 2) + Vector(3, 4) == Vector(4, 6)
```

Test unsupported operands:

```python
def test_vector_addition_rejects_non_vector():
    with pytest.raises(TypeError):
        Vector(1, 2) + 10
```

Test reflected behavior:

```python
def test_scalar_multiplication_from_left_and_right():
    assert Vector(1, 2) * 3 == Vector(3, 6)
    assert 3 * Vector(1, 2) == Vector(3, 6)
```

Test immutability expectations:

```python
def test_addition_does_not_mutate_operands():
    first = Vector(1, 2)
    second = Vector(3, 4)

    result = first + second

    assert first == Vector(1, 2)
    assert second == Vector(3, 4)
    assert result == Vector(4, 6)
```

Tests should express the meaning of the operator.

They protect future readers from changing behavior accidentally.

---

# Common Mistake: Using Operators for Side Effects

Operators should usually compute values.

This is suspicious:

```python
logger << "message"
```

Maybe a framework defines this convention.

But in ordinary Python, a method is clearer:

```python
logger.info("message")
```

This is also suspicious:

```python
queue + item
```

if it mutates the queue.

Users usually expect `+` to produce a value, not mutate the left operand.

For mutation, use:

```python
queue.append(item)
```

or maybe:

```python
queue += [item]
```

if the object is collection-like and `+=` is documented as mutation.

Operators that hide side effects are hard to reason about.

---

# Common Mistake: Returning Plain Built-In Types Accidentally

Suppose:

```python
class Vector:
    def __init__(self, values):
        self.values = list(values)

    def __add__(self, other):
        return self.values + other.values
```

Now:

```python
Vector([1, 2]) + Vector([3, 4])
```

returns:

```python
[1, 2, 3, 4]
```

Maybe that is not what users expect.

If adding vectors should return a vector, wrap the result:

```python
def __add__(self, other):
    if type(other) is not Vector:
        return NotImplemented
    return Vector(a + b for a, b in zip(self.values, other.values))
```

Operators should usually preserve the abstraction.

If the result type changes, it should be intentional and documented.

---

# Common Mistake: Ignoring Operand Types

This is unsafe:

```python
def __add__(self, other):
    return Vector(self.x + other.x, self.y + other.y)
```

If `other` lacks `x` or `y`, Python raises an attribute error.

That error may be confusing:

```text
AttributeError: 'int' object has no attribute 'x'
```

Better:

```python
def __add__(self, other):
    if type(other) is not Vector:
        return NotImplemented
    return Vector(self.x + other.x, self.y + other.y)
```

Now unsupported operand types produce normal operator errors.

This is both cleaner and more cooperative.

---

# Common Mistake: Over-Accepting Operand Types

The opposite mistake is accepting too much.

Example:

```python
def __add__(self, other):
    return Money(self.amount + int(other), self.currency)
```

This accepts strings, floats, booleans, and many strange objects as long as `int(other)` works.

That may hide bugs.

Strictness can be a virtue.

For money:

```python
def __add__(self, other):
    if type(other) is not Money:
        return NotImplemented
    ...
```

For vector scalar multiplication, accepting `int` and `float` may be reasonable.

For other domains, be conservative.

Operators should not silently guess what users meant.

---

# Common Mistake: Forgetting Reflected Methods

If you support:

```python
Vector(1, 2) * 3
```

users may expect:

```python
3 * Vector(1, 2)
```

If the operation is symmetric, implement the reflected method:

```python
def __rmul__(self, scalar):
    return self.__mul__(scalar)
```

But remember:

This is not safe for every operation.

Subtraction, division, and exponentiation are order-sensitive.

Write reflected methods according to the real math or domain rule.

---

# Common Mistake: Defining Ordering Without a Natural Order

This is questionable:

```python
class User:
    def __lt__(self, other):
        return self.email < other.email
```

Is email the natural ordering of users?

Maybe in one screen.

But another screen may sort by creation date.

Another may sort by last login.

Another may sort by role.

If a type has many plausible orderings, do not define `<`.

Use sort keys:

```python
users.sort(key=lambda user: user.email)
users.sort(key=lambda user: user.created_at)
```

Ordering methods should express the type's natural order, not a temporary UI preference.

---

# Common Mistake: Hashing Mutable Operator Types

Suppose:

```python
class PermissionSet:
    def __init__(self, permissions):
        self.permissions = set(permissions)

    def __eq__(self, other):
        if type(other) is not PermissionSet:
            return NotImplemented
        return self.permissions == other.permissions

    def __hash__(self):
        return hash(frozenset(self.permissions))
```

This is dangerous because `permissions` can change.

If the object is in a set and then permissions change, the hash changes.

Safer:

```python
class PermissionSet:
    def __init__(self, permissions):
        self.permissions = frozenset(permissions)

    def __eq__(self, other):
        if type(other) is not PermissionSet:
            return NotImplemented
        return self.permissions == other.permissions

    def __hash__(self):
        return hash(self.permissions)
```

If equality is value-based and the object is hashable, make the value stable.

---

# Common Mistake: Confusing Addition and Append

For collection-like objects, `+` usually combines collections and returns a new collection.

Example:

```python
[1, 2] + [3]
```

returns:

```python
[1, 2, 3]
```

It does not append to the original list.

So for a custom collection:

```python
collection + item
```

may be suspicious.

If you want to add one item, a named method is often clearer:

```python
collection.add(item)
```

If you want to concatenate two collections:

```python
collection + other_collection
```

may be reasonable.

Follow the expectations users already have from Python's built-in types.

---

# A Design Checklist

Before overloading an operator, ask:

```text
Does this operation have an obvious meaning for this type?
```

If not, use a named method.

Ask:

```text
Does the operator follow a known Python, mathematical, or domain convention?
```

If not, be suspicious.

Ask:

```text
Should the operation mutate or return a new object?
```

Make this consistent with the object's mutability.

Ask:

```text
What operand types are supported?
```

Return `NotImplemented` for unsupported types.

Ask:

```text
Do reflected methods make sense?
```

Implement them for mixed-type or symmetric operations when appropriate.

Ask:

```text
Do in-place methods make sense?
```

Implement them for mutable objects when mutation is expected.

Ask:

```text
Does equality affect hashing?
```

Protect hash invariants.

Ask:

```text
Will a reader understand this expression without a private explanation?
```

If the answer is no, use a named method.

---

# A Complete Vector Example

Here is a more complete vector class:

```python
from math import sqrt


class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"

    def __eq__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __add__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        if not isinstance(scalar, int | float):
            return NotImplemented
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        return self.__mul__(scalar)

    def __neg__(self):
        return Vector(-self.x, -self.y)

    def __abs__(self):
        return sqrt(self.x ** 2 + self.y ** 2)
```

Usage:

```python
v = Vector(3, 4)

print(v + Vector(1, 1))
print(v - Vector(1, 2))
print(v * 2)
print(2 * v)
print(-v)
print(abs(v))
```

Output:

```python
Vector(4, 5)
Vector(2, 2)
Vector(6, 8)
Vector(6, 8)
Vector(-3, -4)
5.0
```

This example works because each operator has a familiar vector meaning.

There is no cleverness.

The class feels like a Python object and like a vector.

That is the sweet spot.

---

# A Complete Mutable Collection Example

Now a mutable playlist:

```python
class Playlist:
    def __init__(self, songs=None):
        self._songs = list(songs or [])

    def add(self, song):
        self._songs.append(song)

    def __len__(self):
        return len(self._songs)

    def __iter__(self):
        return iter(self._songs)

    def __repr__(self):
        return f"Playlist({self._songs!r})"

    def __add__(self, other):
        if type(other) is not Playlist:
            return NotImplemented
        return Playlist(self._songs + other._songs)

    def __iadd__(self, other):
        if type(other) is not Playlist:
            return NotImplemented
        self._songs.extend(other._songs)
        return self
```

Usage:

```python
morning = Playlist(["Song A"])
evening = Playlist(["Song B"])

combined = morning + evening

print(morning)
print(combined)
```

Output:

```python
Playlist(['Song A'])
Playlist(['Song A', 'Song B'])
```

Now in-place:

```python
alias = morning
morning += evening

print(morning)
print(alias)
```

Both show:

```python
Playlist(['Song A', 'Song B'])
```

This is coherent:

* `+` creates a new playlist
* `+=` mutates the existing playlist

That matches common mutable collection expectations.

---

# A Complete Immutable Collection Example

Now an immutable playlist:

```python
class FrozenPlaylist:
    def __init__(self, songs=()):
        self._songs = tuple(songs)

    def __len__(self):
        return len(self._songs)

    def __iter__(self):
        return iter(self._songs)

    def __repr__(self):
        return f"FrozenPlaylist({self._songs!r})"

    def __eq__(self, other):
        if type(other) is not FrozenPlaylist:
            return NotImplemented
        return self._songs == other._songs

    def __hash__(self):
        return hash(self._songs)

    def __add__(self, other):
        if type(other) is not FrozenPlaylist:
            return NotImplemented
        return FrozenPlaylist(self._songs + other._songs)
```

No `__iadd__` is necessary.

Then:

```python
playlist = FrozenPlaylist(["A"])
alias = playlist

playlist += FrozenPlaylist(["B"])

print(playlist)
print(alias)
```

`playlist` is rebound to a new object.

`alias` still points to the original.

This matches immutable value expectations.

---

# Practice: Add Vectors

Create a `Vector` class that supports:

* representation
* equality
* addition
* subtraction
* scalar multiplication from both sides

One possible solution:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"

    def __eq__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __add__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        if type(other) is not Vector:
            return NotImplemented
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        if not isinstance(scalar, int | float):
            return NotImplemented
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        return self.__mul__(scalar)
```

Test:

```python
assert Vector(1, 2) + Vector(3, 4) == Vector(4, 6)
assert Vector(3, 4) - Vector(1, 2) == Vector(2, 2)
assert Vector(2, 3) * 10 == Vector(20, 30)
assert 10 * Vector(2, 3) == Vector(20, 30)
```

---

# Practice: Reject Ambiguous Money Addition

Create a frozen `Money` dataclass.

It should support:

* `Money + Money` for the same currency
* `ValueError` for different currencies
* `TypeError` through normal operator behavior for unsupported operand types

One possible solution:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Money:
    amount: int
    currency: str

    def __add__(self, other):
        if type(other) is not Money:
            return NotImplemented
        if self.currency != other.currency:
            raise ValueError("cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)
```

Test:

```python
assert Money(100, "INR") + Money(50, "INR") == Money(150, "INR")
```

Then:

```python
Money(100, "INR") + Money(50, "USD")
```

should raise `ValueError`.

And:

```python
Money(100, "INR") + 50
```

should produce a normal unsupported operand `TypeError`.

---

# Practice: Mutable In-Place Addition

Create a `TodoList` class.

It should support:

* `+` returning a new list
* `+=` mutating the existing list

One possible solution:

```python
class TodoList:
    def __init__(self, items=None):
        self.items = list(items or [])

    def __add__(self, other):
        if type(other) is not TodoList:
            return NotImplemented
        return TodoList(self.items + other.items)

    def __iadd__(self, other):
        if type(other) is not TodoList:
            return NotImplemented
        self.items.extend(other.items)
        return self

    def __repr__(self):
        return f"TodoList({self.items!r})"
```

Test aliasing:

```python
first = TodoList(["write"])
alias = first

first += TodoList(["edit"])

assert alias.items == ["write", "edit"]
```

This proves `+=` mutated the original object.

---

# Practice: Choose Named Methods Instead

For each operation, decide whether an operator or named method is better:

```text
Vector addition
Sending an email
Combining permission sets
Closing a file
Joining a path segment
Charging a credit card
Matrix multiplication
Adding a song to a playlist
```

Likely choices:

```text
Vector addition -> operator
Sending an email -> named method
Combining permission sets -> operator may be okay
Closing a file -> named method or context manager
Joining a path segment -> operator may be okay
Charging a credit card -> named method
Matrix multiplication -> operator
Adding a song to a playlist -> named method
```

The pattern:

```text
calculation or established composition -> operator
action or side effect -> named method
```

---

# Practice: Reflected Subtraction

Implement `NumberBox` so both expressions work correctly:

```python
NumberBox(10) - 3
3 - NumberBox(10)
```

Solution:

```python
class NumberBox:
    def __init__(self, value):
        self.value = value

    def __sub__(self, other):
        if not isinstance(other, int | float):
            return NotImplemented
        return NumberBox(self.value - other)

    def __rsub__(self, other):
        if not isinstance(other, int | float):
            return NotImplemented
        return NumberBox(other - self.value)

    def __repr__(self):
        return f"NumberBox({self.value!r})"
```

Check:

```python
print(NumberBox(10) - 3)
print(3 - NumberBox(10))
```

Expected:

```python
NumberBox(7)
NumberBox(-7)
```

This exercise shows why reflected methods are not always simple delegation.

---

# Summary

Operator overloading lets user-defined classes participate in Python's operator syntax.

Operators are backed by dunder methods such as `__add__`, `__sub__`, `__mul__`, `__eq__`, and `__lt__`.

Binary operators first give the left operand a chance to handle the operation.

If the left operand cannot handle it, Python may try a reflected method on the right operand, such as `__radd__`.

In-place operators use methods such as `__iadd__` when available.

Mutable objects often mutate and return `self` from in-place methods.

Immutable objects often rely on normal binary operations and rebinding.

Unary operators use methods such as `__neg__`, `__pos__`, `__abs__`, and `__invert__`.

Comparison operators use rich comparison methods such as `__eq__` and `__lt__`.

If equality is customized, hashing must be considered.

For unsupported operand types, binary operator methods should usually return `NotImplemented`.

For supported operand types with invalid values, raising an exception can be appropriate.

Operator overloads should preserve object meaning and user expectations.

Use operators when the meaning is natural, conventional, and readable.

Use named methods when the operation is domain-specific, side-effectful, ambiguous, or surprising.

The central design principle is:

```text
operators should make code clearer, not merely shorter
```

---

# Preview of Chapter 54

Chapter 53 studied operators as one major family of data model protocols.

Next we move to descriptors.

Descriptors are the mechanism behind some of Python's most important attribute behavior.

They help explain:

* methods
* properties
* static methods
* class methods
* managed attributes
* validation hooks
* reusable attribute logic

In earlier chapters, we used attributes and methods as if they were straightforward.

Chapter 54 shows that attribute access itself has a protocol layer.

The transition is:

```text
operators customize what objects do with syntax
descriptors customize what happens during attribute access
```

Descriptors are one of the deepest ideas in Python's object model.

Once you understand them, methods, properties, and many framework patterns become much easier to reason about.

