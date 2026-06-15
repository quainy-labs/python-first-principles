# Chapter 62 — Exceptions

---

# Learning Objectives

By the end of this chapter, you should understand:

* What exceptions are.
* Why Python uses exceptions.
* How exception propagation works.
* How `try` and `except` work.
* How `else` fits into exception handling.
* How `finally` fits into cleanup.
* How `raise` works.
* How to re-raise an active exception.
* How exception hierarchy affects matching.
* Why `BaseException` is not usually what you catch.
* Why `Exception` is the normal application-level base.
* How to choose built-in exception types.
* How to write custom exception classes.
* How exception chaining works with `raise ... from ...`.
* How to suppress exception context with `from None`.
* What exception groups and `except*` are for.
* How context managers and exceptions interact.
* How decorators can accidentally hide exceptions.
* When to catch exceptions.
* When to let exceptions propagate.
* How to design robust error boundaries.

Chapter 61 closed Part III with decorators.

Now we begin Part IV: Robust Programs and I/O.

Robust programs do not pretend errors will never happen.

They decide:

```text
which errors can happen
where they should be handled
what context should be preserved
what the caller should see
what cleanup must always happen
```

Exceptions are Python's main mechanism for abnormal control flow.

You have already seen them throughout the book:

* `ValueError` for invalid values
* `TypeError` for wrong types
* `AttributeError` for missing attributes
* `KeyError` for missing dictionary keys
* `IndexError` for invalid sequence indexes
* `StopIteration` for iterator exhaustion
* `FileNotFoundError` for missing files
* `ImportError` and `ModuleNotFoundError` for import failures

This chapter studies exceptions directly.

Not just the syntax.

The design.

---

# What Is an Exception?

An exception is an object that represents an exceptional condition.

Example:

```python
int("abc")
```

raises:

```python
ValueError
```

The string `"abc"` cannot be converted to an integer.

Python does not return a special value like:

```python
None
```

or:

```python
False
```

It raises an exception.

That exception interrupts the normal flow of execution.

Python then looks for code that can handle it.

If no handler is found, the program reports the exception and stops the current top-level execution.

The mental model:

```text
normal result -> return
abnormal condition -> raise
```

---

# Why Exceptions Exist

Without exceptions, every operation that might fail would need manual checking.

Example:

```python
result = parse_integer(text)
if result.failed:
    handle_error(result.error)
else:
    use(result.value)
```

That style can work.

Some languages use it heavily.

Python usually prefers:

```python
try:
    value = int(text)
except ValueError:
    handle_invalid_integer(text)
else:
    use(value)
```

The operation either returns a value or raises an exception.

This keeps normal logic cleaner.

It also lets errors propagate to the level that can actually handle them.

Not every function can decide what to do about an error.

Sometimes the correct behavior is:

```text
let the caller decide
```

Exceptions support that.

---

# Exception Propagation

When an exception is raised, Python looks for a matching handler.

Example:

```python
def parse(text):
    return int(text)


def load():
    return parse("abc")


def main():
    return load()
```

Call:

```python
main()
```

`int("abc")` raises `ValueError`.

If `parse` does not catch it, it propagates to `load`.

If `load` does not catch it, it propagates to `main`.

If `main` does not catch it, it reaches the top level.

This movement up the call stack is exception propagation.

The traceback shows that path.

It tells you where the exception started and how execution reached that point.

Tracebacks are not noise.

They are maps.

---

# A Basic `try` and `except`

Basic form:

```python
try:
    risky_operation()
except SomeException:
    handle_error()
```

Example:

```python
try:
    value = int("abc")
except ValueError:
    value = 0
```

The `try` block runs.

If it raises `ValueError`, the `except ValueError` block runs.

If it does not raise `ValueError`, the `except` block is skipped.

If it raises a different exception, that exception keeps propagating unless another matching handler exists.

Example:

```python
try:
    value = numbers[10]
except ValueError:
    value = 0
```

If `numbers[10]` raises `IndexError`, the `ValueError` handler does not catch it.

That is good.

Catch the errors you can handle.

Do not catch unrelated errors by accident.

---

# Catching the Exception Object

Use `as` to access the exception object:

```python
try:
    value = int(text)
except ValueError as error:
    print(f"invalid integer: {error}")
```

The variable `error` refers to the exception instance inside the `except` block.

The exception object may contain:

* message
* arguments
* attributes
* cause
* context
* traceback

For built-in exceptions, the message is often enough for display or logging.

For custom exceptions, you can add meaningful attributes.

Do not rely too heavily on parsing exception message strings.

Messages are for humans.

Exception types and attributes are for programs.

---

# Multiple `except` Clauses

You can handle different exceptions differently:

```python
try:
    value = config["port"]
    port = int(value)
except KeyError:
    port = 8000
except ValueError:
    raise ValueError("port must be an integer")
```

The first matching `except` runs.

Order matters.

More specific exception types should come before broader ones.

Bad:

```python
try:
    ...
except Exception:
    ...
except ValueError:
    ...
```

The `ValueError` handler is unreachable because `ValueError` is a subclass of `Exception`.

Better:

```python
try:
    ...
except ValueError:
    ...
except Exception:
    ...
```

Catch specific exceptions first.

Catch broad exceptions only at clear boundaries.

---

# Catching Several Exception Types

If several exceptions have the same handling logic, use a tuple:

```python
try:
    value = mapping[key]
except (KeyError, TypeError):
    value = None
```

This catches either `KeyError` or `TypeError`.

Use this when the recovery is truly the same.

Do not group exceptions just because you are impatient.

Example:

```python
except (KeyError, ValueError, TypeError):
    return None
```

may hide too much.

Ask:

```text
Do these failures mean the same thing at this layer?
```

If yes, group them.

If no, handle separately or let some propagate.

---

# The `else` Clause

`try` can have an `else` clause:

```python
try:
    value = int(text)
except ValueError:
    handle_invalid(text)
else:
    use(value)
```

The `else` block runs only if the `try` block did not raise an exception.

Why not put `use(value)` inside the `try` block?

Because a narrower `try` block is clearer.

Compare:

```python
try:
    value = int(text)
    use(value)
except ValueError:
    handle_invalid(text)
```

If `use(value)` raises `ValueError`, the handler catches it too.

That may be wrong.

With `else`, the handler covers only the conversion:

```python
try:
    value = int(text)
except ValueError:
    handle_invalid(text)
else:
    use(value)
```

Use `else` when you want to keep the protected operation small and run follow-up code only on success.

---

# The `finally` Clause

`finally` runs whether an exception happens or not.

Example:

```python
resource = acquire()
try:
    use(resource)
finally:
    release(resource)
```

The release happens if:

* `use` succeeds
* `use` raises
* the `try` block returns
* the `try` block breaks or continues inside a loop

This is why `finally` is used for cleanup.

Example:

```python
file = open("data.txt")
try:
    data = file.read()
finally:
    file.close()
```

Usually, a context manager is clearer:

```python
with open("data.txt") as file:
    data = file.read()
```

But `finally` is the underlying cleanup idea.

---

# `try`, `except`, `else`, and `finally` Together

Full shape:

```python
try:
    risky_operation()
except ExpectedError:
    recover()
else:
    run_if_no_exception()
finally:
    always_cleanup()
```

Execution:

* `try` always starts.
* `except` runs only for matching exceptions.
* `else` runs only if no exception was raised in `try`.
* `finally` runs at the end either way.

Example:

```python
file = open("numbers.txt")
try:
    text = file.read()
    number = int(text)
except ValueError:
    number = 0
else:
    print("parsed successfully")
finally:
    file.close()
```

In real code:

```python
with open("numbers.txt") as file:
    ...
```

is better for file cleanup.

But understanding all four clauses matters.

---

# Raising Exceptions

Use `raise` to raise an exception.

Example:

```python
def divide(a, b):
    if b == 0:
        raise ValueError("b cannot be zero")
    return a / b
```

The raised object is usually an exception instance:

```python
raise ValueError("message")
```

You can also raise an exception class:

```python
raise ValueError
```

Python will instantiate it.

But including a useful message is better:

```python
raise ValueError("port must be between 1 and 65535")
```

Good exception messages explain:

* what was wrong
* what was expected
* sometimes what value was received

Example:

```python
raise ValueError(f"port must be between 1 and 65535; got {port!r}")
```

---

# Re-Raising Exceptions

Inside an `except` block, plain `raise` re-raises the active exception.

Example:

```python
try:
    process(record)
except ValueError:
    log_bad_record(record)
    raise
```

This logs and then lets the original exception continue.

Do not write:

```python
raise error
```

unless you specifically need to raise that object as a new raise operation.

Plain `raise` preserves the original traceback more cleanly.

Use re-raising when:

* you need to log
* you need to clean up
* you need to update metrics
* you need to add local side effects
* but you cannot fully handle the error

If you cannot solve the problem at this layer, let the caller see the error.

---

# Exception Hierarchy

Exceptions are classes.

They form an inheritance hierarchy.

Important roots:

```text
BaseException
Exception
```

`BaseException` is the root of all built-in exceptions.

But most application errors inherit from `Exception`, not `BaseException`.

Why?

Some exceptions are intended to escape most normal handlers:

* `SystemExit`
* `KeyboardInterrupt`
* `GeneratorExit`

These inherit directly from `BaseException`, not `Exception`.

So this:

```python
except Exception:
    ...
```

does not catch `KeyboardInterrupt`.

That is usually good.

Users pressing Ctrl+C should usually be able to stop the program.

Avoid:

```python
except BaseException:
    ...
```

unless you are writing very low-level framework or cleanup code and truly know why.

---

# Common Built-In Exceptions

You do not need to memorize every built-in exception.

But you should know common ones.

`ValueError`:

```text
right type, invalid value
```

Example:

```python
int("abc")
```

`TypeError`:

```text
wrong type or unsupported operation
```

Example:

```python
"a" + 1
```

`KeyError`:

```text
missing mapping key
```

Example:

```python
data["missing"]
```

`IndexError`:

```text
sequence index out of range
```

Example:

```python
items[100]
```

`AttributeError`:

```text
missing attribute
```

Example:

```python
obj.missing
```

`FileNotFoundError`:

```text
file path does not exist
```

`PermissionError`:

```text
operation not permitted
```

`TimeoutError`:

```text
operation timed out
```

Use the most specific appropriate exception.

---

# `ValueError` Versus `TypeError`

This distinction appears constantly.

Use `TypeError` when the type is wrong:

```python
def set_age(age):
    if not isinstance(age, int):
        raise TypeError("age must be an integer")
```

Use `ValueError` when the type is acceptable but the value is invalid:

```python
def set_age(age):
    if age < 0:
        raise ValueError("age cannot be negative")
```

Combined:

```python
def set_age(age):
    if not isinstance(age, int):
        raise TypeError("age must be an integer")
    if age < 0:
        raise ValueError("age cannot be negative")
```

This convention makes errors easier to understand.

It also helps callers catch the right failures.

---

# `KeyError` Versus `ValueError`

If a key is missing from a mapping, `KeyError` is natural:

```python
def require_key(mapping, key):
    if key not in mapping:
        raise KeyError(key)
    return mapping[key]
```

If a value is present but semantically invalid, use `ValueError`:

```python
def parse_status(data):
    status = data["status"]
    if status not in {"draft", "active", "archived"}:
        raise ValueError(f"invalid status: {status!r}")
    return status
```

Missing key:

```text
KeyError
```

Invalid value:

```text
ValueError
```

Choose the exception that describes the failure, not the line of code.

---

# Custom Exceptions

Create custom exceptions when callers need to distinguish your domain errors.

Example:

```python
class PaymentError(Exception):
    pass
```

Then:

```python
class InsufficientFunds(PaymentError):
    pass
```

Use:

```python
def charge(account, amount):
    if account.balance < amount:
        raise InsufficientFunds("account balance is too low")
```

Callers can catch:

```python
try:
    charge(account, amount)
except InsufficientFunds:
    show_insufficient_funds_message()
except PaymentError:
    show_general_payment_error()
```

Custom exceptions should usually inherit from `Exception` or from your own application-specific base exception.

Do not inherit from `BaseException` for normal application errors.

---

# Exception Class Design

A simple custom exception is often enough:

```python
class ConfigurationError(Exception):
    pass
```

Sometimes attributes help:

```python
class InvalidField(Exception):
    def __init__(self, field, value, message):
        self.field = field
        self.value = value
        super().__init__(message)
```

Use:

```python
raise InvalidField(
    field="port",
    value=70000,
    message="port must be between 1 and 65535",
)
```

Now callers can inspect:

```python
error.field
error.value
```

Do this when programmatic handling needs structured data.

If the exception is only for humans, a clear message may be enough.

Avoid overbuilding exception classes.

---

# Exception Chaining

Sometimes you catch a low-level exception and raise a higher-level one.

Example:

```python
def load_config(path):
    try:
        text = Path(path).read_text()
    except OSError as error:
        raise ConfigurationError(f"could not read config file {path!r}") from error
```

The `from error` part preserves the original cause.

Tracebacks will show both:

* the low-level `OSError`
* the higher-level `ConfigurationError`

This is excellent for debugging.

It tells the caller the domain-level failure while keeping the original technical cause.

Use exception chaining when translating errors across abstraction boundaries.

---

# Implicit Exception Context

If you raise a new exception inside an `except` block without `from`, Python still records context.

Example:

```python
try:
    int("abc")
except ValueError:
    raise RuntimeError("failed to parse input")
```

Python can show that the `RuntimeError` occurred while handling the `ValueError`.

This is called exception context.

Explicit chaining is clearer:

```python
except ValueError as error:
    raise RuntimeError("failed to parse input") from error
```

Use `from` when the new exception is caused by the old one.

It communicates intent.

---

# Suppressing Context with `from None`

Sometimes the lower-level exception is an implementation detail that would confuse the caller.

Example:

```python
def get_required(mapping, key):
    try:
        return mapping[key]
    except KeyError:
        raise ValueError(f"missing required value: {key}") from None
```

`from None` suppresses display of the original exception context.

Use sparingly.

It can make tracebacks cleaner.

It can also hide useful debugging information.

Good use:

```text
replace an internal lookup failure with a clearer public API error
```

Bad use:

```text
hide the real cause because the traceback looks messy
```

Prefer preserving cause unless you have a clear reason.

---

# The EAFP Style

Python often favors EAFP:

```text
Easier to Ask Forgiveness than Permission
```

Instead of checking first:

```python
if key in mapping:
    value = mapping[key]
else:
    value = default
```

you may write:

```python
try:
    value = mapping[key]
except KeyError:
    value = default
```

This style is natural when:

* the operation is expected to usually work
* the failure has a clear exception
* checking first would duplicate work
* race conditions could make pre-checks unreliable

But EAFP is not a commandment.

Sometimes explicit checks are clearer:

```python
if age < 0:
    raise ValueError("age cannot be negative")
```

Use judgment.

---

# LBYL Style

LBYL means:

```text
Look Before You Leap
```

Example:

```python
if path.exists():
    text = path.read_text()
else:
    text = ""
```

This is clear.

But it can be unsafe for files:

```text
file exists during check
file is deleted before read
```

So for file operations, EAFP is often better:

```python
try:
    text = path.read_text()
except FileNotFoundError:
    text = ""
```

LBYL is good for validation and readability.

EAFP is good for operations where the operation itself is the reliable test.

Professional Python uses both.

---

# Catch Narrowly

This is too broad:

```python
try:
    process(data)
except Exception:
    pass
```

It hides everything.

Better:

```python
try:
    process(data)
except InvalidRecord as error:
    report_bad_record(error)
```

Catch the exception you expect and can handle.

If you cannot explain why you are catching an exception, you probably should not catch it there.

Questions:

```text
What exact failure do I expect?
What can I do about it here?
Should the caller know?
Would catching this hide a bug?
```

Good exception handling is specific.

---

# Keep `try` Blocks Small

Bad:

```python
try:
    raw = read_file(path)
    data = parse_json(raw)
    user = build_user(data)
    save_user(user)
except ValueError:
    handle_invalid_data()
```

Which line raised `ValueError`?

Parsing?

Building?

Saving?

Better:

```python
raw = read_file(path)

try:
    data = parse_json(raw)
except ValueError as error:
    raise InvalidInput("file must contain valid JSON") from error

user = build_user(data)
save_user(user)
```

Small `try` blocks make handlers accurate.

They prevent accidentally catching errors from unrelated code.

---

# Do Not Silence Errors Accidentally

This is dangerous:

```python
try:
    update_user(user)
except Exception:
    return False
```

The caller gets `False`.

The real error is gone.

At minimum, log:

```python
try:
    update_user(user)
except DatabaseError:
    logger.exception("failed to update user")
    return False
```

But even this depends on design.

Maybe the caller needs to know the update failed.

Maybe returning `False` is too weak.

Errors are information.

Do not throw information away unless you are truly replacing it with something better.

---

# Logging Exceptions

When logging inside an `except` block, use exception-aware logging.

Example:

```python
try:
    process(record)
except InvalidRecord:
    logger.exception("invalid record: %r", record)
```

`logger.exception` includes traceback information when called inside an exception handler.

Do not do:

```python
logger.error("failed: %s", error)
```

if you need the traceback.

Also avoid logging and re-raising at every layer.

That can produce duplicate logs.

A common pattern:

* low-level code raises
* middle-level code adds context if useful
* boundary-level code logs

Log where you can make the message meaningful.

---

# Error Boundaries

An error boundary is a layer where exceptions are caught and translated into something appropriate for that layer.

Examples:

CLI boundary:

```python
try:
    run_command()
except ConfigurationError as error:
    print(f"configuration error: {error}")
    return 2
```

Web boundary:

```python
try:
    user = service.get_user(id)
except UserNotFound:
    return Response(status=404)
```

Worker boundary:

```python
try:
    process_job(job)
except TemporaryFailure:
    retry_later(job)
except PermanentFailure:
    mark_failed(job)
```

Boundaries are good places to catch exceptions because they know how to translate errors for the outside world.

Deep helper functions often should not catch much.

They usually lack enough context.

---

# Exceptions and APIs

Your function's exceptions are part of its API.

Example:

```python
def load_user(id):
    ...
```

Callers need to know:

* What happens if the user does not exist?
* What happens if the database is unavailable?
* What happens if `id` is invalid?

Possible design:

```python
def load_user(id):
    if not isinstance(id, int):
        raise TypeError("id must be an integer")
    if id < 1:
        raise ValueError("id must be positive")
    ...
    if missing:
        raise UserNotFound(id)
```

Document important domain exceptions.

Do not make callers reverse-engineer error behavior from source code.

---

# Exceptions Versus Return Values

Should a function raise or return a status?

Raise when:

* the operation cannot produce the promised result
* the caller likely cannot continue normally without handling it
* the failure is exceptional for that function's contract
* you need rich error information and propagation

Return a value when:

* absence is expected and normal
* the result naturally includes success/failure
* the caller checks it frequently
* no stack unwinding is needed

Example:

```python
dict.get(key)
```

returns a default because missing keys are common and expected.

But:

```python
dict[key]
```

raises `KeyError` because the contract says the key should exist.

Both APIs are useful.

Design the contract first.

---

# `None` Is Not an Error Message

This function is weak:

```python
def parse_user(data):
    if "email" not in data:
        return None
```

Why did it return `None`?

Missing email?

Invalid email?

Wrong data type?

Database unavailable?

Better:

```python
def parse_user(data):
    if "email" not in data:
        raise ValueError("missing required field: email")
```

Or return a structured result if failure is normal:

```python
ParseResult(success=False, error="missing required field: email")
```

Do not use `None` as a vague failure bucket.

Ambiguous failure values create fragile code.

---

# Exceptions in Iteration

`StopIteration` is special.

It signals iterator exhaustion.

You normally do not catch it around a `for` loop:

```python
for item in items:
    process(item)
```

The loop handles it.

Manual `next()` may catch it:

```python
iterator = iter(items)
try:
    first = next(iterator)
except StopIteration:
    first = None
```

or:

```python
first = next(iter(items), None)
```

Inside generator functions, do not manually raise `StopIteration` to stop.

Use:

```python
return
```

or let the function end.

Iteration has its own exception convention.

Respect it.

---

# Exceptions in Context Managers

Chapter 60 showed that `__exit__` receives exception information.

Example:

```python
def __exit__(self, exc_type, exc, traceback):
    cleanup()
    return False
```

If `__exit__` returns true, it suppresses the exception.

Most context managers should not suppress exceptions.

This is important:

```python
with transaction:
    save_user(user)
```

If `save_user` fails, the transaction should roll back.

But the exception should usually continue.

Cleanup and suppression are different.

Good context managers clean up.

Only special context managers suppress.

---

# Exceptions in Decorators

Decorators can accidentally change exception behavior.

Bad:

```python
def safe(function):
    def wrapper(*args, **kwargs):
        try:
            return function(*args, **kwargs)
        except Exception:
            return None
    return wrapper
```

This hides every error from every decorated function.

Better:

```python
def log_errors(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        try:
            return function(*args, **kwargs)
        except ExpectedError:
            logger.exception("operation failed")
            raise
    return wrapper
```

Decorators that catch exceptions should be extremely clear about:

* what they catch
* what they return
* what they log
* whether they re-raise
* whether they change the function's contract

---

# Exception Groups

Modern Python has exception groups for representing multiple exceptions together.

The main class is:

```python
ExceptionGroup
```

Example:

```python
raise ExceptionGroup(
    "multiple failures",
    [
        ValueError("bad value"),
        TypeError("bad type"),
    ],
)
```

Exception groups are useful when several operations happen concurrently or independently and more than one fails.

For example:

* concurrent tasks
* batch validation
* parallel file processing
* multiple cleanup failures

Normal `except` handles the whole group as one exception.

`except*` can handle matching sub-exceptions inside the group.

---

# `except*`

`except*` handles exception groups by type.

Example:

```python
try:
    raise ExceptionGroup(
        "errors",
        [
            ValueError("bad value"),
            TypeError("bad type"),
        ],
    )
except* ValueError as group:
    print("handled value errors")
except* TypeError as group:
    print("handled type errors")
```

Each `except*` receives a subgroup of matching exceptions.

This is different from normal `except`.

Use `except*` only when working with exception groups.

Most everyday exception handling still uses normal `except`.

Exception groups become more important in concurrent programming.

We will see them again when discussing modern concurrency.

---

# `finally` Pitfalls

`finally` is for cleanup.

Be careful with `return` inside `finally`.

Example:

```python
def bad():
    try:
        raise ValueError("original")
    finally:
        return "hidden"
```

The return from `finally` hides the exception.

This is almost always a bug.

Similarly, `break` or `continue` in `finally` can discard active exceptions.

Python 3.14 can emit a warning for some control-flow statements in `finally`.

Design rule:

```text
finally should clean up, not decide the main result
```

Avoid `return`, `break`, and `continue` in `finally`.

---

# Cleanup Errors

Cleanup can fail.

Example:

```python
try:
    use(resource)
finally:
    resource.close()
```

What if `use(resource)` raises and `resource.close()` also raises?

The cleanup error may obscure the original error.

Sometimes that is acceptable because cleanup failure is serious.

Sometimes you want to preserve the original error and log cleanup failure.

There is no universal answer.

But do not ignore the possibility.

For critical systems, cleanup error policy should be deliberate.

Context managers face the same question in `__exit__`.

---

# Assertions Are Not Error Handling

Python has `assert`:

```python
assert age >= 0
```

If the condition is false, Python raises `AssertionError`.

Assertions are for internal sanity checks and developer assumptions.

They are not for validating user input.

Why?

Python can run with optimizations that remove assert statements.

This is wrong:

```python
def set_age(age):
    assert age >= 0
    self.age = age
```

Use:

```python
def set_age(age):
    if age < 0:
        raise ValueError("age cannot be negative")
```

Use assertions for:

```text
this should be impossible if my code is correct
```

Use exceptions for:

```text
this input or operation can fail at runtime
```

---

# Designing Custom Exception Hierarchies

For an application or library, define a base exception:

```python
class AppError(Exception):
    pass
```

Then specific errors:

```python
class ConfigurationError(AppError):
    pass


class ValidationError(AppError):
    pass


class ExternalServiceError(AppError):
    pass
```

This lets callers catch:

```python
except AppError:
    ...
```

for all expected application-level errors.

They can also catch specific subclasses.

Do not create huge hierarchies too early.

Start with meaningful distinctions callers need.

If nobody handles two exceptions differently, they may not need separate classes yet.

---

# Domain Exceptions

Domain exceptions describe business or application rules.

Example:

```python
class OrderError(Exception):
    pass


class OrderAlreadyShipped(OrderError):
    pass


class PaymentRequired(OrderError):
    pass
```

Use:

```python
def cancel_order(order):
    if order.shipped:
        raise OrderAlreadyShipped(order.id)
    if not order.paid:
        raise PaymentRequired(order.id)
```

These exceptions mean something in the problem domain.

They are better than generic:

```python
raise Exception("cannot cancel")
```

Domain-specific exceptions make boundary handling clearer.

For example, a web layer can map:

```text
OrderAlreadyShipped -> 409 Conflict
PaymentRequired -> 402 or domain-specific response
```

---

# Wrapping Low-Level Errors

Low-level errors should often be translated at boundaries.

Example:

```python
def load_settings(path):
    try:
        text = Path(path).read_text()
    except OSError as error:
        raise ConfigurationError(
            f"could not read settings from {path!r}"
        ) from error
```

Why wrap?

The caller asked for settings.

The meaningful failure is:

```text
configuration could not be loaded
```

The cause may be:

```text
file missing
permission denied
disk error
encoding problem
```

Chaining preserves both levels.

This is robust error translation.

---

# Retrying Exceptions

Some exceptions represent temporary failures.

Example:

* timeout
* rate limit
* temporary network problem
* database deadlock

Others usually should not be retried:

* invalid input
* authentication failure
* missing required field
* programming bug

Retry only when the operation is safe to retry.

Questions:

```text
Is the operation idempotent?
Could it have partially succeeded?
Will retry duplicate side effects?
Which exceptions are temporary?
How many attempts?
What delay?
```

Retries are not just exception handling.

They are system design.

---

# A Practical Parsing Example

```python
class ParseError(Exception):
    pass
```

Parser:

```python
def parse_port(text):
    try:
        port = int(text)
    except ValueError as error:
        raise ParseError(f"port must be an integer; got {text!r}") from error

    if not 1 <= port <= 65535:
        raise ParseError(f"port must be between 1 and 65535; got {port!r}")

    return port
```

Use:

```python
try:
    port = parse_port(config["port"])
except KeyError as error:
    raise ParseError("missing required config key: port") from error
```

This design:

* catches low-level conversion errors
* raises domain-level parse errors
* preserves causes
* validates range separately

The caller sees `ParseError`.

The traceback still shows original causes.

---

# A Practical File Example

```python
class DataLoadError(Exception):
    pass
```

Loader:

```python
def load_lines(path):
    try:
        with open(path) as file:
            return [line.rstrip("\n") for line in file]
    except FileNotFoundError as error:
        raise DataLoadError(f"data file not found: {path}") from error
    except PermissionError as error:
        raise DataLoadError(f"permission denied reading data file: {path}") from error
    except OSError as error:
        raise DataLoadError(f"could not read data file: {path}") from error
```

Why separate handlers?

Because the messages are better.

The caller gets one domain exception type:

```python
DataLoadError
```

But the message and cause preserve detail.

This is a common robust-I/O pattern.

Chapter 63 will go deeper into files and serialization.

---

# A Practical Validation Example

```python
class ValidationError(Exception):
    pass
```

Validation:

```python
def validate_user(data):
    errors = []

    if "email" not in data:
        errors.append("email is required")
    elif "@" not in data["email"]:
        errors.append("email must contain @")

    if "age" in data and data["age"] < 0:
        errors.append("age cannot be negative")

    if errors:
        raise ValidationError("; ".join(errors))
```

This collects multiple validation errors into one message.

For richer APIs, use structured attributes:

```python
class ValidationError(Exception):
    def __init__(self, errors):
        self.errors = errors
        super().__init__("validation failed")
```

Then callers can inspect:

```python
error.errors
```

Use structured exceptions when callers need structured error data.

---

# A Practical Boundary Example

Service layer:

```python
class UserNotFound(Exception):
    pass


def get_user(id):
    user = repository.find_user(id)
    if user is None:
        raise UserNotFound(id)
    return user
```

Web boundary:

```python
def get_user_endpoint(request, id):
    try:
        user = get_user(id)
    except UserNotFound:
        return Response(status=404, body="user not found")

    return Response(status=200, body=serialize_user(user))
```

The service raises a domain exception.

The web boundary translates it into HTTP.

This separation is healthy.

Do not make the service return HTTP responses.

Do not make the web boundary inspect database internals.

Exceptions can preserve clean architecture boundaries.

---

# Common Mistake: Bare `except`

Avoid:

```python
try:
    ...
except:
    ...
```

This catches almost everything, including things you usually should not catch.

Use:

```python
except Exception:
```

only when you really want broad application-level exceptions.

Better:

```python
except ValueError:
```

or:

```python
except OSError:
```

Bare `except` is rarely appropriate outside very low-level cleanup or top-level crash reporting.

Even then, be careful.

---

# Common Mistake: Catching and Ignoring

This is usually bad:

```python
try:
    send_email(message)
except Exception:
    pass
```

What if email sending always fails?

What if credentials are wrong?

What if there is a programming bug?

Nobody knows.

At minimum:

```python
except EmailError:
    logger.exception("failed to send email")
```

But decide whether the caller should know.

Silent failure is rarely robust.

It is often just invisible failure.

---

# Common Mistake: Catching Too High

This can hide bugs:

```python
try:
    total = price * quantity
except Exception:
    total = 0
```

If `price` is a string, that is a bug.

Returning zero may create bad data.

Catch expected operational failures.

Do not catch programming mistakes and pretend they are business defaults.

Errors should be handled at the layer that understands them.

If no layer understands them, they should be visible.

---

# Common Mistake: Raising Generic `Exception`

Weak:

```python
raise Exception("invalid user")
```

Better:

```python
raise ValueError("email is required")
```

or:

```python
raise ValidationError("email is required")
```

Generic `Exception` makes it hard for callers to catch specific failures.

Use specific built-in exceptions or custom domain exceptions.

The exception type is part of the information.

Do not throw it away.

---

# Common Mistake: Losing the Original Cause

Weak:

```python
try:
    data = json.loads(text)
except JSONDecodeError:
    raise ConfigurationError("invalid configuration")
```

Better:

```python
try:
    data = json.loads(text)
except JSONDecodeError as error:
    raise ConfigurationError("invalid configuration") from error
```

The chained version preserves the original parse error.

That can include line and column information.

When translating exceptions, preserve cause unless hiding it is intentional.

---

# Common Mistake: Exception Messages Without Context

Weak:

```python
raise ValueError("invalid")
```

Better:

```python
raise ValueError("port must be between 1 and 65535")
```

Often better:

```python
raise ValueError(f"port must be between 1 and 65535; got {port!r}")
```

But be careful with secrets.

Do not include passwords, tokens, private keys, or sensitive personal data in exception messages.

Error messages should help debugging without leaking sensitive information.

---

# Common Mistake: Using Exceptions for Normal Loop Control

`StopIteration` is part of iteration.

But do not use custom exceptions for ordinary loops when normal control flow is clearer.

Weak:

```python
try:
    while True:
        item = get_next_or_raise()
        process(item)
except NoMoreItems:
    pass
```

If this is an iterator, implement the iterator protocol.

If this is a normal loop, use a condition:

```python
while has_items():
    process(get_next())
```

Exceptions can express abnormal flow.

They can also support protocols.

But they should not make simple loops harder to read.

---

# Common Mistake: Overusing Custom Exceptions

Do not create a new exception class for every possible message.

This may be too much:

```python
class MissingEmailError(ValidationError):
    pass


class InvalidEmailFormatError(ValidationError):
    pass


class EmailTooLongError(ValidationError):
    pass
```

unless callers handle them differently.

If all are displayed the same way, one `ValidationError` with structured details may be better.

Create exception types for meaningful handling differences.

Not for every sentence.

---

# Exception Handling Checklist

When handling an exception, ask:

```text
Do I know exactly what failed?
```

If no, narrow the `try` block.

Ask:

```text
Can I recover here?
```

If no, add context and re-raise or let it propagate.

Ask:

```text
Am I catching the narrowest useful exception type?
```

If no, narrow it.

Ask:

```text
Will this hide a programming bug?
```

If yes, do not catch it here.

Ask:

```text
Should I translate this into a domain exception?
```

If crossing abstraction boundaries, often yes.

Ask:

```text
Should I preserve the original cause?
```

Usually yes, with `raise ... from error`.

Ask:

```text
Should this be logged here?
```

Only if this layer is a meaningful logging boundary.

---

# Practice: Basic Handling

Write a function that parses an integer and returns `0` for invalid input.

Solution:

```python
def parse_or_zero(text):
    try:
        return int(text)
    except ValueError:
        return 0
```

Test:

```python
assert parse_or_zero("10") == 10
assert parse_or_zero("abc") == 0
```

This is acceptable if `0` is truly the desired default.

If invalid input should be reported, raise instead.

---

# Practice: Use `else`

Rewrite this to avoid catching errors from `use(value)`:

```python
try:
    value = int(text)
    use(value)
except ValueError:
    handle_invalid(text)
```

Solution:

```python
try:
    value = int(text)
except ValueError:
    handle_invalid(text)
else:
    use(value)
```

Now the handler only covers `int(text)`.

---

# Practice: Custom Exception

Create a custom exception for invalid configuration.

Solution:

```python
class ConfigurationError(Exception):
    pass
```

Use:

```python
def require_setting(settings, name):
    try:
        return settings[name]
    except KeyError as error:
        raise ConfigurationError(f"missing setting: {name}") from error
```

This translates a low-level `KeyError` into a domain-level configuration error.

---

# Practice: Preserve Cause

Improve this code:

```python
try:
    data = json.loads(text)
except JSONDecodeError:
    raise ValueError("invalid JSON")
```

Solution:

```python
try:
    data = json.loads(text)
except JSONDecodeError as error:
    raise ValueError("invalid JSON") from error
```

Now the original JSON parse error remains available in the traceback.

---

# Practice: Avoid Bare Except

Rewrite:

```python
try:
    path.unlink()
except:
    pass
```

Better:

```python
try:
    path.unlink()
except FileNotFoundError:
    pass
```

Or:

```python
from contextlib import suppress


with suppress(FileNotFoundError):
    path.unlink()
```

This suppresses only the expected harmless error.

---

# Practice: Design an Error Boundary

Suppose service code raises:

```python
class UserNotFound(Exception):
    pass
```

Write a CLI boundary that prints a message and returns exit code `1`.

Solution:

```python
def main():
    try:
        user = load_user_from_arguments()
    except UserNotFound as error:
        print(f"user not found: {error}")
        return 1

    print(user)
    return 0
```

The boundary translates a domain exception into user-facing output and an exit code.

---

# Summary

Exceptions represent abnormal conditions.

Raising an exception interrupts normal control flow.

Python searches up the call stack for a matching handler.

`try` marks code that may raise.

`except` handles matching exceptions.

`else` runs only when the `try` block succeeds.

`finally` runs for cleanup whether the `try` block succeeds or fails.

Use `raise` to raise exceptions.

Use plain `raise` inside an `except` block to re-raise the active exception.

Catch specific exceptions whenever possible.

Keep `try` blocks small.

Use built-in exceptions such as `ValueError`, `TypeError`, `KeyError`, `IndexError`, `AttributeError`, and `OSError` subclasses when they fit.

Use custom exceptions when callers need to distinguish domain-level failures.

Use `raise ... from error` when translating exceptions across abstraction boundaries.

Use `from None` only when hiding lower-level context makes the public error clearer.

Exception groups and `except*` support multiple simultaneous errors, especially in concurrent or batch contexts.

Do not use assertions for runtime input validation.

Do not catch and ignore broad exceptions.

Do not hide failures behind vague `None` returns unless absence is truly normal.

The design principle is:

```text
handle exceptions where you can add meaning, recover safely, or translate for a boundary
```

If you cannot do one of those things, let the exception propagate.

---

# Preview of Chapter 63

Chapter 62 focused on robust error handling.

Next we study files and serialization.

Files and serialization are where exception handling becomes concrete.

Programs must deal with:

* missing files
* permission errors
* encoding problems
* invalid JSON
* malformed CSV
* partial writes
* unsafe input
* data format compatibility
* resource cleanup

Chapter 63 will cover:

* text files and binary files
* file modes
* encodings
* path handling
* reading and writing safely
* JSON
* CSV
* pickle and its risks
* serialization boundaries
* atomic-ish write patterns
* error handling around I/O

The transition is:

```text
exceptions teach how failures move
files and serialization show where failures happen constantly
```

Robust I/O is where clean Python starts to feel like professional engineering.

