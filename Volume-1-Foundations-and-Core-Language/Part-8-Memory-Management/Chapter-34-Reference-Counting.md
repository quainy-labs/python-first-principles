# Chapter 34 — Reference Counting

---

# Learning Objectives

By the end of this chapter, you should understand:

* What reference counting is.
* Why CPython uses reference counting.
* What increases an object's reference count.
* What decreases an object's reference count.
* How assignment affects references.
* How rebinding affects references.
* How containers affect reference counts.
* How function calls temporarily affect references.
* Why returned objects can survive after a function frame disappears.
* What happens when a reference count reaches zero.
* Why reference counting handles many objects immediately.
* Why reference cycles are a problem for plain reference counting.
* Why `sys.getrefcount()` can be surprising.
* How reference counting connects to garbage collection.

This chapter is specifically about CPython's memory-management behavior.

Python the language defines object behavior.

CPython, the most widely used implementation, uses reference counting as a primary memory-management mechanism.

Other Python implementations may use different strategies.

---

# Concept Overview

Reference counting tracks how many references point to an object.

Example:

```python
a = []
```

Conceptually:

```text
a ─────▶ list object
```

The list object has at least one reference.

Now:

```python
b = a
```

Conceptually:

```text
a ─────┐
       v
b ─▶ list object
```

Now the same list object has another reference.

If one name is rebound:

```python
a = None
```

Conceptually:

```text
a ─▶ None

b ─▶ list object
```

The list still exists because `b` refers to it.

If `b` is also rebound:

```python
b = None
```

Now no user-visible names refer to the list.

In CPython, when an object's reference count reaches zero, the object can be deallocated immediately.

---

# Why Reference Counting Exists

Python creates objects constantly.

Examples:

```python
numbers = [1, 2, 3]
name = "Ada"
result = 10 + 20
data = {"active": True}
```

Objects require memory.

If unreachable objects were never cleaned up, programs would keep using more memory.

Reference counting answers:

```text
Is anything still using this object?
```

If the count reaches zero:

```text
no references remain
```

Then the object can be cleaned up.

This is simple and often immediate.

That immediacy is one reason CPython object cleanup often appears predictable.

But reference counting alone cannot solve every memory problem.

Reference cycles require another mechanism, covered in the next chapter.

---

# Reference Count Mental Model

Think of every object as having a counter:

```text
object
    reference count: number of references pointing here
```

Example:

```python
items = []
```

Model:

```text
items ─────▶ list object
             refcount: at least 1
```

Example:

```python
alias = items
```

Model:

```text
items ─────┐
           v
alias ─▶ list object
          refcount: increased
```

Do not treat the exact number as the most important thing.

Python internals, temporary references, function calls, and implementation details can affect exact counts.

The important model is:

```text
new reference -> count goes up
reference removed -> count goes down
count reaches zero -> object can be deallocated
```

---

# Assignment Increases References

Assignment binds a name to an object.

Example:

```python
items = []
```

The name `items` now refers to the list object.

That adds a reference.

Another assignment:

```python
other = items
```

The name `other` now refers to the same list object.

That adds another reference.

Important:

This does not copy the list.

It adds another reference to the same object.

This is why:

```python
items.append("x")
print(other)
```

prints:

```text
['x']
```

Both names refer to the same object.

---

# Rebinding Decreases Old References

Rebinding a name removes its reference to the old object and creates a reference to a new object.

Example:

```python
items = []
other = items

items = None
```

Before rebinding:

```text
items ─────┐
           v
other ─▶ list object
```

After rebinding:

```text
items ─▶ None

other ─▶ list object
```

The list object's reference count decreased because `items` no longer points to it.

The list still exists because `other` still points to it.

If:

```python
other = None
```

then the list may have no remaining references and can be deallocated.

---

# Deleting Names

`del` removes a name binding.

Example:

```python
items = []
alias = items

del items
```

After `del items`, the name `items` no longer exists.

The list still exists because `alias` refers to it.

```python
print(alias)
```

Output:

```text
[]
```

Now:

```python
del alias
```

If no other references exist, the list can be cleaned up.

`del` does not directly destroy an object.

It removes a reference.

If that was the last reference, the object can be destroyed.

---

# Containers Increase References

Containers store references to objects.

Example:

```python
name = "Ada"
items = [name]
```

Now the string object `"Ada"` is referenced by:

* the name `name`
* the list slot `items[0]`

Conceptually:

```text
name ───────▶ "Ada"

items ──────▶ list object
              └── index 0 ─▶ "Ada"
```

Putting an object into a list, tuple, dictionary, or set adds a reference from that container.

Removing it from the container removes that reference.

Example:

```python
items.pop()
```

Now the list no longer references `"Ada"`.

If `name` still exists, the string still has a reference.

---

# Dictionaries Hold References To Keys And Values

A dictionary stores references to both keys and values.

Example:

```python
key = "name"
value = "Ada"

data = {key: value}
```

Conceptually:

```text
key ─────▶ "name"
value ───▶ "Ada"

data ────▶ dict object
           ├── key reference ───▶ "name"
           └── value reference ─▶ "Ada"
```

Deleting a dictionary entry removes the dictionary's references to both key and value:

```python
del data["name"]
```

If no other references exist, those objects may become eligible for cleanup.

In practice, strings and small integers may have implementation-level references.

Do not rely on exact cleanup behavior for such objects.

Focus on the reference model.

---

# Function Calls Add Temporary References

Passing an object to a function creates a parameter binding.

Example:

```python
def show(items):
    print(items)

values = []
show(values)
```

During the call:

```text
module frame
└── values ─▶ list object

show frame
└── items ──▶ same list object
```

The parameter `items` is another reference while the function is running.

When the function returns, the frame disappears.

The parameter reference disappears.

The list remains alive because `values` still refers to it.

This explains why function calls can temporarily increase reference counts.

---

# Return Values Transfer References

When a function returns an object, the caller can keep a reference to it.

Example:

```python
def make_list():
    items = []
    return items

result = make_list()
```

During the call:

```text
make_list frame
└── items ─▶ list object
```

During return, the object is returned to the caller.

After assignment:

```text
module frame
└── result ─▶ list object
```

The local name `items` disappears.

The object survives because `result` now refers to it.

If the caller ignores the return value:

```python
make_list()
```

the returned list may become unreachable immediately.

---

# Temporary Objects

Expressions can create temporary objects.

Example:

```python
result = [1, 2] + [3, 4]
```

Conceptually:

```text
create list [1, 2]
create list [3, 4]
create new concatenated list [1, 2, 3, 4]
bind result to concatenated list
temporary lists may become unreachable
```

Python manages these temporary objects.

You usually do not think about them directly.

But they explain why expression evaluation can affect reference counts briefly.

This also explains why exact reference counts are not always intuitive.

---

# `sys.getrefcount()`

CPython exposes a function for inspecting reference counts:

```python
import sys

items = []
print(sys.getrefcount(items))
```

The result may be surprising.

Why?

Calling `sys.getrefcount(items)` temporarily passes `items` as an argument.

That call itself creates an additional reference.

So the number reported is usually one higher than you might expect.

Example:

```python
import sys

items = []
alias = items

print(sys.getrefcount(items))
```

The exact number is not important.

Use `getrefcount()` only as a learning/debugging tool.

Do not write application logic that depends on exact reference counts.

---

# Reference Count Reaches Zero

In CPython, when an object's reference count reaches zero, the object can be deallocated immediately.

Example:

```python
def make():
    data = [1, 2, 3]

make()
```

The list object is created inside `make()`.

The local name `data` refers to it.

When the function returns, the frame disappears.

The local reference disappears.

If nothing else refers to the list, its reference count reaches zero.

CPython can clean it up.

This immediate cleanup is one practical difference between CPython and some other garbage-collected runtimes.

---

# Object Finalization Preview

Some objects perform cleanup when destroyed.

For example, file objects may release operating-system resources.

Example:

```python
file = open("example.txt")
```

You should not rely on reference counting alone to close files.

Use context managers:

```python
with open("example.txt") as file:
    data = file.read()
```

Context managers are covered later.

For now, understand:

Reference counting may clean objects quickly in CPython, but explicit resource management is still important.

Memory cleanup and external resource cleanup are related but not identical.

---

# Reference Cycles

Reference counting has a weakness.

It cannot handle cycles by itself.

Example:

```python
a = []
b = []

a.append(b)
b.append(a)
```

Conceptually:

```text
a ─▶ list A ─▶ list B
     ▲          │
     └──────────┘

b ─▶ list B
```

Now remove the names:

```python
del a
del b
```

The two lists still refer to each other.

Their reference counts are not zero.

But the program cannot reach them anymore.

This is a reference cycle.

Plain reference counting cannot clean it up because each object keeps the other object's count above zero.

This is why CPython also has a cyclic garbage collector.

That is the next chapter.

---

# Cycles In Real Programs

Cycles can happen naturally.

Example:

```python
parent = {"children": []}
child = {"parent": parent}

parent["children"].append(child)
```

Conceptually:

```text
parent dict ─▶ children list ─▶ child dict
     ▲                           │
     └──────── parent key ◀──────┘
```

This structure is useful.

Parent-child relationships often need links in both directions.

The cycle is not automatically a bug.

But it means reference counting alone is not enough.

Python needs cycle detection.

---

# Reference Counting And Performance

Reference counting has tradeoffs.

Benefits:

* Many objects are cleaned up immediately.
* Object lifetime is often predictable in CPython.
* Simple unreachable objects do not wait for a later garbage collection pass.

Costs:

* Reference counts must be updated constantly.
* Cycles require another mechanism.
* Exact behavior is implementation-specific.
* Reference count changes affect CPython internals and performance.

As a Python programmer, you usually do not manually manage reference counts.

But understanding the model helps you reason about memory behavior.

---

# Reference Counting And Aliasing

Aliasing creates multiple references to the same object.

Example:

```python
a = {"count": 0}
b = a
c = a
```

Conceptually:

```text
a ─────┐
b ─────┼──▶ dict object
c ─────┘
```

The dictionary has multiple references.

Mutation through any name affects the same object:

```python
b["count"] += 1

print(c)
```

Output:

```text
{'count': 1}
```

Reference counting explains why the object remains alive as long as any alias still exists.

Names and references explain why all aliases see the same mutation.

---

# Reference Counting And Containers

Containers can keep objects alive even when no direct name refers to them.

Example:

```python
items = []
items.append({"name": "Ada"})
```

The dictionary has no direct name.

But it is still alive because the list refers to it.

Conceptually:

```text
items ─▶ list object
          └── index 0 ─▶ dict object {"name": "Ada"}
```

If you remove it:

```python
items.pop()
```

and no other references exist, the dictionary can be cleaned up.

This is why containers are central to memory behavior.

They are reference owners.

---

# Reference Counting And Closures

Closures can keep objects alive.

Example:

```python
def make_holder():
    data = []

    def add(value):
        data.append(value)
        return data

    return add

holder = make_holder()
```

The `make_holder` frame is gone.

But `data` is still alive.

Why?

The returned function has a closure that refers to it.

Conceptually:

```text
holder ─▶ function object
          └── closure ─▶ list object []
```

The closure reference contributes to the object's lifetime.

This is not special magic.

It is still references.

---

# Reference Counting And Globals

Global names can keep objects alive for a long time.

Example:

```python
cache = {}

def store(key, value):
    cache[key] = value
```

Objects stored in `cache` remain reachable through the global dictionary.

Even if local function frames end, the global dictionary still owns references.

This is why caches and registries can grow memory usage.

Reference counting will not remove objects that are still reachable.

From Python's perspective, the program may still use them.

If a cache should forget old entries, the program must remove them intentionally or use a structure designed for that behavior.

---

# Common Mistakes

## Misconception 1

### Reference counting counts variables only.

It counts references, not just names.

References can come from names, containers, frames, closures, and temporary internal operations.

## Misconception 2

### `del` destroys an object directly.

`del` removes a reference.

The object is destroyed only if no references remain.

## Misconception 3

### A returned object should disappear when the function frame disappears.

The local frame disappears.

The object survives if the caller receives and stores a reference.

## Misconception 4

### Reference counting handles all unreachable objects.

Reference counting alone cannot handle reference cycles.

CPython also uses cyclic garbage collection.

## Misconception 5

### `sys.getrefcount()` gives the exact intuitive count.

Calling `getrefcount()` itself adds a temporary reference.

Other implementation details may also affect counts.

## Misconception 6

### If memory grows, reference counting is broken.

Memory growth can happen because objects are still reachable.

Global caches, long-lived containers, and retained closures can all keep objects alive.

## Misconception 7

### Reference counting behavior is identical in every Python implementation.

Reference counting is a CPython implementation detail.

Other implementations may manage memory differently.

---

# Real-world Usage

## Understanding Mutating Functions

```python
def add_error(errors, message):
    errors.append(message)
```

The list remains alive because the caller owns it.

The function temporarily adds a parameter reference.

## Understanding Caches

```python
cache[user_id] = user
```

The cache dictionary keeps the user object alive.

## Understanding Temporary Objects

```python
result = transform(load_data())
```

Intermediate objects may exist only long enough to be passed into another function.

## Understanding Closures

```python
handler = make_handler(config)
```

The returned handler may keep `config` alive through closure references.

## Understanding Cycles

```python
parent.children.append(child)
child.parent = parent
```

Both objects may reference each other.

Reference counting alone cannot clean such cycles if they become unreachable.

---

# Internal Mechanics

In CPython, every object has reference-count information.

At a high level:

```text
new reference created -> increment reference count
reference removed     -> decrement reference count
count reaches zero    -> deallocate object
```

Examples that can increment:

* binding a name
* inserting into a container
* passing as an argument
* creating a closure reference
* temporary interpreter operations

Examples that can decrement:

* rebinding a name
* deleting a name
* removing from a container
* function frame ending
* temporary reference ending

This process is automatic.

Python code does not manually increment or decrement reference counts.

Understanding it helps you reason about when objects can become unreachable.

---

# Concept Connections

Reference counting connects to earlier chapters:

* Names and references: names contribute references.
* Mutability: aliases can mutate the same referenced object.
* Identity: multiple references can point to one object.
* Functions: parameter bindings temporarily add references.
* Scope: frame removal drops local references.
* Closures: closure cells can keep objects alive.
* Lists and dictionaries: containers own references to contained objects.
* Stack vs heap: frames hold references to managed objects.

Reference counting prepares you for:

* Garbage collection.
* Reference cycles.
* Object lifecycle.
* Weak references.
* Resource management.
* CPython internals.

---

# Active Recall

## Easy Recall Questions

1. What is reference counting?
2. What implementation is reference counting most associated with?
3. What happens when assignment binds a name to an object?
4. What happens to the old reference when a name is rebound?
5. Does `del` destroy an object directly?
6. How can a list keep an object alive?
7. Why can function calls temporarily increase reference counts?
8. What happens when a reference count reaches zero in CPython?
9. What is a reference cycle?
10. Why can `sys.getrefcount()` be surprising?

## Deep Understanding Questions

1. Why is reference counting about references rather than variables?
2. Why can an object survive after the function that created it returns?
3. Why do containers affect object lifetime?
4. Why can reference cycles defeat plain reference counting?
5. Why should application logic not depend on exact reference counts?
6. Why can global caches cause memory growth?

## Predict-the-Output Questions

### Question 1

```python
a = []
b = a
a = None

print(b)
```

### Question 2

```python
items = []
container = [items]
items.append("x")

print(container)
```

### Question 3

```python
def make():
    data = []
    return data

result = make()
result.append(1)

print(result)
```

### Question 4

```python
a = []
b = []
a.append(b)
b.append(a)

del a
del b
```

What memory-management issue does this create?

### Question 5

```python
def store(container):
    value = {"temporary": True}
    container.append(value)

items = []
store(items)

print(items)
```

Why does the dictionary survive?

---

# Practical Exercises

## Exercise 1

Draw reference diagrams for:

```python
a = []
b = a
c = b
```

Then rebind each name one by one and explain when the list can be cleaned up.

## Exercise 2

Create a list containing a dictionary.

Delete the direct dictionary name but keep the list.

Explain why the dictionary survives.

## Exercise 3

Use `sys.getrefcount()` on a list.

Add an alias and observe the count.

Explain why the exact number may be higher than expected.

## Exercise 4

Write a function that creates and returns a list.

Explain how the returned object survives the function frame.

## Exercise 5

Write a function that creates a list but does not return it.

Explain why it can be cleaned up after the function returns.

## Exercise 6

Create a reference cycle using two lists.

Draw the object graph.

## Exercise 7

Create a parent-child dictionary cycle.

Explain why this kind of structure can be useful but complicates cleanup.

## Exercise 8

Create a closure that keeps a list alive.

Explain where the reference comes from.

## Exercise 9

Create a global cache dictionary.

Store objects in it.

Explain why those objects remain alive.

## Exercise 10

Explain why this statement is incomplete:

```text
Python deletes an object when a variable goes out of scope.
```

Rewrite it using references and reachability.

---

# Summary

In this chapter we learned:

* Reference counting tracks how many references point to an object.
* CPython uses reference counting as a primary memory-management mechanism.
* Assignment creates references.
* Rebinding removes a reference to the old object and creates a reference to another object.
* `del` removes a reference, not necessarily the object.
* Containers store references and can keep objects alive.
* Dictionaries hold references to keys and values.
* Function calls create temporary parameter references.
* Returned objects survive if callers keep references.
* Temporary objects can briefly affect reference counts.
* `sys.getrefcount()` includes a temporary reference from the call itself.
* When a reference count reaches zero in CPython, the object can be deallocated.
* Reference cycles cannot be solved by plain reference counting.
* Closures, globals, and containers can keep objects alive.

Core model:

```text
object
    |
    ├── reference from name
    ├── reference from container
    ├── reference from frame
    └── reference from closure

reference count reaches zero -> object can be cleaned up
```

Reference counting turns the idea "references keep objects alive" into a concrete CPython mechanism.

---

# Preview of Chapter 35

Next we study garbage collection.

Reference counting handles many objects immediately, but cycles require more help.

Chapter 35 explains how Python detects unreachable object groups that still reference each other.

We will study:

* Why reference cycles are difficult.
* What cyclic garbage collection does.
* What reachability means.
* How container objects participate in cycles.
* Why simple objects usually do not need cycle tracking.
* How `gc` can be used for learning and debugging.
* Why deterministic cleanup should use context managers, not garbage collection timing.

The transition is direct:

```text
reference counting handles simple unreachable objects
garbage collection handles unreachable cycles
```

Garbage collection completes the basic memory cleanup model.
