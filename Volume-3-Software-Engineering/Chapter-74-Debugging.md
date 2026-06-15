# Chapter 74 — Debugging

Debugging is the discipline of finding the cause of incorrect behavior.

Testing tells you that something is wrong.

Debugging tells you why.

That difference matters.

A failing test is a signal.

A traceback is a clue.

A log line is evidence.

A user report is a witness statement.

None of them is automatically the cause.

Professional debugging is not frantic editing.

It is investigation.

You observe.

You reproduce.

You reduce.

You inspect.

You form a hypothesis.

You test the hypothesis.

You make the smallest fix that addresses the cause.

Then you add or update tests so the bug does not quietly return.

That is the loop.

```text
observe -> reproduce -> isolate -> explain -> fix -> verify
```

The better you become at this loop, the calmer you become around broken systems.

---

# Debugging Is Not Guessing

Beginners often debug by guessing.

They change a line.

They run the program.

They change another line.

They add a print statement.

They comment something out.

They reinstall a package.

They restart the server.

Sometimes the problem disappears.

But they do not know why.

That is dangerous.

A bug that disappears without explanation can return without warning.

Professional debugging tries to replace guessing with evidence.

The key question is:

```text
what observation would prove or disprove my current idea?
```

If you cannot answer that, you are probably not debugging yet.

You are poking.

Poking is allowed at the beginning when you are orienting yourself.

But you should quickly move toward evidence.

---

# The Debugging Mindset

Debugging requires patience, suspicion, and humility.

Patience because the first obvious explanation may be wrong.

Suspicion because code often behaves according to details you forgot.

Humility because the bug may be in code you wrote with full confidence.

The computer is not offended by your theory.

It simply executes the program.

When behavior surprises you, the useful response is not:

```text
that makes no sense
```

The useful response is:

```text
what assumption did I make that the program is contradicting?
```

Almost every bug contains a false assumption.

Examples:

* this value cannot be `None`
* this list always has items
* this function is called once
* this field is always present
* this date is always timezone-aware
* this path exists
* this API returns JSON
* this query returns one row
* this cache is empty at startup
* this task finishes before that task begins
* this code runs only in one process

Debugging is the process of locating the false assumption.

---

# Start With the Symptom

A symptom is the visible failure.

Examples:

* a test fails
* an exception is raised
* an API returns `500`
* a command exits with code `1`
* a report contains wrong totals
* a page is blank
* a process hangs
* memory grows over time
* a job runs too slowly
* a user sees incorrect permissions

The symptom is where the investigation starts.

It is not always where the bug lives.

For example, an API may return `500` because a database row is missing.

The missing row may be caused by an earlier background job.

The background job may have skipped work because of an invalid configuration value.

The configuration value may have been renamed during deployment.

The symptom is the smoke.

The cause may be elsewhere.

Good debugging follows evidence backward from symptom to cause.

---

# Reproduce the Failure

The first serious goal is reproduction.

If you cannot reproduce a failure, you cannot reliably know whether you fixed it.

Reproduction means finding a repeatable way to make the bug happen.

For a test failure, reproduction may be:

```bash
pytest tests/test_checkout.py::test_discount_is_applied
```

For a CLI bug:

```bash
python -m tool import users.csv
```

For an API bug:

```bash
curl -X POST http://localhost:8000/users \
  -H "Content-Type: application/json" \
  -d '{"email": ""}'
```

For a production-only bug, reproduction may require:

* same input data
* same environment variables
* same dependency versions
* same database state
* same feature flags
* same timezone
* same user permissions
* same concurrency pattern

Do not skip reproduction.

Without reproduction, you are trying to hit a moving target in fog.

---

# Make the Failure Smaller

After reproducing the bug, reduce it.

Reduction means finding the smallest example that still fails.

Suppose a report is wrong for a file with ten thousand rows.

Try ten rows.

Then three rows.

Then one row.

If one row still fails, the problem is easier to inspect.

If one row passes but two rows fail, the interaction between rows matters.

Reduction turns a broad mystery into a smaller one.

For example:

```python
def test_total_for_multiple_items():
    items = [
        {"price": 100, "quantity": 2},
        {"price": 50, "quantity": 1},
    ]

    assert total(items) == 250
```

If that fails, try:

```python
def test_total_for_one_item_with_quantity():
    items = [{"price": 100, "quantity": 2}]

    assert total(items) == 200
```

The smaller test may reveal that quantity is ignored.

The reduction process is not busywork.

It sharpens the bug.

---

# Read the Error Message

This sounds obvious.

It is often skipped.

Read the error message carefully.

Read all of it.

Many debugging sessions waste time because the developer reacts to the first line and ignores the rest.

For example:

```text
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

This tells you:

* the operation was `+`
* one operand was an `int`
* the other operand was a `str`

That is strong evidence.

The next question is:

```text
where did the string enter the calculation?
```

Another example:

```text
KeyError: 'email'
```

This tells you code expected a key named `email`.

The next questions are:

* what dictionary was being accessed?
* where was it created?
* should `email` be required?
* should missing email be validated earlier?
* is the input shape different from what the code expects?

Error messages are not decorations.

They are evidence.

---

# Understand Tracebacks

A traceback shows the call stack at the point where an exception was raised.

Example:

```text
Traceback (most recent call last):
  File "app.py", line 20, in <module>
    main()
  File "app.py", line 16, in main
    checkout(cart)
  File "app.py", line 9, in checkout
    total = calculate_total(cart.items)
  File "app.py", line 5, in calculate_total
    return sum(item["price"] * item["quantity"] for item in items)
KeyError: 'quantity'
```

Read it from top to bottom to understand the call path.

Read it from bottom upward to find the immediate failure.

The bottom line says:

```text
KeyError: 'quantity'
```

The frame above it says where:

```text
calculate_total
```

The earlier frames say how execution arrived there:

```text
main -> checkout -> calculate_total
```

The immediate failing line is not always the root cause.

The root cause may be that invalid item data was created earlier.

But the traceback gives you the first concrete place to inspect.

---

# Do Not Ignore the Middle of the Traceback

Long tracebacks can be intimidating.

Frameworks make them longer.

A web application traceback may include dozens of framework frames.

Do not panic.

Look for the frames that belong to your project.

For example:

```text
File ".../site-packages/fastapi/routing.py", line ...
File ".../site-packages/starlette/routing.py", line ...
File "/app/orders/api.py", line 42, in create_order
File "/app/orders/service.py", line 88, in checkout
File "/app/orders/pricing.py", line 17, in total
```

The framework frames show the request machinery.

Your frames show your behavior.

The important frame is often the last frame in your code before the exception.

But sometimes the bug is earlier in your code.

Use the traceback as a map.

Do not treat it as a single line.

---

# Traceback Chaining

Python can show chained exceptions.

Example:

```python
try:
    user_id = int(raw_user_id)
except ValueError as error:
    raise InvalidUserId("user id must be an integer") from error
```

The traceback may show both exceptions.

The lower-level exception explains what originally failed.

The higher-level exception explains how your application interpreted that failure.

Chaining is useful because it preserves cause.

Without chaining, code may hide important evidence.

This is less helpful:

```python
try:
    user_id = int(raw_user_id)
except ValueError:
    raise InvalidUserId("user id must be an integer")
```

This loses the original exception context if not handled carefully.

When debugging, pay attention to phrases like:

```text
The above exception was the direct cause of the following exception
```

or:

```text
During handling of the above exception, another exception occurred
```

They tell you there is more than one failure involved.

---

# The traceback Module

Python's `traceback` module provides tools for extracting, formatting, and printing traceback information.

It is useful when you need to capture exception details programmatically.

Example:

```python
import traceback


try:
    risky_operation()
except Exception:
    text = traceback.format_exc()
    save_error_report(text)
```

This captures the current exception traceback as text.

You may also format a traceback object:

```python
import traceback


try:
    risky_operation()
except Exception as error:
    lines = traceback.format_exception(error)
```

This is useful for:

* error reports
* diagnostic logs
* test assertions on failure output
* custom exception handling
* debugging tools

Do not use `traceback` as a way to hide errors.

Capturing an exception should usually be paired with either handling it meaningfully or reporting it clearly.

---

# Print Debugging

Print debugging means adding output to inspect program state.

Example:

```python
def calculate_total(items):
    print("items:", items)
    return sum(item["price"] * item["quantity"] for item in items)
```

Print debugging is simple and often useful.

It is especially helpful when:

* you are exploring unfamiliar code
* the program is small
* the failure is easy to reproduce
* using a debugger would be slower
* you need to inspect a value quickly

But print debugging has limits.

Prints can be forgotten in code.

They can produce too much output.

They can change timing in concurrent code.

They can expose sensitive data.

They can be awkward in servers, background workers, and tests.

Use prints when they help.

Remove them when done.

For longer-lived diagnostic output, use logging.

Chapter 75 will cover logging deeply.

---

# Use repr When Printing Values

When debugging strings, use `repr`.

Compare:

```python
value = "admin "

print(value)
```

Output:

```text
admin
```

The trailing space is easy to miss.

With `repr`:

```python
print(repr(value))
```

Output:

```text
'admin '
```

Now the trailing space is visible.

`repr` is useful for debugging:

* empty strings
* whitespace
* newline characters
* tabs
* `None`
* bytes
* lists
* dictionaries
* objects with useful representations

When a value looks correct but behaves incorrectly, inspect its representation and type.

---

# Inspect Types

Many Python bugs are type expectation bugs.

Examples:

* string instead of integer
* naive datetime instead of timezone-aware datetime
* list instead of dictionary
* bytes instead of string
* `None` instead of object
* float instead of `Decimal`
* generator instead of list

Inspect type directly:

```python
print(type(value), repr(value))
```

For example:

```python
price = "100"
quantity = 2

print(type(price), repr(price))
```

Output:

```text
<class 'str'> '100'
```

The value visually looks numeric.

It is a string.

Python's dynamic typing makes this kind of bug common.

Type hints and static type checking help prevent some of them.

Debugging helps when they reach runtime.

---

# Logging While Debugging

Logging is structured diagnostic output.

Compared with print, logging can include:

* severity levels
* timestamps
* module names
* request IDs
* user IDs
* stack traces
* output destinations
* formatting configuration

Example:

```python
import logging


logger = logging.getLogger(__name__)


def calculate_total(items):
    logger.debug("Calculating total for %r", items)
    return sum(item["price"] * item["quantity"] for item in items)
```

The `%r` formatting shows representations.

Do not write:

```python
logger.debug(f"Calculating total for {items}")
```

That eagerly formats the message even if debug logging is disabled.

Use logging's lazy formatting:

```python
logger.debug("Calculating total for %r", items)
```

Logging is especially useful when debugging:

* web requests
* production incidents
* background jobs
* distributed systems
* long-running processes
* async tasks
* intermittent failures

But logging is not a substitute for understanding.

It supplies evidence.

You still interpret it.

---

# Breakpoints

A breakpoint pauses execution so you can inspect program state.

In Python, the simplest breakpoint is:

```python
breakpoint()
```

When execution reaches that line, Python enters the debugger.

The default debugger is `pdb`.

You can inspect variables, move through the stack, step line by line, and continue execution.

Example:

```python
def calculate_total(items):
    breakpoint()
    return sum(item["price"] * item["quantity"] for item in items)
```

When the breakpoint is hit, you will see a prompt like:

```text
(Pdb)
```

At that prompt, you can ask questions about the running program.

This is more powerful than printing because you can inspect many values without editing code repeatedly.

---

# pdb

`pdb` is Python's built-in debugger.

It supports:

* breakpoints
* stepping through code
* continuing execution
* inspecting variables
* moving through stack frames
* listing source code
* evaluating expressions
* post-mortem debugging

Common commands include:

```text
h      help
w      where am I in the stack?
u      move up to an older stack frame
d      move down to a newer stack frame
l      list source code
n      next line
s      step into function call
r      run until current function returns
c      continue execution
p x    print expression x
pp x   pretty-print expression x
q      quit debugger
```

You do not need to memorize every command at once.

Start with:

```text
w
p
n
s
c
q
```

Those cover many debugging sessions.

---

# Stepping Through Code

`next` and `step` are different.

`next` executes the next line in the current function.

If that line calls another function, `next` runs the whole call and stops after it returns.

`step` enters the called function.

Example:

```python
def total(items):
    return sum_prices(items)
```

If the debugger is on:

```python
return sum_prices(items)
```

then:

```text
n
```

runs `sum_prices` and stops after the line.

But:

```text
s
```

enters `sum_prices`.

Use `step` when you suspect the called function.

Use `next` when you trust the called function and want to stay at the current level.

Debugging becomes calmer when you move deliberately.

---

# Moving Through Stack Frames

When an exception happens, the current frame is often deep in the call stack.

`pdb` lets you move through frames.

Use:

```text
w
```

to see the stack.

Use:

```text
u
```

to move up to the caller.

Use:

```text
d
```

to move back down.

This matters because the failing line may not have all the context.

For example, `calculate_total` may fail because it received invalid items.

Move up to the caller to inspect where those items came from.

Debugging often means following bad data backward through frames.

---

# Conditional Breakpoints

Sometimes a bug happens only for a specific value.

Instead of stopping on every call, use a conditional breakpoint.

In `pdb`, you can set a breakpoint with a condition:

```text
b orders.py:42, user_id == "u-123"
```

This means:

```text
stop at line 42 only when user_id is "u-123"
```

Conditional breakpoints are useful for:

* loops
* large datasets
* repeated function calls
* rare values
* specific users
* specific state transitions

Without a condition, you may stop hundreds of times before reaching the interesting case.

With a condition, the debugger works closer to your hypothesis.

---

# Post-Mortem Debugging

Post-mortem debugging means entering the debugger after an exception has occurred.

You can run:

```python
import pdb


try:
    main()
except Exception:
    pdb.post_mortem()
```

In an interactive session, after an exception, you can use:

```python
import pdb
pdb.pm()
```

This opens the debugger at the point of failure.

Post-mortem debugging is useful because you do not need to guess where to place a breakpoint in advance.

The program already failed.

You inspect the failed state.

---

# Running a Script Under pdb

You can run a script under `pdb` from the command line:

```bash
python -m pdb script.py
```

For a module:

```bash
python -m pdb -m package.module
```

This starts execution under debugger control.

You can set breakpoints before continuing.

For example:

```text
(Pdb) b app.py:42
(Pdb) c
```

This is useful when:

* the program exits too quickly
* you want to debug startup code
* you do not want to insert `breakpoint()` into source
* you need to inspect command-line behavior

Use inline `breakpoint()` for quick local debugging.

Use `python -m pdb` when controlling execution from the start is cleaner.

---

# Debugging Tests

Tests are excellent debugging entry points.

If a test fails, run only that test:

```bash
pytest tests/test_checkout.py::test_discount_is_applied
```

Then make it more verbose if needed:

```bash
pytest -vv tests/test_checkout.py::test_discount_is_applied
```

To stop at the first failure:

```bash
pytest -x
```

To enter the debugger on failure:

```bash
pytest --pdb
```

You can also insert:

```python
breakpoint()
```

inside the code or test.

A failing test gives you a repeatable reproduction.

That is a gift.

Use it.

---

# Debugging With Assertions

Assertions are useful during debugging because they make assumptions executable.

Example:

```python
def calculate_total(items):
    assert all("price" in item for item in items)
    assert all("quantity" in item for item in items)
    return sum(item["price"] * item["quantity"] for item in items)
```

If the assertion fails, you have found a violated assumption.

But be careful.

Python can remove `assert` statements when run with optimization:

```bash
python -O app.py
```

Do not use `assert` for essential runtime validation in production code.

Use explicit exceptions for real input validation:

```python
if "quantity" not in item:
    raise ValueError("item is missing quantity")
```

Assertions are excellent for internal invariants and debugging assumptions.

They are not a replacement for user-facing validation.

---

# Form a Hypothesis

After observing evidence, form a hypothesis.

A hypothesis is a specific explanation that can be tested.

Weak:

```text
the checkout is broken
```

Better:

```text
the discount is applied before item quantities are multiplied
```

Better still:

```text
calculate_total ignores quantity when an item has a discount field
```

A good debugging hypothesis points toward an experiment.

For example:

```text
If the bug is caused by discounted items ignoring quantity,
then a cart with one discounted item and quantity 2 should produce the wrong total.
```

Now write or run that case.

Hypotheses keep debugging directed.

Without them, you wander.

---

# Test One Thing at a Time

When debugging, change one thing at a time.

If you change five things and the bug disappears, you do not know which change mattered.

This creates weak fixes.

For example, do not simultaneously:

* change the query
* clear the cache
* update the fixture
* alter the serializer
* restart the worker

If the failure disappears, the cause remains unclear.

Instead, isolate variables.

Change one factor.

Observe.

Revert or keep based on evidence.

Then move to the next factor.

This is slower for five minutes and faster for the whole debugging session.

---

# Binary Search the Cause

Binary search is useful when a bug was introduced somewhere in a sequence.

Examples:

* a long commit history
* a large input file
* many feature flags
* many configuration values
* a large list of rows

For commits, `git bisect` can locate the first bad commit.

Conceptually:

```text
known good commit
known bad commit
test midpoint
keep narrowing
```

For data, split the input in half.

If the first half fails, the bug is there.

If the second half fails, the bug is there.

If neither half fails alone, the bug may depend on interaction between parts.

Binary search is powerful because it reduces large spaces quickly.

Use it whenever the problem has an ordered or splittable search space.

---

# Debugging Data Flow

Many bugs are data-flow bugs.

The wrong value appears somewhere.

The task is to find where it changed.

Ask:

* where is the value created?
* where is it transformed?
* where is it validated?
* where is it stored?
* where is it read?
* where does it cross a boundary?
* when does it become wrong?

For example:

```text
request JSON -> validation -> domain object -> database row -> API response
```

If the response is wrong, inspect each stage.

Maybe the request was correct.

Maybe validation converted a type incorrectly.

Maybe the domain object was correct.

Maybe the database stored a truncated value.

Maybe the response serializer renamed a field.

Debugging data flow means following the value through the system.

---

# Debugging Control Flow

Other bugs are control-flow bugs.

The wrong branch runs.

The wrong function is called.

A loop exits too early.

An exception path is triggered unexpectedly.

Questions:

* which branch executed?
* why did that condition evaluate that way?
* how many loop iterations happened?
* did the function return early?
* did an exception skip later code?
* was the callback registered?
* was the task scheduled?

Add temporary logging or breakpoints around decisions:

```python
if user.is_admin:
    logger.debug("admin path for user_id=%s", user.id)
else:
    logger.debug("non-admin path for user_id=%s", user.id)
```

Control-flow bugs often come from conditions that are almost right.

Boundary values matter.

---

# Debugging State

State bugs happen when an object or system remembers something incorrectly.

Examples:

* a cache contains stale data
* a global setting changed
* a database transaction did not commit
* a session object is reused
* a class attribute is shared accidentally
* a mutable default argument persists across calls

Classic Python example:

```python
def add_item(item, items=[]):
    items.append(item)
    return items
```

This list is shared across calls.

Debugging reveals it:

```python
print(add_item("a"))
print(add_item("b"))
```

Output:

```text
['a']
['a', 'b']
```

The fix:

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

State bugs often require asking:

```text
who else can see or mutate this object?
```

Volumes I and II prepared you for this.

Names, references, mutability, class attributes, descriptors, closures, and globals all matter during debugging.

---

# Debugging None

`None` bugs are common.

Examples:

```text
AttributeError: 'NoneType' object has no attribute 'id'
TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
```

The immediate failure says a value was `None`.

The cause is usually earlier.

Ask:

* where was the value supposed to be created?
* can the function return `None`?
* was a missing database row handled?
* did a dictionary lookup use `.get()`?
* did validation allow an absent field?
* did a branch forget to return?

Example:

```python
def find_user(users, email):
    for user in users:
        if user.email == email:
            return user
```

If no user matches, the function implicitly returns `None`.

Maybe that is intended.

Maybe it is not.

Make the contract explicit:

```python
def find_user(users, email):
    for user in users:
        if user.email == email:
            return user
    raise LookupError(f"user not found: {email}")
```

Debugging `None` often improves API design.

---

# Debugging Imports

Import bugs can be confusing.

Common symptoms:

* `ModuleNotFoundError`
* `ImportError`
* circular import errors
* wrong module imported
* code runs at import time unexpectedly
* local file shadows installed package

Example shadowing:

```text
project/
    requests.py
```

If your code says:

```python
import requests
```

Python may import your local `requests.py` instead of the third-party package.

Debugging import problems often starts with:

```python
import module
print(module.__file__)
```

This tells you where the module came from.

Also inspect:

```python
import sys
print(sys.path)
```

Import bugs are about search paths, module names, and execution timing.

The import chapters in Volume I are not abstract here.

They are practical debugging tools.

---

# Debugging Circular Imports

Circular imports happen when modules depend on each other during import.

Example:

```python
# users.py
from orders import Order


class User:
    ...
```

```python
# orders.py
from users import User


class Order:
    ...
```

When Python imports `users`, it imports `orders`.

When importing `orders`, it tries to import `users` again.

But `users` is only partially initialized.

Symptoms may include messages about a partially initialized module.

Fixes include:

* move shared types to a third module
* import inside a function when appropriate
* use `typing.TYPE_CHECKING` for type-only imports
* reduce module-level side effects
* reorganize ownership boundaries

Circular imports often reveal design coupling.

The debugging fix may be architectural, not just syntactic.

---

# Debugging Attribute Errors

An `AttributeError` means an object does not have the requested attribute.

Example:

```text
AttributeError: 'dict' object has no attribute 'email'
```

This often means code expected an object but received a dictionary.

Inspect:

```python
print(type(user), repr(user))
```

If you expected:

```python
user.email
```

but received:

```python
{"email": "a@example.com"}
```

then either the caller passed the wrong shape or the function's expectation is wrong.

Attribute errors are often boundary bugs.

Data crosses from JSON, database rows, forms, or external APIs into Python objects.

Somewhere, the shape changed.

---

# Debugging Key Errors

A `KeyError` means a dictionary key is missing.

Example:

```text
KeyError: 'quantity'
```

Do not immediately replace:

```python
item["quantity"]
```

with:

```python
item.get("quantity")
```

That may hide the real bug.

Ask:

* should `quantity` be required?
* if missing, should there be a default?
* where is item data validated?
* is the input from an old API version?
* did a serializer rename the key?

Sometimes `.get()` is right.

Sometimes it turns a clear failure into a quiet wrong result.

Debugging is not only making the exception disappear.

It is restoring correct behavior.

---

# Debugging Off-by-One Errors

Off-by-one errors happen at boundaries.

Examples:

* loop starts too early
* loop stops too late
* inclusive boundary treated as exclusive
* exclusive boundary treated as inclusive
* index shifted by one
* date range includes one extra day
* pagination skips or duplicates an item

Example:

```python
def first_n_items(items, n):
    return items[: n - 1]
```

Bug:

```python
assert first_n_items([1, 2, 3], 2) == [1, 2]
```

Actual:

```text
[1]
```

Boundary tests help:

```python
def test_first_n_items():
    assert first_n_items([1, 2, 3], 0) == []
    assert first_n_items([1, 2, 3], 1) == [1]
    assert first_n_items([1, 2, 3], 2) == [1, 2]
```

When debugging boundaries, test:

```text
before
at
after
```

---

# Debugging Timezones

Datetime bugs are famously slippery.

Common problems:

* naive datetime mixed with aware datetime
* server timezone differs from local timezone
* daylight saving transitions
* date stored in UTC but displayed as local
* string parsing loses timezone
* midnight boundary errors
* tests depend on current date

Inspect:

```python
print(value, value.tzinfo)
```

Ask:

* is this datetime timezone-aware?
* what timezone is storage using?
* what timezone is display using?
* where is conversion supposed to happen?
* are tests using fixed times?

Avoid debugging time with the real current clock if you can.

Use fixed datetimes in tests.

Make timezone conversion explicit.

Time bugs often come from invisible context.

Make the context visible.

---

# Debugging Floating-Point Issues

Floating-point numbers can surprise you.

Example:

```python
0.1 + 0.2
```

may produce:

```text
0.30000000000000004
```

If a test fails with a tiny numerical difference, the bug may be precision, not business logic.

Use approximate comparisons for measurements.

Use `Decimal` or integer cents for money.

Inspect exact representation:

```python
print(repr(value))
```

Ask what kind of number the domain requires.

Debugging numeric bugs often starts as a code problem and ends as a modeling problem.

---

# Debugging Concurrency

Concurrency bugs are difficult because timing matters.

Symptoms:

* intermittent failures
* deadlocks
* race conditions
* missing updates
* duplicate work
* tests pass alone but fail together
* logs appear in surprising order
* shared state becomes inconsistent

Questions:

* what state is shared?
* who can mutate it?
* what synchronization protects it?
* can operations interleave?
* are locks acquired in consistent order?
* are tasks awaited?
* are exceptions inside tasks observed?

Adding print statements can change timing.

This can hide concurrency bugs.

Use structured logs, deterministic tests where possible, and smaller reproductions.

Concurrency debugging is mostly about interleavings.

The code may be correct in one order and wrong in another.

---

# Debugging Async Code

Async bugs often involve missing `await`, cancelled tasks, swallowed exceptions, and event loop assumptions.

Common symptoms:

* coroutine was never awaited
* task exception was never retrieved
* code runs in unexpected order
* timeout occurs
* background task silently fails
* test exits before task completes

Example bug:

```python
async def handler():
    send_email()
    return {"ok": True}
```

If `send_email` is async, this is wrong.

It should be:

```python
async def handler():
    await send_email()
    return {"ok": True}
```

Warnings are evidence.

Do not ignore:

```text
RuntimeWarning: coroutine was never awaited
```

Async debugging requires tracking tasks, awaits, and cancellation paths.

The event loop is part of the program's control flow.

---

# Debugging Memory Growth

Memory growth may come from:

* unbounded caches
* retained references
* global lists
* closures holding large objects
* reference cycles with finalizers
* large temporary objects
* queues not drained
* tasks accumulating
* native extension leaks

First distinguish:

```text
high memory use
```

from:

```text
memory leak
```

High memory use may be expected for large inputs.

A leak means memory grows over time and is not released when expected.

Debugging memory often requires:

* reproducing with a smaller workload
* measuring memory at intervals
* checking object counts
* inspecting caches
* looking for retained references
* using tools such as `tracemalloc`

Memory bugs connect directly to Volume I's memory chapters and Volume II's object lifecycle chapters.

Python manages memory automatically.

It does not make memory behavior irrelevant.

---

# tracemalloc

`tracemalloc` helps trace Python memory allocations.

Example:

```python
import tracemalloc


tracemalloc.start()

run_workload()

snapshot = tracemalloc.take_snapshot()
top = snapshot.statistics("lineno")

for stat in top[:10]:
    print(stat)
```

This can show where memory was allocated.

It is useful when debugging Python-level memory growth.

It may not fully explain memory used by native extensions, external libraries, or the operating system.

Use it as one lens, not the only lens.

---

# faulthandler

Most Python exceptions produce tracebacks normally.

But some failures are lower-level.

Examples:

* segmentation faults
* stack overflows
* fatal interpreter errors
* native extension crashes

The `faulthandler` module can dump Python tracebacks when serious faults occur.

You can enable it:

```python
import faulthandler


faulthandler.enable()
```

Or from the command line:

```bash
python -X faulthandler script.py
```

This is useful when the interpreter crashes instead of raising a normal Python exception.

Crashes are more common when native extensions, C libraries, or low-level integrations are involved.

That is why Chapter 71 mattered before this volume.

Native boundaries change the debugging game.

---

# Debugging Performance Problems

Performance debugging is not the same as ordinary correctness debugging.

The symptom is not wrong output.

The symptom is slow behavior.

First reproduce the slowness.

Then measure.

Do not guess.

Common causes:

* inefficient algorithm
* unnecessary database queries
* N+1 query pattern
* repeated serialization
* missing cache
* excessive logging
* network latency
* large object creation
* lock contention
* slow regular expression
* repeated imports in hot paths

Use timing and profiling tools.

Chapter 79 will cover profiling in detail.

For now, remember:

```text
performance debugging starts with measurement
```

The slow part is often not where you expected.

---

# Debugging With Minimal Examples

A minimal example is the smallest code that reproduces the bug.

It is useful for:

* understanding the issue
* asking for help
* filing bug reports
* testing assumptions
* separating your code from a library issue

For example, instead of sharing an entire application, reduce the issue to:

```python
from datetime import datetime


value = datetime.fromisoformat("2026-01-01T10:00:00+05:30")
print(value.tzinfo)
```

Minimal examples force clarity.

If you cannot reduce the bug, you may not yet understand its conditions.

The act of reducing often reveals the cause.

---

# Debugging Environment Problems

Sometimes the code is fine and the environment is wrong.

Examples:

* wrong Python version
* wrong virtual environment
* missing dependency
* different dependency version
* environment variable missing
* working directory different
* file permissions differ
* timezone differs
* locale differs
* operating system differs
* CPU architecture differs

Inspect:

```python
import sys
import os

print(sys.executable)
print(sys.version)
print(os.getcwd())
```

From shell:

```bash
python --version
python -m pip freeze
which python
```

Environment bugs often look like code bugs until you compare contexts.

If it works on one machine and fails on another, ask what differs.

---

# Debugging Dependency Versions

Dependencies change.

A bug may appear after upgrading a package.

Check:

```bash
python -m pip show package-name
```

or inspect inside Python:

```python
import package
print(package.__version__)
```

Not every package exposes `__version__`, but many do.

If a dependency upgrade caused the bug, read release notes and changelogs.

Also ask:

* was the dependency version pinned?
* did transitive dependencies change?
* did the lock file update?
* does CI use the same versions as local?

Debugging dependency issues is partly package management.

Chapter 76 will cover packaging more deeply.

---

# Debugging Configuration

Configuration bugs are common because configuration often lives outside code.

Examples:

* wrong database URL
* missing API key
* feature flag enabled unexpectedly
* debug mode off
* timeout too low
* region mismatch
* wrong file path
* stale secret

Good configuration debugging prints or logs safe summaries.

Do not dump secrets.

Bad:

```python
logger.info("API key: %s", api_key)
```

Better:

```python
logger.info("API key configured: %s", bool(api_key))
```

or:

```python
logger.info("Using API host: %s", api_host)
```

Configuration should be observable without exposing sensitive values.

---

# Debugging Production Incidents

Production debugging has higher stakes.

You may not be able to pause the process.

You may not be able to reproduce locally immediately.

You may be dealing with real users and data.

The priorities are:

1. Protect users and data.
2. Stabilize the system.
3. Preserve evidence.
4. Identify the cause.
5. Fix or mitigate.
6. Prevent recurrence.

Do not start by making random production changes.

Collect evidence:

* error rate
* affected users
* recent deployments
* logs
* metrics
* traces
* configuration changes
* dependency changes
* database migrations
* feature flag changes

Production debugging is as much operational discipline as code reading.

---

# Preserve Evidence

When a bug happens in production, evidence can disappear.

Examples:

* logs rotate
* temporary files are deleted
* queues drain
* caches expire
* failed containers restart
* database rows are modified
* metrics aggregate away detail

Preserve useful evidence early.

Capture:

* traceback
* request ID
* user ID or anonymized identifier
* input shape
* timestamps
* deployment version
* environment
* relevant logs
* configuration version

Be careful with sensitive data.

Do not copy secrets, passwords, tokens, private messages, or unnecessary personal data into bug reports.

Good debugging respects privacy.

---

# Avoid Debugging by Restart

Restarting can be a valid mitigation.

It can clear stuck processes, release resources, and restore service.

But if every debugging session ends at:

```text
restart it and see
```

the team loses knowledge.

If restart fixes the symptom, ask:

* what state did restart clear?
* memory?
* cache?
* connection pool?
* lock?
* background task?
* stale configuration?

Restarting can be part of incident response.

It should not be the whole explanation.

---

# Debugging by Reading Code

Sometimes the best debugger is careful reading.

Read the code around the failure.

Then read the caller.

Then read the data model.

Then read the tests.

Look for:

* hidden assumptions
* mismatched names
* default values
* early returns
* broad exception handlers
* mutable defaults
* global state
* implicit conversions
* old compatibility paths
* TODO comments
* recently changed code

Do not read randomly.

Read along the path of execution.

Use the traceback and reproduction to guide you.

---

# Broad Exception Handlers

Broad exception handlers can hide bugs.

Example:

```python
try:
    process_order(order)
except Exception:
    return None
```

This catches everything.

It may hide programming errors, data errors, and infrastructure errors.

The caller only sees `None`.

Debugging becomes harder because the original failure is swallowed.

Better:

```python
try:
    process_order(order)
except PaymentDeclined:
    return "payment_declined"
```

Catch exceptions you can handle.

Log or re-raise unexpected ones.

If you must catch broadly at a top-level boundary, preserve the traceback:

```python
logger.exception("Unexpected order processing failure")
```

The exception information is evidence.

Do not throw it away.

---

# Silent Failures

Silent failures are bugs that do not raise obvious errors.

Examples:

* function returns wrong value
* background job skips work
* data is partially written
* cache serves stale value
* permissions are too broad
* duplicate event is ignored
* email is not sent

Silent failures are harder than exceptions because there is no traceback.

You need other evidence:

* tests
* logs
* metrics
* database records
* output comparisons
* audit trails
* manual reproduction

For silent failures, add checks near invariants.

Example:

```python
if order.status == "paid" and not order.payment_id:
    raise RuntimeError("paid order must have payment_id")
```

Invariants turn impossible states into visible failures.

Visible failures are easier to debug than corrupted state.

---

# Invariants

An invariant is something that should always be true.

Examples:

* paid orders have payment IDs
* account balance cannot be negative
* published posts have titles
* deleted users cannot log in
* percentage is between 0 and 100
* start date is before end date

Invariants are debugging anchors.

If an invariant is violated, you know the system crossed an invalid boundary somewhere.

You can enforce invariants with:

* validation
* assertions
* dataclass `__post_init__`
* property setters
* database constraints
* type constraints
* tests

The earlier invalid state is detected, the easier debugging becomes.

Late failures are harder because bad state has traveled farther.

---

# Debugging With Git

Version control is a debugging tool.

Useful commands include:

```bash
git diff
git log
git blame file.py
git show commit
git bisect
```

Use `git diff` to see what changed locally.

Use `git log` to inspect recent commits.

Use `git blame` carefully to find when a line changed.

Do not use blame to blame people.

Use it to find context.

Use `git show` to inspect the full change around a commit.

Use `git bisect` when you know one revision was good and another is bad.

Debugging often means understanding history.

Git stores that history.

---

# Debugging After a Failed Refactor

Refactoring should preserve behavior.

If behavior changes after refactoring, compare old and new code.

Ask:

* did the order of operations change?
* did default values change?
* did exceptions change?
* did mutability change?
* did a lazy operation become eager?
* did a generator become a list?
* did a public name move?
* did imports create circular dependencies?
* did tests over-specify old internals?

Run the old and new behavior on the same inputs if possible.

Golden master or characterization tests can help.

The safest refactors are small.

Large refactors create large search spaces when something breaks.

---

# Debugging Test Failures

When a test fails, ask what kind of failure it is.

Possibilities:

* production code is wrong
* test expectation is wrong
* fixture setup is wrong
* mock is configured incorrectly
* test depends on order
* test depends on time
* test depends on environment
* test is too coupled to implementation
* test reveals an unintended behavior change

Do not assume the test is wrong because it failed.

Do not assume production code is wrong either.

Investigate.

The test is evidence.

Like all evidence, it needs interpretation.

---

# Debugging Flaky Tests

Flaky tests require urgency because they damage trust.

First, reproduce the flake.

Run the test repeatedly:

```bash
pytest tests/test_worker.py::test_processes_job -q
```

If needed, run the file or suite repeatedly.

Look for:

* time dependence
* random data
* shared state
* test order dependency
* leftover files
* database leakage
* async tasks not awaited
* real network calls
* concurrency races

Do not simply rerun CI until it passes.

That teaches the team to ignore red builds.

A flaky test is a bug in either the test or the system.

Treat it as such.

---

# Debugging With Logs

Logs are most useful when they answer:

* what happened?
* when?
* for which request?
* for which user or entity?
* under what configuration?
* with what outcome?

Useful log:

```text
payment_declined order_id=ord_123 customer_id=cust_9 reason=insufficient_funds
```

Less useful log:

```text
error happened
```

When debugging with logs, correlate events.

Use request IDs, trace IDs, job IDs, or entity IDs.

Logs without correlation are fragments.

Correlated logs become a story.

Chapter 75 will focus fully on logging because good logs are designed before incidents happen.

---

# Debugging With Metrics

Metrics show patterns.

They answer questions like:

* when did errors start?
* how many users are affected?
* did latency increase?
* did throughput drop?
* did memory grow?
* did retries spike?
* did a queue backlog form?

Metrics are less detailed than logs.

They are better for shape and scale.

For example:

```text
error rate jumped from 0.1% to 12% after deployment
```

That suggests a deployment-related issue.

Metrics guide where to look.

Tracebacks and logs explain individual failures.

Use both.

---

# Debugging With Feature Flags

Feature flags can cause bugs when different users see different behavior.

Debugging flag-related issues requires knowing:

* which flag was evaluated
* what value it returned
* for which user or context
* whether the flag changed recently
* whether fallback behavior applies

Log flag decisions when they matter.

In tests, control flags explicitly.

Do not let important behavior depend on whatever flag state exists in the environment.

Feature flags are dynamic configuration.

Dynamic configuration needs visibility.

---

# Debugging Permissions

Permission bugs are serious.

Symptoms:

* user can access data they should not access
* user cannot access data they should access
* admin-only action available to non-admin
* ownership checks fail
* tenant isolation breaks

Debug permission bugs by making the actors explicit.

Ask:

* who is the current user?
* what roles do they have?
* what resource are they accessing?
* who owns the resource?
* what tenant or organization does it belong to?
* what policy should apply?
* where is that policy enforced?

Write tests for both allowed and denied cases.

Security debugging should end with regression tests.

---

# Debugging Serialization

Serialization bugs happen at boundaries.

Examples:

* datetime becomes string in wrong format
* `Decimal` becomes float
* bytes cannot be JSON encoded
* enum value changes
* missing field breaks older clients
* extra field breaks strict clients

Inspect the serialized form directly.

Do not only inspect the Python object.

For example:

```python
payload = serialize_invoice(invoice)
print(payload)
```

Ask:

* what is the public format?
* who consumes it?
* is this change backward compatible?
* are field names stable?
* are nulls allowed?
* are numbers represented safely?

Serialization bugs often require contract tests.

---

# Debugging Regular Expressions

Regular expressions can fail silently by not matching.

When debugging a regex, inspect:

* the pattern
* the input representation
* match vs search behavior
* greedy vs non-greedy quantifiers
* anchors
* flags
* groups

Use `repr` on input:

```python
print(repr(text))
```

Newlines and spaces matter.

Break complex regexes into named pieces or use verbose mode:

```python
pattern = re.compile(
    r"""
    ^
    (?P<year>\d{4})
    -
    (?P<month>\d{2})
    -
    (?P<day>\d{2})
    $
    """,
    re.VERBOSE,
)
```

If a regex becomes too hard to debug, a parser may be better.

---

# Debugging Recursion

Recursive bugs often show up as:

```text
RecursionError: maximum recursion depth exceeded
```

Ask:

* what is the base case?
* is the base case reachable?
* does each recursive call move closer to the base case?
* is the input smaller or simpler each time?
* are cycles possible?

Example bug:

```python
def countdown(n):
    print(n)
    countdown(n - 1)
```

There is no base case.

Fix:

```python
def countdown(n):
    if n < 0:
        return
    print(n)
    countdown(n - 1)
```

For recursive data structures, cycles can cause infinite recursion.

Track visited nodes when needed.

---

# Debugging Object Identity

Some bugs come from confusing equality and identity.

Example:

```python
if value is 1000:
    ...
```

This is wrong.

Use equality:

```python
if value == 1000:
    ...
```

Use `is` for identity checks, especially:

```python
if value is None:
    ...
```

When debugging object identity, inspect:

```python
print(id(a), id(b))
print(a == b)
print(a is b)
```

Volume I's identity and equality chapter matters directly here.

Many subtle bugs are old concepts returning in practical clothing.

---

# Debugging Mutability

Mutability bugs happen when an object changes unexpectedly.

Example:

```python
def add_tag(user, tag):
    user.tags.append(tag)
```

If multiple users share the same `tags` list accidentally, adding a tag to one affects another.

Debug by checking identity:

```python
print(user1.tags is user2.tags)
```

Defensive copying can help:

```python
def set_tags(self, tags):
    self.tags = list(tags)
```

But copying everywhere is not a design strategy.

Understand ownership.

Ask:

```text
who is allowed to mutate this object?
```

Mutability bugs are reference bugs.

Python names point to objects.

Objects may be shared.

---

# Debugging Descriptors and Properties

Properties and descriptors can hide computation behind attribute access.

Example:

```python
user.display_name
```

may call:

```python
User.display_name.__get__(user, User)
```

If attribute access behaves unexpectedly, inspect the class:

```python
print(type(user))
print(type(user).__dict__.get("display_name"))
```

Ask:

* is this a property?
* is this a descriptor?
* is `__getattr__` involved?
* is `__getattribute__` customized?
* is a dataclass field shadowing something?

Descriptors were not an isolated advanced topic.

They affect debugging whenever attribute access is not plain storage.

---

# Debugging Decorators

Decorators wrap functions.

That can complicate debugging.

Problems include:

* wrapper hides original function name
* arguments are changed
* exceptions are swallowed
* return value is transformed
* metadata is lost
* async function wrapped incorrectly

Good decorators use `functools.wraps`:

```python
from functools import wraps


def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("calling", func.__name__)
        return func(*args, **kwargs)
    return wrapper
```

Without `wraps`, tracebacks and introspection can become less helpful.

When debugging decorated functions, remember:

```text
the function being called may be the wrapper
```

Inspect:

```python
print(function)
print(getattr(function, "__wrapped__", None))
```

---

# Debugging Context Managers

Context managers can hide setup and cleanup.

If behavior changes inside a `with` block, inspect:

* `__enter__`
* the block body
* `__exit__`
* whether exceptions are suppressed

Important detail:

```python
def __exit__(self, exc_type, exc, traceback):
    return True
```

Returning `True` suppresses the exception.

If errors disappear mysteriously inside a context manager, check `__exit__`.

Context managers are powerful because they centralize cleanup.

They are tricky because control flow passes through protocol methods.

---

# Debugging Generators

Generators run lazily.

This causes surprises.

Example:

```python
values = (parse(row) for row in rows)
```

No parsing happens yet.

Parsing happens when the generator is consumed.

If an exception arises later, the cause may be inside generator code defined earlier.

To debug:

```python
for value in values:
    print(repr(value))
```

or temporarily materialize:

```python
values = list(values)
```

Be careful materializing huge generators.

The bug may depend on laziness.

Generators also get exhausted.

If output is empty on second use, check whether the generator was already consumed.

---

# Debugging Subprocesses

Subprocess bugs involve another program.

Always inspect:

* command arguments
* exit code
* stdout
* stderr
* working directory
* environment
* timeout

Example:

```python
import subprocess


result = subprocess.run(
    ["python", "-m", "tool"],
    capture_output=True,
    text=True,
    check=False,
)

print(result.returncode)
print(result.stdout)
print(result.stderr)
```

If `check=True`, Python raises an exception for nonzero exit codes.

That can be useful.

But while debugging, capturing output explicitly often gives more evidence.

Subprocesses have their own environment.

Do not assume they see the same paths and variables you see.

---

# Debugging Path Problems

Path bugs often come from assumptions about the current working directory.

Bad:

```python
Path("config/settings.toml").read_text()
```

This depends on where the process was started.

Better:

```python
BASE_DIR = Path(__file__).resolve().parent
config_path = BASE_DIR / "config" / "settings.toml"
```

When debugging paths, print:

```python
print(Path.cwd())
print(path)
print(path.resolve())
print(path.exists())
```

Also check permissions.

A path can exist and still be unreadable.

---

# Debugging Unicode and Encoding

Text bugs often involve encoding.

Symptoms:

* `UnicodeDecodeError`
* replacement characters
* corrupted output
* length mismatches
* sorting surprises
* normalization differences

Inspect:

```python
print(repr(text))
print(text.encode("utf-8"))
```

When reading files, specify encoding:

```python
path.read_text(encoding="utf-8")
```

When handling user input, remember that visually identical text can have different Unicode representations.

For example, accented characters may be precomposed or built from combining marks.

If exact comparison fails mysteriously, investigate normalization with `unicodedata`.

Text is data.

Data has representation.

---

# Debugging With IDEs

Many developers use IDE debuggers.

Common features include:

* visual breakpoints
* conditional breakpoints
* variable watches
* call stack navigation
* step into
* step over
* step out
* exception breakpoints
* remote debugging

The concepts are the same as `pdb`.

The interface is different.

Do not confuse tool fluency with debugging skill.

An IDE can make inspection easier.

It cannot decide what hypothesis to test.

The debugging process still belongs to you.

---

# Debugging in Notebooks

Notebook debugging has special issues.

Cells can run out of order.

State persists between cells.

Variables may exist even if the cell that creates them is no longer visible.

Imports may refer to old code until reloaded.

When debugging notebooks:

* restart the kernel
* run cells from top to bottom
* reduce state
* move reusable code into modules
* test module code outside the notebook

Notebooks are excellent for exploration.

They can be confusing for reproducibility.

If a notebook bug disappears after restart, hidden state was likely involved.

---

# Debugging Native Extension Crashes

Pure Python errors usually raise exceptions.

Native extension errors can crash the interpreter.

Symptoms:

* segmentation fault
* illegal instruction
* abort
* process exits without Python traceback
* memory corruption

Use:

```bash
python -X faulthandler script.py
```

Also isolate:

* which extension is involved?
* which input triggers the crash?
* does it happen on another Python version?
* does it happen on another platform?
* did a wheel or dependency recently change?
* is the extension compatible with this Python version?

Native debugging may require tools outside Python, such as platform debuggers and sanitizers.

But even then, Python-level reduction helps.

Find the smallest Python call that crashes.

---

# Fix the Cause, Not the Symptom

Debugging should end with a cause.

Then fix the cause.

Example symptom:

```text
KeyError: 'quantity'
```

Bad fix:

```python
quantity = item.get("quantity", 1)
```

This might be correct.

Or it might hide invalid data.

Better debugging asks:

```text
is missing quantity valid?
```

If yes, defaulting to `1` is a domain rule and should be tested.

If no, validation should reject the item earlier with a clear error.

A good fix aligns with the intended behavior.

It does not merely silence the exception.

---

# Add a Regression Test

After fixing a bug, add a regression test.

The test should fail before the fix and pass after the fix.

Example:

```python
def test_discounted_item_quantity_is_applied():
    items = [
        {"price": 100, "quantity": 2, "discount": 10},
    ]

    assert total(items) == 180
```

This test records the bug's lesson.

Without it, a future refactor may reintroduce the same mistake.

Bug fixes without tests are incomplete when the behavior can reasonably be tested.

The test is the memory of the debugging session.

---

# Write Down the Explanation

For significant bugs, write a short explanation.

It may go in:

* commit message
* pull request description
* issue tracker
* incident report
* code comment when truly needed

Useful explanation:

```text
Discounted items used a separate calculation path that multiplied price by discount
but forgot quantity. Added regression coverage for quantity > 1 with discount.
```

Weak explanation:

```text
Fix bug.
```

The explanation helps future maintainers understand why the change exists.

Debugging creates knowledge.

Do not throw it away.

---

# A Debugging Checklist

When debugging, ask:

* What exactly is the symptom?
* Can I reproduce it?
* What is the smallest reproduction?
* What changed recently?
* What does the traceback say?
* Which frames belong to my code?
* What assumptions does the failing code make?
* Which assumption is false?
* What data shape is actually present?
* What branch actually executed?
* What state changed unexpectedly?
* Is this environment-specific?
* Is this version-specific?
* Is time, randomness, concurrency, or global state involved?
* What hypothesis am I testing?
* What observation would disprove it?
* Did the fix address the cause?
* Is there a regression test?

You will not ask every question every time.

But the checklist keeps you honest when the bug is slippery.

---

# Common Debugging Mistakes

Common mistakes include:

* changing code before reproducing the bug
* ignoring the error message
* reading only the last line of a traceback
* assuming the symptom is the cause
* changing several things at once
* hiding exceptions with broad handlers
* replacing clear failures with silent defaults
* debugging stale code or wrong environment
* forgetting test data setup
* trusting mocks too much
* ignoring warnings
* leaving debug prints in production code
* failing to add a regression test
* fixing the example but not the general rule

These mistakes are normal.

The goal is to recognize them sooner.

Debugging skill grows through deliberate practice.

---

# Chapter Summary

Debugging is the disciplined investigation of incorrect behavior.

Testing reveals that something is wrong.

Debugging explains why.

The core debugging loop is:

```text
observe -> reproduce -> isolate -> explain -> fix -> verify
```

A symptom is the visible failure, not necessarily the root cause.

Reproduction is essential because you need to know whether a fix worked.

Reduction makes failures smaller and easier to reason about.

Error messages and tracebacks are evidence.

Tracebacks show the call stack and should be read as a map, not as a single line.

The `traceback` module helps capture and format traceback information programmatically.

Print debugging is useful for quick inspection, but logging is better for durable diagnostics.

`breakpoint()` and `pdb` let you pause execution, inspect state, move through stack frames, and step through code.

Post-mortem debugging lets you inspect state after an exception.

Assertions can expose violated assumptions, but they should not replace production validation.

Good debugging uses hypotheses and tests one idea at a time.

Binary search helps locate bugs across commits, data, configuration, and other large search spaces.

Many bugs are data-flow, control-flow, or state bugs.

Python-specific debugging often involves `None`, imports, mutability, identity, descriptors, decorators, generators, context managers, and dynamic typing.

Concurrency, async code, memory growth, and native extension crashes require extra care.

Production debugging requires protecting users, preserving evidence, and avoiding random changes.

Fixes should address causes, not only symptoms.

Every fixed bug that can be tested should leave behind a regression test.

The central lesson is:

```text
debugging is the art of turning surprise into understanding
```

Once you understand the cause, the fix becomes smaller, safer, and easier to trust.

---

# Exercises

1. Take a failing test and write down the exact symptom before changing code.

2. Create a minimal reproduction for a bug involving `None`.

3. Use `breakpoint()` inside a function and inspect local variables with `pdb`.

4. Run a script with `python -m pdb` and set a breakpoint before continuing.

5. Debug a `KeyError` without immediately replacing indexing with `.get()`. Decide whether the key is required.

6. Create a mutable default argument bug, reproduce it, and fix it.

7. Write a regression test for an off-by-one error.

8. Use `repr` to debug a string containing hidden whitespace.

9. Use `traceback.format_exc()` to capture an exception traceback as text.

10. Take a bug you recently fixed and write a short explanation of the false assumption behind it.

---

# Preview of Chapter 75

Chapter 74 studied debugging.

We learned how to reproduce failures, reduce examples, read tracebacks, inspect state, use `pdb`, form hypotheses, debug Python-specific issues, and turn fixes into regression tests.

Next we study logging.

Logging is one of the main tools that makes debugging possible after code leaves your machine.

In local debugging, you can pause the program and inspect variables.

In production, you often cannot.

You need the program to leave useful evidence as it runs.

Good logging answers:

* what happened?
* when did it happen?
* where did it happen?
* which request or job was involved?
* which user, account, or entity was affected?
* what was the outcome?
* what context helps explain the behavior?

Bad logging creates noise.

Good logging creates observability.

The transition is:

```text
debugging investigates failure
logging preserves evidence for that investigation
```

Chapter 75 will show how to design logs that help humans understand real systems without flooding them with useless text.
