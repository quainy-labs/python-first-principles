# Chapter 61 — Decorators

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a decorator is.
* Why decorators exist.
* How decorator syntax translates to ordinary assignment.
* Why functions can wrap other functions.
* How closures make decorators possible.
* How to write a basic function decorator.
* Why wrappers need `*args` and `**kwargs`.
* Why `functools.wraps` matters.
* How to write decorators that return values correctly.
* How to write decorators with arguments.
* How multiple decorators stack.
* How decorators work on methods.
* How class decorators work.
* How decorators compare with context managers.
* How decorators compare with inheritance and composition.
* How common standard-library decorators fit the pattern.
* When decorators improve design.
* When decorators hide too much.

Chapter 60 studied context managers.

Context managers wrap a block of execution:

```python
with timer():
    do_work()
```

Decorators wrap functions, methods, or classes:

```python
@timer
def do_work():
    ...
```

Both abstractions remove repeated structure.

Context managers say:

```text
run this setup and cleanup around this block
```

Decorators say:

```text
transform this definition when it is created
```

Decorators are everywhere in Python:

```python
@property
@classmethod
@staticmethod
@dataclass
@contextmanager
@functools.cache
@app.route("/users")
@pytest.mark.slow
```

You have already used decorators many times.

Now we study how they work.

---

# A Decorator Is a Callable That Returns a Replacement

At the simplest level, a decorator is a callable that takes an object and returns an object.

Most commonly:

```text
function in -> function out
```

Example:

```python
def decorator(function):
    return function
```

Use:

```python
@decorator
def greet():
    return "hello"
```

This decorator does nothing.

It receives the function object and returns the same function object.

The decorated function still works:

```python
greet()
```

returns:

```python
"hello"
```

Even this no-op example proves the core idea:

```text
decorator syntax passes the defined object through another callable
```

---

# Decorator Syntax Is Assignment

This:

```python
@decorator
def greet():
    return "hello"
```

means roughly:

```python
def greet():
    return "hello"


greet = decorator(greet)
```

The name `greet` is rebound to whatever `decorator(greet)` returns.

That is why decorators are powerful.

They can replace the original function with:

* the same function
* a wrapper function
* a callable object
* a descriptor-like object
* something else callable

Most function decorators return a wrapper function.

But the mechanism is simply:

```text
define object
call decorator with object
bind name to result
```

This happens when the function definition is executed.

Usually that means import time for module-level functions.

---

# Functions Are Objects

Decorators rely on a fact you already know:

```text
functions are objects
```

You can assign a function to another name:

```python
def greet():
    return "hello"


say_hello = greet
```

You can pass a function to another function:

```python
def call(function):
    return function()


call(greet)
```

You can return a function from a function:

```python
def make_greeter():
    def greet():
        return "hello"

    return greet
```

Decorators use all of this.

A decorator receives a function object.

It often creates a new function that calls the original.

Then it returns the new function.

---

# A First Real Decorator

Let us write a decorator that prints before and after a function call.

```python
def announce(function):
    def wrapper():
        print("before")
        result = function()
        print("after")
        return result

    return wrapper
```

Use:

```python
@announce
def greet():
    print("hello")
```

Call:

```python
greet()
```

Output:

```text
before
hello
after
```

What happened?

The original `greet` function was passed to `announce`.

`announce` returned `wrapper`.

The name `greet` now refers to `wrapper`.

When you call `greet()`, you are calling the wrapper.

The wrapper calls the original function inside it.

---

# The Closure Behind the Decorator

How does `wrapper` remember `function`?

Closure.

In:

```python
def announce(function):
    def wrapper():
        print("before")
        result = function()
        print("after")
        return result

    return wrapper
```

`wrapper` uses `function` from the enclosing scope.

Even after `announce` returns, `wrapper` keeps a reference to `function`.

That is a closure.

Closures from Volume I now become practical.

The pattern is:

```text
outer function receives original function
inner function remembers original function
outer function returns inner function
```

This is the heart of function decorators.

---

# Supporting Arguments

The first decorator only works for functions with no arguments.

This fails:

```python
@announce
def greet(name):
    print(f"hello {name}")


greet("Maya")
```

because `wrapper()` does not accept `name`.

Fix with `*args` and `**kwargs`:

```python
def announce(function):
    def wrapper(*args, **kwargs):
        print("before")
        result = function(*args, **kwargs)
        print("after")
        return result

    return wrapper
```

Now the wrapper accepts any positional and keyword arguments and passes them through.

Use:

```python
@announce
def greet(name, punctuation="!"):
    print(f"hello {name}{punctuation}")
```

Call:

```python
greet("Maya", punctuation=".")
```

Output:

```text
before
hello Maya.
after
```

Most general-purpose decorators should preserve the wrapped function's call shape by using:

```python
*args, **kwargs
```

---

# Returning the Result

A wrapper must return the original function's result unless it intentionally changes behavior.

Bug:

```python
def announce(function):
    def wrapper(*args, **kwargs):
        print("before")
        function(*args, **kwargs)
        print("after")

    return wrapper
```

Use:

```python
@announce
def add(a, b):
    return a + b
```

Now:

```python
add(2, 3)
```

returns:

```python
None
```

because the wrapper did not return the result.

Correct:

```python
def announce(function):
    def wrapper(*args, **kwargs):
        print("before")
        result = function(*args, **kwargs)
        print("after")
        return result

    return wrapper
```

Decorators wrap behavior.

They should not accidentally throw away return values.

---

# Exceptions in Wrapped Functions

Consider:

```python
def announce(function):
    def wrapper(*args, **kwargs):
        print("before")
        result = function(*args, **kwargs)
        print("after")
        return result

    return wrapper
```

If the wrapped function raises, `"after"` will not print:

```python
@announce
def fail():
    raise ValueError("bad")
```

Call:

```python
fail()
```

Output:

```text
before
```

Then `ValueError` propagates.

If you need cleanup-like behavior, use `try/finally`:

```python
def announce(function):
    def wrapper(*args, **kwargs):
        print("before")
        try:
            return function(*args, **kwargs)
        finally:
            print("after")

    return wrapper
```

This is similar in spirit to context managers.

The wrapper controls what happens around a function call.

---

# `functools.wraps`

Our decorator has a problem.

```python
@announce
def greet():
    """Return a greeting."""
    return "hello"
```

Check:

```python
print(greet.__name__)
print(greet.__doc__)
```

You may see:

```python
wrapper
None
```

Why?

Because `greet` now refers to the wrapper function.

The wrapper has its own metadata.

Use `functools.wraps`:

```python
from functools import wraps


def announce(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print("before")
        result = function(*args, **kwargs)
        print("after")
        return result

    return wrapper
```

Now metadata is preserved:

```python
greet.__name__
greet.__doc__
```

`wraps` also sets `__wrapped__`, which helps introspection tools find the original function.

Professional decorators should almost always use `functools.wraps`.

---

# A Timing Decorator

Timing is a common decorator example.

```python
from functools import wraps
from time import perf_counter


def timed(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        start = perf_counter()
        try:
            return function(*args, **kwargs)
        finally:
            elapsed = perf_counter() - start
            print(f"{function.__name__} took {elapsed:.3f}s")

    return wrapper
```

Use:

```python
@timed
def build_index(items):
    return sorted(items)
```

Call:

```python
build_index([3, 1, 2])
```

This returns the sorted list and prints timing.

Notice:

```python
try:
    return function(*args, **kwargs)
finally:
    ...
```

The timing prints even if the function raises.

The exception still propagates.

This is usually the right behavior for timing.

---

# A Logging Decorator

Example:

```python
from functools import wraps


def log_calls(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        result = function(*args, **kwargs)
        print(f"{function.__name__} returned {result!r}")
        return result

    return wrapper
```

Use:

```python
@log_calls
def add(a, b):
    return a + b
```

Call:

```python
add(2, 3)
```

Output:

```text
calling add
add returned 5
```

This is useful for teaching.

In production, use the `logging` module instead of `print`.

Also be careful logging arguments or return values.

They may contain secrets or large data.

Decorators make cross-cutting behavior easy.

That does not remove responsibility.

---

# Decorators Run at Definition Time

Decorators are applied when the function is defined.

For a module-level function, that usually means import time.

Example:

```python
def decorate(function):
    print(f"decorating {function.__name__}")
    return function


@decorate
def greet():
    return "hello"
```

When Python executes the definition, it prints:

```text
decorating greet
```

Calling `greet()` later does not apply the decorator again.

The wrapper may run on every call, but the decorator function itself ran when the definition was created.

This distinction matters for decorators that register routes, tests, plugins, or commands.

Registration often happens at import time.

---

# Decorators with Arguments

Sometimes a decorator needs configuration.

We want:

```python
@repeat(3)
def greet():
    print("hello")
```

This requires three layers:

```python
from functools import wraps


def repeat(times):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = function(*args, **kwargs)
            return result

        return wrapper

    return decorator
```

Why three layers?

```text
repeat(times) receives decorator arguments
decorator(function) receives the function being decorated
wrapper(*args, **kwargs) receives call arguments
```

Use:

```python
@repeat(3)
def greet():
    print("hello")
```

This is equivalent to:

```python
def greet():
    print("hello")


greet = repeat(3)(greet)
```

First `repeat(3)` returns a decorator.

Then that decorator receives `greet`.

---

# A Retry Decorator

Retries are a practical decorator-with-arguments example.

```python
from functools import wraps


def retry(times, exceptions=(Exception,)):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            last_error = None
            for _ in range(times):
                try:
                    return function(*args, **kwargs)
                except exceptions as error:
                    last_error = error
            raise last_error

        return wrapper

    return decorator
```

Use:

```python
@retry(3, exceptions=(TimeoutError,))
def fetch_data():
    ...
```

This retries only `TimeoutError`.

Design concerns:

* Should there be delay between retries?
* Should delay grow over time?
* Which exceptions are safe to retry?
* Is the function idempotent?
* Should failures be logged?
* Should final error preserve full context?

The decorator shape is simple.

The policy is not.

Decorators can make hard behavior look too easy, so design carefully.

---

# Decorator Stacking

You can apply multiple decorators:

```python
@decorator_a
@decorator_b
def function():
    ...
```

This means:

```python
def function():
    ...


function = decorator_a(decorator_b(function))
```

The decorator closest to the function applies first.

Then the one above it applies to the result.

Call order depends on wrappers.

Example:

```python
def outer(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print("outer before")
        result = function(*args, **kwargs)
        print("outer after")
        return result
    return wrapper


def inner(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print("inner before")
        result = function(*args, **kwargs)
        print("inner after")
        return result
    return wrapper
```

Use:

```python
@outer
@inner
def greet():
    print("hello")
```

Call output:

```text
outer before
inner before
hello
inner after
outer after
```

Stacking order matters.

---

# Decorators on Methods

Decorators can wrap methods too.

Example:

```python
def log_calls(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        return function(*args, **kwargs)
    return wrapper
```

Use:

```python
class User:
    def __init__(self, name):
        self.name = name

    @log_calls
    def greet(self):
        return f"hello {self.name}"
```

When:

```python
user.greet()
```

is called, `self` is passed as the first positional argument to `wrapper`.

Then `wrapper` passes it to the original method:

```python
function(*args, **kwargs)
```

This is why general decorators use `*args, **kwargs`.

They work for functions and methods.

---

# Decorator Order with `@classmethod` and `@staticmethod`

Decorator order matters with method-transforming decorators.

Example:

```python
class User:
    @classmethod
    @log_calls
    def create(cls):
        return cls()
```

This first applies `log_calls` to the function, then `classmethod` wraps the result.

That is usually what you want.

This order may behave differently:

```python
class User:
    @log_calls
    @classmethod
    def create(cls):
        return cls()
```

Now `log_calls` receives a `classmethod` object, not a plain function.

That may not work with a decorator expecting a normal callable.

General rule:

```text
method-shaping decorators such as classmethod, staticmethod, and property usually belong closest to the function or in a carefully understood order
```

When stacking decorators, read from bottom to top for application.

Then think from outside to inside for call behavior.

---

# Class Decorators

Decorators can also decorate classes.

Example:

```python
def add_table_name(cls):
    cls.table_name = cls.__name__.lower()
    return cls
```

Use:

```python
@add_table_name
class User:
    pass
```

Now:

```python
User.table_name
```

is:

```python
"user"
```

This is equivalent to:

```python
class User:
    pass


User = add_table_name(User)
```

Class decorators are often simpler than metaclasses when you want to transform one class after creation.

Dataclasses use this idea:

```python
@dataclass
class Point:
    x: int
    y: int
```

The class object is passed through the `dataclass` decorator.

The decorator returns the processed class.

---

# Decorator Factories for Classes

Class decorators can also take arguments.

Example:

```python
def table(name):
    def decorator(cls):
        cls.table_name = name
        return cls

    return decorator
```

Use:

```python
@table("users")
class User:
    pass
```

Equivalent:

```python
class User:
    pass


User = table("users")(User)
```

This is often easier than a metaclass:

```python
class User(metaclass=ModelMeta, table="users"):
    ...
```

Use class decorators when the transformation is local and explicit.

Use metaclasses only when class creation itself needs deeper control or inherited behavior.

---

# Callable Class Decorators

A decorator can be a callable object.

Example:

```python
from functools import wraps


class CountCalls:
    def __init__(self, function):
        self.function = function
        self.count = 0
        wraps(function)(self)

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.function(*args, **kwargs)
```

Use:

```python
@CountCalls
def greet():
    return "hello"
```

Now:

```python
greet()
greet()
print(greet.count)
```

prints:

```python
2
```

This works because the decorator returns an object that is callable.

Function-based decorators are more common.

Callable class decorators are useful when the decorator needs persistent state.

---

# Standard Library Decorators

You already know several decorators.

`property`:

```python
class Circle:
    @property
    def area(self):
        return 3.14159 * self.radius ** 2
```

`classmethod`:

```python
class User:
    @classmethod
    def anonymous(cls):
        return cls("anonymous")
```

`staticmethod`:

```python
class Email:
    @staticmethod
    def normalize(value):
        return value.strip().lower()
```

`dataclass`:

```python
@dataclass
class Point:
    x: int
    y: int
```

`contextmanager`:

```python
@contextmanager
def managed():
    yield
```

`functools.cache` or `lru_cache`:

```python
@lru_cache(maxsize=128)
def fib(n):
    ...
```

These look different in purpose, but they share one mechanism:

```text
take an object, return a replacement object
```

---

# Caching Decorators

Caching is a common decorator use.

Example:

```python
from functools import lru_cache


@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

The decorator stores previous results.

Repeated calls with the same arguments can return cached values.

Design concerns:

* Are arguments hashable?
* Can results become stale?
* How much memory can the cache use?
* Should the cache be cleared?
* Is the function pure enough for caching?

Caching decorators are powerful because they change performance behavior without changing call sites.

They can also hide memory and freshness issues.

Use them deliberately.

---

# Registration Decorators

Frameworks often use decorators to register functions.

Example:

```python
routes = {}


def route(path):
    def decorator(function):
        routes[path] = function
        return function

    return decorator
```

Use:

```python
@route("/users")
def users():
    return "users"
```

The decorator runs when the function is defined.

It stores the function in `routes`.

It returns the original function.

This kind of decorator does not necessarily wrap behavior.

It records metadata or registers the function.

That is common in:

* web frameworks
* CLI frameworks
* task queues
* plugin systems
* test frameworks

Decorator does not always mean wrapper.

It means transformation or registration at definition time.

---

# Validation Decorators

Decorators can validate inputs.

Example:

```python
from functools import wraps


def require_positive(function):
    @wraps(function)
    def wrapper(value, *args, **kwargs):
        if value <= 0:
            raise ValueError("value must be positive")
        return function(value, *args, **kwargs)

    return wrapper
```

Use:

```python
@require_positive
def square_root(value):
    return value ** 0.5
```

This works.

But validation decorators can become awkward when function signatures differ.

This decorator assumes the first argument is the value to validate.

For broad validation, explicit checks inside the function or a dedicated validation library may be clearer.

Decorators are best when the validation rule is truly cross-cutting and the call shape is consistent.

---

# Authorization Decorators

Web frameworks often use decorators for authorization:

```python
def require_admin(function):
    @wraps(function)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_admin:
            raise PermissionError("admin required")
        return function(request, *args, **kwargs)

    return wrapper
```

Use:

```python
@require_admin
def delete_user(request, user_id):
    ...
```

This can be clear because authorization is a cross-cutting concern.

But too many decorators can hide the route's behavior.

Example:

```python
@route("/users/{id}")
@require_admin
@rate_limit("10/minute")
@audit_log
@transactional
def delete_user(request, id):
    ...
```

This may be appropriate in a framework.

It may also become hard to reason about.

Decorator stacks need discipline.

---

# Decorators and Context Managers

Decorators and context managers both wrap behavior.

Decorator:

```python
@timed
def work():
    ...
```

Context manager:

```python
with timer():
    work()
```

Use a decorator when the behavior should apply every time the function is called.

Use a context manager when the behavior should apply to a specific block.

Example:

```python
@timed
def build_index():
    ...
```

means every call is timed.

```python
with timer():
    build_index()
    save_index()
```

means this block is timed.

The scope differs.

Decorators attach behavior to definitions.

Context managers attach behavior to execution blocks.

---

# Decorators and Inheritance

Decorating a method changes that method object on the class.

Subclasses inherit the decorated method unless they override it.

Example:

```python
class Base:
    @log_calls
    def save(self):
        ...


class Child(Base):
    pass
```

`Child().save()` uses the decorated method.

If the subclass overrides:

```python
class Child(Base):
    def save(self):
        ...
```

the override is not decorated unless you decorate it too or call `super().save()`.

This matters when decorators enforce behavior such as permissions, transactions, or logging.

If the behavior must apply to all subclasses, an explicit base-class method pattern may be safer.

Decorators are local to the object they decorate.

Inheritance can bypass them through overriding.

---

# Decorators and Introspection

Tools often inspect functions.

They may read:

* `__name__`
* `__doc__`
* annotations
* signatures
* `__wrapped__`

Without `functools.wraps`, decorators can break introspection.

Example:

```python
def bad_decorator(function):
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)
    return wrapper
```

Now tools see:

```python
wrapper
```

instead of the original function name.

With:

```python
@wraps(function)
```

metadata is copied and `__wrapped__` points to the original.

This helps:

* documentation tools
* test tools
* web frameworks
* type inspection
* debugging
* decorator stacking

Use `wraps`.

It is small and important.

---

# Decorators and Type Hints

Decorators can confuse static type checkers if they change call signatures.

Simple wrapper:

```python
def log_calls(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)
    return wrapper
```

At runtime this works.

But type checkers may need help preserving the original signature.

Modern Python typing provides tools such as `ParamSpec` and `TypeVar` for well-typed decorators.

That belongs in the static typing chapter.

For now, remember:

```text
a decorator can preserve runtime behavior while obscuring static type information
```

The more a decorator changes arguments or return values, the more carefully it must be typed and documented.

---

# Decorators That Change Return Values

Some decorators intentionally change return values.

Example:

```python
def as_json(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        result = function(*args, **kwargs)
        return json.dumps(result)

    return wrapper
```

Use:

```python
@as_json
def user_data():
    return {"name": "Maya"}
```

Now:

```python
user_data()
```

returns a string, not a dictionary.

This can be useful.

It can also surprise callers.

If a decorator changes return type, make that clear in naming and documentation.

Invisible behavior changes make APIs harder to trust.

---

# Decorators That Change Arguments

Some decorators inject or modify arguments.

Example:

```python
def with_database(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        database = connect()
        try:
            return function(database, *args, **kwargs)
        finally:
            database.close()

    return wrapper
```

Use:

```python
@with_database
def load_users(database):
    return database.query("select * from users")
```

This works but changes how the function is called and understood.

Frameworks often do this.

Application code should be cautious.

Dependency injection through explicit arguments is often clearer.

Decorators that modify arguments should be obvious and well-tested.

---

# Decorator State

Decorators can hold state in closures.

Example:

```python
def count_calls(function):
    count = 0

    @wraps(function)
    def wrapper(*args, **kwargs):
        nonlocal count
        count += 1
        print(f"{function.__name__} called {count} times")
        return function(*args, **kwargs)

    return wrapper
```

Use:

```python
@count_calls
def greet():
    return "hello"
```

Each decorated function gets its own `count` variable.

Stateful decorators can be useful.

But think about:

* thread safety
* test isolation
* reset behavior
* memory use
* whether state belongs somewhere more explicit

Hidden state inside decorators can make debugging difficult.

---

# Decorating Classes Versus Metaclasses

Chapter 57 taught metaclasses.

Class decorators are often simpler.

Class decorator:

```python
def register(cls):
    registry[cls.__name__] = cls
    return cls
```

Use:

```python
@register
class JsonPlugin:
    pass
```

Metaclass:

```python
class RegistryMeta(type):
    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)
        registry[name] = cls
        return cls
```

The decorator is explicit.

The metaclass is inherited.

Use a class decorator when:

* one class opts into behavior
* transformation happens after class creation
* no custom namespace is needed
* inheritance-wide behavior is unnecessary

Use a metaclass only for deeper class-creation control.

---

# Decorating Functions Manually

Decorator syntax is optional.

This:

```python
@timed
def work():
    ...
```

is equivalent to:

```python
def work():
    ...


work = timed(work)
```

Manual decoration can be useful when decoration is conditional:

```python
def work():
    ...


if debug:
    work = log_calls(work)
```

Be cautious.

Conditional decoration can make behavior environment-dependent.

That may be useful for debugging.

It may also be confusing.

The `@` syntax is usually preferred when decoration is part of the function's definition.

---

# Common Mistake: Forgetting to Return the Wrapper

Bug:

```python
def decorator(function):
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)
```

There is no:

```python
return wrapper
```

Use:

```python
@decorator
def greet():
    return "hello"
```

Now `greet` becomes `None`, because the decorator returned `None`.

Correct:

```python
def decorator(function):
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

The decorator must return the replacement object.

---

# Common Mistake: Calling the Function Too Early

Bug:

```python
def decorator(function):
    return function()
```

This calls the function at decoration time.

That is usually wrong.

Correct wrapper:

```python
def decorator(function):
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

Decorators usually should return something callable for later.

They should not execute the decorated function immediately unless that is very intentionally the design.

---

# Common Mistake: Forgetting `*args` and `**kwargs`

Bug:

```python
def decorator(function):
    def wrapper():
        return function()

    return wrapper
```

This breaks decorated functions that need arguments.

General wrapper:

```python
def decorator(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

Use this shape unless the decorator is intentionally limited to a specific signature.

---

# Common Mistake: Forgetting `functools.wraps`

Bug:

```python
def decorator(function):
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

This works but damages metadata.

Better:

```python
from functools import wraps


def decorator(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

Make this muscle memory.

Professional decorators should preserve the original function's identity as much as possible.

---

# Common Mistake: Swallowing Exceptions

Bug:

```python
def safe(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        try:
            return function(*args, **kwargs)
        except Exception:
            return None

    return wrapper
```

This hides all errors.

Maybe the function failed because of bad input.

Maybe there is a bug.

Maybe the database is down.

Now callers only see `None`.

Catch narrow exceptions.

Handle only what you can handle.

Let unexpected errors propagate.

Decorators that hide failures are dangerous.

---

# Common Mistake: Too Many Decorators

This can become hard to read:

```python
@route("/orders/{id}")
@require_login
@require_permission("orders:read")
@rate_limit("60/minute")
@cache_response(30)
@trace
@transactional
def get_order(request, id):
    ...
```

Maybe every decorator is justified.

But the function's behavior is now spread across many wrappers.

Ask:

* Is the order obvious?
* Are errors easy to trace?
* Does each decorator preserve metadata?
* Are side effects documented?
* Would middleware, composition, or explicit code be clearer?

Decorators are wonderful until they become fog.

Use them to clarify repeated structure, not to bury behavior.

---

# Common Mistake: Confusing Decorators with Decorator Calls

This:

```python
@decorator
def function():
    ...
```

passes `function` to `decorator`.

This:

```python
@decorator()
def function():
    ...
```

calls `decorator()` first.

The result must itself be a decorator.

Example:

```python
@repeat(3)
def greet():
    ...
```

means:

```python
greet = repeat(3)(greet)
```

If a decorator takes no configuration, use:

```python
@decorator
```

If a decorator factory takes configuration, use:

```python
@decorator(...)
```

Some advanced decorators support both forms, but that adds complexity.

Start with one clear form.

---

# Common Mistake: Hiding Import-Time Side Effects

Registration decorators run when definitions execute.

Example:

```python
@route("/users")
def users():
    ...
```

The route registration usually happens at import time.

This is normal in many frameworks.

But import-time side effects can surprise people.

Avoid decorators that perform expensive work at import time:

* network calls
* database connections
* large file scans
* irreversible global changes

Registration is usually okay.

Heavy execution should usually wait until runtime.

---

# Design Checklist

Before writing a decorator, ask:

```text
Is this behavior truly cross-cutting?
```

If no, put it in the function body.

Ask:

```text
Should this apply to every call?
```

If no, use a context manager around selected calls.

Ask:

```text
Will the decorator preserve arguments, return values, metadata, and exceptions?
```

If yes, use `*args`, `**kwargs`, `return`, and `wraps`.

Ask:

```text
Does the decorator change the function's contract?
```

If yes, name and document it clearly.

Ask:

```text
Does order matter when stacked?
```

If yes, keep stacks short and clear.

Ask:

```text
Does decoration do work at import time?
```

If yes, keep it lightweight.

Ask:

```text
Would a function, context manager, class, descriptor, or middleware be clearer?
```

Decorators are one tool, not the only tool.

---

# Practice: Basic Decorator

Write a decorator that prints before and after a function call.

Solution:

```python
from functools import wraps


def announce(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print("before")
        try:
            return function(*args, **kwargs)
        finally:
            print("after")

    return wrapper
```

Use:

```python
@announce
def greet(name):
    print(f"hello {name}")
```

Call:

```python
greet("Maya")
```

---

# Practice: Timing Decorator

Write a timing decorator.

Solution:

```python
from functools import wraps
from time import perf_counter


def timed(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        start = perf_counter()
        try:
            return function(*args, **kwargs)
        finally:
            elapsed = perf_counter() - start
            print(f"{function.__name__}: {elapsed:.3f}s")

    return wrapper
```

Test:

```python
@timed
def add(a, b):
    return a + b


assert add(2, 3) == 5
```

The decorator should not change the return value.

---

# Practice: Decorator with Arguments

Write a decorator that repeats a function call.

Solution:

```python
from functools import wraps


def repeat(times):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = function(*args, **kwargs)
            return result

        return wrapper

    return decorator
```

Use:

```python
@repeat(3)
def greet():
    print("hello")
```

This prints `"hello"` three times.

---

# Practice: Registration Decorator

Write a command registry.

Solution:

```python
commands = {}


def command(name):
    def decorator(function):
        commands[name] = function
        return function

    return decorator
```

Use:

```python
@command("hello")
def hello():
    return "hello"
```

Test:

```python
assert commands["hello"] is hello
```

This decorator registers the function but does not wrap it.

---

# Practice: Class Decorator

Write a class decorator that adds a `slug` attribute.

Solution:

```python
def add_slug(cls):
    cls.slug = cls.__name__.lower()
    return cls
```

Use:

```python
@add_slug
class UserProfile:
    pass
```

Test:

```python
assert UserProfile.slug == "userprofile"
```

Class decorators receive and return class objects.

---

# Practice: Explain Stacking

Given:

```python
@a
@b
@c
def f():
    ...
```

What is the equivalent assignment?

Answer:

```python
def f():
    ...


f = a(b(c(f)))
```

Decorators apply from bottom to top.

The call behavior depends on what each decorator returns.

---

# Practice: Spot the Bug

What is wrong?

```python
def log(function):
    def wrapper(*args, **kwargs):
        print("calling")
        function(*args, **kwargs)

    return wrapper
```

Answer:

The wrapper does not return the wrapped function's result.

It also does not use `functools.wraps`.

Better:

```python
from functools import wraps


def log(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print("calling")
        return function(*args, **kwargs)

    return wrapper
```

---

# Summary

Decorators transform functions, methods, or classes at definition time.

Decorator syntax:

```python
@decorator
def function():
    ...
```

is roughly:

```python
function = decorator(function)
```

A function decorator usually returns a wrapper function.

Closures let the wrapper remember the original function.

General wrappers should accept `*args` and `**kwargs`, pass them through, and return the original result.

`functools.wraps` preserves metadata and should be used for most wrappers.

Decorators with arguments are decorator factories.

They add one more layer:

```text
factory arguments -> decorator -> wrapper
```

Multiple decorators apply from bottom to top.

Decorators can wrap methods, but order matters with `classmethod`, `staticmethod`, and `property`.

Class decorators transform class objects after creation.

Decorators can also register functions or classes without wrapping them.

Decorators are useful for logging, timing, caching, validation, authorization, registration, retries, framework routes, test markers, and class transformations.

They should be used carefully because they can hide behavior, change contracts, obscure metadata, swallow exceptions, or create import-time side effects.

The design principle is:

```text
use decorators when repeated definition-level behavior becomes clearer as a reusable wrapper or transformation
```

If behavior should apply only to a block, use a context manager.

If behavior belongs inside the function's core logic, keep it explicit.

If behavior changes the function's contract, make that change visible.

---

# Preview of Chapter 62

Chapter 61 completes Volume II Part III: Pythonic Abstractions.

We studied:

* iterators
* generators
* context managers
* decorators

These abstractions are Pythonic because they make common control-flow patterns reusable:

```text
iterators        -> consume values one at a time
generators       -> produce values lazily
context managers -> manage scoped setup and cleanup
decorators       -> transform definitions
```

Next we begin Part IV: Robust Programs and I/O.

Chapter 62 studies exceptions.

We have already seen exceptions in many places:

* `StopIteration` ends iterators.
* `__exit__` receives exception information.
* decorators can log, transform, suppress, or propagate errors.
* validation raises `ValueError` or `TypeError`.

Now we study exceptions directly.

Chapter 62 will cover:

* what exceptions are
* how `try`, `except`, `else`, and `finally` work
* exception hierarchy
* raising exceptions
* custom exception classes
* chaining exceptions
* exception groups
* when to catch and when to let errors propagate
* robust error-handling design

The transition is:

```text
Pythonic abstractions control flow
exceptions handle abnormal flow
```

Robust Python begins when errors are treated as part of the design, not as afterthoughts.

