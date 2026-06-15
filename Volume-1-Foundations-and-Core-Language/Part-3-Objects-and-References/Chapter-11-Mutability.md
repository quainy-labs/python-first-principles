# Chapter 11 — Mutability

---

# Learning Objectives

By the end of this chapter, you should understand:

* What mutability means.
* What immutability means.
* Why mutability is a property of objects, not names.
* Why rebinding and mutation are different operations.
* Why lists can change in place but integers and strings cannot.
* How aliases make mutation visible through multiple names.
* Why function arguments can appear to "change outside" a function.
* Why mutable default arguments are dangerous.
* Why copying matters when working with mutable objects.
* The difference between shallow copy and deep copy at a high level.
* Why immutable objects make reasoning easier.
* Why mutable objects are useful despite the risks.
* How mutability connects to identity, equality, and memory diagrams.

This chapter answers the question left by Chapter 10:

> If names refer to objects, what happens when the object itself can change?

---

# Concept Overview

Chapter 09 taught:

```text
Python programs manipulate objects.
```

Chapter 10 taught:

```text
names refer to objects
```

Now we add the next layer:

```text
some objects can change in place
some objects cannot
```

That is mutability.

An object is mutable if its value or internal state can change without creating a different object.

An object is immutable if its value cannot change after the object is created.

Example of a mutable object:

```python
items = [1, 2]
items.append(3)
print(items)
```

Output:

```text
[1, 2, 3]
```

The list object changed in place.

Example of an immutable object:

```python
x = 10
x = x + 1
print(x)
```

Output:

```text
11
```

The integer object `10` did not change into `11`.

The name `x` was rebound to a different integer object.

This distinction is essential.

---

# Core Mental Model

There are two different operations learners often confuse:

```text
rebinding:
    changing which object a name points to

mutation:
    changing an object itself
```

Rebinding:

```python
x = [1, 2]
x = [3, 4]
```

Diagram:

```text
before:
x ─────▶ list object [1, 2]

after:
x ─────▶ different list object [3, 4]
```

Mutation:

```python
x = [1, 2]
x.append(3)
```

Diagram:

```text
before:
x ─────▶ list object [1, 2]

after:
x ─────▶ same list object [1, 2, 3]
```

The arrow did not change.

The object changed.

---

# Mutability Belongs to Objects

Names are not mutable or immutable.

Objects are mutable or immutable.

Example:

```python
x = 10
x = "hello"
x = [1, 2, 3]
```

The name `x` can be rebound to different objects.

That does not mean `x` is mutable.

The objects have mutability properties:

```text
10          -> immutable int object
"hello"    -> immutable str object
[1, 2, 3]  -> mutable list object
```

Correct phrasing:

```text
The list object is mutable.
The integer object is immutable.
The name x can be rebound.
```

Incorrect phrasing:

```text
The variable x is mutable.
```

That wording hides the real model.

---

# What Mutable Means

A mutable object can change while keeping the same identity.

Example:

```python
items = [1, 2]

print(id(items))
items.append(3)
print(id(items))

print(items)
```

Output conceptually:

```text
same identity number
same identity number
[1, 2, 3]
```

The list's value changed.

The list's identity stayed the same.

Mental model:

```text
before:
items ─────▶ list object #1 [1, 2]

after append:
items ─────▶ list object #1 [1, 2, 3]
```

Same object.

Changed value.

That is mutation.

---

# What Immutable Means

An immutable object cannot change its value after creation.

Example:

```python
x = 10
print(id(x))

x = x + 1
print(id(x))
print(x)
```

Output conceptually:

```text
identity for int object 10
identity for int object 11
11
```

The exact `id()` behavior for small integers can be affected by CPython caching, so do not build program logic on it.

The conceptual rule remains:

```text
integer objects do not change value in place
```

When you write:

```python
x = x + 1
```

Python evaluates `x + 1`, produces an integer object representing `11`, then rebinds `x`.

It does not edit the old integer object `10`.

---

# Common Mutable and Immutable Types

Common immutable types:

| Type | Example |
| --- | --- |
| `int` | `10` |
| `float` | `3.14` |
| `bool` | `True` |
| `str` | `"hello"` |
| `tuple` | `(1, 2)` |
| `frozenset` | `frozenset({1, 2})` |
| `NoneType` | `None` |

Common mutable types:

| Type | Example |
| --- | --- |
| `list` | `[1, 2, 3]` |
| `dict` | `{"name": "Ada"}` |
| `set` | `{1, 2, 3}` |
| `bytearray` | `bytearray(b"abc")` |
| Most custom class instances | `User()` |

This table is a starting point.

The important habit is asking:

```text
Does this operation create a new object?
Or does it change the existing object?
```

---

# Lists Are Mutable

Lists can change in place.

Example:

```python
items = ["a", "b"]
items.append("c")

print(items)
```

Output:

```text
['a', 'b', 'c']
```

The list object is the same object.

Its contents changed.

Other list mutation operations include:

```python
items.append("x")
items.extend(["y", "z"])
items.insert(0, "start")
items.remove("x")
items.pop()
items.sort()
items.reverse()
items[0] = "changed"
```

These operations modify the list object.

They are not merely rebinding the name.

---

# Strings Are Immutable

Strings cannot change in place.

Example:

```python
text = "hello"
upper_text = text.upper()

print(text)
print(upper_text)
```

Output:

```text
hello
HELLO
```

The method `upper()` does not change the original string.

It returns a new string object.

This is why:

```python
text = "hello"
text.upper()
print(text)
```

prints:

```text
hello
```

The result of `text.upper()` was created, but the name `text` was not rebound to it.

To keep the new string:

```python
text = text.upper()
print(text)
```

Output:

```text
HELLO
```

This is rebinding, not mutation.

---

# Integers Are Immutable

Integers cannot change in place.

Example:

```python
count = 1
count = count + 1
```

This does not modify the integer object `1`.

It creates or obtains an integer object representing `2`, then rebinds `count`.

Mental model:

```text
before:
count ─────▶ int object 1

after:
count ─────▶ int object 2
```

This is why aliases to immutable values do not behave like aliases to mutable containers.

Example:

```python
a = 1
b = a
a = a + 1

print(a)
print(b)
```

Output:

```text
2
1
```

Rebinding `a` does not affect `b`.

---

# Dictionaries Are Mutable

Dictionaries can change in place.

Example:

```python
user = {"name": "Ada"}
user["age"] = 36

print(user)
```

Output:

```text
{'name': 'Ada', 'age': 36}
```

The dictionary object changed.

Common dictionary mutations:

```python
user["city"] = "London"
user.update({"active": True})
user.pop("age")
user.clear()
```

All of these change the dictionary object.

If another name refers to the same dictionary, it sees the change.

---

# Sets Are Mutable

Sets can change in place.

Example:

```python
seen = set()
seen.add("python")
seen.add("python")

print(seen)
```

Output:

```text
{'python'}
```

The set object is mutable.

Common set mutations:

```python
seen.add(item)
seen.remove(item)
seen.discard(item)
seen.update(other_items)
seen.clear()
```

The object identity stays the same while contents can change.

---

# Tuples Are Immutable, But Be Careful

Tuples are immutable.

Example:

```python
point = (10, 20)
point[0] = 99
```

This raises:

```text
TypeError
```

You cannot replace an item inside a tuple.

But a tuple can contain references to mutable objects.

Example:

```python
data = ([1, 2], "label")
data[0].append(3)

print(data)
```

Output:

```text
([1, 2, 3], 'label')
```

Did the tuple change?

The tuple still refers to the same two objects.

Its slots were not reassigned.

But the list object inside the tuple changed.

Mental model:

```text
tuple object
├── reference to list object [1, 2, 3]
└── reference to str object "label"
```

Tuple immutability means the tuple's own references cannot be changed.

It does not freeze mutable objects inside it.

---

# Rebinding vs Mutation

This is the core distinction.

Rebinding:

```python
items = [1, 2]
items = [3, 4]
```

The name `items` now refers to a different list object.

Mutation:

```python
items = [1, 2]
items.append(3)
```

The name `items` still refers to the same list object.

That object changed.

Diagram:

```text
rebinding:

before:
items ─────▶ list object #1 [1, 2]

after:
items ─────▶ list object #2 [3, 4]


mutation:

before:
items ─────▶ list object #1 [1, 2]

after:
items ─────▶ list object #1 [1, 2, 3]
```

Most mutability bugs come from confusing these two operations.

---

# Aliasing Makes Mutation Visible

Aliasing means multiple names refer to the same object.

Example:

```python
a = [1, 2]
b = a

b.append(3)

print(a)
```

Output:

```text
[1, 2, 3]
```

Why did `a` appear to change?

Because `a` and `b` refer to the same list object.

Diagram before mutation:

```text
a ─┐
   ├────▶ list object [1, 2]
b ─┘
```

After:

```text
a ─┐
   ├────▶ list object [1, 2, 3]
b ─┘
```

The name `a` did not change.

The object changed.

Since `a` still points to that object, printing `a` shows the new value.

---

# Rebinding an Alias Does Not Mutate

Compare this:

```python
a = [1, 2]
b = a

b = [3, 4]

print(a)
print(b)
```

Output:

```text
[1, 2]
[3, 4]
```

Diagram:

```text
after a = [1, 2]:
a ─────▶ list object #1 [1, 2]

after b = a:
a ─┐
   ├────▶ list object #1 [1, 2]
b ─┘

after b = [3, 4]:
a ─────▶ list object #1 [1, 2]

b ─────▶ list object #2 [3, 4]
```

Rebinding `b` did not affect `a`.

Again:

```text
rebinding changes a name
mutation changes an object
```

---

# Function Arguments and Mutation

Chapter 10 taught:

> Function calls bind parameter names to argument objects.

Now we can see why mutability matters.

Example:

```python
def add_item(values):
    values.append("new")

items = []
add_item(items)
print(items)
```

Output:

```text
['new']
```

During the function call:

```text
items  ─┐
        ├────▶ list object []
values ─┘
```

Inside the function:

```python
values.append("new")
```

mutates the list object.

After the call:

```text
items ─────▶ list object ["new"]
```

The caller sees the change because the same object was mutated.

---

# Rebinding a Parameter Does Not Affect Caller

Compare:

```python
def reset(values):
    values = []

items = [1, 2, 3]
reset(items)
print(items)
```

Output:

```text
[1, 2, 3]
```

Inside the function, this line:

```python
values = []
```

rebinds the local name `values`.

It does not rebind the caller's name `items`.

Diagram during call before rebinding:

```text
items  ─┐
        ├────▶ list object [1, 2, 3]
values ─┘
```

After local rebinding:

```text
items ─────▶ list object [1, 2, 3]

values ────▶ new list object []
```

The original list was not changed.

---

# Mutation Through Parameter vs Rebinding Parameter

These two functions behave differently:

```python
def mutate(values):
    values.append(4)

def rebind(values):
    values = [4]
```

Use:

```python
items = [1, 2, 3]
mutate(items)
print(items)
```

Output:

```text
[1, 2, 3, 4]
```

Use:

```python
items = [1, 2, 3]
rebind(items)
print(items)
```

Output:

```text
[1, 2, 3]
```

Reason:

```text
mutate:
    changes the object

rebind:
    changes only the local parameter binding
```

---

# Mutable Default Arguments

One of Python's most famous bugs comes from mutable default arguments.

Example:

```python
def add_task(task, tasks=[]):
    tasks.append(task)
    return tasks

print(add_task("write"))
print(add_task("test"))
print(add_task("ship"))
```

Many beginners expect:

```text
['write']
['test']
['ship']
```

Actual output:

```text
['write']
['write', 'test']
['write', 'test', 'ship']
```

Why?

Default argument values are created when the function is defined, not each time the function is called.

The default list object is reused.

Each call mutates the same list.

---

# Correct Pattern for Mutable Defaults

Use `None` as the default and create a new mutable object inside the function.

Correct:

```python
def add_task(task, tasks=None):
    if tasks is None:
        tasks = []

    tasks.append(task)
    return tasks
```

Now:

```python
print(add_task("write"))
print(add_task("test"))
print(add_task("ship"))
```

Output:

```text
['write']
['test']
['ship']
```

Each call without an explicit `tasks` argument creates a new list.

If the caller passes an existing list, the function still mutates that list:

```python
shared = []
add_task("write", shared)
add_task("test", shared)
print(shared)
```

Output:

```text
['write', 'test']
```

That is expected because the caller explicitly provided the mutable object.

---

# Why None Is Used as a Sentinel

`None` is commonly used to mean:

```text
no value was provided
```

Example:

```python
def collect(value, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(value)
    return bucket
```

The check uses `is None` because `None` is a singleton object.

This is an identity check.

It asks:

```text
Is bucket the specific None object?
```

This connects mutability to identity.

Chapter 12 will study identity checks more formally.

---

# In-place Methods Usually Return None

Many mutating methods return `None`.

Example:

```python
items = [3, 1, 2]
result = items.sort()

print(items)
print(result)
```

Output:

```text
[1, 2, 3]
None
```

Why does `sort()` return `None`?

Because it mutates the list in place.

Returning `None` helps signal:

```text
the object was changed
no new sorted list was returned
```

Correct:

```python
items = [3, 1, 2]
items.sort()
print(items)
```

If you want a new sorted list:

```python
items = [3, 1, 2]
new_items = sorted(items)

print(items)
print(new_items)
```

Output:

```text
[3, 1, 2]
[1, 2, 3]
```

---

# In-place Method vs Function Returning New Object

Compare:

```python
items = [3, 1, 2]
items.sort()
```

with:

```python
items = [3, 1, 2]
sorted_items = sorted(items)
```

`items.sort()`:

```text
mutates the existing list
returns None
```

`sorted(items)`:

```text
creates a new sorted list
leaves original list unchanged
```

This distinction appears across Python APIs.

Ask:

```text
Does this method mutate the object?
Does this function return a new object?
```

---

# Augmented Assignment

Augmented assignment uses operators such as:

```python
+=
-=
*=
/=
```

It can involve mutation or rebinding depending on the object type.

Example with an integer:

```python
x = 10
x += 1
print(x)
```

Conceptually:

```text
x is rebound to int object 11
```

Integers are immutable.

Example with a list:

```python
items = [1, 2]
items += [3]
print(items)
```

Output:

```text
[1, 2, 3]
```

For lists, `+=` mutates the existing list in place.

This matters with aliases.

---

# Augmented Assignment With Aliases

Example:

```python
a = [1, 2]
b = a

a += [3]

print(a)
print(b)
print(a is b)
```

Output:

```text
[1, 2, 3]
[1, 2, 3]
True
```

The list object was mutated.

Both names still refer to it.

Compare with:

```python
a = (1, 2)
b = a

a += (3,)

print(a)
print(b)
print(a is b)
```

Output:

```text
(1, 2, 3)
(1, 2)
False
```

Tuples are immutable.

`a += (3,)` creates a new tuple and rebinds `a`.

The name `b` still refers to the old tuple.

---

# Copying Mutable Objects

If assignment does not copy, how do we make a separate object?

For lists:

```python
a = [1, 2, 3]
b = a.copy()
```

or:

```python
b = list(a)
```

or:

```python
b = a[:]
```

Now:

```python
print(a is b)
print(a == b)
```

Output:

```text
False
True
```

They are different list objects with equal values.

Diagram:

```text
a ─────▶ list object #1 [1, 2, 3]

b ─────▶ list object #2 [1, 2, 3]
```

---

# Shallow Copy

A shallow copy creates a new outer container but keeps references to the same inner objects.

Example:

```python
row = [1, 2]
a = [row]
b = a.copy()
```

Diagram:

```text
a ─────▶ outer list #1
          └──▶ row list [1, 2]

b ─────▶ outer list #2
          └──▶ same row list [1, 2]
```

Now:

```python
row.append(3)

print(a)
print(b)
```

Output:

```text
[[1, 2, 3]]
[[1, 2, 3]]
```

The outer lists are different.

The inner list is shared.

That is shallow copy.

---

# Deep Copy

A deep copy recursively copies nested objects.

Python provides this through the `copy` module.

Example:

```python
import copy

a = [[1, 2], [3, 4]]
b = copy.deepcopy(a)

a[0].append(99)

print(a)
print(b)
```

Output:

```text
[[1, 2, 99], [3, 4]]
[[1, 2], [3, 4]]
```

The nested lists are not shared.

Deep copy is useful when you need independent nested mutable structures.

But it can be expensive and sometimes semantically wrong for complex objects.

Use it intentionally.

---

# Copying Is a Design Decision

When writing code, decide whether you want to share or copy.

Sharing:

```python
b = a
```

means:

```text
b refers to the same object
```

Shallow copy:

```python
b = a.copy()
```

means:

```text
b is a new outer container
inner objects may still be shared
```

Deep copy:

```python
b = copy.deepcopy(a)
```

means:

```text
b is a recursively independent copy where possible
```

There is no universally correct choice.

The correct choice depends on whether shared changes should be visible.

---

# Mutability and Equality

Mutation can change equality results.

Example:

```python
a = [1, 2]
b = [1, 2]

print(a == b)

a.append(3)

print(a == b)
```

Output:

```text
True
False
```

The identity of `a` did not change.

The value of the list object changed.

Equality compares values according to type-specific rules.

Chapter 12 will develop identity and equality in detail.

---

# Mutability and Hashing

Some objects can be dictionary keys or set elements.

Example:

```python
locations = {
    ("Paris", "France"): "city"
}
```

Tuples can be keys if their contents are hashable.

Lists cannot be dictionary keys:

```python
data = {
    [1, 2]: "value"
}
```

This raises:

```text
TypeError
```

Why?

Lists are mutable and unhashable.

Dictionary keys need stable hash behavior.

If a key could change after insertion, the dictionary's internal lookup structure would break.

We will study dictionaries and hashing later.

For now:

> Immutability often makes objects safer to use as keys.

---

# Why Immutability Is Useful

Immutable objects are easier to reason about.

If you have:

```python
x = "hello"
y = x
```

you do not worry that some operation through `y` will change the string object for `x`.

Strings cannot change in place.

Immutability helps with:

* Predictability
* Safer sharing
* Hashing
* Dictionary keys
* Set elements
* Avoiding accidental side effects
* Reasoning in concurrent code

This is why Python uses immutable objects for fundamental values such as numbers and strings.

---

# Why Mutability Is Useful

Mutable objects are useful because real programs need evolving state.

Example:

```python
shopping_cart = []
shopping_cart.append("apples")
shopping_cart.append("bread")
```

It would be inefficient and awkward to create a completely new cart object for every change in many scenarios.

Mutable objects help with:

* Building collections incrementally
* Updating configuration
* Caching
* Tracking state
* Modeling real-world entities
* Efficient algorithms

The goal is not to avoid mutation everywhere.

The goal is to control it deliberately.

---

# Side Effects

A side effect is a change that happens outside a function's return value.

Example:

```python
def add_item(items, item):
    items.append(item)
```

This function mutates the object passed in.

It has a side effect.

Example without mutation:

```python
def with_item(items, item):
    return items + [item]
```

This returns a new list and leaves the original list unchanged.

Both designs can be valid.

But the function name and documentation should make the behavior clear.

---

# API Design: Mutate or Return New

When designing a function, choose one behavior clearly.

Mutating design:

```python
def normalize_in_place(records):
    for record in records:
        record["name"] = record["name"].strip().title()
```

This changes the input objects.

Non-mutating design:

```python
def normalized(records):
    result = []
    for record in records:
        result.append({
            **record,
            "name": record["name"].strip().title(),
        })
    return result
```

This returns new dictionaries.

Neither is automatically better.

The important thing is that callers know which behavior they are getting.

---

# Defensive Copying

Sometimes a function copies input to avoid mutating caller-owned data.

Example:

```python
def sorted_copy(items):
    copied = list(items)
    copied.sort()
    return copied
```

This function does not mutate the original input list.

Another example:

```python
def store_tags(tags):
    return list(tags)
```

If you store a caller's mutable list directly, later caller mutations may affect your stored state.

Copying can prevent that.

But copying also has cost.

Use it when shared mutation would be a bug.

---

# Mutable State in Classes Preview

Most custom class instances are mutable by default.

Example:

```python
class User:
    pass

user = User()
user.name = "Ada"
user.name = "Grace"
```

The same `user` object has changing state.

Classes will be studied later.

For now, recognize that object state can be mutable not only in lists and dictionaries, but also in your own objects.

---

# Mutability and Concurrency Preview

Shared mutable state becomes especially risky when multiple execution paths interact with it.

Example:

```text
Thread A reads list
Thread B modifies list
Thread A continues with outdated assumptions
```

We will study concurrency later.

For now:

> Shared mutable objects require careful reasoning.

This is one reason immutability is valued in many programming designs.

---

# Common Misconceptions

## Misconception 1

### Reassignment mutates the object.

Reassignment usually rebinds a name.

Example:

```python
x = 10
x = 20
```

The integer object `10` did not become `20`.

The name `x` was rebound.

---

## Misconception 2

### If two names point to the same object, rebinding one changes the other.

Rebinding changes only the selected name.

Example:

```python
a = [1, 2]
b = a
b = [3, 4]
```

`a` still refers to `[1, 2]`.

---

## Misconception 3

### If two names point to the same mutable object, mutation through one name is private.

Mutation changes the object.

All aliases see that object.

Example:

```python
a = []
b = a
b.append(1)
print(a)
```

prints:

```text
[1]
```

---

## Misconception 4

### Strings change when methods like upper() are called.

String methods return new strings.

The original string is unchanged.

---

## Misconception 5

### list.sort() returns a sorted list.

`list.sort()` mutates the list in place and returns `None`.

Use `sorted(items)` if you want a new sorted list.

---

## Misconception 6

### Mutable default arguments are recreated on every call.

Default arguments are evaluated when the function is defined.

A mutable default can be shared across calls.

---

## Misconception 7

### A shallow copy makes everything independent.

A shallow copy creates a new outer container.

Nested mutable objects may still be shared.

---

# Real-world Usage

## Debugging Aliasing Bugs

If a list, dictionary, or set changes unexpectedly, ask:

```text
Who else has a reference to this object?
Was this object copied or shared?
Did a function mutate it?
```

Use `id()` while debugging to check whether two names refer to the same object.

Do not use `id()` as normal business logic.

---

## Function Design

Make mutation behavior explicit.

Names help:

```python
sort_in_place(items)
sorted_copy(items)
add_user(users, user)
with_user(users, user)
```

Documentation should say whether a function mutates arguments.

This avoids caller surprises.

---

## Configuration Objects

Mutable configuration dictionaries can create hidden coupling.

Example:

```python
defaults = {"debug": False}
config = defaults
config["debug"] = True
```

Now `defaults` also appears changed because both names refer to the same dictionary.

Use copying when defaults should remain unchanged.

---

## Caches

Caches are intentionally mutable.

Example:

```python
cache = {}

def get_user(user_id):
    if user_id not in cache:
        cache[user_id] = load_user(user_id)
    return cache[user_id]
```

The dictionary changes over time.

That mutation is the point.

The engineering question is not:

```text
Is mutation bad?
```

The question is:

```text
Is mutation controlled, expected, and safe?
```

---

## Data Processing

In data processing, decide whether each step mutates data or returns new data.

Mutation can be efficient.

Returning new objects can be easier to reason about.

Example:

```python
records.sort(key=lambda record: record["created_at"])
```

mutates `records`.

```python
sorted_records = sorted(records, key=lambda record: record["created_at"])
```

leaves `records` unchanged.

The right choice depends on ownership and expectations.

---

# Concept Connections

This chapter connects directly to earlier chapters:

```text
Chapter 09:
Objects have identity, type, and value.

Chapter 10:
Names refer to objects.

Chapter 11:
Some objects can change value while keeping identity.
```

It prepares Chapter 12:

```text
identity:
    did we keep the same object?

equality:
    do two objects have equal values?

memory diagrams:
    which names and containers refer to which objects?
```

It also prepares later topics:

```text
functions:
    argument mutation and side effects

data structures:
    list, dict, and set operations

classes:
    mutable instance state

concurrency:
    shared mutable state risks

software engineering:
    API design and defensive copying
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Mutable object | Object whose value/state can change in place |
| Immutable object | Object whose value cannot change after creation |
| Rebinding | Changing which object a name refers to |
| Mutation | Changing an object while preserving identity |
| Alias | Another reference to the same object |
| Shallow copy | New outer container, shared inner objects |
| Deep copy | Recursive copy of nested objects where possible |
| Side effect | Change beyond simply returning a value |

Core diagrams:

```text
rebinding:

x ─────▶ object #1

x ─────▶ object #2
```

```text
mutation:

x ─────▶ object #1 with old value

x ─────▶ object #1 with new value
```

```text
aliasing + mutation:

a ─┐
   ├────▶ same mutable object, changed in place
b ─┘
```

---

# Active Recall

## Easy Recall Questions

1. What is a mutable object?
2. What is an immutable object?
3. Is mutability a property of names or objects?
4. Name three mutable built-in types.
5. Name three immutable built-in types.
6. What is rebinding?
7. What is mutation?
8. What does `list.append()` do?
9. What does `str.upper()` return?
10. Why is `None` often used as a default value instead of `[]`?

---

## Deep Understanding Questions

1. Why does aliasing make mutation more important to understand?
2. Why does rebinding a parameter not affect the caller's name?
3. Why can mutating a parameter affect the caller's object?
4. Why are mutable default arguments dangerous?
5. Why does `list.sort()` return `None`?
6. Why does shallow copy not fully isolate nested structures?
7. Why are immutable objects easier to reason about?
8. Why are mutable objects still useful?
9. Why can tuples contain mutable objects even though tuples are immutable?
10. Why do dictionary keys need stable hash behavior?

---

## Explain In Your Own Words

1. Explain rebinding versus mutation using a diagram.
2. Explain why `a = b` does not copy a list.
3. Explain why `b.append(3)` can affect `a`.
4. Explain why `"hello".upper()` does not change `"hello"`.
5. Explain the mutable default argument bug.
6. Explain shallow copy with a nested list.
7. Explain when you would choose mutation over returning a new object.

---

## Predict-the-Output Questions

### Question 1

```python
a = [1, 2]
b = a
b.append(3)

print(a)
```

What is printed?

Answer:

```text
[1, 2, 3]
```

Reason:

`a` and `b` refer to the same list object. The list was mutated.

---

### Question 2

```python
a = [1, 2]
b = a
b = [3, 4]

print(a)
print(b)
```

What is printed?

Answer:

```text
[1, 2]
[3, 4]
```

Reason:

`b = [3, 4]` rebinds `b` to a new list. It does not mutate the list referred to by `a`.

---

### Question 3

```python
text = "hello"
text.upper()
print(text)
```

What is printed?

Answer:

```text
hello
```

Reason:

Strings are immutable. `upper()` returns a new string, but the result was not assigned.

---

### Question 4

```python
items = [3, 1, 2]
result = items.sort()

print(items)
print(result)
```

What is printed?

Answer:

```text
[1, 2, 3]
None
```

Reason:

`sort()` mutates the list in place and returns `None`.

---

### Question 5

```python
def add(value, values=[]):
    values.append(value)
    return values

print(add(1))
print(add(2))
```

What is printed?

Answer:

```text
[1]
[1, 2]
```

Reason:

The default list is created once when the function is defined and reused across calls.

---

### Question 6

```python
a = [[1, 2]]
b = a.copy()

a[0].append(3)

print(b)
```

What is printed?

Answer:

```text
[[1, 2, 3]]
```

Reason:

`copy()` made a shallow copy of the outer list. The inner list is shared.

---

# Mental Model Questions

1. Draw `items = [1, 2]; items.append(3)`.
2. Draw `items = [1, 2]; items = [3, 4]`.
3. Draw `a = []; b = a; b.append(1)`.
4. Draw a function call that mutates a list argument.
5. Draw a function call that rebinds a parameter.
6. Draw a shallow copy of a nested list.
7. Draw a tuple that contains a mutable list.

---

# Practical Exercises

## Exercise 1

Compare rebinding and mutation:

```python
a = [1, 2]
b = a
a = [3, 4]

print(b)
```

Then:

```python
a = [1, 2]
b = a
a.append(3)

print(b)
```

Explain why the outputs differ.

---

## Exercise 2

String immutability:

```python
text = "python"
new_text = text.replace("p", "P")

print(text)
print(new_text)
```

Explain why `text` is unchanged.

---

## Exercise 3

Function parameter mutation:

```python
def add_done(tasks):
    tasks.append("done")

items = []
add_done(items)
print(items)
```

Draw the name-object diagram during the function call.

---

## Exercise 4

Mutable default bug:

```python
def collect(value, bucket=[]):
    bucket.append(value)
    return bucket

print(collect("a"))
print(collect("b"))
print(collect("c"))
```

Predict the output.

Then rewrite the function using `None` as the default.

---

## Exercise 5

Shallow copy:

```python
a = [[1], [2]]
b = a.copy()

b[0].append(99)

print(a)
print(b)
```

Explain why both outputs show `99`.

---

## Exercise 6

Deep copy:

```python
import copy

a = [[1], [2]]
b = copy.deepcopy(a)

b[0].append(99)

print(a)
print(b)
```

Explain why the outputs differ from Exercise 5.

---

## Exercise 7

API design:

Write two functions:

```python
def add_item_in_place(items, item):
    ...

def with_item(items, item):
    ...
```

The first should mutate `items`.

The second should return a new list.

Test both with aliases and explain the difference.

---

# Summary

In this chapter we learned:

* Mutability is a property of objects.
* Mutable objects can change in place while keeping identity.
* Immutable objects cannot change value after creation.
* Rebinding changes which object a name refers to.
* Mutation changes the object itself.
* Lists, dictionaries, and sets are mutable.
* Integers, strings, booleans, and tuples are immutable.
* Tuples can contain references to mutable objects.
* Aliasing makes mutation visible through multiple names.
* Function parameters are local names bound to argument objects.
* Rebinding a parameter does not affect the caller's name.
* Mutating a passed object can affect the caller-visible object.
* Mutable default arguments can accidentally share state across calls.
* In-place methods often return `None`.
* Assignment does not copy objects.
* Shallow copies share nested objects.
* Deep copies recursively copy nested objects where possible.
* Mutation is useful, but shared mutable state must be controlled.

The core distinction is:

```text
rebinding:
    name ─────▶ different object

mutation:
    name ─────▶ same object, changed value/state
```

You now understand why aliases and mutable objects can produce surprising behavior.

---

# Preview of Chapter 12

We now have three essential pieces:

```text
objects have identity, type, and value
names refer to objects
some objects can mutate in place
```

Next we combine these ideas into a complete reasoning toolkit:

* Identity
* Equality
* `is`
* `==`
* Memory diagrams
* Object relationships

Chapter 12 will make the difference between "same object" and "equal value" precise.
