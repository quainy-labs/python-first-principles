# Chapter 78 — Static Type Checking

Type hints make expectations visible.

Static type checking tests those expectations without running the program.

That is the difference between Chapter 77 and this chapter.

Chapter 77 explained the language of annotations:

```python
def find_user(email: str) -> User | None:
    ...
```

Chapter 78 explains what tools can do with that information.

A static type checker can look at code like this:

```python
user = find_user("asha@example.com")
send_email(user.email)
```

and report:

```text
user may be None
```

before the code runs.

That is valuable.

It catches a class of bugs earlier than tests, logs, and production tracebacks.

But static type checking is not magic.

It does not prove your program is correct.

It does not replace tests.

It does not validate external data at runtime.

It does not understand every dynamic Python pattern.

It is a tool.

Used well, it improves confidence and refactoring speed.

Used poorly, it becomes a noisy gate that developers learn to silence.

The craft is learning how to make static checking useful without pretending Python is a different language.

---

# What Static Type Checking Means

Static type checking means analyzing code before execution.

The checker reads source files, type hints, imports, and sometimes library stubs.

It builds a model of what values can flow where.

Then it reports mismatches.

Example:

```python
def greet(name: str) -> str:
    return "Hello, " + name


greet(123)
```

A type checker can report:

```text
Argument 1 to "greet" has incompatible type "int"; expected "str"
```

Python itself will still run the file if you ask it to.

The checker is separate from the interpreter.

That separation is essential:

```text
static type checking is feedback, not execution
```

You decide how strictly your project treats that feedback.

In professional projects, type checking is often part of CI, so new type errors block merges.

---

# Static Does Not Mean Correct

This code type-checks:

```python
def discount(price: int, percent: int) -> int:
    return price + (price * percent // 100)
```

The types are fine.

The behavior is wrong.

Discount should subtract, not add.

Static type checking catches type mismatches.

It does not understand all business rules.

Tests are still needed:

```python
def test_discount_subtracts_percent():
    assert discount(100, 20) == 80
```

Types and tests protect different dimensions.

Types ask:

```text
are values used in compatible ways?
```

Tests ask:

```text
does behavior match expectations?
```

Good engineering uses both.

---

# Static Does Not Mean Runtime

This code has annotations:

```python
def add(a: int, b: int) -> int:
    return a + b
```

At runtime:

```python
add("a", "b")
```

returns:

```text
ab
```

Python does not automatically enforce the annotations.

A static checker can flag the call before runtime.

But if you never run the checker, the annotation does not protect anything.

This is a common beginner mistake:

```text
I added type hints, so my code is type safe.
```

Not quite.

Type hints are information.

Static type checking is a process that uses that information.

---

# Popular Type Checkers

Python has several type checkers.

Common ones include:

* mypy
* Pyright
* basedpyright
* Pyre
* pytype

The two most commonly encountered in many projects are mypy and Pyright.

mypy is a long-standing Python type checker with extensive configuration and ecosystem support.

Pyright is a fast type checker from Microsoft, also used as the engine behind Pylance in many editor workflows.

Different tools may disagree in edge cases.

That is normal.

The Python type system has specifications, but tool behavior can differ around strictness, inference, incomplete libraries, and newer features.

Choose a checker for the project.

Then configure it consistently.

Do not switch tools casually without reviewing differences.

---

# Installing and Running mypy

mypy can be installed with pip:

```bash
python -m pip install mypy
```

Run it on a file:

```bash
mypy program.py
```

Run it on a package or directory:

```bash
mypy src tests
```

The command prints errors it finds.

For example:

```text
src/shop/email.py:12: error: Argument 1 to "send" has incompatible type "int"; expected "str"
```

The pieces are:

```text
file
line
error message
```

Sometimes there is also an error code.

Read type errors carefully.

They are often telling you where an assumption became inconsistent.

---

# Running Pyright

Pyright is commonly installed through Node tooling or used through editor integrations.

In many projects, a command may look like:

```bash
pyright
```

or:

```bash
npx pyright
```

Pyright reads project configuration and reports type errors.

In editor workflows, you may see Pyright diagnostics live as you type.

The important workflow idea is the same:

```text
run the checker on the project
review diagnostics
fix real mismatches
configure intentional boundaries
```

This chapter uses many mypy-shaped examples because mypy's CLI and messages are familiar in Python projects.

The concepts apply broadly.

---

# A First Example

Consider:

```python
def normalize_email(email: str) -> str:
    return email.strip().lower()


value = normalize_email(None)
```

At runtime, this fails:

```text
AttributeError: 'NoneType' object has no attribute 'strip'
```

A type checker can catch it earlier:

```text
Argument 1 to "normalize_email" has incompatible type "None"; expected "str"
```

The fix depends on the intended behavior.

If `None` is invalid:

```python
value = normalize_email(email)
```

where `email` is guaranteed to be a string.

If `None` is allowed:

```python
def normalize_email(email: str | None) -> str | None:
    if email is None:
        return None
    return email.strip().lower()
```

The type error forces the contract to become explicit.

That is the point.

---

# Missing None Handling

One of the biggest benefits of static type checking is finding missing `None` handling.

Example:

```python
def find_user(email: str) -> User | None:
    ...


def send_welcome(email: str) -> None:
    user = find_user(email)
    send_email(user.email)
```

The problem is:

```text
find_user can return None
```

Fix:

```python
def send_welcome(email: str) -> None:
    user = find_user(email)
    if user is None:
        raise LookupError(f"user not found: {email}")
    send_email(user.email)
```

Now the type checker understands that after the `None` check, `user` is a `User`.

This is type narrowing.

It turns a possible runtime `AttributeError` into an earlier design decision:

```text
what should happen when the user is missing?
```

---

# Incorrect Return Types

Type checkers can detect functions that return the wrong shape.

Example:

```python
def user_count() -> int:
    return "10"
```

The annotation promises `int`.

The implementation returns `str`.

The checker can report this mismatch.

Sometimes the fix is to change the code:

```python
def user_count() -> int:
    return 10
```

Sometimes the annotation was wrong:

```python
def user_count() -> str:
    return "10"
```

Do not blindly change annotations to silence errors.

Ask:

```text
which contract is correct?
```

Static checking is most useful when annotations represent real promises.

---

# Attribute Errors Before Runtime

Type checkers can catch impossible attribute access.

Example:

```python
def format_user(user: dict[str, str]) -> str:
    return user.email
```

A dictionary has keys.

It does not have an `email` attribute.

The code should be:

```python
def format_user(user: dict[str, str]) -> str:
    return user["email"]
```

Or the type should be a class:

```python
@dataclass
class User:
    email: str


def format_user(user: User) -> str:
    return user.email
```

The type error exposes a data modeling question:

```text
is this a dictionary-shaped payload or a domain object?
```

Good type checking often improves modeling.

---

# Untyped Functions

Many checkers treat unannotated functions more loosely.

Example:

```python
def greet(name):
    return "Hello " + name


greet(123)
```

Depending on configuration, the checker may report little or nothing.

Why?

Because without annotations, the function is dynamically typed.

To get useful checking, annotate boundaries:

```python
def greet(name: str) -> str:
    return "Hello " + name
```

Now:

```python
greet(123)
```

can be reported.

Static checking quality depends on annotation quality.

Untyped areas are like fog on the map.

---

# check-untyped-defs

Some tools can check bodies of untyped functions more aggressively.

In mypy, one option is:

```bash
mypy --check-untyped-defs src
```

This asks mypy to inspect untyped function bodies too.

It can catch more errors during migration.

But without function signatures, the checker still has less information.

For example, it may infer `Any` at boundaries and miss problems.

The best long-term improvement is annotating important function signatures.

Checking untyped bodies is helpful.

It is not a substitute for clear API annotations.

---

# Strict Mode

Many type checkers provide stricter modes.

In mypy:

```bash
mypy --strict src
```

Strict mode enables many additional checks.

It can be excellent for new code.

It can be overwhelming for large existing codebases.

Strict mode may complain about:

* untyped function definitions
* calls to untyped functions
* implicit optional behavior
* unused ignores
* incomplete annotations
* returning `Any`
* missing type parameters

Do not treat strict mode as a moral badge.

Treat it as a configuration choice.

For a new library, strict from the beginning can work beautifully.

For a large legacy application, gradual tightening may be healthier.

---

# Gradual Adoption

Static type checking works best when adoption is deliberate.

For an existing codebase, a practical strategy is:

1. Add the checker with a permissive configuration.
2. Run it in CI without blocking, or block only changed files.
3. Annotate high-value boundaries.
4. Fix obvious errors.
5. Add stricter checks module by module.
6. Prevent new untyped code in selected areas.
7. Reduce `Any` and ignores over time.

Do not demand perfection on day one.

That often leads to mass ignores.

Mass ignores create the appearance of type safety without the substance.

Gradual adoption should move in one direction:

```text
less unknown
more checked
fewer ignores
clearer contracts
```

---

# New Code vs Existing Code

New code can usually start with stronger rules.

Existing code may need migration.

A good policy might be:

```text
new modules must be typed
old modules are typed when touched
critical modules get prioritized
public APIs are typed first
```

This avoids freezing progress while still improving the codebase.

For existing systems, prioritize:

* payment logic
* permissions
* data transformations
* public library APIs
* frequently changed modules
* modules with many bugs
* interfaces between services

Typing the most boring module first may be easy.

Typing the risky boundary first may be more valuable.

---

# Configuration Files

Static checkers should be configured in the project.

mypy can use configuration in files such as:

* `pyproject.toml`
* `mypy.ini`
* `setup.cfg`

Example in `pyproject.toml`:

```toml
[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_ignores = true
disallow_untyped_defs = true
```

Configuration makes behavior reproducible.

Do not rely on one developer's editor settings as the source of truth.

The project should define:

* what gets checked
* which Python version is targeted
* how strict checks are
* how missing imports are handled
* where exceptions are allowed

CI should use the same configuration.

---

# Python Version Settings

Type checking depends on the target Python version.

For example, this syntax:

```python
def first[T](items: list[T]) -> T:
    ...
```

requires newer Python syntax than older `TypeVar` style.

If your project supports older Python versions, the checker must know.

Configuration example:

```toml
[tool.mypy]
python_version = "3.10"
```

If the checker assumes Python 3.13 but your package supports Python 3.10, it may allow syntax or library features your users cannot run.

Type checker configuration should match packaging metadata.

Chapter 76's `requires-python` and Chapter 78's checker target should agree.

---

# Type Checking in CI

Local type checking is useful.

CI type checking is enforceable.

A typical CI step:

```bash
mypy src tests
```

or:

```bash
pyright
```

CI ensures everyone sees the same checks.

It prevents new type errors from entering the main branch.

For gradual adoption, CI can start with a subset:

```bash
mypy src/shop/payments src/shop/orders
```

Then expand.

The rule should be clear:

```text
checked areas stay checked
```

Do not let type checking become optional after the team depends on it.

---

# Editor Integration

Type checkers are valuable in editors.

They can show errors while you type.

They can improve:

* autocomplete
* jump to definition
* rename refactoring
* signature help
* unreachable code hints
* missing attribute warnings

This feedback shortens the loop.

Instead of waiting for CI, you see mismatches immediately.

But editor settings should not be the only source of truth.

Use project configuration and CI.

The editor is the fast feedback layer.

CI is the shared enforcement layer.

---

# reveal_type

`reveal_type` is a debugging tool for type checkers.

Example:

```python
def process(value: str | None) -> None:
    reveal_type(value)
    if value is not None:
        reveal_type(value)
```

A checker may report:

```text
Revealed type is "str | None"
Revealed type is "str"
```

This helps you understand what the checker believes.

Use `reveal_type` when:

* a type error is confusing
* narrowing is not working
* generics infer an unexpected type
* a value became `Any`
* a library stub behaves unexpectedly

Remove `reveal_type` after debugging unless your tool treats it specially and your project permits it.

It is a diagnostic probe, not production logic.

---

# Any Leakage

`Any` can spread through code.

Example:

```python
from typing import Any


def load_config() -> Any:
    ...


config = load_config()
config.user.email.this.does.not.exist()
```

The checker may allow this because `config` is `Any`.

This is called `Any` leakage.

It weakens static checking.

Common sources:

* untyped functions
* untyped third-party libraries
* JSON parsing
* dynamic imports
* `cast`
* ignored imports
* missing stubs

Contain `Any` at boundaries.

Convert unknown data into known shapes:

```python
def load_config() -> Config:
    raw = read_config_file()
    return parse_config(raw)
```

The sooner unknown data becomes typed data, the more useful the checker becomes.

---

# Missing Imports and Stubs

Third-party libraries may not provide type information.

A checker may report missing stubs or missing type information.

Options include:

* install a stub package
* upgrade the library
* write local stubs
* ignore that import
* wrap the library behind typed code

Stub packages are often named like:

```text
types-requests
types-PyYAML
```

Be careful with broad ignores:

```toml
ignore_missing_imports = true
```

This can turn many imported values into `Any`.

A better approach is often per-module ignores for specific libraries:

```toml
[[tool.mypy.overrides]]
module = ["some_untyped_library.*"]
ignore_missing_imports = true
```

The goal is to isolate untyped boundaries, not flood the project with `Any`.

---

# Stub Files

Stub files use the `.pyi` extension.

They describe types without implementation.

Example:

```python
# external_client.pyi
class Client:
    def fetch_user(self, user_id: str) -> dict[str, object]: ...
```

Stubs are useful when:

* the source is untyped
* the source is generated
* the implementation is in C
* you need local types for an untyped dependency
* you want to type a boundary without modifying runtime code

Stubs must match runtime behavior.

Wrong stubs are dangerous because they create false confidence.

Treat stubs as code.

Review them.

Test important assumptions where possible.

---

# py.typed and Typed Packages

Packages that want to expose inline type hints to users should include a `py.typed` marker.

This connects to Chapter 76.

Example package:

```text
src/
    clientlib/
        __init__.py
        py.typed
        api.py
```

The marker tells type checkers:

```text
this package provides type information
```

If a typed library forgets `py.typed`, users may not get the intended checking.

Static type checking is not only code annotation.

It is also packaging metadata.

---

# Error Codes

Many type checkers include error categories or codes.

Example shape:

```text
error: Item "None" of "User | None" has no attribute "email"  [union-attr]
```

The code helps identify the kind of issue.

It also allows targeted ignores:

```python
send_email(user.email)  # type: ignore[union-attr]
```

Targeted ignores are better than broad ignores:

```python
send_email(user.email)  # type: ignore
```

A broad ignore may hide future unrelated errors on the same line.

Use error codes to be precise.

---

# type: ignore

Sometimes the checker is wrong or cannot understand a valid pattern.

You can ignore a line:

```python
value = dynamic_library.magic()  # type: ignore[no-untyped-call]
```

Use ignores sparingly.

A good ignore has:

* a specific error code
* a short reason when helpful
* a narrow scope

Example:

```python
plugin = load_plugin(name)  # type: ignore[no-any-return]  # plugin API is dynamic
```

Bad:

```python
result = do_everything()  # type: ignore
```

An ignore is a debt marker.

Sometimes debt is reasonable.

Invisible debt is not.

---

# warn_unused_ignores

Unused ignores should be removed.

If an ignored error no longer exists, the ignore is now hiding nothing useful.

Configuration:

```toml
[tool.mypy]
warn_unused_ignores = true
```

This helps keep the codebase clean.

Ignores should not become permanent wallpaper.

When the checker improves or the code changes, remove obsolete ignores.

---

# cast and Static Checking

`cast` tells the checker to trust you.

Example:

```python
from typing import cast


user = cast(User, load_from_cache("user"))
```

At runtime, this does not validate anything.

It returns the original value.

Use `cast` when you know more than the checker and cannot easily express it.

But prefer real narrowing or parsing when possible.

Better:

```python
value = load_from_cache("user")
if not isinstance(value, User):
    raise TypeError("cache value is not User")
user = value
```

`cast` is a static assertion.

It should not become a substitute for runtime checks at unsafe boundaries.

---

# Type Narrowing in Practice

Type narrowing is how code proves a value is more specific.

Example:

```python
def length(value: str | None) -> int:
    if value is None:
        return 0
    return len(value)
```

The checker knows `value` is `str` after the `None` return.

For unions:

```python
def normalize(value: int | str) -> str:
    if isinstance(value, int):
        return str(value)
    return value.strip()
```

The checker knows the final branch is `str`.

When narrowing fails, code may be unclear.

Sometimes introducing a local variable or simpler branch helps both humans and tools.

---

# Exhaustiveness

Some checkers can help with exhaustive handling.

Example with literals:

```python
from typing import Literal


Status = Literal["pending", "paid", "failed"]


def label(status: Status) -> str:
    if status == "pending":
        return "Pending"
    if status == "paid":
        return "Paid"
    if status == "failed":
        return "Failed"
    raise AssertionError("unreachable")
```

If a new status is added:

```python
Status = Literal["pending", "paid", "failed", "refunded"]
```

a checker may help reveal missing handling depending on configuration and patterns.

This is especially useful for:

* enums
* literals
* tagged unions
* state machines

Static checking can make state changes safer.

---

# Overrides

When a subclass overrides a method, its signature must remain compatible.

Example:

```python
class Sender:
    def send(self, message: str) -> None:
        ...


class EmailSender(Sender):
    def send(self, message: int) -> None:
        ...
```

This is unsafe.

Code expecting a `Sender` may pass a string.

The subclass expects an integer.

A type checker can catch invalid overrides.

Modern typing also provides an `override` decorator in newer Python versions:

```python
from typing import override


class EmailSender(Sender):
    @override
    def send(self, message: str) -> None:
        ...
```

This helps tools detect when a method was meant to override but does not.

---

# Protocol Checking

Protocols become more useful with static checking.

Example:

```python
class EmailSender(Protocol):
    def send(self, to: str, subject: str, body: str) -> None:
        ...


def welcome(sender: EmailSender) -> None:
    sender.send("a@example.com", "Welcome", "Hello")
```

Implementation:

```python
class BadSender:
    def send(self, to: str) -> None:
        ...
```

A checker can report that `BadSender` does not satisfy `EmailSender`.

No inheritance is required.

This gives Python's duck typing a static feedback layer.

The object still runs dynamically.

The checker verifies the expected shape before runtime.

---

# TypedDict Checking

`TypedDict` helps check dictionary-shaped data.

Example:

```python
class UserPayload(TypedDict):
    id: int
    email: str


def send(payload: UserPayload) -> None:
    print(payload["email"])
```

The checker can detect:

```python
send({"id": 1})
```

because `email` is missing.

It can also detect:

```python
send({"id": "1", "email": "a@example.com"})
```

because `id` should be an integer.

This is useful for JSON-like structures.

But remember: external JSON still needs runtime validation.

`TypedDict` helps static code.

It does not verify incoming network data by itself.

---

# Generic Errors

Generics can produce confusing errors.

Example:

```python
def first[T](items: list[T]) -> T:
    return items[0]


value = first([])
```

What is `T`?

The empty list gives little information.

You may need an annotation:

```python
items: list[int] = []
value = first(items)
```

Generic inference depends on context.

When errors get strange, simplify:

* add intermediate variables
* annotate empty collections
* use `reveal_type`
* reduce complex expressions
* check which value became `Any`

Generics are powerful because they preserve relationships.

They require readable code to work well.

---

# Invariance

Some generic containers are invariant.

This surprises people.

Example concept:

```python
class Animal: ...
class Dog(Animal): ...
```

A `Dog` is an `Animal`.

But `list[Dog]` is not safely a `list[Animal]`.

Why?

If a function accepts `list[Animal]`, it may append a `Cat`.

That would break a list that was actually supposed to contain only dogs.

Mutable containers are often invariant because mutation can violate type safety.

Read-only abstractions can be more flexible.

For example:

```python
from collections.abc import Sequence


def names(animals: Sequence[Animal]) -> list[str]:
    ...
```

A `Sequence[Dog]` can often be used where `Sequence[Animal]` is expected because the function cannot append a cat.

Mutability affects type relationships.

---

# Dynamic Python Patterns

Python supports dynamic patterns that type checkers may not fully understand.

Examples:

* adding attributes at runtime
* dynamic imports
* monkey patching
* metaclass-generated methods
* decorators that change signatures
* ORMs with dynamic fields
* plugin systems
* `__getattr__`
* `setattr`

Static checkers approximate Python.

They do not execute arbitrary dynamic behavior.

When dynamic patterns are important, options include:

* protocols
* stubs
* casts
* typed wrapper functions
* plugin support for the checker
* reducing dynamic behavior at boundaries

Do not fight the checker with random ignores.

Model the dynamic behavior where possible.

Ignore only where necessary.

---

# Decorators and Type Checking

Decorators can hide function signatures.

Bad decorator:

```python
def trace(func):
    def wrapper(*args, **kwargs):
        print("calling")
        return func(*args, **kwargs)
    return wrapper
```

The wrapper loses type information.

Better:

```python
from collections.abc import Callable
from functools import wraps
from typing import ParamSpec, TypeVar


P = ParamSpec("P")
R = TypeVar("R")


def trace(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print("calling")
        return func(*args, **kwargs)
    return wrapper
```

This preserves the callable signature.

Decorators are a place where advanced typing can pay off.

But do not start here if the basics are not yet stable.

---

# Overloads and Checkers

Overloads are checked statically.

Example:

```python
@overload
def get(name: str) -> str: ...


@overload
def get(name: None) -> None: ...


def get(name: str | None) -> str | None:
    if name is None:
        return None
    return name.upper()
```

The checker uses overloads to understand calls:

```python
value1 = get("asha")  # str
value2 = get(None)    # None
```

Overloads must match the implementation.

If overloads promise behavior the implementation does not provide, the type system becomes misleading.

Use overloads for real API precision, not for wishful thinking.

---

# Tests and Type Checking Together

Type checking and tests should run together.

Example project checks:

```bash
pytest
mypy src tests
```

or:

```bash
pytest
pyright
```

Tests catch behavior.

Type checking catches shape mismatches.

One can catch what the other misses.

Example:

```python
def apply_discount(price: int, percent: int) -> int:
    return price + percent
```

Types pass.

Tests fail.

Example:

```python
def normalize(email: str) -> str:
    return email.strip().lower()


normalize(None)
```

Type checking catches it.

The best feedback loop uses both.

---

# Type Checking Tests

Should tests be type checked?

Often yes.

Tests contain helper functions, fixtures, factories, mocks, and important examples.

Type checking tests can catch:

* wrong fixture usage
* outdated test helpers
* incorrect factory data
* API misuse
* stale assumptions after refactors

But tests may use dynamic patterns more heavily than production code.

You may choose different strictness for tests.

Example:

```toml
[[tool.mypy.overrides]]
module = ["tests.*"]
disallow_untyped_defs = false
```

Do not ignore tests entirely by default.

But configure pragmatically.

---

# Type Checking and Mocking

Mocks can weaken type checking because plain mocks behave like anything.

Example:

```python
sender = Mock()
sender.sned("a@example.com")
```

The typo may not be caught.

Options:

* use `autospec`
* use typed fake classes
* annotate mocks with protocols
* prefer fakes for domain boundaries

Example:

```python
class FakeEmailSender:
    def __init__(self) -> None:
        self.messages: list[Message] = []

    def send(self, message: Message) -> None:
        self.messages.append(message)
```

This fake is both testable and type-checkable.

Chapter 73's mocking judgment applies here too.

Overly dynamic tests reduce static confidence.

---

# Type Checking External Data

External data is unknown at runtime.

Examples:

* JSON request bodies
* environment variables
* files
* database rows
* message queues
* third-party API responses

Do not simply cast external data into trusted types.

Weak:

```python
payload = cast(UserPayload, json.loads(body))
```

Better:

```python
payload = parse_user_payload(json.loads(body))
```

where `parse_user_payload` validates and returns a typed shape.

Static checking helps after validation.

Runtime validation protects the boundary.

The boundary pattern is:

```text
unknown input -> validate/parse -> typed internal value
```

This is one of the most important professional uses of typing.

---

# Type Checking Public Libraries

For public libraries, static types affect users.

If your function is annotated:

```python
def connect(timeout: int) -> Client:
    ...
```

users' editors and type checkers rely on that.

Changing the annotation can be a compatibility change.

For example, changing:

```python
def find_user(email: str) -> User:
    ...
```

to:

```python
def find_user(email: str) -> User | None:
    ...
```

is a meaningful API change.

It tells users they must now handle absence.

Maybe the runtime always behaved that way.

But the typed contract changed.

Public annotations deserve review.

---

# Type Checking Applications

Applications use type checking differently from libraries.

An application controls its own callers.

It can be stricter internally.

It can add types around:

* settings
* request models
* service boundaries
* database access
* background jobs
* external clients
* domain objects

Applications often benefit from typed configuration and typed boundaries.

Many production bugs come from:

* missing environment variable
* wrong JSON shape
* optional database value treated as required
* external API field absent
* feature flag value unexpected

Static checking cannot validate raw inputs.

But once inputs are parsed into typed internal models, it helps preserve assumptions.

---

# Type Checking and Refactoring

Static type checking shines during refactoring.

Suppose you change:

```python
def send_email(to: str, subject: str, body: str) -> None:
    ...
```

to:

```python
def send_email(message: EmailMessage) -> None:
    ...
```

A type checker can find old call sites:

```python
send_email(user.email, "Welcome", "Hello")
```

This is faster and safer than manually searching.

The type checker becomes a refactoring assistant.

It does not know the business goal.

It knows which code no longer matches the declared shape.

That is extremely useful.

---

# Type Checking and Dead Code

Some type checkers can detect unreachable code or impossible checks.

Example:

```python
def process(value: str) -> None:
    if value is None:
        return
```

If `value` is annotated as `str`, the `None` branch is impossible.

This may reveal:

* the annotation is wrong
* the check is obsolete
* the function used to accept `None`
* callers changed

Unreachable-code diagnostics are not merely cleanup.

They reveal outdated assumptions.

---

# False Positives

A false positive is an error report for code that is actually safe.

They happen.

Reasons include:

* checker limitation
* dynamic framework behavior
* incomplete stubs
* complex control flow
* unsupported pattern
* code that is correct but hard to model

Do not react with:

```text
type checking is useless
```

Instead ask:

* can I write the code more clearly?
* can I add a protocol?
* can I add a helper variable?
* can I narrow the type explicitly?
* is a small `cast` justified?
* is a targeted ignore appropriate?

Sometimes the checker is wrong.

Sometimes the code is clever in a way humans also find hard to understand.

False positives are engineering tradeoffs.

---

# False Negatives

A false negative is a bug the checker misses.

Examples:

* business logic error
* untyped `Any` value
* runtime data shape mismatch
* incorrect cast
* wrong stub
* mutation through aliasing
* concurrency bug

Static checking is not proof.

If a checker passes, that means:

```text
the checker did not find a type inconsistency under its model
```

It does not mean:

```text
the program is correct
```

Keep testing.

Keep reviewing.

Keep validating external inputs.

---

# Too Much Strictness Too Soon

Strict checking can fail as an adoption strategy if introduced carelessly.

Symptoms:

* hundreds of errors
* developers add broad ignores
* CI becomes noisy
* type checking is seen as punishment
* nobody trusts the output

Better:

```text
start where value is high
make progress visible
keep checked areas clean
increase strictness deliberately
```

Type checking should create useful pressure.

Not despair.

The goal is long-term code health.

---

# Too Little Strictness Forever

The opposite problem is adding a checker but never tightening it.

Symptoms:

* checker always passes because everything is `Any`
* untyped functions dominate
* missing imports are globally ignored
* public APIs are not annotated
* errors are ignored without reasons

This creates type-checking theater.

The project can claim it uses static typing, but the checker has little information.

A healthy setup should improve over time.

Track:

* number of typed modules
* number of ignores
* amount of `Any`
* strictness of critical packages
* CI coverage

Progress matters more than slogans.

---

# Useful Strictness Options

Different tools have different names, but useful checks include:

* disallow untyped function definitions
* check untyped function bodies
* warn about returning `Any`
* warn about unused ignores
* disallow untyped calls in checked code
* require generic type parameters
* warn about unreachable code
* require explicit optional handling

Do not enable options blindly.

Understand what each option protects.

For example:

```text
warn_return_any
```

helps prevent `Any` from leaking out of a function.

```text
disallow_untyped_defs
```

ensures functions have annotations.

Each strictness option changes the pressure on the codebase.

Use them intentionally.

---

# Per-Module Strictness

You can apply stricter rules to some modules than others.

Example:

```toml
[tool.mypy]
python_version = "3.12"
warn_unused_ignores = true

[[tool.mypy.overrides]]
module = ["shop.payments.*", "shop.permissions.*"]
disallow_untyped_defs = true
warn_return_any = true
```

This lets critical modules become stricter first.

Per-module strictness is useful for gradual adoption.

It avoids the false choice between:

```text
everything strict today
nothing strict ever
```

There is a middle path.

Use it.

---

# Baselines

Some teams use a baseline file to record existing type errors.

The workflow is:

```text
existing errors are known
new errors are blocked
baseline shrinks over time
```

Not every tool supports baselines the same way.

You can also approximate this by checking only selected modules or changed files.

The principle is useful:

```text
do not let legacy errors block adoption
do not allow new errors casually
```

Baselines should shrink.

If they never shrink, they become a landfill.

---

# Type Checking Generated Code

Generated code can be difficult to type check.

Examples:

* ORM models
* API clients generated from schemas
* protobuf code
* dynamic settings modules
* parser outputs

Options include:

* generated stubs
* checker plugins
* excluding generated files
* typed wrappers around generated code
* configuring lower strictness

Do not waste days fighting generated code if a typed boundary can contain it.

Example:

```python
def get_user(client: GeneratedClient, user_id: str) -> User:
    raw = client.fetch_user(user_id)
    return parse_user(raw)
```

The generated client may be messy.

The rest of your code can use `User`.

---

# Type Checking Dynamic Frameworks

Frameworks often use dynamic behavior.

Examples:

* Django models
* SQLAlchemy models
* Pydantic models
* dependency injection systems
* web route decorators
* plugin loaders
* serializers

Many frameworks provide plugins, stubs, or recommended patterns.

Use them when they exist.

If not, create typed boundaries.

Example:

```python
def current_user(request: Request) -> User:
    user = request.state.user
    if not isinstance(user, User):
        raise RuntimeError("request has no authenticated user")
    return user
```

Now the rest of the code can use a real `User` type.

Dynamic framework edges should be converted into typed application concepts.

---

# Type Checker Directives

Directives tell a checker something about code.

Examples:

```python
# type: ignore[assignment]
```

```python
from typing import cast
```

```python
from typing import assert_type
```

```python
from typing import reveal_type
```

Directives are powerful because they affect analysis.

Use them with care.

If your code is full of directives, the checker may be serving the directives rather than the program.

The best typed code usually needs few special comments.

---

# assert_type

`assert_type` can be used to assert what a type checker should infer.

Example:

```python
from typing import assert_type


value = first([1, 2, 3])
assert_type(value, int)
```

This is useful in tests for type-heavy libraries.

Most application code does not need it.

It is more common when you maintain APIs where type inference itself is part of the contract.

For ordinary debugging, `reveal_type` is more common.

---

# Type Checking Libraries With Tests

Typed libraries may test their type behavior.

This can include:

* checking example code
* using `assert_type`
* running checker against sample files
* testing stubs with stub-testing tools

Why?

Because public type hints are part of the library's API.

If a library promises:

```python
def first[T](items: Sequence[T]) -> T:
    ...
```

it should preserve that inference behavior.

Library authors need to think about type users, not only runtime users.

---

# Type Checking and Documentation

Type errors often reveal documentation gaps.

Example:

```python
def find_user(email: str) -> User | None:
    ...
```

Documentation should mention what `None` means:

```text
Returns None when no user exists for the email.
```

Type hints show shape.

Documentation explains semantics.

When static checking forces a contract to become precise, update documentation too.

---

# Type Checking and API Design

Some APIs are hard to type because they are unclear.

Example:

```python
def configure(option, value):
    ...
```

What options exist?

What values match each option?

Can the return type vary?

Trying to type this may reveal that the API should be redesigned.

Maybe use:

```python
@dataclass
class Config:
    timeout_seconds: int
    debug: bool
    api_url: str
```

instead of arbitrary option/value pairs.

Static typing pressures APIs toward explicitness.

That pressure is often useful.

---

# When Not to Add More Types

More typing is not always better immediately.

Be cautious when:

* the design is still exploratory
* the code is throwaway
* the annotation would be much harder to read than the code
* the checker cannot model the framework well
* a simple test would provide better confidence
* runtime validation is the real issue

Type hints are a tool for clarity.

If an annotation harms clarity, reconsider.

Maybe the code needs redesign.

Maybe the type should be an alias.

Maybe strictness can wait.

Good engineering does not maximize type syntax.

It maximizes maintainable understanding.

---

# A Practical Static Typing Workflow

A healthy workflow for a project might be:

1. Add type hints to public and high-value functions.
2. Run a type checker locally.
3. Use `reveal_type` to understand confusing inference.
4. Fix real errors by correcting code or annotations.
5. Add runtime validation at external boundaries.
6. Use targeted ignores only when justified.
7. Configure the checker in the project.
8. Run the checker in CI.
9. Tighten strictness gradually.
10. Review type changes like API changes.

This workflow keeps type checking connected to engineering value.

The goal is not a green checker at any cost.

The goal is a checker that tells the truth often enough to be trusted.

---

# Common Static Type Checking Mistakes

Common mistakes include:

* adding hints but never running a checker
* assuming checker success means program correctness
* using `Any` to silence everything
* using broad `type: ignore`
* ignoring missing imports globally
* enabling strict mode on a legacy codebase without a migration plan
* never tightening after initial adoption
* treating runtime validation as unnecessary
* fighting dynamic framework behavior instead of creating typed boundaries
* not checking CI
* forgetting tests still matter
* publishing typed libraries without checking their public API
* mismatching checker Python version with package metadata

These mistakes are fixable.

The cure is to treat static checking as an engineering system, not a decoration.

---

# Static Type Checking Checklist

When using static checking, ask:

* Is the checker configured in the repository?
* Does the target Python version match the package metadata?
* Are important public APIs annotated?
* Are high-risk modules checked?
* Are untyped imports isolated?
* Is `Any` contained?
* Are ignores targeted and justified?
* Are unused ignores reported?
* Are tests checked where useful?
* Does CI run the checker?
* Are external inputs validated at runtime?
* Are type changes reviewed as API changes?
* Is strictness improving over time?

This checklist keeps type checking from becoming theater.

The value is not in saying:

```text
we use types
```

The value is in catching real mistakes earlier.

---

# Chapter Summary

Static type checking analyzes Python code before execution using type hints, inference, stubs, and configuration.

Python remains dynamically typed at runtime.

A type checker is a separate feedback tool.

Type checking catches shape mismatches such as incompatible arguments, missing `None` handling, incorrect return values, impossible attribute access, invalid overrides, and mismatched generic types.

It does not prove business correctness.

It does not replace tests.

It does not validate external data at runtime.

mypy and Pyright are common type checkers, though projects may use others.

Checker behavior depends on configuration.

Untyped functions and `Any` reduce checker usefulness.

Gradual adoption is often the best strategy for existing codebases.

New code can usually be stricter from the start.

Checker configuration should live in the repository and run in CI.

`reveal_type` helps inspect what the checker believes.

Stub files and stub packages provide type information for code that lacks inline annotations.

`py.typed` tells type checkers that a package distributes inline type information.

Targeted `type: ignore[code]` comments are better than broad ignores.

`cast` should be used sparingly because it does not validate at runtime.

Static checking works best when unknown external data is parsed into typed internal values.

Dynamic Python patterns can be modeled with protocols, stubs, wrappers, plugins, casts, or narrow ignores.

Tests and type checking complement each other.

The central lesson is:

```text
static type checking turns visible expectations into early feedback
```

That feedback is valuable when it is accurate, trusted, and part of the normal engineering workflow.

---

# Exercises

1. Install a type checker and run it on a small typed file.

2. Create a function that forgets to handle `None`, then use a checker to find the error.

3. Add a `pyproject.toml` checker configuration for a small project.

4. Use `reveal_type` to inspect a value before and after an `is None` check.

5. Add a targeted `type: ignore[code]` with a short reason, then remove it by improving the code.

6. Install or create a stub for an untyped dependency.

7. Convert a value from `Any` into a typed dataclass at a boundary.

8. Type-check a test file that uses fixtures or fakes.

9. Add a stricter checker rule to one module and fix the resulting errors.

10. Compare a bug caught by tests with a bug caught by static type checking.

---

# Preview of Chapter 79

Chapter 78 studied static type checking.

We learned how type checkers use annotations to catch mismatches before runtime, how to configure and adopt checking gradually, how to handle `Any`, stubs, ignores, casts, dynamic code, CI, tests, and public API typing.

Next we study profiling.

Testing tells us whether behavior is correct.

Static type checking tells us whether shapes are consistent.

Profiling tells us where time and resources are actually spent.

Performance work without profiling is often guesswork.

Profiling helps answer:

* which function is slow?
* how many times is it called?
* where is memory allocated?
* which path dominates runtime?
* is the database slow or the Python loop?
* is the bottleneck CPU, I/O, memory, or waiting?

The transition is:

```text
static type checking prevents shape mistakes
profiling prevents performance guesswork
```

Chapter 79 will show how to measure before optimizing and how to interpret the measurements without chasing the wrong bottleneck.
