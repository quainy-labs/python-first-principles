# Chapter 31 — Specialized Collections: `deque`, `Counter`, `defaultdict`, `heapq`, and `bisect`

---

# Learning Objectives

By the end of this chapter, you should understand:

* Why specialized collection tools exist.
* When a list is not the right data structure.
* What `deque` is and why it is useful for queues.
* How `deque.append()`, `deque.appendleft()`, `deque.pop()`, and `deque.popleft()` work.
* What `Counter` is and why it is useful for counting.
* How `Counter` differs from a normal dictionary.
* What `defaultdict` is and why it is useful for grouping.
* How `defaultdict` differs from `.setdefault()`.
* What `heapq` is and how it supports priority queues.
* Why heaps do not keep a fully sorted list.
* What `bisect` is and how it supports binary search in sorted lists.
* How `bisect_left`, `bisect_right`, `insort_left`, and `insort_right` behave.
* How to choose between list, dictionary, set, `deque`, `Counter`, `defaultdict`, heap, and sorted-list approaches.

This chapter uses a few tools from Python's standard library.

You do not need to understand the full import system yet. That comes later in the modules part.

For now, read imports as:

```text
bring this existing tool into the current program
```

---

# Concept Overview

The previous chapters covered Python's core collection types:

* lists
* tuples
* dictionaries
* sets

These are enough for many programs.

But some access patterns deserve specialized tools.

Examples:

```text
Need efficient add/remove at both ends -> deque
Need to count many values -> Counter
Need to group values by key -> defaultdict
Need the smallest priority item repeatedly -> heapq
Need to search or insert into sorted order -> bisect
```

The central idea:

```text
data structure choice should match access pattern
```

A list can be used as a queue, but frequent `pop(0)` is inefficient.

A dictionary can count values, but `Counter` makes the intent direct.

A dictionary can group records, but `defaultdict(list)` removes repetitive setup code.

A list can be sorted repeatedly, but a heap can be better when you repeatedly need the smallest item.

A sorted list can be searched linearly, but `bisect` can find insertion positions efficiently.

---

# Why Specialized Collections Exist

No single data structure is best for every operation.

A list is good at:

* ordered storage
* indexing
* appending at the end
* popping from the end

A dictionary is good at:

* key lookup
* mapping keys to values
* records
* grouping

A set is good at:

* uniqueness
* membership
* relationship operations

Specialized tools exist because some patterns appear often enough to deserve focused support:

```text
queue behavior
counting
grouping
priority ordering
sorted insertion
```

This is an engineering principle:

Use the simplest structure that fits the access pattern.

Do not use a specialized tool because it looks advanced.

Use it when it makes the code clearer or more efficient.

---

# `deque`

`deque` stands for double-ended queue.

It is pronounced like "deck".

Import:

```python
from collections import deque
```

Basic use:

```python
from collections import deque

queue = deque()

queue.append("first")
queue.append("second")
queue.append("third")

print(queue.popleft())
print(queue.popleft())
print(queue.popleft())
```

Output:

```text
first
second
third
```

This is queue behavior:

```text
first in, first out
```

The important benefit:

`deque.popleft()` is designed for efficient removal from the left.

A list's `pop(0)` must shift remaining elements.

---

# Why Not Use A List As A Queue?

You can use a list as a queue:

```python
queue = []

queue.append("first")
queue.append("second")

item = queue.pop(0)
```

This works.

But `pop(0)` removes the first element and shifts every remaining element left.

For small lists, this may not matter.

For large or frequent queue operations, it matters.

`deque` avoids this problem:

```python
from collections import deque

queue = deque()

queue.append("first")
queue.append("second")

item = queue.popleft()
```

Mental model:

```text
list  -> efficient at the right end
deque -> efficient at both ends
```

---

# `deque` Operations

Common operations:

```python
from collections import deque

items = deque()

items.append("right")
items.appendleft("left")

print(items)
```

Output:

```text
deque(['left', 'right'])
```

Remove from right:

```python
right = items.pop()
```

Remove from left:

```python
left = items.popleft()
```

Add many values:

```python
items.extend(["a", "b"])
items.extendleft(["x", "y"])
```

Important detail:

`extendleft()` adds each item to the left one by one, so the final order may look reversed from the input.

Example:

```python
items = deque()
items.extendleft(["a", "b", "c"])

print(items)
```

Output:

```text
deque(['c', 'b', 'a'])
```

---

# `deque` As A Stack

A list is already good as a stack:

```python
stack = []
stack.append("first")
stack.append("second")

print(stack.pop())
```

Output:

```text
second
```

`deque` can also act as a stack:

```python
from collections import deque

stack = deque()
stack.append("first")
stack.append("second")

print(stack.pop())
```

Output:

```text
second
```

For stack behavior only, a list is usually fine.

Use `deque` when you need efficient operations at both ends.

---

# `deque` With `maxlen`

A `deque` can have a maximum length.

Example:

```python
from collections import deque

recent = deque(maxlen=3)

recent.append("a")
recent.append("b")
recent.append("c")
recent.append("d")

print(recent)
```

Output:

```text
deque(['b', 'c', 'd'], maxlen=3)
```

When the deque is full, adding a new item discards an item from the opposite end.

This is useful for:

* recent events
* fixed-size history
* rolling windows
* last N logs

Example:

```python
last_errors = deque(maxlen=10)
```

---

# `deque.rotate()`

`rotate()` moves items around the deque.

Example:

```python
from collections import deque

items = deque(["a", "b", "c", "d"])
items.rotate(1)

print(items)
```

Output:

```text
deque(['d', 'a', 'b', 'c'])
```

Positive rotation moves items right.

Negative rotation moves items left:

```python
items.rotate(-1)
```

`rotate()` is useful for round-robin scheduling and cyclic behavior.

Do not use it where a simple index would be clearer.

---

# `Counter`

`Counter` is a dictionary-like tool for counting hashable values.

Import:

```python
from collections import Counter
```

Example:

```python
from collections import Counter

letters = Counter("banana")

print(letters)
```

Output:

```text
Counter({'a': 3, 'n': 2, 'b': 1})
```

This replaces the manual counting pattern:

```python
counts = {}

for character in "banana":
    counts[character] = counts.get(character, 0) + 1
```

Use `Counter` when the purpose is counting.

It communicates intent directly.

---

# Accessing Counter Values

`Counter` behaves like a dictionary for many operations.

Example:

```python
from collections import Counter

counts = Counter("banana")

print(counts["a"])
print(counts["z"])
```

Output:

```text
3
0
```

Important difference from normal dictionaries:

Missing keys return `0` instead of raising `KeyError`.

This makes sense for counting:

```text
if we have never seen it, its count is zero
```

You can update counts:

```python
counts.update("bandana")
```

---

# `Counter.most_common()`

`most_common()` returns items ordered by count.

Example:

```python
from collections import Counter

words = ["python", "java", "python", "go", "python", "go"]
counts = Counter(words)

print(counts.most_common())
print(counts.most_common(2))
```

Output:

```text
[('python', 3), ('go', 2), ('java', 1)]
[('python', 3), ('go', 2)]
```

This is useful for:

* word frequencies
* event counts
* analytics
* top-N reports
* log analysis

`Counter` is not just shorter than a dictionary.

It gives counting-specific behavior.

---

# Counter Arithmetic

Counters support arithmetic-like operations.

Example:

```python
from collections import Counter

a = Counter({"apple": 3, "banana": 1})
b = Counter({"apple": 1, "orange": 2})

print(a + b)
print(a - b)
```

Output:

```text
Counter({'apple': 4, 'orange': 2, 'banana': 1})
Counter({'apple': 2, 'banana': 1})
```

Counter arithmetic keeps positive counts.

This can be useful for inventory-like logic.

But do not overuse it if explicit dictionary logic would be clearer.

---

# `defaultdict`

`defaultdict` is a dictionary that creates a default value for missing keys.

Import:

```python
from collections import defaultdict
```

Example:

```python
from collections import defaultdict

groups = defaultdict(list)

groups["admin"].append("Ada")
groups["admin"].append("Grace")
groups["user"].append("Linus")

print(groups)
```

Output resembles:

```text
defaultdict(<class 'list'>, {'admin': ['Ada', 'Grace'], 'user': ['Linus']})
```

When `groups["admin"]` is missing, `defaultdict` calls `list()` to create an empty list.

Then `.append()` runs on that list.

---

# Why `defaultdict` Exists

Without `defaultdict`, grouping often looks like:

```python
groups = {}

for user in users:
    role = user["role"]

    if role not in groups:
        groups[role] = []

    groups[role].append(user)
```

With `defaultdict`:

```python
from collections import defaultdict

groups = defaultdict(list)

for user in users:
    groups[user["role"]].append(user)
```

This removes repetitive setup code.

Use `defaultdict` when:

* missing keys should automatically get a default value
* grouping is the goal
* counting with custom defaults is needed

Use a normal dictionary when missing keys should be treated as errors.

---

# `defaultdict(int)` For Counting

`defaultdict(int)` can count values.

Example:

```python
from collections import defaultdict

counts = defaultdict(int)

for character in "banana":
    counts[character] += 1

print(counts)
```

Output resembles:

```text
defaultdict(<class 'int'>, {'b': 1, 'a': 3, 'n': 2})
```

Why does this work?

`int()` returns `0`.

When a key is missing:

```python
counts[character]
```

creates:

```text
0
```

Then `+= 1` updates it.

For pure counting, `Counter` is usually clearer.

For grouping or custom defaults, `defaultdict` is often better.

---

# `defaultdict` vs `.setdefault()`

Using `.setdefault()`:

```python
groups = {}

for user in users:
    groups.setdefault(user["role"], []).append(user)
```

Using `defaultdict`:

```python
from collections import defaultdict

groups = defaultdict(list)

for user in users:
    groups[user["role"]].append(user)
```

Both work.

`defaultdict` is usually clearer for repeated grouping.

`.setdefault()` is fine for small one-off cases.

Be careful with readability:

```python
groups.setdefault(role, []).append(user)
```

is compact, but some readers find it dense.

---

# `heapq`

`heapq` provides heap operations on lists.

A heap is useful when you repeatedly need the smallest item.

Import:

```python
import heapq
```

Example:

```python
import heapq

numbers = []

heapq.heappush(numbers, 5)
heapq.heappush(numbers, 2)
heapq.heappush(numbers, 8)
heapq.heappush(numbers, 1)

print(heapq.heappop(numbers))
print(heapq.heappop(numbers))
```

Output:

```text
1
2
```

The smallest value comes out first.

This is priority queue behavior.

---

# Heap Mental Model

A heap is not a fully sorted list.

Example:

```python
import heapq

numbers = [5, 2, 8, 1]
heapq.heapify(numbers)

print(numbers)
```

The result may look like:

```text
[1, 2, 8, 5]
```

This is not fully sorted.

The heap property is weaker:

```text
the smallest item is always available at index 0
```

That is enough for priority queue operations.

If you need all values sorted, use `sorted()`.

If you repeatedly need the next smallest value, use a heap.

---

# `heapq.heapify()`

`heapify()` transforms a list into a heap in place.

Example:

```python
import heapq

tasks = [5, 1, 3, 2]
heapq.heapify(tasks)

print(heapq.heappop(tasks))
print(heapq.heappop(tasks))
```

Output:

```text
1
2
```

`heapify()` mutates the list.

After heapifying, treat the list as a heap.

Do not casually append with:

```python
tasks.append(0)
```

Use:

```python
heapq.heappush(tasks, 0)
```

Heap operations maintain the heap property.

Normal list operations may break it.

---

# Priority Queues With Tuples

Heaps compare items.

Tuples compare lexicographically.

This makes `(priority, item)` useful.

Example:

```python
import heapq

queue = []

heapq.heappush(queue, (2, "write tests"))
heapq.heappush(queue, (1, "fix production bug"))
heapq.heappush(queue, (3, "refactor"))

while queue:
    priority, task = heapq.heappop(queue)
    print(priority, task)
```

Output:

```text
1 fix production bug
2 write tests
3 refactor
```

Lower priority numbers come out first.

If two priorities are equal, Python compares the second tuple item.

That can be a problem if the second item is not comparable.

For robust priority queues, include a tie-breaker counter.

---

# Priority Queue Tie-Breakers

Problem:

```python
heapq.heappush(queue, (1, task_object))
heapq.heappush(queue, (1, another_task_object))
```

If priorities are equal, Python tries to compare task objects.

That may fail.

Common pattern:

```python
import heapq

queue = []
counter = 0

def push(priority, task):
    global counter
    counter += 1
    heapq.heappush(queue, (priority, counter, task))
```

Now comparison uses:

```text
priority first
counter second
task never compared unless both earlier fields match
```

In real code, you can use `itertools.count()` for a tie-breaker later, after iterators are introduced.

For now, understand the tuple-ordering idea.

---

# `heapq.nsmallest()` And `nlargest()`

`heapq` provides helpers for top-N queries.

Example:

```python
import heapq

numbers = [10, 4, 7, 1, 9, 2]

print(heapq.nsmallest(3, numbers))
print(heapq.nlargest(3, numbers))
```

Output:

```text
[1, 2, 4]
[10, 9, 7]
```

These are useful when you need a small number of extreme values.

If you need all values sorted:

```python
sorted(numbers)
```

is clearer.

---

# `bisect`

`bisect` helps find insertion positions in sorted lists.

Import:

```python
import bisect
```

Example:

```python
import bisect

numbers = [10, 20, 30, 40]

position = bisect.bisect_left(numbers, 25)

print(position)
```

Output:

```text
2
```

Index `2` is where `25` should be inserted to keep the list sorted:

```text
[10, 20, 25, 30, 40]
```

`bisect` uses binary search to find the position.

---

# `bisect_left()` And `bisect_right()`

Duplicates make left and right insertion positions different.

Example:

```python
import bisect

numbers = [10, 20, 20, 20, 30]

print(bisect.bisect_left(numbers, 20))
print(bisect.bisect_right(numbers, 20))
```

Output:

```text
1
4
```

`bisect_left()` returns the position before existing equal values.

`bisect_right()` returns the position after existing equal values.

Mental model:

```text
left  -> insert before duplicates
right -> insert after duplicates
```

---

# `insort_left()` And `insort_right()`

`insort` finds the insertion point and inserts the value.

Example:

```python
import bisect

numbers = [10, 20, 30]

bisect.insort_left(numbers, 25)

print(numbers)
```

Output:

```text
[10, 20, 25, 30]
```

This mutates the list.

Important performance detail:

Binary search finds the position efficiently.

But insertion into a list may still require shifting elements.

So `bisect` helps with search position, but list insertion cost still exists.

---

# Maintaining A Sorted List

Example:

```python
import bisect

scores = []

for score in [70, 95, 80, 90]:
    bisect.insort(scores, score)

print(scores)
```

Output:

```text
[70, 80, 90, 95]
```

This is useful when values arrive over time and you want to keep the list sorted.

But for very large datasets with frequent insertions, a list may not be the best structure.

`bisect` is good when:

* the data is already sorted
* you need search positions
* insertions are not too frequent or data size is manageable

---

# `bisect` For Ranges

`bisect` can find how many values fall below a threshold.

Example:

```python
import bisect

scores = [50, 60, 70, 80, 90]

passing_index = bisect.bisect_left(scores, 70)

print(passing_index)
print(scores[passing_index:])
```

Output:

```text
2
[70, 80, 90]
```

`passing_index` is the first index where values are at least `70`.

This pattern appears in:

* thresholds
* grade bands
* time ranges
* sorted timestamps
* percentile-like calculations

---

# Choosing The Right Tool

Use a list when:

* order matters
* indexing matters
* appending at the end is enough
* the collection is a general sequence

Use `deque` when:

* you need efficient left-end operations
* you need queue behavior
* you need fixed-size recent history

Use `Counter` when:

* you are counting hashable values
* top counts matter
* missing values should behave like zero

Use `defaultdict` when:

* missing keys should create default containers
* grouping is the main task

Use `heapq` when:

* you repeatedly need the smallest item
* you need a priority queue
* full sorting every time would be wasteful

Use `bisect` when:

* a list is already sorted
* you need insertion points
* you need binary-search-style threshold logic

---

# Common Mistakes

## Misconception 1

### A list is always fine as a queue.

A list works for small queues.

For frequent left-end removals, `deque` is the right tool.

## Misconception 2

### `Counter` is just a shorter dictionary.

`Counter` has counting-specific behavior, including zero for missing keys and `most_common()`.

## Misconception 3

### `defaultdict` should replace every dictionary.

Use `defaultdict` only when automatic missing-key creation is the intended behavior.

If missing keys should be errors, use a normal dictionary.

## Misconception 4

### A heap is a sorted list.

A heap is not fully sorted.

It only guarantees efficient access to the smallest item.

## Misconception 5

### After `heapify()`, normal list operations are always safe.

Normal list operations can break the heap property.

Use heap operations to maintain the heap.

## Misconception 6

### `bisect` makes insertion into a list fully cheap.

`bisect` finds the position efficiently.

The actual insertion may still shift list elements.

## Misconception 7

### Specialized tools are always better.

Specialized tools are better only when their access pattern matches the problem.

Simple code with lists and dictionaries is often the correct choice.

---

# Real-world Usage

## Queue Processing

```python
from collections import deque

queue = deque(tasks)

while queue:
    task = queue.popleft()
    process(task)
```

## Recent History

```python
from collections import deque

recent_errors = deque(maxlen=100)
```

## Counting Events

```python
from collections import Counter

event_counts = Counter(event.type for event in events)
```

## Grouping Records

```python
from collections import defaultdict

users_by_role = defaultdict(list)

for user in users:
    users_by_role[user.role].append(user)
```

## Priority Queue

```python
import heapq

queue = []
heapq.heappush(queue, (1, "urgent"))
heapq.heappush(queue, (5, "later"))
```

## Sorted Threshold Lookup

```python
import bisect

index = bisect.bisect_left(sorted_scores, passing_score)
passing_scores = sorted_scores[index:]
```

---

# Internal Mechanics

At a high level:

```text
deque      -> optimized for both ends
Counter    -> dict subclass for counts
defaultdict -> dict subclass with missing-key factory
heapq      -> heap operations on a list
bisect     -> binary search over sorted lists
```

These tools are not random conveniences.

Each one exists because a common operation has a specific access pattern.

The key engineering question is:

```text
What operation does my program need to do repeatedly?
```

Then choose the structure that supports that operation clearly.

---

# Concept Connections

Specialized collections connect to earlier chapters:

* Lists: `heapq` and `bisect` operate on lists; lists can act as stacks.
* Dictionaries: `Counter` and `defaultdict` build on dictionary behavior.
* Sets: counting and membership depend on hashable values.
* Tuples: priority queue entries often use tuples.
* Loops: queues, grouping, counting, and heaps are processed with loops.
* Functions: dispatch and processing often pass behavior around.
* Comprehensions: `Counter` can consume generator expressions and comprehensions.

They prepare you for:

* Custom data structures.
* Algorithmic thinking.
* Queues and stacks.
* Priority-based scheduling.
* Efficient lookup and search.
* Choosing structures based on access patterns.

---

# Active Recall

## Easy Recall Questions

1. What is `deque` used for?
2. Why is `deque.popleft()` better than `list.pop(0)` for large queues?
3. What does `Counter` count?
4. What does `Counter` return for a missing key?
5. What does `defaultdict(list)` create for a missing key?
6. What is a heap useful for?
7. Is a heap fully sorted?
8. What does `bisect_left()` return?
9. What is the difference between `bisect_left()` and `bisect_right()`?
10. When should you avoid specialized tools?

## Deep Understanding Questions

1. Why does data-structure choice depend on access pattern?
2. Why is a list good as a stack but weaker as a queue?
3. Why is `Counter` clearer than a manual dictionary for counting?
4. Why can `defaultdict` hide missing-key bugs if used carelessly?
5. Why does a heap only need to expose the smallest item efficiently?
6. Why does `bisect` not remove the cost of list insertion?

## Predict-the-Output Questions

### Question 1

```python
from collections import deque

items = deque(["a", "b"])
items.appendleft("z")
print(items)
```

### Question 2

```python
from collections import Counter

counts = Counter("banana")
print(counts["a"])
print(counts["x"])
```

### Question 3

```python
from collections import defaultdict

groups = defaultdict(list)
groups["admin"].append("Ada")
print(groups["admin"])
```

### Question 4

```python
import heapq

values = [5, 1, 3]
heapq.heapify(values)
print(heapq.heappop(values))
```

### Question 5

```python
import bisect

values = [10, 20, 20, 30]
print(bisect.bisect_left(values, 20))
print(bisect.bisect_right(values, 20))
```

---

# Practical Exercises

## Exercise 1

Use `deque` to process a queue of tasks.

Add tasks with `append()` and process them with `popleft()`.

## Exercise 2

Use `deque(maxlen=5)` to store the five most recent values.

Append ten values and inspect the result.

## Exercise 3

Use `Counter` to count words in a sentence.

Print the three most common words.

## Exercise 4

Rewrite a manual counting dictionary using `Counter`.

Compare readability.

## Exercise 5

Use `defaultdict(list)` to group users by role.

Then write the same logic with a normal dictionary.

## Exercise 6

Use `heapq` to implement a priority queue of tasks.

Each task should have a numeric priority.

## Exercise 7

Create a priority queue where two tasks have the same priority.

Add a tie-breaker value so Python does not need to compare task objects.

## Exercise 8

Use `bisect_left()` and `bisect_right()` on a sorted list with duplicates.

Explain the difference.

## Exercise 9

Use `bisect.insort()` to maintain a sorted list as values arrive one by one.

## Exercise 10

For each case, choose the best tool:

* processing tasks in first-in, first-out order
* counting words
* grouping records by category
* repeatedly retrieving the smallest priority task
* keeping a sorted list of scores
* storing a fixed coordinate

Explain your choices.

---

# Summary

In this chapter we learned:

* Specialized collection tools exist for specific access patterns.
* `deque` supports efficient operations at both ends.
* `deque` is better than a list for frequent queue operations.
* `deque(maxlen=...)` is useful for fixed-size recent history.
* `Counter` counts hashable values.
* `Counter` returns zero for missing keys.
* `Counter.most_common()` supports frequency analysis.
* `defaultdict` creates default values for missing keys.
* `defaultdict(list)` is useful for grouping.
* `heapq` provides heap operations on lists.
* A heap is not a fully sorted list.
* A heap is useful when repeatedly retrieving the smallest item.
* Tuple ordering is useful for priority queue entries.
* `bisect` finds insertion positions in sorted lists.
* `bisect_left()` inserts before duplicates.
* `bisect_right()` inserts after duplicates.
* `insort()` inserts while preserving sorted order.
* Specialized tools should be chosen by access pattern, not novelty.

Core model:

```text
problem access pattern
    |
    v
choose collection/tool
    |
    ├── queue/recent history -> deque
    ├── counting -> Counter
    ├── grouping -> defaultdict
    ├── priority -> heapq
    └── sorted search/insert -> bisect
```

This chapter moves from knowing collections to choosing collections.

---

# Preview of Chapter 32

Next we study custom data structures.

So far, we have used Python's built-in and standard collection tools.

Chapter 32 asks a deeper design question:

> How do we build our own structure when the program needs a specific behavior?

We will study:

* What a custom data structure is.
* Why wrappers around built-in collections are useful.
* How to design small stack and queue abstractions.
* How to expose clear methods.
* How to protect invariants.
* How to choose internal storage.
* When not to build a custom structure.

The transition is direct:

```text
built-in collections -> specialized tools -> custom structures
```

Custom data structures are where collection choice becomes API design.
