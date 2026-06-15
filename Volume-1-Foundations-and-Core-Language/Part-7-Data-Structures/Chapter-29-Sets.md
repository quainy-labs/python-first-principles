# Chapter 29 — Sets

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a set is.
* Why sets exist.
* How sets differ from lists, tuples, and dictionaries.
* Why sets contain unique values.
* Why set elements must be hashable.
* How to create sets.
* Why `{}` is not an empty set.
* How to add and remove set elements.
* How membership checks work conceptually.
* How to use union, intersection, difference, and symmetric difference.
* How subset and superset checks work.
* How sets are used for deduplication.
* How sets are used for relationship logic.
* How set iteration order should be treated.
* Basic complexity intuition for set operations.
* Common mistakes with sets.

Sets are collections of unique hashable values.

They are built for membership and relationship logic.

---

# Concept Overview

A set is an unordered collection of unique hashable values.

Example:

```python
languages = {"Python", "JavaScript", "Go"}
```

Conceptually:

```text
set object
    contains "Python"
    contains "JavaScript"
    contains "Go"
```

Unlike a list, a set is not organized by position.

You do not access set elements by index:

```python
languages[0]
```

This raises:

```text
TypeError
```

A set answers a different question:

```text
Is this value present?
```

Example:

```python
print("Python" in languages)
```

Output:

```text
True
```

---

# Why Sets Exist

Programs often need uniqueness and fast membership checks.

Examples:

```text
unique user ids
visited pages
seen email addresses
enabled permissions
tags on an article
words in a document
nodes already visited in a graph
```

Lists can store these values:

```python
seen = []
```

But checking membership in a list scans values from left to right:

```python
if user_id in seen:
    ...
```

A set is designed for membership:

```python
seen = set()
```

Now the intent is clear:

```text
track what has already been seen
```

Sets solve two major problems:

* Deduplication.
* Fast membership checks.

They also support mathematical relationship operations such as union and intersection.

---

# Set Literals

A set literal uses curly braces with values.

Example:

```python
numbers = {1, 2, 3}
```

This creates a set.

Duplicate values are removed:

```python
numbers = {1, 2, 2, 3, 3, 3}

print(numbers)
```

Output may display:

```text
{1, 2, 3}
```

The duplicates are not stored.

A set contains each value at most once.

---

# Empty Set

This is not an empty set:

```python
empty = {}
```

It is an empty dictionary.

Check:

```python
print(type({}))
```

Output:

```text
<class 'dict'>
```

To create an empty set, use:

```python
empty = set()
```

Check:

```python
print(type(empty))
```

Output:

```text
<class 'set'>
```

This is a common beginner mistake because both sets and dictionaries use curly braces.

Curly braces with key-value pairs create a dictionary:

```python
{"name": "Ada"}
```

Curly braces with values create a set:

```python
{"Ada", "Grace"}
```

Empty curly braces create a dictionary because that syntax existed first.

---

# Creating Sets With `set()`

`set()` creates a set from an iterable.

Example:

```python
letters = set("banana")

print(letters)
```

Output may display:

```text
{'b', 'a', 'n'}
```

The order is not the point.

The uniqueness is the point.

Example:

```python
numbers = set([1, 2, 2, 3])
print(numbers)
```

Output:

```text
{1, 2, 3}
```

This is a common deduplication pattern.

---

# Sets Are Unordered

Sets are not position-based.

Example:

```python
values = {"a", "b", "c"}

for value in values:
    print(value)
```

The output order should not be relied on.

It may look stable in one run or environment.

Do not build logic that depends on set iteration order.

If you need sorted output:

```python
for value in sorted(values):
    print(value)
```

If you need insertion order and uniqueness together, you may need a different design, such as using dictionary keys.

---

# Sets Store References

Sets store references to objects, like lists, tuples, and dictionaries.

Example:

```python
name = "Ada"
names = {name}
name = "Grace"

print(names)
```

Output:

```text
{'Ada'}
```

Rebinding `name` does not change the set.

The set contains a reference to the string object that existed when the set was created.

Sets do not store variable names.

They store object references.

---

# Elements Must Be Hashable

Set elements must be hashable.

This works:

```python
values = {1, "Ada", (10, 20)}
```

This fails:

```python
values = {[1, 2]}
```

Python raises:

```text
TypeError
```

Lists are mutable and unhashable.

Sets use hash-based lookup, just like dictionaries.

That means values inside a set must have stable hash behavior.

Hashable values can be checked efficiently.

Mutable values such as lists cannot safely be set elements.

---

# Sets And Dictionaries

Sets and dictionaries are closely related.

Dictionary:

```text
hashable key -> value
```

Set:

```text
hashable value exists
```

Example dictionary:

```python
permissions = {
    "read": True,
    "write": True,
}
```

Example set:

```python
permissions = {"read", "write"}
```

If you only care whether a permission exists, a set is clearer.

If you need to associate extra data with each permission, use a dictionary.

---

# Membership Checks

Sets are designed for membership checks.

Example:

```python
allowed_extensions = {".py", ".md", ".txt"}

extension = ".py"

if extension in allowed_extensions:
    print("allowed")
```

Output:

```text
allowed
```

This is usually faster and clearer than using a list for frequent membership checks:

```python
allowed_extensions = [".py", ".md", ".txt"]
```

For tiny collections, performance does not matter much.

For large collections or repeated checks, sets are the right tool.

---

# Adding Elements

Use `.add()` to add one element.

Example:

```python
seen = set()

seen.add("Ada")
seen.add("Grace")
seen.add("Ada")

print(seen)
```

Output may display:

```text
{'Ada', 'Grace'}
```

Adding `"Ada"` twice does not create a duplicate.

Sets contain unique values.

`.add()` mutates the set and returns `None`.

---

# Updating With Multiple Elements

Use `.update()` to add multiple elements from an iterable.

Example:

```python
values = {1, 2}
values.update([2, 3, 4])

print(values)
```

Output:

```text
{1, 2, 3, 4}
```

`.update()` adds each element from the iterable.

Compare with `.add()`:

```python
values = {1, 2}
values.add((3, 4))

print(values)
```

Output:

```text
{1, 2, (3, 4)}
```

`.add()` adds one object.

`.update()` iterates and adds many objects.

---

# Removing Elements

Sets provide several removal methods.

`remove()` removes an element and raises an error if missing:

```python
values = {1, 2, 3}
values.remove(2)

print(values)
```

Output:

```text
{1, 3}
```

If missing:

```python
values.remove(99)
```

Python raises:

```text
KeyError
```

`discard()` removes an element if present and does nothing if missing:

```python
values.discard(99)
```

No error.

Use `remove()` when missing is a bug.

Use `discard()` when missing is acceptable.

---

# `pop()` And `clear()`

`pop()` removes and returns an arbitrary element.

Example:

```python
values = {"a", "b", "c"}
item = values.pop()

print(item)
print(values)
```

Do not assume which item is removed.

Sets are unordered.

`clear()` removes all elements:

```python
values = {1, 2, 3}
values.clear()

print(values)
```

Output:

```text
set()
```

Notice that an empty set displays as:

```text
set()
```

not:

```text
{}
```

because `{}` means empty dictionary.

---

# Deduplication

Sets remove duplicates.

Example:

```python
names = ["Ada", "Grace", "Ada", "Linus", "Grace"]
unique_names = set(names)

print(unique_names)
```

Output may display:

```text
{'Ada', 'Grace', 'Linus'}
```

If order does not matter, this is simple.

If order matters, use a dictionary-based approach:

```python
unique_names = list(dict.fromkeys(names))

print(unique_names)
```

Output:

```text
['Ada', 'Grace', 'Linus']
```

This works because dictionary keys are unique and preserve insertion order.

---

# Union

Union combines values from both sets.

Example:

```python
python_users = {"Ada", "Grace"}
javascript_users = {"Grace", "Linus"}

all_users = python_users | javascript_users

print(all_users)
```

Output may display:

```text
{'Ada', 'Grace', 'Linus'}
```

Method form:

```python
all_users = python_users.union(javascript_users)
```

Union answers:

```text
What is in either set?
```

---

# Intersection

Intersection keeps values present in both sets.

Example:

```python
python_users = {"Ada", "Grace"}
javascript_users = {"Grace", "Linus"}

both = python_users & javascript_users

print(both)
```

Output:

```text
{'Grace'}
```

Method form:

```python
both = python_users.intersection(javascript_users)
```

Intersection answers:

```text
What is in both sets?
```

---

# Difference

Difference keeps values in the left set that are not in the right set.

Example:

```python
python_users = {"Ada", "Grace"}
javascript_users = {"Grace", "Linus"}

python_only = python_users - javascript_users

print(python_only)
```

Output:

```text
{'Ada'}
```

Method form:

```python
python_only = python_users.difference(javascript_users)
```

Difference answers:

```text
What is in the first set but not the second?
```

Order matters:

```python
print(python_users - javascript_users)
print(javascript_users - python_users)
```

These produce different sets.

---

# Symmetric Difference

Symmetric difference keeps values that are in exactly one set.

Example:

```python
python_users = {"Ada", "Grace"}
javascript_users = {"Grace", "Linus"}

exclusive = python_users ^ javascript_users

print(exclusive)
```

Output may display:

```text
{'Ada', 'Linus'}
```

Method form:

```python
exclusive = python_users.symmetric_difference(javascript_users)
```

Symmetric difference answers:

```text
What is in one set or the other, but not both?
```

---

# Subsets And Supersets

A set is a subset if all its elements are contained in another set.

Example:

```python
required = {"read"}
granted = {"read", "write"}

print(required <= granted)
```

Output:

```text
True
```

This asks:

```text
Are all required permissions granted?
```

A set is a superset if it contains all elements of another set:

```python
print(granted >= required)
```

Output:

```text
True
```

Strict subset:

```python
print(required < granted)
```

Strict means:

```text
subset, but not equal
```

---

# Disjoint Sets

Two sets are disjoint if they have no elements in common.

Example:

```python
admins = {"Ada", "Grace"}
banned = {"Linus"}

print(admins.isdisjoint(banned))
```

Output:

```text
True
```

If one user appears in both:

```python
banned = {"Grace"}

print(admins.isdisjoint(banned))
```

Output:

```text
False
```

This is useful for conflict checks.

---

# Mutating Set Operations

Operators such as `|`, `&`, `-`, and `^` create new sets.

Example:

```python
a = {1, 2}
b = {2, 3}

c = a | b

print(a)
print(c)
```

Output:

```text
{1, 2}
{1, 2, 3}
```

Mutating forms update in place:

```python
a |= b
```

Common mutating methods:

```python
a.update(b)
a.intersection_update(b)
a.difference_update(b)
a.symmetric_difference_update(b)
```

Use non-mutating operations when you need to preserve the original sets.

Use mutating operations when updating the set is the intent.

---

# Set Equality

Set equality compares elements, not order.

Example:

```python
a = {1, 2, 3}
b = {3, 2, 1}

print(a == b)
```

Output:

```text
True
```

Sets are unordered.

The two sets contain the same values, so they are equal.

This is useful when order should not matter.

Example:

```python
expected = {"read", "write"}
actual = {"write", "read"}

print(actual == expected)
```

Output:

```text
True
```

---

# Frozenset

`frozenset` is an immutable set.

Example:

```python
permissions = frozenset({"read", "write"})

print(permissions)
```

You cannot add to it:

```python
permissions.add("delete")
```

Python raises:

```text
AttributeError
```

Why does `frozenset` exist?

Because normal sets are mutable and unhashable.

A `frozenset` can be hashable if its elements are hashable.

That means it can be used as a dictionary key or as an element inside another set.

Example:

```python
groups = {
    frozenset({"read", "write"}): "editor",
}
```

`frozenset` is less common than `set`, but useful when an immutable set is needed.

---

# Sets And Lists

Use a list when:

* Order matters.
* Duplicates matter.
* You need indexing or slicing.
* You need to preserve sequence structure.

Use a set when:

* Uniqueness matters.
* Membership checks are frequent.
* Order does not matter.
* You need set operations.

Example:

```python
visited_pages = set()
page_history = []
```

`visited_pages` answers:

```text
Have we seen this page before?
```

`page_history` answers:

```text
In what order did we visit pages?
```

Different questions require different structures.

---

# Sets And Dictionaries

Use a set when you only care whether a value exists.

Example:

```python
enabled_features = {"search", "export"}
```

Use a dictionary when each key has associated data.

Example:

```python
feature_config = {
    "search": {"enabled": True, "limit": 20},
    "export": {"enabled": True, "format": "csv"},
}
```

Set:

```text
feature exists
```

Dictionary:

```text
feature -> configuration
```

The distinction is about meaning.

---

# Sets And Strings

Sets are useful for character membership.

Example:

```python
vowels = {"a", "e", "i", "o", "u"}

def count_vowels(text):
    count = 0

    for character in text.casefold():
        if character in vowels:
            count += 1

    return count

print(count_vowels("Python"))
```

Output:

```text
1
```

The set makes membership clear and efficient:

```text
is this character one of these values?
```

---

# Sets For Visited Tracking

Sets are common in traversal algorithms.

Example:

```python
visited = set()

def visit(node):
    if node in visited:
        return

    visited.add(node)

    for neighbor in node.neighbors:
        visit(neighbor)
```

The set prevents repeated work.

It can also prevent infinite traversal when relationships contain cycles.

This pattern appears in:

* Graph traversal.
* Web crawling.
* Dependency resolution.
* File scanning.
* Search algorithms.

---

# Sets For Permission Checks

Sets model permissions well.

Example:

```python
required = {"read", "write"}
granted = {"read", "write", "delete"}

if required <= granted:
    print("allowed")
```

Output:

```text
allowed
```

This is clearer than manually checking each permission:

```python
if "read" in granted and "write" in granted:
    print("allowed")
```

Set operations express relationship logic directly.

---

# Sets For Comparing Collections

Sets are useful for comparing expected and actual values.

Example:

```python
expected = {"id", "name", "email"}
actual = {"id", "name", "email", "debug"}

missing = expected - actual
extra = actual - expected

print(missing)
print(extra)
```

Output:

```text
set()
{'debug'}
```

This is common in validation:

```text
which required fields are missing?
which unexpected fields were provided?
```

---

# Copying Sets

Set assignment does not copy.

Example:

```python
a = {1, 2}
b = a

b.add(3)

print(a)
print(b)
```

Output:

```text
{1, 2, 3}
{1, 2, 3}
```

There is one set object and two names.

To copy:

```python
b = a.copy()
```

or:

```python
b = set(a)
```

Sets contain hashable elements, so shallow-copy problems are usually less surprising than with lists of lists.

But if a set contains custom mutable-but-hashable objects, object mutation can still be dangerous.

That advanced case comes later.

---

# Complexity Intuition

Common set operation intuition:

| Operation | Intuition |
|---|---|
| `value in set` | Usually fast membership |
| `set.add(value)` | Usually fast insert |
| `set.remove(value)` | Usually fast removal if present |
| `set.discard(value)` | Usually fast removal if present or absent |
| `a | b` | Visits elements from both sets |
| `a & b` | Finds shared elements |
| `a - b` | Finds elements only in left set |
| `a <= b` | Checks containment relationship |
| iterating | Visits every element |

Sets are optimized for membership, uniqueness, and relationship operations.

They are not designed for indexing, ordering, or slicing.

---

# Choosing Between List, Tuple, Dict, And Set

Use a list for ordered mutable collections:

```python
tasks = []
```

Use a tuple for fixed ordered groups:

```python
point = (x, y)
```

Use a dictionary for key-value mapping:

```python
users_by_id = {user.id: user}
```

Use a set for uniqueness and membership:

```python
seen_ids = set()
```

Ask:

```text
Do I need order?
Do I need mutation?
Do I need key-value mapping?
Do I need uniqueness?
Do I need fast membership?
```

The answer usually determines the structure.

---

# Common Mistakes

## Misconception 1

### `{}` creates an empty set.

It creates an empty dictionary.

Use `set()` for an empty set.

## Misconception 2

### Sets preserve insertion order like dictionaries.

Do not rely on set iteration order.

Sets are not sequence types.

## Misconception 3

### Sets can contain lists.

Lists are unhashable and cannot be set elements.

Use tuples when you need fixed hashable compound values.

## Misconception 4

### `remove()` and `discard()` behave the same.

`remove()` raises `KeyError` if the value is missing.

`discard()` does nothing if the value is missing.

## Misconception 5

### `pop()` removes the first set element.

Sets have no first element.

`pop()` removes an arbitrary element.

## Misconception 6

### A set is just a list without duplicates.

A set is not indexed, not ordered, and supports set algebra.

It is a different structure with different tradeoffs.

## Misconception 7

### Set operations mutate by default.

Operators such as `|`, `&`, `-`, and `^` create new sets.

Mutating forms use update methods or augmented assignment.

---

# Real-world Usage

## Deduplicating Values

```python
unique_emails = set(emails)
```

## Membership Checks

```python
if extension in allowed_extensions:
    process(file)
```

## Required Fields

```python
required = {"id", "name", "email"}
provided = set(payload)

missing = required - provided
```

## Permissions

```python
if required_permissions <= user_permissions:
    allow()
```

## Tag Comparison

```python
shared_tags = article_a.tags & article_b.tags
```

## Visited Tracking

```python
visited = set()
```

## Conflict Detection

```python
if requested_ports & unavailable_ports:
    raise ValueError("some ports are unavailable")
```

---

# Internal Mechanics

At a high level, a set is hash-table-based.

Conceptual model:

```text
value -> hash(value) -> candidate location -> membership check
```

Unlike dictionaries, sets do not store a separate value for each key.

A set stores the presence of values.

Dictionary:

```text
key -> associated value
```

Set:

```text
value exists
```

This is why sets and dictionaries share hashability requirements.

The exact CPython implementation is more detailed.

For now:

```text
set = mutable collection of unique hashable object references
```

---

# Concept Connections

Sets connect to earlier chapters:

* Objects: sets are objects.
* Names and references: names refer to set objects.
* Mutability: sets can be changed in place.
* Identity and equality: equal sets do not need to be identical.
* Lists: sets can deduplicate list values.
* Tuples: hashable tuples can be set elements.
* Dictionaries: sets share hashability and fast membership ideas.
* Booleans: membership checks produce booleans.
* Operators: set algebra uses operators such as `|`, `&`, `-`, and `^`.
* Loops: sets are iterable but unordered.

Sets prepare you for:

* Comprehension patterns.
* Graph traversal.
* Deduplication pipelines.
* Relationship logic.
* Performance-aware collection choice.
* Hash table internals.

---

# Active Recall

## Easy Recall Questions

1. What is a set?
2. What does a set guarantee about its elements?
3. How do you create an empty set?
4. Why does `{}` not create an empty set?
5. Can a list be a set element?
6. What does `.add()` do?
7. What is the difference between `.remove()` and `.discard()`?
8. What does union mean?
9. What does intersection mean?
10. What does difference mean?

## Deep Understanding Questions

1. Why must set elements be hashable?
2. Why should you not rely on set iteration order?
3. When is a set better than a list?
4. When is a dictionary better than a set?
5. Why does `set(names)` remove duplicates?
6. Why does `pop()` remove an arbitrary element?
7. How do set operations express relationship logic more clearly than manual loops?

## Predict-the-Output Questions

### Question 1

```python
values = {1, 2, 2, 3}
print(values)
```

### Question 2

```python
print(type({}))
print(type(set()))
```

### Question 3

```python
a = {"read", "write"}
b = {"write", "delete"}

print(a & b)
print(a - b)
```

### Question 4

```python
required = {"read"}
granted = {"read", "write"}

print(required <= granted)
```

### Question 5

```python
values = {1, 2}
alias = values
alias.add(3)

print(values)
```

### Question 6

```python
values = set("banana")
print(values)
```

Explain what should and should not be relied on.

---

# Practical Exercises

## Exercise 1

Create a set of programming languages.

Add one language.

Try adding a duplicate.

Print the set.

## Exercise 2

Create an empty set correctly.

Then create `{}` and print its type.

Explain the difference.

## Exercise 3

Given a list of email addresses with duplicates, create a set of unique emails.

Then create an order-preserving unique list.

## Exercise 4

Given:

```python
required = {"id", "name", "email"}
provided = {"id", "email", "debug"}
```

Compute:

* missing fields
* extra fields
* shared fields

## Exercise 5

Model user permissions with sets.

Check whether a user has all required permissions.

## Exercise 6

Create two sets of tags.

Find:

* tags in either article
* tags in both articles
* tags only in the first article
* tags in exactly one article

## Exercise 7

Write a function that receives a list and returns `True` if it contains duplicates.

Use a set.

## Exercise 8

Write a function that counts vowels in a string using a set of vowels.

## Exercise 9

Try to put a list inside a set.

Observe the error.

Then replace the list with a tuple.

## Exercise 10

Choose whether each should be a list, tuple, dictionary, or set:

* Unique tags on a blog post.
* Ordered steps in a workflow.
* RGB color.
* User id to user object mapping.
* Visited nodes in a graph.
* Rows in a CSV file.

Explain your choices.

---

# Summary

In this chapter we learned:

* Sets store unique hashable values.
* Sets are not sequence types.
* Sets are unordered.
* `{}` creates an empty dictionary, not an empty set.
* `set()` creates an empty set.
* `set(iterable)` deduplicates values.
* Set elements must be hashable.
* Sets and dictionaries share hash-table ideas.
* `in` checks membership.
* `.add()` adds one element.
* `.update()` adds many elements.
* `.remove()` raises if the element is missing.
* `.discard()` does not raise if the element is missing.
* `.pop()` removes an arbitrary element.
* Union combines elements from either set.
* Intersection keeps elements in both sets.
* Difference keeps elements only in the left set.
* Symmetric difference keeps elements in exactly one set.
* Subset and superset checks express containment relationships.
* `frozenset` is an immutable set.
* Sets are useful for deduplication, membership, permissions, visited tracking, and relationship logic.

Core model:

```text
set object
    |
    ├── contains hashable value
    ├── contains hashable value
    └── contains hashable value
```

Sets are mutable collections of unique hashable object references.

---

# Preview of Chapter 30

Next we study comprehension patterns.

Lists, dictionaries, and sets can all be built with comprehensions.

Comprehensions combine:

```text
expression
loop
optional condition
collection construction
```

We will study:

* List comprehensions.
* Dictionary comprehensions.
* Set comprehensions.
* Nested comprehensions.
* Filtering with conditions.
* Transforming values.
* Readability rules.
* When a normal loop is better.

The transition is direct:

```text
list -> list comprehension
dict -> dictionary comprehension
set  -> set comprehension
```

Comprehensions are Python's compact syntax for building collections from existing iterables.
