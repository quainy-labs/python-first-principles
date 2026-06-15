# Capstone 04 - URL Shortener

## Project Brief

In this project, you will build a URL shortener: a small web service that accepts a long URL, creates a short code, stores the mapping, redirects visitors from the short URL to the original URL, and records basic click analytics.

The project takes the general REST API skills from Capstone 03 and turns them into a product-shaped service. It teaches redirects, unique identifier generation, collision handling, URL validation, persistence, analytics counters, HTTP semantics, abuse-aware design, tests, and operational thinking.

By the end, the reader should have a working short-link service with an API for creating and inspecting links, a redirect endpoint for users, SQLite persistence, predictable short-code behavior, and tests that protect the core edge cases.

---

# Why This Project Matters

A URL shortener looks simple.

Input:

```text
https://example.com/a/very/long/path
```

Output:

```text
https://short.local/x7Kp2a
```

That seems like one dictionary:

```python
codes["x7Kp2a"] = long_url
```

But real design questions appear quickly:

* How is the short code generated?
* What if two generated codes collide?
* Should users be allowed to choose custom codes?
* What is a valid URL?
* Should the service accept `javascript:` URLs?
* What status code should the redirect use?
* Should click counts be recorded?
* Should bots count as clicks?
* What happens when a code does not exist?
* Can short links be disabled?
* Should the API reveal all links?
* How should tests avoid randomness?

This is why the project matters.

It is small enough to build, but rich enough to teach real API design.

The central lesson is:

```text
small services become real systems when edge cases matter
```

---

# What You Will Build

You will build a service named `shorty`.

It will support:

```text
create a short link
create a short link with a custom code
redirect a short code to the original URL
inspect link details
list links
disable a link
record click counts
record last accessed time
return clear errors for invalid URLs and unknown codes
```

The core HTTP endpoints:

```text
GET     /health
POST    /links
GET     /links
GET     /links/{code}
POST    /links/{code}/disable
GET     /{code}
```

The last endpoint is the public redirect endpoint.

For example:

```text
GET /x7Kp2a
```

should redirect to the original long URL.

This capstone should use FastAPI and SQLite, continuing from Capstone 03.

---

# Example API Usage

Create a short link:

```bash
curl -X POST http://localhost:8000/links \
  -H "Content-Type: application/json" \
  -d '{"url": "https://www.python.org/doc/"}'
```

Response:

```json
{
  "code": "x7Kp2a",
  "url": "https://www.python.org/doc/",
  "short_url": "http://localhost:8000/x7Kp2a",
  "clicks": 0,
  "disabled": false,
  "created_at": "2026-06-15T10:00:00+00:00",
  "last_accessed_at": null
}
```

Redirect:

```bash
curl -i http://localhost:8000/x7Kp2a
```

Response:

```text
HTTP/1.1 307 Temporary Redirect
location: https://www.python.org/doc/
```

Inspect:

```bash
curl http://localhost:8000/links/x7Kp2a
```

Response:

```json
{
  "code": "x7Kp2a",
  "url": "https://www.python.org/doc/",
  "short_url": "http://localhost:8000/x7Kp2a",
  "clicks": 1,
  "disabled": false,
  "created_at": "2026-06-15T10:00:00+00:00",
  "last_accessed_at": "2026-06-15T10:05:00+00:00"
}
```

The user-facing behavior is simple.

The engineering underneath is the point.

---

# Requirements

The URL Shortener must:

* create short links for valid HTTP and HTTPS URLs
* reject unsupported URL schemes
* generate short codes automatically
* support custom codes
* reject duplicate custom codes
* handle generated-code collisions
* redirect existing enabled codes
* return `404` for unknown codes
* return `410 Gone` for disabled codes
* increment click count on redirect
* store last accessed time
* persist data in SQLite
* test service, repository, and API behavior

It should support:

```text
POST /links
GET /links
GET /links/{code}
POST /links/{code}/disable
GET /{code}
```

The API should return structured JSON for API endpoints.

The redirect endpoint should return an actual redirect response.

---

# Non-Requirements

This capstone will not include:

* user accounts
* authentication
* per-user link ownership
* payments
* public analytics dashboards
* browser UI
* QR codes
* Redis caching
* rate limiting
* custom domains
* distributed ID generation
* production deployment

Those are realistic extensions.

They are not necessary for the core learning goal.

This project should stay focused on the link lifecycle.

---

# Project Structure

A clean structure:

```text
url-shortener/
    pyproject.toml
    README.md
    src/
        shorty/
            __init__.py
            main.py
            api.py
            schemas.py
            model.py
            repository.py
            service.py
            database.py
            codes.py
            errors.py
    tests/
        test_codes.py
        test_repository.py
        test_service.py
        test_api.py
```

Responsibilities:

`codes.py` generates and validates short codes.

`model.py` defines the link domain object.

`schemas.py` defines API request and response models.

`repository.py` owns SQL.

`service.py` owns application rules.

`api.py` owns HTTP routes.

`database.py` owns SQLite setup.

`errors.py` owns application-specific exceptions.

This keeps one of the trickiest parts clear:

```text
code generation is application logic, not route logic
```

---

# Data Model

A short link needs:

* code
* original URL
* click count
* disabled flag
* created timestamp
* last accessed timestamp

Domain object:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class ShortLink:
    code: str
    url: str
    clicks: int
    disabled: bool
    created_at: str
    last_accessed_at: str | None
```

The short code is the public identifier.

Unlike the notes API, this project does not need to expose an integer ID.

The code itself is the resource identifier:

```text
/links/x7Kp2a
```

This design is product-shaped.

The URL path contains the thing users care about.

---

# SQLite Schema

Schema:

```sql
CREATE TABLE IF NOT EXISTS links (
    code TEXT PRIMARY KEY,
    url TEXT NOT NULL,
    clicks INTEGER NOT NULL DEFAULT 0,
    disabled INTEGER NOT NULL DEFAULT 0,
    created_at TEXT NOT NULL,
    last_accessed_at TEXT
);
```

Why is `code` the primary key?

Because codes must be unique and are used for lookup.

Redirect needs to be fast:

```sql
SELECT * FROM links WHERE code = ?
```

This is the central query.

The schema should reflect the access pattern.

---

# URL Validation

The service should accept only `http` and `https` URLs.

Valid:

```text
https://www.python.org/
http://example.com/path
```

Invalid:

```text
javascript:alert(1)
file:///etc/passwd
ftp://example.com/file
not a url
```

Why reject non-HTTP schemes?

Because a URL shortener redirects users.

Redirecting to dangerous or unexpected schemes can be harmful.

Use standard parsing:

```python
from urllib.parse import urlparse


def validate_url(url: str) -> str:
    cleaned = url.strip()
    parsed = urlparse(cleaned)
    if parsed.scheme not in {"http", "https"}:
        raise InvalidURL("URL must use http or https")
    if not parsed.netloc:
        raise InvalidURL("URL must include a host")
    return cleaned
```

This is not perfect URL security.

It is a reasonable first boundary.

---

# Short Code Design

A short code should be:

* short
* URL-safe
* case-sensitive or case-insensitive by design
* hard enough to guess for casual use
* validated before storage

Use an alphabet:

```python
ALPHABET = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
```

A six-character code gives:

```text
62 ** 6 = 56,800,235,584 possible codes
```

That is more than enough for this capstone.

Code generation:

```python
import secrets


def generate_code(length: int = 6) -> str:
    return "".join(secrets.choice(ALPHABET) for _ in range(length))
```

Use `secrets`, not `random`.

Even though this is not password generation, public short codes should not be trivially predictable.

This reinforces the security chapter.

---

# Custom Codes

Users may request custom codes:

```json
{
  "url": "https://www.python.org/doc/",
  "custom_code": "python-docs"
}
```

This should create:

```text
http://localhost:8000/python-docs
```

Custom code rules:

```text
length between 3 and 32
letters, numbers, hyphen, underscore only
cannot already exist
cannot be reserved
```

Reserved words:

```text
health
links
docs
openapi.json
```

Why reserve words?

Because the redirect endpoint lives at:

```text
/{code}
```

If someone creates code `links`, it conflicts with:

```text
/links
```

Routing design and code validation must agree.

---

# Collision Handling

Generated codes can collide.

It is unlikely.

It is still possible.

The service must handle it.

Algorithm:

```text
repeat up to N attempts:
    generate code
    if code does not exist:
        use it
if all attempts collide:
    raise CodeGenerationFailed
```

Example:

```python
def allocate_code(repository, generator, max_attempts=10) -> str:
    for _ in range(max_attempts):
        code = generator()
        if not repository.exists(code):
            return code
    raise CodeGenerationFailed("could not allocate unique code")
```

Tests should force a collision.

Do not rely on random chance.

Pass a fake generator in tests:

```python
codes = iter(["abc123", "abc123", "xyz789"])

def fake_generator():
    return next(codes)
```

This teaches deterministic testing around randomness.

---

# Redirect Status Code

Which redirect status should the service use?

Common options:

```text
301 Moved Permanently
302 Found
307 Temporary Redirect
308 Permanent Redirect
```

For this capstone, use:

```text
307 Temporary Redirect
```

Why?

It avoids promising that the redirect is permanent.

It also preserves the HTTP method if non-GET requests are ever involved.

In practice, short links are often GET requests, but choosing deliberately matters.

FastAPI response:

```python
from fastapi.responses import RedirectResponse


return RedirectResponse(url=link.url, status_code=307)
```

HTTP semantics are part of API design.

---

# Click Tracking

When a code is redirected, increment its click count.

Also update `last_accessed_at`.

This should happen before returning the redirect response.

Repository method:

```python
def record_click(self, code: str, now: str) -> ShortLink | None:
    ...
```

The method should:

```text
find the link
increment clicks
set last_accessed_at
return updated link
```

There is a subtle issue:

```text
read clicks -> add 1 -> write clicks
```

can lose updates under concurrency.

For this capstone, use a single SQL update:

```sql
UPDATE links
SET clicks = clicks + 1,
    last_accessed_at = ?
WHERE code = ?
```

This is safer than incrementing in Python.

It also teaches that databases can perform state changes atomically.

---

# Disabled Links

A link can be disabled.

Disabled means:

```text
the mapping still exists
but redirect should no longer happen
```

Why not delete it?

Because keeping the record preserves analytics and prevents accidental code reuse.

Redirect behavior:

```text
unknown code -> 404 Not Found
disabled code -> 410 Gone
```

`410 Gone` is more specific than `404`.

It says the resource used to exist but is no longer available.

This teaches expressive HTTP status codes.

---

# API Schemas

Create request:

```python
from pydantic import BaseModel, Field


class CreateLinkRequest(BaseModel):
    url: str = Field(min_length=1, max_length=2048)
    custom_code: str | None = Field(default=None, min_length=3, max_length=32)
```

Response:

```python
class LinkResponse(BaseModel):
    code: str
    url: str
    short_url: str
    clicks: int
    disabled: bool
    created_at: str
    last_accessed_at: str | None
```

The response includes `short_url`, even though it is derived.

Why?

Because it is useful for clients.

APIs should sometimes include derived fields when they improve usability and do not expose internals.

---

# Building `short_url`

The API needs a base URL.

In development:

```text
http://localhost:8000
```

In production:

```text
https://short.example.com
```

Do not hard-code this deeply.

Use configuration:

```python
BASE_URL = "http://localhost:8000"
```

or read from environment:

```python
base_url = os.environ.get("SHORTY_BASE_URL", "http://localhost:8000")
```

Then:

```python
short_url = f"{base_url.rstrip('/')}/{link.code}"
```

Configuration turns deployment details into explicit settings.

---

# Repository Layer

Repository methods:

```python
class LinkRepository:
    def create(self, link: ShortLink) -> ShortLink:
        ...

    def exists(self, code: str) -> bool:
        ...

    def get(self, code: str) -> ShortLink | None:
        ...

    def list(self, limit: int, offset: int) -> list[ShortLink]:
        ...

    def disable(self, code: str) -> ShortLink | None:
        ...

    def record_click(self, code: str, now: str) -> ShortLink | None:
        ...
```

The repository owns SQL.

It converts database rows into `ShortLink`.

It should not know about FastAPI responses.

---

# Service Layer

Service responsibilities:

* validate URL
* validate custom code
* allocate generated code
* reject duplicate custom code
* create links
* inspect links
* disable links
* redirect links
* record clicks

Example:

```python
class LinkService:
    def create_link(self, url: str, custom_code: str | None = None) -> ShortLink:
        url = validate_url(url)
        now = self.clock()

        if custom_code is not None:
            code = validate_custom_code(custom_code)
            if self.repository.exists(code):
                raise CodeAlreadyExists(f"code already exists: {code}")
        else:
            code = allocate_code(self.repository, self.code_generator)

        link = ShortLink(
            code=code,
            url=url,
            clicks=0,
            disabled=False,
            created_at=now,
            last_accessed_at=None,
        )
        return self.repository.create(link)
```

The service is where the product rules live.

---

# Error Types

Define errors:

```python
class ShortyError(Exception):
    pass


class InvalidURL(ShortyError):
    pass


class InvalidCode(ShortyError):
    pass


class CodeAlreadyExists(ShortyError):
    pass


class CodeNotFound(ShortyError):
    pass


class LinkDisabled(ShortyError):
    pass


class CodeGenerationFailed(ShortyError):
    pass
```

Translate them:

```text
InvalidURL -> 400
InvalidCode -> 400
CodeAlreadyExists -> 409
CodeNotFound -> 404
LinkDisabled -> 410
CodeGenerationFailed -> 503
```

`409 Conflict` is appropriate for duplicate custom codes.

`503 Service Unavailable` can be reasonable if the service cannot allocate a code after repeated attempts.

The exact choices should be documented.

---

# Route Design

Create link:

```text
POST /links
```

List links:

```text
GET /links?limit=20&offset=0
```

Inspect:

```text
GET /links/{code}
```

Disable:

```text
POST /links/{code}/disable
```

Redirect:

```text
GET /{code}
```

The redirect route should be registered carefully.

Because `/{code}` is broad, make sure more specific routes such as `/links` and `/health` are defined clearly.

Framework route matching rules matter.

This is another reason to reserve words like `links` and `health`.

---

# Testing Code Generation

Tests should not depend on real randomness.

Inject a generator.

Example:

```python
def test_generated_code_collision_is_retried(repository):
    repository.create(existing_link(code="abc123"))

    generated = iter(["abc123", "xyz789"])

    service = LinkService(
        repository=repository,
        code_generator=lambda: next(generated),
        clock=fake_clock,
    )

    link = service.create_link("https://example.com")

    assert link.code == "xyz789"
```

This test proves collision handling.

It does not wait for a collision to happen by luck.

That is professional testing.

---

# Testing Redirects

API tests should verify redirect behavior.

With an HTTP test client, redirects may be followed automatically.

Disable that when testing status and location headers.

Example:

```python
response = client.get("/abc123", follow_redirects=False)

assert response.status_code == 307
assert response.headers["location"] == "https://example.com"
```

Also verify click count:

```python
client.get("/abc123", follow_redirects=False)
details = client.get("/links/abc123").json()

assert details["clicks"] == 1
```

Redirects are behavior.

Test them as behavior.

---

# Pagination

List links should support pagination:

```text
GET /links?limit=20&offset=0
```

Rules:

```text
limit default 20
limit max 100
offset default 0
offset cannot be negative
```

This repeats the REST API capstone lesson.

List endpoints should not return unlimited data forever.

Even if the project is small, the habit matters.

---

# Abuse and Safety Notes

URL shorteners can be abused.

They can hide malicious destinations.

They can be used for spam.

They can redirect users to phishing pages.

This capstone does not build a complete abuse-prevention system.

But it should teach awareness.

Possible controls:

* restrict URL schemes
* reject internal/private hosts in stricter versions
* rate limit creation
* scan suspicious URLs
* allow disabling links
* log creation and redirect events
* avoid open admin listing in production

For this learning project, validation and disable behavior are enough.

But readers should understand that a real public shortener needs more protection.

---

# Analytics

This capstone tracks basic analytics:

```text
click count
last accessed time
```

Do not track IP addresses or user agents in the core project.

Why?

Because analytics can become privacy-sensitive quickly.

The first version should teach the concept without collecting unnecessary data.

A later extension can add click events:

```text
code
timestamp
referrer
user agent
country
```

But that requires privacy and retention decisions.

Data collection should be intentional.

---

# Milestone 1 - Skeleton

Build:

* FastAPI app
* health endpoint
* SQLite connection helper
* schema initialization

Completion criteria:

```text
The app runs and GET /health returns {"status": "ok"}.
```

This proves the service can start.

---

# Milestone 2 - Code Utilities

Build:

* code generator
* custom code validator
* reserved word detection
* URL validator

Tests:

* generated code length
* generated code alphabet
* custom code accepts safe characters
* custom code rejects spaces
* reserved codes fail
* invalid URL schemes fail

Completion criteria:

```text
Inputs are validated before storage.
```

This milestone teaches boundaries.

---

# Milestone 3 - Repository

Build:

* create link
* get link
* exists check
* list links
* disable link
* record click

Tests:

* create and get
* exists true and false
* duplicate code fails at database level
* record click increments in SQL
* disable changes disabled flag

Completion criteria:

```text
The database layer supports the short-link lifecycle.
```

---

# Milestone 4 - Service

Build:

* create with generated code
* create with custom code
* collision retry
* duplicate custom code error
* redirect lookup
* disabled link behavior
* click recording

Tests:

* generated creation
* custom creation
* duplicate custom code
* collision retry
* unknown code
* disabled code
* click count update

Completion criteria:

```text
Product rules work without HTTP.
```

---

# Milestone 5 - API

Build:

* `POST /links`
* `GET /links`
* `GET /links/{code}`
* `POST /links/{code}/disable`
* `GET /{code}`
* error translation

Tests:

* create returns 201
* duplicate custom code returns 409
* invalid URL returns 400
* inspect returns link details
* redirect returns 307
* unknown redirect returns 404
* disabled redirect returns 410

Completion criteria:

```text
The HTTP API exposes shortener behavior correctly.
```

---

# Milestone 6 - Documentation and Packaging

Build:

* README with curl examples
* project dependencies
* local run command
* test command
* configuration notes

Completion criteria:

```text
Another developer can run, test, and use the service locally.
```

This milestone teaches project handoff.

---

# Common Mistakes

The first common mistake is using predictable codes.

Use `secrets`, not simple incrementing IDs, for public codes.

The second common mistake is ignoring collisions because they are unlikely.

Unlikely is not impossible.

The third common mistake is accepting any URL scheme.

Do not redirect users to arbitrary schemes.

The fourth common mistake is returning `200` with a JSON body for redirects.

A shortener should use real HTTP redirects.

The fifth common mistake is overwriting custom codes.

Duplicate custom codes should return `409 Conflict`.

The sixth common mistake is deleting disabled links.

Disable preserves history and prevents accidental code reuse.

The seventh common mistake is testing randomness directly.

Inject a fake generator.

---

# Extension Ideas

After the core works, add:

* expiration dates
* QR code generation
* click event table
* admin dashboard
* authentication
* per-user links
* custom domains
* rate limiting
* spam checks
* private IP blocking
* Redis cache for redirects
* Dockerfile
* deployment guide

Each extension should preserve the core behavior:

```text
valid URL -> unique code -> redirect -> analytics
```

---

# What This Capstone Teaches

This capstone teaches product-shaped API design.

It connects:

```text
REST APIs
redirects
status codes
URL validation
short-code generation
collision handling
SQLite persistence
analytics counters
service layers
repository layers
security awareness
tests
```

The central lesson is:

```text
simple web services still need careful product rules
```

A URL shortener is not just a dictionary.

It is a public contract with redirects, identifiers, edge cases, and abuse risks.

---

# Completion Checklist

The capstone is complete when:

```text
The app has a health endpoint.
POST /links creates a short link.
Generated codes use a safe alphabet.
Generated-code collisions are retried.
Custom codes are validated.
Duplicate custom codes return 409.
Invalid URLs return 400.
GET /{code} redirects with 307.
Unknown codes return 404.
Disabled codes return 410.
Redirect increments click count.
Redirect updates last accessed time.
GET /links/{code} returns details.
GET /links supports pagination.
SQLite persistence works.
SQL uses parameters.
Service logic is tested.
Repository logic is tested.
API behavior is tested.
Randomness is injectable in tests.
README includes curl examples.
```

If all of these are true, the reader has built a real product-shaped web service.

---

# Exercises

1. Create the project structure.

2. Add the FastAPI app and health endpoint.

3. Create the SQLite schema.

4. Define the `ShortLink` domain model.

5. Write the URL validator.

6. Write the short-code generator using `secrets`.

7. Write the custom-code validator.

8. Add reserved word checks.

9. Implement repository create/get/exists methods.

10. Implement repository list, disable, and record-click methods.

11. Write repository tests.

12. Implement service create behavior.

13. Test generated-code collision retry.

14. Implement redirect behavior.

15. Add API routes.

16. Test redirect status and location header.

17. Test click count after redirect.

18. Test invalid URL and duplicate custom code errors.

19. Add pagination to `GET /links`.

20. Document local usage with curl examples.

---

# Preview of Capstone 05

Capstone 04 built a URL Shortener.

It connected REST APIs, redirects, short-code generation, collision handling, URL validation, SQLite persistence, analytics counters, service design, repository design, tests, and abuse-aware thinking.

Capstone 05 will build an ORM.

The ORM will move deeper into how Python objects can map to database tables, using descriptors, metaclasses or class decorators, query construction, model fields, persistence, and SQL generation.

The transition is:

```text
URL Shortener uses database persistence through a repository
ORM explores how persistence frameworks are built
```

The next project will connect the Python data model with database abstraction.
