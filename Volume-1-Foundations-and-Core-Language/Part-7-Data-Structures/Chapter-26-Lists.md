# Chapter 26 — Lists

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a list is.
* Why lists exist.
* How lists store references to objects.
* Why lists are mutable.
* How indexing works.
* How negative indexing works.
* How slicing works.
* How list assignment and mutation work.
* How common list methods behave.
* Why some methods return `None`.
* How aliasing affects lists.
* How shallow copies work.
* How nested lists behave.
* How iteration over lists works.
* How list growth works conceptually.
* Basic complexity intuition for common list operations.
* Common mistakes with lists.

Lists are Python's most common mutable sequence.

They are where the object-reference model becomes practical every day.

---

# Concept Overview

A list is an ordered collection of references to objects.

Example:

```python
numbers = [10, 20, 30]
```

Conceptually:

```text
numbers ─────▶ list object
               ├── index 0 ─▶ int object 10
               ├── index 1 ─▶ int object 20
               └── index 2 ─▶ int object 30
```

The list contains references.

It does not contain the integer objects inside the list object like physical boxes.

This distinction matters because lists can hold any objects:

```python
items = [1, "hello", True, None]
```

The list stores references to:

* an `int`
* a `str`
* a `bool`
* `None`

Lists preserve order.

Lists are mutable.

Lists can grow and shrink.

---

# Why Lists Exist

Programs often need to work with many values.

Examples:

```text
all users in a system
all lines in a file
all prices in a cart
all errors found during validation
all tasks waiting to run
all results returned by an API
```

Without lists, you would need many separate names:

```python
score_1 = 90
score_2 = 85
score_3 = 72
```

This does not scale.

A list lets one name refer to a collection:

```python
scores = [90, 85, 72]
```

Now the program can:

* Iterate over values.
* Add values.
* Remove values.
* Sort values.
* Slice values.
* Pass the whole collection to a function.

Lists solve the problem of ordered, changeable collections.

---

# List Literals

A list literal uses square brackets.

Example:

```python
empty = []
numbers = [1, 2, 3]
names = ["Ada", "Linus", "Grace"]
mixed = [1, "two", 3.0]
```

Lists can contain expressions:

```python
x = 10
y = 20

values = [x, y, x + y]
print(values)
```

Output:

```text
[10, 20, 30]
```

The expressions are evaluated first.

Then the list stores references to the resulting objects.

---

# Lists Store References

This is the most important list idea.

Example:

```python
a = ["python"]
b = [a]
```

Conceptually:

```text
a ─────▶ list object ["python"]

b ─────▶ list object
          └── index 0 ─▶ same list object as a
```

The inner list is not copied.

The outer list stores a reference to it.

Mutation shows this:

```python
a = ["python"]
b = [a]

a.append("java")

print(b)
```

Output:

```text
[['python', 'java']]
```

`b` sees the change because `b[0]` refers to the same list object as `a`.

---

# Lists Are Mutable

A mutable object can be changed without creating a new object.

Example:

```python
numbers = [1, 2, 3]
print(id(numbers))

numbers.append(4)
print(numbers)
print(id(numbers))
```

Output resembles:

```text
4380000000
[1, 2, 3, 4]
4380000000
```

The exact `id` is not important.

The important point:

```text
same list object
changed contents
```

Mutation preserves identity.

This connects directly to the mutability chapter.

---

# Creating Lists

Common ways to create lists:

```python
items = []
numbers = [1, 2, 3]
letters = list("abc")
values = list(range(5))
```

Output:

```python
print(letters)
print(values)
```

```text
['a', 'b', 'c']
[0, 1, 2, 3, 4]
```

`list()` consumes an iterable and creates a list from its items.

Examples:

```python
list("Python")
list((1, 2, 3))
list(range(3))
```

Later chapters will explain iterables more deeply.

For now:

```text
list(iterable) -> list containing the iterable's values
```

---

# Indexing

Lists are ordered.

Each item has an index.

Indexes start at `0`.

Example:

```python
names = ["Ada", "Linus", "Grace"]

print(names[0])
print(names[1])
print(names[2])
```

Output:

```text
Ada
Linus
Grace
```

Index model:

```text
index:   0       1        2
value: "Ada"  "Linus"  "Grace"
```

Why zero?

At a low level, sequence indexing is often based on offset from the beginning.

Index `0` means:

```text
the item zero steps from the start
```

---

# IndexError

An index must exist.

Example:

```python
names = ["Ada", "Linus", "Grace"]
print(names[3])
```

Python raises:

```text
IndexError
```

The valid indexes are:

```text
0, 1, 2
```

Index `3` is one past the last item.

Common mistake:

```python
for index in range(len(names) + 1):
    print(names[index])
```

This eventually tries to access an invalid index.

Correct:

```python
for index in range(len(names)):
    print(names[index])
```

Usually better:

```python
for name in names:
    print(name)
```

---

# Negative Indexing

Negative indexes count from the end.

Example:

```python
names = ["Ada", "Linus", "Grace"]

print(names[-1])
print(names[-2])
print(names[-3])
```

Output:

```text
Grace
Linus
Ada
```

Model:

```text
positive index:   0       1        2
value:          "Ada"  "Linus"  "Grace"
negative index:  -3      -2       -1
```

`-1` means the last item.

`-2` means the item before the last.

Negative indexing is useful for:

* Last item.
* Recently appended item.
* End-relative access.

---

# Index Assignment

Because lists are mutable, you can replace an item at an index.

Example:

```python
names = ["Ada", "Linus", "Grace"]
names[1] = "Guido"

print(names)
```

Output:

```text
['Ada', 'Guido', 'Grace']
```

The list object is the same.

One reference inside the list changed.

Conceptually:

```text
before:
index 1 ─▶ "Linus"

after:
index 1 ─▶ "Guido"
```

The old string object is not modified.

The list slot is rebound to refer to a different object.

---

# Slicing

Slicing creates a new list containing selected references.

Syntax:

```python
sequence[start:stop]
```

Example:

```python
numbers = [10, 20, 30, 40, 50]

print(numbers[1:4])
```

Output:

```text
[20, 30, 40]
```

The start index is included.

The stop index is excluded.

Model:

```text
numbers[1:4]
        includes indexes 1, 2, 3
        excludes index 4
```

This is the same half-open idea used by `range()`.

---

# Slice Defaults

If `start` is omitted, slicing starts at the beginning.

```python
numbers = [10, 20, 30, 40, 50]

print(numbers[:3])
```

Output:

```text
[10, 20, 30]
```

If `stop` is omitted, slicing continues to the end.

```python
print(numbers[2:])
```

Output:

```text
[30, 40, 50]
```

If both are omitted:

```python
copy = numbers[:]
```

This creates a shallow copy of the list.

---

# Slice Step

Slicing can include a step:

```python
sequence[start:stop:step]
```

Example:

```python
numbers = [0, 1, 2, 3, 4, 5]

print(numbers[::2])
```

Output:

```text
[0, 2, 4]
```

Reverse copy:

```python
print(numbers[::-1])
```

Output:

```text
[5, 4, 3, 2, 1, 0]
```

This creates a new list.

It does not reverse the original list in place.

---

# Slicing Creates A Shallow Copy

Example:

```python
original = [[1], [2], [3]]
copy = original[:]

copy.append([4])
copy[0].append("changed")

print(original)
print(copy)
```

Output:

```text
[[1, 'changed'], [2], [3]]
[[1, 'changed'], [2], [3], [4]]
```

Why?

The outer list was copied.

The inner lists were not copied.

Conceptually:

```text
original ─▶ outer list A
             ├── inner list 1
             ├── inner list 2
             └── inner list 3

copy ─────▶ outer list B
             ├── same inner list 1
             ├── same inner list 2
             └── same inner list 3
```

Appending to `copy` changes only outer list B.

Mutating `copy[0]` mutates a shared inner list.

---

# `append()`

`append()` adds one object to the end of a list.

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

`append()` mutates the list.

It returns `None`.

Example:

```python
items = ["a", "b"]
result = items.append("c")

print(items)
print(result)
```

Output:

```text
['a', 'b', 'c']
None
```

Returning `None` is intentional.

It signals:

```text
this method mutates the object in place
```

---

# `extend()`

`extend()` adds each item from an iterable to the list.

Example:

```python
items = ["a", "b"]
items.extend(["c", "d"])

print(items)
```

Output:

```text
['a', 'b', 'c', 'd']
```

Difference between `append()` and `extend()`:

```python
items = ["a", "b"]
items.append(["c", "d"])
print(items)
```

Output:

```text
['a', 'b', ['c', 'd']]
```

```python
items = ["a", "b"]
items.extend(["c", "d"])
print(items)
```

Output:

```text
['a', 'b', 'c', 'd']
```

`append()` adds one object.

`extend()` adds many items from an iterable.

---

# `insert()`

`insert()` adds an item at a specific index.

Example:

```python
items = ["a", "c"]
items.insert(1, "b")

print(items)
```

Output:

```text
['a', 'b', 'c']
```

Items at and after the insertion point shift right.

This matters for performance.

Inserting near the front of a large list requires moving many references.

Use `append()` when adding to the end.

Use `collections.deque` later when frequent front insertions are needed.

---

# `pop()`

`pop()` removes and returns an item.

By default, it removes the last item.

Example:

```python
items = ["a", "b", "c"]
last = items.pop()

print(last)
print(items)
```

Output:

```text
c
['a', 'b']
```

You can pop by index:

```python
items = ["a", "b", "c"]
middle = items.pop(1)

print(middle)
print(items)
```

Output:

```text
b
['a', 'c']
```

Popping from the end is efficient.

Popping from the front or middle requires shifting later references left.

---

# `remove()`

`remove()` removes the first matching value.

Example:

```python
items = ["a", "b", "a"]
items.remove("a")

print(items)
```

Output:

```text
['b', 'a']
```

Only the first `"a"` is removed.

If the value is missing:

```python
items.remove("x")
```

Python raises:

```text
ValueError
```

Safe pattern:

```python
if "x" in items:
    items.remove("x")
```

But remember: membership search in a list scans values from left to right.

---

# `clear()`

`clear()` removes all items from a list in place.

Example:

```python
items = ["a", "b", "c"]
same_list = items

items.clear()

print(items)
print(same_list)
```

Output:

```text
[]
[]
```

Both names refer to the same list object.

Clearing through one name affects what the other name sees.

This is mutation.

---

# `del`

`del` can remove an item by index.

Example:

```python
items = ["a", "b", "c"]
del items[1]

print(items)
```

Output:

```text
['a', 'c']
```

`del` can also remove a slice:

```python
items = ["a", "b", "c", "d"]
del items[1:3]

print(items)
```

Output:

```text
['a', 'd']
```

`del` mutates the list.

It is a statement, not a method.

---

# `sort()`

`sort()` sorts a list in place.

Example:

```python
numbers = [3, 1, 2]
result = numbers.sort()

print(numbers)
print(result)
```

Output:

```text
[1, 2, 3]
None
```

`sort()` mutates and returns `None`.

If you need a new sorted list, use `sorted()`:

```python
numbers = [3, 1, 2]
sorted_numbers = sorted(numbers)

print(numbers)
print(sorted_numbers)
```

Output:

```text
[3, 1, 2]
[1, 2, 3]
```

This distinction is important:

```text
list.sort() -> mutate existing list
sorted(list) -> create new sorted list
```

---

# `reverse()`

`reverse()` reverses a list in place.

Example:

```python
items = ["a", "b", "c"]
result = items.reverse()

print(items)
print(result)
```

Output:

```text
['c', 'b', 'a']
None
```

If you need a reversed copy:

```python
items = ["a", "b", "c"]
copy = items[::-1]
```

If you need to iterate in reverse without creating a list:

```python
for item in reversed(items):
    print(item)
```

Choose based on intent.

---

# `index()` And `count()`

`index()` returns the index of the first matching value.

Example:

```python
items = ["a", "b", "a"]

print(items.index("a"))
```

Output:

```text
0
```

If the value is missing, `index()` raises `ValueError`.

`count()` counts matching values:

```python
print(items.count("a"))
```

Output:

```text
2
```

Both methods scan the list.

For large lists and frequent lookup, other data structures may be better.

Sets and dictionaries are covered later.

---

# List Concatenation

The `+` operator creates a new list.

Example:

```python
a = [1, 2]
b = [3, 4]
c = a + b

print(c)
print(a)
print(b)
```

Output:

```text
[1, 2, 3, 4]
[1, 2]
[3, 4]
```

The original lists are unchanged.

This is different from `extend()`:

```python
a = [1, 2]
a.extend([3, 4])

print(a)
```

Output:

```text
[1, 2, 3, 4]
```

`+` creates a new list.

`extend()` mutates an existing list.

---

# List Repetition

The `*` operator repeats list references.

Example:

```python
values = [0] * 3
print(values)
```

Output:

```text
[0, 0, 0]
```

This is fine for immutable objects.

But be careful with nested mutable objects:

```python
matrix = [[]] * 3
matrix[0].append("x")

print(matrix)
```

Output:

```text
[['x'], ['x'], ['x']]
```

Why?

All three slots refer to the same inner list.

Correct:

```python
matrix = [[] for _ in range(3)]
matrix[0].append("x")

print(matrix)
```

Output:

```text
[['x'], [], []]
```

Each iteration creates a new inner list.

---

# Membership

Use `in` to test whether a value appears in a list.

Example:

```python
names = ["Ada", "Linus", "Grace"]

print("Ada" in names)
print("Guido" in names)
```

Output:

```text
True
False
```

List membership uses equality checks.

Python scans from left to right until it finds a match or reaches the end.

For large lists, repeated membership checks can be slow.

If you need frequent membership checks and order does not matter, a set may be better.

---

# Iterating Over Lists

Lists are iterable.

Example:

```python
names = ["Ada", "Linus", "Grace"]

for name in names:
    print(name)
```

Output:

```text
Ada
Linus
Grace
```

This is the normal way to process list values.

Use direct iteration when you need values.

Use `enumerate()` when you need index and value:

```python
for index, name in enumerate(names):
    print(index, name)
```

Use `range(len(...))` only when you genuinely need index arithmetic.

---

# Mutating While Iterating

Be careful when mutating a list while iterating over it.

Bug:

```python
numbers = [1, 2, 3, 4, 5, 6]

for number in numbers:
    if number % 2 == 0:
        numbers.remove(number)

print(numbers)
```

This may appear to work in some cases and fail in others.

The problem is that removing an item shifts later items left while the loop is moving forward.

Safer:

```python
numbers = [1, 2, 3, 4, 5, 6]
odds = []

for number in numbers:
    if number % 2 != 0:
        odds.append(number)

print(odds)
```

Often best:

```python
odds = [number for number in numbers if number % 2 != 0]
```

Comprehensions are studied later in this part.

---

# Aliasing

Aliasing happens when two names refer to the same object.

Example:

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)
print(b)
```

Output:

```text
[1, 2, 3, 4]
[1, 2, 3, 4]
```

There are not two lists.

There is one list and two names:

```text
a ─────┐
       v
b ─▶ list object [1, 2, 3, 4]
```

This follows the same names-and-references model from earlier chapters.

---

# Copying Lists

To create a shallow copy:

```python
a = [1, 2, 3]
b = a.copy()

b.append(4)

print(a)
print(b)
```

Output:

```text
[1, 2, 3]
[1, 2, 3, 4]
```

Other shallow-copy options:

```python
b = a[:]
b = list(a)
```

All create a new outer list.

For lists containing immutable objects, that is usually enough.

For nested mutable objects, shallow copies still share inner objects.

---

# Nested Lists

A nested list is a list containing lists.

Example:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
]
```

Access:

```python
print(matrix[0])
print(matrix[0][1])
```

Output:

```text
[1, 2, 3]
2
```

Model:

```text
matrix[0]    -> first row list
matrix[0][1] -> second item in first row
```

Nested lists are common for:

* Tables.
* Grids.
* Matrices.
* Nested data.

But for serious numeric matrix work, libraries such as NumPy are usually better.

---

# Nested List Aliasing Mistake

Common bug:

```python
rows = [[0] * 3] * 3
rows[0][0] = 1

print(rows)
```

Output:

```text
[[1, 0, 0], [1, 0, 0], [1, 0, 0]]
```

All rows refer to the same inner list.

Correct:

```python
rows = [[0] * 3 for _ in range(3)]
rows[0][0] = 1

print(rows)
```

Output:

```text
[[1, 0, 0], [0, 0, 0], [0, 0, 0]]
```

The comprehension creates a new inner list each time.

---

# List Unpacking

Lists can be unpacked into names.

Example:

```python
point = [10, 20]
x, y = point

print(x)
print(y)
```

Output:

```text
10
20
```

The number of names must match the number of values:

```python
x, y = [10, 20, 30]
```

raises:

```text
ValueError
```

Extended unpacking:

```python
first, *middle, last = [1, 2, 3, 4, 5]

print(first)
print(middle)
print(last)
```

Output:

```text
1
[2, 3, 4]
5
```

Unpacking is useful when list structure is known.

---

# Lists As Stacks

A stack is a last-in, first-out structure.

Lists can act as stacks using `append()` and `pop()`.

Example:

```python
stack = []

stack.append("first")
stack.append("second")
stack.append("third")

print(stack.pop())
print(stack.pop())
print(stack.pop())
```

Output:

```text
third
second
first
```

This is efficient when pushing and popping from the end.

Avoid using the front of a list as a queue for large workloads.

For queues, `collections.deque` is usually better.

---

# Lists As Queues

You can write:

```python
queue = []

queue.append("first")
queue.append("second")

print(queue.pop(0))
```

Output:

```text
first
```

But `pop(0)` shifts every remaining item left.

That is inefficient for large lists.

Use `collections.deque` when you need efficient queue behavior:

```python
from collections import deque

queue = deque()
queue.append("first")
queue.append("second")

print(queue.popleft())
```

`deque` is covered later in this part.

---

# List Growth

Lists can grow.

Example:

```python
items = []

for number in range(5):
    items.append(number)
```

Python lists are dynamic arrays internally.

At a conceptual level:

```text
list stores references in a contiguous array-like area
append adds a reference at the end
if there is no room, Python allocates more space
```

Python over-allocates list capacity so repeated `append()` calls are usually efficient.

You do not need to manage capacity manually.

But the model explains why appending at the end is usually efficient and inserting at the front is not.

---

# Complexity Intuition

Common list operation intuition:

| Operation | Intuition |
|---|---|
| `items[index]` | Fast direct access |
| `items.append(value)` | Usually fast |
| `items.pop()` | Fast from end |
| `value in items` | Scans from left to right |
| `items.index(value)` | Scans from left to right |
| `items.insert(0, value)` | Shifts many references |
| `items.pop(0)` | Shifts many references |
| slicing | Creates a new list |
| `items.sort()` | Sorts in place |

This is not a full algorithms chapter.

But you should begin connecting data structures to cost.

The way data is stored affects how operations behave.

---

# Lists And Function Arguments

Passing a list to a function passes a reference to the list object.

Example:

```python
def add_item(items):
    items.append("new")

values = ["old"]
add_item(values)

print(values)
```

Output:

```text
['old', 'new']
```

The function did not receive a copy.

It received a reference to the same list object.

If the function mutates the list, the caller sees the mutation.

If the function rebinds its local parameter, the caller's name is unaffected:

```python
def replace_items(items):
    items = ["new"]

values = ["old"]
replace_items(values)

print(values)
```

Output:

```text
['old']
```

This is the parameter-binding model from the functions chapter applied to lists.

---

# Designing Functions With Lists

Be clear about whether a function mutates a list or returns a new list.

Mutating:

```python
def remove_empty_in_place(values):
    values[:] = [value for value in values if value]
```

Returning new list:

```python
def without_empty(values):
    return [value for value in values if value]
```

The names should communicate behavior.

Good names:

```python
sort_in_place(values)
append_error(errors, message)
without_empty(values)
normalized_names(names)
```

Avoid hidden mutation in functions that sound like they return a new value.

---

# Common Mistakes

## Misconception 1

### A list stores objects directly inside itself.

A list stores references to objects.

This is why aliasing and shallow-copy behavior matter.

## Misconception 2

### `b = a` copies a list.

It does not.

It creates another name for the same list object.

Use `a.copy()`, `a[:]`, or `list(a)` for a shallow copy.

## Misconception 3

### Slicing deeply copies nested lists.

Slicing creates a shallow copy.

Nested mutable objects are still shared.

## Misconception 4

### Methods like `append()` and `sort()` return the changed list.

They return `None`.

They mutate the list in place.

## Misconception 5

### `append()` and `extend()` do the same thing.

`append()` adds one object.

`extend()` adds items from an iterable.

## Misconception 6

### `[[0] * 3] * 3` creates three independent rows.

It creates three references to the same inner list.

Use a comprehension to create independent rows.

## Misconception 7

### Removing from a list while iterating is always safe.

It can skip values because indexes shift.

Build a new list or iterate over a copy.

## Misconception 8

### Lists are always the best collection.

Lists are excellent ordered mutable sequences.

Other structures are better for frequent membership checks, queue operations, key-value lookup, or uniqueness.

---

# Real-world Usage

## Collecting Results

```python
errors = []

for record in records:
    if not record.is_valid:
        errors.append(record.error)
```

## Ordered Input

```python
steps = ["download", "validate", "transform", "save"]

for step in steps:
    run_step(step)
```

## Batch Processing

```python
batch = []

for item in stream:
    batch.append(item)

    if len(batch) == 100:
        process(batch)
        batch.clear()
```

## Stack Behavior

```python
stack = [root]

while stack:
    node = stack.pop()
    stack.extend(node.children)
```

## Sorting

```python
users.sort(key=lambda user: user.created_at)
```

## Transformation

```python
names = [user.name for user in users]
```

Comprehensions are covered in detail later in this part.

---

# Internal Mechanics

At a high level, a Python list is a dynamic array of references.

Conceptually:

```text
list object
    |
    v
array-like storage of object references
```

When you access by index:

```python
items[2]
```

Python can jump directly to the reference at index `2`.

When you append:

```python
items.append(value)
```

Python places a new reference at the end if there is capacity.

When capacity is exhausted, Python allocates larger storage and moves references.

When you insert at the front:

```python
items.insert(0, value)
```

Python must shift existing references to make room.

This is why list operations have different performance characteristics.

The full CPython implementation details come later.

For now, keep the core model:

```text
list = ordered mutable sequence of references
```

---

# Concept Connections

Lists connect to earlier chapters:

* Objects: a list is an object.
* Names and references: names refer to list objects.
* Mutability: lists can change while preserving identity.
* Identity and equality: two lists can be equal but not identical.
* Operators: `+`, `*`, `in`, and comparisons interact with lists.
* Expressions: list literals and indexing are expressions.
* Loops: lists are commonly iterated.
* Functions: passing a list passes a reference to the list object.
* Recursion: nested lists can be processed recursively.
* Functional programming: transformations often create new lists.

Lists prepare you for:

* Tuples.
* Dictionaries.
* Sets.
* Comprehensions.
* Stacks.
* Queues.
* Custom data structures.
* Memory management.

---

# Active Recall

## Easy Recall Questions

1. What is a list?
2. Are lists mutable or immutable?
3. What does `items[0]` access?
4. What does `items[-1]` access?
5. What does slicing create?
6. What does `append()` return?
7. What is the difference between `append()` and `extend()`?
8. What is aliasing?
9. What is a shallow copy?
10. Why is `pop(0)` inefficient for large lists?

## Deep Understanding Questions

1. Why does `b = a` not copy a list?
2. Why can a shallow copy still share nested mutable objects?
3. Why do mutating list methods usually return `None`?
4. Why is appending at the end usually efficient?
5. Why is inserting at the front usually expensive?
6. Why does `[[0] * 3] * 3` create shared rows?
7. How does passing a list to a function connect to names and references?

## Predict-the-Output Questions

### Question 1

```python
a = [1, 2]
b = a
b.append(3)

print(a)
print(b)
```

### Question 2

```python
a = [1, 2]
b = a.copy()
b.append(3)

print(a)
print(b)
```

### Question 3

```python
items = ["a", "b"]
result = items.append("c")

print(items)
print(result)
```

### Question 4

```python
matrix = [[]] * 2
matrix[0].append("x")

print(matrix)
```

### Question 5

```python
values = [10, 20, 30, 40]
print(values[1:3])
print(values[::-1])
```

### Question 6

```python
def change(items):
    items.append("new")

values = ["old"]
change(values)
print(values)
```

### Question 7

```python
def replace(items):
    items = ["new"]

values = ["old"]
replace(values)
print(values)
```

---

# Practical Exercises

## Exercise 1

Create a list of five numbers.

Print:

* The first item.
* The last item.
* The middle three items.
* The reversed list.

## Exercise 2

Write a function that takes a list of numbers and returns a new list containing only positive numbers.

Do not mutate the input list.

## Exercise 3

Write a function that removes empty strings from a list in place.

Then write a second version that returns a new list.

Explain the difference.

## Exercise 4

Create a list aliasing example with two names referring to the same list.

Draw the memory diagram.

## Exercise 5

Create a shallow-copy example using nested lists.

Show which mutations affect both lists and which mutations affect only one outer list.

## Exercise 6

Create a 3 by 3 grid correctly using a comprehension.

Then intentionally create the broken version with list repetition.

Explain why the broken version shares rows.

## Exercise 7

Use a list as a stack.

Push five values and pop them all.

Explain the order.

## Exercise 8

Write code that uses `append()`, `extend()`, and `insert()`.

Show the difference between them.

## Exercise 9

Write a function that receives a list and sorts it in place.

Write another function that receives a list and returns a sorted copy.

Name both functions clearly.

## Exercise 10

Given a list of records, collect all invalid records into an `errors` list.

Use a loop first.

Then write a comprehension version.

---

# Summary

In this chapter we learned:

* Lists are ordered mutable sequences.
* Lists store references to objects.
* List literals create list objects.
* Indexing retrieves one item by position.
* Negative indexes count from the end.
* Slicing creates a new list.
* Slices are shallow copies.
* List mutation preserves list identity.
* `append()` adds one object.
* `extend()` adds many items from an iterable.
* `insert()` shifts items to make room.
* `pop()` removes and returns an item.
* `remove()` removes the first matching value.
* `sort()` and `reverse()` mutate in place and return `None`.
* Aliasing means multiple names refer to the same list.
* Shallow copies share nested mutable objects.
* Nested list repetition can accidentally share inner lists.
* Lists can act as stacks efficiently at the end.
* Lists are less suitable for frequent front removals.
* Passing a list to a function passes a reference to the list object.

Core model:

```text
name ─────▶ list object
             ├── index 0 ─▶ object
             ├── index 1 ─▶ object
             └── index 2 ─▶ object
```

Lists are where Python's names, references, mutability, loops, and functions meet in everyday code.

---

# Preview of Chapter 27

Next we study tuples.

Lists are mutable sequences.

Tuples are immutable sequences.

We will study:

* What tuples are.
* Why tuples exist.
* Tuple literals.
* Tuple packing and unpacking.
* Immutability.
* How tuples can contain mutable objects.
* Returning multiple values.
* Tuples as lightweight records.
* When to choose a tuple instead of a list.

The transition is direct:

```text
list -> ordered mutable sequence
tuple -> ordered immutable sequence
```

Understanding lists first makes tuples much easier to understand because the sequence model is shared, but mutability changes the design tradeoffs.
