# Chapter 98 — APIs and Automation

Python is one of the best languages for automation.

That is not an accident.

Python has a rare combination of qualities:

* readable syntax
* strong standard library
* excellent HTTP libraries
* good file handling
* good data handling
* easy scripting
* strong ecosystem integrations
* cross-platform support
* enough structure for production systems

Automation means using software to perform repeatable work.

Integration means connecting systems so data and actions can move between them.

Together, APIs and automation let Python connect the world.

Examples:

* fetch data from a REST API
* send a Slack notification
* create a ticket
* sync customers between systems
* download reports every morning
* transform CSV files
* upload processed data to storage
* call an internal service
* respond to a webhook
* run scheduled jobs
* monitor an endpoint
* generate invoices
* enrich records from third-party APIs
* automate operational workflows

This chapter is about practical automation engineering.

It is not only about writing:

```python
requests.get(url)
```

That is the easy part.

The hard part is building integrations that handle authentication, errors, retries, rate limits, pagination, idempotency, logging, data validation, and operational failure.

---

# Part IV Opening

Part IV of Volume IV is about Automation and Integration.

The previous part focused on machine learning and AI engineering.

We studied models, RAG, and agents.

Now we step back into a broader software engineering reality:

```text
most useful systems need to connect to other systems
```

Even an AI system needs integration.

It may call APIs, read documents, write tickets, send emails, run jobs, or process files.

Automation is the connective tissue.

Python is widely used for this because it is good at glue code.

Glue code is not an insult.

Glue code is what makes separate systems work together.

The professional challenge is making glue code reliable.

This part begins with APIs and automation because they are fundamental to many career paths:

* backend engineering
* data engineering
* DevOps
* AI engineering
* security automation
* business operations
* platform engineering
* analytics engineering

The central idea is:

```text
automation becomes engineering when it is reliable, observable, and safe
```

---

# What Is an API?

API stands for Application Programming Interface.

An API is a contract that lets software interact with other software.

In this chapter, we mostly mean web APIs.

A web API lets a program communicate with a service over HTTP.

Example:

```text
GET https://api.example.com/customers/123
```

The request asks for customer `123`.

The service returns a response.

The response may contain JSON:

```json
{
  "id": "123",
  "name": "Anika",
  "status": "active"
}
```

The API contract defines:

* URL paths
* HTTP methods
* query parameters
* headers
* request body shape
* response body shape
* authentication
* status codes
* errors
* rate limits
* versioning

An API is not only a URL.

It is a set of promises between systems.

Automation depends on those promises.

---

# HTTP Basics

HTTP is the protocol behind most web APIs.

An HTTP request includes:

* method
* URL
* headers
* optional body

An HTTP response includes:

* status code
* headers
* optional body

Common methods:

* `GET`
* `POST`
* `PUT`
* `PATCH`
* `DELETE`

`GET` retrieves information.

`POST` creates a resource or triggers an action.

`PUT` usually replaces a resource.

`PATCH` partially updates a resource.

`DELETE` removes a resource.

Common status codes:

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
* `502 Bad Gateway`
* `503 Service Unavailable`
* `504 Gateway Timeout`

Good API clients treat status codes as control signals.

Do not parse the response body as if every response is success.

Check what happened first.

---

# JSON

JSON is the most common data format for web APIs.

JSON maps naturally to Python structures:

```text
JSON object -> Python dict
JSON array  -> Python list
JSON string -> Python str
JSON number -> Python int or float
JSON true   -> Python True
JSON false  -> Python False
JSON null   -> Python None
```

Python's standard library has `json`:

```python
import json

data = json.loads('{"name": "Anika", "active": true}')
text = json.dumps({"name": "Anika", "active": True})
```

HTTP libraries often provide response helpers:

```python
payload = response.json()
```

Be careful with untrusted JSON.

Very large or malicious JSON can consume memory or CPU.

In production systems, consider:

* response size limits
* timeouts
* schema validation
* defensive parsing
* clear error handling

JSON is simple.

Systems built around JSON are not always simple.

---

# Requests

`requests` is the classic Python HTTP client.

It is popular because it is simple and readable.

Example:

```python
import requests

response = requests.get("https://api.example.com/customers/123", timeout=10)
response.raise_for_status()
customer = response.json()
```

Important details:

* `timeout=10` prevents hanging forever
* `raise_for_status()` raises an exception for HTTP error status codes
* `response.json()` parses JSON response content

Do not omit timeouts in real automation.

Without timeouts, a script may hang indefinitely.

A small script that hangs may be annoying.

A production job that hangs may block a pipeline, consume workers, or delay business operations.

The professional habit is:

```text
every network call should have a timeout
```

---

# HTTPX

`httpx` is another Python HTTP client.

It has a requests-like API, but also supports async usage and HTTP/2.

Synchronous example:

```python
import httpx

response = httpx.get("https://api.example.com/customers/123", timeout=10)
response.raise_for_status()
customer = response.json()
```

Async example:

```python
import httpx


async def fetch_customer(customer_id: str):
    async with httpx.AsyncClient(timeout=10) as client:
        response = await client.get(f"https://api.example.com/customers/{customer_id}")
        response.raise_for_status()
        return response.json()
```

Use `requests` when simple synchronous HTTP is enough.

Use `httpx` when you need:

* async support
* HTTP/2 support
* stricter timeout behavior
* type annotations
* a modern client interface

Do not use async just because it feels advanced.

Use async when you are doing many I/O-bound calls and the surrounding application can benefit from concurrency.

---

# Sessions and Clients

For repeated requests, use a session or client.

With `requests`:

```python
import requests

with requests.Session() as session:
    session.headers.update({"Authorization": f"Bearer {token}"})

    response = session.get("https://api.example.com/customers", timeout=10)
    response.raise_for_status()
```

With `httpx`:

```python
import httpx

with httpx.Client(timeout=10) as client:
    response = client.get("https://api.example.com/customers")
    response.raise_for_status()
```

Clients can reuse connections.

They can also centralize:

* base headers
* authentication
* cookies
* timeout settings
* connection pooling
* event hooks

Repeated automation should not scatter raw HTTP calls everywhere.

Create a small API client wrapper when the integration becomes important.

---

# API Client Wrappers

A client wrapper gives your code a clean interface.

Instead of writing:

```python
requests.get(
    f"https://api.example.com/customers/{customer_id}",
    headers={"Authorization": f"Bearer {token}"},
    timeout=10,
)
```

everywhere, write:

```python
client.get_customer(customer_id)
```

Example:

```python
class CustomerApiClient:
    def __init__(self, base_url: str, token: str):
        self.base_url = base_url.rstrip("/")
        self.session = requests.Session()
        self.session.headers.update({"Authorization": f"Bearer {token}"})

    def get_customer(self, customer_id: str) -> dict:
        response = self.session.get(
            f"{self.base_url}/customers/{customer_id}",
            timeout=10,
        )
        response.raise_for_status()
        return response.json()
```

This concentrates API details in one place.

Benefits:

* easier testing
* consistent timeouts
* consistent authentication
* consistent error handling
* clearer business code
* easier API changes later

Small scripts can use direct calls.

Important automation deserves a client boundary.

---

# Authentication

APIs need authentication.

Common patterns:

* API keys
* bearer tokens
* Basic authentication
* OAuth 2
* signed requests
* mTLS
* session cookies

API key example:

```python
headers = {"X-API-Key": api_key}
```

Bearer token example:

```python
headers = {"Authorization": f"Bearer {token}"}
```

Never hard-code secrets in source code.

Use:

* environment variables
* secret managers
* deployment platform secrets
* local development `.env` files excluded from version control

Bad:

```python
API_KEY = "sk_live_secret_value"
```

Better:

```python
import os

api_key = os.environ["SERVICE_API_KEY"]
```

Secrets are not configuration trivia.

They are credentials.

Treat them like credentials.

---

# OAuth

OAuth is common for APIs that act on behalf of users.

Examples:

* Google APIs
* Microsoft APIs
* GitHub APIs
* Slack APIs
* payment systems
* CRM platforms

OAuth usually involves:

* client ID
* client secret
* authorization URL
* redirect URI
* authorization code
* access token
* refresh token
* scopes

Scopes define what the token can do.

For example:

```text
read:user
repo:status
calendar.readonly
```

Use the narrowest scopes possible.

Do not ask for write access when read access is enough.

OAuth automation should handle token refresh carefully.

Expired tokens are normal.

Refresh failures need clear error handling.

Store tokens securely.

---

# Query Parameters

Query parameters appear after `?` in a URL:

```text
https://api.example.com/customers?status=active&limit=100
```

Do not build query strings manually when the client can do it safely.

With `requests`:

```python
response = requests.get(
    "https://api.example.com/customers",
    params={"status": "active", "limit": 100},
    timeout=10,
)
```

This handles encoding.

Manual string building can break when values contain spaces, `&`, `?`, or non-ASCII characters.

Structured APIs exist for a reason.

Use them.

---

# Request Bodies

For JSON request bodies:

```python
payload = {
    "name": "Anika",
    "email": "anika@example.com",
}

response = requests.post(
    "https://api.example.com/customers",
    json=payload,
    timeout=10,
)
```

The `json=` argument serializes the payload and sets the appropriate content type.

For form data:

```python
response = requests.post(
    "https://api.example.com/login",
    data={"username": username, "password": password},
    timeout=10,
)
```

For files:

```python
with open("report.pdf", "rb") as file:
    response = requests.post(
        "https://api.example.com/upload",
        files={"file": file},
        timeout=30,
    )
```

Choose the body format the API expects.

Do not guess.

Read the API documentation.

---

# Status Codes and Errors

Automation should handle errors deliberately.

Basic pattern:

```python
response = requests.get(url, timeout=10)

try:
    response.raise_for_status()
except requests.HTTPError as exc:
    raise RuntimeError(f"API request failed: {response.status_code}") from exc
```

Different errors need different responses.

`400` usually means your request is wrong.

`401` means authentication failed.

`403` means permission denied.

`404` means resource not found.

`409` may mean conflict.

`429` means rate limited.

`500` and above usually mean server-side or upstream problems.

Do not retry every error.

Retrying a bad request will not make it good.

Retry transient failures.

Fix permanent failures.

---

# Timeouts

Timeouts are mandatory in serious automation.

There are different kinds of timeouts:

* connect timeout
* read timeout
* write timeout
* pool timeout
* total timeout

Simple:

```python
requests.get(url, timeout=10)
```

More explicit:

```python
requests.get(url, timeout=(3, 10))
```

This means:

```text
3 seconds to connect
10 seconds to read
```

HTTPX has strict timeout behavior and configurable timeout objects.

The exact values depend on the task.

A user-facing request may need a short timeout.

A background export job may tolerate longer.

Choose timeouts based on user experience and operational needs.

No timeout is rarely acceptable.

---

# Retries

Retries handle transient failures.

Examples:

* temporary network failure
* `502 Bad Gateway`
* `503 Service Unavailable`
* `504 Gateway Timeout`
* some `429 Too Many Requests` cases

Retries should use backoff.

Bad retry loop:

```python
for _ in range(100):
    call_api()
```

Better:

```text
try
wait a little
try again
wait longer
try again
stop after a limit
```

Backoff reduces pressure on a struggling service.

Jitter adds randomness to avoid many clients retrying at the same moment.

Retry design should consider idempotency.

Retrying a `GET` is usually safer than retrying a `POST` that creates a charge.

Retries without idempotency can duplicate actions.

---

# Idempotency

An operation is idempotent if repeating it has the same effect as doing it once.

Example:

```text
set customer's status to inactive
```

Doing this twice leaves the customer inactive.

Non-idempotent example:

```text
charge customer $100
```

Doing this twice charges $200.

Some APIs support idempotency keys.

Example:

```python
headers = {
    "Idempotency-Key": operation_id,
}
```

The server uses the key to recognize repeated attempts for the same operation.

This is essential for safe retries of create or payment-like operations.

Automation should identify which operations are safe to retry and which require protection.

This is not theoretical.

Many production incidents are duplicate-action incidents.

---

# Rate Limits

APIs often limit request rate.

Rate limits protect services from overload and abuse.

A rate-limited response often uses status code:

```text
429 Too Many Requests
```

The response may include headers such as:

```text
Retry-After
X-RateLimit-Remaining
X-RateLimit-Reset
```

Automation should respect rate limits.

Strategies:

* slow down requests
* use backoff
* batch requests
* cache results
* avoid unnecessary polling
* request higher limits when justified
* design jobs to resume

Do not write a loop that hammers an API until it blocks you.

Good automation is a polite client.

---

# Pagination

APIs often return data in pages.

Instead of returning one million records at once, the API returns a page at a time.

Pagination styles include:

* page number
* offset and limit
* cursor
* next URL

Example cursor loop:

```python
def iter_customers(client):
    cursor = None

    while True:
        params = {"limit": 100}
        if cursor is not None:
            params["cursor"] = cursor

        response = client.get("/customers", params=params)
        response.raise_for_status()
        payload = response.json()

        yield from payload["items"]

        cursor = payload.get("next_cursor")
        if cursor is None:
            break
```

Pagination bugs are common.

Watch for:

* infinite loops
* skipped records
* duplicated records
* changing data during pagination
* missing final page
* rate limits

Test pagination with multiple pages.

Do not only test the first page.

---

# Webhooks

Polling means repeatedly asking:

```text
has something changed?
```

Webhooks reverse the flow.

The service calls your endpoint when an event happens.

Examples:

* payment succeeded
* issue created
* user signed up
* file uploaded
* build finished
* order shipped

A webhook handler is an HTTP endpoint.

It receives event data.

It verifies authenticity.

It processes the event.

Webhook systems need:

* signature verification
* idempotency
* fast response
* background processing
* retry handling
* event logging
* duplicate detection
* schema versioning

Do not process long jobs directly inside the webhook request if the provider expects quick acknowledgement.

Accept the event, record it, and process asynchronously when appropriate.

---

# Scheduling

Automation often runs on a schedule.

Examples:

* every hour
* every day at midnight
* every Monday morning
* every five minutes
* after a file arrives

Scheduling options include:

* cron
* systemd timers
* cloud schedulers
* task queues
* workflow orchestrators
* CI scheduled workflows
* application-level schedulers

Simple cron idea:

```text
0 6 * * * run daily report at 06:00
```

Scheduling is easy to start and easy to under-design.

Ask:

* What happens if a job fails?
* What happens if a previous run is still running?
* What timezone is used?
* What happens during daylight saving changes?
* Where are logs?
* How are alerts sent?
* Can the job resume?
* Is the job idempotent?

Scheduled automation needs operations thinking.

---

# Files as Integration

Not every integration is an API.

Many real workflows use files:

* CSV
* Excel
* JSON
* XML
* Parquet
* PDFs
* ZIP archives
* logs

File automation may include:

* watching a folder
* downloading files
* validating file names
* parsing content
* transforming records
* writing output files
* uploading files
* archiving processed files
* moving failed files aside

Python's standard library is strong here:

```python
from pathlib import Path

input_dir = Path("incoming")

for path in input_dir.glob("*.csv"):
    print(path.name)
```

File workflows need careful design:

* avoid partial reads while a file is still being written
* validate schemas
* handle duplicate files
* handle bad rows
* archive inputs
* write outputs atomically
* log processing results

Files are simple until they become production interfaces.

Then they deserve contracts too.

---

# Data Validation

Automation moves data between systems.

Bad data can cause bad actions.

Validate inputs.

Validation may include:

* required fields
* data types
* allowed values
* numeric ranges
* date formats
* uniqueness
* referential integrity
* schema version
* business rules

Example:

```python
def validate_customer(record: dict) -> None:
    if not record.get("email"):
        raise ValueError("missing email")

    if record.get("status") not in {"active", "inactive"}:
        raise ValueError("invalid status")
```

For larger systems, use schema libraries or typed models.

Validation should happen at boundaries:

* after reading files
* after receiving API responses
* before sending API requests
* before writing database records
* before triggering external actions

Trust boundaries are validation boundaries.

---

# Logging

Automation needs logs.

Logs should answer:

* What ran?
* When did it run?
* What input did it process?
* How many records succeeded?
* How many failed?
* Which API calls failed?
* What was retried?
* How long did it take?
* What output was produced?

Use Python's `logging` module for real scripts.

```python
import logging

logger = logging.getLogger(__name__)

logger.info("starting customer sync")
logger.warning("skipping invalid record", extra={"customer_id": customer_id})
logger.exception("sync failed")
```

Avoid using only `print` in important automation.

`print` is fine for quick experiments.

Logging gives levels, timestamps, handlers, and integration with production systems.

Do not log secrets.

Do not log full sensitive payloads unless you have a safe reason and controls.

---

# Observability

Logging is part of observability.

Observability also includes:

* metrics
* traces
* alerts
* dashboards
* health checks
* audit logs

Useful automation metrics:

* run count
* success count
* failure count
* records processed
* API latency
* retry count
* rate-limit count
* queue depth
* job duration
* cost
* stale data age

Alerts should be actionable.

Bad alert:

```text
something failed
```

Better alert:

```text
customer sync failed 3 times; last successful run was 4 hours ago; 12,430 records pending
```

Automation without observability becomes invisible until someone complains.

Professional automation tells you when it is unhealthy.

---

# Configuration

Automation should be configurable.

Configuration may include:

* API base URLs
* timeouts
* retry limits
* credentials
* file paths
* batch sizes
* schedule settings
* feature flags
* environment names

Use environment variables or config files appropriately.

Example:

```python
import os

API_BASE_URL = os.environ.get("API_BASE_URL", "https://api.example.com")
TIMEOUT_SECONDS = float(os.environ.get("TIMEOUT_SECONDS", "10"))
```

Do not mix configuration with business logic.

Do not hard-code production endpoints into reusable scripts.

Do not commit secrets.

Good configuration makes the same code usable in development, staging, and production.

---

# Dry Runs

A dry run shows what would happen without making changes.

Dry runs are extremely useful for automation that modifies external systems.

Example:

```python
def sync_customer(customer, dry_run: bool = False):
    if dry_run:
        logger.info("would sync customer", extra={"customer_id": customer["id"]})
        return

    api.update_customer(customer)
```

Dry runs help with:

* testing
* review
* migration planning
* debugging
* stakeholder confidence

But dry runs must be honest.

If dry-run behavior differs too much from real behavior, it creates false confidence.

The best dry run exercises as much logic as possible while skipping only irreversible external actions.

---

# Idempotent Jobs

Scheduled automation should be idempotent when possible.

If a job fails halfway, you may need to run it again.

If rerunning creates duplicates, the system becomes fragile.

Idempotent design patterns:

* use stable operation IDs
* upsert instead of blind insert
* track processed records
* write checkpoints
* use idempotency keys
* make output filenames deterministic
* skip already processed inputs
* design rollback or compensation

Example:

```text
sync customer C123 for date 2026-06-15
```

can have a stable operation key:

```text
customer-sync:C123:2026-06-15
```

Idempotency is one of the biggest differences between a script and a reliable job.

---

# Checkpointing

Checkpointing records progress.

If a job processes 1,000,000 records and fails after 800,000, you do not want to start from zero if avoidable.

Checkpoint examples:

* last processed ID
* last timestamp
* page cursor
* file offset
* batch number
* completed operation IDs

Checkpointing must be designed carefully.

If the checkpoint is written too early, records may be skipped after failure.

If it is written too late, records may be repeated.

This is where idempotency helps.

If repeating is safe, checkpointing becomes less dangerous.

For important jobs, design failure and resume behavior explicitly.

---

# Queues

Queues decouple producers from workers.

Instead of doing work immediately, a system enqueues a job.

Workers process jobs from the queue.

Queues are useful for:

* background work
* retries
* load smoothing
* asynchronous processing
* webhook handling
* long-running tasks
* fan-out workflows

Queue systems include:

* Redis-backed queues
* RabbitMQ
* cloud queues
* task queues such as Celery or RQ
* workflow orchestrators

Queue jobs need:

* idempotency
* retry policy
* dead-letter handling
* visibility into failures
* payload validation
* timeouts

A queue is not a trash can for hard work.

It is a reliability boundary.

Design it carefully.

---

# Web Scraping

Sometimes no API exists.

People reach for web scraping.

Scraping means extracting data from web pages.

Scraping has risks:

* page structure changes
* legal or terms-of-service issues
* rate limiting
* blocked requests
* brittle selectors
* dynamic JavaScript rendering
* incomplete data
* robots.txt considerations

Prefer official APIs when available.

If scraping is necessary:

* respect website policies
* identify your client appropriately
* limit request rate
* cache results
* handle layout changes
* monitor failures
* avoid collecting unnecessary personal data

Scraping can be practical.

It can also be fragile.

Treat it as a last-resort integration strategy, not the default.

---

# Security

Automation often has powerful credentials.

Security concerns include:

* leaked API keys
* overbroad OAuth scopes
* weak secret storage
* unvalidated webhooks
* command injection
* insecure file handling
* unsafe deserialization
* logging sensitive data
* dependency vulnerabilities
* SSRF-style risks when fetching user-provided URLs

Security habits:

* store secrets safely
* use least privilege
* rotate credentials
* verify webhook signatures
* validate inputs
* avoid shell=True with untrusted input
* restrict file paths
* set timeouts
* limit response sizes
* log safely
* review dependencies

Automation is often trusted because it runs in the background.

That trust makes it attractive to attackers.

Secure it.

---

# Testing Automation

Automation should be tested.

Test levels:

* pure function unit tests
* API client tests with mocked responses
* contract tests against sandbox APIs
* integration tests
* dry-run tests
* end-to-end tests for critical workflows

Mocking HTTP responses can test:

* success
* `401`
* `404`
* `429`
* `500`
* malformed JSON
* timeout
* pagination
* retry behavior

Do not only test happy paths.

Automation fails in the edges:

* missing field
* duplicate event
* expired token
* slow server
* partial file
* unexpected status code
* second page

Good tests make automation boring.

Boring is good.

---

# A Practical API Sync Example

Imagine syncing customers from one API into another system.

Professional design:

```text
load configuration
create API client
fetch customers with pagination
validate each customer
skip or quarantine invalid records
upsert valid customers
retry transient failures
respect rate limits
record checkpoints
log summary
emit metrics
alert on repeated failure
```

Code sketch:

```python
def sync_customers(source_client, target_client, checkpoint_store):
    cursor = checkpoint_store.get("customer_cursor")
    processed = 0
    failed = 0

    while True:
        page = source_client.list_customers(cursor=cursor, limit=100)

        for customer in page.items:
            try:
                validate_customer(customer)
                target_client.upsert_customer(customer)
                processed += 1
            except Exception:
                failed += 1
                logger.exception(
                    "customer sync failed",
                    extra={"customer_id": customer.get("id")},
                )

        checkpoint_store.set("customer_cursor", page.next_cursor)

        if page.next_cursor is None:
            break

        cursor = page.next_cursor

    logger.info(
        "customer sync complete",
        extra={"processed": processed, "failed": failed},
    )
```

This is not full production code.

But it shows the shape.

The automation is not a single API call.

It is a controlled process.

---

# Common Mistakes

The first common mistake is omitting timeouts.

Network calls need timeouts.

The second common mistake is retrying everything.

Retry transient failures, not permanent bad requests.

The third common mistake is ignoring idempotency.

Retries can duplicate actions.

The fourth common mistake is only testing the first page of paginated APIs.

Pagination bugs are common.

The fifth common mistake is hard-coding secrets.

Use secure configuration.

The sixth common mistake is treating webhooks as trusted.

Verify signatures and handle duplicates.

The seventh common mistake is writing scripts with no logs.

If nobody can see what happened, nobody can operate it.

The eighth common mistake is ignoring rate limits.

Good clients respect API limits.

The ninth common mistake is mixing business logic with HTTP details everywhere.

Use client wrappers for important integrations.

The tenth common mistake is assuming automation is done when it works once.

Production automation must survive failure.

---

# Professional Automation Checklist

Before shipping automation, check:

* Are credentials stored securely?
* Are permissions least-privilege?
* Are timeouts set?
* Are transient failures retried with backoff?
* Are non-retryable failures handled clearly?
* Are rate limits respected?
* Is pagination tested?
* Are writes idempotent where possible?
* Are duplicate webhook events handled?
* Are inputs validated?
* Are outputs validated?
* Are logs useful and safe?
* Are metrics emitted?
* Are failures alerted?
* Is there a dry-run mode for risky changes?
* Can the job resume after partial failure?
* Are secrets excluded from logs?
* Are tests covering unhappy paths?
* Is ownership clear?
* Is documentation available for operators?

This checklist turns automation from a clever script into a dependable tool.

---

# Summary

APIs and automation are how Python connects systems.

APIs define contracts for software-to-software communication, usually over HTTP.

Python can call APIs with libraries such as `requests` and `httpx`.

JSON is the most common data format, but files, webhooks, queues, and schedules are also integration surfaces.

Professional automation requires timeouts, authentication, retries, rate-limit handling, pagination, idempotency, validation, logging, observability, configuration, dry runs, checkpointing, and security.

Small scripts are useful.

But important automation must survive real-world failure.

The central lesson is:

```text
automation becomes engineering when it is safe to run repeatedly
```

That means the script is only the beginning.

The operating behavior is the product.

---

# Exercises

1. Write a Python script that calls a public JSON API with a timeout.

2. Add `raise_for_status()` and handle HTTP errors clearly.

3. Pass query parameters using `params` instead of string concatenation.

4. Create a small API client class around three endpoints.

5. Add authentication through an environment variable.

6. Implement pagination for an API that returns multiple pages.

7. Add retry behavior for `503` but not for `400`.

8. Explain why retrying a payment-like `POST` is dangerous without idempotency.

9. Add an idempotency key to a create request.

10. Respect a `Retry-After` header from a `429` response.

11. Write a webhook handler design that verifies signatures.

12. Design a scheduled job and describe what happens if it overlaps with itself.

13. Process a directory of CSV files and archive completed files.

14. Add validation for required fields in a record.

15. Add logging with success and failure counts.

16. Add a dry-run mode to a script that would otherwise modify external data.

17. Design checkpointing for a long-running sync job.

18. List security risks for an automation script with API credentials.

19. Write tests for timeout, malformed JSON, and pagination.

20. Design an alert for a failed daily report job.

---

# Preview of Chapter 99

Chapter 98 studied APIs and Automation.

We learned how Python connects systems through HTTP APIs, JSON, clients, authentication, query parameters, request bodies, status codes, timeouts, retries, idempotency, rate limits, pagination, webhooks, schedules, files, validation, logging, observability, configuration, dry runs, checkpoints, queues, and secure automation practices.

Next we study Scripting for Real Workflows.

APIs connect systems.

Scripts connect tasks.

The transition is:

```text
APIs and automation connect services
scripts turn everyday operational work into repeatable tools
```

Chapter 99 will show how to design Python scripts that are usable, safe, configurable, testable, and helpful in real teams.
