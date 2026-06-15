# Chapter 30 — Comprehension Patterns

---

# Learning Objectives

By the end of this chapter, you should understand:

* What comprehensions are.
* Why comprehensions exist.
* How list comprehensions work.
* How dictionary comprehensions work.
* How set comprehensions work.
* How expressions, loops, and conditions combine inside comprehensions.
* How filtering works in a comprehension.
* How transformation works in a comprehension.
* How nested comprehensions work.
* How comprehension variables are scoped.
* How comprehensions compare to loops.
* How comprehensions compare to `map()` and `filter()`.
* When comprehensions improve readability.
* When a normal loop is better.
* Common mistakes with comprehensions.

Comprehensions are compact syntax for building collections from iterables.

They combine ideas you already know:

```text
expression + loop + optional condition + collection construction
```

---

# Concept Overview

A comprehension builds a collection.

Example:

```python
numbers = [1, 2, 3]
squares = [number * number for number in numbers]

print(squares)
```

Output:

```text
[1, 4, 9]
```

The list comprehension:

```python
[number * number for number in numbers]
```

means:

```text
for each number in numbers:
    compute number * number
    put the result in a new list
```

Equivalent loop:

```python
squares = []

for number in numbers:
    squares.append(number * number)
```

The comprehension is shorter.

More importantly, it expresses the purpose directly:

```text
build a list by transforming each item
```

---

# Why Comprehensions Exist

Programs often build one collection from another.

Examples:

```text
names -> normalized names
users -> active users
records -> ids
words -> unique lowercase words
products -> product id to product
numbers -> squares
files -> allowed files
```

Without comprehensions, this usually requires:

```python
result = []

for item in items:
    if condition(item):
        result.append(transform(item))
```

That pattern is common enough that Python gives it dedicated syntax.

Comprehensions make collection-building:

* Shorter.
* More direct.
* Less error-prone for simple transformations.
* Easier to recognize as a standard pattern.

But comprehensions are not always better.

They are best when the collection-building logic is simple and readable.

---

# List Comprehension Syntax

Basic syntax:

```python
[expression for item in iterable]
```

Example:

```python
numbers = [1, 2, 3]
doubled = [number * 2 for number in numbers]

print(doubled)
```

Output:

```text
[2, 4, 6]
```

Parts:

```text
expression: number * 2
loop:       for number in numbers
result:     list
```

Execution model:

```text
create empty list
for each item:
    evaluate expression
    append result
return list
```

---

# Transforming Values

The expression part transforms each item.

Example:

```python
names = ["ada", "linus", "grace"]
display_names = [name.title() for name in names]

print(display_names)
```

Output:

```text
['Ada', 'Linus', 'Grace']
```

The original list is unchanged.

The comprehension creates a new list.

Another example:

```python
prices = [10, 20, 30]
with_tax = [price * 1.1 for price in prices]
```

The expression can be any expression:

```python
f"{name}@example.com"
price * quantity
user.email.casefold()
record["id"]
```

Keep it readable.

If the expression becomes complex, use a helper function.

---

# Filtering Values

A comprehension can include an `if` clause.

Syntax:

```python
[expression for item in iterable if condition]
```

Example:

```python
numbers = [1, 2, 3, 4, 5, 6]
evens = [number for number in numbers if number % 2 == 0]

print(evens)
```

Output:

```text
[2, 4, 6]
```

Execution model:

```text
for each number:
    if condition is truthy:
        evaluate expression
        append result
```

Equivalent loop:

```python
evens = []

for number in numbers:
    if number % 2 == 0:
        evens.append(number)
```

The condition decides whether the item contributes to the result.

---

# Transforming And Filtering Together

Comprehensions can transform and filter at the same time.

Example:

```python
names = [" Ada ", "", "  Grace", "Linus  "]

cleaned = [name.strip().title() for name in names if name.strip()]

print(cleaned)
```

Output:

```text
['Ada', 'Grace', 'Linus']
```

This works, but notice that `name.strip()` appears twice.

For simple cases, that may be acceptable.

For clarity, use a helper function:

```python
def normalize_name(name):
    return name.strip().title()

cleaned = [normalize_name(name) for name in names if name.strip()]
```

If the condition and transformation are both complex, a normal loop may be better.

---

# Comprehension Order

Read a comprehension in this order:

```python
[expression for item in iterable if condition]
```

Read as:

```text
for item in iterable:
    if condition:
        produce expression
```

Example:

```python
[number * number for number in numbers if number > 0]
```

Read as:

```text
for each number in numbers
if number is positive
produce number squared
```

Do not start by reading the expression only.

Understand the loop and condition together.

---

# Dictionary Comprehensions

A dictionary comprehension builds a dictionary.

Syntax:

```python
{key_expression: value_expression for item in iterable}
```

Example:

```python
names = ["Ada", "Linus", "Grace"]
name_lengths = {name: len(name) for name in names}

print(name_lengths)
```

Output:

```text
{'Ada': 3, 'Linus': 5, 'Grace': 5}
```

Equivalent loop:

```python
name_lengths = {}

for name in names:
    name_lengths[name] = len(name)
```

Dictionary comprehensions need:

* A key expression.
* A value expression.
* A loop.
* Optional filtering.

---

# Dictionary Comprehension With Filtering

Example:

```python
users = [
    {"id": 1, "name": "Ada", "active": True},
    {"id": 2, "name": "Linus", "active": False},
    {"id": 3, "name": "Grace", "active": True},
]

active_users = {
    user["id"]: user
    for user in users
    if user["active"]
}

print(active_users)
```

Conceptually:

```text
for each user:
    if user is active:
        map user id to user object
```

This pattern is common when creating lookup tables.

Example:

```python
users_by_id = {user["id"]: user for user in users}
```

The result is a dictionary where lookup by id is direct.

---

# Duplicate Keys In Dictionary Comprehensions

Dictionary keys must be unique.

If a comprehension produces the same key more than once, the later value wins.

Example:

```python
pairs = [("a", 1), ("b", 2), ("a", 3)]

data = {key: value for key, value in pairs}

print(data)
```

Output:

```text
{'a': 3, 'b': 2}
```

The second `"a"` replaces the first `"a"`.

This is not special to comprehensions.

It is normal dictionary behavior.

When using dictionary comprehensions, ask:

```text
Can two items produce the same key?
If yes, which value should win?
```

---

# Set Comprehensions

A set comprehension builds a set.

Syntax:

```python
{expression for item in iterable}
```

Example:

```python
words = ["Python", "python", "PYTHON", "Java"]
normalized = {word.casefold() for word in words}

print(normalized)
```

Output may display:

```text
{'python', 'java'}
```

The set removes duplicates.

Equivalent loop:

```python
normalized = set()

for word in words:
    normalized.add(word.casefold())
```

Set comprehensions are useful for deduplication plus transformation.

---

# Set Comprehension With Filtering

Example:

```python
words = ["", "Python", "  ", "Java"]

non_empty = {word.strip() for word in words if word.strip()}

print(non_empty)
```

Output may display:

```text
{'Python', 'Java'}
```

Again, note the repeated `word.strip()`.

For clarity:

```python
def clean(word):
    return word.strip()

non_empty = {clean(word) for word in words if clean(word)}
```

But this calls `clean()` twice.

A normal loop may be clearer if avoiding repeated work matters:

```python
non_empty = set()

for word in words:
    cleaned = word.strip()

    if cleaned:
        non_empty.add(cleaned)
```

Comprehensions are not mandatory.

---

# List, Dict, And Set Compared

List comprehension:

```python
[expression for item in iterable]
```

Dictionary comprehension:

```python
{key_expression: value_expression for item in iterable}
```

Set comprehension:

```python
{expression for item in iterable}
```

Examples:

```python
numbers = [1, 2, 3]

list_result = [number * 2 for number in numbers]
dict_result = {number: number * 2 for number in numbers}
set_result = {number * 2 for number in numbers}
```

Results:

```text
[2, 4, 6]
{1: 2, 2: 4, 3: 6}
{2, 4, 6}
```

The brackets determine the collection type.

The expressions determine what goes into it.

---

# Generator Expression Preview

A generator expression looks like a list comprehension but uses parentheses.

Example:

```python
squares = (number * number for number in range(5))
```

This does not immediately create a list.

It creates a generator object.

Generators are covered later in Volume II.

For now, know the difference:

```python
[number * number for number in range(5)]  # list now
(number * number for number in range(5))  # generator, values later
```

When passed directly to a function, extra parentheses are often omitted:

```python
total = sum(number * number for number in range(5))
```

This is a generator expression, not a tuple comprehension.

There is no tuple comprehension syntax.

Use `tuple(...)` with a generator expression:

```python
squares = tuple(number * number for number in range(5))
```

---

# Nested Loops In Comprehensions

Comprehensions can contain multiple `for` clauses.

Example:

```python
pairs = [(x, y) for x in [1, 2] for y in ["a", "b"]]

print(pairs)
```

Output:

```text
[(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
```

Equivalent loop:

```python
pairs = []

for x in [1, 2]:
    for y in ["a", "b"]:
        pairs.append((x, y))
```

The order of `for` clauses matches the loop nesting order.

Read:

```python
[(x, y) for x in xs for y in ys]
```

as:

```python
for x in xs:
    for y in ys:
        produce (x, y)
```

---

# Nested Comprehensions

A nested comprehension is a comprehension inside another comprehension.

Example:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
]

doubled = [[value * 2 for value in row] for row in matrix]

print(doubled)
```

Output:

```text
[[2, 4, 6], [8, 10, 12]]
```

Outer comprehension:

```text
for row in matrix
```

Inner comprehension:

```text
for value in row
```

This can be useful for simple matrix transformations.

But nested comprehensions can become hard to read quickly.

If the reader has to stop and decode it, use normal loops.

---

# Flattening With A Comprehension

Flattening one level:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
]

flat = [value for row in matrix for value in row]

print(flat)
```

Output:

```text
[1, 2, 3, 4, 5, 6]
```

Equivalent loop:

```python
flat = []

for row in matrix:
    for value in row:
        flat.append(value)
```

This is a common pattern, but it can be confusing at first because the expression appears before the loops.

Read the loops left to right:

```text
for row in matrix
    for value in row
        produce value
```

---

# Conditions With Nested Loops

Example:

```python
pairs = [
    (x, y)
    for x in range(3)
    for y in range(3)
    if x != y
]

print(pairs)
```

Output:

```text
[(0, 1), (0, 2), (1, 0), (1, 2), (2, 0), (2, 1)]
```

Equivalent loop:

```python
pairs = []

for x in range(3):
    for y in range(3):
        if x != y:
            pairs.append((x, y))
```

When multiple loops and conditions appear, formatting matters.

Multi-line comprehensions are often more readable than one long line.

---

# Conditional Expressions Inside Comprehensions

Do not confuse filtering `if` with conditional expressions.

Filtering:

```python
[number for number in numbers if number >= 0]
```

This keeps only non-negative numbers.

Conditional expression:

```python
["positive" if number >= 0 else "negative" for number in numbers]
```

This produces one label for every number.

Example:

```python
numbers = [-2, 0, 3]
labels = ["non-negative" if number >= 0 else "negative" for number in numbers]

print(labels)
```

Output:

```text
['negative', 'non-negative', 'non-negative']
```

Difference:

```text
if at the end -> filter items
if inside expression -> choose produced value
```

---

# Filtering vs Conditional Output

Compare:

```python
numbers = [-2, 0, 3]

filtered = [number for number in numbers if number >= 0]
labels = ["keep" if number >= 0 else "drop" for number in numbers]

print(filtered)
print(labels)
```

Output:

```text
[0, 3]
['drop', 'keep', 'keep']
```

The filtered comprehension may produce fewer items.

The conditional-output comprehension produces one item for each input item.

Use filtering when excluding values.

Use conditional expressions when choosing among output values.

---

# Comprehension Variables

The loop variable in a comprehension is local to the comprehension.

Example:

```python
values = [number * 2 for number in range(3)]

print(values)
```

After the comprehension, `number` is not available as a new name in the surrounding scope.

This differs from normal `for` loops:

```python
for number in range(3):
    pass

print(number)
```

The loop variable remains bound after a normal loop.

Comprehension scoping helps avoid leaking temporary names.

---

# Comprehensions And Side Effects

Do not use comprehensions mainly for side effects.

Bad:

```python
[print(name) for name in names]
```

This creates a list of `None` values only to print names.

Use a loop:

```python
for name in names:
    print(name)
```

Comprehensions should build collections.

Loops should perform repeated actions.

Rule:

```text
If you do not need the produced collection, use a loop.
```

---

# Comprehensions And Mutation

Comprehensions usually create new collections.

Example:

```python
numbers = [1, 2, 3]
doubled = [number * 2 for number in numbers]

print(numbers)
print(doubled)
```

Output:

```text
[1, 2, 3]
[2, 4, 6]
```

The original list is unchanged.

This makes comprehensions useful for pure-style transformations.

But the expression can still mutate objects if you call mutating methods:

```python
items = [[], []]
result = [item.append("x") for item in items]

print(items)
print(result)
```

Output:

```text
[['x'], ['x']]
[None, None]
```

This is poor style.

Use a loop when mutation is the point.

---

# Comprehensions vs `map()`

From the functional programming chapter:

```python
result = list(map(transform, values))
```

Comprehension:

```python
result = [transform(value) for value in values]
```

Both transform values.

The comprehension is often clearer when the transformation is visible.

`map()` can be clean when:

* The function already exists.
* There is no filtering.
* The pipeline style is already established.

Example:

```python
lengths = list(map(len, names))
```

This is reasonable.

So is:

```python
lengths = [len(name) for name in names]
```

Choose the one that reads better in context.

---

# Comprehensions vs `filter()`

`filter()`:

```python
result = list(filter(is_valid, values))
```

Comprehension:

```python
result = [value for value in values if is_valid(value)]
```

The comprehension makes the kept value explicit.

When filtering and transforming:

```python
result = [transform(value) for value in values if is_valid(value)]
```

This is usually clearer than combining `map()` and `filter()`:

```python
result = list(map(transform, filter(is_valid, values)))
```

Again, clarity decides.

---

# Readability Rules

Use a comprehension when:

* You are building a list, dictionary, or set.
* The loop is simple.
* The condition is simple.
* The expression is simple.
* The result is easier to understand than the equivalent loop.

Use a normal loop when:

* There are multiple steps.
* You need intermediate names.
* You need error handling.
* You need logging.
* You are mutating existing objects.
* There are multiple conditions with different actions.
* The comprehension becomes difficult to read.

Comprehensions are not a badge of skill.

They are a readability tool.

---

# Formatting Comprehensions

Short comprehensions can be one line:

```python
squares = [number * number for number in numbers]
```

Longer comprehensions should be formatted over multiple lines:

```python
active_users_by_id = {
    user["id"]: user
    for user in users
    if user["active"]
}
```

Nested comprehensions should usually be multi-line or rewritten as loops.

Good formatting reveals structure:

```python
result = [
    transform(value)
    for value in values
    if is_valid(value)
]
```

This layout maps directly to:

```text
produce transform(value)
for each value
if valid
```

---

# Avoid Repeated Expensive Work

Be careful when the condition and expression repeat expensive work.

Example:

```python
cleaned = [normalize(value) for value in values if normalize(value)]
```

`normalize(value)` is called twice for kept values.

If `normalize()` is expensive or has side effects, this is a problem.

Use a loop:

```python
cleaned = []

for value in values:
    normalized = normalize(value)

    if normalized:
        cleaned.append(normalized)
```

The loop is longer but clearer and more efficient.

Later, assignment expressions can solve some cases, but they should be used carefully.

---

# Assignment Expression Preview

Python has an assignment expression operator:

```python
:=
```

It can bind a value inside an expression.

Example:

```python
cleaned = [
    normalized
    for value in values
    if (normalized := normalize(value))
]
```

This avoids calling `normalize(value)` twice.

However, assignment expressions can make code harder to read if overused.

For beginners, prefer the explicit loop when intermediate state matters.

This book will revisit assignment expressions later.

---

# Common Patterns

## Extracting A Field

```python
emails = [user["email"] for user in users]
```

## Filtering Records

```python
active_users = [user for user in users if user["active"]]
```

## Building A Lookup Dictionary

```python
users_by_id = {user["id"]: user for user in users}
```

## Deduplicating With Transformation

```python
normalized_words = {word.casefold() for word in words}
```

## Flattening One Level

```python
flat = [value for row in matrix for value in row]
```

## Filtering Missing Values

```python
present = [value for value in values if value is not None]
```

---

# Common Mistakes

## Misconception 1

### Comprehensions are always better than loops.

They are better only when they are clearer.

Normal loops are better for multi-step logic, side effects, mutation, and error handling.

## Misconception 2

### A comprehension should be used for printing.

Do not use a comprehension just for side effects.

Use a loop for repeated actions.

## Misconception 3

### `{expression for item in items}` creates a dictionary.

It creates a set.

A dictionary comprehension needs `key: value`.

## Misconception 4

### Parentheses create a tuple comprehension.

There is no tuple comprehension.

Parentheses around comprehension syntax create a generator expression.

Use `tuple(...)` to build a tuple from a generator expression.

## Misconception 5

### The final `if` chooses between two output values.

The final `if` filters items.

Use a conditional expression to choose between output values.

## Misconception 6

### Nested comprehensions are always Pythonic.

Nested comprehensions can be readable for simple patterns.

They become harmful when they hide control flow.

## Misconception 7

### Dictionary comprehensions preserve all duplicate generated keys.

Dictionaries cannot have duplicate keys.

Later values replace earlier values.

---

# Real-world Usage

## API Data Extraction

```python
emails = [user["email"] for user in response["users"]]
```

## Configuration Filtering

```python
enabled = {
    name: config
    for name, config in features.items()
    if config["enabled"]
}
```

## Validation

```python
missing = [field for field in required_fields if field not in payload]
```

## Normalization

```python
normalized_tags = {tag.strip().casefold() for tag in tags if tag.strip()}
```

## Lookup Tables

```python
products_by_id = {product.id: product for product in products}
```

## Matrix Transformation

```python
doubled = [[value * 2 for value in row] for row in matrix]
```

---

# Internal Mechanics

At a high level, a comprehension is compiled as collection-building code.

Conceptually, a list comprehension:

```python
[expression for item in iterable if condition]
```

behaves like:

```python
result = []

for item in iterable:
    if condition:
        result.append(expression)
```

A dictionary comprehension:

```python
{key: value for item in iterable}
```

behaves like:

```python
result = {}

for item in iterable:
    result[key] = value
```

A set comprehension:

```python
{expression for item in iterable}
```

behaves like:

```python
result = set()

for item in iterable:
    result.add(expression)
```

The exact bytecode is not important yet.

The mental model is:

```text
comprehension = compact collection-building loop
```

---

# Concept Connections

Comprehensions connect to earlier chapters:

* Expressions: the produced value is an expression.
* Conditionals: optional `if` filters items.
* Loops: comprehensions contain loop logic.
* Lists: list comprehensions build lists.
* Dictionaries: dictionary comprehensions build mappings.
* Sets: set comprehensions build unique collections.
* Functions: helper functions keep comprehensions readable.
* Functional programming: comprehensions often replace simple `map()` and `filter()` pipelines.
* Scope: comprehension loop variables do not leak into the surrounding scope.

Comprehensions prepare you for:

* Specialized data structures.
* Iterators.
* Generators.
* Data processing pipelines.
* Cleaner collection transformations.

---

# Active Recall

## Easy Recall Questions

1. What does a comprehension build?
2. What is the basic form of a list comprehension?
3. What is the basic form of a dictionary comprehension?
4. What is the basic form of a set comprehension?
5. What does the final `if` in a comprehension do?
6. How do you create a dictionary comprehension instead of a set comprehension?
7. Is there tuple comprehension syntax?
8. When should you use a normal loop instead of a comprehension?
9. What does a comprehension variable do after the comprehension ends?
10. Why should comprehensions avoid side effects?

## Deep Understanding Questions

1. Why are comprehensions easier to understand after learning expressions, loops, and conditionals?
2. Why can duplicate keys be a problem in dictionary comprehensions?
3. Why can repeated function calls inside comprehensions be inefficient?
4. Why are comprehensions often clearer than `map()` and `filter()` chains?
5. Why can nested comprehensions become difficult to read?
6. How does the collection type determine the comprehension result?

## Predict-the-Output Questions

### Question 1

```python
numbers = [1, 2, 3]
print([number * 2 for number in numbers])
```

### Question 2

```python
numbers = [1, 2, 3, 4]
print([number for number in numbers if number % 2 == 0])
```

### Question 3

```python
names = ["Ada", "Linus"]
print({name: len(name) for name in names})
```

### Question 4

```python
words = ["Python", "python", "PYTHON"]
print({word.casefold() for word in words})
```

### Question 5

```python
pairs = [("a", 1), ("a", 2)]
print({key: value for key, value in pairs})
```

### Question 6

```python
matrix = [[1, 2], [3, 4]]
print([value for row in matrix for value in row])
```

---

# Practical Exercises

## Exercise 1

Create a list of squares from numbers `1` through `10` using a list comprehension.

Then write the equivalent loop.

## Exercise 2

Given a list of names, create a list of normalized names using `.strip()` and `.title()`.

Filter out empty names.

## Exercise 3

Given a list of users represented as dictionaries, build a list of active user emails.

## Exercise 4

Build a dictionary that maps each word to its length.

Use a dictionary comprehension.

## Exercise 5

Build a dictionary that maps user id to user dictionary.

Filter out inactive users.

## Exercise 6

Build a set of normalized tags from a list of messy tag strings.

Use a set comprehension.

## Exercise 7

Flatten one level of a nested list:

```python
matrix = [[1, 2], [3, 4], [5, 6]]
```

Use a comprehension.

Then write the equivalent nested loop.

## Exercise 8

Write a comprehension that labels numbers as `"even"` or `"odd"`.

Then write a comprehension that keeps only even numbers.

Explain the difference.

## Exercise 9

Take a complex comprehension from your own code or invent one.

Rewrite it as a normal loop with intermediate names.

Explain which version is clearer.

## Exercise 10

Write examples of:

* a good list comprehension
* a good dictionary comprehension
* a good set comprehension
* a comprehension that should be rewritten as a loop

---

# Summary

In this chapter we learned:

* Comprehensions build collections from iterables.
* List comprehensions build lists.
* Dictionary comprehensions build dictionaries.
* Set comprehensions build sets.
* Comprehensions combine expressions, loops, and optional conditions.
* The final `if` filters items.
* Conditional expressions choose produced values.
* Dictionary comprehensions require `key: value`.
* Set comprehensions use `{expression for ...}`.
* Parenthesized comprehension syntax creates a generator expression, not a tuple comprehension.
* Nested loops in comprehensions follow normal nested-loop order.
* Nested comprehensions can be useful but can become hard to read.
* Comprehension variables do not leak into the surrounding scope.
* Comprehensions should build collections, not perform side effects.
* Normal loops are better for multi-step logic, mutation, logging, and error handling.

Core model:

```text
comprehension
    |
    ├── expression to produce
    ├── loop over iterable
    ├── optional filter condition
    └── collection type from brackets
```

Comprehensions are compact collection-building loops.

---

# Preview of Chapter 31

Next we study specialized collection tools.

Lists, tuples, dictionaries, sets, and comprehensions cover Python's most common built-in collection patterns.

Now we will look at specialized collection tools for specific access patterns:

* `deque` for efficient queue and double-ended operations.
* `Counter` for counting.
* `defaultdict` for grouping and default values.
* `heapq` for priority queues.
* `bisect` for maintaining sorted order with binary search.

The transition is direct:

```text
general collections -> specialized access patterns
```

Specialized structures exist because no single collection is optimal for every operation.
