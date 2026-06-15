# Chapter 58 — Iterators

---

# Learning Objectives

By the end of this chapter, you should understand:

* What iteration means in Python.
* The difference between an iterable and an iterator.
* How `iter()` works.
* How `next()` works.
* Why `StopIteration` ends an iterator.
* How `for` loops use the iteration protocol.
* Why containers usually return fresh iterators.
* Why iterators are usually one-shot objects.
* How to write an iterator class.
* How to write an iterable container class.
* Why `__iter__` should return `self` for iterators.
* Why `__iter__` should usually return a new iterator for containers.
* How iteration supports lazy processing.
* How membership, unpacking, comprehensions, and built-ins rely on iteration.
* How files are iterators over lines.
* How dictionary iteration works.
* How to avoid common iterator bugs.
* When to implement an iterator manually.
* When a generator is better.

Chapter 57 completed the Python data model section.

Now we begin Part III: Pythonic Abstractions.

The first abstraction is iteration.

Iteration is everywhere in Python.

When you write:

```python
for item in items:
    print(item)
```

you are using the iteration protocol.

When you write:

```python
[item.upper() for item in names]
```

you are using the iteration protocol.

When you write:

```python
sum(numbers)
```

you are using the iteration protocol.

When you write:

```python
first, second = pair
```

you are using the iteration protocol.

Iteration is not only about loops.

It is a common language interface for consuming values one at a time.

---

# The Core Idea

Iteration means:

```text
produce values one at a time
```

An object is iterable if Python can ask it for an iterator.

An iterator is an object that produces the next value on demand.

The two central built-in functions are:

```python
iter()
next()
```

Example:

```python
numbers = [10, 20, 30]

iterator = iter(numbers)

print(next(iterator))
print(next(iterator))
print(next(iterator))
```

Output:

```python
10
20
30
```

If you call `next()` again:

```python
next(iterator)
```

Python raises:

```python
StopIteration
```

That exception is not an error in the usual sense.

It is the signal that the iterator is exhausted.

The iteration protocol is:

```text
iterable -> iter(iterable) -> iterator
iterator -> next(iterator) -> next value or StopIteration
```

---

# Iterable Versus Iterator

This distinction is essential.

An iterable is an object that can provide an iterator.

Examples:

```python
list
tuple
str
dict
set
range
file objects
many custom containers
```

An iterator is an object that remembers its current position and returns the next item.

Example:

```python
numbers = [10, 20, 30]
iterator = iter(numbers)
```

`numbers` is an iterable.

`iterator` is an iterator.

Check:

```python
print(iter(numbers) is numbers)
print(iter(iterator) is iterator)
```

For a list:

```python
iter(numbers) is numbers
```

is usually:

```python
False
```

because a list is not its own iterator.

For an iterator:

```python
iter(iterator) is iterator
```

is:

```python
True
```

This is the rule:

```text
iterables produce iterators
iterators produce themselves from iter()
```

---

# Why the Distinction Matters

A list can be looped over many times:

```python
numbers = [1, 2, 3]

for number in numbers:
    print(number)

for number in numbers:
    print(number)
```

Both loops print all values.

Why?

Each `for` loop asks the list for a fresh iterator.

But an iterator is consumed:

```python
numbers = [1, 2, 3]
iterator = iter(numbers)

for number in iterator:
    print(number)

for number in iterator:
    print(number)
```

The second loop prints nothing.

The iterator was already exhausted.

This is one of the most common iterator surprises.

Containers are reusable.

Iterators are usually one-shot.

---

# The Iterator Protocol

An iterator must implement two methods:

```python
__iter__
__next__
```

`__iter__` returns the iterator object.

For an iterator, that usually means:

```python
return self
```

`__next__` returns the next value or raises `StopIteration`.

Example:

```python
class CountUpTo:
    def __init__(self, limit):
        self.current = 1
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.limit:
            raise StopIteration

        value = self.current
        self.current += 1
        return value
```

Use:

```python
counter = CountUpTo(3)

print(next(counter))
print(next(counter))
print(next(counter))
```

Output:

```python
1
2
3
```

Next call:

```python
next(counter)
```

raises `StopIteration`.

Because `CountUpTo` implements `__iter__`, it also works in a `for` loop:

```python
for number in CountUpTo(3):
    print(number)
```

---

# What a `for` Loop Really Does

This:

```python
for item in iterable:
    body(item)
```

is roughly:

```python
iterator = iter(iterable)

while True:
    try:
        item = next(iterator)
    except StopIteration:
        break

    body(item)
```

The real implementation is in Python's interpreter, but the behavior is the same.

This explains why `for` loops work with:

* lists
* tuples
* strings
* dictionaries
* sets
* ranges
* files
* generators
* custom iterables

The loop does not need to know the object's concrete type.

It only needs the iteration protocol.

This is duck typing again:

```text
if it can provide an iterator, the for loop can consume it
```

---

# `StopIteration`

`StopIteration` ends an iterator.

Example:

```python
iterator = iter([1])

print(next(iterator))
print(next(iterator))
```

The first call returns:

```python
1
```

The second call raises:

```python
StopIteration
```

`for` loops catch `StopIteration` internally.

That is why this does not show an exception:

```python
for item in [1]:
    print(item)
```

The loop stops quietly.

You should raise `StopIteration` only from iterator `__next__` methods or related low-level iterator code.

Do not use `StopIteration` as a general control-flow exception in ordinary application logic.

It has a specific protocol meaning:

```text
this iterator has no more values
```

---

# `next()` with a Default

The `next()` built-in can accept a default value:

```python
next(iterator, default)
```

If the iterator is exhausted, Python returns the default instead of raising `StopIteration`.

Example:

```python
iterator = iter([])

value = next(iterator, None)

print(value)
```

Output:

```python
None
```

This is useful when you want the first item if it exists:

```python
first = next(iter(items), None)
```

But be careful if `None` is a valid item.

In that case, use a unique sentinel:

```python
missing = object()
first = next(iter(items), missing)

if first is missing:
    print("no items")
```

This pattern avoids confusing a real `None` value with "no value".

---

# Containers Should Usually Return Fresh Iterators

A container stores values.

Examples:

* list
* tuple
* set
* dict
* custom collection

A container should usually return a new iterator each time `iter(container)` is called.

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)
```

Now:

```python
team = Team(["Asha", "Maya"])

for member in team:
    print(member)

for member in team:
    print(member)
```

Both loops work.

Each loop receives a fresh list iterator.

This is the normal container design.

The container is reusable.

The iterator is disposable.

---

# Iterators Are Usually Their Own Iterators

An iterator should return itself from `__iter__`.

Example:

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value
```

Now:

```python
countdown = Countdown(3)
iter(countdown) is countdown
```

is:

```python
True
```

This matters because `for` loops call `iter()` before consuming.

If an object is already an iterator, `iter()` should not reset it.

It should return the same iterator at its current position.

Do not make an iterator's `__iter__` reset itself unless you have a very specific and well-documented reason.

Resetting during `iter()` breaks normal iterator expectations.

---

# Iterator State

An iterator remembers where it is.

Example:

```python
iterator = iter(["a", "b", "c"])

print(next(iterator))
print(next(iterator))
```

The iterator remembers that `"a"` and `"b"` have already been produced.

The next call returns:

```python
"c"
```

This state can live in attributes:

```python
class CountUpTo:
    def __init__(self, limit):
        self.limit = limit
        self.current = 1
```

The state can also live inside generator objects, file objects, database cursors, or C-level iterator structures.

The important idea is:

```text
iteration is not just a collection
it is a process of producing values over time
```

This is why iterators are powerful for streams.

They do not need all values to exist at once.

---

# Lazy Iteration

Lists store all their elements.

Iterators can produce values lazily.

Example:

```python
class Squares:
    def __init__(self, limit):
        self.current = 0
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration
        value = self.current ** 2
        self.current += 1
        return value
```

This does not store every square.

It computes the next square when requested.

Usage:

```python
for square in Squares(5):
    print(square)
```

Output:

```python
0
1
4
9
16
```

Lazy iteration is useful when:

* data is large
* data is expensive to compute
* values arrive over time
* only some values may be needed
* memory should stay low

This idea leads naturally to generators in Chapter 59.

---

# `range` Is Iterable and Lazy-Like

`range` is a built-in iterable.

Example:

```python
numbers = range(1_000_000)
```

This does not create a list with one million integers.

It creates a range object that can produce values when iterated.

Use:

```python
for number in range(3):
    print(number)
```

Output:

```python
0
1
2
```

`range` is reusable:

```python
values = range(3)

list(values)
list(values)
```

Both produce:

```python
[0, 1, 2]
```

`range` is an iterable, not a one-shot iterator.

Each call to `iter(range_object)` returns an iterator over the range.

---

# Files Are Iterators

File objects are iterable.

They produce lines.

Example:

```python
with open("data.txt") as file:
    for line in file:
        print(line.rstrip())
```

This reads one line at a time.

It does not need to load the entire file into memory.

That is the power of iteration.

Contrast:

```python
lines = file.readlines()
```

This loads all lines into a list.

Sometimes that is fine.

For large files, line-by-line iteration is better.

A file object is also its own iterator in practice:

```python
iter(file) is file
```

This means once you consume lines, the file position advances.

If you loop again, you continue from the current position unless you seek back.

---

# Dictionary Iteration

Dictionaries are iterable.

By default, iterating over a dictionary yields keys:

```python
user = {
    "name": "Maya",
    "email": "maya@example.com",
}

for key in user:
    print(key)
```

Output:

```text
name
email
```

To iterate over values:

```python
for value in user.values():
    print(value)
```

To iterate over key-value pairs:

```python
for key, value in user.items():
    print(key, value)
```

Dictionary views are iterable:

```python
user.keys()
user.values()
user.items()
```

They reflect the dictionary's current contents.

This is an important difference from building a separate list.

---

# Sets Are Iterable but Unordered by Meaning

Sets are iterable:

```python
colors = {"red", "green", "blue"}

for color in colors:
    print(color)
```

But a set is not a sequence.

It has no meaningful position order.

Do not write code that depends on set iteration order for domain meaning.

If you need sorted output:

```python
for color in sorted(colors):
    print(color)
```

Iteration means:

```text
the object can produce values one at a time
```

It does not always mean:

```text
the object has a meaningful order
```

Lists and tuples are ordered.

Sets are not ordered by concept.

Dictionaries preserve insertion order in modern Python, but their default iteration still means keys, not values.

---

# Strings Are Iterable

Strings are iterable over characters:

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

This is why:

```python
list("abc")
```

returns:

```python
['a', 'b', 'c']
```

This can surprise you when a function expects an iterable of words but receives a single string.

Example:

```python
def print_items(items):
    for item in items:
        print(item)
```

Calling:

```python
print_items("hello")
```

prints characters.

If strings should be rejected, check explicitly:

```python
def print_items(items):
    if isinstance(items, str):
        raise TypeError("items must be an iterable of items, not a string")
    for item in items:
        print(item)
```

Strings are sequences.

They are iterable.

That is useful, but sometimes too permissive.

---

# Unpacking Uses Iteration

This:

```python
first, second = [10, 20]
```

uses iteration.

Python consumes values from the right-hand side.

This also works:

```python
first, second, third = range(3)
```

And:

```python
name, email = ("Maya", "maya@example.com")
```

If there are too many or too few values, Python raises `ValueError`.

Example:

```python
first, second = [1, 2, 3]
```

raises because there are too many values.

Extended unpacking also uses iteration:

```python
first, *middle, last = range(5)
```

Result:

```python
first = 0
middle = [1, 2, 3]
last = 4
```

Unpacking is not tuple-only.

It works with iterables.

---

# Comprehensions Use Iteration

List comprehensions consume iterables:

```python
squares = [number * number for number in range(5)]
```

Set comprehensions:

```python
unique_lengths = {len(word) for word in words}
```

Dictionary comprehensions:

```python
by_id = {user.id: user for user in users}
```

Generator expressions:

```python
total = sum(number * number for number in range(5))
```

All of these rely on iteration.

The loop part:

```python
for number in range(5)
```

uses the same protocol as a normal `for` loop.

This is why custom iterables automatically work in comprehensions.

If your object can be iterated, it fits many Python features at once.

---

# Built-In Functions Consume Iterables

Many built-ins accept iterables:

```python
sum(numbers)
min(numbers)
max(numbers)
any(flags)
all(flags)
list(items)
tuple(items)
set(items)
sorted(items)
enumerate(items)
zip(a, b)
map(function, items)
filter(function, items)
```

This is one of Python's great unifying ideas.

You do not need separate APIs for lists, tuples, ranges, files, and custom containers.

If an object is iterable, many tools can consume it.

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)
```

Now:

```python
team = Team(["Asha", "Maya"])

print(list(team))
print(sorted(team))
print("Maya" in team)
```

Iteration makes the object fit into the language.

---

# Membership and Iteration

The `in` operator checks membership.

If an object defines `__contains__`, Python can use it.

If not, Python may fall back to iteration.

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)
```

Now:

```python
"Maya" in Team(["Asha", "Maya"])
```

can work by iterating.

But if membership can be faster or clearer, define `__contains__`:

```python
class Team:
    def __init__(self, members):
        self._members = set(members)

    def __iter__(self):
        return iter(self._members)

    def __contains__(self, member):
        return member in self._members
```

Iteration gives fallback behavior.

`__contains__` gives direct membership behavior.

Use both when both have clear meaning.

---

# `enumerate`

`enumerate` wraps an iterable and produces index-value pairs.

Example:

```python
names = ["Asha", "Maya", "Dev"]

for index, name in enumerate(names):
    print(index, name)
```

Output:

```text
0 Asha
1 Maya
2 Dev
```

You can choose the start:

```python
for line_number, line in enumerate(file, start=1):
    print(line_number, line)
```

`enumerate` returns an iterator.

It does not create all pairs upfront.

It asks the underlying iterable for values one at a time.

This is a common iterator wrapper pattern.

---

# `zip`

`zip` combines multiple iterables.

Example:

```python
names = ["Asha", "Maya"]
scores = [90, 95]

for name, score in zip(names, scores):
    print(name, score)
```

Output:

```text
Asha 90
Maya 95
```

`zip` is lazy.

It returns an iterator.

It stops when the shortest input is exhausted.

Example:

```python
list(zip([1, 2, 3], ["a"]))
```

returns:

```python
[(1, 'a')]
```

Modern Python also supports strict zipping:

```python
zip(a, b, strict=True)
```

This raises an error if the inputs do not have the same length.

Use `strict=True` when mismatched lengths would indicate a bug.

---

# `map` and `filter`

`map` applies a function lazily:

```python
numbers = [1, 2, 3]
squares = map(lambda number: number * number, numbers)
```

`squares` is an iterator.

To see values:

```python
list(squares)
```

Output:

```python
[1, 4, 9]
```

`filter` keeps values that pass a predicate:

```python
numbers = [1, 2, 3, 4]
evens = filter(lambda number: number % 2 == 0, numbers)
```

Then:

```python
list(evens)
```

returns:

```python
[2, 4]
```

Comprehensions are often more readable:

```python
squares = [number * number for number in numbers]
evens = [number for number in numbers if number % 2 == 0]
```

But `map` and `filter` are useful in iterator pipelines, especially when laziness matters.

---

# `reversed`

`reversed()` asks an object for reverse iteration.

For sequences:

```python
for item in reversed([1, 2, 3]):
    print(item)
```

Output:

```text
3
2
1
```

Custom classes can support reverse iteration with:

```python
__reversed__
```

Example:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)

    def __reversed__(self):
        return reversed(self._members)
```

Now:

```python
for member in reversed(team):
    print(member)
```

works.

Only implement `__reversed__` when reverse order has a clear meaning.

---

# Iterator Exhaustion

Once an iterator is exhausted, it should stay exhausted.

Example:

```python
iterator = iter([1])

next(iterator)
next(iterator, None)
next(iterator, None)
```

After the first value, later calls return the default because the iterator remains exhausted.

Custom iterators should follow this expectation.

Bad design:

```python
class BadIterator:
    def __next__(self):
        if finished:
            self.reset()
            raise StopIteration
```

An iterator should not secretly reset after exhaustion.

If reset behavior is needed, make it explicit:

```python
counter.reset()
```

or create a fresh iterator from a container:

```python
iter(container)
```

Iterator exhaustion should be predictable.

---

# A Container and an Iterator as Separate Classes

Sometimes it is useful to separate the iterable container from the iterator.

Example:

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        return CountdownIterator(self.start)


class CountdownIterator:
    def __init__(self, current):
        self.current = current

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value
```

Usage:

```python
countdown = Countdown(3)

print(list(countdown))
print(list(countdown))
```

Both produce:

```python
[3, 2, 1]
```

Why?

`Countdown` is reusable.

Each call to `iter(countdown)` returns a fresh `CountdownIterator`.

This is the right design when an object represents a reusable iterable range or collection.

---

# A Container Returning a Generator

You do not always need a separate iterator class.

`__iter__` can be a generator function:

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        current = self.start
        while current > 0:
            yield current
            current -= 1
```

Usage:

```python
countdown = Countdown(3)
print(list(countdown))
print(list(countdown))
```

Both work.

Each call to `__iter__` creates a new generator object.

Generators are the next chapter.

For now, notice that they are often the simplest way to implement iteration.

Manual iterator classes are useful when you need explicit control.

Generators are usually more concise.

---

# Sequence Fallback: `__getitem__`

Older Python iteration behavior can also use `__getitem__`.

If an object does not define `__iter__`, Python may try integer indexing starting at zero until `IndexError`.

Example:

```python
class OldSequence:
    def __init__(self, values):
        self._values = list(values)

    def __getitem__(self, index):
        return self._values[index]
```

Now:

```python
for value in OldSequence([1, 2, 3]):
    print(value)
```

can work.

But modern custom iterable classes should normally define `__iter__`.

It is clearer.

It directly expresses the protocol.

Use `__getitem__` when the object genuinely supports indexing.

Use `__iter__` when the object supports iteration.

Often a sequence supports both.

---

# Infinite Iterators

An iterator does not have to end.

Example:

```python
class CountForever:
    def __init__(self, start=0):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        value = self.current
        self.current += 1
        return value
```

This iterator never raises `StopIteration`.

Use carefully:

```python
counter = CountForever()

print(next(counter))
print(next(counter))
print(next(counter))
```

But do not write:

```python
list(CountForever())
```

That never finishes.

Infinite iterators are useful with limiting tools:

```python
from itertools import islice

first_five = list(islice(CountForever(), 5))
```

The `itertools` module provides many tools for controlled iterator work.

It will appear later in the book's standard-library coverage.

---

# Iterator Pipelines

Because iterators produce values lazily, they can be chained.

Example:

```python
numbers = range(1_000_000)
squares = (number * number for number in numbers)
evens = (square for square in squares if square % 2 == 0)
first_ten = []

for value in evens:
    first_ten.append(value)
    if len(first_ten) == 10:
        break
```

This does not compute all one million squares.

It computes values as needed.

Iterator pipelines are powerful for:

* log processing
* file processing
* data cleaning
* streaming APIs
* large datasets
* incremental transformations

But long lazy pipelines can be hard to debug.

Keep each step understandable.

Use names for meaningful stages.

---

# Iterating While Mutating

Be careful when mutating a collection while iterating over it.

Example:

```python
items = [1, 2, 3, 4]

for item in items:
    if item % 2 == 0:
        items.remove(item)
```

This can skip values or behave unexpectedly because the list changes while its iterator is moving through it.

Safer:

```python
items = [item for item in items if item % 2 != 0]
```

Or iterate over a copy:

```python
for item in items[:]:
    if item % 2 == 0:
        items.remove(item)
```

Dictionaries are stricter.

Changing dictionary size during iteration can raise an error.

General rule:

```text
do not structurally mutate a collection while iterating over it
```

If you need transformation, build a new collection.

---

# Custom Iteration Order

A class can choose its iteration order.

Example:

```python
class Leaderboard:
    def __init__(self):
        self._scores = {}

    def add(self, name, score):
        self._scores[name] = score

    def __iter__(self):
        return iter(
            sorted(
                self._scores,
                key=self._scores.get,
                reverse=True,
            )
        )
```

Now:

```python
board = Leaderboard()
board.add("Asha", 90)
board.add("Maya", 95)

for name in board:
    print(name)
```

Output:

```text
Maya
Asha
```

This design says:

```text
iterating over a leaderboard yields names by descending score
```

That may be reasonable.

But document non-obvious iteration order.

When users write:

```python
for item in obj:
```

they need to know what `item` means.

---

# What Should Iteration Yield?

For a custom class, decide what iteration means.

Examples:

```python
for user in users:
```

Probably yields user objects.

```python
for key in settings:
```

Probably yields keys, like a dictionary.

```python
for row in table:
```

Probably yields rows.

```python
for node in tree:
```

Maybe yields nodes in depth-first order.

Maybe breadth-first.

Maybe direct children only.

If several meanings are plausible, avoid making `__iter__` too clever.

Provide named methods:

```python
tree.depth_first()
tree.breadth_first()
tree.children()
```

The default iteration should be the most obvious behavior.

If there is no obvious behavior, do not implement `__iter__`.

---

# Multiple Iteration Views

Sometimes an object supports several useful iteration views.

Dictionary example:

```python
dict.keys()
dict.values()
dict.items()
```

Custom example:

```python
class Table:
    def __init__(self, rows):
        self._rows = list(rows)

    def __iter__(self):
        return iter(self._rows)

    def columns(self):
        return zip(*self._rows)
```

Default iteration yields rows.

Named method yields columns.

This is good design.

Use `__iter__` for the primary obvious iteration.

Use named methods for alternative traversals.

---

# Iterators and Memory

Compare:

```python
squares = [number * number for number in range(1_000_000)]
```

with:

```python
squares = (number * number for number in range(1_000_000))
```

The first creates a list immediately.

The second creates a generator iterator that computes values lazily.

If you need all values repeatedly, a list may be right.

If you need to stream once, an iterator is better.

Memory design question:

```text
do I need all values now, or can I consume them one at a time?
```

Lists are concrete.

Iterators are lazy.

Both are useful.

Choose based on the workflow.

---

# Reusing Values from an Iterator

If you need to iterate multiple times, do not keep a one-shot iterator unless you know it can be recreated.

Example:

```python
iterator = (number * number for number in range(3))

print(list(iterator))
print(list(iterator))
```

Output:

```python
[0, 1, 4]
[]
```

The generator iterator is exhausted after the first list conversion.

If you need reuse, store concrete values:

```python
values = list(number * number for number in range(3))

print(list(values))
print(list(values))
```

Or store a factory:

```python
def make_values():
    return (number * number for number in range(3))


print(list(make_values()))
print(list(make_values()))
```

Understand whether you have:

```text
data
or a stream of data
```

That distinction changes design.

---

# `iter(callable, sentinel)`

The `iter()` built-in has a second form:

```python
iter(callable, sentinel)
```

It repeatedly calls the callable until the result equals the sentinel.

Example:

```python
def read_value():
    return input("value: ")


for value in iter(read_value, "quit"):
    print(f"you entered {value}")
```

This keeps calling `read_value()` until it returns `"quit"`.

This form is useful for repeated reads.

Classic file-block example:

```python
with open("data.bin", "rb") as file:
    for block in iter(lambda: file.read(4096), b""):
        process(block)
```

The callable reads a block.

The sentinel is the empty bytes object, which signals end of file.

This is an elegant iterator pattern, but it can be dense.

Use it when the team will recognize it or when it clearly improves the loop.

---

# Iterables in Function Design

Functions should often accept iterables, not just lists.

Less flexible:

```python
def total(values: list[int]) -> int:
    return sum(values)
```

More flexible:

```python
def total(values):
    return sum(values)
```

Now callers can pass:

* list
* tuple
* range
* generator
* file-derived numbers
* custom iterable

Type hints later can express this with `Iterable[int]`.

Design idea:

```text
if you only need to loop once, accept an iterable
if you need indexing, require a sequence
if you need repeated iteration, document it or materialize values
```

Do not require more from callers than you need.

This makes functions more Pythonic and more reusable.

---

# When to Materialize an Iterable

Sometimes you should convert an iterable into a list.

Example:

```python
def average(values):
    values = list(values)
    if not values:
        raise ValueError("average requires at least one value")
    return sum(values) / len(values)
```

Why materialize?

Because this function needs:

* sum of values
* number of values
* empty check

It could compute in one pass without a list:

```python
def average(values):
    total = 0
    count = 0
    for value in values:
        total += value
        count += 1
    if count == 0:
        raise ValueError("average requires at least one value")
    return total / count
```

This version is better for large streams.

Materialize when:

* the input is small enough
* you need repeated passes
* you need indexing or length
* concrete storage simplifies code enough

Avoid materializing when:

* input may be huge
* input may be infinite
* values arrive from a stream
* one pass is enough

---

# Common Mistake: Returning a List Iterator Over Internal State Without Thinking

This is common:

```python
class Team:
    def __iter__(self):
        return iter(self._members)
```

It is often fine.

But it exposes live iteration over internal state.

If the team changes during iteration, behavior may be surprising.

Alternative:

```python
def __iter__(self):
    return iter(tuple(self._members))
```

This returns an iterator over a snapshot.

That costs memory but isolates the iteration.

There is no universal answer.

Ask:

```text
should iteration reflect live state or a snapshot?
```

For most simple collections, live iteration is normal.

For concurrent or mutation-heavy objects, snapshots may be safer.

---

# Common Mistake: Making a Container Its Own Iterator

This is usually bad:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self._index >= len(self._members):
            raise StopIteration
        value = self._members[self._index]
        self._index += 1
        return value
```

This makes the team object itself a one-shot iterator.

First loop works.

Second loop prints nothing.

That is surprising for a container.

Better:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)
```

Only make an object its own iterator when the object represents an iteration process, not a reusable collection.

---

# Common Mistake: Forgetting `StopIteration`

This iterator is broken:

```python
class BrokenCounter:
    def __init__(self):
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        self.current += 1
        return self.current
```

It never stops.

That may be intentional for an infinite iterator.

But if you meant to count to a limit, you must raise `StopIteration`:

```python
class Counter:
    def __init__(self, limit):
        self.current = 0
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration
        self.current += 1
        return self.current
```

Termination is part of iterator design.

If the iterator is infinite, document or make it obvious.

---

# Common Mistake: Catching `StopIteration` Too Broadly

This is awkward:

```python
try:
    for item in items:
        process(item)
except StopIteration:
    pass
```

The `for` loop already handles iterator exhaustion.

You rarely need to catch `StopIteration` around a `for` loop.

Catch it when you manually call `next()`:

```python
iterator = iter(items)

try:
    first = next(iterator)
except StopIteration:
    first = None
```

Or use the default form:

```python
first = next(iter(items), None)
```

Let `for` loops do their job.

---

# Common Mistake: Consuming an Iterator Accidentally

This function logs values:

```python
def process(values):
    print(list(values))
    for value in values:
        handle(value)
```

If `values` is an iterator, `list(values)` consumes it.

The loop then sees nothing.

Safer options:

Materialize once:

```python
def process(values):
    values = list(values)
    print(values)
    for value in values:
        handle(value)
```

Or avoid logging all values:

```python
def process(values):
    for value in values:
        print(value)
        handle(value)
```

Be aware of functions that consume iterables:

```python
list()
tuple()
set()
sum()
any()
all()
max()
min()
sorted()
```

Some consume fully.

Some may stop early.

But after consumption, a one-shot iterator has moved forward.

---

# Common Mistake: Assuming Length

Not every iterable has a length.

This works:

```python
len([1, 2, 3])
```

This fails:

```python
len(iter([1, 2, 3]))
```

Many iterators do not know how many values remain.

This is especially true for:

* files
* streams
* generators
* network data
* infinite iterators

If your function needs `len(values)`, it needs more than an iterable.

It needs a sized collection or sequence.

If it only loops, accept any iterable.

This distinction becomes important in type hints:

```text
Iterable
Sequence
Collection
Iterator
```

The type system chapter will return to these names.

---

# Common Mistake: Assuming Indexing

Not every iterable supports indexing.

This works:

```python
items = [10, 20, 30]
items[0]
```

This fails:

```python
iterator = iter(items)
iterator[0]
```

If you need the first item from an iterable:

```python
first = next(iter(items))
```

If the iterable might be empty:

```python
first = next(iter(items), None)
```

If you need many random accesses, convert to a list:

```python
items = list(items)
```

Again:

```text
iteration is sequential access
indexing is positional random access
```

They are related but not the same.

---

# Common Mistake: Iterating Strings by Accident

Strings are iterable.

This can cause subtle bugs:

```python
def send_emails(addresses):
    for address in addresses:
        send(address)
```

This is fine:

```python
send_emails(["a@example.com", "b@example.com"])
```

This is bad:

```python
send_emails("a@example.com")
```

It iterates characters.

If a single string should be rejected:

```python
def send_emails(addresses):
    if isinstance(addresses, str):
        raise TypeError("addresses must be an iterable of email strings")
    for address in addresses:
        send(address)
```

Or design a separate function:

```python
send_email(address)
send_emails(addresses)
```

Clear APIs prevent accidental string iteration.

---

# Common Mistake: Hidden Infinite Iteration

This is dangerous:

```python
def first_match(values, predicate):
    for value in values:
        if predicate(value):
            return value
```

If no value matches and `values` is infinite, this never returns.

That may be acceptable if documented.

But be aware.

For potentially infinite iterables, design with limits:

```python
from itertools import islice


def first_match(values, predicate, limit):
    for value in islice(values, limit):
        if predicate(value):
            return value
    return None
```

Iteration can represent infinity.

That is powerful.

It also means some operations may never finish.

---

# A Practical Example: Log Lines

Suppose we want to process a log file lazily.

```python
class LogFile:
    def __init__(self, path):
        self.path = path

    def __iter__(self):
        with open(self.path) as file:
            for line in file:
                yield line.rstrip("\n")
```

Usage:

```python
for line in LogFile("app.log"):
    if "ERROR" in line:
        print(line)
```

This opens the file when iteration begins.

It yields one line at a time.

It closes the file when iteration ends normally.

This design is memory efficient.

It also makes `LogFile` reusable:

```python
log = LogFile("app.log")
list(log)
list(log)
```

Each iteration opens the file again.

That may be exactly what you want.

---

# A Practical Example: Paginated API

Imagine an API that returns pages of results.

We can hide pagination behind iteration:

```python
class PaginatedResults:
    def __init__(self, client, first_url):
        self.client = client
        self.first_url = first_url

    def __iter__(self):
        url = self.first_url
        while url is not None:
            response = self.client.get(url)
            for item in response["items"]:
                yield item
            url = response["next"]
```

Usage:

```python
for item in PaginatedResults(client, "/users"):
    process(item)
```

The caller does not manage pages.

The iterable produces items.

This is a strong abstraction when:

* pagination is an implementation detail
* users want items
* lazy fetching is acceptable
* errors are handled clearly

Be careful with hidden network calls.

Iteration may look simple, but this object performs I/O.

Document that behavior.

---

# A Practical Example: Tree Traversal

Trees can have multiple traversal orders.

Define a node:

```python
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = list(children or [])
```

Depth-first traversal:

```python
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = list(children or [])

    def depth_first(self):
        yield self
        for child in self.children:
            yield from child.depth_first()
```

Use:

```python
for node in root.depth_first():
    print(node.value)
```

Should `Node.__iter__` be depth-first?

Maybe.

But if there are multiple natural traversals, named methods may be clearer:

```python
root.depth_first()
root.breadth_first()
root.children
```

Default iteration should not make a surprising choice.

---

# A Practical Example: Batches

Sometimes iteration groups values.

Example:

```python
class Batches:
    def __init__(self, iterable, size):
        self.iterable = iterable
        self.size = size

    def __iter__(self):
        iterator = iter(self.iterable)
        while True:
            batch = []
            try:
                for _ in range(self.size):
                    batch.append(next(iterator))
            except StopIteration:
                if batch:
                    yield batch
                return
            yield batch
```

Usage:

```python
for batch in Batches(range(10), 3):
    print(batch)
```

Output:

```python
[0, 1, 2]
[3, 4, 5]
[6, 7, 8]
[9]
```

This demonstrates a custom iterable wrapper.

It consumes another iterable and yields grouped values.

Chapter 59 will show how generator functions make this kind of code easier to write as a standalone function.

---

# Iterator Design Checklist

When designing iteration for a class, ask:

```text
Is this object naturally iterable?
```

If no, do not implement `__iter__`.

Ask:

```text
What should each yielded item be?
```

If unclear, use named methods.

Ask:

```text
Should this object be reusable?
```

If yes, return a fresh iterator from `__iter__`.

Ask:

```text
Is this object itself an iterator process?
```

If yes, return `self` from `__iter__` and implement `__next__`.

Ask:

```text
Can iteration be infinite?
```

If yes, document it and design limits.

Ask:

```text
Does iteration perform I/O?
```

If yes, make that behavior clear.

Ask:

```text
Will mutation during iteration be a problem?
```

If yes, consider snapshots or documentation.

Ask:

```text
Would a generator be simpler?
```

Often, yes.

---

# Practice: Manual Iterator

Create an iterator that counts down from a number to 1.

Solution:

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value
```

Test:

```python
assert list(Countdown(3)) == [3, 2, 1]
```

Remember:

`Countdown(3)` is a one-shot iterator.

After it is consumed, it is exhausted.

---

# Practice: Reusable Iterable

Create a reusable countdown iterable.

Solution:

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        return CountdownIterator(self.start)


class CountdownIterator:
    def __init__(self, current):
        self.current = current

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value
```

Test:

```python
countdown = Countdown(3)

assert list(countdown) == [3, 2, 1]
assert list(countdown) == [3, 2, 1]
```

This shows the difference between iterable container and iterator.

---

# Practice: Team Iterable

Create a `Team` class that stores members and supports iteration.

Solution:

```python
class Team:
    def __init__(self, members):
        self._members = list(members)

    def __iter__(self):
        return iter(self._members)
```

Test:

```python
team = Team(["Asha", "Maya"])

assert list(team) == ["Asha", "Maya"]
assert list(team) == ["Asha", "Maya"]
```

The second assertion proves the team returns fresh iterators.

---

# Practice: First Item Helper

Write a function that returns the first item from any iterable, or a default if it is empty.

Solution:

```python
def first(iterable, default=None):
    return next(iter(iterable), default)
```

Test:

```python
assert first([1, 2, 3]) == 1
assert first([], "missing") == "missing"
assert first(range(10)) == 0
```

If `None` can be a real value, use a sentinel in your own calling code.

---

# Practice: Avoid Accidental Consumption

What is wrong here?

```python
def debug_and_sum(numbers):
    print(list(numbers))
    return sum(numbers)
```

Answer:

If `numbers` is an iterator, `list(numbers)` consumes it.

Then `sum(numbers)` sees no values.

Fix:

```python
def debug_and_sum(numbers):
    numbers = list(numbers)
    print(numbers)
    return sum(numbers)
```

This is safe for finite inputs that fit in memory.

For large streams, avoid printing all values.

---

# Practice: Choose Iterable or Sequence

For each function, decide what it needs.

```text
sum all values once
get the first item by index
loop through file lines
sort values
calculate average in one pass
calculate median
```

Likely answers:

```text
sum all values once -> iterable
get first item by index -> sequence
loop through file lines -> iterable
sort values -> iterable accepted, but sorted materializes
average in one pass -> iterable
median -> likely sequence/list or materialize iterable
```

The question is:

```text
which operations does the function actually need?
```

Do not require a list when any iterable will do.

Do not accept any iterable if you need indexing and repeated passes unless you materialize it.

---

# Summary

Iteration is one of Python's central protocols.

An iterable is an object that can produce an iterator.

An iterator is an object that produces values one at a time.

`iter(obj)` asks an object for an iterator.

`next(iterator)` asks an iterator for the next value.

When an iterator is exhausted, it raises `StopIteration`.

`for` loops call `iter()` and repeatedly call `next()` until `StopIteration`.

Containers usually return fresh iterators.

Iterators usually return themselves from `__iter__`.

Containers are usually reusable.

Iterators are usually one-shot.

Iteration powers loops, comprehensions, unpacking, membership checks, many built-ins, files, dictionary views, and lazy pipelines.

Lazy iteration lets programs process large or infinite streams without storing every value at once.

Custom classes should implement `__iter__` only when there is a clear meaning for iteration.

Manual iterator classes are useful, but generator functions are often simpler.

The design principle is:

```text
iteration should expose the natural sequence of values an object provides
```

If there are multiple possible sequences, use named methods.

If the object is a reusable collection, return a fresh iterator.

If the object is a one-time stream, make that behavior clear.

---

# Preview of Chapter 59

Chapter 58 taught the iterator protocol directly.

We wrote iterator classes by implementing:

```python
__iter__
__next__
```

That was important because it revealed the machinery.

But many iterators are easier to write with generator functions.

Chapter 59 introduces generators.

Generators let you write lazy iterators with ordinary function syntax and the `yield` keyword.

We will study:

* what `yield` does
* how generator functions create generator objects
* why generators are iterators
* how generator state is suspended and resumed
* how `yield from` works
* how generator expressions differ from list comprehensions
* how to build streaming pipelines
* how generator cleanup works
* common generator mistakes

The transition is:

```text
iterators define the protocol
generators make the protocol pleasant to implement
```

Once generators become comfortable, much of Python's lazy, memory-efficient style becomes natural.

