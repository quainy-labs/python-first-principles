# Chapter 75 — Logging

Logging is how a running program leaves evidence.

When code is on your machine, you can often stop it, inspect variables, run a debugger, rerun a test, or add a temporary print.

When code is running in production, you usually cannot do that.

The program may be serving real users.

The failure may have already happened.

The process may have restarted.

The request may be gone.

The user may only report:

```text
something failed around 10:42
```

If the system did not record useful information while it was running, debugging becomes guesswork.

Logging exists to avoid that.

A log is a record of an event.

Good logs help answer:

* what happened?
* when did it happen?
* where did it happen?
* how severe was it?
* which request, user, job, or entity was involved?
* what outcome occurred?
* what context helps explain the event?

Bad logs do the opposite.

They flood the system with noise.

They omit context.

They hide the real error.

They expose sensitive data.

They make incidents harder, not easier.

Logging is not just a Python API.

It is an engineering design practice.

---

# Logging and Debugging

Chapter 74 studied debugging.

Debugging is investigation.

Logging is evidence collection.

The connection is direct:

```text
debugging asks questions
logging preserves answers
```

In local debugging, you can ask the program questions interactively.

In production, you mostly ask the logs questions after the fact.

Examples:

```text
Did this request reach the server?
Which user made it?
Which branch did the code take?
Which external service failed?
How long did the database query take?
Was the payment declined or did the payment API time out?
Did the background job retry?
Did the retry eventually succeed?
```

If logs do not contain that information, you may need to reproduce the failure elsewhere.

Sometimes that is possible.

Sometimes it is not.

Good logging gives future debugging sessions a head start.

---

# Logs Are Events

A log entry should represent an event.

An event is something that happened.

Examples:

```text
user_login_succeeded
user_login_failed
payment_authorized
payment_declined
order_created
email_queued
email_send_failed
job_started
job_finished
configuration_loaded
external_api_timeout
```

Weak log:

```text
here
```

Better log:

```text
order_created order_id=ord_123 customer_id=cust_9 item_count=3
```

The better log has meaning.

It tells a reader what happened and gives context.

Logs should not merely prove that code reached a line.

They should describe meaningful events in the system.

---

# Logging Is Not Printing

`print` writes text to standard output.

Logging creates structured event records that can be routed, filtered, formatted, stored, and searched.

Print:

```python
print("payment failed")
```

Logging:

```python
logger.warning(
    "payment_declined",
    extra={
        "order_id": order.id,
        "customer_id": order.customer_id,
        "reason": decline_reason,
    },
)
```

The difference is not only style.

Logging supports:

* severity levels
* named loggers
* handlers
* formatters
* filters
* exception tracebacks
* structured context
* configuration
* integration with libraries
* output to files, streams, sockets, queues, and monitoring systems

`print` is fine for tiny scripts and quick experiments.

Production applications should use logging.

---

# Python's logging Module

Python's standard library includes the `logging` module.

It is flexible and widely used.

The main concepts are:

* logger
* log record
* level
* handler
* formatter
* filter
* configuration

A logger is the object your code calls.

A log record is the event created by the logger.

A level says how severe or detailed the event is.

A handler sends the record somewhere.

A formatter turns the record into text or another output shape.

A filter can accept, reject, or modify records.

Configuration connects these pieces.

The simplest useful pattern is:

```python
import logging


logger = logging.getLogger(__name__)


def create_order(order):
    logger.info("order_created", extra={"order_id": order.id})
```

This module-level logger pattern is the standard starting point.

---

# getLogger(__name__)

Most modules should define a logger like this:

```python
import logging


logger = logging.getLogger(__name__)
```

`__name__` is the module name.

If the module is:

```text
shop.orders.service
```

then the logger name is:

```text
shop.orders.service
```

This creates a hierarchy matching the package hierarchy.

That hierarchy matters.

It lets an application configure broad or narrow logging behavior.

For example:

```text
shop
shop.orders
shop.orders.service
shop.payments
```

You can configure all `shop` logs together or increase detail only for `shop.payments`.

Do not create random logger names without reason.

Use module names unless you have a deliberate operational category.

---

# The Root Logger

The root logger is the top-level logger.

Calling module-level functions like this uses the root logger:

```python
logging.warning("something happened")
```

That is acceptable in tiny scripts.

In larger programs, prefer named loggers:

```python
logger = logging.getLogger(__name__)
logger.warning("something happened")
```

Named loggers preserve where the message came from.

The root logger is usually configured once by the application entry point.

Library modules should usually not configure the root logger themselves.

They should emit logs and let the application decide where logs go.

This is an important distinction:

```text
libraries log
applications configure logging
```

---

# Logging Levels

Python defines standard levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

They are ordered by severity.

`DEBUG` is the most detailed ordinary level.

`CRITICAL` is the most severe.

The level helps decide:

* whether the event should be emitted
* how urgent it is
* where it should be routed
* whether it should trigger alerts
* how much attention it deserves

Levels are not decoration.

They are operational signals.

If everything is logged as `ERROR`, errors become meaningless.

If important failures are logged as `INFO`, they may be missed.

Choose levels carefully.

---

# DEBUG

`DEBUG` is for detailed diagnostic information.

Use it for information that helps developers understand internal behavior.

Examples:

```python
logger.debug("cache_lookup key=%s hit=%s", key, hit)
logger.debug("parsed %s rows from %s", row_count, path)
logger.debug("using feature flags %r", active_flags)
```

`DEBUG` logs may be disabled in production most of the time.

They can be enabled temporarily when investigating an issue.

Do not put essential audit or business events only at `DEBUG`.

If you need the event for normal operations, use `INFO` or higher.

---

# INFO

`INFO` is for normal meaningful events.

Examples:

```python
logger.info("order_created order_id=%s customer_id=%s", order.id, order.customer_id)
logger.info("job_finished job_id=%s duration_ms=%s", job.id, duration_ms)
logger.info("server_started port=%s", port)
```

`INFO` should describe healthy system progress.

It should not be so noisy that nobody can read it.

Good `INFO` logs are often lifecycle or business events:

* request received
* job started
* job completed
* order created
* email queued
* file imported
* configuration loaded

Not every function call deserves an `INFO` log.

If logs become a transcript of every line of code, they stop being useful.

---

# WARNING

`WARNING` means something unexpected happened or may become a problem, but the system can continue.

Examples:

```python
logger.warning("retrying_payment_api order_id=%s attempt=%s", order.id, attempt)
logger.warning("deprecated_config_key_used key=%s", key)
logger.warning("cache_miss_for_required_warm_key key=%s", key)
```

A warning should make a maintainer curious.

It should not necessarily wake someone up.

Warnings are useful for:

* recoverable external failures
* fallback behavior
* deprecated usage
* suspicious but non-fatal states
* retries
* unexpected input that was handled

If warnings are constant and harmless, fix the code or lower the level.

Permanent warnings become background noise.

---

# ERROR

`ERROR` means a function, request, job, or operation failed.

Examples:

```python
logger.error("payment_capture_failed order_id=%s", order.id)
logger.error("report_generation_failed report_id=%s", report.id)
```

Use `ERROR` when something important did not complete successfully.

But be careful not to log the same error repeatedly at multiple layers.

For example, if a low-level function logs an error and raises, and the top-level handler logs the same exception again, logs may contain duplicates.

Often it is better to log exceptions at a boundary where there is enough context:

* API handler boundary
* background job boundary
* CLI command boundary
* message consumer boundary

Lower-level code can raise meaningful exceptions.

Boundary code can log them with context.

---

# CRITICAL

`CRITICAL` means the program or a major subsystem may be unable to continue.

Examples:

```python
logger.critical("database_unavailable service_startup_failed")
logger.critical("cannot_load_required_configuration path=%s", path)
```

Use this level rarely.

If everything is critical, nothing is critical.

`CRITICAL` should usually indicate a condition that requires immediate attention or shutdown.

---

# Choosing the Right Level

Ask:

```text
who needs to know about this event?
when do they need to know?
what action should they take?
```

If the event helps a developer during investigation, use `DEBUG`.

If it records normal progress, use `INFO`.

If it is unexpected but handled, use `WARNING`.

If an operation failed, use `ERROR`.

If the system may not continue, use `CRITICAL`.

Logging levels should reflect operational meaning, not emotional intensity.

---

# Basic Configuration

For a small script, `basicConfig` may be enough:

```python
import logging


logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)

logger = logging.getLogger(__name__)
logger.info("started")
```

This configures the root logger.

It sets:

* minimum level
* output format
* default handler if none exists

In real applications, logging configuration often belongs near the entry point:

```python
def main():
    configure_logging()
    run_application()
```

Avoid configuring logging deeply inside business modules.

That makes libraries surprising and hard to integrate.

---

# Loggers, Handlers, and Formatters

A logger creates records.

A handler sends records somewhere.

A formatter controls output shape.

Example:

```python
import logging


logger = logging.getLogger("shop")
logger.setLevel(logging.INFO)

handler = logging.StreamHandler()
handler.setLevel(logging.INFO)

formatter = logging.Formatter(
    "%(asctime)s %(levelname)s %(name)s %(message)s"
)
handler.setFormatter(formatter)

logger.addHandler(handler)
```

This says:

```text
shop logger accepts INFO and above
stream handler emits INFO and above
formatter controls the text layout
```

Both loggers and handlers can have levels.

A message must pass the logger level and the handler level.

This is a common source of confusion.

If a log does not appear, check both.

---

# Handler Destinations

Handlers decide where logs go.

Common handlers include:

* `StreamHandler`
* `FileHandler`
* `RotatingFileHandler`
* `TimedRotatingFileHandler`
* `QueueHandler`
* `SocketHandler`
* `SMTPHandler`
* `SysLogHandler`
* `NullHandler`

In many modern deployments, applications write logs to standard output and the platform collects them.

Examples:

* Docker
* Kubernetes
* systemd
* cloud application platforms

For those environments, `StreamHandler` is often enough.

The platform handles collection, rotation, storage, and search.

Do not assume file logging is always best.

The right handler depends on the runtime environment.

---

# Formatter Output

A formatter decides how a log record appears.

Text format:

```python
"%(asctime)s %(levelname)s %(name)s %(message)s"
```

Possible output:

```text
2026-01-01 10:15:22,123 INFO shop.orders order_created order_id=ord_123
```

Useful fields include:

* `asctime`
* `levelname`
* `name`
* `message`
* `module`
* `funcName`
* `lineno`
* `process`
* `threadName`

Do not include every possible field by default.

Too much information makes logs hard to scan.

Include fields that help your operational environment.

For production systems, structured logs are often better than dense text.

---

# Structured Logging

Structured logging means recording logs as fields instead of only free-form text.

Text log:

```text
created order ord_123 for customer cust_9
```

Structured log:

```json
{
  "event": "order_created",
  "order_id": "ord_123",
  "customer_id": "cust_9",
  "level": "INFO"
}
```

Structured logs are easier to search and aggregate.

You can ask:

```text
show all logs where order_id = ord_123
show payment failures grouped by reason
show requests with duration_ms > 5000
```

In Python's standard logging, structured context can be added through `extra`, custom formatters, logger adapters, filters, or third-party libraries.

Even if your final output is text, write messages with stable keys:

```python
logger.info(
    "order_created order_id=%s customer_id=%s item_count=%s",
    order.id,
    order.customer_id,
    len(order.items),
)
```

This is easier to search than:

```python
logger.info("The order was created successfully.")
```

---

# The extra Argument

The `extra` argument adds custom attributes to a log record.

Example:

```python
logger.info(
    "order_created",
    extra={
        "order_id": order.id,
        "customer_id": order.customer_id,
    },
)
```

A formatter can include those fields:

```python
formatter = logging.Formatter(
    "%(asctime)s %(levelname)s %(name)s order_id=%(order_id)s %(message)s"
)
```

Be careful.

If the formatter expects `order_id` but a log record does not provide it, formatting can fail.

For broad application logging, custom JSON formatters or filters often handle missing fields more gracefully.

`extra` is useful, but it should be used consistently.

---

# LoggerAdapter

`LoggerAdapter` attaches contextual information to log calls.

Example:

```python
import logging


base_logger = logging.getLogger(__name__)
logger = logging.LoggerAdapter(
    base_logger,
    {"request_id": "req_123"},
)

logger.info("request_started")
```

The adapter injects context into records.

This is useful when many logs share the same context:

* request ID
* job ID
* tenant ID
* customer ID
* worker name

Without an adapter, every log call may repeat the same extra fields.

Adapters make context easier to carry.

For more complex systems, context variables and filters may be more ergonomic.

---

# Correlation IDs

A correlation ID links logs that belong to the same flow.

In web applications, this is often a request ID.

In background systems, it may be:

* job ID
* message ID
* trace ID
* workflow ID
* import ID
* order ID

Without correlation, logs are fragments.

With correlation, logs become a story.

Example:

```text
request_id=req_123 request_started path=/orders
request_id=req_123 order_created order_id=ord_9
request_id=req_123 payment_authorized payment_id=pay_7
request_id=req_123 request_finished status=201 duration_ms=184
```

Now you can follow one request.

Correlation IDs are one of the highest-value logging practices in production systems.

Add them early.

---

# Logging Exceptions

When logging an exception, preserve the traceback.

Inside an exception handler, use:

```python
logger.exception("failed_to_process_order order_id=%s", order.id)
```

`logger.exception` logs at `ERROR` level and includes exception information.

It should be called only while handling an exception.

Equivalent:

```python
logger.error(
    "failed_to_process_order order_id=%s",
    order.id,
    exc_info=True,
)
```

Bad:

```python
try:
    process_order(order)
except Exception as error:
    logger.error("failed: %s", error)
```

This logs only the exception string.

It loses the traceback.

The traceback is usually the most useful part.

Good:

```python
try:
    process_order(order)
except Exception:
    logger.exception("failed_to_process_order order_id=%s", order.id)
    raise
```

Log the context.

Preserve the stack.

Re-raise unless you truly handled the failure.

---

# Do Not Log and Swallow

This is a common bad pattern:

```python
try:
    charge_customer(order)
except Exception:
    logger.exception("payment_failed")
```

After logging, execution continues.

Maybe the order is treated as successful.

Maybe the caller never knows payment failed.

Logging an exception is not handling it.

If the failure should affect behavior, represent that in code:

```python
try:
    charge_customer(order)
except PaymentDeclined:
    order.mark_payment_declined()
    logger.info("payment_declined order_id=%s", order.id)
except Exception:
    logger.exception("payment_capture_failed order_id=%s", order.id)
    raise
```

Logs are evidence.

They are not control flow.

---

# Avoid Duplicate Exception Logs

Duplicate logs make incidents noisy.

Bad layering:

```python
def repository_save(order):
    try:
        ...
    except Exception:
        logger.exception("repository_save_failed")
        raise


def checkout(order):
    try:
        repository_save(order)
    except Exception:
        logger.exception("checkout_failed")
        raise


def api_handler(request):
    try:
        checkout(order)
    except Exception:
        logger.exception("request_failed")
        return error_response()
```

One failure creates three stack traces.

Sometimes multiple logs are justified when each adds distinct context.

Often they are not.

A better approach:

* lower layers raise meaningful exceptions
* boundary layer logs once with enough context

For example:

```python
def api_handler(request):
    try:
        checkout(order)
    except Exception:
        logger.exception(
            "checkout_request_failed request_id=%s order_id=%s",
            request.id,
            order.id,
        )
        return error_response()
```

One clear log beats three vague logs.

---

# Lazy Formatting

Use logging's lazy formatting:

```python
logger.debug("loaded %s rows from %s", row_count, path)
```

Avoid eager f-string formatting for debug logs:

```python
logger.debug(f"loaded {row_count} rows from {path}")
```

Why?

With lazy formatting, the string interpolation happens only if the log will be emitted.

With an f-string, formatting happens before the logging call.

For cheap values, the difference may not matter.

For expensive formatting, large objects, or disabled debug logs, it can matter.

This is especially important:

```python
logger.debug("payload=%r", expensive_to_render(payload))
```

The function call still happens before logging.

If computing the value is expensive, guard it:

```python
if logger.isEnabledFor(logging.DEBUG):
    logger.debug("payload=%s", expensive_to_render(payload))
```

Use the guard only when needed.

Do not make ordinary logging code noisy without reason.

---

# Log Messages Should Be Stable

Stable messages are easier to search.

Weak:

```python
logger.info("Created order %s for customer %s", order.id, customer.id)
```

Better:

```python
logger.info(
    "order_created order_id=%s customer_id=%s",
    order.id,
    customer.id,
)
```

The event name `order_created` is stable.

The fields are predictable.

Search becomes easier.

Avoid highly variable prose:

```python
logger.info("%s just bought %s and now has %s points!", name, item, points)
```

Logs are for machines and humans under pressure.

Make them boring in the best way.

---

# What to Log

Useful logs often include:

* lifecycle events
* major business events
* external boundary calls
* retries
* failures
* degraded behavior
* security-relevant decisions
* configuration summaries
* background job outcomes
* data import/export summaries
* slow operations
* unexpected but handled states

Examples:

```python
logger.info("job_started job_id=%s type=%s", job.id, job.type)
logger.info("job_finished job_id=%s duration_ms=%s", job.id, duration_ms)
logger.warning("external_api_retry service=%s attempt=%s", service, attempt)
logger.error("external_api_failed service=%s", service, exc_info=True)
```

Do not log every variable.

Do not log every line.

Log events that help someone understand system behavior.

---

# What Not to Log

Do not log secrets.

Examples:

* passwords
* API keys
* access tokens
* refresh tokens
* private keys
* session cookies
* one-time passwords
* full payment card numbers
* sensitive personal data

Bad:

```python
logger.info("login_attempt email=%s password=%s", email, password)
```

Better:

```python
logger.info("login_attempt email=%s", email)
```

Even email addresses may be considered personal data depending on context and policy.

Be deliberate.

For highly sensitive systems, use internal IDs or hashed identifiers where appropriate.

Logs often live longer and travel farther than application memory.

Treat them as data stores with security responsibilities.

---

# Redaction

Redaction means removing or masking sensitive values before they reach logs.

Example:

```python
def mask_token(token):
    if not token:
        return "<missing>"
    return token[:4] + "..." + token[-4:]
```

Log:

```python
logger.info("api_token_loaded token=%s", mask_token(token))
```

For structured data, redact before logging:

```python
def redact_user_payload(payload):
    redacted = dict(payload)
    redacted.pop("password", None)
    redacted.pop("token", None)
    return redacted
```

Do not rely on humans to remember redaction at every call site in sensitive systems.

Centralized filters, formatters, or logging wrappers may be needed.

Security-conscious logging is designed, not improvised.

---

# Logging User Input

User input is useful for debugging.

It is also risky.

User input may contain:

* secrets
* personal information
* malicious strings
* very large payloads
* binary data
* control characters
* misleading text

Avoid logging full raw payloads by default.

Prefer summaries:

```python
logger.info(
    "user_import_received file_name=%s row_count=%s size_bytes=%s",
    file_name,
    row_count,
    size_bytes,
)
```

For validation errors, log safe context:

```python
logger.warning(
    "invalid_signup_payload missing_fields=%s",
    missing_fields,
)
```

If raw payload logging is required for a temporary investigation, make it explicit, access-controlled, and short-lived.

---

# Logging in Libraries

Libraries should log, but they should not usually configure logging.

Library module:

```python
import logging


logger = logging.getLogger(__name__)


def parse_file(path):
    logger.debug("parsing_file path=%s", path)
```

The library does not call `basicConfig`.

The application using the library decides:

* whether to show the log
* where to send it
* what level to use
* what format to use

For libraries, avoid noisy logs at `INFO`.

Library code may be used in many contexts.

Detailed library internals usually belong at `DEBUG`.

Warnings should indicate something the application developer should know.

---

# NullHandler

Library authors sometimes attach a `NullHandler` to the package logger:

```python
import logging


logging.getLogger(__name__).addHandler(logging.NullHandler())
```

This prevents "no handler" warnings in older logging patterns and makes the library quiet unless the application configures logging.

Modern Python has a `lastResort` handler, but `NullHandler` remains a common library practice.

The principle is:

```text
a library should not surprise the application with logging configuration
```

Emit records.

Let applications decide how to handle them.

---

# Logging Configuration with dictConfig

For applications, `logging.config.dictConfig` is often cleaner than manual setup.

Example:

```python
import logging.config


LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "default": {
            "format": "%(asctime)s %(levelname)s %(name)s %(message)s",
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "default",
            "level": "INFO",
        },
    },
    "root": {
        "handlers": ["console"],
        "level": "INFO",
    },
}


logging.config.dictConfig(LOGGING)
```

`dictConfig` is useful because configuration can be:

* centralized
* environment-specific
* loaded from files
* adjusted without touching every module
* easier to review than scattered setup code

Be careful with:

```python
"disable_existing_loggers": True
```

It can unexpectedly silence loggers created before configuration.

In many applications, `False` is the safer default.

---

# Duplicate Log Lines

Duplicate log lines often come from handler propagation mistakes.

Example:

```python
logger = logging.getLogger("shop.orders")
logger.addHandler(handler)

root = logging.getLogger()
root.addHandler(handler)
```

If `shop.orders` propagates to root, the same record may be emitted twice.

Remember:

* loggers form a hierarchy
* records propagate upward by default
* handlers attached at multiple levels can duplicate output

Common fix:

```text
attach handlers high in the hierarchy
let child loggers propagate
```

Or, for a special logger:

```python
logger.propagate = False
```

Use `propagate = False` deliberately.

Do not scatter it randomly.

---

# Logging in Web Applications

Web application logs should make request flow visible.

Useful fields include:

* request ID
* method
* path
* status code
* duration
* user ID when safe
* tenant or organization ID
* client IP when appropriate
* error information

Example:

```python
logger.info(
    "request_finished method=%s path=%s status=%s duration_ms=%s request_id=%s",
    request.method,
    request.path,
    response.status_code,
    duration_ms,
    request.id,
)
```

For errors:

```python
logger.exception(
    "request_failed method=%s path=%s request_id=%s",
    request.method,
    request.path,
    request.id,
)
```

Avoid logging full request bodies by default.

Bodies may contain secrets or personal data.

Log shape and identifiers first.

---

# Logging Background Jobs

Background jobs need lifecycle logs.

Useful events:

* job received
* job started
* job finished
* job failed
* job retried
* job abandoned
* job duration
* number of records processed

Example:

```python
logger.info("job_started job_id=%s job_type=%s", job.id, job.type)

try:
    result = process_job(job)
except Exception:
    logger.exception("job_failed job_id=%s job_type=%s", job.id, job.type)
    raise
else:
    logger.info(
        "job_finished job_id=%s job_type=%s processed=%s duration_ms=%s",
        job.id,
        job.type,
        result.processed_count,
        result.duration_ms,
    )
```

Jobs often fail after the original user request is gone.

Logs may be the only easy way to understand what happened.

---

# Logging External Calls

External calls are important boundaries.

Examples:

* payment API
* email provider
* search service
* object storage
* internal microservice
* third-party data provider

Log failures and retries.

For high-value operations, log success summaries too.

Example:

```python
logger.info(
    "payment_authorization_started order_id=%s provider=%s",
    order.id,
    provider.name,
)

try:
    response = provider.authorize(order)
except TimeoutError:
    logger.warning(
        "payment_authorization_timeout order_id=%s provider=%s",
        order.id,
        provider.name,
    )
    raise
```

Be careful with provider responses.

They may contain sensitive data.

Log stable identifiers, status, duration, and safe error codes.

---

# Logging Retries

Retries should be visible.

Example:

```python
logger.warning(
    "retrying_external_call service=%s attempt=%s max_attempts=%s reason=%s",
    service_name,
    attempt,
    max_attempts,
    error.__class__.__name__,
)
```

Retries are not always errors.

The final outcome matters.

If a retry succeeds, the operation may be healthy but degraded.

If all retries fail, log the final failure with exception information.

Avoid logging every retry at `ERROR`.

That can make one temporary outage look like many independent failures.

Use levels to express meaning:

```text
WARNING for retrying
ERROR for final failure
```

---

# Logging Slow Operations

Slow operations are often more useful to log than ordinary fast operations.

Example:

```python
if duration_ms > 1000:
    logger.warning(
        "slow_database_query query_name=%s duration_ms=%s",
        query_name,
        duration_ms,
    )
```

Slow logs help find:

* database bottlenecks
* network latency
* large inputs
* lock contention
* slow external services
* inefficient algorithms

Do not log every tiny timing detail at `INFO`.

Set thresholds.

Make slow logs actionable.

Chapter 79 will study profiling, but logging can provide early performance clues in real systems.

---

# Logging Security Events

Security-relevant events deserve careful logging.

Examples:

* login success
* login failure
* password reset requested
* password changed
* account locked
* permission denied
* API key created
* API key revoked
* admin action performed
* suspicious rate limit exceeded

Example:

```python
logger.warning(
    "permission_denied user_id=%s resource=%s action=%s",
    user.id,
    resource.id,
    action,
)
```

Security logs must avoid sensitive secrets.

They should include enough context for audit and investigation.

They may need different retention and access rules than ordinary application logs.

Security logging is not the same as dumping everything.

It is selective, consistent, and careful.

---

# Logging Audit Events

Audit logs are records of important business or security actions.

Examples:

* user invited
* role changed
* invoice deleted
* payment refunded
* data exported
* configuration changed

Audit logs differ from debug logs.

They may need to be:

* durable
* searchable
* tamper-resistant
* access-controlled
* retained for a defined period
* shown to administrators

Do not assume ordinary application logs are enough for auditing.

Sometimes audit events should be stored in a database table or dedicated audit system.

But the logging mindset still applies:

```text
record meaningful events with safe context
```

---

# Logging and Privacy

Logs can become privacy risks.

They often contain:

* identifiers
* IP addresses
* emails
* filenames
* search queries
* error context
* user actions
* operational metadata

Before logging personal data, ask:

* is it necessary?
* who can access it?
* how long is it retained?
* is there a safer identifier?
* does policy allow it?
* can it be redacted or hashed?

Privacy-aware logging is part of responsible engineering.

The fact that a value is useful does not automatically mean it should be logged.

---

# Log Volume

Logs cost money and attention.

High log volume can cause:

* storage cost
* slow searches
* rate limits
* dropped logs
* noisy dashboards
* hidden important events
* performance overhead

Manage volume with:

* appropriate levels
* sampling
* thresholds
* aggregation
* structured events
* avoiding logs inside hot loops
* avoiding repeated identical messages

Bad:

```python
for row in million_rows:
    logger.info("processing row %s", row.id)
```

Better:

```python
logger.info("import_started row_count=%s", len(rows))

for row in rows:
    process(row)

logger.info("import_finished row_count=%s", len(rows))
```

If individual row failures matter, log failures with row IDs.

Do not log every successful row unless there is a strong reason.

---

# Sampling Logs

Sampling means logging only some repeated events.

For example, if a non-critical event happens thousands of times per second, log a sample:

```python
if random.random() < 0.01:
    logger.info("cache_miss key_prefix=%s", key[:4])
```

Sampling can reduce volume.

But use it carefully.

Do not sample rare critical errors.

Do not sample audit events.

Do not sample events needed for exact accounting.

Sampling is for high-volume diagnostic events where approximate visibility is enough.

---

# Rate-Limited Logging

Rate-limited logging prevents repeated messages from flooding logs.

Example problem:

```text
external service unavailable
external service unavailable
external service unavailable
...
```

Thousands of identical logs may not add information.

A rate-limited logger might emit:

```text
external_service_unavailable repeated=523 window_seconds=60
```

Python's standard logging does not provide a single universal rate-limiting API.

You can implement rate limiting with filters, counters, wrappers, or logging infrastructure.

The principle is:

```text
preserve signal
reduce repetition
```

---

# Logging in Loops

Logging inside loops requires care.

Sometimes it is right:

```python
for failed_row in failed_rows:
    logger.warning("row_failed row_id=%s reason=%s", failed_row.id, failed_row.reason)
```

Sometimes it is noise:

```python
for row in rows:
    logger.info("processing row")
```

Ask:

* how many times can this run?
* is each event meaningful?
* will this help debugging?
* could this expose data?
* could this overwhelm logs?

For bulk operations, log summaries:

```python
logger.info(
    "import_finished total=%s succeeded=%s failed=%s duration_ms=%s",
    total,
    succeeded,
    failed,
    duration_ms,
)
```

Summaries often carry more signal than per-item chatter.

---

# Warnings and Logging

Python has a `warnings` module separate from logging.

Warnings are often used for:

* deprecations
* questionable usage
* compatibility notices
* runtime conditions developers should see

Logging can capture warnings:

```python
logging.captureWarnings(True)
```

This routes warnings through logging.

Warnings and logs have different roles.

Use warnings when calling code should be alerted to usage that may need changing.

Use logging when the application should record runtime events.

Deprecation warnings belong to callers.

Operational failures belong to logs.

---

# stack_info and stacklevel

Logging supports stack-related options.

`exc_info=True` logs an exception traceback.

`stack_info=True` logs the current stack even when there is no exception.

Example:

```python
logger.debug("unexpected_path_reached", stack_info=True)
```

This can be useful when you want to know how code reached a point.

`stacklevel` helps logging helper functions report the caller's location.

Example:

```python
def log_deprecated(message):
    logger.warning(message, stacklevel=2)
```

Without `stacklevel`, the log may point to the helper.

With `stacklevel=2`, it can point to the caller.

Use these features when location information matters.

Do not include stack info casually in high-volume logs.

Stacks are large.

---

# Filters

Filters can decide whether a log record should be emitted.

They can also add context.

Example:

```python
class HealthCheckFilter(logging.Filter):
    def filter(self, record):
        return "health_check" not in record.getMessage()
```

Attach it to a handler:

```python
handler.addFilter(HealthCheckFilter())
```

Filters can be useful for:

* removing noisy health checks
* adding request IDs
* redacting sensitive fields
* suppressing known noisy library messages
* routing records

Be careful.

Filters can hide evidence.

If you filter something out, make sure it is truly not needed.

---

# Context Variables

Modern Python applications often use `contextvars` to hold request-local context.

This is useful in async systems where thread-local storage is not enough.

Conceptually:

```python
request_id_var = ContextVar("request_id", default="-")
```

A logging filter can read the current request ID and attach it to each record.

This lets code log normally:

```python
logger.info("order_created order_id=%s", order.id)
```

while the logging system adds:

```text
request_id=req_123
```

Context propagation is especially useful in:

* web applications
* async services
* background workers
* message consumers

The goal is to avoid manually passing request IDs into every log call while still preserving correlation.

---

# Queue-Based Logging

Logging can block.

If a handler writes to a slow destination, application code may slow down.

Queue-based logging separates log creation from log output.

Application threads send records to a queue.

A listener thread handles output.

Python's logging cookbook describes `QueueHandler` and `QueueListener` patterns.

This is useful when:

* logs go to slow destinations
* many threads log heavily
* you want centralized formatting
* you want application threads to spend less time on I/O

Queue logging adds complexity.

Use it when the operational need justifies it.

---

# Logging and Threads

Python's logging module is designed to be thread-safe for ordinary use.

Multiple threads can log through the same logger.

Still, threaded applications need context.

Include:

* thread name if useful
* request ID
* job ID
* worker ID
* entity IDs

Thread name alone is rarely enough.

Example format field:

```python
"%(asctime)s %(levelname)s %(threadName)s %(name)s %(message)s"
```

Threaded logs can interleave.

Structured context helps restore the story.

---

# Logging and Async

Async applications interleave tasks in one thread.

Thread-local context may not identify the logical request.

Use request IDs, task context, or context variables.

Async logging concerns include:

* preserving context across awaits
* avoiding blocking handlers
* logging task failures
* logging cancellation
* avoiding huge logs from concurrent tasks

If an async task fails silently, the log may be the only evidence.

Always observe background task failures.

Do not create tasks and forget them without error handling.

---

# Logging and Multiprocessing

Multiprocessing complicates logging because each process has its own memory and logging configuration.

Problems include:

* multiple processes writing to the same file
* interleaved output
* missing configuration in child processes
* duplicated handlers after process start
* lost logs during shutdown

Queue-based logging is often used for multiprocessing.

Child processes send log records to a central listener.

The listener writes them.

If your application uses multiple processes, test logging behavior under that model.

Do not assume single-process logging configuration automatically works everywhere.

---

# Logging to Files

File logging is useful in some environments.

Example:

```python
logging.basicConfig(
    filename="app.log",
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)
```

But files need management.

Questions:

* who rotates the file?
* how large can it grow?
* who reads it?
* how is it collected?
* what permissions protect it?
* what happens when disk is full?

For long-running applications, use rotation or external log management.

Unbounded log files can break systems.

---

# Log Rotation

Log rotation prevents files from growing forever.

Python provides handlers such as:

* `RotatingFileHandler`
* `TimedRotatingFileHandler`

Example:

```python
from logging.handlers import RotatingFileHandler


handler = RotatingFileHandler(
    "app.log",
    maxBytes=10_000_000,
    backupCount=5,
)
```

This rotates after a size threshold and keeps a number of backups.

In containerized environments, the platform may handle rotation.

Do not rotate in the application if the platform already expects logs on stdout.

Logging strategy depends on deployment.

---

# JSON Logs

JSON logs are common in production systems.

Example:

```json
{
  "timestamp": "2026-01-01T10:15:22.123Z",
  "level": "INFO",
  "logger": "shop.orders",
  "event": "order_created",
  "order_id": "ord_123",
  "customer_id": "cust_9"
}
```

Benefits:

* easier parsing
* better search
* consistent fields
* integration with log platforms
* safer aggregation

Costs:

* custom formatter needed
* field discipline required
* less pleasant raw terminal output

For local development, text logs may be friendlier.

For production, JSON logs are often more useful.

Many teams use different formatters per environment.

---

# Designing Log Fields

Good field names are stable and consistent.

Prefer:

```text
request_id
user_id
order_id
duration_ms
status_code
service
attempt
error_type
```

Avoid changing names casually:

```text
req
request
rid
requestId
request_id
```

Pick conventions and keep them.

Consistent fields make searching easier.

They also make dashboards and alerts more reliable.

Logging is a data model.

Treat field names like part of an interface.

---

# Event Names

Event names should be stable.

Examples:

```text
request_started
request_finished
request_failed
order_created
payment_declined
job_started
job_finished
job_failed
external_api_timeout
```

Stable event names help search and alerting.

Avoid prose-only logs where the event cannot be reliably extracted.

Weak:

```python
logger.info("The order has been created successfully.")
```

Better:

```python
logger.info("order_created order_id=%s", order.id)
```

Best in structured systems:

```python
logger.info("order_created", extra={"event": "order_created", "order_id": order.id})
```

The exact style depends on your logging infrastructure.

The principle is stable naming.

---

# Logging Errors From Libraries

Third-party libraries may emit logs.

Because logging is hierarchical, application configuration can control them.

For example:

```python
logging.getLogger("urllib3").setLevel(logging.WARNING)
```

This can reduce noisy dependency logs.

But do not silence dependency errors blindly.

Some library logs contain useful operational information.

Adjust levels based on observed signal, not annoyance alone.

---

# Logging Configuration by Environment

Different environments need different logging.

Local development:

* readable text
* maybe `DEBUG`
* console output

Testing:

* capture logs
* fail on unexpected errors when useful
* avoid noisy output

Staging:

* production-like format
* more diagnostic detail

Production:

* structured logs
* correlation IDs
* controlled volume
* safe redaction
* appropriate retention

Configuration should make these differences explicit.

Do not edit code to change log levels between environments.

Use configuration.

---

# Testing Logs

Logs can be tested when they are part of behavior.

pytest provides `caplog`.

Example:

```python
def test_payment_decline_is_logged(caplog):
    process_declined_payment(order)

    assert "payment_declined" in caplog.text
```

More specific:

```python
def test_payment_decline_is_logged(caplog):
    with caplog.at_level(logging.WARNING):
        process_declined_payment(order)

    assert any(
        record.levelname == "WARNING"
        and "payment_declined" in record.message
        for record in caplog.records
    )
```

Do not test every log.

Test logs when they are:

* audit behavior
* security behavior
* operationally important
* part of a public or team contract

Over-testing logs makes harmless wording changes painful.

Test important events, not incidental phrasing.

---

# caplog

`caplog` is pytest's log capture fixture.

Example:

```python
import logging


def test_logs_warning(caplog):
    with caplog.at_level(logging.WARNING):
        logger.warning("low_disk_space disk=%s", "/data")

    assert "low_disk_space" in caplog.text
```

You can inspect records:

```python
record = caplog.records[0]

assert record.levelname == "WARNING"
assert record.name == __name__
```

Testing records is often more robust than testing formatted output.

Formatted output may differ by environment.

The record contains the logging facts.

---

# Logging in Tests

Tests should not be noisy by default.

If every test emits logs, failure output becomes hard to read.

Most test runners capture logs and show them on failure.

That is usually helpful.

When debugging a test, you can increase log level:

```bash
pytest -o log_cli=true --log-cli-level=DEBUG
```

Use test logging intentionally.

Do not rely on humans reading logs to know whether tests passed.

Tests should assert behavior.

Logs can help diagnose failures.

---

# Logging Versus Metrics

Logs and metrics serve different purposes.

Logs describe events.

Metrics measure quantities over time.

Log:

```text
payment_declined order_id=ord_123 reason=insufficient_funds
```

Metric:

```text
payment_declined_count{reason="insufficient_funds"} += 1
```

Logs help investigate individual cases.

Metrics help see trends and alert on aggregate behavior.

Do not use logs as your only metrics system.

Do not use metrics as your only debugging evidence.

They complement each other.

---

# Logging Versus Tracing

Distributed tracing follows a request across service boundaries.

A trace shows spans:

```text
API request
  database query
  payment service call
  email queue publish
```

Logs are event records.

Traces show timing and relationships.

In distributed systems, logs should include trace IDs or request IDs so they can be connected with traces.

The tools differ, but the goal is shared:

```text
make system behavior observable
```

Logging is one part of observability.

It is not the whole story.

---

# Observability

Observability is the ability to understand a system's internal state from its external signals.

Common signals are:

* logs
* metrics
* traces

Logs answer:

```text
what happened in this event?
```

Metrics answer:

```text
how often, how many, how fast?
```

Traces answer:

```text
where did this request spend time?
```

Good systems use all three.

This chapter focuses on logs, but professional engineering should treat logging as part of a larger observability design.

---

# Logging During Startup

Startup logs are important.

They should confirm safe configuration without revealing secrets.

Useful startup logs:

```text
service_started version=1.4.2 environment=production
database_configured host=db.internal database=shop
cache_configured backend=redis
feature_flags_loaded count=18
```

Dangerous startup logs:

```text
DATABASE_URL=postgres://user:password@host/db
API_KEY=secret-value
```

Startup failures often happen before request handling exists.

Logs may be the only evidence.

Make startup logs concise and safe.

---

# Logging Shutdown

Shutdown logs help explain exits.

Useful events:

* shutdown requested
* worker stopping
* queue drained
* in-flight requests completed
* cleanup failed
* shutdown complete

Example:

```python
logger.info("shutdown_started reason=%s", reason)
logger.info("shutdown_finished duration_ms=%s", duration_ms)
```

Shutdown logs are especially useful for background workers and long-running services.

They help distinguish:

* graceful shutdown
* crash
* forced termination
* restart loop

---

# Logging Configuration Values Safely

Configuration is often the cause of production issues.

It is useful to log safe summaries.

Good:

```python
logger.info(
    "configuration_loaded environment=%s debug=%s api_host=%s timeout_seconds=%s",
    environment,
    debug,
    api_host,
    timeout_seconds,
)
```

Bad:

```python
logger.info("configuration=%r", config)
```

The full config may contain secrets.

Log values that help diagnose behavior.

Mask or omit sensitive values.

---

# Logging Database Activity

Database logging can be very useful and very noisy.

Useful:

* slow queries
* failed transactions
* migration start and finish
* connection pool exhaustion
* deadlocks
* retryable database errors

Potentially noisy:

* every query
* every row
* every connection checkout

For application logs, prefer high-level database events:

```python
logger.warning(
    "slow_query query_name=%s duration_ms=%s row_count=%s",
    query_name,
    duration_ms,
    row_count,
)
```

Use database-specific tools for deeper query analysis.

Application logs should not become a full database trace unless you intentionally enable that for diagnosis.

---

# Logging Data Imports

Data imports need summary logs.

Example:

```python
logger.info(
    "import_finished import_id=%s total_rows=%s inserted=%s updated=%s failed=%s",
    import_id,
    total,
    inserted,
    updated,
    failed,
)
```

For failures, log enough to locate the problem:

```python
logger.warning(
    "import_row_failed import_id=%s row_number=%s reason=%s",
    import_id,
    row_number,
    reason,
)
```

Do not log entire rows if they may contain sensitive data.

Use row numbers, error codes, and safe identifiers.

---

# Logging CLI Tools

CLI tools have two audiences:

* humans reading terminal output
* maintainers reading logs

Sometimes `print` is right for user-facing output.

Logging is right for diagnostics.

Example:

```python
def main():
    configure_logging()
    logger.info("command_started command=import")
    print("Import complete")
```

Do not send internal debug logs to normal stdout if the CLI output is meant to be parsed by another program.

Use stderr for logs when appropriate.

Keep machine-readable output clean.

---

# Logging Data Structures

Be careful logging large structures.

Bad:

```python
logger.debug("users=%r", users)
```

If `users` is huge, this may flood logs.

Better:

```python
logger.debug("users_loaded count=%s first_user_id=%s", len(users), users[0].id)
```

When debugging deep structures, log summaries:

* length
* keys
* type
* relevant IDs
* selected fields

Example:

```python
logger.debug(
    "payload_received keys=%s size_bytes=%s",
    sorted(payload.keys()),
    len(raw_body),
)
```

Summaries preserve signal without dumping everything.

---

# Logging Object Representations

Logging an object uses its string representation.

This can be helpful or terrible.

If an object's `__repr__` includes sensitive data, logging it is dangerous.

Example:

```python
@dataclass
class User:
    email: str
    password_hash: str
```

The generated representation includes both fields.

Logging:

```python
logger.info("user=%r", user)
```

may expose `password_hash`.

Be deliberate with `__repr__` on sensitive classes.

For dataclasses, fields can be excluded from representation:

```python
from dataclasses import dataclass, field


@dataclass
class User:
    email: str
    password_hash: str = field(repr=False)
```

Object representation affects logging safety.

---

# Logging Exceptions With Context

This is weak:

```python
logger.exception("failed")
```

This is better:

```python
logger.exception(
    "invoice_generation_failed invoice_id=%s customer_id=%s",
    invoice.id,
    invoice.customer_id,
)
```

Context turns a stack trace into an investigation starting point.

Good exception logs include:

* operation name
* entity ID
* request or job ID
* external service name
* safe error code
* relevant state summary

Do not include secrets or full payloads.

The goal is enough context to find the case and reproduce or reason about it.

---

# Logging Expected Failures

Not every failure is an error.

Example:

```text
payment declined because the card was rejected
```

That may be a normal business outcome.

It should not necessarily be logged as `ERROR`.

Maybe:

```python
logger.info(
    "payment_declined order_id=%s reason=%s",
    order.id,
    decline_reason,
)
```

or `WARNING` if unusual.

An exception in code and an operational error are not always the same.

Choose log levels based on system meaning.

Expected business outcomes should not page operators.

---

# Logging Unexpected Failures

Unexpected failures deserve stronger logs.

Example:

```python
try:
    payment = gateway.capture(order)
except PaymentDeclined:
    logger.info("payment_declined order_id=%s", order.id)
    order.mark_declined()
except Exception:
    logger.exception("payment_capture_unexpected_failure order_id=%s", order.id)
    raise
```

This distinguishes:

```text
customer card was declined
```

from:

```text
our system could not complete payment capture
```

That distinction matters operationally.

Logs should reflect it.

---

# Logging State Transitions

State transitions are worth logging when they matter.

Example:

```python
logger.info(
    "order_status_changed order_id=%s old_status=%s new_status=%s",
    order.id,
    old_status,
    new_status,
)
```

State transition logs are useful for:

* orders
* payments
* subscriptions
* background jobs
* deployments
* user accounts
* approval workflows

Do not log every trivial field change.

Log transitions that help reconstruct important behavior.

---

# Logging Without Leaking Control Flow

Logs should not become required for correctness.

Bad:

```python
logger.info("sent email")
return None
```

The caller cannot tell whether email was sent except by reading logs.

Better:

```python
result = mailer.send(message)
logger.info("email_sent message_id=%s", result.message_id)
return result
```

Logs record behavior.

Return values and exceptions communicate behavior inside the program.

Do not make logs the only API.

---

# Logging and Alerting

Logs can feed alerts, but not every log should alert.

Alert-worthy conditions usually require action.

Examples:

* error rate exceeds threshold
* queue backlog grows
* payment provider unavailable
* database connection pool exhausted
* security violation spike
* worker crash loop

Alerting on every `ERROR` log can create alert fatigue.

Better alerting often uses metrics derived from logs or direct metrics.

Still, log levels influence alerting systems.

Use them responsibly.

---

# Logging During Incident Response

During an incident, logs should help answer:

* when did the problem start?
* what changed?
* which users or operations are affected?
* which component is failing?
* is the failure increasing or decreasing?
* are retries succeeding?
* did mitigation work?

Incident-friendly logs have:

* timestamps
* levels
* service names
* versions
* request IDs
* entity IDs
* error types
* duration fields

Logs that say:

```text
oops
```

are not incident-friendly.

Write logs for the tired engineer at 3 a.m.

That engineer may be you.

---

# Common Logging Mistakes

Common mistakes include:

* using `print` everywhere in production code
* logging secrets
* logging full raw payloads
* logging too much at `INFO`
* logging expected business outcomes as `ERROR`
* swallowing exceptions after logging
* losing tracebacks
* duplicate exception logging
* configuring logging inside libraries
* attaching handlers at multiple hierarchy levels
* inconsistent field names
* logs without correlation IDs
* vague messages like `failed`
* logs that cannot be searched reliably
* logging every loop iteration
* never testing important audit logs

These mistakes are common because logging feels easy.

Good logging is not hard because the API is hard.

It is hard because the judgment matters.

---

# A Logging Checklist

Before adding a log, ask:

* What event happened?
* Who will use this log?
* What level should it be?
* What context will help debugging?
* Is a correlation ID available?
* Could this log expose sensitive data?
* Could this log be emitted too often?
* Is the message stable and searchable?
* Should this be a metric instead?
* Should this be an audit event instead?
* Is this exception logged with traceback?
* Will this duplicate another log?

Good logs are intentional.

They are not random strings scattered through code.

---

# A Practical Logging Setup

For a small application, a practical setup might be:

```python
import logging
import logging.config


def configure_logging(level="INFO"):
    logging.config.dictConfig(
        {
            "version": 1,
            "disable_existing_loggers": False,
            "formatters": {
                "default": {
                    "format": "%(asctime)s %(levelname)s %(name)s %(message)s",
                },
            },
            "handlers": {
                "console": {
                    "class": "logging.StreamHandler",
                    "formatter": "default",
                },
            },
            "root": {
                "handlers": ["console"],
                "level": level,
            },
        }
    )
```

Then modules use:

```python
logger = logging.getLogger(__name__)
```

Application entry point:

```python
def main():
    configure_logging()
    logger.info("application_started")
    run()
```

This is not enough for every production system.

But it shows the separation:

```text
application configures
modules emit
```

---

# A Good Logging Example

Consider a background import job:

```python
def run_import(job, importer):
    logger.info(
        "import_started job_id=%s file_name=%s",
        job.id,
        job.file_name,
    )

    try:
        result = importer.import_file(job.path)
    except ValidationError as error:
        logger.warning(
            "import_rejected job_id=%s file_name=%s reason=%s",
            job.id,
            job.file_name,
            error.code,
        )
        raise
    except Exception:
        logger.exception(
            "import_failed job_id=%s file_name=%s",
            job.id,
            job.file_name,
        )
        raise

    logger.info(
        "import_finished job_id=%s inserted=%s updated=%s failed=%s",
        job.id,
        result.inserted,
        result.updated,
        result.failed,
    )
```

This is useful because it records:

* start
* expected rejection
* unexpected failure with traceback
* successful summary
* job ID
* file name
* counts

It does not log every row.

It does not swallow exceptions.

It preserves context.

---

# A Poor Logging Example

Poor version:

```python
def run_import(job, importer):
    print("starting")
    try:
        result = importer.import_file(job.path)
    except Exception as error:
        print("failed", error)
        return None
    print("done")
    return result
```

Problems:

* uses `print`
* no job ID
* no file context
* no traceback
* catches all exceptions
* swallows the failure
* no result summary
* messages are not searchable

This code may appear simpler.

It is much worse during an incident.

---

# Chapter Summary

Logging is how running programs leave evidence.

Debugging investigates failure.

Logging preserves information for that investigation.

Python's `logging` module is built around loggers, log records, levels, handlers, formatters, filters, and configuration.

Most modules should use:

```python
logger = logging.getLogger(__name__)
```

Applications should configure logging near the entry point.

Libraries should emit logs but usually should not configure logging.

Levels communicate operational meaning:

* `DEBUG` for detailed diagnostics
* `INFO` for normal meaningful events
* `WARNING` for unexpected but handled conditions
* `ERROR` for failed operations
* `CRITICAL` for severe system-level failure

Good logs describe events, not random code locations.

Stable event names and consistent fields make logs searchable.

Correlation IDs connect logs across a request, job, or workflow.

Exception logs should preserve tracebacks with `logger.exception` or `exc_info=True`.

Do not log and swallow failures unless you truly handled them.

Avoid duplicate exception logs across layers.

Use lazy formatting instead of eager f-strings for ordinary log messages.

Never log secrets.

Be deliberate with personal data and raw payloads.

Redaction, masking, and safe summaries are part of responsible logging.

Structured logs are often better for production search and aggregation.

Logs, metrics, and traces are different observability signals and should complement one another.

The central lesson is:

```text
good logs make future debugging possible
```

They are not noise.

They are operational memory.

---

# Exercises

1. Replace `print` statements in a small script with module-level logging.

2. Configure logging with `basicConfig` using timestamp, level, logger name, and message.

3. Write a function that logs start, success, and failure for a background job.

4. Use `logger.exception` inside an exception handler and verify that a traceback is included.

5. Rewrite a vague log message such as `failed` into a stable event name with useful context.

6. Add a request ID or job ID to related log messages.

7. Identify one value that should never be logged in an application and write a redaction helper for it.

8. Use pytest's `caplog` fixture to test an important warning log.

9. Create a `dictConfig` logging setup with a console handler.

10. Review a loop that logs too often and replace it with a summary log.

---

# Preview of Chapter 76

Chapter 75 studied logging.

We learned how Python's logging system works and how to design logs that help real debugging, production operations, security review, and incident response.

Next we study packaging.

Packaging is how Python code becomes something other people and systems can install, run, and depend on.

It connects:

* modules
* packages
* project layout
* dependencies
* metadata
* build systems
* wheels
* source distributions
* virtual environments
* command-line entry points
* versioning
* publishing

Testing, debugging, and logging help make code reliable.

Packaging helps make code deliverable.

The transition is:

```text
logging helps you operate software
packaging helps you distribute software
```

Chapter 76 will show how Python projects are shaped, built, installed, and shared in professional environments.
