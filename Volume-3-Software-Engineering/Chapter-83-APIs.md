# Chapter 83 — APIs

An API is a contract.

The letters stand for Application Programming Interface.

That sounds formal.

The idea is simple:

```text
an API is how one piece of software offers behavior to another piece of software
```

A function has an API.

A class has an API.

A module has an API.

A package has an API.

A command-line tool has an API.

An HTTP service has an API.

An event stream has an API.

An API tells callers:

* what they can call
* what inputs are accepted
* what outputs are returned
* what errors can happen
* what behavior is promised
* what is stable
* what may change

Architecture decides where boundaries exist.

APIs decide how those boundaries are used.

If the boundary is unclear, code becomes tangled.

If the API is unclear, callers guess.

Guessed contracts are fragile contracts.

Good APIs reduce guessing.

---

# APIs Are Everywhere

Many developers hear API and think only of HTTP:

```text
GET /users/123
POST /orders
```

HTTP APIs are important.

But APIs are broader.

This function is an API:

```python
def normalize_email(email: str) -> str:
    return email.strip().lower()
```

This class is an API:

```python
class EmailSender:
    def send(self, message: EmailMessage) -> None:
        ...
```

This module is an API:

```python
from payments import charge_card, refund_payment
```

This CLI command is an API:

```bash
python -m importer users.csv --dry-run
```

This event is an API:

```json
{
  "type": "OrderPaid",
  "order_id": "ord_123",
  "payment_id": "pay_456"
}
```

Anywhere one part of software depends on another, there is an API.

---

# Public and Private APIs

Not every callable is meant for everyone.

Public APIs are meant to be used by callers outside the implementation.

Private APIs are internal details.

In Python, privacy is mostly convention.

Leading underscores communicate internal use:

```python
def _normalize_internal(value: str) -> str:
    ...
```

This does not prevent access.

It communicates:

```text
do not rely on this unless you own this module
```

Public APIs need more care.

Changing a private helper may be fine.

Changing a public function may break users.

API design starts by deciding what is public.

If everything is public, everything becomes hard to change.

---

# API Surface Area

API surface area is the amount of behavior exposed to callers.

Large surface area means more names, parameters, return shapes, options, and behaviors that callers can depend on.

Small surface area is easier to maintain.

Example large surface:

```python
class UserService:
    def create_user(...): ...
    def update_user(...): ...
    def delete_user(...): ...
    def normalize_email(...): ...
    def hash_password(...): ...
    def send_welcome_email(...): ...
    def render_email_template(...): ...
    def open_database_session(...): ...
```

Some of these may be internal details.

A tighter public surface:

```python
class UserService:
    def register(self, command: RegisterUser) -> UserId:
        ...
```

The fewer promises you expose, the easier it is to improve the implementation.

Good APIs are generous in usefulness and conservative in surface area.

---

# Function APIs

A function API includes:

* name
* parameters
* parameter types
* default values
* return type
* exceptions
* side effects
* performance expectations
* mutation behavior

Example:

```python
def calculate_total(items: Sequence[LineItem]) -> Money:
    ...
```

This says:

```text
callers pass line items
the function returns money
the function does not require a list specifically
```

A vague API:

```python
def process(data):
    ...
```

This raises questions:

* what is data?
* what happens to it?
* what returns?
* can it fail?
* does it mutate?

Good function APIs answer questions at the boundary.

---

# Names Are API Design

Names are part of the API.

Compare:

```python
def handle(user):
    ...
```

with:

```python
def deactivate_user(user_id: UserId) -> None:
    ...
```

The second name tells a story.

It says:

```text
this function deactivates a user
it expects a user ID
it is called for its effect
```

Good API names are:

* specific
* domain-oriented
* consistent
* boring enough to be obvious

Avoid names that reveal implementation instead of behavior.

Example:

```python
def update_row_flag(...):
    ...
```

Maybe the domain name is:

```python
def mark_invoice_sent(...):
    ...
```

Callers care about behavior.

Implementation names leak details.

---

# Parameters

Parameters should make correct calls easy.

Too many positional parameters can be dangerous:

```python
create_user("Asha", "asha@example.com", True, False, 30)
```

What do `True`, `False`, and `30` mean?

Keyword arguments improve clarity:

```python
create_user(
    name="Asha",
    email="asha@example.com",
    active=True,
    admin=False,
    trial_days=30,
)
```

For larger inputs, use a dataclass or command object:

```python
@dataclass
class RegisterUser:
    name: str
    email: str
    trial_days: int
```

Then:

```python
register_user(RegisterUser(name="Asha", email="asha@example.com", trial_days=30))
```

Parameter design is API design.

---

# Return Values

Return values should be predictable.

Bad:

```python
def find_user(email: str):
    if found:
        return user
    if invalid:
        return False
    return None
```

This API returns:

* `User`
* `False`
* `None`

Callers must guess what each means.

Better:

```python
def find_user(email: str) -> User | None:
    ...
```

Invalid email can be handled separately:

```python
def find_user(email: EmailAddress) -> User | None:
    ...
```

or:

```python
def find_user(email: str) -> User | None:
    if not valid_email(email):
        raise ValueError("invalid email")
```

Return shape should express meaning clearly.

Do not use mixed return types casually.

---

# Exceptions as API

Exceptions are part of an API.

If a function may raise a domain error, callers need to know.

Example:

```python
class PaymentDeclined(Exception):
    ...


def charge(order: Order) -> PaymentResult:
    ...
```

Documentation should say:

```text
Raises PaymentDeclined when the payment provider rejects the payment.
```

Use specific exception types.

Bad:

```python
raise Exception("failed")
```

Better:

```python
raise PaymentProviderUnavailable(provider="stripe")
```

Specific exceptions let callers handle different failures differently.

Expected business failures and unexpected system failures should not look the same.

---

# Side Effects

Some APIs return values.

Some APIs cause effects.

Examples:

* send email
* write file
* save database row
* publish event
* mutate object
* log audit entry

Side effects should be obvious.

This name is unclear:

```python
def invoice(order):
    ...
```

Does it create an invoice?

Return one?

Save one?

Send one?

Better:

```python
def create_invoice(order: Order) -> Invoice:
    ...


def save_invoice(invoice: Invoice) -> None:
    ...


def send_invoice(invoice: Invoice) -> None:
    ...
```

Side-effect APIs need careful naming and tests.

Hidden effects surprise callers.

---

# Idempotency

An API is idempotent if repeating the same operation has the same effect as doing it once.

Example:

```text
mark invoice as sent
```

Calling it twice may be harmless.

Example not idempotent:

```text
charge card
```

Calling it twice may charge twice.

Idempotency matters for:

* retries
* distributed systems
* HTTP APIs
* background jobs
* payment operations
* event consumers

If an API may be retried, define idempotency behavior.

For payment-like APIs, idempotency keys are common:

```python
def charge(order: Order, idempotency_key: str) -> PaymentResult:
    ...
```

APIs should make dangerous repetition hard.

---

# Class APIs

A class API includes:

* constructor
* public methods
* public attributes
* properties
* invariants
* lifecycle
* context manager behavior
* equality behavior
* string representation

Example:

```python
class Cart:
    def add(self, item: LineItem) -> None:
        ...

    def total(self) -> Money:
        ...
```

This is clear.

Callers know how to use the cart.

Avoid exposing internal collections directly:

```python
cart.items.append(item)
```

If callers can mutate internals freely, the class cannot protect invariants.

Prefer methods that express valid operations:

```python
cart.add(item)
cart.remove(item_id)
```

The public class API should preserve the object's rules.

---

# Module APIs

A module API is the set of names intended for import.

Example:

```python
from payments import charge, refund, PaymentDeclined
```

The module should make public names easy to find.

`__all__` can communicate exports:

```python
__all__ = ["charge", "refund", "PaymentDeclined"]
```

Internal helpers can use leading underscores:

```python
def _build_provider_payload(...):
    ...
```

A module with too many public names becomes hard to understand.

Group related behavior.

Keep internal details internal.

Module API design affects package usability.

---

# Package APIs

A package API is what users of the package rely on.

Example:

```python
from requests import get
```

or:

```python
from your_package import Client
```

Package APIs need stability.

If users import:

```python
from your_package.internal.client_impl import Client
```

they are depending on internals.

You can discourage that with naming:

```text
your_package/_internal/
```

and provide a public import:

```python
from your_package import Client
```

Design public imports deliberately.

The easier the public path is, the less likely users are to reach into internals.

---

# CLI APIs

A command-line interface is an API.

Example:

```bash
file-organizer scan ~/Downloads --dry-run --json
```

The API includes:

* command name
* subcommands
* arguments
* flags
* environment variables
* exit codes
* stdout format
* stderr behavior
* config files

If scripts depend on your CLI output, changing it is a breaking change.

Human-friendly output and machine-readable output should be separated.

Example:

```bash
tool status
tool status --json
```

The `--json` output should be stable.

Pretty terminal text can evolve more freely.

---

# HTTP APIs

HTTP APIs are contracts over the network.

They include:

* URLs
* methods
* request headers
* request bodies
* response status codes
* response bodies
* authentication
* pagination
* filtering
* sorting
* rate limits
* error format
* versioning

Example:

```http
GET /users/u-123
```

Response:

```json
{
  "id": "u-123",
  "email": "asha@example.com",
  "active": true
}
```

Callers depend on this shape.

Changing `email` to `email_address` can break clients.

HTTP APIs need extra care because callers may be outside your codebase.

---

# HTTP Methods

Common HTTP methods have conventional meanings.

`GET` retrieves.

`POST` creates or triggers processing.

`PUT` replaces.

`PATCH` partially updates.

`DELETE` deletes.

These meanings matter because clients, proxies, caches, and developers expect them.

Bad:

```http
GET /charge-card
```

This changes state through a retrieval method.

Better:

```http
POST /payments
```

Use method semantics honestly.

Surprising APIs create bugs.

---

# Status Codes

HTTP status codes communicate outcome.

Common examples:

* `200 OK`
* `201 Created`
* `204 No Content`
* `400 Bad Request`
* `401 Unauthorized`
* `403 Forbidden`
* `404 Not Found`
* `409 Conflict`
* `422 Unprocessable Content`
* `429 Too Many Requests`
* `500 Internal Server Error`
* `503 Service Unavailable`

Use status codes consistently.

Do not return `200 OK` for every response with an error inside the body:

```json
{
  "success": false,
  "error": "not found"
}
```

Better:

```http
404 Not Found
```

with a clear error body.

Status codes are part of the API.

---

# Error Responses

Error responses should be predictable.

Bad:

```json
{
  "message": "bad"
}
```

Better:

```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Email address is invalid.",
    "field": "email"
  }
}
```

Good error APIs include:

* stable error code
* human-readable message
* field information when relevant
* request ID for support
* safe details

Do not expose internal tracebacks to public clients.

Log internal details server-side with a request ID.

Return safe, useful errors to clients.

---

# Pagination

APIs that return lists need pagination.

Bad:

```http
GET /orders
```

returning every order forever.

Better:

```http
GET /orders?limit=50&cursor=abc123
```

Response:

```json
{
  "items": [...],
  "next_cursor": "def456"
}
```

Pagination protects:

* server memory
* database performance
* network bandwidth
* client reliability

Design pagination early.

Retrofitting it after clients depend on unbounded lists is painful.

---

# Filtering and Sorting

List APIs often need filtering and sorting.

Example:

```http
GET /orders?status=paid&sort=created_at&direction=desc
```

Define allowed filters.

Define allowed sort fields.

Validate inputs.

Do not pass raw query parameters directly into SQL or ORM order clauses.

Filtering and sorting are API features and security boundaries.

They need tests.

---

# Versioning

APIs change.

Versioning defines how change is managed.

For packages, versions may look like:

```text
1.4.2
```

For HTTP APIs, versions may appear in:

```text
/v1/orders
```

or headers.

The exact strategy depends on the system.

The important question is:

```text
how do clients know what contract they are using?
```

Not every change needs a new version.

Adding an optional response field is often backward compatible.

Removing a field usually is not.

Changing meaning is often worse than changing shape.

Versioning is about trust.

---

# Backward Compatibility

A backward-compatible change does not break existing clients.

Usually compatible:

* adding optional fields
* adding new endpoints
* accepting new optional request fields
* adding new enum values only if clients tolerate them

Usually breaking:

* removing fields
* renaming fields
* changing field types
* changing required parameters
* changing error codes
* changing semantics
* making optional fields required

Be careful with enum-like values.

Clients may assume they know all possible values.

If you add a new status, old clients may fail unless designed defensively.

Compatibility is not only syntax.

It is behavior.

---

# Deprecation

Deprecation is a warning that an API will be removed or changed later.

A good deprecation includes:

* what is deprecated
* what to use instead
* when removal may happen
* how to migrate

In Python:

```python
import warnings


def old_function():
    warnings.warn(
        "old_function is deprecated; use new_function instead",
        DeprecationWarning,
        stacklevel=2,
    )
```

For HTTP APIs, deprecation may use documentation, headers, dashboards, and client communication.

Do not remove public APIs abruptly unless you control all callers and have migrated them.

Deprecation is API empathy.

---

# Documentation

APIs need documentation.

Good API docs answer:

* what does this do?
* how do I call it?
* what inputs are valid?
* what output should I expect?
* what errors can happen?
* what examples show normal use?
* what behavior is guaranteed?
* what is intentionally unspecified?

Type hints help.

Docstrings help.

Examples help most.

Example:

```python
def refund(payment_id: PaymentId, amount: Money) -> Refund:
    """Refund part or all of a captured payment.

    Raises PaymentNotCaptured if the payment has not been captured.
    Raises RefundLimitExceeded if amount exceeds the refundable balance.
    """
```

Documentation should describe behavior that types cannot express.

---

# API Examples

Examples are part of API design.

A good example shows the intended path.

Python example:

```python
client = PaymentsClient(api_key)
payment = client.charge(order_id="ord_123", amount=Money("100.00", "INR"))
```

HTTP example:

```http
POST /payments
Content-Type: application/json

{
  "order_id": "ord_123",
  "amount": "100.00",
  "currency": "INR"
}
```

Response:

```json
{
  "id": "pay_456",
  "status": "captured"
}
```

Examples reduce interpretation.

If you cannot write a simple example, the API may be too confusing.

---

# API Testing

APIs need tests at the contract boundary.

For function APIs:

```python
def test_find_user_returns_none_when_missing():
    assert users.find_by_email("missing@example.com") is None
```

For HTTP APIs:

```python
def test_create_user_returns_201(client):
    response = client.post("/users", json={"email": "a@example.com"})

    assert response.status_code == 201
    assert response.json()["email"] == "a@example.com"
```

For CLI APIs:

```python
def test_cli_json_output():
    result = run_cli(["status", "--json"])

    assert result.returncode == 0
    assert json.loads(result.stdout)["status"] == "ok"
```

Tests protect the contract.

They should focus on what callers rely on.

---

# Contract Tests

Contract tests verify that an implementation satisfies an API contract.

Example storage contract:

```python
def storage_contract(storage: Storage) -> None:
    storage.save("hello.txt", b"hello")
    assert storage.load("hello.txt") == b"hello"
```

Run it against:

* in-memory storage
* local filesystem storage
* cloud storage adapter

Contract tests are useful when multiple implementations share an API.

They connect to LSP from Chapter 81.

If implementations cannot pass the same contract tests, they are not truly substitutable.

---

# Schema

Some APIs use schemas.

Schemas define data shape.

Examples:

* JSON Schema
* OpenAPI schemas
* Protocol Buffers
* Avro
* Pydantic models
* dataclass-based schemas

Schema benefits:

* validation
* documentation
* code generation
* compatibility checks
* clearer contracts

But schemas must match reality.

A stale schema is worse than no schema because it lies confidently.

Keep schemas close to tests and implementation.

---

# OpenAPI

OpenAPI is a common specification format for HTTP APIs.

It can describe:

* endpoints
* methods
* parameters
* request bodies
* response bodies
* status codes
* authentication
* schemas

Frameworks such as FastAPI can generate OpenAPI documents from Python code and type hints.

Generated documentation is helpful.

But generation does not remove design responsibility.

You still must decide:

* resource names
* error shapes
* versioning
* pagination
* compatibility
* security

Tools can document an API.

They cannot make it good by themselves.

---

# Internal APIs

Internal APIs are used inside one organization or codebase.

They still need care.

The fact that callers are internal does not mean changes are free.

Internal APIs can have many consumers.

Examples:

* shared package APIs
* internal HTTP services
* event topics
* database views
* CLI tools used by automation

Breaking internal APIs can break deployments, jobs, reports, and teams.

Use compatibility discipline where dependency exists.

Internal does not mean informal.

It means the audience is known.

---

# Event APIs

Events are APIs.

Example:

```json
{
  "type": "OrderPaid",
  "version": 1,
  "order_id": "ord_123",
  "paid_at": "2026-01-01T10:00:00Z"
}
```

Consumers depend on:

* event type
* field names
* field meanings
* ordering expectations
* delivery guarantees
* version
* idempotency behavior

Event APIs are often harder to change than request/response APIs because many consumers may process events asynchronously.

Design event payloads carefully.

Include event IDs.

Include timestamps.

Version important event schemas.

Make consumers idempotent when messages may be redelivered.

---

# API Security

APIs are security boundaries.

Important questions:

* who can call this?
* how are callers authenticated?
* what are they authorized to do?
* what input is trusted?
* what output may reveal data?
* are rate limits needed?
* are errors safe?
* are logs safe?
* is tenant isolation enforced?

Never rely on client-side checks alone.

Backend APIs must enforce authorization.

Example:

```python
def get_invoice(current_user: User, invoice_id: InvoiceId) -> Invoice:
    invoice = invoices.get(invoice_id)
    if invoice.tenant_id != current_user.tenant_id:
        raise Forbidden()
    return invoice
```

Security is part of the API contract.

---

# Rate Limits

Public or shared APIs may need rate limits.

Rate limits protect:

* service availability
* databases
* external providers
* fairness between clients
* cost
* security

HTTP APIs commonly return:

```http
429 Too Many Requests
```

and may include headers describing limits.

Rate limit behavior should be documented.

Clients need to know whether to retry, back off, or fail.

Rate limiting is not only infrastructure.

It affects API design.

---

# API Observability

APIs should be observable.

For HTTP APIs, log:

* request ID
* method
* path or route template
* status code
* duration
* authenticated subject when safe
* error type

For function or service APIs, record important domain events and failures.

For event APIs, record publish and consume outcomes.

Observability lets teams answer:

```text
who called this API?
how often?
how slow?
how many failed?
which clients are affected?
```

An API without observability is difficult to operate.

---

# API Performance

API design affects performance.

Examples:

* unbounded list endpoints
* chatty APIs requiring many calls
* responses with too much data
* no filtering
* no pagination
* expensive synchronous work
* poor caching semantics
* no batch endpoints where batching is needed

Bad:

```text
client calls GET /users/{id} 1,000 times
```

Maybe better:

```http
POST /users/batch
```

or:

```http
GET /users?ids=u1,u2,u3
```

Do not design only for elegance of one call.

Design for real usage patterns.

---

# API Evolution

APIs should be designed to evolve.

Helpful practices:

* add fields instead of changing existing fields
* avoid exposing internal database IDs when inappropriate
* use stable error codes
* reserve room for pagination
* version event schemas
* document deprecations
* tolerate unknown fields where possible
* avoid making clients depend on field order
* make enum handling forward-compatible when possible

Evolution is easiest when the first version leaves room.

But do not overdesign endlessly.

Create enough stability for the known clients and likely change.

---

# API Design Smells

Common API smells include:

* vague names like `process`
* too many positional parameters
* mixed return types
* hidden side effects
* generic exceptions
* raw dictionaries everywhere
* no documented errors
* unbounded list responses
* inconsistent field names
* changing public behavior silently
* leaking database schema
* exposing provider-specific details
* returning `200` for errors
* no request IDs
* no contract tests
* public callers using internal modules

Smells are invitations to inspect the contract.

They do not always require a rewrite.

They do require attention.

---

# API Design Checklist

When designing an API, ask:

* Who are the callers?
* What behavior is promised?
* What inputs are valid?
* What outputs are stable?
* What errors can happen?
* What side effects occur?
* Is the operation idempotent?
* How is it authenticated and authorized?
* How will it be tested?
* How will it be documented?
* How will it evolve?
* What is private?
* What is public?
* What observability exists?
* What would break existing callers?

The checklist is not bureaucracy.

It is contract thinking.

---

# Chapter Summary

APIs are contracts between software components.

They exist at many levels: functions, classes, modules, packages, CLIs, HTTP services, events, and internal services.

Good APIs make expectations clear.

They define inputs, outputs, errors, side effects, stability, and behavior.

Public APIs require more care than private helpers because callers depend on them.

Small API surface area is easier to maintain.

Function and class APIs should use clear names, parameters, return values, exceptions, and side-effect boundaries.

Module and package APIs should expose intentional public names and hide internals by convention.

CLI APIs include commands, arguments, flags, exit codes, stdout, stderr, and machine-readable output.

HTTP APIs include methods, paths, status codes, request bodies, response bodies, authentication, pagination, filtering, errors, rate limits, and versioning.

Event APIs include event names, schemas, versions, IDs, timestamps, delivery assumptions, and idempotency.

Backward compatibility matters whenever callers rely on behavior.

Deprecation gives callers a migration path.

Documentation and examples reduce guessing.

Contract tests protect API promises.

Security, observability, performance, and evolution are part of API design.

The central lesson is:

```text
an API is what another part of the system is allowed to rely on
```

Design that contract deliberately.

---

# Exercises

1. Pick one function and write down its full API: parameters, return value, exceptions, side effects, and mutation behavior.

2. Refactor a function with too many positional parameters into keyword arguments or a command dataclass.

3. Identify a public module API and add `__all__` to make it explicit.

4. Design a CLI command with stable JSON output and meaningful exit codes.

5. Design an HTTP error response format with stable error codes.

6. Add pagination to an unbounded list endpoint design.

7. Write contract tests for two implementations of the same interface.

8. Write a deprecation warning for an old Python function.

9. Design an event payload for `OrderPaid` with version, ID, timestamp, and domain fields.

10. Review one API and list which changes would be backward compatible and which would be breaking.

---

# Preview of Chapter 84

Chapter 83 studied APIs.

We saw that APIs are contracts at every level of software: functions, classes, modules, packages, CLIs, HTTP services, internal services, and events.

Next we study microservices.

Microservices are an architectural style where a system is split into independently deployable services that communicate over APIs.

Microservices build on everything from this volume:

* testing
* logging
* debugging
* packaging
* type hints
* profiling
* design patterns
* SOLID
* architecture
* API design

They are powerful when independent deployment, scaling, ownership, and fault isolation matter.

They are costly when used before those needs are real.

The transition is:

```text
APIs define contracts
microservices distribute those contracts across processes and teams
```

Chapter 84 will explain microservices honestly: where they help, what they cost, and how Python systems should approach them without romanticizing distribution.
