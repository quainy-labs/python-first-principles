# Chapter 19 — Loops

---

# Learning Objectives

By the end of this chapter, you should understand:

* What loops are and why programs need repetition.
* The difference between `while` loops and `for` loops.
* How loop variables work as normal Python names.
* How `range()` works.
* How to count forward and backward.
* How to use `break`, `continue`, and loop `else`.
* How nested loops work.
* How to reason about nested loop execution order.
* How to iterate over strings, lists, tuples, dictionaries, and sets.
* How to use `enumerate()` and `zip()`.
* How to avoid infinite loops.
* How to avoid mutation bugs while iterating.
* How common looping patterns appear in real programs.

Conditionals decide whether code runs.

Loops decide how many times code runs.

---

# Concept Overview

A loop repeats a block of code.

Without a loop:

```python
print("hello")
print("hello")
print("hello")
```

With a loop:

```python
for _ in range(3):
    print("hello")
```

Output:

```text
hello
hello
hello
```

The loop version expresses intent:

```text
repeat this block three times
```

Python has two main loop statements:

* `while`
* `for`

Use `while` when repetition depends on a condition.

Use `for` when repetition processes items from an iterable.

---

# Why Loops Exist

Programs often need repeated work.

Examples:

```text
Process every file in a directory.
Validate every record in a dataset.
Retry an operation until it succeeds.
Read lines from a file.
Search for a matching item.
Build a report row by row.
Poll a queue until it is empty.
Ask the user again until input is valid.
```

Without loops, repeated logic must be copied.

Copied logic is:

* Longer.
* Harder to update.
* More error-prone.
* Less expressive.

Loops let one block represent repeated behavior.

---

# The `while` Loop

A `while` loop repeats while a condition is truthy.

Syntax:

```python
while condition:
    block
```

Example:

```python
count = 0

while count < 3:
    print(count)
    count += 1
```

Output:

```text
0
1
2
```

Execution model:

```text
check condition
if truthy:
    run block
    go back to condition
if falsy:
    exit loop
```

The condition is checked before every iteration.

If the condition is falsy the first time, the loop body never runs.

Example:

```python
count = 5

while count < 3:
    print(count)
```

This prints nothing.

---

# Updating Loop State

A `while` loop usually needs some state to change.

Example:

```python
count = 0

while count < 3:
    print(count)
    count += 1
```

The variable `count` changes each iteration.

If it did not change, the loop would never finish.

Bug:

```python
count = 0

while count < 3:
    print(count)
```

This is an infinite loop.

The condition stays true forever.

Rule:

For every `while` loop, identify:

* Initial state.
* Continuation condition.
* State update.
* Exit condition.

---

# Avoiding Infinite Loops

An infinite loop happens when the loop condition never becomes falsy.

Example:

```python
number = 10

while number > 0:
    print(number)
    number += 1
```

This moves away from the exit condition.

Correct:

```python
number = 10

while number > 0:
    print(number)
    number -= 1
```

Output:

```text
10
9
8
7
6
5
4
3
2
1
```

Infinite loops are not always mistakes.

Servers, event loops, and command loops may run until interrupted.

Example:

```python
while True:
    command = input("> ")

    if command == "quit":
        break

    print(f"running {command}")
```

This is intentional because `break` provides the exit.

---

# The `for` Loop

A `for` loop iterates over an iterable.

Syntax:

```python
for item in iterable:
    block
```

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

The mental model is:

```text
ask iterable for next item
assign item to loop variable
run block
repeat until no items remain
```

A `for` loop does not require you to manually update an index.

That is one reason Python loops are usually more readable than loops in languages that rely heavily on counters.

---

# Loop Variables Are Names

The loop variable is a normal Python name.

Example:

```python
for item in [10, 20, 30]:
    print(item)
```

During each iteration:

```text
item -> 10
item -> 20
item -> 30
```

After the loop, the name still exists:

```python
for item in [10, 20, 30]:
    pass

print(item)
```

Output:

```text
30
```

The name `item` refers to the last assigned value.

This is not usually a problem, but it matters when reading code.

Loop variables do not create a special private scope.

Function scope is covered in the next part.

---

# Iterating Over Strings

Strings are iterable.

Example:

```python
for character in "Python":
    print(character)
```

Output:

```text
P
y
t
h
o
n
```

Each iteration receives one character string.

This is useful for:

* Validation.
* Counting characters.
* Parsing simple formats.
* Searching text.

Example:

```python
digits = 0

for character in "room 101":
    if character.isdigit():
        digits += 1

print(digits)
```

Output:

```text
3
```

---

# Iterating Over Lists and Tuples

Lists and tuples are iterable.

Example:

```python
scores = [80, 95, 72]

for score in scores:
    print(score)
```

Output:

```text
80
95
72
```

Usually, iterate over values directly.

Prefer:

```python
for score in scores:
    print(score)
```

Over:

```python
for index in range(len(scores)):
    print(scores[index])
```

Direct iteration is clearer when you only need values.

Use indexes only when indexes matter.

---

# Iterating Over Dictionaries

Dictionaries are iterable.

By default, iterating over a dictionary gives keys.

Example:

```python
settings = {"theme": "dark", "debug": False}

for key in settings:
    print(key)
```

Output:

```text
theme
debug
```

To iterate over values:

```python
for value in settings.values():
    print(value)
```

To iterate over key-value pairs:

```python
for key, value in settings.items():
    print(key, value)
```

Output:

```text
theme dark
debug False
```

Dictionary iteration is common in configuration, JSON-like data, counters, and mappings.

---

# Iterating Over Sets

Sets are iterable.

Example:

```python
tags = {"python", "backend", "api"}

for tag in tags:
    print(tag)
```

Sets do not preserve a meaningful sorted order.

Do not write logic that depends on set iteration order.

If order matters, sort first:

```python
for tag in sorted(tags):
    print(tag)
```

---

# `range()`

`range()` represents a sequence of integers.

Example:

```python
for number in range(5):
    print(number)
```

Output:

```text
0
1
2
3
4
```

`range(5)` means:

```text
start at 0
stop before 5
```

The stop value is excluded.

This is called a half-open range.

---

# `range(start, stop)`

Example:

```python
for number in range(2, 6):
    print(number)
```

Output:

```text
2
3
4
5
```

The start value is included.

The stop value is excluded.

---

# `range(start, stop, step)`

The third argument controls the step.

Example:

```python
for number in range(0, 10, 2):
    print(number)
```

Output:

```text
0
2
4
6
8
```

The step says how much to add each time.

Step can be negative.

That enables reverse counting.

---

# Reverse Looping With `range()`

Use a negative step to count backward.

Example:

```python
for number in range(5, 0, -1):
    print(number)
```

Output:

```text
5
4
3
2
1
```

The stop value is still excluded.

This means:

```python
range(5, 0, -1)
```

includes `5`, but excludes `0`.

To include `0`:

```python
for number in range(5, -1, -1):
    print(number)
```

Output:

```text
5
4
3
2
1
0
```

Reverse range mistakes usually happen because the stop value is forgotten.

---

# Reverse Looping Over a Sequence

Use `reversed()` to loop over a sequence backward.

Example:

```python
names = ["Ada", "Linus", "Grace"]

for name in reversed(names):
    print(name)
```

Output:

```text
Grace
Linus
Ada
```

This is clearer than manually managing indexes.

If you need reverse indexes:

```python
items = ["a", "b", "c"]

for index in range(len(items) - 1, -1, -1):
    print(index, items[index])
```

Output:

```text
2 c
1 b
0 a
```

Use direct reverse iteration when possible.

Use reverse indexes when the index itself matters.

---

# `range()` Is Not a List

In Python 3, `range()` creates a range object.

Example:

```python
numbers = range(5)

print(numbers)
```

Output:

```text
range(0, 5)
```

It does not build a full list immediately.

To see the values:

```python
print(list(range(5)))
```

Output:

```text
[0, 1, 2, 3, 4]
```

This is memory-efficient.

`range(1_000_000)` does not store one million integer objects in a list.

It stores enough information to produce the sequence.

---

# `break`

`break` exits the nearest enclosing loop immediately.

Example:

```python
numbers = [3, 5, 8, 9]

for number in numbers:
    if number % 2 == 0:
        print("found even number")
        break
```

Output:

```text
found even number
```

The loop stops when `8` is found.

`9` is never processed.

Use `break` when the loop's job is complete.

Common uses:

* Search.
* Retry until success.
* Read until sentinel value.
* Exit command loop.

---

# `continue`

`continue` skips the rest of the current iteration and moves to the next one.

Example:

```python
numbers = [1, 2, 3, 4, 5]

for number in numbers:
    if number % 2 == 0:
        continue

    print(number)
```

Output:

```text
1
3
5
```

Even numbers are skipped.

Use `continue` when one item should be ignored but the loop should keep going.

---

# `break` vs `continue`

`break` stops the loop.

`continue` skips one iteration.

Example:

```python
for number in [1, 2, 3, 4, 5]:
    if number == 3:
        break
    print(number)
```

Output:

```text
1
2
```

Example:

```python
for number in [1, 2, 3, 4, 5]:
    if number == 3:
        continue
    print(number)
```

Output:

```text
1
2
4
5
```

Choose based on intent:

```text
stop completely -> break
skip this item -> continue
```

---

# Loop `else`

Python loops can have an `else` clause.

The loop `else` runs only if the loop finishes without `break`.

Example:

```python
numbers = [1, 3, 5]

for number in numbers:
    if number % 2 == 0:
        print("found even")
        break
else:
    print("no even number found")
```

Output:

```text
no even number found
```

If `break` runs, `else` is skipped:

```python
numbers = [1, 4, 5]

for number in numbers:
    if number % 2 == 0:
        print("found even")
        break
else:
    print("no even number found")
```

Output:

```text
found even
```

Loop `else` means:

```text
no break occurred
```

It does not mean:

```text
the loop condition was false initially
```

Loop `else` is useful for search.

---

# Nested Loops

A nested loop is a loop inside another loop.

Example:

```python
for row in range(3):
    for column in range(2):
        print(row, column)
```

Output:

```text
0 0
0 1
1 0
1 1
2 0
2 1
```

Mental model:

```text
outer loop picks one row
    inner loop runs completely for that row
outer loop moves to next row
    inner loop runs completely again
```

For every single outer iteration, the inner loop completes all its iterations.

---

# Reading Nested Loop Output

Use a trace table for nested loops.

Code:

```python
for x in range(2):
    for y in range(3):
        print(x, y)
```

Trace:

```text
x = 0, y = 0
x = 0, y = 1
x = 0, y = 2
x = 1, y = 0
x = 1, y = 1
x = 1, y = 2
```

The inner variable changes faster.

The outer variable changes slower.

This pattern appears in:

* Grids.
* Matrices.
* Tables.
* Pair comparisons.
* Nested data.

---

# Nested Looping Over Lists

Example:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
]

for row in matrix:
    for value in row:
        print(value)
```

Output:

```text
1
2
3
4
5
6
```

The outer loop iterates over rows.

The inner loop iterates over values inside each row.

This is common for two-dimensional data.

---

# Breaking From Nested Loops

`break` exits only the nearest loop.

Example:

```python
for row in range(3):
    for column in range(3):
        if column == 1:
            break
        print(row, column)
```

Output:

```text
0 0
1 0
2 0
```

The `break` exits the inner loop only.

The outer loop continues.

If you need to stop both loops, common options are:

* Use a flag.
* Move the logic into a function and `return`.
* Use a helper generator.

Function approach:

```python
def find_pair(grid, target):
    for row_index, row in enumerate(grid):
        for column_index, value in enumerate(row):
            if value == target:
                return row_index, column_index

    return None
```

Returning from a function exits both loops cleanly.

---

# Nested Loop Cost

Nested loops multiply work.

Example:

```python
for x in range(3):
    for y in range(4):
        print(x, y)
```

The inner block runs:

```text
3 * 4 = 12 times
```

If both loops depend on input size:

```python
for a in items:
    for b in items:
        compare(a, b)
```

The work grows quickly as `items` grows.

This is the beginning of complexity thinking.

Later chapters discuss data-structure complexity in detail.

---

# `enumerate()`

Use `enumerate()` when you need both index and value.

Example:

```python
names = ["Ada", "Linus", "Grace"]

for index, name in enumerate(names):
    print(index, name)
```

Output:

```text
0 Ada
1 Linus
2 Grace
```

This is better than:

```python
for index in range(len(names)):
    print(index, names[index])
```

`enumerate()` expresses the intent directly:

```text
iterate with indexes
```

You can choose the starting index:

```python
for position, name in enumerate(names, start=1):
    print(position, name)
```

Output:

```text
1 Ada
2 Linus
3 Grace
```

---

# `zip()`

Use `zip()` to iterate over multiple iterables in parallel.

Example:

```python
names = ["Ada", "Linus", "Grace"]
scores = [95, 88, 91]

for name, score in zip(names, scores):
    print(name, score)
```

Output:

```text
Ada 95
Linus 88
Grace 91
```

`zip()` stops when the shortest iterable is exhausted.

Example:

```python
names = ["Ada", "Linus", "Grace"]
scores = [95, 88]

for name, score in zip(names, scores):
    print(name, score)
```

Output:

```text
Ada 95
Linus 88
```

The extra `"Grace"` is ignored.

This behavior is useful, but it can hide bugs when lengths should match.

---

# Mutating While Iterating

Be careful when changing a collection while looping over it.

Bug:

```python
numbers = [1, 2, 3, 4]

for number in numbers:
    if number % 2 == 0:
        numbers.remove(number)

print(numbers)
```

This kind of code can skip elements because the list shifts while the loop is reading it.

Safer option:

```python
numbers = [1, 2, 3, 4]
filtered = []

for number in numbers:
    if number % 2 != 0:
        filtered.append(number)

print(filtered)
```

Output:

```text
[1, 3]
```

Another safe option is to iterate over a copy:

```python
for number in numbers[:]:
    if number % 2 == 0:
        numbers.remove(number)
```

But building a new list is often clearer.

List comprehensions are covered later in the data structures part.

---

# Common Loop Patterns

## Counting

```python
count = 0

for item in items:
    if item.is_valid:
        count += 1
```

## Accumulating

```python
total = 0

for price in prices:
    total += price
```

## Searching

```python
found = None

for user in users:
    if user.id == target_id:
        found = user
        break
```

## Filtering

```python
active_users = []

for user in users:
    if user.is_active:
        active_users.append(user)
```

## Transforming

```python
names = []

for user in users:
    names.append(user.name)
```

## Retrying

```python
attempts = 0

while attempts < 3:
    attempts += 1

    if send_request():
        break
else:
    print("request failed")
```

These patterns appear constantly in real programs.

---

# Choosing `for` vs `while`

Use `for` when you know the source of items:

```python
for line in lines:
    process(line)
```

Use `while` when the loop is controlled by a condition:

```python
while queue:
    task = queue.pop()
    process(task)
```

Use `while True` when the loop is intentionally open-ended and exits internally:

```python
while True:
    command = input("> ")

    if command == "quit":
        break
```

Most collection processing in Python uses `for`.

Most state-driven repetition uses `while`.

---

# Common Mistakes

## Misconception 1

### `range(5)` includes 5.

It does not.

`range(5)` produces:

```text
0, 1, 2, 3, 4
```

The stop value is excluded.

## Misconception 2

### `range()` creates a list.

In Python 3, it creates a range object.

Use `list(range(...))` only when you truly need a list.

## Misconception 3

### `break` exits all nested loops.

`break` exits only the nearest loop.

## Misconception 4

### `continue` stops the loop.

`continue` skips the current iteration.

The loop continues with the next item.

## Misconception 5

### Loop `else` means the loop did not run.

Loop `else` means the loop ended without `break`.

## Misconception 6

### You should use `range(len(items))` for every list.

Iterate directly over values unless you need indexes.

## Misconception 7

### Mutating a list while iterating over it is always safe.

It can skip elements or produce confusing behavior.

Prefer building a new list or iterating over a copy.

## Misconception 8

### Reverse looping requires manual index arithmetic.

Use `reversed()` when you only need values in reverse order.

Use reverse `range()` only when indexes matter.

---

# Real-world Usage

## Processing Records

```python
for record in records:
    if not record.is_valid:
        continue

    save(record)
```

## Searching Data

```python
for account in accounts:
    if account.email == email:
        matched_account = account
        break
else:
    matched_account = None
```

## Building Reports

```python
rows = []

for order in orders:
    rows.append({
        "id": order.id,
        "total": order.total,
    })
```

## Walking Nested Data

```python
for section in document:
    for paragraph in section.paragraphs:
        process(paragraph)
```

## Retrying Work

```python
for attempt in range(1, 4):
    if connect():
        break
else:
    raise RuntimeError("could not connect")
```

---

# Internal Mechanics

At a high level, a `while` loop works like this:

```text
evaluate condition
if falsy, jump after loop
run loop body
jump back to condition
```

A `for` loop works differently:

```text
get iterator from iterable
ask iterator for next item
if item exists:
    assign item to loop variable
    run loop body
    ask for next item
if no item exists:
    exit loop
```

This means a `for` loop depends on Python's iteration protocol.

The full protocol is covered later in the chapter on iterators.

For now, the important idea is:

`for` loops do not need indexes because Python asks objects for their next value.

---

# Concept Connections

Loops connect to earlier chapters:

* Conditionals: loop exits and skips often depend on `if`.
* Operators: comparisons and arithmetic update loop state.
* Expressions: loop conditions and iterable expressions are evaluated.
* Names and references: loop variables are names bound to objects.
* Mutability: changing containers during iteration can affect behavior.
* Strings: strings can be looped over character by character.

Loops prepare you for:

* Functions that process collections.
* Data structures.
* Comprehensions.
* Iterators.
* Generators.
* File processing.
* Async event loops.

---

# Active Recall

## Easy Recall Questions

1. What does a loop do?
2. When should you use a `while` loop?
3. When should you use a `for` loop?
4. What does `range(5)` produce?
5. What does `break` do?
6. What does `continue` do?
7. What does loop `else` mean?
8. What does `enumerate()` provide?
9. What does `zip()` do?
10. What does `reversed()` do?

## Deep Understanding Questions

1. Why is direct iteration usually better than `range(len(items))`?
2. Why can mutating a list while iterating over it cause bugs?
3. Why does `break` in a nested loop exit only the inner loop?
4. How does reverse `range()` handle the stop value?
5. Why can nested loops become expensive as input grows?
6. How is a `for` loop conceptually different from a `while` loop?

## Predict-the-Output Questions

### Question 1

```python
for number in range(3):
    print(number)
```

### Question 2

```python
for number in range(5, 0, -2):
    print(number)
```

### Question 3

```python
for x in range(2):
    for y in range(2):
        print(x, y)
```

### Question 4

```python
for number in [1, 2, 3, 4]:
    if number == 3:
        break
    print(number)
else:
    print("done")
```

### Question 5

```python
for number in [1, 2, 3, 4]:
    if number % 2 == 0:
        continue
    print(number)
```

### Question 6

```python
names = ["Ada", "Linus"]

for index, name in enumerate(names, start=1):
    print(index, name)
```

---

# Practical Exercises

## Exercise 1

Write a `while` loop that counts from 1 to 10.

Then rewrite it as a `for` loop.

## Exercise 2

Write a loop that prints numbers from 10 down to 1.

Use `range()` with a negative step.

## Exercise 3

Write a loop that prints the characters of a string in reverse order.

Use `reversed()`.

## Exercise 4

Given a list of numbers, calculate:

* Total.
* Count.
* Average.

Do not use `sum()` for the first version.

## Exercise 5

Given a list of names, print each name with its position starting at 1.

Use `enumerate()`.

## Exercise 6

Given two lists, `names` and `scores`, print them side by side.

Use `zip()`.

## Exercise 7

Write a nested loop that prints coordinates for a 3 by 3 grid.

## Exercise 8

Write a search loop that finds the first number divisible by 7.

Use `break`.

## Exercise 9

Write a loop that skips invalid records using `continue`.

## Exercise 10

Rewrite a loop that mutates a list during iteration into a safer version that builds a new list.

---

# Summary

Loops express repetition.

The core ideas are:

* `while` repeats while a condition is truthy.
* `for` iterates over an iterable.
* `range()` represents integer sequences.
* Negative steps support reverse counting.
* `reversed()` supports reverse iteration over sequences.
* `break` exits the nearest loop.
* `continue` skips the current iteration.
* Loop `else` runs only when no `break` occurs.
* Nested loops run the inner loop fully for each outer iteration.
* `enumerate()` gives indexes and values.
* `zip()` pairs values from multiple iterables.
* Mutating a collection while iterating over it requires care.

Loops are the foundation for processing collections, files, records, input streams, and many real workflows.

---

# Preview of Chapter 20

Next we study functions.

Loops let you repeat work.

Functions let you name, reuse, and organize work.

Chapter 20 begins Part VI by turning expressions, conditionals, and loops into reusable units of behavior.

We will study:

* What a function object is.
* How `def` creates a function and binds it to a name.
* The difference between defining a function and calling it.
* Parameters and arguments.
* Return values.
* Default arguments.
* Positional and keyword arguments.
* Side effects.
* Function frames at a first-principles level.
* Why focused functions make programs easier to reason about.

The connection is direct:

```text
expressions produce objects
conditionals choose paths
loops repeat paths
functions package paths into reusable behavior
```

Functions are the next major abstraction layer in Python.
