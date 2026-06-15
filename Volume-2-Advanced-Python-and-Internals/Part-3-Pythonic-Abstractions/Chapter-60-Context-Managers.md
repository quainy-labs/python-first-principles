# Chapter 60 — Context Managers

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a context manager is.
* Why the `with` statement exists.
* How `with` relates to `try` and `finally`.
* What `__enter__` does.
* What `__exit__` does.
* What gets assigned after `as`.
* How context managers handle exceptions.
* Why returning `True` from `__exit__` suppresses exceptions.
* Why most `__exit__` methods should return `False` or `None`.
* How files use context management.
* How locks, transactions, timers, and temporary state use context management.
* How to write a class-based context manager.
* How to write a generator-based context manager with `contextlib.contextmanager`.
* How `contextlib.closing`, `suppress`, `nullcontext`, and `ExitStack` fit in.
* How multiple context managers work in one `with` statement.
* How context managers improve resource lifetime design.
* Which context manager mistakes are common.

Chapter 59 studied generators.

Generators produce values over time.

They can also need cleanup.

Context managers solve a related but different problem:

```text
run setup before a block
run cleanup after the block
```

The most familiar example is files:

```python
with open("data.txt") as file:
    text = file.read()
```

This opens the file, gives you the file object, and closes the file when the block ends.

The block may end normally.

The block may end because of an exception.

Either way, the context manager gets a chance to clean up.

That is the central promise of context managers.

---

# The Problem Context Managers Solve

Many programs need this pattern:

```python
resource = acquire_resource()
try:
    use(resource)
finally:
    release_resource(resource)
```

The `finally` block is important.

It runs even if an error happens.

Example:

```python
file = open("data.txt")
try:
    text = file.read()
finally:
    file.close()
```

This is correct.

But it is repetitive.

It also separates the resource-management idea across several lines.

The `with` statement expresses the pattern directly:

```python
with open("data.txt") as file:
    text = file.read()
```

This is shorter.

More importantly, it is clearer:

```text
use this resource for this block
clean it up when the block ends
```

Context managers package setup and cleanup into reusable objects.

---

# The Context Manager Protocol

An object is a context manager if it implements:

```python
__enter__
__exit__
```

Basic shape:

```python
class Manager:
    def __enter__(self):
        ...
        return value

    def __exit__(self, exc_type, exc, traceback):
        ...
        return False
```

`__enter__` runs before the block.

`__exit__` runs after the block.

If the block exits because of an exception, `__exit__` receives exception information.

If the block exits normally, `__exit__` receives:

```python
None, None, None
```

The return value of `__exit__` controls exception suppression.

If `__exit__` returns a true value, the exception is suppressed.

If it returns a false value, the exception propagates.

Most context managers should not suppress exceptions.

So most `__exit__` methods return:

```python
False
```

or simply return nothing, which means `None`, a false value.

---

# A First Context Manager

Example:

```python
class Announce:
    def __enter__(self):
        print("entering")
        return self

    def __exit__(self, exc_type, exc, traceback):
        print("leaving")
```

Use:

```python
with Announce() as manager:
    print("inside")
```

Output:

```text
entering
inside
leaving
```

The sequence is:

```text
create manager
call __enter__
run block
call __exit__
```

The object returned by `__enter__` is assigned to the name after `as`.

In this example:

```python
manager
```

receives:

```python
self
```

because `__enter__` returned `self`.

---

# `as` Receives the Return Value of `__enter__`

This is subtle.

In:

```python
with expression as target:
    ...
```

`target` receives the return value of `__enter__`, not necessarily the context manager object itself.

Example:

```python
class GivesList:
    def __enter__(self):
        return []

    def __exit__(self, exc_type, exc, traceback):
        print("done")
```

Use:

```python
with GivesList() as items:
    items.append("a")
    items.append("b")
    print(items)
```

Output:

```text
['a', 'b']
done
```

The manager is a `GivesList` object.

The `as` target is a list.

This is how `open()` works too.

The context expression creates a file context manager.

The value after `as` is the file object used inside the block.

Often they are the same object.

But they do not have to be.

---

# What `with` Roughly Means

This:

```python
with manager as value:
    body(value)
```

roughly means:

```python
enter = manager.__enter__
exit = manager.__exit__
value = enter()

try:
    body(value)
except:
    if not exit(*exception_info):
        raise
else:
    exit(None, None, None)
```

The real semantics are precise and use special method lookup, but this model is useful.

Important guarantees:

* the context expression is evaluated
* `__enter__` is called
* if `__enter__` succeeds, `__exit__` will be called
* if the block raises, exception details are passed to `__exit__`
* if `__exit__` returns true, the exception is suppressed
* otherwise, the exception continues

The most important practical rule:

```text
if __enter__ finishes successfully, __exit__ gets a chance to run
```

---

# If `__enter__` Fails

If `__enter__` raises an exception, the block does not run.

Also, that context manager's `__exit__` is not called because entering did not complete successfully.

Example:

```python
class BrokenEnter:
    def __enter__(self):
        print("entering")
        raise RuntimeError("cannot enter")

    def __exit__(self, exc_type, exc, traceback):
        print("leaving")
```

Use:

```python
with BrokenEnter():
    print("inside")
```

Output:

```text
entering
```

Then `RuntimeError` propagates.

`inside` does not print.

`leaving` does not print.

If acquisition is partially completed inside `__enter__`, the `__enter__` method itself must clean up before raising.

Context managers must be careful during setup.

---

# Exception Information in `__exit__`

`__exit__` receives three arguments:

```python
exc_type
exc
traceback
```

If no exception occurred:

```python
exc_type is None
exc is None
traceback is None
```

If an exception occurred, they describe it.

Example:

```python
class ShowException:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc, traceback):
        print(f"exc_type = {exc_type}")
        print(f"exc = {exc}")
        return False
```

Use:

```python
with ShowException():
    raise ValueError("bad")
```

Output includes:

```text
exc_type = <class 'ValueError'>
exc = bad
```

Then the `ValueError` continues because `__exit__` returned false.

This lets a context manager log, transform, clean up after, or suppress exceptions.

Suppressing should be rare and deliberate.

---

# Suppressing Exceptions

If `__exit__` returns a true value, Python suppresses the exception.

Example:

```python
class SuppressValueError:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc, traceback):
        return exc_type is ValueError
```

Use:

```python
with SuppressValueError():
    raise ValueError("ignored")

print("continued")
```

Output:

```text
continued
```

The `ValueError` was suppressed.

But:

```python
with SuppressValueError():
    raise TypeError("not ignored")
```

propagates because `__exit__` returns false.

Suppression can be useful.

But silent exception suppression can hide bugs.

If a context manager suppresses exceptions, its name should make that obvious.

---

# Most Context Managers Should Not Suppress

This is the normal pattern:

```python
def __exit__(self, exc_type, exc, traceback):
    cleanup()
    return False
```

or:

```python
def __exit__(self, exc_type, exc, traceback):
    cleanup()
```

Because returning `None` is false.

Files do not suppress exceptions:

```python
with open("data.txt") as file:
    raise ValueError("bad")
```

The file is closed.

The exception still propagates.

That is usually what you want:

```text
cleanup should happen
errors should remain visible
```

Suppress only when the context manager's purpose is suppression or recovery.

---

# Files as Context Managers

The classic context manager is a file:

```python
with open("data.txt") as file:
    data = file.read()
```

This ensures the file is closed when the block ends.

Equivalent idea:

```python
file = open("data.txt")
try:
    data = file.read()
finally:
    file.close()
```

The `with` version is cleaner.

It also makes the lifetime visible:

```text
the file is meant to be used only inside this block
```

After the block:

```python
file.closed
```

is:

```python
True
```

Do not rely on garbage collection to close files.

Use context managers for deterministic cleanup.

---

# Locks as Context Managers

Locks are another common example.

Without `with`:

```python
lock.acquire()
try:
    update_shared_state()
finally:
    lock.release()
```

With `with`:

```python
with lock:
    update_shared_state()
```

The lock is acquired before the block and released after the block.

This matters because exceptions should not leave locks held.

Context managers are excellent for paired operations:

```text
acquire / release
open / close
begin / commit-or-rollback
set / restore
enter / leave
```

When you see paired lifecycle operations, consider a context manager.

---

# Transactions as Context Managers

Database transactions often fit context managers.

Conceptual example:

```python
with database.transaction() as transaction:
    transaction.insert(user)
    transaction.insert(profile)
```

Possible behavior:

* begin transaction in `__enter__`
* commit if block succeeds
* roll back if block raises
* close transaction resources in `__exit__`

Simplified class:

```python
class Transaction:
    def __init__(self, connection):
        self.connection = connection

    def __enter__(self):
        self.connection.begin()
        return self

    def __exit__(self, exc_type, exc, traceback):
        if exc_type is None:
            self.connection.commit()
        else:
            self.connection.rollback()
        return False
```

The exception is not suppressed.

Rollback happens.

Then the error continues.

This is usually correct.

The context manager protects consistency without hiding failure.

---

# Temporary State

Context managers are useful for temporary changes.

Example:

```python
class TemporaryValue:
    def __init__(self, obj, name, value):
        self.obj = obj
        self.name = name
        self.value = value

    def __enter__(self):
        self.old_value = getattr(self.obj, self.name)
        setattr(self.obj, self.name, self.value)
        return self

    def __exit__(self, exc_type, exc, traceback):
        setattr(self.obj, self.name, self.old_value)
```

Use:

```python
settings.debug = False

with TemporaryValue(settings, "debug", True):
    run_debug_code()

print(settings.debug)
```

The old value is restored even if `run_debug_code()` raises.

This pattern appears in:

* tests
* configuration overrides
* environment changes
* warning filters
* decimal precision
* temporary working directories

Temporary changes should be restored reliably.

Context managers make the lifetime explicit.

---

# Timers as Context Managers

Timing a block is a good context manager example.

```python
from time import perf_counter


class Timer:
    def __enter__(self):
        self.start = perf_counter()
        return self

    def __exit__(self, exc_type, exc, traceback):
        self.end = perf_counter()
        self.elapsed = self.end - self.start
```

Use:

```python
with Timer() as timer:
    total = sum(range(1_000_000))

print(timer.elapsed)
```

`__enter__` records the start.

`__exit__` records the end.

The `timer` object remains available after the block.

This is a useful pattern:

```text
return self from __enter__ when the caller should inspect manager state
```

---

# Returning a Different Object from `__enter__`

Sometimes the manager should return something else.

Example:

```python
class ManagedList:
    def __init__(self):
        self.items = []

    def __enter__(self):
        return self.items

    def __exit__(self, exc_type, exc, traceback):
        print(f"collected {len(self.items)} items")
```

Use:

```python
with ManagedList() as items:
    items.append("a")
    items.append("b")
```

The `as` variable is the list.

The manager still exists behind the scenes.

This design is good when the block should work with a resource rather than the manager wrapper.

Files work this way conceptually.

The block wants the file object.

The context manager handles lifecycle.

---

# Multiple Context Managers

You can use multiple context managers in one `with` statement:

```python
with open("input.txt") as source, open("output.txt", "w") as target:
    target.write(source.read())
```

This is equivalent to nesting:

```python
with open("input.txt") as source:
    with open("output.txt", "w") as target:
        target.write(source.read())
```

Entering happens left to right.

Exiting happens right to left.

That mirrors nested blocks.

If the second context manager fails to enter, the first one is exited.

This is one reason multiple context managers in one line are safe for common paired resources.

Use parentheses for readability when the line is long:

```python
with (
    open("input.txt") as source,
    open("output.txt", "w") as target,
):
    target.write(source.read())
```

---

# Context Managers and `return`

`__exit__` runs when leaving the block, even if the block uses `return`.

Example:

```python
def read_first_line(path):
    with open(path) as file:
        return file.readline()
```

The file is still closed.

This matters because block exit can happen through:

* normal completion
* exception
* `return`
* `break`
* `continue`

The context manager gets its exit call.

This is the same spirit as `finally`.

The block's control flow does not skip cleanup.

---

# Context Managers and Special Method Lookup

The `with` statement uses special method lookup for `__enter__` and `__exit__`.

This is similar to other dunder protocols.

That means defining `__enter__` on an individual instance in a casual way is not the normal route.

Define context manager methods on the class:

```python
class Manager:
    def __enter__(self):
        ...

    def __exit__(self, exc_type, exc, traceback):
        ...
```

This follows the data model pattern from Chapter 52.

Protocol methods belong on the type.

Objects then participate in language syntax.

---

# `contextlib.contextmanager`

Class-based context managers are explicit.

But simple context managers can be written with a generator and `contextlib.contextmanager`.

Example:

```python
from contextlib import contextmanager


@contextmanager
def announce():
    print("entering")
    try:
        yield
    finally:
        print("leaving")
```

Use:

```python
with announce():
    print("inside")
```

Output:

```text
entering
inside
leaving
```

The code before `yield` is setup.

The yielded value is assigned after `as`, if any.

The code after `yield` is cleanup.

The `try/finally` ensures cleanup runs when exceptions occur.

This connects directly to Chapter 59.

---

# Yielding a Value from a Generator Context Manager

Example:

```python
from contextlib import contextmanager


@contextmanager
def managed_list():
    items = []
    try:
        yield items
    finally:
        print(f"collected {len(items)} items")
```

Use:

```python
with managed_list() as items:
    items.append("a")
    items.append("b")
```

The yielded `items` list is assigned to the `as` target.

This:

```python
yield items
```

plays the role of `__enter__` returning a value.

The code after `yield` plays the role of `__exit__`.

Generator-based context managers are concise for simple setup/cleanup.

Class-based context managers are better when behavior needs more structure or multiple methods.

---

# Generator Context Manager Must Yield Once

A function decorated with `@contextmanager` must yield exactly once.

This is correct:

```python
@contextmanager
def manager():
    setup()
    try:
        yield resource
    finally:
        cleanup()
```

This is wrong:

```python
@contextmanager
def manager():
    setup()
    cleanup()
```

It never yields.

This is also wrong:

```python
@contextmanager
def manager():
    yield "first"
    yield "second"
```

A context manager manages one block.

It enters once and exits once.

The generator should yield once at the boundary between setup and cleanup.

---

# Exception Handling in Generator Context Managers

If the `with` block raises, the exception is thrown into the generator at the `yield` point.

Example:

```python
@contextmanager
def show_error():
    try:
        yield
    except ValueError as error:
        print(f"saw value error: {error}")
        raise
```

Use:

```python
with show_error():
    raise ValueError("bad")
```

The generator catches the exception, prints, and re-raises.

If a generator context manager catches an exception and does not re-raise it, the exception may be suppressed.

That can be surprising.

So write exception handling carefully.

If you only need cleanup, prefer:

```python
try:
    yield
finally:
    cleanup()
```

This does not suppress exceptions.

---

# `contextlib.suppress`

`contextlib.suppress` intentionally suppresses specified exceptions.

Example:

```python
from contextlib import suppress


with suppress(FileNotFoundError):
    Path("missing.txt").unlink()
```

This says:

```text
ignore FileNotFoundError in this block
```

This is better than a vague bare `except`.

It names the suppression at the block level.

Use it sparingly.

Good use:

```text
ignore a specific harmless missing-file condition
```

Bad use:

```python
with suppress(Exception):
    risky_operation()
```

That can hide real bugs.

Suppression should be narrow and intentional.

---

# `contextlib.closing`

Some objects have a `close()` method but do not implement context management.

`contextlib.closing` adapts them:

```python
from contextlib import closing


with closing(resource) as value:
    use(value)
```

At exit, it calls:

```python
value.close()
```

This is useful for older APIs or third-party objects that follow a close pattern without supporting `with`.

If you control the class, it is usually better to implement `__enter__` and `__exit__` directly.

But `closing` is a practical adapter.

---

# `contextlib.nullcontext`

`nullcontext` is a context manager that does almost nothing.

It is useful when code sometimes needs a real context manager and sometimes does not.

Example:

```python
from contextlib import nullcontext


def read_text(source):
    if isinstance(source, str):
        manager = open(source)
    else:
        manager = nullcontext(source)

    with manager as file:
        return file.read()
```

If `source` is a path, open it.

If `source` is already a file-like object, use it as-is.

`nullcontext(value)` returns `value` from `__enter__`.

This avoids awkward branching around the whole block.

---

# `contextlib.ExitStack`

`ExitStack` handles dynamic numbers of context managers.

Example:

```python
from contextlib import ExitStack


def read_all(paths):
    with ExitStack() as stack:
        files = [
            stack.enter_context(open(path))
            for path in paths
        ]
        return [file.read() for file in files]
```

Why not write:

```python
with open(a) as first, open(b) as second:
    ...
```

Because the number of paths may be dynamic.

`ExitStack` lets you enter context managers programmatically.

It exits them in reverse order.

It is useful for:

* dynamic resource lists
* optional contexts
* complex setup where partial failure must clean up
* framework code

Do not use `ExitStack` for simple fixed cases.

Use normal `with` when you can.

---

# Context Managers as Decorators

Some context manager helpers can also act as decorators.

For example, context managers built with `ContextDecorator` behavior can wrap an entire function.

Conceptually:

```python
@some_context()
def function():
    ...
```

can mean:

```python
def function():
    with some_context():
        ...
```

This is useful for cross-cutting concerns like timing or temporary settings.

But be careful.

Decorators apply to the whole function.

Context managers show the exact block.

Prefer the `with` statement when a smaller block is clearer.

---

# Context Managers and Transactions

Transaction context managers deserve special care.

Simplified:

```python
class Transaction:
    def __enter__(self):
        self.begin()
        return self

    def __exit__(self, exc_type, exc, traceback):
        if exc_type is None:
            self.commit()
        else:
            self.rollback()
        return False
```

This means:

```text
success -> commit
exception -> rollback and propagate error
```

That is usually good.

But real transactions may have subtleties:

* nested transactions
* savepoints
* connection pooling
* retry behavior
* specific exception classes
* commit failures
* rollback failures

Context managers provide the shape.

They do not remove domain complexity.

Design transactional behavior explicitly.

---

# Context Managers and Tests

Tests often use context managers.

Examples:

```python
with pytest.raises(ValueError):
    Product(-1)
```

This context manager expects an exception.

If the exception happens, it suppresses it and lets the test pass.

If the exception does not happen, the test fails.

Another example:

```python
with tempfile.TemporaryDirectory() as directory:
    path = Path(directory) / "data.txt"
    path.write_text("hello")
```

The temporary directory is cleaned up after the block.

Tests benefit from context managers because test setup must be cleaned up reliably.

---

# Context Managers and Global State

Global state changes should be temporary and well-scoped.

Example:

```python
class ChangeDirectory:
    def __init__(self, path):
        self.path = path

    def __enter__(self):
        self.old_path = Path.cwd()
        os.chdir(self.path)
        return Path.cwd()

    def __exit__(self, exc_type, exc, traceback):
        os.chdir(self.old_path)
```

Use:

```python
with ChangeDirectory(project_dir):
    run_build()
```

The working directory is restored even if `run_build()` fails.

This is much safer than:

```python
os.chdir(project_dir)
run_build()
os.chdir(old_dir)
```

because the last line can be skipped by exceptions.

Global state changes should almost always use `try/finally` or a context manager.

---

# Context Manager Reuse

Some context manager objects are one-use.

Some can be reused.

Example:

```python
manager = open("data.txt")
```

You should not generally enter the same file object multiple times after it has closed.

Generator-based context managers created by `@contextmanager` are also one-shot objects.

This is wrong:

```python
manager = announce()

with manager:
    ...

with manager:
    ...
```

Create a fresh manager:

```python
with announce():
    ...

with announce():
    ...
```

If you design class-based context managers, decide whether instances are reusable.

If not, document that or make misuse fail clearly.

---

# Reentrant Context Managers

A reentrant context manager can be entered multiple times, even nested.

Example concept:

```python
with manager:
    with manager:
        ...
```

This requires careful state management.

Locks have reentrant and non-reentrant variants.

A normal lock may block or fail if entered twice by the same thread.

A reentrant lock tracks entry count.

For your own context managers, do not assume reentrancy.

If nested use should work, design for it.

If nested use should not work, fail clearly.

Stateful context managers are easy to get wrong when reentered accidentally.

---

# Asynchronous Context Managers

Python also has asynchronous context managers.

They use:

```python
async with
```

and methods:

```python
__aenter__
__aexit__
```

Example shape:

```python
async with client.session() as session:
    ...
```

This chapter focuses on synchronous context managers.

Async context managers belong with asyncio and asynchronous programming later in the book.

The conceptual parallel is:

```text
with       -> __enter__ / __exit__
async with -> __aenter__ / __aexit__
```

The async version allows setup and cleanup to await asynchronous operations.

---

# Common Mistake: Forgetting to Return the Resource

This context manager is awkward:

```python
class OpenFile:
    def __init__(self, path):
        self.path = path

    def __enter__(self):
        self.file = open(self.path)

    def __exit__(self, exc_type, exc, traceback):
        self.file.close()
```

Use:

```python
with OpenFile("data.txt") as file:
    file.read()
```

`file` is `None` because `__enter__` returned nothing.

Fix:

```python
def __enter__(self):
    self.file = open(self.path)
    return self.file
```

Return the object the block should use.

That may be `self`.

It may be an internal resource.

But return it intentionally.

---

# Common Mistake: Suppressing Exceptions Accidentally

This suppresses every exception:

```python
def __exit__(self, exc_type, exc, traceback):
    cleanup()
    return True
```

That is dangerous.

The block can fail and the caller may never know.

Usually:

```python
def __exit__(self, exc_type, exc, traceback):
    cleanup()
    return False
```

or:

```python
def __exit__(self, exc_type, exc, traceback):
    cleanup()
```

Only return `True` when suppression is the explicit purpose.

Names like `suppress`, `ignore_missing`, or `expect_error` make suppression visible.

---

# Common Mistake: Cleanup That Can Hide the Original Error

If cleanup raises an exception, it can obscure the original exception.

Example:

```python
def __exit__(self, exc_type, exc, traceback):
    self.cleanup_that_may_fail()
```

If the block raised `ValueError` and cleanup raises `RuntimeError`, debugging becomes harder.

Sometimes cleanup failure must propagate.

Sometimes it should be logged while preserving the original error.

Design carefully.

For critical resources, cleanup errors matter.

For best-effort cleanup, you may need narrow exception handling.

Avoid broad silent suppression.

---

# Common Mistake: Doing Too Much in `__enter__`

`__enter__` can raise.

If setup has multiple steps, partial cleanup is your responsibility.

Example:

```python
def __enter__(self):
    self.first = acquire_first()
    self.second = acquire_second()
    return self
```

If `acquire_second()` fails, `__exit__` is not called.

So `first` must be cleaned up inside `__enter__`:

```python
def __enter__(self):
    self.first = acquire_first()
    try:
        self.second = acquire_second()
    except Exception:
        release_first(self.first)
        raise
    return self
```

For complex dynamic setup, `ExitStack` can help.

The guarantee is:

```text
if __enter__ succeeds, __exit__ runs
```

It is not:

```text
__exit__ always runs no matter how __enter__ fails
```

---

# Common Mistake: Using Context Managers for Unclear Scope

This is clear:

```python
with open(path) as file:
    data = file.read()
```

This may be unclear:

```python
with user:
    ...
```

What does entering a user mean?

Does it log in?

Lock the user?

Start a transaction?

Change permissions?

Context managers should make scope meaningful.

Good names help:

```python
with user.impersonation():
    ...
```

or:

```python
with locked(user):
    ...
```

If the meaning of entering and exiting is not obvious, use a named method or helper that explains it.

---

# Common Mistake: Using a Context Manager Instead of a Function

Do not use `with` just to look sophisticated.

This is unnecessary:

```python
with CalculateTotal(order) as total:
    print(total)
```

If there is no setup/cleanup boundary, use a function:

```python
total = calculate_total(order)
```

Context managers are for scoped lifecycle.

They are not a replacement for every helper function.

Ask:

```text
what must be true before the block?
what must be restored or cleaned up after the block?
```

If there is no good answer, a context manager may not be the right abstraction.

---

# Common Mistake: Hiding Long-Running Work in `__enter__`

This can surprise users:

```python
with RemoteSession() as session:
    ...
```

if `__enter__` performs slow network setup.

It may still be a good design.

But document it.

Context manager entry can do real work.

Users should understand whether entering:

* opens a file
* connects to a server
* starts a transaction
* acquires a lock
* changes global state
* allocates a large resource

The `with` statement makes scope clear.

It does not make expensive work obvious by itself.

Good names and documentation matter.

---

# A Full Example: Temporary Environment Variable

```python
import os


class temporary_env:
    def __init__(self, name, value):
        self.name = name
        self.value = value

    def __enter__(self):
        self.old_exists = self.name in os.environ
        self.old_value = os.environ.get(self.name)
        os.environ[self.name] = self.value
        return self

    def __exit__(self, exc_type, exc, traceback):
        if self.old_exists:
            os.environ[self.name] = self.old_value
        else:
            os.environ.pop(self.name, None)
```

Use:

```python
with temporary_env("APP_MODE", "test"):
    run_tests()
```

After the block, the environment is restored.

This is better than setting the variable manually and hoping every exit path restores it.

---

# A Full Example: Capturing Output

```python
from io import StringIO
import sys


class capture_stdout:
    def __enter__(self):
        self.old_stdout = sys.stdout
        self.buffer = StringIO()
        sys.stdout = self.buffer
        return self.buffer

    def __exit__(self, exc_type, exc, traceback):
        sys.stdout = self.old_stdout
```

Use:

```python
with capture_stdout() as output:
    print("hello")

assert output.getvalue() == "hello\n"
```

This is useful in tests.

But it changes global state.

The context manager makes the change temporary.

Still, be careful in threaded programs where global output is shared.

---

# A Full Example: Timer with Logging

```python
from time import perf_counter


class log_time:
    def __init__(self, label):
        self.label = label

    def __enter__(self):
        self.start = perf_counter()
        return self

    def __exit__(self, exc_type, exc, traceback):
        elapsed = perf_counter() - self.start
        print(f"{self.label}: {elapsed:.3f}s")
```

Use:

```python
with log_time("build index"):
    build_index()
```

This does not suppress exceptions.

If `build_index()` fails, the time is still logged and the exception continues.

That is usually the right behavior for timing.

---

# A Full Example with `@contextmanager`

Class version:

```python
class temporary_value:
    def __init__(self, obj, name, value):
        self.obj = obj
        self.name = name
        self.value = value

    def __enter__(self):
        self.old_value = getattr(self.obj, self.name)
        setattr(self.obj, self.name, self.value)

    def __exit__(self, exc_type, exc, traceback):
        setattr(self.obj, self.name, self.old_value)
```

Generator version:

```python
from contextlib import contextmanager


@contextmanager
def temporary_value(obj, name, value):
    old_value = getattr(obj, name)
    setattr(obj, name, value)
    try:
        yield
    finally:
        setattr(obj, name, old_value)
```

Use:

```python
with temporary_value(settings, "debug", True):
    run_debug_code()
```

The generator version is concise.

The class version may be better if you need:

* reusable manager objects
* several methods
* richer state inspection
* inheritance
* more explicit control

Both are valid.

---

# Choosing Class-Based or Generator-Based

Use a class-based context manager when:

* the manager has meaningful state
* the manager needs multiple methods
* the manager may be reused carefully
* inheritance or composition matters
* setup and cleanup are complex
* you want explicit protocol methods

Use `@contextmanager` when:

* setup is simple
* cleanup is simple
* one `yield` expresses the resource boundary
* a small helper would be clearer than a full class

Example good generator context manager:

```python
@contextmanager
def changed_directory(path):
    old = Path.cwd()
    os.chdir(path)
    try:
        yield
    finally:
        os.chdir(old)
```

Example good class context manager:

```python
class Transaction:
    ...
```

where transaction state and methods matter.

---

# Design Checklist

Before writing a context manager, ask:

```text
What is acquired before the block?
```

If nothing, maybe you need a function.

Ask:

```text
What must be released, restored, committed, or rolled back after the block?
```

That is the cleanup responsibility.

Ask:

```text
What should the as target receive?
```

Return that from `__enter__` or yield it from `@contextmanager`.

Ask:

```text
Should exceptions be suppressed?
```

Usually no.

Ask:

```text
Can setup fail halfway?
```

If yes, handle partial cleanup in `__enter__`.

Ask:

```text
Is this manager reusable or one-shot?
```

Document or enforce the answer.

Ask:

```text
Would contextlib already solve this?
```

Check `suppress`, `closing`, `nullcontext`, `ExitStack`, and `contextmanager`.

---

# Practice: Simple Class Context Manager

Write a context manager that prints before and after a block.

Solution:

```python
class announce:
    def __enter__(self):
        print("start")
        return self

    def __exit__(self, exc_type, exc, traceback):
        print("end")
```

Use:

```python
with announce():
    print("inside")
```

Expected:

```text
start
inside
end
```

---

# Practice: Timer

Write a timer context manager.

Solution:

```python
from time import perf_counter


class Timer:
    def __enter__(self):
        self.start = perf_counter()
        return self

    def __exit__(self, exc_type, exc, traceback):
        self.end = perf_counter()
        self.elapsed = self.end - self.start
```

Use:

```python
with Timer() as timer:
    sum(range(1000))

assert timer.elapsed >= 0
```

The `as` target receives the timer object because `__enter__` returns `self`.

---

# Practice: Suppress One Exception

Write a context manager that suppresses `FileNotFoundError`.

Solution:

```python
class suppress_file_not_found:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc, traceback):
        return exc_type is FileNotFoundError
```

Use:

```python
with suppress_file_not_found():
    Path("missing.txt").unlink()
```

In real code, prefer:

```python
from contextlib import suppress


with suppress(FileNotFoundError):
    Path("missing.txt").unlink()
```

The standard-library version is clearer.

---

# Practice: Generator Context Manager

Write a generator-based context manager that temporarily sets an attribute.

Solution:

```python
from contextlib import contextmanager


@contextmanager
def temporary_attr(obj, name, value):
    old_value = getattr(obj, name)
    setattr(obj, name, value)
    try:
        yield
    finally:
        setattr(obj, name, old_value)
```

Use:

```python
with temporary_attr(settings, "debug", True):
    assert settings.debug is True
```

After the block, the old value is restored.

---

# Practice: Multiple Files

Use multiple context managers to copy text from one file to another.

Solution:

```python
with open("input.txt") as source, open("output.txt", "w") as target:
    target.write(source.read())
```

Equivalent nested form:

```python
with open("input.txt") as source:
    with open("output.txt", "w") as target:
        target.write(source.read())
```

Entering happens left to right.

Exiting happens right to left.

---

# Practice: Spot the Bug

What is wrong?

```python
class Manager:
    def __enter__(self):
        self.resource = acquire()

    def __exit__(self, exc_type, exc, traceback):
        self.resource.close()
```

Answer:

`__enter__` does not return the resource or `self`.

So:

```python
with Manager() as resource:
    ...
```

assigns `None` to `resource`.

Fix:

```python
def __enter__(self):
    self.resource = acquire()
    return self.resource
```

or:

```python
return self
```

depending on what the block should use.

---

# Practice: Should This Be a Context Manager?

Decide whether each fits a context manager:

```text
Open and close a file
Calculate a total
Acquire and release a lock
Temporarily change working directory
Parse a date string
Begin and commit/rollback a transaction
Measure time for a block
Validate an email address
```

Likely answers:

```text
Open and close a file -> yes
Calculate a total -> no, use function
Acquire and release a lock -> yes
Temporarily change working directory -> yes
Parse a date string -> no, use function/classmethod
Transaction -> yes
Measure time for a block -> yes
Validate email -> no, use function/property/validator
```

The pattern:

```text
scoped lifecycle -> context manager
plain computation -> function or method
```

---

# Summary

Context managers package setup and cleanup around a block of code.

The `with` statement uses the context manager protocol.

A context manager implements `__enter__` and `__exit__`.

`__enter__` runs before the block.

Its return value is assigned to the name after `as`.

`__exit__` runs after the block if `__enter__` succeeded.

If the block raised an exception, `__exit__` receives exception information.

If the block exited normally, `__exit__` receives three `None` values.

Returning a true value from `__exit__` suppresses the exception.

Most context managers should not suppress exceptions.

Files, locks, transactions, timers, temporary settings, and test helpers are natural context managers.

`contextlib.contextmanager` lets a generator function define a context manager with setup before `yield` and cleanup after `yield`.

`contextlib.suppress`, `closing`, `nullcontext`, and `ExitStack` provide useful standard tools.

The design principle is:

```text
use context managers for scoped lifecycle, not ordinary computation
```

When a block needs a guarantee that something will be cleaned up, restored, released, committed, rolled back, or measured, a context manager may be the right abstraction.

---

# Preview of Chapter 61

Chapter 60 studied context managers as a Pythonic abstraction for scoped setup and cleanup.

Next we study decorators.

Decorators modify or wrap functions, methods, and classes.

They are used for:

* logging
* timing
* caching
* validation
* registration
* permissions
* retries
* framework routes
* test markers
* class transformation

Chapter 61 will explain:

* what decorator syntax means
* how functions can wrap other functions
* why closures matter for decorators
* how to preserve metadata with `functools.wraps`
* how decorators with arguments work
* how class decorators work
* when decorators improve design
* when decorators hide too much

The transition is:

```text
context managers wrap a block of execution
decorators wrap callable or class definitions
```

Both abstractions let us factor repeated structure out of business logic.

Used carefully, they make code cleaner.

Used carelessly, they make behavior harder to see.

