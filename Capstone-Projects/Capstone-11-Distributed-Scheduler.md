# Capstone 11 - Distributed Scheduler

## Project Brief

In this project, you will build a small distributed scheduler.

A scheduler decides when work should run.

A distributed scheduler decides when work should run when there may be many workers, many scheduled jobs, process crashes, retries, clocks, locks, leases, and persistent state involved.

This capstone is intentionally the final project.

It combines ideas from the whole book:

```text
data modeling
databases
APIs
background workers
time handling
concurrency
retries
idempotency
logging
testing
observability
failure recovery
architecture
operational thinking
```

The final project will support:

```text
creating scheduled jobs
storing schedules durably
claiming due jobs with leases
running jobs on worker processes
worker heartbeats
retrying failed runs
marking exhausted jobs
recovering abandoned leases
listing jobs and runs
pausing and resuming schedules
recording execution history
exposing a small HTTP API
testing scheduling, claiming, retries, leases, and recovery
```

This project is not a production replacement for Airflow, Celery Beat, Temporal, Dagster, Prefect, Kubernetes CronJobs, Sidekiq Enterprise, Quartz, Argo Workflows, or cloud schedulers.

It is a learning implementation.

The goal is to understand how scheduled work is coordinated safely when many things can go wrong.

By the end, the reader should understand why distributed scheduling is not just a timer, why leases are safer than naive locks, why jobs must be idempotent, how workers prove they are alive, and how durable state makes recovery possible.

---

# Why This Project Matters

Many real systems need work to happen later.

Examples:

```text
send a reminder email tomorrow
run a report every morning
sync billing data every hour
clean old sessions every night
retry failed webhooks
refresh machine learning features
expire temporary accounts
process subscriptions at renewal time
rebuild search indexes
publish scheduled content
```

At first, scheduling sounds simple.

The program can sleep until the right time.

Then it can run the job.

That works for one process and one job.

It breaks when the system grows.

What if the process restarts?

What if two workers run the same job?

What if a worker claims a job and then dies?

What if a job succeeds but the worker crashes before recording success?

What if the database is temporarily unavailable?

What if a retry should happen in five minutes?

What if an operator needs to know what happened yesterday?

These are not edge cases.

They are normal production realities.

A distributed scheduler exists because real scheduled work needs coordination, persistence, recovery, and visibility.

---

# The Mental Model

A distributed scheduler has five main pieces.

The first piece is a schedule.

A schedule describes work that should happen at one or more times.

Example:

```text
run "send_daily_summary" every day at 09:00 UTC
```

The second piece is a run.

A run is one concrete execution attempt created from a schedule.

Example:

```text
daily summary for 2026-06-15 09:00 UTC
```

The third piece is a worker.

A worker claims due runs and executes them.

The fourth piece is a lease.

A lease is a temporary claim that says:

```text
worker-7 owns this run until this deadline
```

The fifth piece is history.

History records what happened:

```text
created
claimed
started
succeeded
failed
retried
expired lease
dead
```

The core flow is:

```text
schedule definition
    -> due run generated
    -> worker claims run with lease
    -> worker executes task
    -> result recorded
    -> next run scheduled if needed
```

The scheduler is not a single timer.

It is a state machine backed by durable storage.

---

# What You Will Build

You will build a package named `distsched`.

Suggested structure:

```text
distsched/
    __init__.py
    models.py
    storage.py
    schedules.py
    registry.py
    worker.py
    api.py
    cli.py
    clock.py
    errors.py
tests/
    test_schedules.py
    test_storage.py
    test_claiming.py
    test_worker.py
    test_retries.py
    test_leases.py
    test_api.py
```

The public API should feel like this:

```python
from distsched import Scheduler, task

scheduler = Scheduler("scheduler.sqlite3")

@task("send_summary")
def send_summary(account_id: int) -> None:
    ...

scheduler.create_schedule(
    name="daily-summary",
    task_name="send_summary",
    kwargs={"account_id": 42},
    interval_seconds=86400,
)

scheduler.worker(worker_id="worker-1").run()
```

The scheduler will use SQLite for the learning implementation.

SQLite keeps setup simple while still requiring real transactions.

The design should explain which parts would change in PostgreSQL or another production database.

---

# Requirements

The scheduler must support durable schedules.

It must support durable runs.

It must support one-time schedules and interval schedules.

It must create runs when schedules become due.

It must claim due runs atomically.

It must use leases instead of permanent locks.

It must record worker heartbeats.

It must retry failed runs when attempts remain.

It must mark exhausted runs as dead.

It must recover runs whose leases expire.

It must pause and resume schedules.

It must record run history.

It must expose a small HTTP API or CLI for inspection.

It must include tests that use an injectable clock.

It must document at-least-once execution and idempotency.

---

# Non-Requirements

The scheduler does not need Kubernetes.

It does not need true multi-node deployment for the learning version.

It does not need cron expression support in the first version.

It does not need DAG workflows.

It does not need dependency graphs.

It does not need distributed consensus.

It does not need exactly-once execution.

It does not need a full web dashboard.

It does not need authentication.

It does not need high availability.

It does not need millions of jobs.

These omissions are deliberate.

The project should teach the foundation before adding industrial complexity.

---

# Scheduler Versus Queue

The Task Queue capstone accepted jobs and ran them later.

A scheduler is related, but different.

A queue usually answers:

```text
what work is waiting now?
```

A scheduler answers:

```text
what work should become runnable at a specific time?
```

The scheduler may produce queue-like runs.

But it also owns schedule definitions, next run times, recurring behavior, lease recovery, and execution history.

The relationship is:

```text
schedule -> run -> worker execution
```

The task queue starts from a job.

The scheduler starts from time.

---

# Data Model

Use separate tables for schedules, runs, workers, and events.

This separation matters.

A schedule is a definition.

A run is an instance.

A worker is an executor.

An event is history.

Suggested tables:

```text
schedules
runs
workers
run_events
```

This is more structure than the task queue capstone.

That is intentional.

The final capstone should feel closer to a production system.

---

# Schedules Table

The `schedules` table can store:

```sql
CREATE TABLE schedules (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    task_name TEXT NOT NULL,
    args_json TEXT NOT NULL,
    kwargs_json TEXT NOT NULL,
    kind TEXT NOT NULL,
    interval_seconds INTEGER,
    run_at TEXT,
    next_run_at TEXT,
    paused INTEGER NOT NULL DEFAULT 0,
    max_attempts INTEGER NOT NULL DEFAULT 3,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL
);
```

`kind` can be:

```text
once
interval
```

For one-time schedules, `run_at` is the desired time.

For interval schedules, `interval_seconds` controls recurrence.

`next_run_at` says when the scheduler should create the next run.

`paused` prevents new runs from being created.

The task payload is stored as JSON, just like in the task queue capstone.

---

# Runs Table

The `runs` table can store:

```sql
CREATE TABLE runs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    schedule_id INTEGER NOT NULL,
    task_name TEXT NOT NULL,
    args_json TEXT NOT NULL,
    kwargs_json TEXT NOT NULL,
    state TEXT NOT NULL,
    scheduled_for TEXT NOT NULL,
    attempts INTEGER NOT NULL DEFAULT 0,
    max_attempts INTEGER NOT NULL,
    lease_owner TEXT,
    lease_expires_at TEXT,
    started_at TEXT,
    finished_at TEXT,
    last_error TEXT,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL,
    FOREIGN KEY(schedule_id) REFERENCES schedules(id)
);
```

Run states:

```text
pending
leased
running
succeeded
retrying
dead
cancelled
```

The distinction between `leased` and `running` can be subtle.

For a small implementation, claiming can move directly to `running`.

But keeping `leased` in the state model helps explain the lifecycle.

The important part is that a run has a temporary owner and a lease expiration.

---

# Workers Table

Workers should record heartbeats.

```sql
CREATE TABLE workers (
    id TEXT PRIMARY KEY,
    started_at TEXT NOT NULL,
    heartbeat_at TEXT NOT NULL,
    status TEXT NOT NULL
);
```

Worker states:

```text
active
stopping
stale
stopped
```

The worker periodically updates `heartbeat_at`.

This gives operators a way to know which workers are alive.

It also gives the system a clue when claimed work may need recovery.

The lease deadline is still the authority for run recovery.

Heartbeats are observability and supporting evidence.

---

# Events Table

Run events provide history.

```sql
CREATE TABLE run_events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    run_id INTEGER NOT NULL,
    event_type TEXT NOT NULL,
    message TEXT,
    created_at TEXT NOT NULL,
    FOREIGN KEY(run_id) REFERENCES runs(id)
);
```

Events can include:

```text
created
claimed
started
succeeded
failed
retry_scheduled
lease_expired
marked_dead
cancelled
```

The latest state is stored on the run.

The event table explains how it got there.

This is a practical observability pattern.

State answers:

```text
where is it now?
```

Events answer:

```text
how did it get there?
```

---

# Time Handling

Use timezone-aware UTC datetimes for persisted timestamps.

Use an injectable clock.

```python
from datetime import datetime, timezone

def utc_now():
    return datetime.now(timezone.utc)
```

Tests should use a fake clock.

Time is central to this project.

If time is hard to test, the scheduler will be hard to trust.

The clock should be passed into storage, scheduler, and worker components.

That lets tests create schedules, advance time, generate due runs, and verify retries without real waiting.

---

# Creating Runs

The scheduler must turn due schedules into runs.

For each active schedule where:

```text
paused = false
next_run_at <= now
```

create a run.

Then update the schedule's `next_run_at`.

For a one-time schedule:

```text
next_run_at = null
```

or mark the schedule complete.

For an interval schedule:

```text
next_run_at = previous_next_run_at + interval_seconds
```

Using the previous scheduled time avoids drift.

If a run is due at 09:00 and generated at 09:05, the next run should usually be 10:00, not 10:05.

This is an important scheduler design decision.

---

# Catch-Up Policy

What happens if the scheduler is offline for one hour?

Suppose a schedule runs every minute.

On restart, should it create 60 missed runs?

There is no universal answer.

Possible policies:

```text
skip missed runs and schedule the next future run
create one catch-up run
create all missed runs
```

For this capstone, use one catch-up run by default.

That means if a schedule is overdue, create one run and advance `next_run_at` until it is in the future.

This prevents a flood of old work.

Document the behavior.

Schedulers must have explicit missed-run policies.

---

# Atomic Run Creation

Run creation must be transactional.

If the scheduler creates a run but fails before updating `next_run_at`, it may create the same run again.

Use a transaction:

```text
begin
select due schedules
for each due schedule:
    insert run
    update schedule next_run_at
commit
```

In SQLite, use explicit transactions.

In a production database, you would also think about row locks, uniqueness constraints, and concurrent scheduler instances.

For the learning version, one scheduler loop is acceptable.

But the chapter should explain the duplicate-run problem.

---

# Claiming Runs

Workers claim runs when they are due and not already owned.

Eligible runs:

```text
state in pending or retrying
scheduled_for <= now
lease_owner is null
```

Or runs with expired leases:

```text
state in leased or running
lease_expires_at <= now
```

Claiming should set:

```text
state = running
lease_owner = worker_id
lease_expires_at = now + lease_seconds
attempts = attempts + 1
started_at = now
updated_at = now
```

This must be atomic.

Two workers must not claim the same run.

In SQLite, use a transaction with `BEGIN IMMEDIATE`.

The principle is:

```text
selecting work and marking ownership must happen as one critical operation
```

---

# Leases

A lease is a temporary right to work on something.

It is not a permanent lock.

That difference matters.

A lock can be abandoned forever if the holder dies.

A lease expires.

If a worker claims a run until 10:00 and dies at 09:59, another worker can recover the run after 10:00.

Lease fields:

```text
lease_owner
lease_expires_at
```

Worker behavior:

```text
claim run
execute task
finish before lease expires
or extend lease while running
```

For short tasks, a fixed lease duration is enough.

For long tasks, workers need lease renewal.

This project should implement fixed leases first and explain renewal as an extension.

---

# Heartbeats

Workers should heartbeat periodically.

Heartbeat means:

```text
I am alive at this time
```

The worker updates its row:

```sql
UPDATE workers
SET heartbeat_at = ?, status = 'active'
WHERE id = ?
```

Heartbeats help answer:

```text
which workers are currently alive?
which workers have gone stale?
how many workers are processing jobs?
when did a worker last check in?
```

Heartbeats do not replace leases.

A worker can heartbeat but still lose a lease if it runs too long.

A worker can die and stop heartbeating, but the run should be recoverable through lease expiration.

Use both concepts clearly.

---

# Running Tasks

Task execution should reuse the task registry idea from the Task Queue capstone.

Store task names and JSON payloads.

Workers register Python functions.

```python
@task("send_email")
def send_email(to: str, subject: str) -> None:
    ...
```

The worker claims a run.

It looks up the task name.

It calls:

```python
func(*args, **kwargs)
```

On success:

```text
state = succeeded
finished_at = now
lease_owner = null
lease_expires_at = null
```

On failure:

```text
record error
retry or mark dead
clear lease
```

---

# Retry Policy

Retries should be explicit.

For a failed run:

```text
if attempts < max_attempts:
    state = retrying
    scheduled_for = now + retry_delay
else:
    state = dead
```

Use exponential backoff:

```python
delay = base_delay * (2 ** (attempts - 1))
```

Optional max delay:

```python
delay = min(delay, max_delay)
```

Retries make the scheduler resilient to temporary failures.

They also make idempotency necessary.

The scheduler must never promise exactly-once execution.

---

# At-Least-Once Execution

This scheduler provides at-least-once execution.

That means a run may execute more than once.

Example:

```text
worker claims run
worker performs side effect
worker crashes before recording success
lease expires
another worker retries the run
side effect may happen again
```

This is unavoidable in many distributed systems.

The honest contract is:

```text
the scheduler tries not to lose work
the scheduler may repeat work
task code must be idempotent
```

This must be stated clearly in the project documentation.

The final capstone should not teach the fantasy of exactly-once execution.

It should teach the professional habit of designing safe retries.

---

# Idempotency

Idempotency means repeating an operation does not produce duplicate harm.

Safe:

```text
set report status to generated
write object to deterministic path
upsert row by unique key
send webhook with idempotency key
charge payment with provider idempotency key
```

Unsafe:

```text
insert duplicate row without constraint
send email without dedupe key
charge card without idempotency key
append to file blindly
increment counter blindly
```

The scheduler can help by assigning a run id.

Task code can use:

```text
schedule_id
run_id
idempotency_key
```

The project should recommend passing metadata to tasks:

```python
def send_summary(account_id: int, context: RunContext) -> None:
    ...
```

The context can include run id and scheduled time.

---

# Run Context

A run context gives task functions operational metadata.

```python
@dataclass(frozen=True)
class RunContext:
    run_id: int
    schedule_id: int
    worker_id: str
    scheduled_for: datetime
    attempt: int
```

The worker can pass context as a reserved keyword:

```python
func(*args, context=context, **kwargs)
```

Or it can pass context only to tasks that request it.

The simpler version can always pass it and document the convention.

This makes tasks easier to make idempotent and observable.

---

# Pause And Resume

Operators need control.

Schedules should be pausable.

When paused:

```text
no new runs are created
existing runs may continue
```

Resume should allow future runs again.

Do not delete schedule state when pausing.

Pausing is reversible.

CLI examples:

```bash
python -m distsched pause daily-summary
python -m distsched resume daily-summary
```

API examples:

```text
POST /schedules/daily-summary/pause
POST /schedules/daily-summary/resume
```

---

# Cancellation

Cancellation is different from pausing.

Pausing affects future run creation.

Cancellation affects a specific run.

For this project, allow cancelling pending or retrying runs.

Do not try to safely stop a currently running Python function.

Stopping running code is hard and unsafe in general.

If a running run is cancelled, mark it as cancellation requested and let the worker finish or notice cooperatively.

For the first version:

```text
pending/retrying -> cancelled
running -> not supported or cancellation_requested
```

Document the limitation.

---

# API Design

Expose a small HTTP API using FastAPI or the mini framework concepts from earlier.

Endpoints:

```text
POST /schedules
GET /schedules
GET /schedules/{name}
POST /schedules/{name}/pause
POST /schedules/{name}/resume
GET /runs
GET /runs/{id}
GET /runs/{id}/events
GET /workers
```

The API does not need authentication for the capstone.

But it should mention that a real scheduler API is operationally sensitive.

In production, schedule creation and cancellation must be protected.

---

# CLI Design

The CLI can be the main user interface if you do not want an HTTP API.

Commands:

```text
init
schedule create
schedule list
schedule show
schedule pause
schedule resume
run list
run show
worker run
worker list
tick
```

`tick` is useful for tests and manual operation.

It means:

```text
generate due runs once
```

`worker run` starts a worker loop.

This split mirrors real systems:

```text
scheduler loop creates due work
worker loop executes due work
```

For the learning implementation, one process may do both.

But the conceptual separation should remain.

---

# Scheduler Loop

The scheduler loop periodically creates due runs.

```python
while not stopping:
    scheduler.create_due_runs()
    sleep(poll_interval)
```

This loop should be testable through a single-step method:

```python
scheduler.tick()
```

`tick` performs one pass.

Tests should call `tick`.

The infinite loop should only be used by the CLI.

This pattern appeared in the task queue capstone too.

Long-running processes should expose one-step behavior for tests.

---

# Worker Loop

The worker loop periodically claims and runs work.

```python
while not stopping:
    heartbeat()
    run = claim_next_run(worker_id)
    if run is None:
        sleep(poll_interval)
        continue
    execute(run)
```

The worker should also expose:

```python
worker.run_once()
```

Tests should prefer `run_once`.

The worker should heartbeat even when no work is available.

This keeps worker visibility current.

---

# Lease Recovery

Lease recovery means making abandoned work eligible again.

If a run is `running` and its lease has expired, the system should be able to retry it.

There are two approaches.

Approach one:

```text
claim_next_run treats expired leases as eligible
```

Approach two:

```text
separate recovery job marks expired runs as retrying
```

For this capstone, the first approach is simpler.

When a worker claims work, it may claim:

```text
pending due runs
retrying due runs
running runs with expired lease
```

If claiming an expired running run, record a `lease_expired` event.

This gives history.

---

# Atomic Claim With SQLite

SQLite does not support `SELECT FOR UPDATE` the way PostgreSQL does.

Use `BEGIN IMMEDIATE`.

The claim operation:

```text
BEGIN IMMEDIATE
select one eligible run
if no run:
    COMMIT
    return None
update selected run with lease owner, lease expiration, attempts
insert claimed event
COMMIT
return run
```

This is enough for the learning version.

In PostgreSQL, you might use:

```sql
SELECT ...
FOR UPDATE SKIP LOCKED
```

That pattern is common for worker systems.

The capstone should mention it without requiring it.

---

# Observability

A distributed scheduler must be inspectable.

The system should answer:

```text
how many schedules exist?
which schedules are paused?
which runs are pending?
which runs are running?
which runs are dead?
which workers are alive?
which runs failed recently?
how long do runs take?
how often are retries happening?
```

Minimum observability:

```text
structured logs
run events
list commands
show commands
worker heartbeats
state counts
```

This project should not treat observability as decoration.

For distributed systems, observability is part of correctness.

If operators cannot understand the system, they cannot safely run it.

---

# Structured Logging

Use Python's `logging` module.

Log important events with fields when possible:

```text
worker_id
run_id
schedule_id
task_name
state
attempt
duration
error
```

Even if using plain log strings, include identifiers.

Bad:

```text
Job failed
```

Better:

```text
run_id=42 task=send_summary attempt=2 failed error="timeout"
```

Logs should help someone debug the system at 2 AM.

That is the standard.

---

# Metrics

Metrics can be simple counters and gauges.

Examples:

```text
schedules_total
runs_pending
runs_running
runs_succeeded_total
runs_failed_total
runs_dead_total
worker_heartbeat_age_seconds
run_duration_seconds
claim_latency_seconds
```

The capstone does not need Prometheus integration.

It can expose a simple `stats()` method or `/stats` endpoint.

The important lesson is what to measure.

Mature systems are operated through signals.

---

# Testing Strategy

The distributed scheduler needs layered tests.

Test schedule calculation without storage.

Test storage without workers.

Test claiming without task execution.

Test worker execution with fake tasks.

Test retry behavior with fake clocks.

Test lease expiration deterministically.

Test API or CLI separately.

Do not rely on long sleeps.

Do not require real distributed machines.

The system can simulate multiple workers by creating two worker objects with different worker ids.

That is enough to test many coordination rules.

---

# Testing Schedules

Schedule tests should cover:

```text
one-time schedule due now
one-time schedule due in future
interval schedule next run calculation
paused schedule does not create runs
missed interval uses chosen catch-up policy
invalid interval is rejected
payload must be JSON serializable
```

Use fake time.

Example:

```python
clock.set("2026-06-15T09:00:00Z")
scheduler.create_interval("hourly", interval_seconds=3600)
scheduler.tick()
assert one_run_created()
```

Then advance time.

Test the next run.

---

# Testing Claiming

Claiming tests should simulate multiple workers.

Create one due run.

Worker A claims it.

Worker B tries to claim.

Worker B should get no run.

Then expire the lease.

Worker B tries again.

Worker B should claim it.

This test proves the lease model.

It also proves that ownership is durable state, not memory inside one worker object.

---

# Testing Retries

Register a task that fails.

Create a run with `max_attempts=2`.

Worker runs once.

Expected:

```text
state = retrying
attempts = 1
last_error exists
scheduled_for moved into future
```

Advance fake time.

Worker runs again.

Expected:

```text
state = dead
attempts = 2
finished_at exists
```

This test is the essence of failure handling.

---

# Testing Success

Register a task that records a call.

Create a due run.

Worker runs once.

Expected:

```text
task called with expected args
run state = succeeded
finished_at set
lease cleared
succeeded event recorded
```

Success tests should also verify that completed runs are never claimed again.

---

# Testing Heartbeats

Create a worker.

Run `heartbeat`.

Assert the workers table contains the worker id and heartbeat time.

Advance fake time.

Heartbeat again.

Assert the timestamp changed.

Add a helper to find stale workers:

```text
heartbeat_at < now - stale_after
```

Test it.

This is simple, but it teaches operational visibility.

---

# Milestone 1 - Models And Clock

Create dataclasses for:

```text
Schedule
Run
WorkerRecord
RunEvent
RunContext
```

Create state constants.

Create an injectable clock.

Write tests for time formatting and parsing.

This milestone gives the project vocabulary.

---

# Milestone 2 - Storage Schema

Implement SQLite initialization.

Create tables:

```text
schedules
runs
workers
run_events
```

Add row-to-model helpers.

Write tests that initialize a temporary database and verify tables exist.

This milestone creates the durable foundation.

---

# Milestone 3 - Schedule Creation

Implement:

```text
create_one_time_schedule
create_interval_schedule
get_schedule
list_schedules
pause_schedule
resume_schedule
```

Validate:

```text
unique names
known task names if registry is available
positive intervals
JSON-serializable payloads
future or present run times
```

Write storage tests for each operation.

---

# Milestone 4 - Due Run Generation

Implement:

```text
scheduler.tick()
storage.create_due_runs(now)
next_run_at update
catch-up policy
run event creation
```

Test one-time and interval schedules.

Test paused schedules.

Test missed-run behavior.

This milestone turns schedules into executable work.

---

# Milestone 5 - Task Registry

Implement task registration:

```python
@task("name")
def func(...):
    ...
```

Support duplicate-name errors.

Support unknown-task errors.

This repeats an idea from the task queue capstone because it is a common pattern.

The reader should recognize it now.

---

# Milestone 6 - Claiming And Leases

Implement:

```text
claim_next_run(worker_id, lease_seconds)
lease_owner
lease_expires_at
attempt increment
claimed event
expired lease recovery
```

Use transactions.

Test two workers competing for one run.

Test expired leases.

This is the core distributed coordination milestone.

---

# Milestone 7 - Worker Execution

Implement:

```text
Worker.heartbeat
Worker.run_once
success handling
failure handling
retry scheduling
dead state
event recording
```

Use fake tasks in tests.

Make sure leases are cleared after success and failure.

Make sure errors are recorded.

---

# Milestone 8 - Scheduler And Worker Loops

Add:

```text
Scheduler.run_forever
Worker.run_forever
graceful KeyboardInterrupt handling
poll intervals
```

Keep infinite loops out of most tests.

Expose one-step methods:

```text
tick
run_once
heartbeat
```

This keeps the system testable.

---

# Milestone 9 - CLI Or API

Implement a user interface.

Minimum CLI:

```text
distsched init
distsched schedule create
distsched schedule list
distsched schedule pause
distsched schedule resume
distsched run list
distsched run show
distsched worker run
distsched worker list
distsched tick
```

Or implement the HTTP API listed earlier.

If using FastAPI, keep the scheduler core independent of FastAPI.

The API should be an adapter, not the whole system.

---

# Milestone 10 - Observability And Documentation

Add:

```text
structured logs
stats method
event display
README
architecture diagram
failure mode notes
idempotency guidance
```

The final capstone should feel operational.

The reader should know how to run it, inspect it, and reason about failures.

---

# Common Mistake 1 - Using Sleep As The Scheduler

This is not enough:

```python
while True:
    time.sleep(60)
    run_job()
```

That loses state on restart.

It cannot coordinate workers.

It cannot recover abandoned work.

It cannot explain history.

A real scheduler stores schedule and run state durably.

---

# Common Mistake 2 - Claiming Without A Transaction

This is unsafe:

```python
run = select_next_run()
mark_running(run.id)
```

Two workers can select the same run.

Claiming must be atomic.

Use a transaction that selects and updates as one operation.

This lesson is central.

---

# Common Mistake 3 - Permanent Locks

If a worker sets:

```text
locked = true
```

and then dies, the run may be stuck forever.

Use leases.

Leases expire.

Expired work can be recovered.

This is one of the key differences between toy coordination and resilient coordination.

---

# Common Mistake 4 - No Execution History

If the system only stores the current state, debugging becomes hard.

Operators need to know:

```text
when was the run created?
who claimed it?
when did it fail?
what error happened?
when was it retried?
why is it dead?
```

Run events answer those questions.

History is not optional in serious schedulers.

---

# Common Mistake 5 - Pretending Exactly Once

Do not claim this scheduler guarantees exactly-once execution.

It does not.

Most practical systems do not.

The scheduler can provide durable at-least-once execution with leases and retries.

Task code must handle duplicates.

This is the professional truth.

---

# Common Mistake 6 - Untestable Time

If tests use real sleeps, they become slow and flaky.

Inject the clock.

Advance fake time in tests.

Scheduling systems are time systems.

Time systems need controlled time in tests.

---

# Common Mistake 7 - Mixing Core Logic With API Code

Do not put scheduling rules inside route handlers.

The API should call the scheduler service.

The scheduler service should call storage.

The worker should call the same storage and registry.

This keeps the core usable from CLI, tests, and HTTP.

---

# Extension Ideas

Add cron expressions.

Add timezone-specific schedules.

Add per-schedule retry policies.

Add jitter.

Add max concurrency per task.

Add named queues.

Add worker capabilities.

Add lease renewal for long-running tasks.

Add cooperative cancellation.

Add dead-run requeue.

Add manual run trigger.

Add schedule versioning.

Add audit logs for schedule changes.

Add PostgreSQL support with `FOR UPDATE SKIP LOCKED`.

Add Redis-based leases.

Add a web dashboard.

Add Prometheus metrics.

Add OpenTelemetry traces.

Add DAG dependencies.

Add task result storage.

Add distributed leader election.

Add high-availability scheduler instances.

Each extension is a real systems engineering topic.

The core project should stay understandable before adding them.

---

# What This Project Teaches

This project teaches production-minded Python engineering.

It teaches durable scheduling.

It teaches run generation.

It teaches atomic claiming.

It teaches leases.

It teaches worker heartbeats.

It teaches retries and backoff.

It teaches idempotency.

It teaches failure recovery.

It teaches operational visibility.

It teaches API and CLI boundaries.

It teaches testable time.

It connects the whole book:

```text
dataclasses model schedules and runs
JSON serializes payloads
SQLite stores durable state
transactions protect claims
functions become registered tasks
exceptions become retry decisions
logging creates operational evidence
tests simulate time and failures
APIs expose control and inspection
architecture separates core logic from adapters
```

The reader should leave with this mental model:

```text
A distributed scheduler is durable state plus time rules plus lease-based workers plus observable recovery.
```

That sentence contains much of professional backend engineering.

---

# Completion Checklist

The project is complete when:

```text
Schedules can be created.
One-time schedules create one due run.
Interval schedules create recurring runs.
Paused schedules do not create new runs.
Schedules can be resumed.
Runs are stored durably.
Workers can heartbeat.
Workers can claim due runs.
Claiming is atomic.
Leases prevent permanent stuck locks.
Expired leases can be recovered.
Successful runs are marked succeeded.
Failed runs record errors.
Failed runs retry when attempts remain.
Exhausted runs become dead.
Run events record lifecycle history.
At-least-once execution is documented.
Idempotency guidance is documented.
The CLI or API can inspect schedules, runs, workers, and events.
Tests use fake time.
Tests cover schedule generation.
Tests cover multiple workers claiming.
Tests cover lease expiration.
Tests cover success, failure, retries, and dead runs.
Documentation explains failure modes and limitations.
```

When all of these are true, the reader has built a serious final capstone.

---

# Exercises

1. Create the `distsched` package structure.

2. Define schedule, run, worker, event, and context dataclasses.

3. Define state constants.

4. Implement UTC time helpers.

5. Add an injectable fake clock for tests.

6. Create the SQLite schema.

7. Implement row-to-model mapping.

8. Implement one-time schedule creation.

9. Implement interval schedule creation.

10. Implement schedule listing and lookup.

11. Implement pause and resume.

12. Implement due schedule selection.

13. Implement run creation from schedules.

14. Implement next run calculation.

15. Implement the catch-up policy.

16. Add run event creation.

17. Implement the task registry.

18. Implement worker heartbeat.

19. Implement atomic claim with leases.

20. Test two workers competing for one run.

21. Test expired lease recovery.

22. Implement successful run completion.

23. Implement failure recording.

24. Implement retry scheduling with backoff.

25. Implement dead state for exhausted runs.

26. Add worker `run_once`.

27. Add scheduler `tick`.

28. Add CLI command `schedule create`.

29. Add CLI command `schedule list`.

30. Add CLI command `run list`.

31. Add CLI command `worker run`.

32. Add event inspection.

33. Add stats output.

34. Write failure-mode documentation.

35. Write idempotency guidance.

36. Add a complete example application.

---

# Final Capstone Reflection

Capstone 11 built a Distributed Scheduler.

It brought together scheduled work, persistent state, worker processes, leases, heartbeats, retries, idempotency, observability, and failure recovery.

It is a fitting final project because it asks the reader to think beyond syntax.

The question is no longer only:

```text
can I write Python code?
```

The question becomes:

```text
can I design a Python system that survives real conditions?
```

That is the transition from learning a language to practicing engineering.

Across the capstones, the reader has built:

```text
a CLI application
a file automation tool
a REST API
a URL shortener
an ORM
a task queue
an in-memory data server
a web framework
an event loop
an interpreter
a distributed scheduler
```

Together, these projects turn the four volumes into working memory.

The final lesson is simple:

```text
Python is not only a language for scripts.
Python is a language for building systems.
```

And now the reader has seen those systems from the inside.
