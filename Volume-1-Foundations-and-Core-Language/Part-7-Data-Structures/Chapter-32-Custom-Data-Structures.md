# Chapter 32 — Custom Data Structures

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a custom data structure is.
* Why custom data structures exist.
* How custom structures differ from built-in containers.
* How to wrap built-in collections behind a clearer API.
* What invariants are.
* Why invariants matter.
* How to design a stack abstraction.
* How to design a queue abstraction.
* How to design a bounded history structure.
* How to design small record-like structures without overengineering.
* How to choose internal storage.
* How to expose useful methods and hide implementation details.
* When not to build a custom data structure.
* How this topic prepares you for classes.

This chapter closes the data structures part.

So far, we have used existing structures.

Now we ask how to design one.

---

# Concept Overview

A custom data structure is a structure you design for a specific purpose.

It may use built-in collections internally.

Example:

```python
class Stack:
    def __init__(self):
        self._items = []

    def push(self, item):
        self._items.append(item)

    def pop(self):
        return self._items.pop()

    def is_empty(self):
        return not self._items
```

This `Stack` uses a list internally.

But the outside code does not need to know that.

Outside code can say:

```python
stack = Stack()
stack.push("first")
stack.push("second")

print(stack.pop())
```

Output:

```text
second
```

The custom structure gives a name and behavior to a pattern:

```text
last in, first out
```

The list is storage.

The stack is the idea.

---

# Why Custom Data Structures Exist

Built-in collections are general.

Your program's needs are often specific.

Examples:

```text
A stack should expose push and pop.
A queue should expose enqueue and dequeue.
A bounded history should keep only the last N items.
A leaderboard should keep scores ordered.
A cache should limit size and remove old entries.
A graph should make relationships explicit.
```

You can write all of these directly with lists and dictionaries.

But direct use can spread rules across the codebase.

Example:

```python
items.append(task)
task = items.pop(0)
```

This is a queue pattern.

But the list itself does not say:

```text
I am a queue.
```

A custom structure can make that meaning explicit:

```python
queue.enqueue(task)
task = queue.dequeue()
```

Good custom structures create clarity.

Bad custom structures create unnecessary layers.

---

# Storage vs Interface

A custom data structure has two sides:

* Internal storage.
* External interface.

Internal storage is how the structure keeps data.

External interface is how other code uses it.

Example:

```python
class Queue:
    def __init__(self):
        self._items = []

    def enqueue(self, item):
        self._items.append(item)

    def dequeue(self):
        return self._items.pop(0)
```

Internal storage:

```text
self._items list
```

External interface:

```text
enqueue()
dequeue()
```

The interface expresses intent.

The storage is an implementation detail.

Later, you could replace the internal list with a `deque` without changing outside code:

```python
from collections import deque

class Queue:
    def __init__(self):
        self._items = deque()

    def enqueue(self, item):
        self._items.append(item)

    def dequeue(self):
        return self._items.popleft()
```

Outside code still calls:

```python
queue.enqueue(item)
queue.dequeue()
```

That is the value of an interface.

---

# Invariants

An invariant is a rule that should always remain true.

Examples:

```text
A stack removes the most recently added item first.
A queue removes the earliest added item first.
A bounded history never contains more than N items.
A sorted collection remains sorted after insertion.
A non-empty collection has length greater than zero.
```

Custom data structures often exist to protect invariants.

Example:

```python
history = []
limit = 3
```

Everywhere you add to `history`, you must remember:

```python
history.append(item)

if len(history) > limit:
    history.pop(0)
```

If that logic is repeated in many places, someone will eventually forget it.

A custom structure can protect the rule:

```python
history.add(item)
```

The `History` object handles the size limit internally.

---

# A Simple Stack

A stack is last in, first out.

Use cases:

* undo history
* parsing
* depth-first search
* call-stack mental models
* backtracking

Implementation:

```python
class Stack:
    def __init__(self):
        self._items = []

    def push(self, item):
        self._items.append(item)

    def pop(self):
        return self._items.pop()

    def peek(self):
        return self._items[-1]

    def is_empty(self):
        return not self._items

    def size(self):
        return len(self._items)
```

Usage:

```python
stack = Stack()
stack.push("a")
stack.push("b")

print(stack.peek())
print(stack.pop())
print(stack.pop())
print(stack.is_empty())
```

Output:

```text
b
b
a
True
```

The stack hides the list operations behind stack language.

---

# Stack Invariants

The stack invariant is:

```text
pop returns the most recently pushed item that has not yet been popped
```

The implementation uses:

```python
self._items.append(item)
self._items.pop()
```

Both operate at the end of the list.

That preserves last-in, first-out behavior.

If someone accessed `_items` directly and inserted at the front:

```python
stack._items.insert(0, "bad")
```

the stack's meaning could be broken.

The leading underscore in `_items` communicates:

```text
internal detail, do not use directly
```

Python does not enforce this strongly.

It is a convention.

The real protection comes from disciplined API design.

---

# Handling Empty Stack Errors

What should happen if `pop()` is called on an empty stack?

Current behavior:

```python
stack = Stack()
stack.pop()
```

The underlying list raises:

```text
IndexError
```

You can allow that.

Or you can raise a clearer error:

```python
class Stack:
    def __init__(self):
        self._items = []

    def pop(self):
        if not self._items:
            raise IndexError("pop from empty stack")

        return self._items.pop()
```

Design question:

```text
Should the custom structure expose the underlying error,
or provide a domain-specific error message?
```

For beginner structures, clear messages help.

---

# A Simple Queue

A queue is first in, first out.

Use cases:

* task processing
* breadth-first search
* event handling
* producer-consumer workflows

Implementation with `deque`:

```python
from collections import deque

class Queue:
    def __init__(self):
        self._items = deque()

    def enqueue(self, item):
        self._items.append(item)

    def dequeue(self):
        if not self._items:
            raise IndexError("dequeue from empty queue")

        return self._items.popleft()

    def is_empty(self):
        return not self._items

    def size(self):
        return len(self._items)
```

Usage:

```python
queue = Queue()
queue.enqueue("first")
queue.enqueue("second")

print(queue.dequeue())
print(queue.dequeue())
```

Output:

```text
first
second
```

The queue interface makes the behavior obvious.

---

# Queue Invariants

The queue invariant is:

```text
dequeue returns the earliest enqueued item that has not yet been dequeued
```

The implementation uses:

```python
append()    # add to right
popleft()  # remove from left
```

This preserves first-in, first-out order.

The outside user does not need to know whether the queue uses:

* list
* deque
* linked nodes
* another structure

The user only needs the queue behavior.

This separation is one reason custom structures matter.

---

# Bounded History

A bounded history keeps only the most recent N items.

Example:

```python
from collections import deque

class History:
    def __init__(self, limit):
        self._items = deque(maxlen=limit)

    def add(self, item):
        self._items.append(item)

    def latest(self):
        if not self._items:
            return None

        return self._items[-1]

    def all(self):
        return list(self._items)
```

Usage:

```python
history = History(limit=3)

history.add("a")
history.add("b")
history.add("c")
history.add("d")

print(history.all())
print(history.latest())
```

Output:

```text
['b', 'c', 'd']
d
```

The invariant:

```text
history length never exceeds limit
```

The `deque(maxlen=limit)` enforces that internally.

---

# Counter Wrapper

Sometimes a standard tool is correct, but a custom wrapper gives domain meaning.

Example:

```python
from collections import Counter

class WordCounts:
    def __init__(self):
        self._counts = Counter()

    def add_text(self, text):
        for word in text.split():
            self._counts[word.casefold()] += 1

    def count(self, word):
        return self._counts[word.casefold()]

    def most_common(self, n):
        return self._counts.most_common(n)
```

Usage:

```python
counts = WordCounts()
counts.add_text("Python python Java")

print(counts.count("PYTHON"))
print(counts.most_common(1))
```

Output:

```text
2
[('python', 2)]
```

The wrapper adds rules:

* split text into words
* normalize case
* expose word-specific methods

The internal `Counter` remains an implementation detail.

---

# Sorted Scores

Suppose we want to keep scores sorted as they arrive.

Implementation:

```python
import bisect

class SortedScores:
    def __init__(self):
        self._scores = []

    def add(self, score):
        bisect.insort(self._scores, score)

    def lowest(self):
        return self._scores[0]

    def highest(self):
        return self._scores[-1]

    def all(self):
        return list(self._scores)
```

Usage:

```python
scores = SortedScores()
scores.add(90)
scores.add(70)
scores.add(80)

print(scores.all())
print(scores.lowest())
print(scores.highest())
```

Output:

```text
[70, 80, 90]
70
90
```

The invariant:

```text
self._scores is always sorted
```

That invariant holds because all insertion goes through `add()`.

---

# Priority Task Queue

A priority task queue returns the highest-priority task first.

With `heapq`, lower numbers come out first by default.

Example:

```python
import heapq

class PriorityQueue:
    def __init__(self):
        self._items = []
        self._counter = 0

    def push(self, priority, item):
        self._counter += 1
        heapq.heappush(self._items, (priority, self._counter, item))

    def pop(self):
        if not self._items:
            raise IndexError("pop from empty priority queue")

        priority, counter, item = heapq.heappop(self._items)
        return item

    def is_empty(self):
        return not self._items
```

Usage:

```python
queue = PriorityQueue()
queue.push(2, "write tests")
queue.push(1, "fix bug")

print(queue.pop())
print(queue.pop())
```

Output:

```text
fix bug
write tests
```

The counter prevents Python from comparing task objects when priorities tie.

This wrapper hides the tuple details from users.

---

# Designing The Interface

A custom data structure should expose methods that match the concept.

Stack:

```text
push
pop
peek
is_empty
```

Queue:

```text
enqueue
dequeue
is_empty
```

History:

```text
add
latest
all
```

Priority queue:

```text
push
pop
is_empty
```

Good method names reduce cognitive load.

The user should not need to think:

```text
Is this append, insert, pop(0), popleft, or heappush?
```

The custom structure translates domain language into storage operations.

---

# Choosing Internal Storage

Choose internal storage based on operations.

Questions:

```text
Do I need ordering?
Do I need fast membership?
Do I need key lookup?
Do I need efficient front removal?
Do I need priority ordering?
Do I need sorted insertion?
Do I need uniqueness?
```

Examples:

| Need | Internal storage |
|---|---|
| stack | list |
| queue | deque |
| counting | Counter |
| grouping | defaultdict(list) |
| priority | heapq with list |
| sorted list | list with bisect |
| membership | set |
| key lookup | dict |

The public API should be chosen for meaning.

The internal storage should be chosen for behavior.

---

# Protecting Invariants

An invariant is protected when all changes go through controlled methods.

Example:

```python
class PositiveNumbers:
    def __init__(self):
        self._values = []

    def add(self, value):
        if value <= 0:
            raise ValueError("value must be positive")

        self._values.append(value)

    def all(self):
        return list(self._values)
```

Invariant:

```text
all stored numbers are positive
```

If outside code directly mutates `_values`, it can break the invariant:

```python
numbers._values.append(-10)
```

Python allows this.

The underscore convention communicates that outside code should not do it.

Later chapters on classes and properties will add better tools for controlling access.

---

# Returning Copies

Be careful when exposing internal mutable storage.

Bad:

```python
class History:
    def __init__(self):
        self._items = []

    def all(self):
        return self._items
```

Outside code can mutate internal storage:

```python
items = history.all()
items.clear()
```

Better:

```python
def all(self):
    return list(self._items)
```

This returns a shallow copy.

The caller can mutate the returned list without clearing the internal structure.

This is an important design habit:

```text
do not leak mutable internals accidentally
```

---

# Small Record Structures

Sometimes a structure is mostly data with a little meaning.

Before classes are studied deeply, you can still understand the design question.

Example:

```python
user = {
    "id": 1,
    "email": "ada@example.com",
    "active": True,
}
```

A dictionary is flexible.

But it has risks:

```python
user["emial"] = "typo@example.com"
```

The typo creates a new key.

Later, dataclasses and classes will provide stronger structure.

For now:

Use dictionaries for flexible records and external data.

Use custom classes when behavior and invariants matter.

---

# When Not To Build A Custom Structure

Do not build a custom structure just to rename a built-in type.

Unnecessary:

```python
class MyList:
    def __init__(self):
        self.items = []
```

This adds no useful behavior.

A custom structure should earn its existence by providing at least one of these:

* Clearer domain language.
* Protected invariants.
* Hidden implementation details.
* A smaller safer interface.
* Reusable behavior.
* Better fit for a repeated pattern.

If the built-in collection is already clear, use it directly.

Simple is good.

Thin wrappers without purpose are not.

---

# Custom Structures Before Full OOP

This chapter uses classes before the full object-oriented programming volume.

That is intentional and limited.

For now, understand:

```text
class -> creates a new kind of object
__init__ -> initializes the object
self -> the object being operated on
method -> function attached to the object
```

The full class model comes later.

Here, classes are used only to demonstrate how collections can be wrapped into named structures.

Do not worry yet about inheritance, MRO, descriptors, metaclasses, or the full data model.

Those come later in the planned order.

---

# Common Mistakes

## Misconception 1

### A custom data structure must be built from scratch.

Most practical custom structures wrap existing collections.

Using a list, dict, set, deque, Counter, or heap internally is normal.

## Misconception 2

### A custom structure is better because it is custom.

Custom code has maintenance cost.

Use a custom structure only when it clarifies behavior or protects rules.

## Misconception 3

### Internal storage should be exposed directly for convenience.

Exposing mutable internals lets outside code break invariants.

Return copies or provide focused methods when needed.

## Misconception 4

### Method names should mirror internal storage methods.

Method names should mirror the concept.

A queue should expose `enqueue()` and `dequeue()`, even if internally it uses `append()` and `popleft()`.

## Misconception 5

### Underscore attributes are private in a strict sense.

In Python, a leading underscore is a convention.

It communicates internal use.

It does not make access impossible.

## Misconception 6

### Invariants are only for advanced programs.

Every useful data structure has rules.

Naming and protecting those rules is basic good design.

---

# Real-world Usage

## Undo Stack

```python
undo = Stack()
undo.push(previous_state)
state = undo.pop()
```

## Task Queue

```python
queue = Queue()
queue.enqueue(task)
task = queue.dequeue()
```

## Recent History

```python
history = History(limit=50)
history.add(event)
```

## Priority Scheduling

```python
queue = PriorityQueue()
queue.push(priority=1, item=urgent_task)
```

## Domain-Specific Counting

```python
counts = WordCounts()
counts.add_text(document)
```

## Sorted Scores

```python
scores = SortedScores()
scores.add(95)
```

The value is not just storage.

The value is giving a program concept a clear home.

---

# Internal Mechanics

At this stage, custom structures are mostly composition.

Composition means:

```text
one object contains another object and uses it to do work
```

Example:

```python
class Stack:
    def __init__(self):
        self._items = []
```

The `Stack` object contains a list.

The stack methods delegate to list methods:

```python
push -> list.append
pop  -> list.pop
```

This is different from inheritance.

Inheritance comes later.

For now, prefer composition:

```text
custom structure has internal storage
custom methods enforce meaning
```

---

# Concept Connections

Custom data structures connect to earlier chapters:

* Lists: internal storage for stacks and sorted collections.
* Dictionaries: records, indexes, and mappings.
* Sets: membership and uniqueness.
* Specialized collections: `deque`, `Counter`, `defaultdict`, `heapq`, and `bisect` are useful internals.
* Functions: methods are functions attached to objects.
* Scope: methods use local names and object attributes.
* Mutability: custom structures often protect controlled mutation.
* Identity: the structure object can remain the same while internal state changes.

Custom data structures prepare you for:

* Stack vs heap memory discussion.
* Object lifecycle.
* Modules and packages.
* Classes and instances.
* Encapsulation.
* Dunder methods.
* Iterators and containers.

---

# Active Recall

## Easy Recall Questions

1. What is a custom data structure?
2. What is internal storage?
3. What is an external interface?
4. What is an invariant?
5. What invariant does a stack protect?
6. What invariant does a queue protect?
7. Why might a queue use `deque` internally?
8. Why should a method sometimes return a copy?
9. What does a leading underscore conventionally mean?
10. When should you not build a custom structure?

## Deep Understanding Questions

1. Why is a stack more than just a list?
2. Why is hiding implementation details useful?
3. How can outside access to internal mutable storage break invariants?
4. Why is composition a natural first approach for custom structures?
5. Why should method names reflect the concept instead of the internal storage?
6. How does choosing internal storage depend on access pattern?

## Predict-the-Output Questions

### Question 1

```python
stack = Stack()
stack.push("a")
stack.push("b")

print(stack.pop())
print(stack.pop())
```

### Question 2

```python
queue = Queue()
queue.enqueue("a")
queue.enqueue("b")

print(queue.dequeue())
print(queue.dequeue())
```

### Question 3

```python
history = History(limit=2)
history.add("a")
history.add("b")
history.add("c")

print(history.all())
```

### Question 4

```python
scores = SortedScores()
scores.add(90)
scores.add(70)
scores.add(80)

print(scores.all())
```

### Question 5

```python
values = PositiveNumbers()
values.add(10)
values.add(-5)
```

What should happen?

---

# Practical Exercises

## Exercise 1

Implement a `Stack` class with:

* `push`
* `pop`
* `peek`
* `is_empty`
* `size`

Add empty-stack error handling.

## Exercise 2

Implement a `Queue` class using `deque`.

It should support:

* `enqueue`
* `dequeue`
* `is_empty`
* `size`

## Exercise 3

Implement a `History` class with a fixed limit.

It should keep only the most recent N items.

## Exercise 4

Implement a `WordCounts` wrapper around `Counter`.

Normalize words using `.casefold()`.

## Exercise 5

Implement a `SortedScores` class using `bisect.insort`.

Expose:

* `add`
* `lowest`
* `highest`
* `all`

## Exercise 6

Implement a `PriorityQueue` using `heapq`.

Handle equal priorities safely with a counter.

## Exercise 7

Design a custom structure for validation errors.

It should support:

* adding an error
* checking whether errors exist
* returning all errors
* clearing errors

## Exercise 8

Take a program that uses a raw list or dictionary in multiple places.

Wrap it in a small custom structure with meaningful methods.

## Exercise 9

Find an unnecessary custom wrapper.

Explain why using the built-in collection directly would be clearer.

## Exercise 10

For each structure you implemented, write down:

* its invariant
* its internal storage
* its public methods
* one reason it should exist
* one reason it might not be necessary

---

# Summary

In this chapter we learned:

* A custom data structure gives a program-specific concept a clear API.
* Custom structures often wrap built-in collections.
* Internal storage is separate from external interface.
* Invariants are rules that should always remain true.
* A stack protects last-in, first-out behavior.
* A queue protects first-in, first-out behavior.
* `deque` is a good internal choice for queues.
* `Counter`, `defaultdict`, `heapq`, and `bisect` can be internal implementation tools.
* Method names should describe the concept, not the storage.
* Returning internal mutable storage can break invariants.
* Returning copies can protect internal state.
* Custom structures should earn their existence.
* Composition is a natural way to build custom structures.

Core model:

```text
custom structure
    |
    ├── public methods express meaning
    ├── internal storage does the work
    └── invariants protect correctness
```

Custom data structures are where collection knowledge becomes design.

---

# Preview of Part VIII — Memory Management

Next we move from data structures to memory management.

Part VII taught how Python organizes groups of objects:

* lists
* tuples
* dictionaries
* sets
* comprehensions
* specialized collections
* custom structures

Part VIII asks what happens underneath those structures:

* Where do objects live?
* What does stack vs heap mean?
* How does Python track references?
* When can objects be cleaned up?
* What is garbage collection?
* What is object lifecycle?
* What are weak references?

The transition is direct:

```text
data structures organize references
memory management explains object lifetime
```

Chapter 33 begins with stack vs heap.
