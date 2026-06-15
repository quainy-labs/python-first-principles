# Chapter 59 — Generators

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a generator is.
* What makes a function a generator function.
* How `yield` differs from `return`.
* Why calling a generator function does not run the function body immediately.
* How generator objects implement the iterator protocol.
* How generator state is suspended and resumed.
* How local variables survive between `yield` points.
* How `StopIteration` relates to generator completion.
* How `return` works inside a generator.
* How generator expressions differ from list comprehensions.
* How `yield from` delegates to another iterable.
* How generators support lazy pipelines.
* How `send`, `throw`, and `close` work at a practical level.
* How cleanup works with `try` and `finally`.
* When generators are better than manual iterator classes.
* When lists are better than generators.
* Which generator mistakes are common.

Chapter 58 taught the iterator protocol directly.

We wrote classes with:

```python
__iter__
__next__
```

That was important.

It showed the machinery.

But writing iterator classes by hand can be verbose.

Many iterators are simpler as generator functions.

Example:

```python
def countdown(start):
    current = start
    while current > 0:
        yield current
        current -= 1
```

Use:

```python
for number in countdown(3):
    print(number)
```

Output:

```text
3
2
1
```

No explicit class.

No explicit `__iter__`.

No explicit `__next__`.

No explicit `StopIteration`.

The `yield` keyword lets Python build the iterator machinery for us.

---

# The Core Idea

A generator is an iterator created by a generator function or generator expression.

A generator function is a function that contains `yield`.

Example:

```python
def numbers():
    yield 1
    yield 2
    yield 3
```

Calling the function does not run the body immediately:

```python
gen = numbers()
```

`gen` is a generator object.

It is an iterator.

Now call:

```python
next(gen)
```

Python starts running the function until it reaches the first `yield`.

That returns:

```python
1
```

Call again:

```python
next(gen)
```

Execution resumes after the first `yield` and continues until the next `yield`.

That returns:

```python
2
```

Call again:

```python
next(gen)
```

returns:

```python
3
```

Call again:

```python
next(gen)
```

The function has no more values, so the generator raises `StopIteration`.

---

# `yield` Suspends, `return` Finishes

In a normal function:

```python
def add(a, b):
    return a + b
```

`return` sends one result back and the function is done.

In a generator function:

```python
def numbers():
    yield 1
    yield 2
```

`yield` sends a value back but pauses the function.

The function's local state is preserved.

Later, when `next()` is called again, the function resumes where it left off.

Think:

```text
return -> finish the function
yield  -> pause the function and produce one value
```

A generator can yield many times.

A normal function returns once.

---

# Calling a Generator Function

This point is so important that it deserves its own section.

Consider:

```python
def example():
    print("starting")
    yield 1
    print("continuing")
    yield 2
    print("ending")
```

Call it:

```python
gen = example()
```

Nothing prints.

The function body has not started.

Now:

```python
next(gen)
```

prints:

```text
starting
```

and returns:

```python
1
```

Now:

```python
next(gen)
```

prints:

```text
continuing
```

and returns:

```python
2
```

Now:

```python
next(gen)
```

prints:

```text
ending
```

and raises `StopIteration`.

Generator functions are lazy.

Calling them creates a generator object.

Iteration runs the body.

---

# Generators Are Iterators

A generator object follows the iterator protocol.

Example:

```python
def numbers():
    yield 1
    yield 2
```

Create:

```python
gen = numbers()
```

Then:

```python
iter(gen) is gen
```

is:

```python
True
```

And:

```python
next(gen)
```

returns values.

This means generator objects can be used anywhere an iterator can be used:

```python
for value in numbers():
    print(value)
```

```python
list(numbers())
```

```python
sum(numbers())
```

```python
first = next(numbers())
```

Generators are not a separate universe.

They are a convenient way to create iterators.

---

# Manual Iterator Versus Generator

Manual iterator from Chapter 58:

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

Generator version:

```python
def countdown(start):
    current = start
    while current > 0:
        yield current
        current -= 1
```

Both can be used:

```python
list(countdown(3))
```

Output:

```python
[3, 2, 1]
```

The generator version is shorter because Python handles:

* iterator object creation
* `__iter__`
* `__next__`
* preserving state
* raising `StopIteration` when the function ends

Manual iterator classes are still useful when:

* the iterator needs many methods
* state management is complex
* object identity matters
* you need a reusable configurable iterator type
* you want explicit class-based behavior

But for many sequential value producers, generators are clearer.

---

# Local State Is Preserved

Generators preserve local variables between yields.

Example:

```python
def counter():
    count = 0
    while count < 3:
        count += 1
        yield count
```

Use:

```python
gen = counter()

print(next(gen))
print(next(gen))
print(next(gen))
```

Output:

```python
1
2
3
```

The variable `count` does not reset each time.

The generator frame is suspended.

When resumed, it continues with the same local variables.

This is the key mental model:

```text
a generator is a paused function with memory
```

That memory is why generators can be elegant.

You write code as a simple loop.

Python turns it into a resumable iterator.

---

# A Generator's Lifecycle

A generator object moves through states.

Simplified:

```text
created -> running -> suspended -> running -> suspended -> finished
```

When created:

```python
gen = numbers()
```

the body has not run.

When `next(gen)` is called, it runs.

At `yield`, it suspends.

When `next(gen)` is called again, it resumes.

When the function ends or returns, it is finished.

After a generator is finished, it stays finished.

Example:

```python
gen = numbers()
list(gen)
list(gen)
```

The second list is empty.

A generator object is a one-shot iterator.

If you want to iterate again, call the generator function again:

```python
list(numbers())
list(numbers())
```

Each call creates a fresh generator object.

---

# `return` Inside a Generator

A generator can use `return` to stop.

Example:

```python
def numbers():
    yield 1
    return
    yield 2
```

Then:

```python
list(numbers())
```

returns:

```python
[1]
```

The `return` ends the generator.

A generator can also return a value:

```python
def numbers():
    yield 1
    return "done"
```

That return value becomes attached to the `StopIteration` exception.

Most ordinary `for` loops ignore it.

It matters mainly when using advanced generator delegation with `yield from`.

For beginner and intermediate generator code, use `return` mostly to stop early.

Do not expect `list(generator)` to include the returned value.

Only yielded values are produced.

---

# `yield` Produces Values

Only `yield` sends values to the consumer.

Example:

```python
def example():
    yield "a"
    return "b"
```

Then:

```python
list(example())
```

is:

```python
['a']
```

The returned `"b"` is not part of the iteration.

This matters when converting normal functions to generators.

If you want a value to appear in the iteration, use:

```python
yield value
```

If you want to stop the generator, use:

```python
return
```

---

# Generator Expressions

A generator expression is like a lazy comprehension.

List comprehension:

```python
squares = [number * number for number in range(5)]
```

This creates a list immediately:

```python
[0, 1, 4, 9, 16]
```

Generator expression:

```python
squares = (number * number for number in range(5))
```

This creates a generator object.

Values are computed when consumed:

```python
next(squares)
next(squares)
```

returns:

```python
0
1
```

To materialize:

```python
list(squares)
```

But remember: the first two values were already consumed.

The list now contains the remaining values:

```python
[4, 9, 16]
```

Generator expressions are one-shot lazy iterators.

---

# Generator Expression Syntax

Basic syntax:

```python
(expression for item in iterable)
```

With filter:

```python
(expression for item in iterable if condition)
```

Example:

```python
even_squares = (
    number * number
    for number in range(10)
    if number % 2 == 0
)
```

Use:

```python
list(even_squares)
```

Output:

```python
[0, 4, 16, 36, 64]
```

When a generator expression is the only argument to a function call, extra parentheses can be omitted:

```python
sum(number * number for number in range(10))
```

This is common and readable.

With multiple arguments, keep parentheses:

```python
max((score for score in scores), default=0)
```

---

# List Comprehension or Generator Expression?

Use a list comprehension when:

* you need all values now
* you need to iterate multiple times
* you need indexing
* the list is small enough
* concrete data improves clarity

Example:

```python
names = [user.name for user in users]
```

Use a generator expression when:

* values can be consumed one at a time
* input may be large
* you are passing directly into a consuming function
* laziness improves memory usage

Example:

```python
total = sum(order.total for order in orders)
```

This avoids building an intermediate list.

Do not use a generator expression just to look clever.

Choose based on whether you need a collection or a stream.

---

# Lazy Evaluation

Generators compute values when requested.

Example:

```python
def noisy_numbers():
    print("start")
    yield 1
    print("middle")
    yield 2
    print("end")
```

Create:

```python
gen = noisy_numbers()
```

Nothing prints.

Now:

```python
next(gen)
```

prints:

```text
start
```

and returns `1`.

Now:

```python
next(gen)
```

prints:

```text
middle
```

and returns `2`.

This laziness is powerful for expensive work.

If the consumer stops early, later work never happens.

Example:

```python
def find_first_even(numbers):
    for number in numbers:
        if number % 2 == 0:
            return number
```

If `numbers` is a generator pipeline, it may compute only what is needed to find the first even value.

---

# Streaming Pipelines

Generators compose naturally.

Example:

```python
def read_lines(path):
    with open(path) as file:
        for line in file:
            yield line.rstrip("\n")
```

Filter:

```python
def only_errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line
```

Transform:

```python
def timestamps(lines):
    for line in lines:
        yield line[:19]
```

Pipeline:

```python
lines = read_lines("app.log")
errors = only_errors(lines)
times = timestamps(errors)

for timestamp in times:
    print(timestamp)
```

Each stage consumes and produces lazily.

The program does not load the whole log file.

It processes one line at a time.

This is one of the most important generator patterns.

---

# Pipeline Design

Generator pipelines work best when each function has one job.

Good:

```python
lines = read_lines(path)
records = parse_json(lines)
valid_records = filter_valid(records)
emails = extract_emails(valid_records)
```

Each stage is understandable.

Less good:

```python
def process(path):
    with open(path) as file:
        for line in file:
            ...
            ...
            ...
            yield complicated_result
```

Long generator functions can become hard to debug.

Name the stages.

Keep transformations small.

Use tests for each generator stage.

The beauty of generators is not only laziness.

It is composability.

---

# `yield from`

`yield from` delegates yielding to another iterable.

Without `yield from`:

```python
def flatten_once(groups):
    for group in groups:
        for item in group:
            yield item
```

With `yield from`:

```python
def flatten_once(groups):
    for group in groups:
        yield from group
```

Use:

```python
groups = [[1, 2], [3, 4]]
list(flatten_once(groups))
```

Output:

```python
[1, 2, 3, 4]
```

`yield from iterable` means:

```text
yield every value produced by this iterable
```

It is clearer than writing an inner loop when delegation is the point.

---

# Recursive Generators with `yield from`

Trees are a good use case.

```python
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = list(children or [])
```

Depth-first traversal:

```python
def depth_first(node):
    yield node
    for child in node.children:
        yield from depth_first(child)
```

Use:

```python
for node in depth_first(root):
    print(node.value)
```

The `yield from` line says:

```text
yield everything from the child's traversal
```

Without it:

```python
yield depth_first(child)
```

would yield the generator object itself, not the child's nodes.

This is a common mistake.

Use `yield from` when you want to delegate a stream of values.

---

# `yield from` and Return Values

Advanced generator delegation can receive a return value from the subgenerator.

Example:

```python
def child():
    yield 1
    return "done"


def parent():
    result = yield from child()
    yield result
```

Now:

```python
list(parent())
```

returns:

```python
[1, 'done']
```

The `return "done"` from `child` becomes the result of the `yield from child()` expression.

This is powerful but less common than simple delegation.

Most everyday uses of `yield from` are for flattening or delegating iteration.

Learn the simple meaning first:

```text
yield from iterable -> yield each value from iterable
```

Then remember there is a deeper generator-delegation protocol underneath.

---

# Generator Cleanup

Generators can contain `try` and `finally`.

Example:

```python
def read_lines(path):
    file = open(path)
    try:
        for line in file:
            yield line.rstrip("\n")
    finally:
        file.close()
```

If the generator is closed before exhaustion, the `finally` block can run.

This matters when a generator owns a resource.

However, a simpler version uses a context manager:

```python
def read_lines(path):
    with open(path) as file:
        for line in file:
            yield line.rstrip("\n")
```

The `with` statement and generator cleanup cooperate.

Still, be careful.

If a generator that owns a resource is created but never consumed or closed, resource lifetime can become unclear.

For resource-heavy code, context managers may be more explicit.

Chapter 60 studies context managers next.

---

# `close()`

Generator objects have a `close()` method.

Example:

```python
def generator():
    try:
        yield 1
        yield 2
    finally:
        print("closing")
```

Use:

```python
gen = generator()
print(next(gen))
gen.close()
```

Output:

```text
1
closing
```

`close()` tells the generator to finish.

This can trigger cleanup.

Most code does not call `close()` manually because `for` loops and context managers usually manage normal consumption.

But if you stop early and the generator holds resources, explicit cleanup can matter.

---

# `send()`

Generators can receive values through `send()`.

This is more advanced.

Example:

```python
def echo():
    received = yield "ready"
    yield f"received {received}"
```

Use:

```python
gen = echo()

print(next(gen))
print(gen.send("hello"))
```

Output:

```text
ready
received hello
```

The first `next(gen)` starts the generator and runs to the first `yield`.

The `send("hello")` resumes the generator.

The `yield "ready"` expression evaluates to `"hello"` inside the generator.

Most everyday generators do not use `send()`.

It appears in coroutine-like patterns, parser pipelines, and advanced control flows.

Async programming later uses different syntax.

For now, understand that a generator can be more than a one-way producer.

But use `send()` sparingly.

---

# Priming a Generator

You cannot send a non-`None` value into a just-created generator before it reaches the first `yield`.

Example:

```python
gen = echo()
gen.send("hello")
```

This raises an error.

The generator must be started first:

```python
gen = echo()
next(gen)
gen.send("hello")
```

Starting a generator so it reaches its first `yield` is sometimes called priming.

In modern Python, many uses that once relied on generator coroutines are better expressed with `async` and `await`.

This chapter mentions `send()` because generator objects support it.

But most generator functions should simply yield values.

---

# `throw()`

Generator objects also have `throw()`.

It raises an exception at the point where the generator is suspended.

Example:

```python
def generator():
    try:
        yield "working"
    except ValueError:
        yield "handled"
```

Use:

```python
gen = generator()
print(next(gen))
print(gen.throw(ValueError))
```

Output:

```text
working
handled
```

This is advanced.

It can be useful in frameworks that need to inject errors into suspended generator logic.

Most application code should not need `throw()`.

If you find yourself using it often, ask whether a clearer control-flow design exists.

---

# Generators and Exceptions

Exceptions inside generators behave like exceptions inside functions.

Example:

```python
def values():
    yield 1
    raise ValueError("bad value")
    yield 2
```

Use:

```python
for value in values():
    print(value)
```

The loop prints:

```python
1
```

Then the `ValueError` propagates.

`for` loops catch `StopIteration`.

They do not catch every exception.

That is good.

`StopIteration` means normal exhaustion.

Other exceptions usually mean something went wrong.

Do not hide real errors by catching all exceptions around generator consumption.

---

# Generators and `StopIteration`

A generator function should not usually raise `StopIteration` directly.

Instead, use:

```python
return
```

or let the function end.

Example:

```python
def numbers(limit):
    for number in range(limit):
        yield number
```

When the loop finishes, the generator stops.

This is enough.

If you need to stop early:

```python
def until_negative(numbers):
    for number in numbers:
        if number < 0:
            return
        yield number
```

Use `return`, not `raise StopIteration`.

Modern Python transforms accidental `StopIteration` escaping from generator code into a runtime error in many cases.

The practical rule:

```text
inside a generator, yield values and return to stop
```

---

# Generator Expressions and Variable Scope

Generator expressions have their own scope for loop variables.

Example:

```python
gen = (number * number for number in range(3))
```

The `number` variable does not leak into the surrounding scope.

This matches list comprehensions in modern Python.

Another subtle point:

The leftmost iterable is evaluated immediately.

The rest is lazy.

Example:

```python
def source():
    print("creating source")
    return [1, 2, 3]


gen = (number * number for number in source())
```

This prints:

```text
creating source
```

when the generator expression is created.

But the squares are computed later when the generator is consumed.

This detail matters when the iterable expression can raise errors or has side effects.

---

# Parentheses in Generator Expressions

These are equivalent:

```python
sum((number * number for number in range(10)))
```

and:

```python
sum(number * number for number in range(10))
```

When the generator expression is the only argument to a function, the outer parentheses can be omitted.

But this needs parentheses:

```python
max((score for score in scores), default=0)
```

because `max` has another argument.

This is a syntax detail, but it affects readability.

When in doubt, keep parentheses.

Clarity beats clever compactness.

---

# Generators and Files

Files already iterate over lines.

Generators let us layer processing on top.

Example:

```python
def non_empty_lines(path):
    with open(path) as file:
        for line in file:
            stripped = line.strip()
            if stripped:
                yield stripped
```

Use:

```python
for line in non_empty_lines("notes.txt"):
    print(line)
```

This reads one line at a time.

It strips whitespace.

It skips empty lines.

It does not build a list.

This is exactly where generators shine:

```text
small transformation over a stream
```

---

# Generators and Backpressure

Generators produce values only when the consumer asks.

This creates a simple form of backpressure.

Example:

```python
def numbers():
    current = 0
    while True:
        print(f"producing {current}")
        yield current
        current += 1
```

Consumer:

```python
gen = numbers()
print(next(gen))
print(next(gen))
```

Only two values are produced.

The producer does not run ahead and generate infinite values.

This makes generators useful for:

* large files
* event streams
* paginated APIs
* data transformations
* infinite sequences

The consumer controls the pace.

---

# Generators and Infinite Sequences

Generators can represent infinite sequences.

Example:

```python
def count_from(start=0):
    current = start
    while True:
        yield current
        current += 1
```

Use safely:

```python
from itertools import islice


first_five = list(islice(count_from(), 5))
```

Output:

```python
[0, 1, 2, 3, 4]
```

Do not materialize an infinite generator:

```python
list(count_from())
```

That never finishes.

Infinite generators are useful, but they require limiting consumers.

---

# A Generator for Batches

Chapter 58 wrote a batch iterable class.

Here is a generator function version:

```python
def batches(iterable, size):
    iterator = iter(iterable)
    while True:
        batch = []
        try:
            for _ in range(size):
                batch.append(next(iterator))
        except StopIteration:
            if batch:
                yield batch
            return
        yield batch
```

Use:

```python
list(batches(range(10), 3))
```

Output:

```python
[[0, 1, 2], [3, 4, 5], [6, 7, 8], [9]]
```

This is easier than a custom iterable class.

But it still needs careful edge-case handling.

What if `size <= 0`?

Add validation:

```python
def batches(iterable, size):
    if size <= 0:
        raise ValueError("batch size must be positive")
    ...
```

Generators reduce boilerplate.

They do not remove design responsibility.

---

# A Generator for Sliding Windows

A sliding window yields overlapping groups.

Example:

```python
def sliding_window(iterable, size):
    if size <= 0:
        raise ValueError("window size must be positive")

    iterator = iter(iterable)
    window = []

    for _ in range(size):
        try:
            window.append(next(iterator))
        except StopIteration:
            return

    yield tuple(window)

    for item in iterator:
        window = window[1:] + [item]
        yield tuple(window)
```

Use:

```python
list(sliding_window([1, 2, 3, 4], 3))
```

Output:

```python
[(1, 2, 3), (2, 3, 4)]
```

This is a classic lazy transformation.

It avoids creating all windows upfront.

For production code, standard-library tools may help in some versions and cases.

But implementing it teaches generator state clearly.

---

# A Generator for Tree Traversal

Depth-first traversal with generators:

```python
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = list(children or [])
```

Traversal:

```python
def depth_first(node):
    yield node
    for child in node.children:
        yield from depth_first(child)
```

Use:

```python
for node in depth_first(root):
    print(node.value)
```

The generator naturally mirrors the recursive structure.

Manual iterator classes for recursive traversal are possible.

They are usually more complex.

Generators let the code describe the traversal directly.

---

# A Generator for Pagination

Generators can hide pagination:

```python
def paginated_items(client, first_url):
    url = first_url

    while url is not None:
        response = client.get(url)
        yield from response["items"]
        url = response["next"]
```

Use:

```python
for item in paginated_items(client, "/users"):
    process(item)
```

This is elegant.

But it performs network I/O during iteration.

That should be documented.

Lazy does not always mean cheap.

It means work happens when values are requested.

If requesting values performs I/O, errors may happen in the consuming loop, not when the generator is created.

That is important for error handling.

---

# Where Errors Happen

Generator bodies run during iteration, not creation.

Example:

```python
def broken():
    print("running")
    raise ValueError("boom")
    yield 1
```

Create:

```python
gen = broken()
```

No error yet.

Now:

```python
next(gen)
```

prints:

```text
running
```

and raises:

```python
ValueError
```

This affects API design.

If validation should happen immediately, do it outside the generator body or before returning the generator.

Example:

```python
def read_limited(path, limit):
    if limit < 0:
        raise ValueError("limit cannot be negative")

    def generate():
        with open(path) as file:
            for index, line in enumerate(file):
                if index >= limit:
                    break
                yield line

    return generate()
```

Now invalid `limit` fails at call time.

File errors still happen during iteration.

This distinction is subtle but important.

---

# Generator Functions as APIs

When a function contains `yield`, calling it returns a generator object.

That means errors and work inside the function body are delayed.

Example:

```python
def values(limit):
    if limit < 0:
        raise ValueError("limit cannot be negative")
    for number in range(limit):
        yield number
```

Surprise:

```python
gen = values(-1)
```

does not raise immediately.

The body has not run.

The error appears when consuming:

```python
next(gen)
```

If immediate validation matters, wrap the generator:

```python
def values(limit):
    if limit < 0:
        raise ValueError("limit cannot be negative")

    def generate():
        for number in range(limit):
            yield number

    return generate()
```

Now:

```python
values(-1)
```

raises immediately.

This is a design choice.

Many generator APIs accept delayed errors.

But you should know what your function does.

---

# `yield` in `try` Blocks

Generators can yield inside `try` blocks.

Example:

```python
def managed_values():
    print("open")
    try:
        yield 1
        yield 2
    finally:
        print("close")
```

If consumed fully:

```python
for value in managed_values():
    print(value)
```

Output:

```text
open
1
2
close
```

If closed early:

```python
gen = managed_values()
print(next(gen))
gen.close()
```

Output:

```text
open
1
close
```

This is useful for cleanup.

But for resource management, context managers often communicate intent better.

Generators and context managers meet in the next chapter.

---

# Generators and Context Managers

The standard library has tools that turn generator functions into context managers.

Example idea:

```python
from contextlib import contextmanager


@contextmanager
def managed_resource():
    acquire()
    try:
        yield resource
    finally:
        release()
```

This pattern is powerful.

But it belongs in Chapter 60 because context managers have their own protocol:

```python
__enter__
__exit__
```

For now, know that generators are not only for loops.

They can also express setup-yield-cleanup patterns.

That connection is exactly why context managers come next.

---

# Common Mistake: Thinking Generators Are Lists

This is a generator:

```python
values = (number * number for number in range(3))
```

It is not a list.

This fails:

```python
values[0]
```

This may surprise:

```python
len(values)
```

Generators do not generally support indexing or length.

Use:

```python
list(values)
```

if you need a list.

But remember that converting consumes the generator.

Choose the data structure you need.

Generator:

```text
lazy one-time stream
```

List:

```text
stored reusable sequence
```

---

# Common Mistake: Reusing an Exhausted Generator

Example:

```python
values = (number for number in range(3))

print(list(values))
print(list(values))
```

Output:

```python
[0, 1, 2]
[]
```

The generator is exhausted.

Fix by creating a new generator:

```python
def make_values():
    return (number for number in range(3))


print(list(make_values()))
print(list(make_values()))
```

Or store a list if reuse is required:

```python
values = list(range(3))
```

Know whether you need a stream or stored data.

---

# Common Mistake: Hidden Consumption in Debugging

This debug line consumes the generator:

```python
print(list(values))
```

Then later code sees no values.

Example:

```python
def process(values):
    print("debug", list(values))
    return sum(values)
```

If `values` is a generator, `sum(values)` returns `0` after the debug print.

Fix:

```python
def process(values):
    values = list(values)
    print("debug", values)
    return sum(values)
```

This is fine if values are finite and fit in memory.

For large streams, debug differently:

```python
def process(values):
    for value in values:
        print("debug", value)
        handle(value)
```

Generators make consumption visible if you know to look.

---

# Common Mistake: Yielding a Generator Instead of Delegating

Wrong:

```python
def flatten(groups):
    for group in groups:
        yield (item for item in group)
```

This yields generator objects.

It does not yield the items.

Better:

```python
def flatten(groups):
    for group in groups:
        yield from group
```

or:

```python
def flatten(groups):
    for group in groups:
        for item in group:
            yield item
```

Use `yield from` when the intent is:

```text
produce every item from this sub-iterable
```

---

# Common Mistake: Putting Too Much Logic in One Generator

This is hard to test:

```python
def process(path):
    with open(path) as file:
        for line in file:
            ...
            ...
            ...
            yield result
```

Better:

```python
lines = read_lines(path)
records = parse_records(lines)
valid = validate_records(records)
results = transform(valid)
```

Each generator stage can be tested separately.

Small generator functions compose well.

Large generator functions become hidden workflows.

Use names to make the pipeline readable.

---

# Common Mistake: Hiding Expensive Work

Generator functions are lazy, but not free.

This looks harmless:

```python
items = remote_items(client)
```

No network calls may happen yet.

But this:

```python
for item in items:
    ...
```

may perform many network calls.

That can be a good design.

But document it.

Lazy work still happens.

It just happens later.

Readers should know whether iteration is CPU-only, file-backed, network-backed, database-backed, or infinite.

---

# Common Mistake: Catching All Exceptions Around a Pipeline

This is too broad:

```python
try:
    for record in pipeline:
        process(record)
except Exception:
    pass
```

It hides real errors.

Maybe parsing failed.

Maybe the file disappeared.

Maybe a bug exists in the transformation.

Catch specific exceptions where you can handle them meaningfully:

```python
for record in pipeline:
    try:
        process(record)
    except InvalidRecord as error:
        report(error)
```

Generator pipelines should not become error-swallowing tunnels.

Let unexpected errors be visible.

---

# Common Mistake: Forgetting Cleanup When Stopping Early

Suppose a generator owns a resource:

```python
def lines(path):
    file = open(path)
    try:
        for line in file:
            yield line
    finally:
        file.close()
```

If you consume only one line:

```python
gen = lines("data.txt")
first = next(gen)
```

the file may remain open until the generator is closed or garbage collected.

Better:

```python
gen = lines("data.txt")
try:
    first = next(gen)
finally:
    gen.close()
```

Or design the API with a context manager.

For file line iteration, using `with open(...)` directly is often clearer.

Generators are wonderful for streams.

Resource lifetime still needs care.

---

# Testing Generators

Testing a generator usually means consuming it.

Example:

```python
def evens(numbers):
    for number in numbers:
        if number % 2 == 0:
            yield number
```

Test:

```python
assert list(evens([1, 2, 3, 4])) == [2, 4]
```

For infinite generators, use a limit:

```python
from itertools import islice


assert list(islice(count_from(10), 3)) == [10, 11, 12]
```

For pipelines, test stages:

```python
assert list(parse_records(['{"id": 1}'])) == [{"id": 1}]
```

Do not only test the full pipeline.

Generators are easy to compose, so make them easy to test.

---

# Typing Generators

Later chapters cover static type checking in depth.

For now, know that generators can be described by types such as:

```python
Iterator[int]
Iterable[int]
Generator[int, None, None]
```

The simplest useful hint is often:

```python
from collections.abc import Iterator


def numbers() -> Iterator[int]:
    yield 1
    yield 2
```

This says the function returns an iterator of integers.

For advanced generator methods like `send`, the full `Generator` type can describe yielded values, sent values, and return values.

Most everyday generator functions can use `Iterator[T]`.

---

# Generators and Async Generators

Python also has asynchronous generators.

They use:

```python
async def
yield
```

and are consumed with:

```python
async for
```

Example shape:

```python
async def events():
    async for event in source:
        yield event
```

This chapter does not teach async generators deeply.

They belong with asyncio and asynchronous iteration.

But the conceptual connection is direct:

```text
normal generator -> lazy synchronous iterator
async generator  -> lazy asynchronous iterator
```

The iterator protocol has an asynchronous cousin.

We will return to that later.

---

# A Design Checklist

Before writing a generator, ask:

```text
Am I producing a sequence of values over time?
```

If yes, a generator may fit.

Ask:

```text
Do I need all values in memory?
```

If yes, a list may be better.

Ask:

```text
Will the values be consumed only once?
```

If yes, a generator is natural.

Ask:

```text
Does iteration perform I/O?
```

If yes, document that and handle errors carefully.

Ask:

```text
Does the generator own a resource?
```

If yes, design cleanup.

Ask:

```text
Would a generator expression be enough?
```

For simple transformations, yes.

Ask:

```text
Would a named generator function be clearer?
```

For multi-step logic, yes.

Ask:

```text
Do I need class behavior?
```

If yes, a manual iterator class may be better.

---

# Practice: Basic Generator

Write a generator function that yields numbers from `1` to `limit`.

Solution:

```python
def count_up_to(limit):
    current = 1
    while current <= limit:
        yield current
        current += 1
```

Test:

```python
assert list(count_up_to(3)) == [1, 2, 3]
```

Notice that you did not write `StopIteration`.

The generator stops when the function ends.

---

# Practice: Generator Expression

Create a generator expression that yields squares of even numbers from `0` to `9`.

Solution:

```python
even_squares = (
    number * number
    for number in range(10)
    if number % 2 == 0
)
```

Test:

```python
assert list(even_squares) == [0, 4, 16, 36, 64]
```

Then try:

```python
assert list(even_squares) == []
```

The generator is exhausted after the first conversion.

---

# Practice: Flatten One Level

Write a generator that flattens one level of nesting.

Solution:

```python
def flatten_once(groups):
    for group in groups:
        yield from group
```

Test:

```python
assert list(flatten_once([[1, 2], [3], []])) == [1, 2, 3]
```

Explain:

```text
yield from group delegates yielding to each group
```

---

# Practice: Stop Early

Write a generator that yields numbers until a negative number appears.

Solution:

```python
def until_negative(numbers):
    for number in numbers:
        if number < 0:
            return
        yield number
```

Test:

```python
assert list(until_negative([1, 2, -1, 3])) == [1, 2]
```

The `return` stops the generator.

The negative number is not yielded.

---

# Practice: Streaming Lines

Write a generator that yields non-empty stripped lines from an iterable of lines.

Solution:

```python
def non_empty(lines):
    for line in lines:
        stripped = line.strip()
        if stripped:
            yield stripped
```

Test:

```python
lines = [" a ", "", "b\n", "   "]
assert list(non_empty(lines)) == ["a", "b"]
```

This generator accepts any iterable of strings.

It does not require a file.

That makes it easy to test.

---

# Practice: Pipeline

Build a small pipeline:

```python
def parse_ints(lines):
    for line in lines:
        yield int(line)


def only_positive(numbers):
    for number in numbers:
        if number > 0:
            yield number
```

Use:

```python
lines = ["1", "-2", "3"]
numbers = parse_ints(lines)
positive = only_positive(numbers)

assert list(positive) == [1, 3]
```

The stages are lazy.

Each stage can be tested separately.

---

# Practice: Debug Consumption

What does this print?

```python
values = (number for number in range(3))

print(list(values))
print(list(values))
```

Answer:

```python
[0, 1, 2]
[]
```

Why?

The first `list()` consumes the generator.

The second sees an exhausted generator.

Fix by creating a new generator each time:

```python
def values():
    return (number for number in range(3))
```

or by storing a list:

```python
values = list(range(3))
```

---

# Summary

Generators are a concise way to create iterators.

A generator function is any function that contains `yield`.

Calling a generator function returns a generator object without immediately running the function body.

The generator body runs when values are requested.

`yield` produces a value and suspends the function.

The generator resumes from that point on the next request.

Local variables are preserved between yields.

When a generator function ends or returns, the generator is exhausted and raises `StopIteration`.

Only yielded values are part of the iteration.

Returned values are not yielded to ordinary loops.

Generator expressions are lazy comprehension-like expressions.

They are useful when values can be streamed into another consumer.

`yield from` delegates yielding to another iterable and can also receive a subgenerator's return value in advanced use.

Generators are excellent for lazy pipelines, file processing, pagination, tree traversal, infinite sequences, and staged transformations.

Generators are one-shot iterators.

If you need reuse, create a new generator or materialize values.

If a generator owns resources, cleanup matters.

The design principle is:

```text
use generators when the natural shape of the problem is producing values over time
```

Use lists when you need stored reusable data.

Use manual iterator classes when object behavior needs more structure.

Use generators when a clear sequence of `yield` statements tells the story best.

---

# Preview of Chapter 60

Chapter 59 showed how generators can pause, resume, and produce values lazily.

We also saw that generators sometimes need cleanup.

That leads directly to context managers.

Chapter 60 studies the protocol behind:

```python
with resource:
    ...
```

Context managers help manage setup and teardown.

They are used for:

* files
* locks
* database transactions
* temporary configuration
* timing blocks
* resource cleanup
* exception handling boundaries

We will study:

* `__enter__`
* `__exit__`
* the `with` statement
* exception handling inside context managers
* `contextlib`
* generator-based context managers
* nested context managers
* when context managers improve design

The transition is:

```text
generators can produce values over time
context managers control resource lifetime over a block of time
```

Both are Pythonic abstractions for controlling flow clearly.

