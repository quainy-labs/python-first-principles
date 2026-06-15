# Chapter 73 — Mocking and Monkey Patching

Testing real software means dealing with boundaries.

A small pure function is easy to test.

You pass values in.

You check values out.

But production programs rarely stay pure.

They talk to databases.

They read files.

They call APIs.

They inspect environment variables.

They depend on clocks.

They generate random values.

They send emails.

They publish messages to queues.

They use caches.

They start subprocesses.

They call operating-system services.

They use third-party SDKs.

If every test touched all of those real dependencies, the suite would become slow, fragile, expensive, and difficult to run locally.

Mocking and monkey patching give tests a way to control those boundaries.

They let a test say:

```text
for this test, replace the real dependency with this controlled behavior
```

That is powerful.

It is also dangerous.

Used well, mocking makes tests focused and deterministic.

Used carelessly, mocking makes tests brittle, dishonest, and tightly coupled to implementation details.

This chapter is about both sides.

You will learn the tools.

More importantly, you will learn the judgment.

---

# The Problem Mocking Solves

Imagine this function:

```python
def send_welcome_email(user):
    smtp = SmtpClient(host="smtp.example.com")
    smtp.send(
        to=user.email,
        subject="Welcome",
        body=f"Hello {user.name}",
    )
```

Testing it directly has problems.

If the test calls the real SMTP server, the test may:

* require network access
* send real email accidentally
* fail when the SMTP service is down
* be slow
* need credentials
* depend on external configuration
* behave differently in local and CI environments

The behavior we care about may be simple:

```text
when a user signs up, send a welcome email to that user's address
```

We do not need a real SMTP server to test that rule.

We need a controlled substitute.

That substitute can record the attempted email.

Then the test can assert:

```text
the code tried to send the right email
```

Mocking exists for this kind of situation.

It helps tests control dependencies that are outside the immediate behavior being checked.

---

# Test Doubles

A test double is any object used in a test in place of a real dependency.

The phrase comes from stunt doubles in film.

The real actor is not used for every shot.

In tests, the real dependency is not used for every behavior check.

Common test doubles include:

* dummy
* fake
* stub
* spy
* mock

These terms are often used loosely.

But the distinctions help you think clearly.

A dummy is passed around but not used.

A fake is a working simplified implementation.

A stub returns predefined answers.

A spy records how it was used.

A mock is usually a programmable object that can return values, raise exceptions, and later verify calls.

Python's `unittest.mock` library provides mock objects.

pytest's `monkeypatch` fixture helps temporarily change attributes, dictionary values, environment variables, and import paths.

Both are tools for controlling test boundaries.

---

# Dummy Objects

A dummy object exists only because something requires an argument.

The test does not care about it.

For example:

```python
def format_order(order, logger):
    logger.debug("formatting order")
    return f"Order #{order.id}"
```

If the test only cares about formatting and does not care about logging, a dummy logger might be enough:

```python
class DummyLogger:
    def debug(self, message):
        pass


def test_format_order():
    order = Order(id=123)

    result = format_order(order, DummyLogger())

    assert result == "Order #123"
```

The dummy is not part of the assertion.

It only satisfies the function's parameter list.

If the dependency starts to matter, it may no longer be a dummy.

---

# Fakes

A fake is a simplified working implementation.

For example, instead of a real email sender:

```python
class FakeEmailSender:
    def __init__(self):
        self.messages = []

    def send(self, to, subject, body):
        self.messages.append({
            "to": to,
            "subject": subject,
            "body": body,
        })
```

A service can use it:

```python
class SignupService:
    def __init__(self, email_sender):
        self.email_sender = email_sender

    def signup(self, user):
        self.email_sender.send(
            to=user.email,
            subject="Welcome",
            body=f"Hello {user.name}",
        )
```

Test:

```python
def test_signup_sends_welcome_email():
    sender = FakeEmailSender()
    service = SignupService(sender)
    user = User(email="asha@example.com", name="Asha")

    service.signup(user)

    assert sender.messages == [
        {
            "to": "asha@example.com",
            "subject": "Welcome",
            "body": "Hello Asha",
        }
    ]
```

Fakes are often clearer than mocks.

They behave like small real objects.

They can be reused across tests.

They can express domain meaning.

They are especially useful for:

* email senders
* payment gateways in test mode
* in-memory repositories
* fake clocks
* fake queues
* fake file stores
* fake notification systems

The risk is that a fake may drift away from the real implementation.

If your fake database behaves differently from the real database, tests may pass while production fails.

Fakes work best when their behavior is small, obvious, and aligned with the contract you need.

---

# Stubs

A stub provides a predefined response.

For example:

```python
class StubExchangeRateProvider:
    def get_rate(self, from_currency, to_currency):
        return 82
```

Test:

```python
def test_convert_usd_to_inr():
    converter = CurrencyConverter(StubExchangeRateProvider())

    result = converter.convert(10, "USD", "INR")

    assert result == 820
```

The stub does not record calls.

It only answers.

Stubs are useful when the code under test needs information from a dependency.

The test controls the answer so the behavior becomes deterministic.

This is better than calling a real exchange-rate API in a unit test.

---

# Spies

A spy records what happened.

The fake email sender from earlier is also a spy because it records messages.

Another example:

```python
class SpyLogger:
    def __init__(self):
        self.messages = []

    def info(self, message):
        self.messages.append(message)
```

Test:

```python
def test_import_logs_success_message():
    logger = SpyLogger()

    import_file("users.csv", logger)

    assert logger.messages == ["Imported users.csv"]
```

Spies are useful when the behavior is an interaction.

For example:

* an email should be sent
* a metric should be emitted
* an event should be published
* a cache should be invalidated
* a log should be written

But interaction testing can be overdone.

Not every internal function call deserves an assertion.

The more your tests assert about internal collaboration, the harder refactoring becomes.

---

# Mocks

A mock is a flexible object that can imitate another object during a test.

Python's standard-library `unittest.mock` module provides `Mock`, `MagicMock`, `patch`, `AsyncMock`, and related helpers.

A basic mock can be called:

```python
from unittest.mock import Mock


sender = Mock()
sender.send("a@example.com", "Hello", "Body")

sender.send.assert_called_once_with("a@example.com", "Hello", "Body")
```

The mock records the call.

The test can assert how it was used.

Mocks can also return values:

```python
from unittest.mock import Mock


provider = Mock()
provider.get_rate.return_value = 82

assert provider.get_rate("USD", "INR") == 82
```

Mocks can raise exceptions:

```python
from unittest.mock import Mock


provider = Mock()
provider.get_rate.side_effect = TimeoutError("rate service timed out")
```

Mocks can be very convenient.

That convenience is exactly why they require discipline.

---

# The Central Question

Before using a mock, ask:

```text
what behavior am I trying to protect?
```

If the behavior is:

```text
the user receives a welcome email
```

then a fake email sender may be best.

If the behavior is:

```text
the retry logic calls the dependency three times after two failures
```

then a mock or spy may be appropriate.

If the behavior is:

```text
the function returns the correct invoice total
```

then mocking internal helper functions is probably wrong.

The test should call the public behavior and check the result.

Mocks are best at boundaries.

They are risky inside the core.

---

# Behavior Testing vs Interaction Testing

Behavior testing checks outcomes.

Interaction testing checks calls.

Behavior test:

```python
def test_cart_total_includes_quantities():
    cart = Cart()
    cart.add(Product("Pen", price=10), quantity=3)

    assert cart.total() == 30
```

Interaction test:

```python
def test_signup_sends_email():
    sender = Mock()
    service = SignupService(sender)

    service.signup(User(email="a@example.com", name="Asha"))

    sender.send.assert_called_once_with(
        to="a@example.com",
        subject="Welcome",
        body="Hello Asha",
    )
```

Both can be valid.

But they protect different things.

The cart test protects a returned result.

The signup test protects an interaction with an external boundary.

Problems begin when interaction tests are used for ordinary internal logic.

For example:

```python
def test_cart_total_calls_private_sum_method():
    cart = Cart()
    cart._sum_items = Mock(return_value=30)

    assert cart.total() == 30
    cart._sum_items.assert_called_once()
```

This test knows too much.

If the implementation changes but behavior remains correct, the test may fail.

Good tests make refactoring safer.

Over-mocked tests make refactoring painful.

---

# unittest.mock

`unittest.mock` is Python's standard-library mock library.

Its central tools are:

* `Mock`
* `MagicMock`
* `AsyncMock`
* `patch`
* `patch.object`
* `patch.dict`
* `mock_open`
* `call`
* `ANY`
* `create_autospec`

You can use `unittest.mock` with `unittest`, pytest, or another test runner.

The module name says `unittest`, but the tools are not limited to `unittest.TestCase`.

For example, this is a pytest-style test using `unittest.mock`:

```python
from unittest.mock import Mock


def test_signup_sends_email():
    sender = Mock()
    service = SignupService(sender)

    service.signup(User(email="a@example.com", name="Asha"))

    sender.send.assert_called_once()
```

That is perfectly normal.

---

# Mock

`Mock` creates attributes dynamically.

If you access an attribute, you get another mock:

```python
from unittest.mock import Mock


client = Mock()

client.users.create("asha@example.com")

client.users.create.assert_called_once_with("asha@example.com")
```

This is convenient because a mock can imitate nested objects quickly.

It is also dangerous because typos can silently create new attributes.

For example:

```python
client.uesrs.create("asha@example.com")
```

The typo `uesrs` may not fail immediately on a plain mock.

The mock happily creates a new attribute.

This can make tests pass when production code would fail.

That is why specs and autospec matter.

---

# return_value

`return_value` controls what a mock returns when called.

Example:

```python
from unittest.mock import Mock


fetch_user = Mock()
fetch_user.return_value = {"id": "u-1", "name": "Asha"}

assert fetch_user("u-1") == {"id": "u-1", "name": "Asha"}
fetch_user.assert_called_once_with("u-1")
```

For methods:

```python
client = Mock()
client.fetch_user.return_value = {"id": "u-1"}

assert client.fetch_user("u-1") == {"id": "u-1"}
```

Use `return_value` when the dependency simply needs to answer.

If the answer depends on the input, use `side_effect`.

---

# side_effect

`side_effect` lets a mock do something when called.

It can raise an exception:

```python
from unittest.mock import Mock


fetch_user = Mock(side_effect=TimeoutError("service unavailable"))
```

It can be a function:

```python
from unittest.mock import Mock


def fake_fetch(user_id):
    users = {
        "u-1": {"id": "u-1", "name": "Asha"},
        "u-2": {"id": "u-2", "name": "Ravi"},
    }
    return users[user_id]


fetch_user = Mock(side_effect=fake_fetch)

assert fetch_user("u-1")["name"] == "Asha"
```

It can be a sequence:

```python
from unittest.mock import Mock


service = Mock(side_effect=[TimeoutError, TimeoutError, "ok"])
```

Each call consumes the next item.

This is useful for retry tests:

```python
def test_retry_succeeds_on_third_attempt():
    dependency = Mock(side_effect=[TimeoutError, TimeoutError, "ok"])

    result = retry(dependency, attempts=3)

    assert result == "ok"
    assert dependency.call_count == 3
```

`side_effect` is powerful.

If it becomes complex, consider whether a fake object would be clearer.

---

# Call Assertions

Mocks record calls.

Common assertions include:

```python
mock.assert_called()
mock.assert_called_once()
mock.assert_called_with(...)
mock.assert_called_once_with(...)
mock.assert_any_call(...)
mock.assert_has_calls(...)
mock.assert_not_called()
```

Example:

```python
from unittest.mock import Mock


publisher = Mock()

publisher.publish("user.created", {"id": "u-1"})

publisher.publish.assert_called_once_with("user.created", {"id": "u-1"})
```

If multiple calls are expected:

```python
from unittest.mock import Mock, call


publisher = Mock()

publisher.publish("user.created", {"id": "u-1"})
publisher.publish("email.queued", {"user_id": "u-1"})

publisher.publish.assert_has_calls([
    call("user.created", {"id": "u-1"}),
    call("email.queued", {"user_id": "u-1"}),
])
```

Call assertions are useful when interaction is the behavior.

They are noisy when interaction is merely implementation.

---

# call_count

`call_count` records how many times a mock was called.

Example:

```python
def test_cache_prevents_second_fetch():
    source = Mock()
    source.fetch.return_value = {"id": "u-1"}
    cache = UserCache(source)

    cache.get("u-1")
    cache.get("u-1")

    assert source.fetch.call_count == 1
```

This test checks a meaningful performance and behavior property:

```text
the cache should avoid repeated source calls
```

Call count tests can become brittle if they check incidental details.

For example:

```python
assert helper.normalize.call_count == 1
```

If nobody outside the implementation cares how many times `normalize` is called, this assertion may be unnecessary.

Ask whether the count is part of the contract.

---

# mock_calls

`mock_calls` records calls across a mock and its children.

Example:

```python
from unittest.mock import Mock, call


client = Mock()

client.connect()
client.users.create(email="a@example.com")
client.close()

assert client.mock_calls == [
    call.connect(),
    call.users.create(email="a@example.com"),
    call.close(),
]
```

This can be useful for protocols where order matters.

For example:

```text
open transaction
write rows
commit transaction
```

But be careful.

Testing long call sequences often locks tests to implementation.

If order is a public or safety-critical contract, assert it.

If order is incidental, prefer behavior assertions.

---

# MagicMock

`MagicMock` supports many Python magic methods by default.

Magic methods include:

* `__len__`
* `__iter__`
* `__enter__`
* `__exit__`
* `__getitem__`
* `__setitem__`
* `__str__`
* `__contains__`

Example:

```python
from unittest.mock import MagicMock


items = MagicMock()
items.__len__.return_value = 3

assert len(items) == 3
```

For a context manager:

```python
from unittest.mock import MagicMock


resource = MagicMock()
resource.__enter__.return_value = "connection"

with resource as connection:
    assert connection == "connection"

resource.__enter__.assert_called_once()
resource.__exit__.assert_called_once()
```

Use `MagicMock` when the object must behave with Python protocols.

Use ordinary `Mock` when normal attribute and method calls are enough.

---

# AsyncMock

`AsyncMock` is used for async functions.

If production code awaits a dependency, the test double must be awaitable.

Example:

```python
from unittest.mock import AsyncMock


async def test_fetch_user_name():
    client = AsyncMock()
    client.fetch_user.return_value = {"name": "Asha"}

    result = await get_user_name(client, "u-1")

    assert result == "Asha"
    client.fetch_user.assert_awaited_once_with("u-1")
```

Async mocks have await-specific assertions, such as:

```python
mock.assert_awaited()
mock.assert_awaited_once()
mock.assert_awaited_with(...)
mock.assert_awaited_once_with(...)
```

Do not use a plain `Mock` where production code expects an awaitable function.

The test may fail in confusing ways, or worse, avoid exercising the real async path.

Async behavior deserves async-aware tests.

---

# spec

Plain mocks accept almost any attribute.

That flexibility can hide mistakes.

`spec` restricts a mock to the attributes of a real object.

Example:

```python
from unittest.mock import Mock


class EmailSender:
    def send(self, to, subject, body):
        ...


sender = Mock(spec=EmailSender)
sender.send("a@example.com", "Hello", "Body")
```

If you access an attribute not on `EmailSender`, the mock raises `AttributeError`:

```python
sender.deliver("a@example.com")
```

That is useful because production code would also fail if `deliver` does not exist.

Specs make mocks more honest.

They reduce false confidence.

---

# spec_set

`spec_set` is stricter than `spec`.

It prevents getting or setting attributes that do not exist on the specified object.

Example:

```python
from unittest.mock import Mock


sender = Mock(spec_set=EmailSender)
```

With `spec`, setting a new attribute may still be allowed in some cases.

With `spec_set`, the mock is more constrained.

This can catch typos in test setup:

```python
sender.sned.return_value = None
```

The typo should fail.

In larger test suites, stricter mocks often save time.

They make tests fail closer to the mistake.

---

# autospec

Autospeccing makes mocks follow the interface and call signature of the object they replace.

This catches wrong argument usage.

Example:

```python
from unittest.mock import create_autospec


def send_email(to, subject, body):
    ...


mock_send_email = create_autospec(send_email)

mock_send_email("a@example.com", "Hello", "Body")
```

Calling it incorrectly fails:

```python
mock_send_email("a@example.com")
```

That is good.

The real function requires three arguments.

The mock should not accept one argument silently.

Autospec is especially useful with `patch`:

```python
from unittest.mock import patch


@patch("notifications.send_email", autospec=True)
def test_signup_sends_email(mock_send_email):
    signup("a@example.com")

    mock_send_email.assert_called_once()
```

When using mocks for real interfaces, prefer specs or autospec when practical.

Unrestricted mocks are convenient but easy to lie with.

---

# patch

`patch` temporarily replaces an attribute during a test.

The replacement is undone when the test ends.

Example:

```python
from unittest.mock import patch


def test_signup_sends_email():
    with patch("accounts.send_email") as mock_send_email:
        signup("a@example.com")

    mock_send_email.assert_called_once_with("a@example.com")
```

The string `"accounts.send_email"` names the attribute to replace.

During the `with` block, code that looks up `accounts.send_email` sees the mock.

After the block, the original function is restored.

`patch` can also be used as a decorator:

```python
from unittest.mock import patch


@patch("accounts.send_email")
def test_signup_sends_email(mock_send_email):
    signup("a@example.com")

    mock_send_email.assert_called_once_with("a@example.com")
```

Context managers are often easier to read when there are only one or two patches.

Decorators can be compact but become confusing when stacked.

---

# Where to Patch

The most important patching rule is:

```text
patch where the object is looked up
```

This is not always where the object was originally defined.

Suppose `email_service.py` defines:

```python
def send_email(to, subject, body):
    ...
```

And `accounts.py` imports it like this:

```python
from email_service import send_email


def signup(email):
    send_email(email, "Welcome", "Hello")
```

Inside `accounts.signup`, the name being used is:

```text
accounts.send_email
```

So the test should patch:

```python
with patch("accounts.send_email") as mock_send_email:
    signup("a@example.com")
```

Patching this may not work:

```python
with patch("email_service.send_email") as mock_send_email:
    signup("a@example.com")
```

Why?

Because `accounts.py` already imported the function into its own namespace.

The `signup` function looks up `send_email` in `accounts`, not in `email_service`.

This rule causes many mocking bugs.

Remember it:

```text
patch the name used by the system under test
```

---

# Import Style and Patching

Import style affects patching.

Case one:

```python
import email_service


def signup(email):
    email_service.send_email(email, "Welcome", "Hello")
```

Patch:

```python
with patch("accounts.email_service.send_email"):
    signup("a@example.com")
```

Case two:

```python
from email_service import send_email


def signup(email):
    send_email(email, "Welcome", "Hello")
```

Patch:

```python
with patch("accounts.send_email"):
    signup("a@example.com")
```

The function definition looks similar.

The lookup path is different.

Patching is about lookup paths.

If a patch seems ignored, inspect where the production code looks up the name.

---

# patch.object

`patch.object` patches an attribute on an object directly.

Example:

```python
from unittest.mock import patch


def test_processor_uses_clock():
    clock = Clock()

    with patch.object(clock, "now", return_value="2026-01-01"):
        assert clock.now() == "2026-01-01"
```

It is useful when you already have the object.

For classes:

```python
with patch.object(UserService, "send_welcome_email") as mock_send:
    service = UserService()
    service.signup("a@example.com")

mock_send.assert_called_once()
```

Be careful patching methods on classes.

You may be replacing behavior for all instances during the patch scope.

That is sometimes exactly what you want.

It can also make tests interfere if scope is too broad.

Keep patch scopes narrow.

---

# patch.dict

`patch.dict` temporarily changes dictionary contents.

It is often used with `os.environ`.

Example:

```python
import os
from unittest.mock import patch


def get_api_url():
    return os.environ["API_URL"]


def test_get_api_url():
    with patch.dict(os.environ, {"API_URL": "https://example.test"}):
        assert get_api_url() == "https://example.test"
```

After the `with` block, the original environment is restored.

`patch.dict` can also clear the dictionary temporarily:

```python
with patch.dict(os.environ, {}, clear=True):
    ...
```

That is useful when testing behavior with a controlled environment.

Environment variables are global process state.

Do not change them without cleanup.

---

# mock_open

`mock_open` helps mock file reading and writing through `open`.

Example:

```python
from unittest.mock import mock_open, patch


def read_config(path):
    with open(path) as file:
        return file.read()


def test_read_config():
    mocked_open = mock_open(read_data="debug=true")

    with patch("builtins.open", mocked_open):
        assert read_config("settings.ini") == "debug=true"
```

This can be useful for simple file interactions.

But do not mock files by default.

Often `tmp_path` is clearer:

```python
def test_read_config(tmp_path):
    path = tmp_path / "settings.ini"
    path.write_text("debug=true")

    assert read_config(path) == "debug=true"
```

Use a real temporary file when you want to test filesystem behavior.

Use `mock_open` when you want to isolate the code from the filesystem and only verify that file access is attempted in a certain way.

---

# ANY

`ANY` matches any value in a mock assertion.

Example:

```python
from unittest.mock import ANY, Mock


logger = Mock()

logger.info("created user", extra={"request_id": "abc-123"})

logger.info.assert_called_once_with("created user", extra={"request_id": ANY})
```

This is useful when part of the call is not important or is nondeterministic.

Use it sparingly.

If every argument is `ANY`, the assertion may not be meaningful:

```python
mock.send.assert_called_once_with(ANY, ANY, ANY)
```

That only proves a call happened with three arguments.

Sometimes that is enough.

Usually, at least some arguments should be specific.

---

# wraps

`wraps` lets a mock call the real object while still recording usage.

Example:

```python
from unittest.mock import Mock


def normalize(text):
    return text.strip().lower()


spy_normalize = Mock(wraps=normalize)

assert spy_normalize("  PyThOn  ") == "python"
spy_normalize.assert_called_once_with("  PyThOn  ")
```

This creates a spy around real behavior.

It can be useful when you want to verify a call while preserving the original implementation.

But if you need `wraps` often, pause.

You may be testing internals too closely.

---

# reset_mock

`reset_mock` clears call history.

Example:

```python
from unittest.mock import Mock


sender = Mock()

sender.send("a@example.com")
sender.reset_mock()
sender.send("b@example.com")

sender.send.assert_called_once_with("b@example.com")
```

This can be useful when one test has multiple phases.

But many tests are clearer if each phase has its own mock.

If you find yourself resetting mocks repeatedly, the test may be too large.

Consider splitting it.

---

# Monkey Patching

Monkey patching means changing code at runtime.

In tests, it usually means temporarily replacing:

* a function
* a class
* an object attribute
* a module attribute
* an environment variable
* a dictionary entry
* the current working directory
* the import path

Python makes monkey patching possible because names and attributes are dynamic.

For example:

```python
import module


module.some_function = replacement_function
```

That changes what `module.some_function` refers to.

This is powerful because tests can replace dependencies without changing production code.

It is risky because runtime mutation can be confusing if not restored.

Test frameworks provide helpers so temporary changes are undone safely.

---

# pytest monkeypatch

pytest provides a built-in `monkeypatch` fixture.

It can temporarily:

* set attributes
* delete attributes
* set dictionary items
* delete dictionary items
* set environment variables
* delete environment variables
* change the current working directory
* modify `sys.path`

Example:

```python
def test_get_api_url(monkeypatch):
    monkeypatch.setenv("API_URL", "https://example.test")

    assert get_api_url() == "https://example.test"
```

After the test finishes, pytest undoes the change.

That cleanup is the main reason to use the fixture instead of manually mutating global state.

---

# monkeypatch.setattr

`monkeypatch.setattr` temporarily changes an attribute.

Example:

```python
def test_signup_sends_email(monkeypatch):
    sent = []

    def fake_send_email(to, subject, body):
        sent.append((to, subject, body))

    monkeypatch.setattr("accounts.send_email", fake_send_email)

    signup("a@example.com")

    assert sent == [("a@example.com", "Welcome", "Hello")]
```

Like `patch`, you must patch where the object is looked up.

If `accounts.signup` uses `accounts.send_email`, patch `accounts.send_email`.

`monkeypatch.setattr` can also patch object attributes:

```python
monkeypatch.setattr(clock, "now", lambda: fixed_time)
```

By default, `monkeypatch.setattr` expects the attribute to already exist.

That helps catch typos.

---

# monkeypatch.setenv

`monkeypatch.setenv` sets an environment variable for the duration of the test.

Example:

```python
import os


def get_mode():
    return os.environ.get("APP_MODE", "development")


def test_get_mode_from_environment(monkeypatch):
    monkeypatch.setenv("APP_MODE", "production")

    assert get_mode() == "production"
```

The original environment is restored after the test.

This is safer than:

```python
os.environ["APP_MODE"] = "production"
```

because manual mutation can leak into later tests.

Use `monkeypatch.delenv` to remove a variable temporarily:

```python
def test_get_mode_defaults_when_missing(monkeypatch):
    monkeypatch.delenv("APP_MODE", raising=False)

    assert get_mode() == "development"
```

---

# monkeypatch.setitem

`monkeypatch.setitem` temporarily changes a dictionary entry.

Example:

```python
SETTINGS = {
    "timeout": 5,
}


def get_timeout():
    return SETTINGS["timeout"]


def test_get_timeout(monkeypatch):
    monkeypatch.setitem(SETTINGS, "timeout", 30)

    assert get_timeout() == 30
```

After the test, `SETTINGS` is restored.

This is useful for module-level configuration dictionaries.

But if your application relies heavily on mutable global settings, tests may become complicated.

Consider passing configuration explicitly where possible.

---

# monkeypatch.chdir

`monkeypatch.chdir` temporarily changes the current working directory.

Example:

```python
from pathlib import Path


def read_local_config():
    return Path("config.txt").read_text()


def test_read_local_config(monkeypatch, tmp_path):
    (tmp_path / "config.txt").write_text("debug=true")
    monkeypatch.chdir(tmp_path)

    assert read_local_config() == "debug=true"
```

The current working directory is process-global.

Changing it manually can break unrelated tests.

`monkeypatch.chdir` ensures cleanup.

Still, prefer passing explicit paths when possible.

Current-directory-dependent code is often harder to reason about than path-explicit code.

---

# monkeypatch.syspath_prepend

`monkeypatch.syspath_prepend` temporarily adds a path to the beginning of `sys.path`.

This can be useful for import-related tests.

Example:

```python
def test_plugin_import(monkeypatch, tmp_path):
    plugin_dir = tmp_path / "plugins"
    plugin_dir.mkdir()
    (plugin_dir / "sample_plugin.py").write_text("VALUE = 42")

    monkeypatch.syspath_prepend(str(plugin_dir))

    import sample_plugin

    assert sample_plugin.VALUE == 42
```

Import tests can be subtle because Python caches imported modules in `sys.modules`.

If you test imports heavily, you may need to clean up imported module entries too.

Module state is global.

Treat it carefully.

---

# patch vs monkeypatch

`unittest.mock.patch` and pytest's `monkeypatch` overlap.

Both can temporarily replace things.

Use `patch` when you want:

* a mock object created automatically
* call assertions
* `autospec`
* decorators or context managers from the standard library
* compatibility with `unittest`

Use `monkeypatch` when you want:

* simple attribute replacement
* environment variable changes
* dictionary changes
* current-directory changes
* import-path changes
* automatic pytest fixture cleanup

Example with `patch`:

```python
from unittest.mock import patch


def test_signup_sends_email():
    with patch("accounts.send_email") as mock_send:
        signup("a@example.com")

    mock_send.assert_called_once()
```

Example with `monkeypatch`:

```python
def test_signup_sends_email(monkeypatch):
    sent = []

    def fake_send(to, subject, body):
        sent.append((to, subject, body))

    monkeypatch.setattr("accounts.send_email", fake_send)

    signup("a@example.com")

    assert sent
```

Neither is universally better.

Choose the tool that makes the test clearer.

---

# Mocking Time

Time-dependent code is a common source of difficult tests.

Consider:

```python
from datetime import datetime


def greeting():
    hour = datetime.now().hour
    if hour < 12:
        return "Good morning"
    return "Good afternoon"
```

One way to test this is patching `datetime`.

But patching `datetime.datetime` can be awkward because it is a C-implemented type and because import paths matter.

A better design is often dependency injection:

```python
def greeting(now):
    if now.hour < 12:
        return "Good morning"
    return "Good afternoon"
```

Test:

```python
from datetime import datetime


def test_greeting_morning():
    assert greeting(datetime(2026, 1, 1, 9, 0)) == "Good morning"
```

No patch needed.

This is the deeper lesson:

```text
if mocking is painful, the design may be hiding a dependency
```

Patching time can be useful.

Passing time explicitly is often better.

---

# Mocking Randomness

Randomness should be controlled in tests.

Suppose:

```python
import random


def make_invite_code():
    return str(random.randint(1000, 9999))
```

You can patch:

```python
from unittest.mock import patch


def test_make_invite_code():
    with patch("invites.random.randint", return_value=1234):
        assert make_invite_code() == "1234"
```

Again, patch where the name is looked up.

If the module under test is `invites.py`, and it imported `random`, patch `invites.random.randint`.

Another design is to inject the generator:

```python
def make_invite_code(randint):
    return str(randint(1000, 9999))
```

Test:

```python
def test_make_invite_code():
    assert make_invite_code(lambda start, end: 1234) == "1234"
```

For simple code, patching may be acceptable.

For domain logic, explicit dependencies often age better.

---

# Mocking Network Calls

Unit tests should usually avoid real network calls.

Network calls are:

* slow
* unreliable
* environment-dependent
* rate-limited
* sometimes expensive
* sometimes dangerous

Suppose:

```python
import requests


def fetch_user(user_id):
    response = requests.get(f"https://api.example.com/users/{user_id}")
    response.raise_for_status()
    return response.json()
```

One test can patch `requests.get`:

```python
from unittest.mock import Mock, patch


def test_fetch_user():
    response = Mock()
    response.json.return_value = {"id": "u-1", "name": "Asha"}
    response.raise_for_status.return_value = None

    with patch("users.requests.get", return_value=response):
        assert fetch_user("u-1") == {"id": "u-1", "name": "Asha"}
```

This works.

But it also knows that `fetch_user` uses `requests.get`.

If you switch to another HTTP client, the test breaks even if behavior remains correct.

An alternative is to wrap the HTTP client behind your own interface:

```python
class UserApi:
    def __init__(self, http_client):
        self.http_client = http_client

    def fetch_user(self, user_id):
        response = self.http_client.get(f"/users/{user_id}")
        return response.json()
```

Now tests can fake `http_client`.

For API-heavy systems, wrapper boundaries often make tests cleaner.

---

# Mocking Databases

Mocking databases is tricky.

A database is not just a function call.

It has behavior:

* constraints
* transactions
* indexes
* isolation
* query semantics
* type conversions
* locking
* migrations
* connection errors

A mock database may miss the very behavior you need to test.

For pure business logic, avoid the database entirely by passing plain data.

For repository code, prefer integration tests with a real test database when the SQL behavior matters.

Mocking a database cursor may be useful for narrow error paths.

Example:

```python
def test_repository_rolls_back_on_error():
    session = Mock()
    session.commit.side_effect = DatabaseError

    repository = UserRepository(session)

    with pytest.raises(DatabaseError):
        repository.save(User(email="a@example.com"))

    session.rollback.assert_called_once()
```

This test checks transaction error handling.

It does not prove that actual SQL works.

Use the right tool for the risk.

---

# Mocking Filesystems

Do not mock the filesystem automatically.

Temporary directories are often better.

For example:

```python
def test_file_is_written(tmp_path):
    path = tmp_path / "report.txt"

    write_report(path, "hello")

    assert path.read_text() == "hello"
```

This is clearer than mocking `open`.

It tests real file behavior in an isolated location.

Mock `open` when:

* file content is simple
* you want to avoid disk entirely
* you need to simulate file errors
* you need to assert `open` was called with specific arguments

Example error path:

```python
from unittest.mock import patch


def test_read_config_handles_missing_file():
    with patch("builtins.open", side_effect=FileNotFoundError):
        assert read_config("missing.ini") == {}
```

Filesystem behavior is often cheap to test for real.

Use that when it improves confidence.

---

# Mocking Environment Variables

Environment variables should be patched or monkeypatched, never casually mutated.

Good:

```python
def test_debug_enabled(monkeypatch):
    monkeypatch.setenv("DEBUG", "1")

    assert debug_enabled() is True
```

Good:

```python
from unittest.mock import patch


def test_debug_enabled():
    with patch.dict(os.environ, {"DEBUG": "1"}):
        assert debug_enabled() is True
```

Risky:

```python
os.environ["DEBUG"] = "1"
```

The risky version can leak into later tests.

Environment variables are global.

Global state must be restored.

---

# Mocking Logging

Sometimes logs are behavior.

For example:

* audit logs
* security logs
* operational alerts
* structured event logs

In those cases, test them.

For ordinary debug logs, avoid over-testing.

pytest provides `caplog` for logging assertions:

```python
def test_import_logs_success(caplog):
    import_users("users.csv")

    assert "Imported users.csv" in caplog.text
```

You can also pass a fake logger:

```python
def test_import_logs_success():
    logger = SpyLogger()

    import_users("users.csv", logger=logger)

    assert logger.messages == ["Imported users.csv"]
```

Do not mock logging just because it exists.

Test logs when they are part of an observable contract.

---

# Mocking Context Managers

Many resources are context managers.

Examples:

* files
* locks
* database transactions
* network sessions
* temporary resources

If you mock a context manager, configure `__enter__` and `__exit__`.

Example:

```python
from unittest.mock import MagicMock


transaction = MagicMock()
transaction.__enter__.return_value = "session"

with transaction as session:
    assert session == "session"

transaction.__enter__.assert_called_once()
transaction.__exit__.assert_called_once()
```

If this feels awkward, consider whether a real lightweight context manager would be clearer in the test.

Mocks can imitate protocols.

But a small fake class often communicates intent better.

---

# Mocking Properties

Properties are descriptors.

Mocking them requires care.

`PropertyMock` can be used with `patch.object` on the type:

```python
from unittest.mock import PropertyMock, patch


class User:
    @property
    def display_name(self):
        return "real"


def test_display_name_property():
    with patch.object(User, "display_name", new_callable=PropertyMock) as mock_name:
        mock_name.return_value = "fake"

        assert User().display_name == "fake"
```

Patch the class, not the instance, because properties are resolved through descriptor behavior on the type.

This connects back to the descriptor chapter in Volume II.

Understanding Python's object model makes advanced testing less mysterious.

---

# Mocking Class Construction

Sometimes code constructs a dependency internally:

```python
class ReportService:
    def generate(self):
        client = StorageClient()
        client.upload("report.txt")
```

You can patch the class:

```python
from unittest.mock import patch


def test_generate_uploads_report():
    with patch("reports.StorageClient") as MockStorageClient:
        client = MockStorageClient.return_value

        ReportService().generate()

        client.upload.assert_called_once_with("report.txt")
```

When a class is patched, calling it returns its `return_value`.

So:

```python
client = MockStorageClient.return_value
```

is the fake instance created by `StorageClient()`.

This is common, but it can be a design smell.

If construction is important and repeated, consider injecting the dependency:

```python
class ReportService:
    def __init__(self, storage_client):
        self.storage_client = storage_client

    def generate(self):
        self.storage_client.upload("report.txt")
```

Now tests can pass a fake or mock directly.

---

# Mocking Builtins

You can patch builtins such as `open`, `input`, or `print`.

Example with `input`:

```python
from unittest.mock import patch


def ask_name():
    return input("Name: ")


def test_ask_name():
    with patch("builtins.input", return_value="Asha"):
        assert ask_name() == "Asha"
```

Example with `print`:

```python
from unittest.mock import patch


def greet(name):
    print(f"Hello {name}")


def test_greet():
    with patch("builtins.print") as mock_print:
        greet("Asha")

    mock_print.assert_called_once_with("Hello Asha")
```

For CLI output, pytest's `capsys` is often better:

```python
def test_greet(capsys):
    greet("Asha")

    captured = capsys.readouterr()
    assert captured.out == "Hello Asha\n"
```

Again, choose the clearer tool.

---

# Monkey Patching Imports

Sometimes code imports optional dependencies.

Tests may need to simulate missing modules.

This can be done by manipulating `sys.modules`, but it is easy to get wrong.

Example:

```python
def test_optional_dependency_missing(monkeypatch):
    monkeypatch.setitem(sys.modules, "optional_lib", None)

    assert feature_available() is False
```

Import state is global.

Python caches imported modules.

If you are testing import behavior, keep the tests isolated and explicit.

Often it is better to move optional dependency checks behind a small function and test that function carefully.

---

# Mocking Third-Party SDKs

Third-party SDKs are common mock targets.

Examples:

* payment clients
* cloud storage clients
* email services
* analytics clients
* search clients
* AI APIs
* CRM systems

The danger is mocking the SDK too closely.

If your tests know every SDK call, they may break when you upgrade the SDK even if your application behavior is unchanged.

Create your own boundary when the dependency is important:

```python
class PaymentGateway:
    def charge(self, customer_id, amount):
        ...
```

Production implementation:

```python
class StripePaymentGateway:
    def __init__(self, stripe_client):
        self.stripe_client = stripe_client

    def charge(self, customer_id, amount):
        return self.stripe_client.PaymentIntent.create(
            customer=customer_id,
            amount=amount,
            currency="inr",
        )
```

Application code depends on `PaymentGateway`.

Tests fake `PaymentGateway`.

Only the adapter tests need to know the SDK details.

This keeps most tests stable when the SDK changes.

---

# Contract Tests

If you use fakes for external boundaries, you need confidence that the fake matches the real contract.

Contract tests help.

A contract test checks that an implementation satisfies a shared behavior.

For example, both real and fake storage clients should support:

* upload
* download
* missing file behavior
* overwrite behavior
* error behavior

You can write shared tests:

```python
def storage_contract(storage):
    storage.upload("hello.txt", b"hello")

    assert storage.download("hello.txt") == b"hello"
```

Then run the same behavior against the fake and, where practical, the real implementation in integration tests.

This prevents fakes from becoming fantasy worlds.

Fakes are useful only when they preserve the contract that matters.

---

# Over-Mocking

Over-mocking happens when tests replace too much.

Symptoms include:

* tests assert many internal calls
* tests patch private helpers
* tests fail during harmless refactors
* tests pass even when real behavior is broken
* tests duplicate implementation structure
* tests require many mocks before anything can run
* tests are difficult to read
* changing one function breaks dozens of unrelated tests

Example:

```python
@patch("orders.validate_cart")
@patch("orders.calculate_tax")
@patch("orders.calculate_shipping")
@patch("orders.create_invoice")
@patch("orders.send_confirmation")
def test_checkout(mock_send, mock_invoice, mock_shipping, mock_tax, mock_validate):
    ...
```

This test may be trying to test too much, or it may be fighting the design.

Ask:

```text
which behavior matters here?
```

Maybe tax calculation should be tested separately.

Maybe checkout should use real validation and calculation with a fake payment gateway.

Maybe sending confirmation is the only boundary to replace.

Mocks should simplify tests.

If they dominate the test, something is off.

---

# Under-Mocking

Under-mocking happens when tests use real dependencies that should be controlled.

Symptoms include:

* unit tests call real APIs
* tests require credentials
* tests fail offline
* tests send real emails
* tests mutate shared databases
* tests depend on current time
* tests depend on external service uptime
* tests are slow for no good reason

Example:

```python
def test_send_email():
    send_email("real@example.com", "Hello", "Body")
```

This is dangerous if it sends real email.

Better:

```python
def test_send_email_builds_expected_message():
    sender = FakeEmailSender()

    send_welcome_email(user, sender=sender)

    assert sender.messages[0].to == user.email
```

Integration tests may use real services in controlled environments.

Unit tests should not casually cross external boundaries.

---

# Mocking and Design Smells

Mocks often reveal design problems.

If a test requires patching five module-level globals, the code may have hidden dependencies.

If a test must patch time, randomness, environment, database, and network for one function, the function may do too much.

If you need to patch private helpers, the public boundary may be too broad or the helper may deserve extraction.

If every test needs many mocks, object construction may be too hard.

Mocking is not only a testing technique.

It is feedback about coupling.

The point is not:

```text
never mock
```

The point is:

```text
listen when mocking feels painful
```

Painful tests often point toward simpler design.

---

# Prefer Explicit Dependencies

Patching is often needed when dependencies are hidden.

For example:

```python
def generate_report():
    data = database.fetch_rows()
    pdf = render_pdf(data)
    storage.upload("report.pdf", pdf)
```

To test this, you may patch `database`, `render_pdf`, and `storage`.

An explicit design:

```python
def generate_report(database, renderer, storage):
    data = database.fetch_rows()
    pdf = renderer.render(data)
    storage.upload("report.pdf", pdf)
```

Now the test can pass fakes:

```python
def test_generate_report_uploads_pdf():
    database = FakeDatabase(rows=[{"name": "Asha"}])
    renderer = FakeRenderer(output=b"PDF")
    storage = FakeStorage()

    generate_report(database, renderer, storage)

    assert storage.files["report.pdf"] == b"PDF"
```

No patching required.

This is usually easier to understand.

Patching is useful.

Explicit dependencies are often better.

---

# Do Not Mock What You Do Not Own

There is a common testing guideline:

```text
do not mock what you do not own
```

It means you should be careful mocking third-party interfaces directly throughout your application tests.

Why?

Because you may misunderstand the interface.

Your mock may accept calls the real dependency rejects.

Your tests may become tied to SDK details.

When possible, wrap third-party dependencies behind your own boundary.

Then mock or fake your boundary.

This does not mean you can never patch a third-party function.

For small scripts, patching `requests.get` may be fine.

For larger systems, your own adapter layer usually pays off.

---

# Mocking Public Boundaries

Good mock targets are often public boundaries between your code and the outside world.

Examples:

* email sender interface
* payment gateway interface
* clock interface
* notification publisher
* file storage interface
* HTTP client abstraction
* queue publisher
* metrics collector
* feature flag provider

These boundaries express meaningful concepts.

Testing them with fakes or mocks makes sense because interaction is part of behavior.

Bad mock targets are often private steps inside an algorithm.

Examples:

* `_normalize_items`
* `_calculate_subtotal`
* `_build_query`
* `_parse_row`

If these helpers are complex, test them directly or through public behavior.

Do not automatically patch them out.

When you mock the core, you may stop testing the core.

---

# Mocking Return Values vs State

Sometimes a mock returning a value is enough.

Example:

```python
exchange_rates.get_rate.return_value = 82
```

Sometimes stateful behavior is clearer with a fake.

Example:

```python
class FakeStorage:
    def __init__(self):
        self.files = {}

    def upload(self, name, content):
        self.files[name] = content

    def download(self, name):
        return self.files[name]
```

If a dependency has meaningful state across multiple calls, a fake often reads better than a heavily configured mock.

Compare:

```python
storage.download.side_effect = [FileNotFoundError, b"content"]
```

with:

```python
storage = FakeStorage()
storage.upload("file.txt", b"content")
```

The fake may better match the domain.

---

# Mocking Exceptions

Exception paths are important.

Mocks are useful for forcing dependency failures.

Example:

```python
def test_payment_failure_marks_order_failed():
    gateway = Mock()
    gateway.charge.side_effect = PaymentDeclined
    order = Order(total=1000)

    process_payment(order, gateway)

    assert order.status == "payment_failed"
```

Without a mock, triggering `PaymentDeclined` from a real gateway may be difficult.

This is a strong use case for test doubles.

Error handling should not be left untested simply because real failures are hard to produce.

Mocks let you bring rare failures into ordinary test runs.

---

# Mocking Retries

Retry logic is a classic side-effect use case.

Example:

```python
def fetch_with_retry(fetch, attempts):
    last_error = None
    for _ in range(attempts):
        try:
            return fetch()
        except TimeoutError as error:
            last_error = error
    raise last_error
```

Test success after retries:

```python
from unittest.mock import Mock


def test_fetch_with_retry_succeeds_after_timeout():
    fetch = Mock(side_effect=[TimeoutError, "ok"])

    assert fetch_with_retry(fetch, attempts=2) == "ok"
    assert fetch.call_count == 2
```

Test final failure:

```python
import pytest
from unittest.mock import Mock


def test_fetch_with_retry_raises_after_all_attempts():
    fetch = Mock(side_effect=TimeoutError)

    with pytest.raises(TimeoutError):
        fetch_with_retry(fetch, attempts=3)

    assert fetch.call_count == 3
```

Here the interaction count is meaningful.

The retry contract includes how many attempts are made.

---

# Mocking Caches

Cache behavior often depends on interactions.

Example:

```python
def test_cache_avoids_duplicate_load():
    loader = Mock(return_value="data")
    cache = Cache(loader)

    assert cache.get("key") == "data"
    assert cache.get("key") == "data"

    loader.assert_called_once_with("key")
```

This is a good interaction assertion because avoiding duplicate work is the behavior.

But be precise.

If the implementation changes to prefetch related keys, this test may need updating.

The test should reflect the real contract, not incidental implementation.

---

# Mocking Metrics

Metrics are often boundary interactions.

Example:

```python
def test_failed_login_emits_metric():
    metrics = Mock()
    service = LoginService(metrics=metrics)

    service.login("a@example.com", "wrong-password")

    metrics.increment.assert_called_once_with("login.failed")
```

This may be valuable if metrics drive alerts or dashboards.

But avoid testing every incidental metric in low-level unit tests.

Metrics can become noisy.

Test important operational signals.

Do not make harmless instrumentation changes painful.

---

# Mocking Feature Flags

Feature flags influence behavior.

Tests should control them.

Example:

```python
def test_new_checkout_enabled():
    flags = Mock()
    flags.enabled.return_value = True
    service = CheckoutRouter(flags)

    assert service.route() == "new-checkout"
```

Also test disabled behavior:

```python
def test_new_checkout_disabled():
    flags = Mock()
    flags.enabled.return_value = False
    service = CheckoutRouter(flags)

    assert service.route() == "old-checkout"
```

Feature flags are hidden conditionals.

If important behavior depends on them, tests should make both sides visible.

---

# Mocking Security Boundaries

Be careful mocking security behavior.

For example, do not mock away authorization and then claim the endpoint is tested.

Bad:

```python
@patch("api.require_admin", return_value=True)
def test_delete_user(mock_admin, client):
    response = client.delete("/users/u-1")

    assert response.status_code == 204
```

This does not test real authorization.

It tests behavior after authorization has been bypassed.

That may be useful for a narrow handler test.

But you still need tests that verify unauthorized users are rejected.

For security boundaries, mocking can hide the most important risk.

Use it carefully.

---

# Mocking Payments

Payment systems should not be called casually in unit tests.

Use fakes or sandbox adapters.

Example:

```python
class FakePaymentGateway:
    def __init__(self):
        self.charges = []
        self.should_decline = False

    def charge(self, customer_id, amount):
        if self.should_decline:
            raise PaymentDeclined
        self.charges.append((customer_id, amount))
        return PaymentResult(id="pay_test_123")
```

Test:

```python
def test_checkout_charges_customer():
    gateway = FakePaymentGateway()
    service = CheckoutService(gateway)

    result = service.checkout("cust_1", amount=1000)

    assert result.payment_id == "pay_test_123"
    assert gateway.charges == [("cust_1", 1000)]
```

For actual payment provider integration, use dedicated integration tests with sandbox credentials and strict safeguards.

Unit tests should not risk real money movement.

---

# Mocking Email

Email is a classic fake boundary.

Do not send real email in unit tests.

A fake email sender is usually better than a generic mock because messages are domain objects.

Example:

```python
class FakeMailer:
    def __init__(self):
        self.outbox = []

    def send(self, message):
        self.outbox.append(message)
```

Test:

```python
def test_password_reset_sends_email():
    mailer = FakeMailer()
    service = PasswordResetService(mailer)

    service.request_reset("a@example.com")

    assert len(mailer.outbox) == 1
    assert mailer.outbox[0].to == "a@example.com"
    assert "reset" in mailer.outbox[0].subject.lower()
```

Many web frameworks provide test email outboxes for this reason.

Email is an external side effect.

Capture it, inspect it, and keep the test local.

---

# Monkey Patching and Global State

Monkey patching is runtime mutation.

Runtime mutation must be scoped.

Bad:

```python
settings.DEBUG = True
```

If this is not restored, later tests may see the changed value.

Good:

```python
def test_debug_behavior(monkeypatch):
    monkeypatch.setattr(settings, "DEBUG", True)

    assert debug_behavior() == "verbose"
```

The patch is undone after the test.

Global state is one of the main causes of test order dependency.

Monkey patch carefully.

---

# Patching Descriptors

Descriptors include:

* properties
* class methods
* static methods
* custom descriptor objects

Patching them can behave differently from ordinary attributes because descriptor lookup happens through the class.

For properties, patch the class using `PropertyMock`.

For class methods and static methods, patch the attribute where it is looked up:

```python
with patch.object(Service, "from_config") as mock_from_config:
    ...
```

If patching a descriptor seems strange, revisit the descriptor chapter.

Testing tools are not separate from the language model.

They operate through Python's name lookup, attribute lookup, and object protocol.

---

# Patching Constants

Sometimes code uses module-level constants:

```python
TIMEOUT_SECONDS = 5


def call_service(client):
    return client.get(timeout=TIMEOUT_SECONDS)
```

You can patch the constant:

```python
def test_call_service_uses_timeout(monkeypatch):
    monkeypatch.setattr("service.TIMEOUT_SECONDS", 30)
    client = Mock()

    call_service(client)

    client.get.assert_called_once_with(timeout=30)
```

But if many tests need to patch constants, configuration may be too rigid.

Passing configuration explicitly can be clearer:

```python
def call_service(client, timeout_seconds):
    return client.get(timeout=timeout_seconds)
```

Constants are simple until they need to vary.

When they vary in tests and production, treat them as configuration.

---

# Patching Singletons

Singleton-like objects are common:

```python
cache = Cache()


def get_user(user_id):
    return cache.get(user_id)
```

Tests may patch `cache`:

```python
def test_get_user(monkeypatch):
    fake_cache = FakeCache({"u-1": User(id="u-1")})
    monkeypatch.setattr("users.cache", fake_cache)

    assert get_user("u-1").id == "u-1"
```

This works.

But singleton-style globals can make tests harder because dependencies are hidden.

If many functions depend on global singletons, consider dependency injection or an application context object.

The goal is not to ban globals entirely.

The goal is to make important dependencies visible.

---

# Patching in Concurrent Tests

Monkey patching changes shared process state.

That matters for concurrent code.

If one thread reads an attribute while another test patches it, behavior can become unpredictable.

Most ordinary test runners run tests in one process and one thread unless configured otherwise.

But parallel test execution is common in larger projects.

Be cautious with:

* module-level globals
* environment variables
* current working directory
* `sys.path`
* `sys.modules`
* logging configuration
* process-wide random seeds

Parallel tests prefer isolated resources.

Patch scopes should be narrow.

Process-global mutation should be minimized.

---

# Patching Async Boundaries

Async code often depends on async clients.

Use `AsyncMock` or an async fake.

Example:

```python
from unittest.mock import AsyncMock


async def test_service_fetches_user():
    client = AsyncMock()
    client.get_user.return_value = {"id": "u-1"}
    service = UserService(client)

    result = await service.fetch("u-1")

    assert result == {"id": "u-1"}
    client.get_user.assert_awaited_once_with("u-1")
```

An async fake can be clearer:

```python
class FakeUserClient:
    async def get_user(self, user_id):
        return {"id": user_id}
```

Use async fakes when behavior is simple and domain-specific.

Use `AsyncMock` when you need call assertions or configurable failures.

---

# Mocking Generators

If code expects an iterator or generator, the test double must be iterable.

Example:

```python
def count_rows(source):
    return sum(1 for _ in source.rows())
```

Mock:

```python
source = Mock()
source.rows.return_value = iter([{"id": 1}, {"id": 2}])

assert count_rows(source) == 2
```

Be careful if the iterable is consumed more than once.

An iterator can be exhausted.

If the code calls `source.rows()` each time, returning a new iterator may be necessary:

```python
source.rows.side_effect = lambda: iter([{"id": 1}, {"id": 2}])
```

Generator behavior is stateful.

Tests should reflect that.

---

# Mocking Iteration Protocols

For objects used directly in a `for` loop, configure `__iter__`.

Example:

```python
from unittest.mock import MagicMock


items = MagicMock()
items.__iter__.return_value = iter([1, 2, 3])

assert list(items) == [1, 2, 3]
```

If the object has meaningful collection behavior, a real list or fake collection may be clearer.

Use protocol mocks when the protocol behavior is the focus.

Use ordinary data when ordinary data is enough.

---

# Mocking With pytest Fixtures

Mocks can be provided by fixtures.

Example:

```python
import pytest
from unittest.mock import Mock


@pytest.fixture
def email_sender():
    return Mock()


def test_signup_sends_email(email_sender):
    service = SignupService(email_sender)

    service.signup(User(email="a@example.com", name="Asha"))

    email_sender.send.assert_called_once()
```

This is useful when multiple tests need a similar dependency.

But avoid hiding too much in fixtures.

If a fixture creates a deeply configured mock, readers may have to jump around to understand the test.

Prefer explicit setup inside the test when it makes the behavior clearer.

---

# Naming Mocks

Mocks can have names.

Names improve failure output.

Example:

```python
from unittest.mock import Mock


payment_gateway = Mock(name="payment_gateway")
```

If an assertion fails, the mock's representation is easier to understand.

This is small, but useful in large test failures.

Clear names help future readers.

---

# Sealing Mocks

Mock objects can be sealed to prevent automatic creation of new attributes after configuration.

This catches unexpected attribute access.

Example:

```python
from unittest.mock import Mock, seal


client = Mock()
client.fetch.return_value = {"id": "u-1"}
seal(client)
```

After sealing, accessing new mock attributes fails.

This is useful when you want the flexibility of setup followed by stricter behavior during execution.

Sealing is not needed for every test.

But in complex mocks, it can reduce false confidence.

---

# Mocks and Type Hints

Mocks are dynamically flexible.

Type checkers usually cannot fully verify their behavior.

This can weaken static guarantees in tests.

For example:

```python
client = Mock()
client.fetch_user.return_value = {"id": "u-1"}
```

The mock can accept almost anything.

If you want stronger structure, use:

* `Protocol` interfaces
* typed fake classes
* autospec
* explicit adapters

Example with a protocol:

```python
from typing import Protocol


class EmailSender(Protocol):
    def send(self, to: str, subject: str, body: str) -> None:
        ...
```

A fake class implementing that protocol may be clearer than a generic mock.

Static typing and testing complement each other.

Mocks should not erase all interface clarity.

---

# Mocks and Refactoring

Good tests survive refactoring.

They fail when behavior changes.

They pass when implementation changes but behavior stays the same.

Over-mocked tests often do the opposite.

They fail when internal calls are renamed.

They pass when mocked internals mean real code was never exercised.

Before asserting a call, ask:

```text
would I want this test to fail if the implementation changed but behavior stayed correct?
```

If the answer is no, avoid that assertion.

Tests should support refactoring, not punish it.

---

# Mocks and Readability

Readability matters.

Compare:

```python
gateway = Mock()
gateway.charge.return_value.id = "pay_123"
gateway.charge.return_value.status = "succeeded"
gateway.charge.return_value.receipt.url = "https://example.test/r"
```

with:

```python
gateway = FakePaymentGateway(
    result=PaymentResult(
        id="pay_123",
        status="succeeded",
        receipt_url="https://example.test/r",
    )
)
```

The fake may be easier to understand.

Mocks are not always the shortest path to clarity.

If a mock setup starts looking like object construction, construct the object.

---

# A Practical Mocking Process

When deciding how to test code with a dependency, use this process:

1. Identify the behavior being tested.
2. Identify the dependency boundary.
3. Decide whether the real dependency is safe, fast, and deterministic.
4. If yes, consider using it directly.
5. If no, choose a fake, stub, spy, mock, or monkeypatch.
6. Prefer explicit dependency injection when it keeps the design clear.
7. Patch where the name is looked up.
8. Use specs or autospec for mocks that imitate real interfaces.
9. Assert behavior first, interactions only when they are meaningful.
10. Keep patch scope narrow.

This process prevents reflexive mocking.

The goal is not to avoid real dependencies at all costs.

The goal is controlled, meaningful tests.

---

# A Good Mocking Example

Suppose checkout must charge a customer and mark the order paid.

Production shape:

```python
class CheckoutService:
    def __init__(self, payment_gateway):
        self.payment_gateway = payment_gateway

    def checkout(self, order):
        payment = self.payment_gateway.charge(order.customer_id, order.total)
        order.mark_paid(payment.id)
        return order
```

Test:

```python
from unittest.mock import Mock


def test_checkout_charges_customer_and_marks_order_paid():
    gateway = Mock()
    gateway.charge.return_value = Payment(id="pay_123")
    order = Order(customer_id="cust_1", total=1000)
    service = CheckoutService(gateway)

    result = service.checkout(order)

    gateway.charge.assert_called_once_with("cust_1", 1000)
    assert result.status == "paid"
    assert result.payment_id == "pay_123"
```

This test is reasonable.

Charging the payment gateway is a boundary interaction.

The order state is behavior.

Both are important.

---

# A Brittle Mocking Example

Suppose invoice total is calculated like this:

```python
def invoice_total(items):
    subtotal = calculate_subtotal(items)
    tax = calculate_tax(subtotal)
    return subtotal + tax
```

Brittle test:

```python
@patch("invoice.calculate_subtotal", return_value=100)
@patch("invoice.calculate_tax", return_value=18)
def test_invoice_total(mock_tax, mock_subtotal):
    assert invoice_total([{"price": 100}]) == 118
    mock_subtotal.assert_called_once()
    mock_tax.assert_called_once_with(100)
```

This test mostly checks wiring.

It does not test the real subtotal or tax logic.

A behavior test may be better:

```python
def test_invoice_total_includes_tax():
    items = [{"price": 100, "quantity": 1}]

    assert invoice_total(items) == 118
```

If tax calculation is complex, test it separately.

Do not mock the core behavior out of existence.

---

# Choosing Between Fake and Mock

Use a fake when:

* the dependency has domain behavior
* state matters across calls
* readability improves with a small class
* multiple tests use the same substitute
* you want stronger typing

Use a mock when:

* you need call assertions
* behavior is simple to configure
* you need to force an exception
* you need to verify retry counts
* the dependency is a narrow boundary
* creating a fake would be unnecessary ceremony

Use monkeypatch when:

* the dependency is hidden behind a module lookup
* environment variables need control
* global attributes need temporary replacement
* changing production design is not practical

Use the real dependency when:

* it is fast
* it is deterministic
* it is local
* it is safe
* it provides important confidence

The best test double is the one that makes the test honest and clear.

---

# Common Mocking Mistakes

Common mistakes include:

* patching the wrong namespace
* using unrestricted mocks for real interfaces
* asserting private helper calls
* mocking the behavior being tested
* relying on real network calls in unit tests
* patching too broadly
* forgetting that environment variables are global
* mocking databases instead of testing real database behavior where needed
* using `ANY` for everything
* writing tests that only prove mocks were configured
* using mocks when a simple fake would be clearer
* hiding complex mock setup inside fixtures
* ignoring async-specific mocking needs

These mistakes are common because mocking is easy to start and hard to master.

The tool is simple.

The judgment is the craft.

---

# Mocking Checklist

Before committing a mock-heavy test, ask:

* Am I testing behavior or implementation wiring?
* Is this dependency a real boundary?
* Would a fake be clearer?
* Would dependency injection remove the need to patch?
* Did I patch where the name is looked up?
* Should this mock use `spec`, `spec_set`, or `autospec`?
* Is the patch scope narrow?
* Are global changes restored automatically?
* Does the test still exercise the code that matters?
* Would this test survive a harmless refactor?
* Is the assertion meaningful?
* Could this pass while production is broken?

The last question is the sharp one:

```text
could this pass while production is broken?
```

If yes, the test may need a stronger design.

---

# Chapter Summary

Mocking and monkey patching help tests control boundaries.

A test double is an object used in place of a real dependency.

Dummies satisfy parameters without being used.

Fakes are simplified working implementations.

Stubs provide predefined answers.

Spies record interactions.

Mocks are programmable objects that can return values, raise exceptions, and verify calls.

`unittest.mock` provides `Mock`, `MagicMock`, `AsyncMock`, `patch`, `patch.object`, `patch.dict`, `mock_open`, `ANY`, `call`, and autospeccing tools.

Plain mocks are flexible, but that flexibility can hide mistakes.

Specs, `spec_set`, autospec, and sealing make mocks more honest.

`patch` temporarily replaces attributes and restores them after the test.

The most important patching rule is:

```text
patch where the object is looked up
```

pytest's `monkeypatch` fixture temporarily changes attributes, dictionaries, environment variables, current directories, and import paths.

Monkey patching is runtime mutation, so scope and cleanup matter.

Mocks are most useful at boundaries such as APIs, email systems, payment gateways, clocks, random generators, queues, and external services.

Mocks are risky when they replace the core behavior being tested.

Interaction testing is appropriate when the interaction is the behavior.

Behavior testing is usually better for internal logic.

Fakes are often clearer than heavily configured mocks when state or domain meaning matters.

Mocking pain can reveal design problems such as hidden dependencies, excessive global state, and functions that do too much.

The central lesson is:

```text
mock boundaries, not the heart of the behavior
```

Good mocking makes tests deterministic and focused.

Poor mocking makes tests fragile and dishonest.

---

# Exercises

1. Write a fake email sender and use it to test a signup service.

2. Rewrite the same test using `Mock`. Compare readability.

3. Write a test that uses `side_effect` to simulate two timeouts followed by success.

4. Create a test that patches an environment variable with `monkeypatch.setenv`.

5. Create a test that patches an environment variable with `patch.dict`.

6. Write a test that fails because it patches the wrong namespace. Then fix it by patching where the name is looked up.

7. Use `create_autospec` to catch an incorrect function call signature.

8. Write a test for async code using `AsyncMock`.

9. Replace a complex mock setup with a small fake class.

10. Find a test that asserts private helper calls and rewrite it to assert public behavior.

---

# Preview of Chapter 74

Chapter 73 studied mocking and monkey patching.

We learned how to replace boundaries, configure mocks, assert calls, patch names, control environment variables, use fakes, and avoid over-mocking the core behavior of a system.

Next we study debugging.

Testing tells us that something is wrong.

Debugging helps us discover why.

Professional debugging is not random trial and error.

It is a disciplined investigation.

It uses:

* careful reproduction
* error messages
* tracebacks
* logging
* breakpoints
* minimal examples
* hypothesis testing
* binary search through changes
* inspection of state
* understanding of runtime behavior

Mocking and debugging are connected.

Mocks can help reproduce rare failures.

Bad mocks can also hide real failures.

Chapter 74 will show how to move from a failing behavior to a clear cause without thrashing through the codebase.

The transition is:

```text
tests reveal failure
debugging explains failure
```

That skill is central to professional engineering.
