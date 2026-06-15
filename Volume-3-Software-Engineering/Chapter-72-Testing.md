# Chapter 72 — Testing

Testing is where Python knowledge becomes engineering confidence.

Until now, the book has focused on how Python works.

You have learned:

* objects
* references
* functions
* data structures
* modules
* imports
* object-oriented programming
* descriptors
* decorators
* exceptions
* files
* concurrency
* type hints
* bytecode
* CPython internals
* C extensions

That knowledge helps you write code.

Testing helps you trust code.

Those are not the same thing.

A program can look correct.

A program can run once.

A program can pass a manual check.

A program can even work for the case you had in mind when you wrote it.

None of that means the program is reliable.

Professional engineering begins when you stop relying only on memory, hope, and manual inspection.

Testing gives your code an executable memory.

It records what the system is supposed to do.

It lets future changes ask:

```text
did I preserve the behavior that matters?
```

That question is the heart of testing.

---

# What Testing Is

Testing is the practice of running code under controlled conditions and checking whether its behavior matches expectations.

A test usually has three parts:

```text
arrange
act
assert
```

Arrange means preparing the state.

Act means running the behavior being tested.

Assert means checking the result.

For example:

```python
def apply_discount(price, percent):
    return price - (price * percent / 100)


def test_apply_discount_reduces_price():
    price = 100
    percent = 20

    result = apply_discount(price, percent)

    assert result == 80
```

This test is tiny, but the structure is already visible.

The arranged state is:

```python
price = 100
percent = 20
```

The action is:

```python
result = apply_discount(price, percent)
```

The assertion is:

```python
assert result == 80
```

Testing is not only about finding bugs.

Testing is also about preserving design decisions.

When a test exists, the behavior it describes becomes harder to accidentally erase.

Without tests, behavior lives in people's heads.

With tests, behavior becomes part of the project.

---

# What Testing Is Not

Testing is not proof that a program has no bugs.

For most real programs, that standard is impossible.

Testing samples behavior.

It explores important cases.

It protects known contracts.

It reveals mistakes.

It increases confidence.

But it does not prove total correctness.

This matters because beginners often expect tests to provide certainty.

Professional engineers use tests differently.

They ask:

```text
what would make this code safer to change?
```

That is a better question than:

```text
can I prove this code is perfect?
```

Tests do not remove the need for thought.

They make thought repeatable.

---

# Why Tests Matter

Tests matter because software changes.

If code were written once and never modified, testing would still be useful.

But most valuable software is not static.

It changes because:

* requirements change
* bugs are found
* dependencies upgrade
* Python versions change
* performance needs evolve
* security concerns appear
* new users use the system differently
* code is refactored
* features are added
* old assumptions expire

Every change creates risk.

The risk is not only that the new code is wrong.

The risk is that the new code breaks old behavior.

That kind of bug is called a regression.

A regression means:

```text
something that used to work no longer works
```

Tests are the main defense against regressions.

They let a project say:

```text
before this change is accepted, old promises must still hold
```

This is why testing is not a side activity.

Testing is part of how professional teams move quickly without becoming reckless.

---

# Manual Testing

Manual testing means running the program yourself and checking behavior by observation.

For example:

```text
start the web server
open the browser
fill the form
click submit
check the page
```

Manual testing is useful.

It helps you feel the product.

It reveals workflow problems.

It catches visual and usability issues.

It is often the first way a developer explores a feature.

But manual testing does not scale well.

It is slow.

It is easy to forget steps.

It depends on the tester's attention.

It is difficult to repeat exactly.

It becomes painful as the application grows.

Manual testing is like reading code carefully.

It is valuable, but it should not be your only safety system.

Automated tests exist because humans are not good at repeating hundreds or thousands of checks perfectly every day.

---

# Automated Testing

Automated testing means writing code that checks other code.

Instead of manually verifying a function every time you change it, you write a test once and run it repeatedly.

For example:

```python
def normalize_username(username):
    return username.strip().lower()


def test_normalize_username_strips_spaces_and_lowercases():
    assert normalize_username("  Narendra  ") == "narendra"
```

Now every future run of the test suite checks this behavior.

The benefit compounds over time.

One test is helpful.

Ten tests are more helpful.

A thousand meaningful tests become a safety net for the whole system.

The goal is not to test every line mechanically.

The goal is to test important behavior so that change becomes safer.

---

# The Testing Pyramid

A common model for test strategy is the testing pyramid.

It divides tests by scope and cost.

At the bottom are many small tests.

At the top are fewer broad tests.

The pyramid usually looks like this:

```text
              end-to-end tests
          integration tests
      unit tests
```

The exact labels vary by team.

The principle is stable:

```text
small tests are cheaper and faster
broad tests are more realistic but slower and more fragile
```

A healthy project usually has many unit tests, a smaller number of integration tests, and a carefully chosen set of end-to-end tests.

This is not a law.

It is a design pressure.

If all your tests are broad and slow, developers stop running them.

If all your tests are tiny and isolated, you may miss failures at the boundaries.

Good testing uses multiple layers.

---

# Unit Tests

A unit test checks a small piece of behavior in isolation.

The unit might be:

* a function
* a method
* a class
* a small module
* a narrow rule

For example:

```python
def calculate_total(items):
    total = 0
    for item in items:
        total += item["price"] * item["quantity"]
    return total


def test_calculate_total_adds_price_times_quantity():
    items = [
        {"price": 10, "quantity": 2},
        {"price": 5, "quantity": 3},
    ]

    assert calculate_total(items) == 35
```

This is a unit test because it checks one rule without needing a database, network, web server, queue, or external service.

Unit tests are valuable because they are fast.

They are also precise.

When a unit test fails, the failure usually points close to the bug.

But unit tests have a weakness.

They can pass even when pieces do not work together.

That is why unit tests are necessary but not sufficient.

---

# Integration Tests

An integration test checks whether multiple parts work together.

Examples:

* a service function writing to a database
* an API handler calling validation and persistence code
* a parser reading a real file format
* a background job using a queue
* a repository layer using a real database connection

Consider a function that saves users.

A unit test might test validation rules without a database.

An integration test might create a temporary database and verify that the user is really stored.

Integration tests catch boundary problems.

Those problems include:

* schema mismatches
* serialization mistakes
* incorrect SQL
* transaction issues
* import wiring errors
* configuration problems
* file path assumptions
* encoding mistakes
* API contract mismatches

Integration tests are slower than unit tests.

They often need more setup.

They are more likely to fail because of environment issues.

But they provide confidence that isolated components actually cooperate.

---

# End-to-End Tests

An end-to-end test checks a full user or system flow.

For a web application, an end-to-end test might:

```text
open the browser
log in
create a record
edit it
verify the change appears
log out
```

For a CLI application, an end-to-end test might:

```text
run the command
inspect output
check exit code
verify files created on disk
```

End-to-end tests are powerful because they exercise the system the way users experience it.

They are also expensive.

They can be slow.

They can be brittle.

They can fail because of timing, network, browser state, test data, or environment differences.

The mistake is not writing end-to-end tests.

The mistake is relying on them for everything.

Use them for critical workflows.

Do not use them as a substitute for well-designed lower-level tests.

---

# Regression Tests

A regression test is a test written to prevent a bug from returning.

Suppose a user reports that this function crashes for an empty list:

```python
def average(values):
    return sum(values) / len(values)
```

The bug is a division by zero.

You might fix it:

```python
def average(values):
    if not values:
        return 0
    return sum(values) / len(values)
```

Then you write a test:

```python
def test_average_empty_list_returns_zero():
    assert average([]) == 0
```

That test is a regression test.

It says:

```text
this bug happened once
do not let it happen again
```

Regression tests are especially valuable because they are grounded in reality.

They protect against known failures, not imagined ones.

When a production bug is fixed without a test, the project has missed an opportunity to strengthen itself.

---

# Characterization Tests

Sometimes you inherit code that has little or no test coverage.

You may not fully understand the intended behavior.

In that situation, you can write characterization tests.

A characterization test records what the current system does.

It does not always claim that the behavior is ideal.

It says:

```text
this is how the system behaves today
```

For legacy systems, this is useful before refactoring.

You first capture current behavior.

Then you change the implementation.

Then the tests tell you whether you preserved behavior.

For example:

```python
def test_legacy_invoice_rounding_behavior():
    invoice = make_legacy_invoice(amount=10.005)

    assert invoice.total == "10.01"
```

Maybe the rounding rule is strange.

Maybe the formatting should eventually change.

But before refactoring, the current behavior matters.

Characterization tests turn unknown code into documented code.

---

# Test Names

A test name should explain behavior.

Weak test name:

```python
def test_user():
    ...
```

Better test name:

```python
def test_create_user_rejects_duplicate_email():
    ...
```

The second name tells the reader:

* what operation is being tested
* what condition matters
* what outcome is expected

Good test names act like tiny specifications.

When a test fails, the name should help you understand the broken promise.

Prefer behavior-oriented names.

For example:

```python
def test_cart_total_includes_item_quantities():
    ...


def test_parser_rejects_missing_required_header():
    ...


def test_token_expires_after_configured_ttl():
    ...
```

Avoid names that merely repeat implementation details.

For example:

```python
def test_for_loop():
    ...


def test_if_statement():
    ...


def test_private_helper():
    ...
```

Tests should usually describe observable behavior, not internal mechanics.

---

# Assertions

An assertion checks that something is true.

In pytest, ordinary `assert` statements are common:

```python
def test_addition():
    assert 2 + 2 == 4
```

In `unittest`, assertion methods are common:

```python
import unittest


class AdditionTests(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(2 + 2, 4)
```

Both styles check behavior.

The important question is not which spelling looks nicer.

The important question is whether the assertion is meaningful.

Weak assertion:

```python
assert result is not None
```

This may be too vague.

Better assertion:

```python
assert result.status == "paid"
assert result.total == 1250
assert result.currency == "INR"
```

Specific assertions give better failure information.

They also document the contract more clearly.

---

# Testing Exceptions

Many functions are supposed to raise exceptions for invalid input.

That behavior should be tested too.

With pytest:

```python
import pytest


def divide(a, b):
    if b == 0:
        raise ValueError("division by zero is not allowed")
    return a / b


def test_divide_rejects_zero_denominator():
    with pytest.raises(ValueError):
        divide(10, 0)
```

With `unittest`:

```python
import unittest


class DivideTests(unittest.TestCase):
    def test_divide_rejects_zero_denominator(self):
        with self.assertRaises(ValueError):
            divide(10, 0)
```

Testing exceptions matters because errors are part of the API.

If a function promises to reject bad input, the rejection behavior is not accidental.

It is a contract.

Sometimes the exception message matters too.

For example, API validation may return user-facing errors.

In those cases, test the message carefully.

But avoid over-testing messages that are only developer diagnostics unless the exact text is part of the public contract.

---

# Testing Floating-Point Results

Floating-point arithmetic is not exact decimal arithmetic.

This matters in tests.

For example:

```python
result = 0.1 + 0.2
```

The result is close to `0.3`, but it may not be exactly equal to `0.3`.

A brittle test would be:

```python
assert 0.1 + 0.2 == 0.3
```

A better test uses approximate comparison:

```python
import pytest


def test_float_sum():
    assert 0.1 + 0.2 == pytest.approx(0.3)
```

The general lesson is:

```text
assert according to the nature of the value
```

For integers, exact comparison is usually right.

For money, avoid binary floating point and use integers or `Decimal`.

For measurements, approximate comparison may be appropriate.

For sets, order may not matter.

For lists, order usually matters.

The assertion should match the real contract.

---

# pytest

pytest is one of the most widely used testing frameworks in the Python ecosystem.

It is popular because it allows simple tests to stay simple.

A test can be just a function:

```python
def test_uppercase():
    assert "python".upper() == "PYTHON"
```

You do not need to create a class.

You do not need to inherit from a base class.

You can use normal `assert`.

pytest also provides powerful features for larger test suites:

* fixtures
* parametrization
* test discovery
* plugin support
* temporary path helpers
* exception testing
* warning testing
* output capture
* test selection
* assertion introspection

pytest is not part of the Python standard library.

It is a third-party package.

That matters for packaging and environments.

But in many professional Python projects, pytest is the default test runner.

---

# unittest

`unittest` is Python's built-in unit testing framework.

It is part of the standard library.

That means it is available wherever Python is available.

Its style is class-based:

```python
import unittest


def slugify(text):
    return text.strip().lower().replace(" ", "-")


class SlugifyTests(unittest.TestCase):
    def test_slugify_lowercases_and_replaces_spaces(self):
        self.assertEqual(slugify("Hello Python"), "hello-python")


if __name__ == "__main__":
    unittest.main()
```

The important pieces are:

* `unittest.TestCase`
* methods whose names begin with `test`
* assertion methods such as `assertEqual`
* optional `setUp` and `tearDown`
* command-line execution through `python -m unittest`

`unittest` is especially common in older codebases and standard-library-style projects.

Even if you prefer pytest, you should be able to read `unittest` tests.

Professional work often means maintaining code you did not choose.

---

# pytest and unittest Together

pytest can run many `unittest`-style tests.

This is useful during migration.

A project may begin with `unittest` and gradually adopt pytest features.

For example, existing tests may look like this:

```python
import unittest


class TaxTests(unittest.TestCase):
    def test_tax_is_added(self):
        self.assertEqual(calculate_total(100, tax_rate=0.18), 118)
```

pytest can often discover and run this test.

That lets teams improve test ergonomics without rewriting everything at once.

A mature engineer does not treat tools as identity.

Use the tool that fits the project.

The behavior being tested matters more than the framework fashion.

---

# Test Discovery

Test discovery is how a test runner finds tests.

Common pytest conventions include:

```text
test_*.py
*_test.py
```

Inside those files, pytest commonly discovers functions named:

```text
test_*
```

For example:

```text
project/
    src/
        shop/
            cart.py
    tests/
        test_cart.py
```

A simple command runs the tests:

```bash
pytest
```

With `unittest`, a common command is:

```bash
python -m unittest
```

or:

```bash
python -m unittest discover
```

Discovery conventions matter because they reduce friction.

If every developer can run the test suite with one predictable command, tests are more likely to be used.

---

# Where Tests Should Live

There are two common layouts.

One layout keeps tests outside the package:

```text
project/
    src/
        billing/
            invoice.py
    tests/
        test_invoice.py
```

Another layout keeps tests near the code:

```text
project/
    billing/
        invoice.py
        test_invoice.py
```

Both can work.

The separate `tests/` directory is common for applications and libraries.

It makes test code visibly separate from production code.

It also helps avoid accidentally shipping tests as part of the runtime package unless that is desired.

Tests near code can be convenient in some projects, especially when modules are small and tightly organized.

The best layout is the one your team can use consistently.

Consistency matters more than theoretical purity.

---

# Arrange, Act, Assert

The arrange-act-assert pattern keeps tests readable.

Consider this test:

```python
def test_checkout_charges_customer():
    customer = Customer(email="a@example.com")
    cart = Cart()
    cart.add(Product("Book", price=500), quantity=2)
    payment_gateway = FakePaymentGateway()
    checkout = Checkout(payment_gateway)
    receipt = checkout.pay(customer, cart)
    assert receipt.total == 1000
    assert payment_gateway.last_charge.email == "a@example.com"
    assert payment_gateway.last_charge.amount == 1000
```

It works, but the structure is dense.

Spacing can make the phases clearer:

```python
def test_checkout_charges_customer():
    customer = Customer(email="a@example.com")
    cart = Cart()
    cart.add(Product("Book", price=500), quantity=2)
    payment_gateway = FakePaymentGateway()
    checkout = Checkout(payment_gateway)

    receipt = checkout.pay(customer, cart)

    assert receipt.total == 1000
    assert payment_gateway.last_charge.email == "a@example.com"
    assert payment_gateway.last_charge.amount == 1000
```

The blank lines are not decorative.

They make the thought process visible.

Good tests are not only correct.

They are easy to read when they fail six months later.

---

# Fixtures

A fixture is reusable test setup.

In pytest, a fixture is commonly defined with `@pytest.fixture`:

```python
import pytest


@pytest.fixture
def cart():
    cart = Cart()
    cart.add(Product("Notebook", price=100), quantity=3)
    return cart


def test_cart_total(cart):
    assert cart.total() == 300
```

The test asks for the fixture by parameter name:

```python
def test_cart_total(cart):
    ...
```

pytest sees the parameter `cart`, finds the fixture named `cart`, runs it, and passes the result to the test.

Fixtures are useful when setup is:

* repeated
* meaningful
* slightly expensive
* shared across related tests
* easier to name than duplicate

But fixtures can also be overused.

If a fixture hides too much, tests become difficult to understand.

This is a common problem in large suites.

A reader opens a test and sees:

```python
def test_invoice_status(enterprise_customer, seeded_orders, app_context):
    ...
```

Now the reader must chase three fixtures before understanding the test.

Fixtures should reduce noise, not hide the story.

---

# Fixture Scope

Some fixtures are created for every test.

Others can be shared across a module, class, package, or session.

pytest supports fixture scopes such as:

* function
* class
* module
* package
* session

The default scope is function.

That means a fresh fixture value is created for each test.

Fresh setup is often best because tests should not accidentally influence each other.

For expensive setup, broader scope can help.

For example, a test database container might be started once per test session.

But shared fixtures carry risk.

If one test mutates shared state, another test may fail depending on execution order.

The rule is:

```text
share expensive infrastructure carefully
avoid sharing mutable test state casually
```

Fast tests are good.

Independent tests are better.

---

# Parametrized Tests

Parametrization lets one test body run against multiple inputs.

Without parametrization, you might write:

```python
def test_slugify_simple_text():
    assert slugify("Hello World") == "hello-world"


def test_slugify_extra_spaces():
    assert slugify("  Hello  World  ") == "hello-world"


def test_slugify_mixed_case():
    assert slugify("PyThOn") == "python"
```

With pytest parametrization:

```python
import pytest


@pytest.mark.parametrize(
    ("text", "expected"),
    [
        ("Hello World", "hello-world"),
        ("  Hello  World  ", "hello-world"),
        ("PyThOn", "python"),
    ],
)
def test_slugify(text, expected):
    assert slugify(text) == expected
```

Parametrization is excellent for rules with many examples.

It keeps the behavior table visible.

Use it when the same concept is being tested across multiple cases.

Do not force unrelated scenarios into one parametrized test just to reduce lines.

Duplication in tests is not always bad.

Clarity matters more than clever compression.

---

# Edge Cases

An edge case is an input or condition near a boundary.

For example:

* empty list
* one item
* maximum allowed value
* minimum allowed value
* missing field
* duplicate value
* unknown enum
* `None`
* empty string
* whitespace-only string
* non-ASCII text
* invalid date
* leap day
* timeout
* permission denied

Edge cases matter because bugs often live at boundaries.

If a function handles a list, test:

```text
zero elements
one element
many elements
```

If a rule has a threshold, test:

```text
just below threshold
exactly at threshold
just above threshold
```

For example:

```python
def is_adult(age):
    return age >= 18


def test_is_adult_boundary():
    assert is_adult(17) is False
    assert is_adult(18) is True
    assert is_adult(19) is True
```

This kind of test is simple but powerful.

It checks the boundary where mistakes are likely.

---

# Happy Paths and Sad Paths

A happy path is the normal successful flow.

A sad path is an error, rejection, failure, or exceptional flow.

Both matter.

For a login function, happy path:

```text
valid email and password returns a session
```

Sad paths:

```text
unknown email is rejected
wrong password is rejected
locked account is rejected
missing password is rejected
too many attempts are rate-limited
```

Beginners often test only the happy path.

Production systems fail in the sad paths.

Professional tests make invalid behavior visible too.

This does not mean every possible failure deserves a test.

It means important error behavior should be intentional.

If users, callers, or operators depend on an error behavior, test it.

---

# Deterministic Tests

A deterministic test gives the same result every time when the code has not changed.

Nondeterministic tests are dangerous because they reduce trust.

Common sources of nondeterminism include:

* current time
* random numbers
* network calls
* filesystem state
* environment variables
* global mutable state
* test order dependencies
* threads
* async scheduling
* real external services
* timezone assumptions

If a test sometimes passes and sometimes fails, developers learn to ignore failures.

That is one of the worst outcomes for a test suite.

A failing test should mean:

```text
something needs attention
```

Not:

```text
maybe the test is in a mood today
```

To make tests deterministic, control the inputs.

Pass time in explicitly.

Seed random number generators.

Use temporary directories.

Avoid shared global state.

Replace network calls with controlled fakes where appropriate.

---

# Testing Time

Time is a common source of fragile tests.

Consider:

```python
from datetime import datetime, timedelta


def token_expired(created_at, ttl_seconds):
    return datetime.now() >= created_at + timedelta(seconds=ttl_seconds)
```

This function is hard to test precisely because it calls `datetime.now()` directly.

A better design accepts the current time:

```python
from datetime import datetime, timedelta


def token_expired(created_at, ttl_seconds, now):
    return now >= created_at + timedelta(seconds=ttl_seconds)
```

Now the test can control time:

```python
from datetime import datetime


def test_token_expired_after_ttl():
    created_at = datetime(2026, 1, 1, 10, 0, 0)
    now = datetime(2026, 1, 1, 10, 5, 0)

    assert token_expired(created_at, ttl_seconds=300, now=now) is True
```

This is not only easier to test.

It is also better design.

Testing often reveals hidden dependencies.

When a function reaches out to time, randomness, network, disk, or process environment, tests become harder.

That difficulty is information.

It tells you where your code is coupled to the outside world.

---

# Temporary Files and Directories

Tests often need files.

Do not write tests that depend on random files on your machine.

Do not write tests that leave files behind accidentally.

pytest provides `tmp_path`:

```python
def test_write_report(tmp_path):
    report_path = tmp_path / "report.txt"

    write_report(report_path, "hello")

    assert report_path.read_text() == "hello"
```

`tmp_path` gives each test a temporary directory represented as a `pathlib.Path`.

The test can create files safely.

The runner can clean them up.

This keeps tests isolated from the developer's machine and from each other.

Temporary paths are also useful for testing:

* file creation
* file deletion
* serialization
* configuration loading
* directory scanning
* CLI outputs
* import-related behavior

Filesystem tests are not bad.

Uncontrolled filesystem tests are bad.

---

# Environment Variables

Many applications read configuration from environment variables.

Tests must control those variables.

In pytest, `monkeypatch` can temporarily set environment variables:

```python
def read_api_url():
    import os

    return os.environ["API_URL"]


def test_read_api_url(monkeypatch):
    monkeypatch.setenv("API_URL", "https://example.test")

    assert read_api_url() == "https://example.test"
```

After the test, pytest restores the environment.

This matters because environment variables are process-global.

If one test changes an environment variable and does not restore it, another test may fail later.

Global state must be treated carefully in tests.

The same applies to:

* current working directory
* module-level caches
* logging configuration
* locale settings
* warning filters
* imported modules

Global state can be tested.

But it must be cleaned up.

---

# Fakes, Stubs, Spies, and Mocks

Tests often replace a real dependency with a controlled test double.

Common test doubles include:

* fake
* stub
* spy
* mock

People use these words loosely, but the distinctions are useful.

A fake is a working simplified implementation.

For example:

```python
class FakeEmailSender:
    def __init__(self):
        self.sent = []

    def send(self, to, subject, body):
        self.sent.append({
            "to": to,
            "subject": subject,
            "body": body,
        })
```

A stub returns predefined answers.

For example:

```python
class StubExchangeRateProvider:
    def rate_for(self, currency):
        return 82
```

A spy records how it was used.

The fake email sender above is also acting as a spy because tests can inspect `sent`.

A mock usually means an object with programmable expectations or behavior, often created by a mocking library.

Mocks are powerful.

They can also make tests brittle if they assert too much about internal collaboration.

Chapter 73 will focus on mocking and monkey patching in detail.

For now, remember this:

```text
prefer testing observable behavior
mock boundaries, not every internal function call
```

---

# Dependency Injection for Testability

Dependency injection means giving a function or object its dependencies from the outside instead of hard-coding them inside.

Harder to test:

```python
class SignupService:
    def signup(self, email):
        sender = SmtpEmailSender()
        sender.send(email, "Welcome", "Thanks for joining")
```

Easier to test:

```python
class SignupService:
    def __init__(self, sender):
        self.sender = sender

    def signup(self, email):
        self.sender.send(email, "Welcome", "Thanks for joining")
```

Now a test can pass a fake sender:

```python
def test_signup_sends_welcome_email():
    sender = FakeEmailSender()
    service = SignupService(sender)

    service.signup("a@example.com")

    assert sender.sent == [
        {
            "to": "a@example.com",
            "subject": "Welcome",
            "body": "Thanks for joining",
        }
    ]
```

This is not a testing trick.

It is design.

Code that accepts dependencies explicitly is often easier to understand, configure, and reuse.

Testing pressure can guide better architecture.

---

# Testing Pure Functions

Pure functions are easy to test.

A pure function:

* receives inputs
* returns outputs
* does not mutate external state
* does not read hidden state
* does not perform I/O

For example:

```python
def tax_for(amount, rate):
    return amount * rate
```

The test is straightforward:

```python
def test_tax_for():
    assert tax_for(100, 0.18) == 18
```

Pure functions are not always possible.

Programs must eventually read files, call APIs, write databases, send emails, and display output.

But pure cores are valuable.

A common design pattern is:

```text
imperative shell
functional core
```

The shell handles I/O.

The core handles decisions.

The core is easy to test.

For example, instead of mixing file reading and calculation:

```python
def total_from_file(path):
    data = json.loads(path.read_text())
    return sum(item["price"] for item in data["items"])
```

Separate them:

```python
def total_items(items):
    return sum(item["price"] for item in items)


def total_from_file(path):
    data = json.loads(path.read_text())
    return total_items(data["items"])
```

Now `total_items` can be tested with plain data.

The file behavior can be tested separately.

---

# Testing Classes

Classes should be tested through their public behavior.

Suppose you have:

```python
class Cart:
    def __init__(self):
        self._items = []

    def add(self, product, quantity=1):
        self._items.append((product, quantity))

    def total(self):
        return sum(product.price * quantity for product, quantity in self._items)
```

A test should not care that items are stored in `_items`.

It should care about behavior:

```python
def test_cart_total_uses_quantity():
    cart = Cart()

    cart.add(Product("Pen", price=10), quantity=3)

    assert cart.total() == 30
```

If later the implementation changes from a list to a dictionary, the test should still pass if behavior is unchanged.

Avoid tests like:

```python
assert cart._items == [(product, 3)]
```

That test locks onto an implementation detail.

Sometimes testing internals is justified.

But it should be a conscious tradeoff, not the default.

---

# Testing Dataclasses

Dataclasses often hold data and light behavior.

If a dataclass is only a plain container, you may not need many direct tests.

For example:

```python
from dataclasses import dataclass


@dataclass
class Point:
    x: int
    y: int
```

Testing that Python assigns `x` and `y` correctly is usually not useful.

That behavior belongs to the standard library.

But if your dataclass has validation or methods, test those:

```python
from dataclasses import dataclass


@dataclass
class Percentage:
    value: int

    def __post_init__(self):
        if not 0 <= self.value <= 100:
            raise ValueError("percentage must be between 0 and 100")
```

Tests:

```python
import pytest


def test_percentage_accepts_valid_value():
    assert Percentage(75).value == 75


def test_percentage_rejects_invalid_value():
    with pytest.raises(ValueError):
        Percentage(150)
```

Do not test the framework.

Test your behavior.

---

# Testing Iterators and Generators

Iterators and generators are tested by consuming them.

For example:

```python
def first_n_squares(n):
    for number in range(n):
        yield number * number
```

Test:

```python
def test_first_n_squares():
    assert list(first_n_squares(5)) == [0, 1, 4, 9, 16]
```

For infinite generators, do not consume everything.

Use `itertools.islice`:

```python
from itertools import islice


def count_from(start):
    current = start
    while True:
        yield current
        current += 1


def test_count_from():
    assert list(islice(count_from(10), 3)) == [10, 11, 12]
```

If a generator performs cleanup, test that cleanup explicitly.

Generators can hold resources.

They can also have subtle state transitions.

Treat them as behavior, not just syntax.

---

# Testing Context Managers

Context managers define enter and exit behavior.

They are used with `with`.

For example:

```python
class Recorder:
    def __init__(self):
        self.events = []

    def __enter__(self):
        self.events.append("enter")
        return self

    def __exit__(self, exc_type, exc, traceback):
        self.events.append("exit")
```

Test:

```python
def test_context_manager_records_enter_and_exit():
    with Recorder() as recorder:
        recorder.events.append("inside")

    assert recorder.events == ["enter", "inside", "exit"]
```

Also test exception behavior if it matters.

A context manager can suppress exceptions by returning `True` from `__exit__`.

If your context manager is not supposed to suppress exceptions, test that errors still propagate.

---

# Testing Async Code

Async code requires async-aware tests.

With pytest, many projects use `pytest-asyncio` or a similar plugin.

A test may look like:

```python
import pytest


@pytest.mark.asyncio
async def test_fetch_user():
    user = await fetch_user("u-123")

    assert user.id == "u-123"
```

Async tests need special care because concurrency can introduce nondeterminism.

Avoid tests that rely on arbitrary sleep durations:

```python
await asyncio.sleep(0.1)
```

Sometimes waiting is necessary.

But if a test depends on timing guesses, it can become flaky.

Prefer explicit synchronization:

* events
* queues
* task completion
* controlled fakes
* dependency injection

Async code should be tested at both levels:

* pure decision logic outside the event loop
* async boundary behavior inside the event loop

Keep as much business logic as possible outside timing-sensitive code.

---

# Testing Command-Line Programs

Command-line programs should be tested by checking:

* arguments
* exit codes
* stdout
* stderr
* files created
* files modified
* error behavior

If your CLI has a pure function behind it, test that function separately.

For the CLI layer, test the command behavior.

Example:

```python
import subprocess
import sys


def test_cli_uppercases_input():
    result = subprocess.run(
        [sys.executable, "-m", "tool", "hello"],
        capture_output=True,
        text=True,
        check=False,
    )

    assert result.returncode == 0
    assert result.stdout.strip() == "HELLO"
```

CLI tests can be slower than pure unit tests, but they are valuable because packaging and entry points can fail in ways ordinary function tests do not catch.

Use a small number of CLI-level tests for important flows.

Do not test every internal branch only through subprocesses.

---

# Testing APIs

API tests check request and response behavior.

For a web API, useful assertions include:

* status code
* response body
* response headers
* validation errors
* authentication behavior
* authorization behavior
* idempotency
* database side effects
* external calls made or avoided

An API test should be written from the caller's perspective.

For example:

```python
def test_create_user_returns_201(client):
    response = client.post(
        "/users",
        json={"email": "a@example.com", "name": "Asha"},
    )

    assert response.status_code == 201
    assert response.json()["email"] == "a@example.com"
```

Do not only test that a route function calls a service function.

Test the behavior visible at the API boundary.

APIs are contracts.

Tests should protect those contracts.

---

# Testing Databases

Database tests require careful isolation.

Common strategies include:

* transaction rollback after each test
* temporary test database
* in-memory database when compatible
* containers for realistic database engines
* fixtures that create known data
* migrations applied before tests

Avoid using a developer's real database.

Avoid tests that depend on existing rows.

Avoid tests that pass only when run in a certain order.

Database tests should create the data they need.

For example:

```python
def test_find_user_by_email(db_session):
    user = User(email="a@example.com")
    db_session.add(user)
    db_session.commit()

    found = find_user_by_email(db_session, "a@example.com")

    assert found.id == user.id
```

Database tests are often integration tests.

They are slower than pure unit tests.

But if your application depends heavily on database behavior, avoiding database tests is false economy.

SQL, migrations, indexes, constraints, and transactions are real behavior.

Test the parts that matter.

---

# Test Data

Test data should be clear.

Avoid magical data unless it is intentionally meaningful.

Weak:

```python
user = User("x", "y", 123)
```

Better:

```python
user = User(
    email="customer@example.com",
    name="Customer",
    loyalty_points=120,
)
```

Readable test data helps readers understand intent.

Factories can help when creating objects is repetitive:

```python
def make_user(**overrides):
    data = {
        "email": "user@example.com",
        "name": "User",
        "is_active": True,
    }
    data.update(overrides)
    return User(**data)
```

Then:

```python
def test_inactive_user_cannot_login():
    user = make_user(is_active=False)

    assert can_login(user) is False
```

Factories should make tests clearer.

If a factory hides important details, override values explicitly in the test.

---

# Test Isolation

Tests should be independent.

One test should not require another test to run first.

This is important because test runners may:

* run tests in different order
* run tests in parallel
* run a single selected test
* stop after the first failure

Bad:

```python
created_user_id = None


def test_create_user():
    global created_user_id
    created_user_id = create_user("a@example.com").id


def test_delete_user():
    delete_user(created_user_id)
    assert find_user(created_user_id) is None
```

The second test depends on the first.

Better:

```python
def test_delete_user():
    user = create_user("a@example.com")

    delete_user(user.id)

    assert find_user(user.id) is None
```

Each test creates what it needs.

That independence makes the suite easier to trust.

---

# Flaky Tests

A flaky test sometimes passes and sometimes fails without a relevant code change.

Flaky tests are expensive because they damage trust.

Common causes include:

* real time
* race conditions
* random data
* network dependency
* test order dependency
* shared database state
* filesystem leftovers
* insufficient cleanup
* async timing assumptions
* external service instability

Do not normalize flaky tests.

Do not simply rerun them until they pass and forget them.

A flaky test is either:

* revealing a real race or design problem
* written in a nondeterministic way
* depending on an unstable environment

All three deserve attention.

Sometimes a flaky test must be temporarily skipped.

If so, leave a reason and a path back.

Skipping without accountability turns into decay.

---

# Slow Tests

Slow tests reduce feedback.

If the suite takes too long, developers run it less often.

Then bugs are found later.

Late bugs are more expensive.

Common causes of slow tests include:

* excessive database setup
* real network calls
* repeated expensive fixture creation
* unnecessary end-to-end coverage
* sleeps
* large files
* cryptographic work with high cost settings
* starting servers repeatedly
* full application boot for tiny behavior checks

The answer is not always to delete slow tests.

Some slow tests are valuable.

The answer is to separate feedback layers.

For example:

```text
fast unit tests run on every save or commit
integration tests run before merge
end-to-end tests run in CI or scheduled pipelines
```

A good test strategy considers speed as part of design.

---

# Coverage

Code coverage measures how much code was executed by tests.

It can answer questions like:

```text
which lines ran during the test suite?
which branches were exercised?
which files have no coverage?
```

Coverage is useful.

It reveals blind spots.

It can prevent large untested areas from being ignored.

But coverage is not the same as quality.

A test can execute a line without asserting meaningful behavior.

For example:

```python
def test_weak():
    calculate_invoice([{"price": 10}])
```

This may increase coverage.

It does not check the result.

Better:

```python
def test_invoice_total_includes_price():
    assert calculate_invoice([{"price": 10}]).total == 10
```

Coverage should be used as a signal, not a substitute for judgment.

High coverage with poor assertions is theater.

Lower coverage with strong tests may be more useful.

The goal is meaningful confidence.

---

# Branch Coverage

Line coverage tells you whether a line ran.

Branch coverage tells you whether decision paths ran.

Consider:

```python
def fee_for(amount):
    if amount < 100:
        return 5
    return 0
```

A test with `amount=50` executes the `if` branch.

It may also count most lines as covered.

But it does not test the `amount >= 100` behavior.

Better:

```python
def test_fee_for_small_amount():
    assert fee_for(50) == 5


def test_fee_for_large_amount():
    assert fee_for(100) == 0
```

Branch coverage helps reveal untested decisions.

It is especially useful for validation logic, permissions, state machines, and error handling.

---

# Test-Driven Development

Test-driven development, or TDD, is a workflow where tests are written before implementation.

The classic cycle is:

```text
red
green
refactor
```

Red means write a failing test.

Green means write enough code to pass.

Refactor means improve the design while tests stay passing.

TDD is not magic.

It does not guarantee good design.

It does not replace thinking.

But it can be useful because it forces you to define behavior before implementation.

For example:

```python
def test_password_must_have_at_least_eight_characters():
    assert validate_password("short").valid is False
```

Before writing the validation code, you have already decided one rule.

That can clarify design.

TDD works best when requirements are understandable at the behavior level.

It is harder when exploring unknown APIs, user interfaces, or performance-sensitive internals.

Use it as a tool, not a religion.

---

# Tests as Documentation

Good tests document behavior.

Unlike comments, tests can be executed.

A comment may say:

```python
# Empty carts cannot be checked out.
```

A test can say:

```python
def test_empty_cart_cannot_be_checked_out():
    cart = Cart()

    with pytest.raises(EmptyCartError):
        checkout(cart)
```

The test is stronger than the comment because it verifies the claim.

When someone wants to know how a feature behaves, tests can provide examples.

This is one reason test readability matters.

Tests are not just machines for producing green check marks.

They are part of the project's explanation.

---

# Doctest

`doctest` is a standard-library module that checks examples written in docstrings or text files.

For example:

```python
def add(a, b):
    """
    Return the sum of two numbers.

    >>> add(2, 3)
    5
    """
    return a + b
```

The doctest runner can execute the example and verify the output.

Doctests are useful when:

* examples are simple
* documentation should stay executable
* the example is more important than test structure
* public API behavior can be shown clearly

Doctests are less suitable for complex setup, large outputs, nondeterministic behavior, or detailed test organization.

Use doctest for examples.

Use regular test frameworks for larger test suites.

---

# Property-Based Testing

Example-based tests check specific cases.

Property-based tests check general properties across many generated cases.

For example, a sorting function should produce output that:

* has the same elements as the input
* is ordered
* has the same length as the input

Instead of testing only:

```python
assert sort_numbers([3, 1, 2]) == [1, 2, 3]
```

A property-based test might generate many lists and check the properties.

In Python, the Hypothesis library is commonly used for property-based testing.

Conceptually:

```python
from hypothesis import given
from hypothesis import strategies as st


@given(st.lists(st.integers()))
def test_sort_numbers_returns_ordered_values(values):
    result = sort_numbers(values)

    assert result == sorted(values)
```

Property-based testing is powerful for:

* parsers
* serializers
* numerical code
* state machines
* validation logic
* transformations
* algorithms

It is not a replacement for examples.

Examples communicate important named scenarios.

Properties explore broader input space.

Together, they are stronger.

---

# Golden Master Tests

A golden master test compares current output to a previously approved output.

This is useful when output is large or complicated.

Examples:

* generated HTML
* generated JSON
* reports
* formatted documents
* compiler output
* CLI snapshots

The test might read an expected file:

```python
def test_report_output_matches_snapshot(snapshot_path):
    report = render_report(sample_data())

    assert report == snapshot_path.read_text()
```

Golden master tests are useful, but they can become lazy.

If developers update snapshots without reviewing changes, the test loses value.

The important question is:

```text
is this output change intentional?
```

Golden master tests work best when humans review diffs carefully.

---

# Testing Error Messages

Error messages can be part of user experience.

For public APIs, CLI tools, and validation systems, messages may matter.

For example:

```python
def test_missing_email_error_message():
    result = validate_user({"name": "Asha"})

    assert result.errors["email"] == "Email is required."
```

But not every message deserves an exact test.

For internal exceptions, exact text can make tests brittle.

This is often enough:

```python
with pytest.raises(ConfigurationError):
    load_config(path)
```

If the message is part of a contract, test it.

If it is only a debugging aid, consider testing the exception type and context instead.

---

# Testing Security Behavior

Security-sensitive behavior deserves explicit tests.

Examples:

* password hashing uses the expected verifier
* plaintext passwords are not stored
* unauthorized users are rejected
* users cannot access other users' data
* expired tokens fail
* invalid signatures fail
* dangerous file paths are rejected
* input is escaped or sanitized where required
* rate limits are applied
* permission checks happen before state changes

Security tests are not a complete security program.

They do not replace audits, threat modeling, dependency scanning, or careful design.

But they prevent obvious regressions in critical rules.

For example:

```python
def test_user_cannot_read_another_users_invoice(client, user_a, user_b, invoice_b):
    client.login(user_a)

    response = client.get(f"/invoices/{invoice_b.id}")

    assert response.status_code == 403
```

Authorization bugs are often simple and severe.

Test them directly.

---

# Testing Performance Assumptions

Most tests should not assert exact timing.

Timing is noisy.

Different machines produce different results.

But sometimes performance assumptions matter.

For example:

* an algorithm should not become quadratic
* a cache should avoid repeated expensive calls
* a query should not perform N+1 database access
* a parser should handle large inputs

You can test structural performance behavior without relying only on wall-clock time.

For example, test call counts:

```python
def test_user_lookup_uses_cache():
    source = CountingUserSource()
    cache = UserCache(source)

    cache.get("u-1")
    cache.get("u-1")

    assert source.calls == 1
```

For deeper performance work, use profiling and benchmarks.

Chapter 79 will focus on profiling.

In ordinary unit tests, keep performance assertions stable and meaningful.

---

# Continuous Integration

Continuous integration, or CI, runs checks automatically when code changes.

A typical CI pipeline may run:

* formatting checks
* linting
* type checking
* unit tests
* integration tests
* packaging checks
* security scans
* documentation builds

The key benefit is consistency.

Instead of relying on every developer to remember every command, CI runs the expected checks in a clean environment.

CI helps answer:

```text
is this change safe to merge?
```

CI does not replace local testing.

Developers should still run relevant tests before pushing.

But CI catches omissions and environment differences.

A project without CI often depends on individual discipline.

A project with CI builds discipline into the workflow.

---

# Selecting Tests

Large projects need ways to run subsets of tests.

pytest supports selecting tests by path:

```bash
pytest tests/test_cart.py
```

By test name pattern:

```bash
pytest -k cart
```

By marker:

```bash
pytest -m integration
```

`unittest` supports running modules, classes, and methods:

```bash
python -m unittest tests.test_cart.CartTests.test_total
```

Test selection matters because feedback should match the work.

During development, run the narrow tests often.

Before merging, run the broader suite.

Fast local feedback and comprehensive CI are complementary.

---

# Marking Tests

pytest markers let you label tests.

For example:

```python
import pytest


@pytest.mark.integration
def test_create_user_in_database(db_session):
    ...
```

Then:

```bash
pytest -m integration
```

or:

```bash
pytest -m "not integration"
```

Markers are useful for separating:

* unit tests
* integration tests
* slow tests
* network tests
* database tests
* platform-specific tests

Use markers intentionally.

If every test has many labels, selection becomes confusing.

The goal is clear workflow, not decoration.

---

# Skipping Tests

Sometimes a test should be skipped.

Reasons may include:

* platform-specific behavior
* optional dependency missing
* external service unavailable
* feature not supported on this Python version
* known temporary issue

In pytest:

```python
import sys
import pytest


@pytest.mark.skipif(sys.platform == "win32", reason="uses POSIX signals")
def test_signal_handler():
    ...
```

Skipping is acceptable when the reason is honest and specific.

Bad:

```python
@pytest.mark.skip(reason="fails")
```

Better:

```python
@pytest.mark.skip(reason="blocked by payment sandbox outage tracked in ISSUE-431")
```

Skipped tests should not become invisible debt.

If a test is permanently irrelevant, remove it.

If it is temporarily blocked, track it.

---

# Expected Failures

An expected failure marks a test that currently fails for a known reason.

This can be useful when:

* documenting a known bug
* developing a feature incrementally
* tracking behavior that should be fixed later

In pytest:

```python
import pytest


@pytest.mark.xfail(reason="rounding bug tracked in ISSUE-220")
def test_invoice_rounding_half_up():
    assert round_invoice(10.005) == 10.01
```

Expected failures should be used carefully.

They are not a place to hide broken behavior indefinitely.

An expected failure is a reminder:

```text
we know this is wrong
we have chosen not to fix it yet
```

That choice should remain visible.

---

# Test Smells

Test smells are signs that a test suite may become difficult to maintain.

Common smells include:

* tests that assert implementation details
* tests with unclear names
* tests that depend on order
* tests that require real external services
* tests that sleep unnecessarily
* tests with huge hidden fixtures
* tests that check too many unrelated things
* tests that duplicate production logic
* tests that never fail even when code is broken
* tests that are updated mechanically without review

One especially dangerous smell is duplicating the implementation in the test.

For example:

```python
def test_total():
    items = [{"price": 10}, {"price": 20}]

    expected = sum(item["price"] for item in items)

    assert total(items) == expected
```

If `total` is also implemented with the same logic, the test may repeat the bug.

Better:

```python
def test_total_adds_item_prices():
    items = [{"price": 10}, {"price": 20}]

    assert total(items) == 30
```

Expected values should often be concrete.

Concrete examples are easier to inspect.

---

# One Assertion Per Test

You may hear the rule:

```text
one assertion per test
```

This rule has a useful intention.

It prevents tests from checking too many unrelated behaviors.

But taken literally, it can become awkward.

This test is fine:

```python
def test_create_user_response(client):
    response = client.post("/users", json={"email": "a@example.com"})

    assert response.status_code == 201
    assert response.json()["email"] == "a@example.com"
```

The assertions describe one behavior: creating a user returns the expected response.

This test is too broad:

```python
def test_user_system(client):
    create_response = client.post("/users", json={"email": "a@example.com"})
    login_response = client.post("/login", json={"email": "a@example.com"})
    invoice_response = client.get("/invoices")
    delete_response = client.delete("/users/current")

    assert create_response.status_code == 201
    assert login_response.status_code == 200
    assert invoice_response.status_code == 200
    assert delete_response.status_code == 204
```

That test checks many behaviors.

If it fails, the failure may be harder to interpret.

The better rule is:

```text
one concept per test
```

---

# Testing Private Functions

Should you test private functions?

Usually, test public behavior.

Private functions are implementation details.

If tests depend heavily on private functions, refactoring becomes harder.

However, there are exceptions.

Testing a private helper may be reasonable when:

* the helper contains complex logic
* public behavior would require huge setup
* the helper is stable in practice
* extracting it into a public module-level function would be honest

In Python, privacy is conventional.

A leading underscore says:

```text
this is internal
```

It does not make access impossible.

Use judgment.

If a helper is important enough to test directly, it may deserve a clearer design boundary.

---

# Testing and Refactoring

Tests make refactoring safer.

Refactoring means changing code structure without changing behavior.

Examples:

* rename functions
* split modules
* extract classes
* remove duplication
* simplify conditionals
* replace algorithms
* reorganize packages

Without tests, refactoring is risky because you must manually remember behavior.

With tests, you can change structure and repeatedly ask:

```text
does the externally visible behavior still hold?
```

This is why tests should not over-specify internals.

If every test knows the internal structure, refactoring breaks tests even when behavior is preserved.

Good tests protect behavior.

Overly intimate tests freeze implementation.

---

# Testing and Design

Hard-to-test code is often telling you something.

Maybe dependencies are hidden.

Maybe functions do too much.

Maybe global state is overused.

Maybe I/O is mixed with business logic.

Maybe time and randomness are hard-coded.

Maybe classes create their own collaborators instead of receiving them.

Testing does not merely verify design.

Testing pressures design.

For example, this function is hard to test:

```python
def send_welcome_email(user_id):
    user = database.fetch_user(user_id)
    body = template_engine.render("welcome.html", user=user)
    smtp.send(user.email, "Welcome", body)
```

It reaches into database, template engine, and SMTP directly.

A more testable design separates decisions from effects:

```python
def build_welcome_email(user):
    return Email(
        to=user.email,
        subject="Welcome",
        body=f"Hello {user.name}",
    )
```

Now the email construction can be tested simply.

The sending boundary can be tested separately.

Testability is not the only design goal.

But it is a valuable signal.

---

# Testing Legacy Code

Legacy code often lacks tests.

The mistake is trying to test everything at once.

A better strategy is incremental.

Start by testing:

* code you need to change
* code that recently broke
* code with high business value
* code with high risk
* code that many modules depend on
* code that is hard to reason about

Before changing legacy code, write characterization tests around current behavior.

Then make the change.

Then add tests for the new intended behavior.

Do not demand perfect coverage before improvement begins.

Professional engineering often means making messy systems safer one boundary at a time.

---

# Testing Libraries

Library tests have special concerns.

A library may be used by people outside your team.

Its public API is a contract.

Library tests should protect:

* public functions
* public classes
* error types
* documented behavior
* compatibility promises
* supported Python versions
* serialization formats
* deprecation behavior

Libraries also need compatibility testing.

For example, a library may support multiple Python versions.

CI should test those versions.

If a library supports multiple operating systems, CI should include them where practical.

The wider the audience, the more important compatibility testing becomes.

---

# Testing Applications

Application tests have different priorities.

An application usually serves a specific product or organization.

Its tests should protect:

* business workflows
* authorization rules
* data integrity
* critical API behavior
* background jobs
* integrations
* reporting
* migrations
* operational failure modes

Application tests can be more end-to-end than library tests because the deployed behavior matters.

But the pyramid still applies.

Put as much logic as possible in testable units.

Use integration and end-to-end tests to protect the boundaries.

---

# Testing Data Pipelines

Data pipeline tests often need to check transformations.

Important questions include:

* are required columns present?
* are types correct?
* are nulls handled?
* are duplicates handled?
* are filters correct?
* are aggregations correct?
* are time windows correct?
* are late records handled?
* is output schema stable?

A pipeline test might use a tiny input dataset with known output.

For example:

```python
def test_daily_revenue_groups_by_day():
    rows = [
        {"date": "2026-01-01", "amount": 100},
        {"date": "2026-01-01", "amount": 50},
        {"date": "2026-01-02", "amount": 25},
    ]

    result = daily_revenue(rows)

    assert result == [
        {"date": "2026-01-01", "revenue": 150},
        {"date": "2026-01-02", "revenue": 25},
    ]
```

Small representative data is often better than huge fixtures.

A good data test makes the transformation obvious.

---

# Testing Serialization

Serialization turns objects into formats such as JSON, CSV, YAML, bytes, or database rows.

Deserialization turns those formats back into objects.

Important tests include:

* object serializes to expected shape
* serialized data deserializes correctly
* missing fields are handled
* extra fields are handled
* invalid values are rejected
* version compatibility is preserved

Round-trip tests are useful:

```python
def test_user_round_trip():
    user = User(id="u-1", email="a@example.com")

    data = serialize_user(user)
    restored = deserialize_user(data)

    assert restored == user
```

But round-trip tests are not enough.

If both serializer and deserializer share the same wrong assumption, a round trip may still pass.

Also test the external format directly:

```python
def test_user_serializes_to_public_shape():
    user = User(id="u-1", email="a@example.com")

    assert serialize_user(user) == {
        "id": "u-1",
        "email": "a@example.com",
    }
```

External formats are contracts.

Test them explicitly.

---

# Testing Imports and Packaging

Some problems only appear after packaging.

Examples:

* missing package data
* incorrect entry points
* import-time side effects
* optional dependencies not declared
* module paths wrong after installation
* files present locally but absent in wheel

Tests can catch some of this.

For example:

```python
def test_package_imports():
    import my_package

    assert my_package.__version__
```

Packaging checks may build and install the package in a clean environment before running tests.

This becomes more important for libraries distributed to others.

Chapter 76 will cover packaging in detail.

For now, remember:

```text
tests run from the source tree do not prove the package is correctly built
```

---

# Testing Type Hints

Type hints are not tests.

Static type checking is not testing.

But they complement tests.

Tests check runtime behavior.

Type checkers analyze code before runtime.

For example, a type checker may catch:

```python
def total(values: list[int]) -> int:
    return sum(values)


total(["1", "2"])
```

A test could catch this too, but type checking catches an entire class of mistakes earlier.

Volume III separates these topics deliberately.

Testing comes first because it protects behavior.

Type hints and static type checking come later because they protect structural expectations.

Strong engineering uses both.

---

# Good Tests Fail Well

A good test does not merely fail.

It fails informatively.

Consider:

```python
assert result == expected
```

This can be fine if `result` and `expected` are readable.

But if both are giant nested structures, failure output may be hard to understand.

Sometimes it is better to assert important fields separately:

```python
assert response.status_code == 400
assert response.json()["error"]["code"] == "INVALID_EMAIL"
```

Good failure output helps the next developer fix the problem quickly.

That next developer may be you.

And you may not remember what you meant when you wrote the test.

Write tests for tired future humans.

---

# A Practical Test Writing Process

When adding or changing behavior, a practical process is:

1. Identify the behavior that matters.
2. Write the simplest test that describes it.
3. Run the test and see it fail if possible.
4. Implement or change the code.
5. Run the test again.
6. Add edge cases that are likely to matter.
7. Run related tests.
8. Refactor if the design is awkward.
9. Run the broader suite before merging.

This process is not ceremony.

It is a way to keep feedback close to the change.

Small feedback loops make engineering calmer.

---

# A Testing Checklist

When reviewing tests, ask:

* Does the test name describe behavior?
* Is the setup clear?
* Is the assertion meaningful?
* Does the test avoid unnecessary implementation details?
* Does the test run deterministically?
* Does the test clean up after itself?
* Does it cover important edge cases?
* Does it cover important failure paths?
* Is it at the right level of the pyramid?
* Would this test help diagnose a failure?
* Is the test too broad?
* Is the test too coupled?
* Is the test fast enough for its purpose?
* Does it protect a real contract?

Not every test must satisfy every ideal.

But these questions expose tradeoffs.

Testing is engineering.

Engineering is tradeoff management.

---

# Common Beginner Mistakes

Beginners often make predictable testing mistakes.

They write tests after manually confirming everything and only test what already worked.

They test implementation details instead of behavior.

They write one giant test for an entire feature.

They use random data without controlling it.

They depend on the current date.

They require network access for ordinary unit tests.

They share mutable state between tests.

They assert only that something is not `None`.

They chase coverage numbers without improving confidence.

They skip failing tests without tracking why.

They write tests that are harder to understand than the code being tested.

The cure is not shame.

The cure is better habits.

Start small.

Make behavior visible.

Keep tests readable.

Prefer deterministic inputs.

Test important boundaries.

Let the suite grow with the system.

---

# Testing Mindset

Testing is not about distrusting yourself.

It is about respecting change.

Code is read, edited, reused, refactored, optimized, deployed, and misunderstood.

Tests give the project a way to push back when important behavior is accidentally broken.

Good tests do not make developers timid.

They make developers braver.

When the suite is meaningful, you can refactor more confidently.

You can upgrade dependencies more safely.

You can fix bugs without reopening old wounds.

You can onboard new contributors faster.

Testing is not separate from professional programming.

It is one of the main ways programming becomes professional.

---

# Chapter Summary

Testing checks whether code behavior matches expectations under controlled conditions.

Manual testing is useful but does not scale as the main safety system.

Automated tests give code an executable memory.

Unit tests are small, fast, and precise.

Integration tests check whether parts work together.

End-to-end tests protect critical user or system flows.

Regression tests prevent known bugs from returning.

Characterization tests capture current behavior before changing legacy code.

Good test names describe behavior.

Good assertions check meaningful contracts.

pytest provides simple function-based tests and powerful features such as fixtures and parametrization.

`unittest` is Python's standard-library testing framework and remains important in many codebases.

Fixtures should reduce repetition without hiding the story of a test.

Parametrization is useful when one behavior should hold across many examples.

Edge cases and sad paths matter because many production failures occur at boundaries.

Deterministic tests are essential for trust.

Time, randomness, environment variables, global state, files, networks, threads, and async scheduling must be controlled carefully.

Fakes, stubs, spies, and mocks replace real dependencies in tests, but should be used with judgment.

Testability often improves design by exposing hidden dependencies and mixed responsibilities.

Coverage is a useful signal, but meaningful assertions matter more than numbers.

CI makes project checks repeatable in a clean environment.

Flaky tests damage trust and should be treated as real engineering problems.

The central lesson is:

```text
tests are executable confidence
```

They do not prove perfection.

They make important behavior visible, repeatable, and safer to change.

---

# Exercises

1. Write tests for a function that calculates discounts. Include zero discount, full discount, and invalid percentage cases.

2. Write tests for a function that normalizes usernames by trimming spaces and lowercasing text. Include empty strings and whitespace-only strings.

3. Take a function that calls `datetime.now()` directly. Refactor it so the current time can be passed in, then test it.

4. Write a parametrized test for a slugification function.

5. Create a fake email sender and use it to test a signup service.

6. Write a test that uses a temporary directory to verify that a report file is created.

7. Write one happy-path test and three sad-path tests for a login validation function.

8. Find a test that uses `assert result is not None` and replace it with a more meaningful assertion.

9. Write a regression test for a bug you have fixed in the past.

10. Take a broad test that checks multiple concepts and split it into smaller behavior-focused tests.

---

# Preview of Chapter 73

Chapter 72 introduced testing as the foundation of professional engineering confidence.

We studied unit tests, integration tests, end-to-end tests, regression tests, fixtures, parametrization, edge cases, determinism, coverage, CI, and the relationship between testing and design.

Next we study mocking and monkey patching.

These tools matter because real systems have boundaries.

Code often depends on:

* APIs
* databases
* filesystems
* clocks
* random number generators
* environment variables
* email systems
* queues
* payment gateways
* slow services
* unreliable networks

Tests need ways to control those boundaries.

Mocking lets a test replace a dependency with a controlled object.

Monkey patching lets a test temporarily replace attributes, functions, environment variables, or module-level behavior.

Used well, these techniques make difficult code testable.

Used carelessly, they create brittle tests that know too much about implementation details.

The transition is:

```text
testing defines the behavior we trust
mocking and monkey patching control the boundaries around that behavior
```

Chapter 73 will show how to use these tools without letting them take over the design.
