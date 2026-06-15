# Chapter 12 — Identity, Equality, and Memory Diagrams

---

# Learning Objectives

By the end of this chapter, you should understand:

* What identity means in Python.
* What equality means in Python.
* How `is` differs from `==`.
* Why two different objects can be equal.
* Why two names can refer to the same object.
* When identity checks are appropriate.
* Why `is None` is correct.
* Why `is True` and `is False` are usually not the right way to test truth.
* Why small integer and string interning can mislead learners.
* How mutable objects make identity and equality easier to confuse.
* How to draw memory diagrams for names, objects, aliases, containers, and mutation.
* How to use diagrams to debug real Python behavior.
* How this object-reference model prepares you for core language constructs.

This chapter completes the first major Python mental model:

```text
objects have identity, type, and value
names refer to objects
some objects can mutate
identity and equality ask different questions
```

---

# Concept Overview

Python gives you two common ways to compare objects:

```python
is
==
```

They answer different questions.

`is` asks:

```text
Are these two references pointing to the same object?
```

`==` asks:

```text
Are these two objects equal in value?
```

Example:

```python
a = [1, 2]
b = [1, 2]

print(a is b)
print(a == b)
```

Output:

```text
False
True
```

The lists have equal values.

They are different objects.

Diagram:

```text
a ─────▶ list object #1 [1, 2]

b ─────▶ list object #2 [1, 2]
```

Same value.

Different identity.

That is the central distinction.

---

# The Complete Mental Model

From Chapters 09, 10, and 11:

```text
name ─────▶ object
             ├── identity
             ├── type
             └── value/state
```

Now add comparison:

```text
is:
    compares identity

==:
    compares equality/value according to type behavior
```

Example:

```python
a = [1, 2]
b = a
c = [1, 2]
```

Diagram:

```text
a ─┐
   ├────▶ list object #1 [1, 2]
b ─┘

c ─────▶ list object #2 [1, 2]
```

Therefore:

```python
print(a is b)   # True
print(a is c)   # False
print(a == c)   # True
```

This diagram explains all three results.

---

# Identity

Identity means object sameness.

It answers:

> Is this the exact same object?

Every object has an identity.

The built-in function `id()` returns a value representing an object's identity during its lifetime.

Example:

```python
items = [1, 2, 3]

print(id(items))
print(id(items))
```

The two outputs are the same while the object exists.

The exact number does not matter.

The meaning matters:

```text
same object -> same identity
```

---

# The is Operator

The `is` operator checks identity.

Example:

```python
a = []
b = a

print(a is b)
```

Output:

```text
True
```

Why?

`a` and `b` refer to the same list object.

Diagram:

```text
a ─┐
   ├────▶ list object []
b ─┘
```

Now:

```python
a = []
b = []

print(a is b)
```

Output:

```text
False
```

Diagram:

```text
a ─────▶ list object #1 []

b ─────▶ list object #2 []
```

The lists have the same value, but they are not the same object.

---

# Equality

Equality means value comparison.

It answers:

> Should these objects be considered equal?

Example:

```python
print([1, 2] == [1, 2])
```

Output:

```text
True
```

The two lists are different objects, but their contents are equal.

Example:

```python
print("python" == "python")
print(10 == 10)
print({"a": 1} == {"a": 1})
```

Output:

```text
True
True
True
```

Equality is type-specific behavior.

Different types define equality according to their meaning.

---

# The == Operator

The `==` operator calls equality behavior.

For built-in containers, equality usually compares contents.

Example:

```python
print([1, 2, 3] == [1, 2, 3])
print((1, 2) == (1, 2))
print({"x": 1} == {"x": 1})
```

Output:

```text
True
True
True
```

For lists:

```text
same length
corresponding elements equal
```

For dictionaries:

```text
same key-value relationships
```

For strings:

```text
same sequence of characters
```

For numbers:

```text
same numeric value, with type-specific numeric rules
```

The exact implementation differs by type.

The high-level idea is:

```text
== asks for value-level equality
```

---

# Identity Is Not Equality

Identity and equality can agree or disagree.

Case 1: same object, equal value.

```python
a = [1, 2]
b = a

print(a is b)
print(a == b)
```

Output:

```text
True
True
```

Same object.

Equal to itself.

Case 2: different objects, equal value.

```python
a = [1, 2]
b = [1, 2]

print(a is b)
print(a == b)
```

Output:

```text
False
True
```

Different objects.

Equal contents.

Case 3: different objects, different values.

```python
a = [1, 2]
b = [3, 4]

print(a is b)
print(a == b)
```

Output:

```text
False
False
```

Different objects.

Different contents.

---

# Can Same Object Be Not Equal to Itself?

Usually, an object is equal to itself.

Example:

```python
x = [1, 2]
print(x == x)
```

Output:

```text
True
```

However, Python allows custom objects to define unusual equality behavior.

There are also special numeric cases such as floating-point NaN, where:

```python
nan = float("nan")
print(nan == nan)
```

prints:

```text
False
```

This is an advanced edge case from floating-point mathematics.

For now, use the normal mental model:

```text
is checks object identity
== checks type-defined equality
```

Just remember that equality is behavior, and behavior can have special cases.

---

# id() and Identity

`id(obj)` returns an identity value for an object.

Example:

```python
a = [1, 2]
b = a
c = [1, 2]

print(id(a))
print(id(b))
print(id(c))
```

Conceptually:

```text
id(a) == id(b)
id(a) != id(c)
```

Why?

```text
a and b refer to the same object
c refers to a different object
```

Use `id()` for learning and debugging.

Do not use `id()` as a normal application-level identifier.

It is about runtime identity, not business identity.

---

# CPython and Memory Addresses

In CPython, `id()` is closely related to the object's memory address.

But the Python language definition only promises:

```text
id() is unique and constant for the object during its lifetime
```

Do not rely on `id()` as a portable memory address.

Different Python implementations may behave differently internally.

Also, after an object is destroyed, an identity value may be reused for a new object.

So:

```text
id() is useful for understanding object identity
id() is not a memory management API
```

---

# When to Use is

Use `is` when you need identity.

Common correct use:

```python
if value is None:
    ...
```

This checks whether `value` is the specific singleton object `None`.

Another valid use:

```python
sentinel = object()

def get_value(value=sentinel):
    if value is sentinel:
        print("No value provided")
    else:
        print("Value provided")
```

Here, identity is exactly what matters.

The sentinel object is unique.

You are asking:

```text
Is this the exact sentinel object?
```

---

# When to Use ==

Use `==` when you care about value equality.

Examples:

```python
if username == "admin":
    ...
```

```python
if items == []:
    ...
```

```python
if status_code == 200:
    ...
```

```python
if user_input == expected:
    ...
```

These ask whether values are equal.

They do not ask whether two names refer to the same object.

Most comparisons in normal business logic use `==`, not `is`.

---

# is None

`None` is a singleton.

There is only one `None` object in a Python process.

That is why this is correct:

```python
if result is None:
    print("No result")
```

This checks identity:

```text
Is result the specific None object?
```

Do not write:

```python
if result == None:
    ...
```

It often works, but `is None` is the idiomatic and precise form.

It avoids custom equality behavior and states your intent clearly.

---

# is True and is False

Although `True` and `False` are singleton boolean objects, avoid using `is True` and `is False` for normal truth checks.

Usually write:

```python
if is_active:
    ...
```

or:

```python
if not is_active:
    ...
```

Why?

Python conditions use truthiness.

Many objects can be truthy or falsy:

```python
[]
""
0
None
```

If you specifically need to check whether a value is exactly the boolean object `True`, `is True` is possible, but it is uncommon.

In normal code:

```text
is None      -> common and correct
is True      -> usually too strict
if value     -> normal truth check
```

---

# Truthiness Is Not Equality

Truthiness asks:

> Should this object count as true in a condition?

Equality asks:

> Is this object equal to another object?

Identity asks:

> Is this the same object?

Example:

```python
values = []

print(values == [])
print(values is [])
print(bool(values))
```

Output:

```text
True
False
False
```

Explanation:

```text
values == []:
    values is equal to an empty list

values is []:
    values is not the same object as this new empty list literal

bool(values):
    empty list is falsy
```

These are three different questions.

---

# Small Integer Caching

This example can mislead learners:

```python
a = 10
b = 10

print(a is b)
```

In CPython, this often prints:

```text
True
```

Why?

CPython caches some small integer objects as an optimization.

Both names may refer to the same cached integer object.

Do not generalize from this.

Do not write:

```python
if count is 10:
    ...
```

Use:

```python
if count == 10:
    ...
```

Numeric comparison should use equality, not identity.

---

# String Interning

Python may reuse some string objects.

Example:

```python
a = "python"
b = "python"

print(a is b)
```

This may print:

```text
True
```

because Python may intern certain strings.

But this is an implementation optimization.

Do not rely on it for program logic.

Correct:

```python
if language == "python":
    ...
```

Incorrect:

```python
if language is "python":
    ...
```

Use equality for string values.

---

# Literals Can Create New Objects

Each list literal creates a new list object.

Example:

```python
print([] is [])
print([] == [])
```

Output:

```text
False
True
```

Why?

```python
[]
```

creates a new empty list object each time it is evaluated.

The two lists are equal in value.

They are not the same object.

This is why identity checks with list literals are almost always wrong.

---

# Memory Diagrams

Memory diagrams make object relationships visible.

They prevent vague thinking.

A good memory diagram separates:

* Names
* Objects
* References
* Object values
* Object identities when needed
* Container relationships

Basic diagram:

```text
x ─────▶ int object 10
```

Multiple names:

```text
a ─┐
   ├────▶ list object [1, 2]
b ─┘
```

Different equal objects:

```text
a ─────▶ list object #1 [1, 2]

b ─────▶ list object #2 [1, 2]
```

Mutation:

```text
before:
items ─────▶ list object #1 [1, 2]

after:
items ─────▶ list object #1 [1, 2, 3]
```

Rebinding:

```text
before:
items ─────▶ list object #1 [1, 2]

after:
items ─────▶ list object #2 [3, 4]
```

---

# Diagramming Rule 1: Draw Names Separately

Do not draw:

```text
+---------+
| x = 10  |
+---------+
```

That suggests the value is inside the name.

Draw:

```text
x ─────▶ int object 10
```

This keeps the name and object separate.

It matches Python's model.

---

# Diagramming Rule 2: Use One Object for Aliases

Code:

```python
a = [1, 2]
b = a
```

Correct diagram:

```text
a ─┐
   ├────▶ list object [1, 2]
b ─┘
```

Incorrect diagram:

```text
a ─────▶ list object [1, 2]
b ─────▶ list object [1, 2]
```

The incorrect diagram shows two list objects.

But the code creates one list object and two names.

---

# Diagramming Rule 3: Use Separate Objects for Separate Literals

Code:

```python
a = [1, 2]
b = [1, 2]
```

Correct diagram:

```text
a ─────▶ list object #1 [1, 2]

b ─────▶ list object #2 [1, 2]
```

Each list literal creates a new list object.

The values are equal.

The identities are different.

---

# Diagramming Rule 4: Containers Hold References

Code:

```python
inner = [1, 2]
outer = [inner]
```

Diagram:

```text
inner ─────▶ list object #1 [1, 2]
               ▲
               │
outer ─────▶ list object #2
              └──▶ list object #1
```

The outer list contains a reference to the inner list object.

It does not contain a full independent copy of the inner list.

---

# Diagramming Rule 5: Show Mutation on the Object

Code:

```python
a = [1, 2]
b = a
b.append(3)
```

Diagram:

```text
a ─┐
   ├────▶ list object [1, 2, 3]
b ─┘
```

The arrows did not change.

The object changed.

Do not draw `b` as pointing somewhere else.

---

# Diagramming Rule 6: Show Rebinding by Moving the Arrow

Code:

```python
a = [1, 2]
b = a
b = [3, 4]
```

Diagram:

```text
a ─────▶ list object #1 [1, 2]

b ─────▶ list object #2 [3, 4]
```

The object `[1, 2]` was not changed.

The name `b` was rebound to a different object.

---

# Example: Identity and Equality With Lists

Code:

```python
a = [1, 2]
b = a
c = [1, 2]

print(a is b)
print(a is c)
print(a == c)
```

Diagram:

```text
a ─┐
   ├────▶ list object #1 [1, 2]
b ─┘

c ─────▶ list object #2 [1, 2]
```

Output:

```text
True
False
True
```

Reason:

```text
a is b:
    same object

a is c:
    different objects

a == c:
    equal values
```

---

# Example: Mutation Changes Equality

Code:

```python
a = [1, 2]
b = [1, 2]

print(a == b)

a.append(3)

print(a == b)
```

Before mutation:

```text
a ─────▶ list object #1 [1, 2]
b ─────▶ list object #2 [1, 2]
```

`a == b` is `True`.

After mutation:

```text
a ─────▶ list object #1 [1, 2, 3]
b ─────▶ list object #2 [1, 2]
```

`a == b` is `False`.

Identity did not change.

Value changed.

---

# Example: Mutation Through Alias

Code:

```python
a = [1, 2]
b = a

print(a is b)

b.append(3)

print(a)
print(a is b)
```

Diagram:

```text
a ─┐
   ├────▶ list object [1, 2, 3]
b ─┘
```

Output:

```text
True
[1, 2, 3]
True
```

Mutation does not break aliasing.

The same object is still shared.

---

# Example: Rebinding Breaks Aliasing

Code:

```python
a = [1, 2]
b = a

b = [1, 2, 3]

print(a)
print(b)
print(a is b)
```

Diagram:

```text
a ─────▶ list object #1 [1, 2]

b ─────▶ list object #2 [1, 2, 3]
```

Output:

```text
[1, 2]
[1, 2, 3]
False
```

Rebinding `b` changes only `b`.

---

# Example: Nested Lists

Code:

```python
row = [1, 2]
matrix = [row, row]

matrix[0].append(3)

print(matrix)
```

Output:

```text
[[1, 2, 3], [1, 2, 3]]
```

Why did both rows appear to change?

Because both slots in `matrix` refer to the same row object.

Diagram:

```text
row ─────────▶ list object #1 [1, 2, 3]
                 ▲       ▲
                 │       │
matrix ─────▶ outer list object
              ├──┘       │
              └──────────┘
```

There is one inner list object.

It appears twice in the outer list.

---

# Example: Independent Nested Lists

Code:

```python
matrix = [[1, 2], [1, 2]]

matrix[0].append(3)

print(matrix)
```

Output:

```text
[[1, 2, 3], [1, 2]]
```

Diagram:

```text
matrix ─────▶ outer list object
              ├──▶ inner list #1 [1, 2, 3]
              └──▶ inner list #2 [1, 2]
```

There are two separate inner list objects.

Only one was mutated.

---

# Example: List Multiplication Trap

A common mistake:

```python
matrix = [[0] * 3] * 3
matrix[0][0] = 1

print(matrix)
```

Output:

```text
[[1, 0, 0], [1, 0, 0], [1, 0, 0]]
```

Why?

The outer list contains three references to the same inner list object.

Diagram:

```text
inner list [1, 0, 0]
     ▲    ▲    ▲
     │    │    │
outer list slots
```

Correct pattern:

```python
matrix = [[0] * 3 for _ in range(3)]
```

This creates a new inner list for each row.

---

# Equality of Nested Containers

Container equality is recursive.

Example:

```python
a = [[1], [2]]
b = [[1], [2]]

print(a == b)
print(a is b)
print(a[0] is b[0])
```

Output:

```text
True
False
False
```

The outer lists are equal in value.

They are different objects.

Their corresponding inner lists are also different objects.

Equality checks contents recursively.

Identity checks object sameness.

---

# Equality Can Be Customized

For custom classes, equality behavior can be defined by the class.

Example:

```python
class User:
    def __init__(self, email):
        self.email = email
```

Without custom equality, two users may compare by identity-like default behavior:

```python
u1 = User("ada@example.com")
u2 = User("ada@example.com")

print(u1 == u2)
```

This often prints:

```text
False
```

Later, classes can define equality behavior so users with the same email compare equal.

You do not need to implement this yet.

The important point:

> `==` is behavior defined by object type.

---

# Identity Cannot Be Customized

Equality can be customized.

Identity cannot.

`is` always asks whether two references point to the same object.

Custom classes can influence `==`.

They cannot redefine `is`.

This is why `is` is reliable for identity checks.

It is also why it should be used only when identity is truly the question.

---

# Equality and Type

Some equality comparisons across types can be true.

Example:

```python
print(1 == True)
```

Output:

```text
True
```

Why?

`bool` is related to `int` in Python's type hierarchy.

This does not mean `1` and `True` are the same object:

```python
print(1 is True)
```

Output:

```text
False
```

This is another reason to separate equality from identity.

---

# Equality and Floating-Point Values

Floating-point equality can be tricky.

Example:

```python
print(0.1 + 0.2 == 0.3)
```

This often prints:

```text
False
```

This is due to floating-point representation, not object identity.

For approximate numeric comparison, Python provides tools such as:

```python
import math

math.isclose(0.1 + 0.2, 0.3)
```

This belongs more to the numbers chapter.

For now, remember:

```text
== is equality behavior
numeric equality can have domain-specific issues
```

---

# Sentinels

A sentinel is a unique object used to mark a special condition.

Example:

```python
MISSING = object()

def get_config(key, default=MISSING):
    if default is MISSING:
        print("No default provided")
    else:
        print("Default provided")
```

Why use `is`?

Because the sentinel is intended to be unique.

You are not asking:

```text
Is this value equal to MISSING?
```

You are asking:

```text
Is this the exact sentinel object?
```

This pattern is common in library and framework code.

---

# None as a Sentinel

`None` is the most common sentinel.

Example:

```python
def connect(timeout=None):
    if timeout is None:
        timeout = 30
```

This means:

```text
if caller did not provide a timeout, use 30
```

But be careful.

If `None` is a valid value in your domain, you may need a custom sentinel.

Example:

```python
MISSING = object()

def update(name=MISSING):
    if name is MISSING:
        print("name was not provided")
    elif name is None:
        print("name was explicitly set to None")
```

Identity checks make this distinction possible.

---

# Memory Diagrams for Function Calls

Function calls bind parameter names to argument objects.

Example:

```python
def add_item(values):
    values.append("x")

items = []
add_item(items)
```

During the call:

```text
global namespace:
items ─┐
       ├────▶ list object []
local namespace:
values ─┘
```

After mutation:

```text
items ─────▶ list object ["x"]
```

The local name `values` is gone after the function returns.

The object remains because `items` still refers to it.

---

# Memory Diagrams for Rebinding Parameters

Example:

```python
def reset(values):
    values = []

items = [1, 2]
reset(items)
print(items)
```

During call before rebinding:

```text
items ─┐
       ├────▶ list object [1, 2]
values ─┘
```

After local rebinding:

```text
items ─────▶ list object [1, 2]

values ────▶ new list object []
```

After function returns, `values` disappears.

The caller's `items` still refers to the original list.

Output:

```text
[1, 2]
```

---

# Memory Diagrams for Copies

Shallow copy:

```python
a = [[1], [2]]
b = a.copy()
```

Diagram:

```text
a ─────▶ outer list #1
          ├──▶ inner list #1 [1]
          └──▶ inner list #2 [2]

b ─────▶ outer list #2
          ├──▶ inner list #1 [1]
          └──▶ inner list #2 [2]
```

The outer lists differ.

The inner lists are shared.

Deep copy:

```python
import copy
b = copy.deepcopy(a)
```

Diagram:

```text
a ─────▶ outer list #1
          ├──▶ inner list #1 [1]
          └──▶ inner list #2 [2]

b ─────▶ outer list #2
          ├──▶ inner list #3 [1]
          └──▶ inner list #4 [2]
```

Now nested mutable objects are independent where copying is possible.

---

# Common Mistakes

## Misconception 1

### `is` and `==` mean the same thing.

They do not.

`is` checks identity.

`==` checks equality.

Use `is` for object sameness.

Use `==` for value comparison.

---

## Misconception 2

### If `a == b`, then `a is b`.

Two objects can be equal without being identical.

Example:

```python
[1, 2] == [1, 2]
```

is `True`, but those are different list objects.

---

## Misconception 3

### If `a is b`, mutating through `a` will not affect `b`.

If `a is b`, both names refer to the same object.

Mutation through either name changes that object.

---

## Misconception 4

### Small integer identity behavior should be used in real code.

CPython may cache small integers.

Do not use `is` for numeric comparison.

Use `==`.

---

## Misconception 5

### String identity behavior is reliable for value checks.

Python may intern some strings.

Do not use `is` for string comparison.

Use `==`.

---

## Misconception 6

### `id()` is a permanent business identifier.

`id()` is a runtime object identity value.

It is not a database ID, user ID, or stable external identifier.

---

## Misconception 7

### A shallow copy makes nested structures independent.

Shallow copy creates a new outer container.

Nested mutable objects can still be shared.

---

# Real-world Usage

## Debugging Shared State

When data changes unexpectedly, ask:

```text
Are two names pointing to the same object?
Was the object mutated?
Was a shallow copy used where a deep copy was needed?
```

Use diagrams and temporary `id()` checks to investigate.

---

## Sentinel Values

Use identity checks for sentinels:

```python
if value is None:
    ...
```

or:

```python
MISSING = object()

if value is MISSING:
    ...
```

This avoids ambiguity with equality.

---

## API Boundaries

When designing functions, decide whether input objects are:

* Read only
* Mutated in place
* Copied shallowly
* Copied deeply
* Compared by identity
* Compared by value

Document that behavior when it matters.

---

## Tests

In tests, use identity and equality intentionally.

Use:

```python
assert result == expected
```

for value equality.

Use:

```python
assert result is None
```

for `None`.

Use:

```python
assert result is same_object
```

only when object identity is the behavior being tested.

---

## Data Structures

Containers compare by value but store references.

This distinction matters for:

* Nested lists
* Dictionaries containing mutable values
* Caches
* Graphs
* Trees
* Object relationships

Memory diagrams are a practical tool for these cases.

---

# Concept Connections

This chapter closes the first object-reference sequence:

```text
Chapter 09:
Objects have identity, type, and value.

Chapter 10:
Names refer to objects.

Chapter 11:
Some objects can mutate in place.

Chapter 12:
Identity and equality let us reason about object sameness and value comparison.
```

This prepares the next phase:

```text
Numbers:
    immutable numeric objects and arithmetic behavior

Strings:
    immutable text objects and string operations

Booleans:
    truth values, truthiness, and comparisons

Operators:
    type-driven behavior such as +, ==, is, and comparisons

Expressions:
    code that evaluates to objects
```

The model carries forward:

```text
expression
    |
    v
evaluates to object
    |
    v
name may refer to object
    |
    v
operations inspect, compare, mutate, or create objects
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Identity | Which exact object something is |
| Equality | Whether objects should be considered equal in value |
| `is` | Identity comparison |
| `==` | Equality comparison |
| `id()` | Runtime identity value for an object |
| Alias | Multiple references to the same object |
| Mutation | In-place change to an object |
| Rebinding | Changing which object a name refers to |
| Sentinel | Unique object used to mark a special case |
| Shallow copy | New outer container, shared nested objects |
| Deep copy | Recursive copy of nested objects where possible |

Core rules:

```text
Use is for identity.
Use == for equality.
Use is None for None checks.
Use diagrams when aliases or mutation are involved.
Do not rely on small integer or string identity behavior.
```

Core diagram:

```text
name ─────▶ object
             ├── identity
             ├── type
             └── value/state
```

---

# Active Recall

## Easy Recall Questions

1. What question does `is` answer?
2. What question does `==` answer?
3. What does `id()` show?
4. Can two different objects be equal?
5. Can two names refer to the same object?
6. What is a sentinel?
7. Why is `is None` idiomatic?
8. Should you compare strings with `is` or `==`?
9. What does a shallow copy copy?
10. What does a memory diagram show?

---

## Deep Understanding Questions

1. Why can `a == b` be true while `a is b` is false?
2. Why can mutation through one name affect another name?
3. Why does rebinding one name not affect another name?
4. Why is identity useful for sentinel values?
5. Why should small integer caching not influence how you write comparisons?
6. Why does list multiplication create aliasing problems for nested lists?
7. Why are memory diagrams useful for nested containers?
8. Why can equality be customized but identity cannot?
9. Why is `id()` not a stable business identifier?
10. Why does this chapter come before basic language constructs?

---

## Explain In Your Own Words

1. Explain identity versus equality using two list examples.
2. Explain why `[] is []` is false.
3. Explain why `[] == []` is true.
4. Explain why `value is None` is better than `value == None`.
5. Explain how a shallow copy can still share inner objects.
6. Explain why `matrix = [[0] * 3] * 3` is dangerous.
7. Explain how diagrams help distinguish mutation from rebinding.

---

## Predict-the-Output Questions

### Question 1

```python
a = [1, 2]
b = [1, 2]

print(a is b)
print(a == b)
```

What is printed?

Answer:

```text
False
True
```

Reason:

The lists are different objects with equal values.

---

### Question 2

```python
a = []
b = a

print(a is b)
b.append(1)
print(a)
```

What is printed?

Answer:

```text
True
[1]
```

Reason:

Both names refer to the same list object, and that object is mutated.

---

### Question 3

```python
print([] is [])
print([] == [])
```

What is printed?

Answer:

```text
False
True
```

Reason:

Each list literal creates a new list object. The two empty lists are equal in value.

---

### Question 4

```python
matrix = [[0] * 2] * 2
matrix[0][0] = 1

print(matrix)
```

What is printed?

Answer:

```text
[[1, 0], [1, 0]]
```

Reason:

Both rows refer to the same inner list object.

---

### Question 5

```python
value = None

print(value is None)
print(value == None)
```

What is printed?

Answer:

```text
True
True
```

Reason:

Both are true here, but `is None` is preferred because it checks identity with the singleton `None` object.

---

### Question 6

```python
a = [[1]]
b = a.copy()

print(a is b)
print(a[0] is b[0])
```

What is printed?

Answer:

```text
False
True
```

Reason:

The outer list was copied. The inner list is shared.

---

# Mental Model Questions

1. Draw two names pointing to the same list.
2. Draw two equal lists with different identities.
3. Draw mutation through an alias.
4. Draw rebinding after aliasing.
5. Draw a nested list with two references to the same inner list.
6. Draw a shallow copy of a nested list.
7. Draw a custom sentinel object used as a default value.

---

# Practical Exercises

## Exercise 1

For each pair, predict `is` and `==`:

```python
a = [1, 2]
b = a
c = [1, 2]

print(a is b)
print(a == b)
print(a is c)
print(a == c)
```

Draw the memory diagram.

---

## Exercise 2

Investigate list literals:

```python
print([] is [])
print([] == [])
```

Explain why the results differ.

---

## Exercise 3

Fix the matrix aliasing bug:

```python
matrix = [[0] * 3] * 3
matrix[0][0] = 1
print(matrix)
```

Then rewrite it using a list comprehension so each row is independent.

---

## Exercise 4

Use a sentinel:

```python
MISSING = object()

def show(value=MISSING):
    if value is MISSING:
        print("missing")
    else:
        print("provided", value)
```

Test:

```python
show()
show(None)
show(0)
```

Explain why identity is the correct comparison.

---

## Exercise 5

Compare shallow and deep copy:

```python
import copy

a = [[1], [2]]
b = a.copy()
c = copy.deepcopy(a)

a[0].append(99)

print(a)
print(b)
print(c)
```

Draw the object relationships.

---

## Exercise 6

Read this code:

```python
def reset(values):
    values = []

def clear(values):
    values.clear()

items = [1, 2, 3]
reset(items)
print(items)

clear(items)
print(items)
```

Predict the output.

Explain rebinding versus mutation.

---

## Exercise 7

Write three comparisons:

1. One where `is` and `==` are both true.
2. One where `is` is false but `==` is true.
3. One where both are false.

Draw diagrams for each.

---

# Summary

In this chapter we learned:

* Identity asks whether two references point to the same object.
* Equality asks whether two objects should be considered equal in value.
* `is` checks identity.
* `==` checks equality.
* `id()` reveals runtime object identity for learning and debugging.
* Two different objects can be equal.
* Two names can refer to the same object.
* Mutation changes an object while preserving identity.
* Rebinding changes a name-object relationship.
* `is None` is the correct idiom for checking `None`.
* Use `==` for normal numeric and string value comparisons.
* Small integer caching and string interning are implementation details.
* Equality can be customized by object type.
* Identity cannot be customized.
* Shallow copies can share nested mutable objects.
* Memory diagrams make names, objects, aliases, mutation, and copying visible.

The complete core model is:

```text
name ─────▶ object
             ├── identity: same object?
             ├── type: what kind of object?
             └── value: equal value?
```

This model will be used throughout the rest of the book.

---

# Preview of Chapter 13

We are now ready to move from the general object model into Python's everyday value types.

Chapter 13 begins Part IV with numbers.

We will study:

* Integer objects.
* Floating-point objects.
* Complex numbers.
* Numeric operators.
* Numeric immutability.
* Identity and equality with numbers.
* Precision, rounding, and common numeric mistakes.

The important difference is that we will no longer treat these as isolated syntax rules.

We will understand them as operations on objects:

```text
numeric literal -> numeric object
numeric operator -> numeric object behavior
numeric comparison -> boolean object
```

That is the foundation of real Python fluency.
