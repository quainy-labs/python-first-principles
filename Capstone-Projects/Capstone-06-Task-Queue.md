# Capstone 06 - Task Queue

## Project Brief

In this project, you will build a small task queue.

A task queue is a system that accepts work now and performs that work later.

Instead of doing slow, unreliable, or expensive work inside the main request path, the application records a job and lets a worker process execute it in the background.

The final project will support:

```text
enqueue a job
store the job durably
pick the job from a queue
execute the job in a worker
mark the job as succeeded or failed
retry failed jobs
schedule jobs for later
inspect job state
shut workers down cleanly
```

This is not a production replacement for Celery, RQ, Dramatiq, Huey, Sidekiq, SQS, RabbitMQ, Redis Streams, Kafka, or a cloud queue.

It is a learning implementation.

The point is to understand the moving pieces that make background job systems work.

By the end, the reader should understand why task queues are useful, why they are harder than they look, and how Python can be used to build a reliable background processing system from ordinary components.

---

# Why This Project Matters

Most beginner applications do work immediately.

A user clicks a button.

The server handles the request.

The server does all the work before sending a response.

That works for small operations.

It does not work well for slow operations.

Imagine a web application where a user uploads a video.

The application may need to:

```text
store the upload
generate thumbnails
scan for viruses
transcode the video
extract metadata
send a notification
update a search index
bill usage
write analytics events
```

Doing all of that during the HTTP request is a bad experience.

The user waits too long.

The request may time out.

If one step fails, the whole operation becomes unclear.

If the server restarts, the work may disappear.

A task queue changes the shape of the program.

The request handler does only the immediate part:

```text
accept input
validate it
store essential state
enqueue background work
return a response
```

Then a worker does the expensive part separately.

This gives the application room to breathe.

The user does not wait for every slow operation.

The worker can retry failed jobs.

More workers can be added when there is more work.

Jobs can be inspected and recovered.

Task queues appear in many real systems:

```text
sending emails
processing payments
resizing images
generating reports
running machine learning jobs
syncing third-party APIs
publishing webhooks
cleaning old data
updating caches
executing scheduled maintenance
```

Once you understand task queues, many production systems become easier to reason about.

You begin to see that a web application is not only request handlers and database tables.

It is also background work, retries, coordination, failure handling, and observability.

---

# The Mental Model

A task queue has three central ideas.

The first idea is a job.

A job is a stored description of work.

It says:

```text
what function should run
what arguments should be passed
what state the job is in
when it should run
how many times it has been attempted
what happened last time it ran
```

The second idea is a queue.

A queue is a place where jobs wait.

The queue gives workers a way to find the next job.

The third idea is a worker.

A worker is a long-running process that repeatedly:

```text
asks for work
claims one job
executes it
records the result
looks for the next job
```

The simple picture is:

```text
producer -> queue -> worker
```

The producer creates jobs.

The queue stores jobs.

The worker consumes jobs.

This project will make that picture real.

---

# What You Will Build

You will build a Python package named `miniqueue`.

The package will contain:

```text
miniqueue/
    __init__.py
    app.py
    backend.py
    jobs.py
    registry.py
    worker.py
    cli.py
tests/
    test_jobs.py
    test_backend.py
    test_worker.py
    test_retries.py
```

The public API should feel small:

```python
from miniqueue import Queue

queue = Queue("jobs.sqlite3")

job = queue.enqueue("send_email", to="reader@example.com", subject="Hello")

print(job.id)
```

A worker should be able to register task functions:

```python
from miniqueue import Queue, task

queue = Queue("jobs.sqlite3")

@task("send_email")
def send_email(to: str, subject: str) -> None:
    print(f"Sending {subject} to {to}")

queue.worker().run()
```

The queue will use SQLite for persistence.

That choice is intentional.

SQLite is part of Python's standard library through `sqlite3`.

It gives the project durable state without requiring Redis, RabbitMQ, Docker, or cloud services.

The project will focus on the mechanics of a queue rather than infrastructure setup.

---

# Requirements

The task queue must support durable jobs.

If the Python process exits after a job is enqueued, the job must still exist in SQLite.

The queue must support job states:

```text
queued
running
succeeded
failed
retrying
dead
```

The queue must support delayed jobs.

A delayed job is inserted now but should not run until a later time.

The queue must support retries.

If a job fails, the queue should decide whether it should run again or move to a final failed state.

The queue must store enough information to debug a job:

```text
task name
arguments
keyword arguments
state
attempt count
maximum attempts
last error
created time
scheduled time
started time
finished time
```

The worker must claim one job at a time.

The worker must not execute a job that is scheduled for the future.

The worker must not execute jobs in final states.

The worker must handle unknown task names.

The CLI must allow a reader to enqueue, run, inspect, and list jobs.

The tests must cover the core behavior without relying on sleep-heavy timing tests.

---

# Non-Requirements

The queue does not need distributed locking across many machines.

The queue does not need Redis.

The queue does not need a web dashboard.

The queue does not need cron expression parsing.

The queue does not need recurring jobs.

The queue does not need task priorities in the first version.

The queue does not need workflow graphs.

The queue does not need result storage for large return values.

The queue does not need to run untrusted code safely.

These omissions are deliberate.

A learning queue should be small enough that the reader can hold the whole design in their head.

---

# The First Design Decision

The most important design decision is whether a job stores executable code or a reference to executable code.

Do not store executable code.

Store a task name.

For example:

```json
{
  "task_name": "send_email",
  "args": [],
  "kwargs": {
    "to": "reader@example.com",
    "subject": "Hello"
  }
}
```

The worker looks up `send_email` in a registry.

That registry maps names to Python callables.

This is safer and clearer than serializing Python functions.

It also mirrors how real systems often work.

The producer and worker agree on names.

The queue stores names and data.

The worker imports code and executes it.

---

# Job State

A task queue is mostly a state machine.

The job moves through states.

A new job begins as `queued`.

When a worker claims it, it becomes `running`.

If the task finishes, the job becomes `succeeded`.

If the task raises an exception, the worker records the error.

If attempts remain, the job becomes `retrying`.

When its retry delay expires, it becomes eligible again.

If attempts are exhausted, the job becomes `dead`.

A simple transition diagram looks like this:

```text
queued -> running -> succeeded
                 \
                  -> retrying -> running
                              \
                               -> dead
```

The `failed` state is useful for recording a single failed attempt.

In a small implementation, the final state can be called `dead`.

This keeps the distinction clear:

```text
failed attempt -> something went wrong this time
dead job       -> the queue has stopped retrying
```

You may choose to store only final states in the job table.

But the book project should explain the difference because it matters in production systems.

---

# Data Model

The SQLite table should start simple.

```sql
CREATE TABLE IF NOT EXISTS jobs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_name TEXT NOT NULL,
    args_json TEXT NOT NULL,
    kwargs_json TEXT NOT NULL,
    state TEXT NOT NULL,
    attempts INTEGER NOT NULL,
    max_attempts INTEGER NOT NULL,
    run_at TEXT NOT NULL,
    created_at TEXT NOT NULL,
    started_at TEXT,
    finished_at TEXT,
    last_error TEXT
);
```

This table stores all the essential information.

`task_name` identifies the task function.

`args_json` stores positional arguments.

`kwargs_json` stores keyword arguments.

`state` tells the worker what may happen next.

`attempts` counts how many times the job has been attempted.

`max_attempts` limits retry behavior.

`run_at` controls scheduling.

`created_at` records when the job was inserted.

`started_at` records when a worker started the current or latest attempt.

`finished_at` records when the job reached a final state.

`last_error` stores useful debugging information.

This is not every field a production queue needs.

But it is enough to understand the core.

---

# Time Representation

Time is easy to mishandle.

This project should store timestamps as timezone-aware UTC ISO strings.

For example:

```python
from datetime import datetime, timezone

def utc_now() -> datetime:
    return datetime.now(timezone.utc)
```

When storing:

```python
now.isoformat()
```

When reading:

```python
datetime.fromisoformat(value)
```

The project should avoid naive local datetimes.

Naive datetimes create confusion when code runs on different machines, time zones, or deployment environments.

The reader has already learned enough Python by this point to appreciate why explicit time handling matters.

---

# Serialization

Jobs need to store arguments.

The simplest approach is JSON.

```python
import json

args_json = json.dumps(args)
kwargs_json = json.dumps(kwargs)
```

JSON keeps the system honest.

It allows strings, numbers, booleans, lists, dictionaries, and null.

It does not allow arbitrary Python objects.

That is a good constraint.

If a task queue stores arbitrary Python objects, it quickly becomes hard to debug and unsafe to move across processes.

The project should teach this rule:

```text
Queue messages should contain simple data.
Workers should know how to turn that data into real work.
```

If the task needs a database row, enqueue the row id.

If the task needs a file, enqueue the file path or object key.

If the task needs a user, enqueue the user id.

Do not enqueue a live database connection.

Do not enqueue an open file handle.

Do not enqueue a complex Python object graph.

---

# The Job Object

The Python representation of a job can be a dataclass.

```python
from dataclasses import dataclass
from datetime import datetime
from typing import Any

@dataclass(frozen=True)
class Job:
    id: int
    task_name: str
    args: list[Any]
    kwargs: dict[str, Any]
    state: str
    attempts: int
    max_attempts: int
    run_at: datetime
    created_at: datetime
    started_at: datetime | None = None
    finished_at: datetime | None = None
    last_error: str | None = None
```

The dataclass is frozen.

That means the object returned to user code is a snapshot.

Changing `job.state` directly should not mutate the database.

This separation is useful.

The durable state lives in SQLite.

The Python object is a readable representation of that state.

---

# Constants

Avoid scattering string states across the codebase.

Use constants.

```python
QUEUED = "queued"
RUNNING = "running"
SUCCEEDED = "succeeded"
RETRYING = "retrying"
DEAD = "dead"

ELIGIBLE_STATES = {QUEUED, RETRYING}
FINAL_STATES = {SUCCEEDED, DEAD}
```

This helps prevent typo bugs:

```python
"suceeded"
```

That typo looks small.

In a state machine, it can break job recovery.

---

# The Task Registry

The worker needs to know what function belongs to each task name.

Create a registry.

```python
class TaskRegistry:
    def __init__(self) -> None:
        self._tasks = {}

    def register(self, name, func):
        if name in self._tasks:
            raise ValueError(f"Task already registered: {name}")
        self._tasks[name] = func
        return func

    def get(self, name):
        try:
            return self._tasks[name]
        except KeyError:
            raise UnknownTask(name) from None
```

The decorator can use this registry:

```python
registry = TaskRegistry()

def task(name):
    def decorator(func):
        return registry.register(name, func)
    return decorator
```

Then user code can write:

```python
@task("resize_image")
def resize_image(path: str, width: int) -> None:
    ...
```

The task name becomes part of the queue contract.

The function name can change.

The module can move.

But the registered task name should remain stable once jobs are stored.

---

# The Queue Interface

The queue object should own the backend and expose a small API.

```python
class Queue:
    def __init__(self, path: str) -> None:
        self.backend = SQLiteBackend(path)
        self.backend.initialize()

    def enqueue(
        self,
        task_name: str,
        *args,
        delay_seconds: int = 0,
        max_attempts: int = 3,
        **kwargs,
    ) -> Job:
        ...

    def get(self, job_id: int) -> Job | None:
        ...

    def list_jobs(self, state: str | None = None) -> list[Job]:
        ...

    def worker(self) -> "Worker":
        ...
```

The queue is the friendly entry point.

It should not contain every SQLite detail.

Storage logic belongs in `SQLiteBackend`.

Execution logic belongs in `Worker`.

The queue coordinates those pieces.

---

# The Backend

The backend is responsible for durable state.

It should know how to:

```text
create the jobs table
insert jobs
load one job
list jobs
claim the next available job
mark jobs as succeeded
mark jobs for retry
mark jobs as dead
```

The backend should not execute Python functions.

That belongs to the worker.

Keeping this boundary clear makes the system easier to test.

Backend tests can check database behavior without running arbitrary tasks.

Worker tests can use a fake backend when needed.

---

# Insert Job

Enqueuing a job inserts a row.

The default `run_at` is now.

A delayed job has `run_at` in the future.

```python
def enqueue(task_name, args, kwargs, run_at, max_attempts):
    now = utc_now()
    execute(
        """
        INSERT INTO jobs (
            task_name, args_json, kwargs_json, state,
            attempts, max_attempts, run_at, created_at
        )
        VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        """,
        (
            task_name,
            json.dumps(args),
            json.dumps(kwargs),
            QUEUED,
            0,
            max_attempts,
            run_at.isoformat(),
            now.isoformat(),
        ),
    )
```

The job begins with zero attempts.

It has not run yet.

The worker increments attempts when it starts executing the job.

---

# Claiming Work

Claiming work is the hardest part of the basic queue.

The worker must find one eligible job and mark it as running.

Eligible means:

```text
state is queued or retrying
run_at is now or earlier
```

The naive implementation is:

```python
job = select_next_job()
mark_running(job.id)
```

This is fine for a single worker.

It is risky for multiple workers.

Two workers may select the same job before either marks it running.

That race is the heart of queue design.

For this educational implementation, start with a single-worker guarantee.

Then explain the multi-worker problem.

SQLite can support safer claiming using transactions.

One approach is:

```text
begin immediate transaction
select one eligible job
update that job to running
commit
```

`BEGIN IMMEDIATE` obtains a write lock early.

That prevents another writer from claiming the same job at the same time.

This is not the same as a highly scalable distributed queue.

But it teaches the correct shape:

```text
claiming a job must be atomic
```

The claim operation should return a `Job` or `None`.

If no job is available, the worker waits briefly and checks again.

---

# Polling

A simple worker uses polling.

Polling means the worker repeatedly asks:

```text
is there a job now?
```

If there is a job, it runs it.

If there is no job, it sleeps.

```python
while not should_stop:
    job = backend.claim_next_job()
    if job is None:
        time.sleep(poll_interval)
        continue
    run_job(job)
```

Polling is easy to understand.

It is not always the most efficient design.

Real queues often use blocking reads, notifications, sockets, or broker protocols.

But polling is perfect for this project.

It makes the worker behavior explicit.

---

# Worker Execution

The worker receives a claimed job.

The job is already marked `running`.

The worker looks up the task function.

Then it calls:

```python
func(*job.args, **job.kwargs)
```

If the function returns normally, the worker marks the job as succeeded.

If the function raises, the worker records the exception and decides whether to retry.

The worker should catch `Exception`, not `BaseException`.

```python
try:
    func(*job.args, **job.kwargs)
except Exception as exc:
    handle_failure(job, exc)
else:
    backend.mark_succeeded(job.id)
```

Catching `BaseException` would also catch things like `KeyboardInterrupt` and `SystemExit`.

Those should usually be allowed to stop the worker.

---

# Attempts

Attempts should be incremented when the worker starts running the job.

This means a job that crashes the process during execution still counts as having been attempted only if the database was updated before execution.

That is useful.

The claim operation can set:

```text
state = running
attempts = attempts + 1
started_at = now
```

Then the worker executes the task.

If the process dies midway, the job remains `running`.

That introduces another production concern:

```text
stuck running jobs
```

The first version does not need automatic stuck-job recovery.

But the chapter should explain it.

Production queues often use heartbeats, visibility timeouts, leases, or worker process tracking.

---

# Retry Behavior

Retries are one of the main reasons task queues exist.

A job may fail because:

```text
network request timed out
database was temporarily unavailable
third-party API rate limited the request
file was not ready yet
temporary lock existed
```

These failures may succeed later.

The queue should retry them.

But it should not retry forever.

Use `max_attempts`.

If a job has attempts remaining, schedule it again:

```text
state = retrying
run_at = now + retry_delay
last_error = traceback
```

If no attempts remain:

```text
state = dead
finished_at = now
last_error = traceback
```

The retry delay can be simple at first:

```python
retry_delay_seconds = 5
```

Then teach exponential backoff.

---

# Exponential Backoff

Exponential backoff increases the delay after each failure.

For example:

```text
attempt 1 -> wait 5 seconds
attempt 2 -> wait 10 seconds
attempt 3 -> wait 20 seconds
attempt 4 -> wait 40 seconds
```

The formula can be:

```python
delay = base_delay * (2 ** (attempts - 1))
```

With `base_delay = 5`, this gives:

```text
5, 10, 20, 40
```

Backoff protects external systems.

If an API is temporarily down, immediately retrying thousands of jobs can make the failure worse.

Backoff gives the dependency time to recover.

Production systems often add jitter.

Jitter means adding randomness to avoid all jobs retrying at exactly the same moment.

This capstone can mention jitter as an extension.

---

# Error Recording

A failed job should not merely say:

```text
failed
```

It should record useful context.

At minimum, store the exception type and message.

Better, store the traceback.

```python
import traceback

error = traceback.format_exc()
```

This gives the reader realistic debugging information.

The database field `last_error` can store this string.

The CLI can display it when inspecting a job.

Do not store infinite logs in one row.

For this capstone, the latest error is enough.

---

# Idempotency

Task queues force the reader to think about idempotency.

An operation is idempotent if doing it more than once has the same final effect as doing it once.

For example:

```text
set user status to active
```

is usually idempotent.

Running it twice still leaves the user active.

But:

```text
charge credit card
```

is not automatically idempotent.

Running it twice may charge the customer twice.

Retries make idempotency important.

If a job fails after partially completing its work, the queue may run it again.

That can be dangerous unless the task is designed carefully.

The queue cannot solve every idempotency problem.

The task code must often use:

```text
unique request ids
database constraints
deduplication keys
external idempotency keys
careful state checks
```

This project should teach the rule:

```text
If a task may be retried, design the task so retrying is safe.
```

---

# At-Least-Once Delivery

This mini queue should be described as at-least-once.

At-least-once means a job should run one or more times.

It does not promise exactly once.

Exactly-once execution sounds attractive.

In distributed systems, it is extremely hard and often misleading.

A worker can crash after executing a task but before marking it succeeded.

When the system recovers, it may run the job again.

The queue cannot know whether the side effect happened.

For example:

```text
worker sends email
worker crashes before mark_succeeded
job is retried
email may send again
```

This is why idempotency matters.

The honest contract is:

```text
The queue will try hard not to lose work.
The queue may run work more than once.
Tasks must tolerate retries.
```

This is a major production lesson.

---

# Unknown Tasks

A worker may claim a job whose task name is not registered.

This can happen if:

```text
worker code is outdated
task was renamed
wrong worker process is consuming the queue
deployment is incomplete
job data is invalid
```

The worker must handle this clearly.

It should not crash forever on the same unknown job.

For this capstone, treat unknown task as a job failure.

The worker records:

```text
UnknownTask: send_email
```

Then it applies normal retry rules.

Alternatively, unknown task can immediately mark the job dead.

That is also defensible.

The important part is that the behavior is explicit.

---

# Delayed Jobs

Delayed jobs are jobs scheduled for the future.

The enqueue API can support:

```python
queue.enqueue("send_reminder", user_id=42, delay_seconds=3600)
```

The queue stores:

```text
run_at = now + 3600 seconds
```

The worker claim query checks:

```sql
run_at <= current_time
```

This means the job stays invisible until it becomes due.

Delayed jobs are useful for:

```text
reminders
follow-up emails
retry scheduling
timeouts
cleanup work
```

This project does not need recurring jobs.

A recurring scheduler would insert new delayed jobs on a schedule.

That is a separate layer.

---

# The Claim Query

The claim query should choose the oldest eligible job.

```sql
SELECT *
FROM jobs
WHERE state IN ('queued', 'retrying')
  AND run_at <= ?
ORDER BY run_at ASC, id ASC
LIMIT 1
```

Ordering by `run_at` handles scheduled jobs.

Ordering by `id` gives deterministic behavior for jobs due at the same time.

After selecting the row, update it:

```sql
UPDATE jobs
SET state = 'running',
    attempts = attempts + 1,
    started_at = ?
WHERE id = ?
```

In a transaction, these steps form one claim operation.

The backend can then re-read the job and return it.

---

# SQLite Transactions

SQLite automatically opens transactions in many cases, but this project should be explicit.

For claim operations:

```python
conn.execute("BEGIN IMMEDIATE")
try:
    row = ...
    if row is None:
        conn.commit()
        return None
    conn.execute(...)
    conn.commit()
except:
    conn.rollback()
    raise
```

This is more code than the naive approach.

It is also the moment where the reader sees why queues involve coordination.

The task queue is not only about Python function calls.

It is about changing durable state safely.

---

# Mapping Rows to Jobs

The backend needs a helper to turn SQLite rows into `Job` objects.

```python
def row_to_job(row) -> Job:
    return Job(
        id=row["id"],
        task_name=row["task_name"],
        args=json.loads(row["args_json"]),
        kwargs=json.loads(row["kwargs_json"]),
        state=row["state"],
        attempts=row["attempts"],
        max_attempts=row["max_attempts"],
        run_at=parse_time(row["run_at"]),
        created_at=parse_time(row["created_at"]),
        started_at=parse_optional_time(row["started_at"]),
        finished_at=parse_optional_time(row["finished_at"]),
        last_error=row["last_error"],
    )
```

Use `sqlite3.Row` so columns can be accessed by name.

```python
conn.row_factory = sqlite3.Row
```

This makes the code clearer and less fragile than tuple indexes.

---

# Connection Management

For a small SQLite-backed queue, use short-lived connections per operation.

```python
def connect(self):
    conn = sqlite3.connect(self.path)
    conn.row_factory = sqlite3.Row
    return conn
```

Each backend method can open a connection, perform work, and close it.

That keeps the implementation simple.

For a long-running worker, this may feel less efficient.

But it avoids subtle issues around connection reuse, thread boundaries, and stale transaction state.

Later, the reader can optimize.

In learning software, clarity is more valuable than clever connection pooling.

---

# CLI Shape

The CLI should let the reader use the queue without writing a web app.

Example commands:

```bash
python -m miniqueue enqueue send_email --kw to=reader@example.com --kw subject=Hello
python -m miniqueue list
python -m miniqueue show 1
python -m miniqueue work
```

The CLI can be intentionally modest.

It does not need to parse every Python type.

Keyword values can be strings.

For richer values, the CLI can accept JSON:

```bash
python -m miniqueue enqueue generate_report --json '{"user_id": 42}'
```

The CLI is useful because it makes job state visible.

The reader can enqueue a job, run the worker, and inspect what changed.

---

# Worker Modes

The worker should support two modes.

The first mode is `run_once`.

`run_once` claims at most one job.

It returns whether it did work.

```python
did_work = worker.run_once()
```

This mode is excellent for tests.

Tests should not need infinite loops.

The second mode is `run`.

`run` loops forever until interrupted.

```python
worker.run(poll_interval=1.0)
```

This mode is useful for real command-line use.

Designing both modes gives a clean testing story.

---

# Graceful Shutdown

A worker should stop politely when interrupted.

In a simple CLI, `KeyboardInterrupt` can be caught around the loop.

```python
try:
    worker.run()
except KeyboardInterrupt:
    print("Worker stopped")
```

The worker should not leave a job half-marked because of normal control flow.

If the task itself is interrupted, the behavior is more complicated.

For this capstone, keep shutdown simple.

The chapter should still name the production concern:

```text
What happens to a running job when the worker process dies?
```

That question leads to leases and visibility timeouts.

---

# Visibility Timeout

This capstone does not need to implement visibility timeouts, but it should explain them.

A visibility timeout is a lease.

When a worker claims a job, the job becomes invisible to other workers for a period of time.

If the worker finishes, the job is marked succeeded.

If the worker disappears, the lease expires and another worker may retry the job.

This pattern appears in systems like Amazon SQS.

The mini queue can leave running jobs as running.

An extension exercise can add:

```text
locked_until
worker_id
heartbeat_at
```

That extension would let the queue recover jobs whose workers died.

The main implementation should stay focused.

---

# Priority

This project can mention priority without implementing it.

Priority queues choose important jobs first.

The jobs table could add:

```sql
priority INTEGER NOT NULL DEFAULT 0
```

Then the claim query could order by:

```sql
ORDER BY priority DESC, run_at ASC, id ASC
```

However, priority changes the behavior of fairness.

If high-priority jobs always arrive, low-priority jobs may starve.

That is a valuable discussion, but not necessary for the first implementation.

---

# Result Values

Should the queue store task return values?

For this project, no.

A background task should usually produce durable effects:

```text
write a file
update a row
send a message
store a report
publish an event
```

Return values from worker functions disappear unless explicitly stored.

Production queues sometimes support result backends.

But result backends add complexity:

```text
where results live
how long results are retained
how large results can be
who can read results
what happens when result storage fails
```

This capstone should keep the queue centered on job execution and state.

---

# Testing Philosophy

The tests should not depend on long sleeps.

Slow tests teach the wrong habits.

Instead, make time injectable.

For example:

```python
class Queue:
    def __init__(self, path, clock=utc_now):
        self.backend = SQLiteBackend(path, clock=clock)
```

Tests can pass a fake clock.

```python
current = datetime(2026, 1, 1, tzinfo=timezone.utc)

def fake_clock():
    return current
```

This lets tests create delayed jobs and advance time deliberately.

If that feels too much for the first pass, isolate time functions so they can be monkey patched in tests.

The broader lesson is important:

```text
Time-dependent code should be designed for testing.
```

---

# Unit Tests

Test job serialization.

Create args and kwargs.

Store them.

Load the job.

Assert they round trip correctly.

Test delayed jobs.

Enqueue one job due now.

Enqueue one job due later.

Claim work.

Assert the due job is claimed first.

Test success.

Register a task that records that it ran.

Run one worker step.

Assert the job is succeeded.

Test failure.

Register a task that raises.

Run one worker step.

Assert the job is retrying or dead depending on `max_attempts`.

Test unknown task.

Enqueue a job with a task name that is not registered.

Run one worker step.

Assert the job records a clear error.

Test final jobs are not claimed.

Manually mark a job as succeeded.

Ask for the next job.

Assert it is ignored.

---

# Integration Tests

An integration test should use a temporary SQLite database.

In pytest:

```python
def test_worker_executes_enqueued_job(tmp_path):
    path = tmp_path / "jobs.sqlite3"
    queue = Queue(path)

    seen = []

    @task("record")
    def record(value):
        seen.append(value)

    queue.enqueue("record", "hello")

    worker = queue.worker()
    assert worker.run_once() is True

    assert seen == ["hello"]
    assert queue.get(1).state == "succeeded"
```

This test uses a real database.

It gives confidence that the queue, backend, registry, and worker collaborate correctly.

---

# Suggested Implementation Order

Do not start with workers.

Start with storage.

A good implementation order is:

```text
1. Job dataclass and state constants
2. Time helpers
3. SQLite table creation
4. enqueue and get
5. list jobs
6. task registry
7. claim_next_job
8. worker run_once
9. success handling
10. failure and retry handling
11. delayed jobs
12. CLI
13. documentation and examples
```

This order keeps feedback fast.

The reader can test each layer before moving upward.

---

# Milestone 1 - Job Model

Create the `Job` dataclass.

Create state constants.

Create helpers:

```python
def utc_now() -> datetime:
    ...

def serialize_args(args: tuple[Any, ...]) -> str:
    ...

def serialize_kwargs(kwargs: dict[str, Any]) -> str:
    ...

def parse_time(value: str) -> datetime:
    ...
```

Write tests for serialization.

Make sure invalid JSON-unsafe values fail clearly.

For example, this should not silently work:

```python
queue.enqueue("task", object())
```

The reader can catch `TypeError` from `json.dumps` and wrap it in a clearer error.

---

# Milestone 2 - SQLite Backend

Implement `SQLiteBackend.initialize`.

It should create the jobs table.

Then implement `enqueue`.

Then implement `get`.

At this stage, no worker exists.

A test can still verify:

```text
enqueue returns a Job
job has an id
job is queued
job has attempts = 0
job can be loaded again
args and kwargs round trip
```

This milestone proves durable storage.

---

# Milestone 3 - Task Registry

Implement the registry.

Support:

```python
@task("name")
def func(...):
    ...
```

Test:

```text
registered task can be found
duplicate names fail
unknown names fail
decorator returns original callable
```

Returning the original callable matters because decorators should not break normal function use.

---

# Milestone 4 - Claiming Jobs

Implement `claim_next_job`.

It should:

```text
find one eligible job
mark it running
increment attempts
set started_at
return the claimed job
```

Test:

```text
queued job can be claimed
future job cannot be claimed
succeeded job cannot be claimed
oldest due job is claimed first
attempt count increments
```

This milestone is the technical center of the project.

Take time with it.

---

# Milestone 5 - Worker Success

Implement `Worker.run_once`.

For a successful task:

```text
claim job
look up task
call function
mark succeeded
return True
```

If no job exists:

```text
return False
```

Tests should assert both paths.

Do not build the infinite loop yet.

First make one step correct.

---

# Milestone 6 - Worker Failure and Retries

Add failure handling.

When a task raises:

```text
format traceback
if attempts < max_attempts:
    mark retrying
    set run_at to future retry time
else:
    mark dead
    set finished_at
```

Be careful about attempts.

If the claim operation increments attempts before running, then after the first failure `attempts` is already `1`.

The retry condition is:

```python
if job.attempts < job.max_attempts:
    retry
else:
    dead
```

This means `max_attempts=1` tries once and then dies on failure.

That is intuitive.

---

# Milestone 7 - Worker Loop

Add `Worker.run`.

It should call `run_once` repeatedly.

If `run_once` returns `False`, sleep for `poll_interval`.

```python
def run(self, poll_interval=1.0):
    while True:
        did_work = self.run_once()
        if not did_work:
            time.sleep(poll_interval)
```

For testability, do not test this infinite loop directly.

The CLI can use it.

The core behavior is already covered by `run_once`.

---

# Milestone 8 - CLI

Use `argparse`.

Commands:

```text
enqueue
list
show
work
```

The queue database path can come from an option:

```bash
python -m miniqueue --db jobs.sqlite3 list
```

For `enqueue`, accept a task name and JSON payload:

```bash
python -m miniqueue --db jobs.sqlite3 enqueue send_email --kwargs '{"to": "a@example.com"}'
```

The CLI should print useful output:

```text
Enqueued job 12: send_email
```

For `show`, print:

```text
id
task name
state
attempts
run_at
created_at
last_error
```

This makes the queue observable.

---

# Common Mistake 1 - Executing Jobs During Enqueue

Enqueue should not execute work.

This is wrong:

```python
def enqueue(task_name, *args, **kwargs):
    registry.get(task_name)(*args, **kwargs)
```

That is not a queue.

That is a direct function call.

Enqueue should store work.

The worker should execute work.

This separation is the whole point.

---

# Common Mistake 2 - Losing Exceptions

Do not write:

```python
except Exception:
    mark_dead(job.id)
```

That destroys the most important debugging information.

Always record the error.

At minimum:

```python
except Exception as exc:
    mark_failed(job.id, repr(exc))
```

Better:

```python
traceback.format_exc()
```

A failed job without an error is a locked door with no key.

---

# Common Mistake 3 - Infinite Retries

Retries without limits are dangerous.

A permanently broken job will run forever.

It can fill logs.

It can hammer an external service.

It can hide the fact that human attention is needed.

Always have a maximum attempt count.

Move exhausted jobs to a final state.

Final failed jobs are not a failure of the queue.

They are a signal that the system tried and stopped safely.

---

# Common Mistake 4 - Retrying Too Quickly

Immediate retries can be worse than no retries.

If a third-party API is down, retrying every failed job immediately creates more pressure.

Use a delay.

Even a simple five-second delay teaches the right model.

Backoff is better.

Backoff with jitter is better still.

---

# Common Mistake 5 - Assuming Exactly Once

Do not tell the reader that the queue executes jobs exactly once.

It does not.

A simple durable queue should be understood as at-least-once.

That truth affects task design.

If the task sends an email, it may need a deduplication key.

If the task charges money, it must use an external idempotency key.

If the task writes a database row, it may need a unique constraint.

The queue and task code must cooperate.

---

# Common Mistake 6 - Hiding Job State

A task queue without inspection tools is frustrating.

When something fails, the developer needs to know:

```text
what job failed
what task it was
what arguments it had
how many attempts happened
what error occurred
when it is scheduled again
```

That is why the CLI matters.

Even a small `list` and `show` command makes the queue feel real.

---

# Extension Ideas

Add priority.

Add named queues:

```text
email
reports
images
default
```

Add worker names.

Add `locked_until`.

Add stuck job recovery.

Add a result table.

Add job cancellation.

Add rate limiting.

Add concurrency using threads.

Add multiprocessing workers.

Add a small FastAPI dashboard.

Add metrics:

```text
queued count
running count
succeeded count
dead count
average runtime
retry count
```

Add dead job requeue:

```bash
python -m miniqueue retry 42
```

Add task-specific retry configuration.

Add JSON schema validation for payloads.

Each extension teaches a real production concern.

But each extension should be added only after the basic queue is understandable.

---

# What This Project Teaches

This project brings together many parts of Python and software engineering.

It uses dataclasses for structured job objects.

It uses JSON for message serialization.

It uses SQLite for durable persistence.

It uses transactions for safe state changes.

It uses decorators for task registration.

It uses exceptions and tracebacks for failure handling.

It uses loops and time handling for workers.

It uses CLI design for observability.

It uses testing techniques for time-dependent systems.

It teaches the difference between synchronous work and asynchronous work.

It teaches why retries are powerful and dangerous.

It teaches why idempotency is not optional.

It teaches why production systems prefer explicit state machines.

The reader should leave this project with a new mental model:

```text
Background work is not magic.
It is durable state plus workers plus careful transitions.
```

---

# Completion Checklist

The project is complete when:

```text
Jobs can be enqueued.
Jobs are stored in SQLite.
Jobs survive process restart.
Jobs can be listed.
Jobs can be inspected.
Workers can register task functions.
Workers can claim one eligible job.
Workers execute successful jobs.
Successful jobs become succeeded.
Failed jobs record errors.
Failed jobs retry when attempts remain.
Exhausted jobs become dead.
Delayed jobs do not run early.
Unknown tasks are handled clearly.
The worker has run_once for tests.
The worker has run for CLI usage.
The CLI can enqueue jobs.
The CLI can list jobs.
The CLI can show one job.
The CLI can run a worker.
Tests cover storage, claiming, success, failure, retry, and delay behavior.
Documentation explains at-least-once execution and idempotency.
```

When all of these are true, the reader has built a real miniature background job system.

---

# Exercises

1. Create the `miniqueue` package structure.

2. Implement the `Job` dataclass.

3. Add state constants.

4. Implement UTC time helpers.

5. Implement JSON serialization for args and kwargs.

6. Create the SQLite jobs table.

7. Implement enqueue.

8. Implement get by job id.

9. Implement list jobs.

10. Add tests for persistence.

11. Implement the task registry.

12. Implement the `@task` decorator.

13. Add tests for duplicate task names.

14. Implement `claim_next_job`.

15. Add tests for delayed jobs.

16. Implement `Worker.run_once`.

17. Add tests for successful task execution.

18. Add failure handling.

19. Add retry handling with backoff.

20. Add tests for exhausted jobs.

21. Add CLI command `enqueue`.

22. Add CLI command `list`.

23. Add CLI command `show`.

24. Add CLI command `work`.

25. Document idempotency and at-least-once execution.

---

# Preview of Capstone 07

Capstone 06 built a Task Queue.

It introduced durable jobs, background workers, retries, scheduling, state transitions, idempotency, at-least-once execution, and operational thinking.

Capstone 07 will build a Mini Redis.

The Mini Redis project will move from durable background work to an in-memory networked data store.

It will introduce sockets, protocols, command parsing, key-value storage, expiration, persistence options, concurrency, and server design.

The transition is:

```text
Task Queue coordinates work through persistent job state
Mini Redis coordinates clients through a networked in-memory store
```

The next project will make Python listen on the network and speak a small database protocol.
