# Chapter 27 — Tuples

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a tuple is.
* Why tuples exist.
* How tuples differ from lists.
* How tuple literals work.
* Why the comma matters more than parentheses.
* How tuple indexing and slicing work.
* What tuple immutability means.
* Why tuple immutability is shallow.
* How tuples can contain mutable objects.
* How tuple packing and unpacking work.
* How functions return multiple values using tuples.
* How tuples can represent lightweight records.
* When to choose tuples over lists.
* How tuple equality works.
* How tuple hashing works at a practical level.
* Common mistakes with tuples.

Tuples are ordered sequences like lists.

The key difference is mutability.

Lists are mutable.

Tuples are immutable.

That one difference changes how tuples are used.

---

# Concept Overview

A tuple is an ordered collection of references to objects.

Example:

```python
point = (10, 20)
```

Conceptually:

```text
point ─────▶ tuple object
             ├── index 0 ─▶ int object 10
             └── index 1 ─▶ int object 20
```

This looks similar to a list:

```python
point = [10, 20]
```

But the tuple cannot be changed after creation.

This fails:

```python
point = (10, 20)
point[0] = 99
```

Python raises:

```text
TypeError
```

The tuple object does not support item assignment.

---

# Why Tuples Exist

Tuples exist for fixed ordered groups of values.

Examples:

```text
2D point:          (x, y)
RGB color:         (red, green, blue)
database row:      (id, email, created_at)
date parts:        (year, month, day)
return pair:       (success, value)
coordinate:        (latitude, longitude)
```

The important idea is fixed structure.

A list often means:

```text
a collection that may grow, shrink, or be modified
```

A tuple often means:

```text
a fixed group of related values
```

This is not enforced by the type system beyond immutability, but it is a strong convention.

Use a tuple when position has meaning and the group should not be mutated.

Use a list when you need a mutable sequence of items.

---

# Tuple Literals

A tuple literal is usually written with parentheses:

```python
coordinates = (10, 20)
```

But the comma creates the tuple.

Example:

```python
coordinates = 10, 20

print(coordinates)
print(type(coordinates))
```

Output:

```text
(10, 20)
<class 'tuple'>
```

Parentheses are often used for readability.

The comma is the essential part.

This matters for one-item tuples.

---

# One-Item Tuples

This is not a tuple:

```python
value = (10)

print(type(value))
```

Output:

```text
<class 'int'>
```

Parentheses only group the expression.

This is a tuple:

```python
value = (10,)

print(type(value))
```

Output:

```text
<class 'tuple'>
```

The trailing comma creates the tuple.

Without the comma, Python sees only a parenthesized expression.

Mental model:

```text
(10)   -> int expression grouped by parentheses
(10,)  -> tuple containing one item
```

---

# Empty Tuples

An empty tuple is written with empty parentheses:

```python
empty = ()
```

You can also use:

```python
empty = tuple()
```

Empty tuples are less common in everyday code than empty lists.

An empty list often means:

```text
we will collect items here
```

An empty tuple often means:

```text
there are no fixed values
```

Because tuples are immutable, you do not build them up with `append()`.

You create the complete tuple at once.

---

# Tuple Creation With `tuple()`

`tuple()` creates a tuple from an iterable.

Example:

```python
letters = tuple("abc")
numbers = tuple([1, 2, 3])

print(letters)
print(numbers)
```

Output:

```text
('a', 'b', 'c')
(1, 2, 3)
```

Like `list()`, `tuple()` consumes an iterable.

Difference:

```text
list(iterable)  -> mutable sequence
tuple(iterable) -> immutable sequence
```

The values are the same references.

The outer container has different mutability.

---

# Tuples Store References

Tuples store references to objects.

Example:

```python
name = "Ada"
age = 36

person = (name, age)
```

Conceptually:

```text
person ─────▶ tuple object
              ├── index 0 ─▶ str object "Ada"
              └── index 1 ─▶ int object 36
```

The tuple does not store variables.

It stores references to the objects that the expressions evaluated to.

Example:

```python
name = "Ada"
person = (name,)
name = "Grace"

print(person)
```

Output:

```text
('Ada',)
```

Rebinding `name` does not change the tuple.

The tuple still refers to the original string object.

---

# Indexing Tuples

Tuples support indexing just like lists.

Example:

```python
point = (10, 20, 30)

print(point[0])
print(point[1])
print(point[2])
```

Output:

```text
10
20
30
```

Indexes start at `0`.

Negative indexes count from the end:

```python
print(point[-1])
print(point[-2])
```

Output:

```text
30
20
```

Tuple indexing retrieves a reference at a position.

It does not mutate the tuple.

---

# Slicing Tuples

Tuples support slicing.

Example:

```python
values = (10, 20, 30, 40, 50)

print(values[1:4])
print(values[:3])
print(values[2:])
print(values[::-1])
```

Output:

```text
(20, 30, 40)
(10, 20, 30)
(30, 40, 50)
(50, 40, 30, 20, 10)
```

Slicing a tuple creates a new tuple.

Like list slicing, tuple slicing is shallow.

The outer tuple is new.

The contained object references are reused.

---

# Tuple Immutability

Tuples are immutable.

This means you cannot change which objects the tuple indexes refer to.

Example:

```python
point = (10, 20)
point[0] = 99
```

Python raises:

```text
TypeError: 'tuple' object does not support item assignment
```

You also cannot append to a tuple:

```python
point.append(30)
```

Python raises:

```text
AttributeError
```

Tuples do not have mutating methods such as:

* `append`
* `extend`
* `insert`
* `remove`
* `pop`
* `clear`
* `sort`
* `reverse`

The tuple's length and item references are fixed after creation.

---

# Immutability Is Shallow

Tuple immutability protects the tuple's slots.

It does not automatically make contained objects immutable.

Example:

```python
items = ([1, 2], [3, 4])

items[0].append(99)

print(items)
```

Output:

```text
([1, 2, 99], [3, 4])
```

This is allowed.

Why?

The tuple still refers to the same two list objects.

The tuple's references did not change.

The first list object was mutated.

Conceptually:

```text
tuple slot 0 ─▶ list object
                 └── list contents changed
```

Tuple immutability means:

```text
the tuple cannot be changed
```

It does not mean:

```text
all objects inside the tuple cannot be changed
```

---

# Tuple Rebinding vs Tuple Mutation

You cannot mutate a tuple, but you can rebind a name to a new tuple.

Example:

```python
point = (10, 20)
point = (10, 20, 30)

print(point)
```

Output:

```text
(10, 20, 30)
```

This did not change the original tuple.

It changed what the name `point` refers to.

Mental model:

```text
before:
point ─▶ tuple object (10, 20)

after:
point ─▶ different tuple object (10, 20, 30)
```

Rebinding a name is not the same as mutating an object.

This distinction is central to Python.

---

# Tuple Concatenation

The `+` operator creates a new tuple.

Example:

```python
a = (1, 2)
b = (3, 4)
c = a + b

print(c)
print(a)
print(b)
```

Output:

```text
(1, 2, 3, 4)
(1, 2)
(3, 4)
```

The original tuples are unchanged.

Tuple concatenation does not mutate because tuples cannot mutate.

It creates a new tuple containing references from both operands.

---

# Tuple Repetition

The `*` operator repeats tuple references.

Example:

```python
values = ("x",) * 3
print(values)
```

Output:

```text
('x', 'x', 'x')
```

This is safe with immutable values.

With mutable contained values:

```python
items = ([],) * 3
items[0].append("changed")

print(items)
```

Output:

```text
(['changed'], ['changed'], ['changed'])
```

All three tuple slots refer to the same list object.

The tuple is immutable.

The list inside it is mutable.

This is the same reference-sharing issue seen with list repetition.

---

# Tuple Packing

Tuple packing means grouping multiple values into one tuple.

Example:

```python
point = 10, 20

print(point)
```

Output:

```text
(10, 20)
```

Python packed the two values into a tuple.

Parentheses are optional here:

```python
point = (10, 20)
```

Both create the same kind of tuple.

Packing is common when returning multiple values from a function.

---

# Tuple Unpacking

Tuple unpacking assigns tuple items to names.

Example:

```python
point = (10, 20)
x, y = point

print(x)
print(y)
```

Output:

```text
10
20
```

Conceptually:

```text
x -> point[0]
y -> point[1]
```

The number of names must match the number of values:

```python
x, y = (10, 20, 30)
```

raises:

```text
ValueError
```

Unpacking works with other iterables too, but tuples are the classic example.

---

# Extended Unpacking

Extended unpacking uses `*` to collect extra values.

Example:

```python
values = (1, 2, 3, 4, 5)

first, *middle, last = values

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

Important detail:

The starred target receives a list, not a tuple.

```python
print(type(middle))
```

Output:

```text
<class 'list'>
```

This is because the number of collected values is variable.

Python uses a list for the collected portion.

---

# Swapping Names

Tuple packing and unpacking make swapping easy.

Example:

```python
a = 10
b = 20

a, b = b, a

print(a)
print(b)
```

Output:

```text
20
10
```

The right side is evaluated first:

```python
b, a
```

This creates values to unpack.

Then the left side names are rebound.

This works because assignment evaluation order was already covered in expressions.

---

# Returning Multiple Values

Python functions can appear to return multiple values.

Example:

```python
def divide_with_remainder(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder

result = divide_with_remainder(17, 5)
print(result)
```

Output:

```text
(3, 2)
```

The function returned one object: a tuple.

You can unpack it:

```python
quotient, remainder = divide_with_remainder(17, 5)

print(quotient)
print(remainder)
```

Output:

```text
3
2
```

Mental model:

```text
return a, b
```

means:

```text
return tuple containing a and b
```

---

# Tuple Records

Tuples can represent small fixed records.

Example:

```python
user = ("Ada", "ada@example.com", True)
```

Access:

```python
name = user[0]
email = user[1]
is_active = user[2]
```

This works, but it can become unclear.

What does `user[2]` mean?

A tuple is best for records when:

* The record is small.
* The meaning of positions is obvious.
* The tuple is unpacked near where it is created.

Example:

```python
name, email, is_active = user
```

For larger records, consider:

* dictionaries
* dataclasses
* named tuples
* custom classes

Those come later.

---

# Tuple Equality

Tuples compare by their contents.

Example:

```python
print((1, 2) == (1, 2))
print((1, 2) == (2, 1))
```

Output:

```text
True
False
```

Order matters.

Length matters.

Values are compared element by element.

Example:

```python
print((1, 2, 3) == (1, 2))
```

Output:

```text
False
```

Equality does not require identity:

```python
a = (1, 2)
b = (1, 2)

print(a == b)
print(a is b)
```

The identity result is an implementation detail in some cases.

The important distinction remains:

```text
== asks same value?
is asks same object?
```

---

# Tuple Ordering Comparisons

Tuples can be compared lexicographically.

Example:

```python
print((1, 2) < (1, 3))
print((1, 5) < (2, 0))
```

Output:

```text
True
True
```

Python compares element by element:

```text
(1, 2) < (1, 3)
 first elements equal
 compare second elements: 2 < 3
```

This is useful for sorting tuples:

```python
points = [(2, 3), (1, 9), (1, 5)]
print(sorted(points))
```

Output:

```text
[(1, 5), (1, 9), (2, 3)]
```

The first item is primary.

The second item breaks ties.

---

# Tuple Hashing

Tuples can be hashable if all their elements are hashable.

Example:

```python
point = (10, 20)
locations = {point: "home"}

print(locations[(10, 20)])
```

Output:

```text
home
```

This works because integers are hashable and the tuple contains only hashable values.

This fails:

```python
key = ([1, 2], 3)
data = {key: "value"}
```

Python raises:

```text
TypeError
```

Why?

The tuple contains a list.

Lists are mutable and unhashable.

The tuple itself is immutable, but it cannot be hashable if it contains an unhashable object.

Practical rule:

```text
tuple hashability depends on element hashability
```

---

# Tuples As Dictionary Keys

Tuples are commonly used as dictionary keys when a key has multiple parts.

Example:

```python
distances = {
    ("New York", "Boston"): 215,
    ("New York", "Chicago"): 790,
}

print(distances[("New York", "Boston")])
```

Output:

```text
215
```

The tuple key represents a compound identity:

```text
from city + to city
```

Other examples:

```python
cache[(user_id, permission)]
grid[(row, column)]
visited[(x, y)]
```

Dictionaries are covered in detail soon.

For now, remember that tuples can represent fixed multi-part keys when their elements are hashable.

---

# Tuples In Function Parameters

You can unpack tuples passed to functions manually.

Example:

```python
def distance_from_origin(point):
    x, y = point
    return (x ** 2 + y ** 2) ** 0.5

print(distance_from_origin((3, 4)))
```

Output:

```text
5.0
```

The parameter `point` receives one tuple object.

Inside the function, it is unpacked into `x` and `y`.

This is clearer than using indexes repeatedly:

```python
return (point[0] ** 2 + point[1] ** 2) ** 0.5
```

Unpacking gives names to positions.

---

# Tuples And `*args`

When a function accepts `*args`, the extra positional arguments are collected into a tuple.

Example:

```python
def show_args(*args):
    print(args)
    print(type(args))

show_args(1, 2, 3)
```

Output:

```text
(1, 2, 3)
<class 'tuple'>
```

This is important:

```text
*args inside a function is a tuple
```

Why a tuple?

The arguments are a fixed ordered group for that call.

The function can iterate over them, but the collection itself should not need to grow or shrink.

---

# Tuples In Loops

Tuples are iterable.

Example:

```python
point = (10, 20, 30)

for value in point:
    print(value)
```

Output:

```text
10
20
30
```

Tuples are often unpacked during loops.

Example:

```python
pairs = [("Ada", 36), ("Grace", 85)]

for name, age in pairs:
    print(name, age)
```

Output:

```text
Ada 36
Grace 85
```

Each item in `pairs` is a tuple.

The loop unpacks each tuple into `name` and `age`.

This pattern is common with dictionary `.items()`:

```python
for key, value in mapping.items():
    ...
```

---

# Tuple Methods

Tuples have very few methods.

Main methods:

* `count`
* `index`

Example:

```python
values = ("a", "b", "a")

print(values.count("a"))
print(values.index("b"))
```

Output:

```text
2
1
```

Why so few methods?

Most list methods mutate:

* `append`
* `extend`
* `insert`
* `remove`
* `pop`
* `clear`
* `sort`
* `reverse`

Tuples are immutable, so mutating methods do not exist.

---

# Tuple Memory And Performance Intuition

Tuples are often slightly lighter than lists because their size is fixed.

A tuple does not need extra capacity for future appends.

A list may over-allocate space to make appending efficient.

Practical intuition:

```text
list  -> mutable dynamic sequence
tuple -> immutable fixed sequence
```

Do not choose tuples only for micro-performance.

Choose tuples primarily for meaning:

```text
this fixed group of values belongs together
```

Performance is secondary to clarity.

---

# Tuple vs List

Use a list when:

* The collection may grow.
* The collection may shrink.
* Items may be replaced.
* You need mutating methods.
* The collection represents many similar items.

Examples:

```python
errors = []
users = [user1, user2, user3]
tasks = []
```

Use a tuple when:

* The group has fixed length.
* Position has meaning.
* You do not intend to mutate the group.
* The tuple may be used as a dictionary key.
* The function returns a small fixed set of values.

Examples:

```python
point = (x, y)
rgb = (red, green, blue)
quotient, remainder = divmod(a, b)
```

Rule of thumb:

```text
list -> variable collection
tuple -> fixed record or fixed sequence
```

---

# Named Alternatives

Tuples can become unclear when they have many fields.

Example:

```python
user = ("Ada", "ada@example.com", True, "admin", 36)
```

What does `user[3]` mean?

For larger records, named structures are better.

Options:

```text
dict
namedtuple
dataclass
class
```

Example with a dictionary:

```python
user = {
    "name": "Ada",
    "email": "ada@example.com",
    "is_active": True,
}
```

This is more verbose, but the meaning is explicit.

Dataclasses and classes come later.

For now:

Use tuples for small fixed structures where positions are obvious.

---

# Common Mistakes

## Misconception 1

### Parentheses create a tuple.

The comma creates the tuple.

```python
(10)   # int
(10,)  # tuple
```

## Misconception 2

### Tuples are deeply immutable.

Tuple slots are immutable.

Mutable objects inside a tuple can still mutate.

## Misconception 3

### You can append to a tuple because it is a sequence.

Tuples are sequences, but immutable sequences.

They do not have `append()`.

## Misconception 4

### Returning multiple values returns several separate objects.

A function returns one object.

`return a, b` returns one tuple containing two references.

## Misconception 5

### A tuple containing a list can be used as a dictionary key.

No.

A tuple is hashable only if all its elements are hashable.

Lists are unhashable.

## Misconception 6

### Tuples are always better than lists because they are faster.

The main decision should be meaning and mutability.

Use a tuple for fixed structure.

Use a list for mutable collections.

## Misconception 7

### Tuple unpacking copies the contained objects.

Unpacking binds names to the objects referenced by the tuple.

It does not deep-copy the objects.

---

# Real-world Usage

## Coordinates

```python
point = (10, 20)
x, y = point
```

## Multiple Return Values

```python
def min_max(values):
    return min(values), max(values)

low, high = min_max([4, 1, 9])
```

## Dictionary Items

```python
settings = {"debug": True, "theme": "dark"}

for key, value in settings.items():
    print(key, value)
```

Each item produced by `.items()` behaves like a key-value pair.

## Compound Keys

```python
cache_key = (user_id, resource_id)
cache[cache_key] = result
```

## Sorting Records

```python
records = [("Ada", 36), ("Grace", 85), ("Linus", 28)]
print(sorted(records))
```

Tuples sort lexicographically.

## Fixed Configuration

```python
ALLOWED_EXTENSIONS = (".py", ".md", ".txt")
```

This communicates fixed data better than a list when mutation is not expected.

---

# Internal Mechanics

At a high level, a tuple is a fixed-size sequence of object references.

Conceptually:

```text
tuple object
    |
    v
fixed slots containing object references
```

Unlike a list, a tuple does not need capacity for future growth.

After creation:

```text
number of slots is fixed
references in slots cannot be replaced
```

But the referenced objects still have their own behavior.

If a tuple refers to a list:

```text
tuple slot -> list object
```

the tuple slot cannot point somewhere else, but the list object can mutate.

This is why tuple immutability is shallow.

---

# Concept Connections

Tuples connect to earlier chapters:

* Objects: tuples are objects.
* Names and references: tuple names refer to tuple objects.
* Mutability: tuples are immutable containers.
* Identity and equality: equal tuples do not have to be identical.
* Lists: tuples share sequence behavior but not mutability.
* Functions: functions often return tuples.
* Scope: unpacked names become local names inside functions.
* Functional programming: tuples often carry fixed values through transformations.

Tuples prepare you for:

* Dictionaries.
* Dictionary keys.
* Sets and hashability.
* Named tuples.
* Dataclasses.
* Records and structured data.
* Function APIs that return multiple values.

---

# Active Recall

## Easy Recall Questions

1. What is a tuple?
2. Are tuples mutable or immutable?
3. What creates a one-item tuple?
4. What does tuple unpacking do?
5. What does `return a, b` return?
6. Can a tuple contain a list?
7. Can a tuple containing a list be used as a dictionary key?
8. What methods do tuples commonly have?
9. When should you choose a tuple instead of a list?
10. What does `*args` become inside a function?

## Deep Understanding Questions

1. Why does `(10)` not create a tuple?
2. Why is tuple immutability shallow?
3. Why does rebinding a tuple name not mutate the tuple?
4. Why are tuples useful for multiple return values?
5. Why can tuples sometimes be dictionary keys?
6. Why can position-based records become unclear?
7. How does tuple unpacking connect to names and references?

## Predict-the-Output Questions

### Question 1

```python
value = (10)
print(type(value))
```

### Question 2

```python
value = (10,)
print(type(value))
```

### Question 3

```python
point = (10, 20)
x, y = point
print(x)
print(y)
```

### Question 4

```python
items = ([1, 2], [3, 4])
items[0].append(99)
print(items)
```

### Question 5

```python
def f():
    return "a", "b"

result = f()
print(result)
print(type(result))
```

### Question 6

```python
values = (1, 2, 3, 4)
first, *rest = values

print(first)
print(rest)
print(type(rest))
```

### Question 7

```python
a = (1, 2)
b = (1, 2)

print(a == b)
print(a is b)
```

Explain which output should be relied on and which should not.

---

# Practical Exercises

## Exercise 1

Create a tuple representing a 2D point.

Unpack it into `x` and `y`.

Print both values.

## Exercise 2

Create a one-item tuple correctly.

Then create a parenthesized integer expression.

Print both types.

## Exercise 3

Write a function that returns both the minimum and maximum value from a list.

Unpack the result into two names.

## Exercise 4

Create a tuple containing a list.

Mutate the list.

Explain why the tuple allowed this.

## Exercise 5

Create a tuple that can be used as a dictionary key.

Then create one that cannot.

Explain the difference.

## Exercise 6

Use tuple unpacking in a loop over a list of pairs.

Example:

```python
pairs = [("Ada", 36), ("Grace", 85)]
```

## Exercise 7

Use extended unpacking:

```python
first, *middle, last = values
```

Show the type of `middle`.

## Exercise 8

Write a function that accepts a point tuple and returns its distance from the origin.

Unpack the tuple inside the function.

## Exercise 9

Sort a list of tuple records.

Observe how Python sorts by first item, then second item.

## Exercise 10

Choose whether each should be a list or tuple:

* A shopping cart.
* An RGB color.
* A list of validation errors.
* A latitude-longitude pair.
* A batch of tasks.
* A fixed set of file extensions.

Explain your choices.

---

# Summary

In this chapter we learned:

* Tuples are ordered immutable sequences.
* Tuples store references to objects.
* The comma creates a tuple.
* `(10)` is not a tuple, but `(10,)` is.
* Tuple indexing and slicing work like list indexing and slicing.
* Tuple slicing creates a new tuple.
* Tuple immutability is shallow.
* A tuple can contain mutable objects.
* Tuple packing groups values into a tuple.
* Tuple unpacking binds names to tuple items.
* Functions can return tuples to provide multiple values.
* `*args` is collected as a tuple.
* Tuples can represent small fixed records.
* Tuple equality compares contents.
* Tuple ordering comparisons are lexicographic.
* Tuples are hashable only if all elements are hashable.
* Tuples can be useful dictionary keys.
* Lists are better for mutable collections.
* Tuples are better for fixed groups of related values.

Core model:

```text
name ─────▶ tuple object
             ├── fixed slot 0 ─▶ object
             ├── fixed slot 1 ─▶ object
             └── fixed slot 2 ─▶ object
```

Tuples are fixed containers of references.

The tuple cannot change, but the referenced objects may or may not be mutable.

---

# Preview of Chapter 28

Next we study dictionaries.

Lists and tuples are sequence types.

They organize objects by position:

```text
index -> value
```

Dictionaries organize objects by key:

```text
key -> value
```

We will study:

* What dictionaries are.
* Why key-value mapping exists.
* Dictionary literals.
* Key lookup.
* Adding, updating, and deleting entries.
* Hashability.
* Why immutable keys matter.
* Dictionary iteration.
* Common dictionary methods.
* Nested dictionaries.
* Real-world mapping patterns.
* Complexity intuition.

The transition is direct:

```text
list/tuple -> position-based access
dictionary -> meaning-based key access
```

Dictionaries are one of Python's most important data structures because they model relationships between names, keys, identifiers, and values.
