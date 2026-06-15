# Chapter 65 — Concurrency Foundations

Concurrency is one of the most misunderstood topics in programming.

It is also one of the most useful.

Many programs spend much of their time waiting.

They wait for files.

They wait for networks.

They wait for databases.

They wait for users.

They wait for subprocesses.

They wait for queues.

They wait for locks.

They wait for timers.

If a program can do useful work while one operation is waiting, the program may become faster, more responsive, or more scalable.

That is the promise of concurrency.

But concurrency is not free.

It introduces new problems:

* race conditions
* deadlocks
* shared-state bugs
* ordering bugs
* resource leaks
* hidden blocking
* harder testing
* harder debugging
* harder error handling
* harder reasoning

Concurrency can make a slow program fast.

It can also make a simple program unreliable.

This chapter is about the mental model.

We will not yet dive deeply into the full APIs of `threading`, `multiprocessing`, or `asyncio`.

Those come next.

Before using concurrency tools, you need to understand what problem they solve.

---

# The Core Idea

Concurrency means a program can make progress on multiple tasks whose lifetimes overlap.

The tasks do not necessarily run at the exact same instant.

They overlap in time.

Example:

```text
Task A starts
Task A waits for network
Task B runs
Task B waits for disk
Task A resumes
Task C runs
Task B resumes
```

The program is not doing one complete task and then another complete task.

It is interleaving them.

That interleaving is the heart of concurrency.

Sequential execution:

```text
Task A: start -> finish
Task B: start -> finish
Task C: start -> finish
```

Concurrent execution:

```text
Task A: start -> wait  -> resume -> finish
Task B:        start  -> wait   -> resume -> finish
Task C:                 start -> finish
```

The tasks overlap.

That is concurrency.

---

# Concurrency vs Parallelism

Concurrency and parallelism are related, but they are not the same.

Concurrency is about structure.

Parallelism is about simultaneous execution.

Concurrency:

```text
multiple tasks are in progress during the same time period
```

Parallelism:

```text
multiple tasks run at the same physical instant
```

One worker switching between tasks is concurrent:

```text
worker handles task A
task A waits
worker handles task B
task B waits
worker returns to task A
```

Two workers running at the same time is parallel:

```text
worker 1 handles task A
worker 2 handles task B
```

A single-core machine can run concurrent programs.

It cannot truly run CPU instructions from two threads at the exact same instant on that one core.

It can switch quickly between them.

A multi-core machine can run parallel work.

It can execute different CPU instructions on different cores at the same time.

The distinction matters because different problems need different tools.

If the program is waiting on I/O, concurrency may help even without parallel CPU execution.

If the program is doing heavy computation, parallel CPU execution may be necessary.

---

# A Simple Analogy

Imagine one person cooking.

Sequential cooking:

```text
boil pasta completely
then chop vegetables
then make sauce
then set table
```

Concurrent cooking:

```text
start boiling water
while water heats, chop vegetables
while sauce simmers, set table
when pasta is ready, drain it
```

The person is not doing two physical actions at the same instant.

They are organizing tasks so waiting time is not wasted.

Parallel cooking:

```text
one person makes sauce
another person chops vegetables
another person sets table
```

Now multiple people are literally working at the same time.

Concurrency is about overlapping tasks.

Parallelism is about simultaneous execution.

Sometimes you want both.

Sometimes concurrency alone is enough.

---

# Why Programs Wait

CPUs are fast compared with many external operations.

A CPU can execute enormous numbers of instructions while waiting for:

* disk reads
* disk writes
* database responses
* network requests
* user input
* subprocesses
* timers
* locks

Consider a program that downloads three URLs one after another.

Sequential model:

```text
request URL 1
wait
process response 1
request URL 2
wait
process response 2
request URL 3
wait
process response 3
```

If each request spends most of its time waiting on the network, the CPU may be idle.

A concurrent model:

```text
start request URL 1
start request URL 2
start request URL 3
wait for whichever finishes first
process responses as they arrive
```

The total time may be closer to the slowest request than the sum of all requests.

This is why concurrency is common in:

* web servers
* crawlers
* API clients
* database-heavy applications
* GUI applications
* background workers
* build systems
* automation tools
* data pipelines

Concurrency is often about not wasting wait time.

---

# CPU-Bound and I/O-Bound Work

Before choosing a concurrency tool, ask what the program is doing.

There are two broad categories:

```text
CPU-bound work
I/O-bound work
```

CPU-bound work spends most of its time computing.

Examples:

* image processing
* numerical simulation
* compression
* encryption
* parsing huge data
* machine learning computation
* brute-force search
* video encoding

I/O-bound work spends most of its time waiting for input or output.

Examples:

* reading files
* writing files
* calling APIs
* querying databases
* downloading pages
* waiting for user input
* communicating with subprocesses

The difference matters.

For I/O-bound work, concurrency can help because tasks can overlap while waiting.

For CPU-bound work, concurrency alone may not help unless work can actually run in parallel on multiple cores.

The wrong tool can make performance worse.

---

# Blocking

An operation blocks when the current execution path cannot continue until that operation finishes.

Example:

```python
text = input("Name: ")
```

The program waits until the user types something.

File reading can block:

```python
data = path.read_bytes()
```

Network calls can block:

```python
response = fetch_url(url)
```

Sleep blocks:

```python
time.sleep(5)
```

Blocking is not bad by itself.

A simple script can block and still be perfectly fine.

Blocking becomes a problem when one task waits and prevents other useful work from happening.

Example:

```text
web server receives request A
request A waits for database
request B arrives
server cannot handle B because A is blocking the only worker
```

Concurrency gives the program a way to let something else proceed while one operation waits.

---

# Scheduling

A scheduler decides what runs next.

Scheduling happens at different levels.

The operating system schedules processes and threads.

An event loop schedules asynchronous tasks.

A thread pool schedules callables onto worker threads.

A process pool schedules callables onto worker processes.

In a sequential program, scheduling is mostly invisible:

```python
step_one()
step_two()
step_three()
```

In a concurrent program, there may be many possible execution orders.

Example:

```text
Task A reads value
Task B reads value
Task A writes value
Task B writes value
```

or:

```text
Task A reads value
Task A writes value
Task B reads value
Task B writes value
```

Both schedules may be possible.

If your program assumes only one ordering, it may be wrong.

Concurrency is not only about speed.

It is about reasoning under multiple possible schedules.

---

# Processes

A process is an executing program.

Volume I introduced this idea when studying operating systems.

A process has:

* its own memory space
* its own Python interpreter instance
* its own global variables
* its own file descriptors
* its own operating-system resources

Multiple processes are isolated from each other by the operating system.

If one process modifies a normal Python list, another process does not see that mutation automatically.

This isolation is powerful.

It prevents accidental shared-memory bugs.

It also means communication requires explicit mechanisms:

* pipes
* queues
* sockets
* files
* shared memory
* databases
* message brokers

Processes are often useful for CPU-bound work because separate processes can run on separate CPU cores.

In CPython, this matters because of the Global Interpreter Lock, which we will study in Chapter 66.

For now, remember:

```text
processes isolate memory and can provide real parallel CPU execution
```

The cost is:

* higher memory use
* slower startup
* serialization of data between processes
* more explicit communication
* more operational complexity

---

# Threads

A thread is a path of execution inside a process.

One process can have multiple threads.

Threads in the same process share memory.

That means they can access the same Python objects.

This is convenient.

It is also dangerous.

Example:

```python
shared = []
```

Two threads can both append to the same list.

They can both read and write the same dictionary.

They can both update the same object attribute.

Shared memory makes communication easy.

It also makes accidental interference easy.

Threads are often useful for I/O-bound work:

* one thread waits for network
* another thread reads a file
* another thread handles user input
* another thread updates status

In CPython, threads are generally not the best way to speed up pure Python CPU-bound work.

But they are still valuable for overlapping blocking I/O.

The short model:

```text
threads share memory and are useful for overlapping blocking operations
```

The cost is:

* shared-state bugs
* need for locks
* difficult debugging
* nondeterministic ordering
* shutdown complexity

---

# Tasks

In asynchronous programming, a task is a schedulable unit of work managed by an event loop.

Tasks are lighter than operating-system threads.

They usually run in one thread, coordinated by an event loop.

The task voluntarily gives control back when it reaches an `await`.

Conceptually:

```text
task A runs until await
task A pauses
event loop runs task B
task B runs until await
task B pauses
event loop resumes task A when its operation is ready
```

This is cooperative concurrency.

The event loop does not forcibly interrupt ordinary Python code at arbitrary points.

Tasks must reach await points.

This is why async code can be easier to reason about than preemptive threads in some ways.

But it has its own rules.

If an async task performs blocking work without `await`, it can freeze the event loop.

Example:

```python
async def bad():
    time.sleep(5)
```

That blocks the thread running the event loop.

Async code must use non-blocking async-compatible operations:

```python
async def better():
    await asyncio.sleep(5)
```

We will study this properly in Chapter 67.

For now:

```text
async tasks are concurrent units scheduled cooperatively by an event loop
```

---

# Three Main Models in Python

Python gives you several concurrency models.

The big three are:

```text
threads
processes
async tasks
```

Threads:

```text
multiple execution paths inside one process
shared memory
good for blocking I/O
requires synchronization for shared state
```

Processes:

```text
multiple independent processes
separate memory
good for CPU-bound parallelism
requires serialization or IPC for communication
```

Async tasks:

```text
many tasks on an event loop
cooperative scheduling
good for high-concurrency I/O
requires async-compatible libraries
```

There is no universally best model.

There is only the model that fits the problem.

---

# A First Decision Table

Use this rough guide:

```text
many network requests              -> async or threads
many blocking file/API operations  -> threads or async if libraries support it
CPU-heavy pure Python computation  -> processes
GUI responsiveness                 -> background thread or event loop integration
background periodic work           -> thread, process, task queue, or scheduler
high-volume web server             -> async, threads, processes, or a framework model
simple script                      -> maybe no concurrency
```

The final line matters.

Many programs do not need concurrency.

A simple sequential program is easier to test and reason about.

Reach for concurrency when there is a real reason:

* responsiveness
* throughput
* latency
* parallel CPU use
* external wait time
* independent background work

Do not use concurrency just because it sounds advanced.

---

# Shared State

Shared state is data accessed by more than one concurrent worker.

Example:

```python
counter = 0
```

If two workers update `counter`, they share state.

Example:

```python
counter += 1
```

This line looks simple.

But conceptually it is multiple steps:

```text
read counter
add one
write counter
```

If two workers interleave those steps, the result can be wrong.

Possible schedule:

```text
counter starts at 0

worker A reads 0
worker B reads 0
worker A writes 1
worker B writes 1
```

Two increments happened.

The final value is 1.

One update was lost.

This is a race condition.

---

# Race Conditions

A race condition occurs when program correctness depends on the timing or ordering of concurrent operations.

The problem is not that operations happen quickly.

The problem is that different valid schedules produce different results.

Example:

```text
if file does not exist:
    create file
```

Two workers can both check that the file does not exist.

Then both try to create it.

This is a race.

Example:

```text
if user has enough balance:
    subtract amount
```

Two concurrent withdrawals can both see enough balance before either subtraction is committed.

Race conditions are dangerous because they may not reproduce reliably.

The program may pass tests.

It may work on your machine.

It may fail under load.

It may fail only once a month.

It may disappear when you add logging because logging changes timing.

This is why concurrency bugs can feel strange.

They are schedule-dependent.

---

# Atomicity

An operation is atomic when it happens as one indivisible step from the perspective of other concurrent work.

Example:

```text
either the operation has not happened
or it has fully happened
no other task observes it halfway
```

Atomicity is a boundary concept.

At one level, a Python statement looks atomic.

At a lower level, it may involve multiple bytecode instructions.

At another level, a library may provide atomic behavior for a specific operation.

At the file-system level, some operations such as certain same-directory replacements may be atomic from the perspective of readers.

Never assume atomicity just because code is one line.

Ask:

```text
what other concurrent work can observe this?
```

If the answer matters, use a synchronization mechanism or a safer design.

---

# Locks

A lock is a synchronization primitive.

It allows only one worker at a time to enter a protected section.

Conceptually:

```text
acquire lock
read or modify shared state
release lock
```

Only one worker can hold the lock at a time.

A protected section is often called a critical section.

Example:

```python
with lock:
    counter += 1
```

The `with` statement matters.

It ensures the lock is released even if an exception occurs.

Locks can prevent race conditions.

They can also create new problems.

If code holds a lock too long, other workers wait.

If code forgets to release a lock, other workers may block forever.

If two workers acquire multiple locks in different orders, the program can deadlock.

Locks are necessary sometimes.

They are not decoration.

They are part of the correctness model.

---

# Deadlocks

A deadlock occurs when workers wait forever for each other.

Classic example:

```text
worker A holds lock 1
worker B holds lock 2
worker A waits for lock 2
worker B waits for lock 1
```

Neither can proceed.

Neither can release what the other needs.

The program is stuck.

Deadlocks can also happen with:

* queues
* events
* subprocess pipes
* database transactions
* thread joins
* async tasks waiting on each other

Deadlocks are not limited to locks.

They are circular waiting problems.

Ways to reduce deadlock risk:

* use fewer locks
* keep critical sections small
* acquire locks in a consistent order
* use timeouts where appropriate
* avoid holding locks while doing slow I/O
* prefer message passing when it fits
* design ownership clearly

Concurrency design is often about reducing how much can wait on how much.

---

# Message Passing

Instead of sharing state directly, workers can communicate by sending messages.

Example:

```text
worker A sends result to queue
worker B receives result from queue
```

This is often safer than letting both workers mutate the same object.

A queue provides a boundary.

Producer:

```text
put item into queue
```

Consumer:

```text
take item from queue
```

The queue coordinates access.

Message passing is common in:

* thread pools
* process pools
* actor systems
* task queues
* async pipelines
* web workers

The design principle:

```text
share less mutable state
communicate through clearer boundaries
```

This does not eliminate all concurrency bugs.

But it often makes the program easier to reason about.

---

# Immutability Helps

Immutable data is easier to share.

If two workers can read the same value but neither can mutate it, many race conditions disappear.

Examples:

* strings
* numbers
* tuples of immutable values
* frozen dataclasses
* configuration snapshots

Mutable shared data needs coordination.

Immutable shared data often does not.

This connects back to Volume I:

```text
mutability controls whether shared references can observe changes
```

Concurrency makes that lesson more urgent.

If many workers share a mutable object, every mutation becomes a scheduling question.

If many workers share immutable data, the object can be safely read without coordination in many cases.

The best concurrency bug is the one your design prevents from existing.

---

# Ownership

Ownership means deciding which part of the program is allowed to modify a piece of state.

Bad ownership:

```text
any worker can update shared_state whenever it wants
```

Clear ownership:

```text
only worker A owns this state
other workers send requests to worker A
```

Ownership reduces confusion.

It answers:

* who can mutate this object?
* who can read it?
* how are updates requested?
* when is the state considered valid?
* what invariants must be protected?

In sequential code, vague ownership may survive for a while.

In concurrent code, vague ownership becomes expensive quickly.

Concurrency rewards simple state ownership.

---

# Futures

A future represents a result that may not be ready yet.

Conceptually:

```text
start work now
get a handle to its future result
ask for the result later
```

A future can usually be in states such as:

```text
pending
running
completed with value
completed with exception
cancelled
```

Python's `concurrent.futures` module provides futures for thread and process pools.

The exact API comes later.

For now, the mental model is enough.

Instead of:

```python
result = slow_operation()
```

you can think:

```text
future = schedule slow operation
do other work
result = future result when needed
```

Futures make concurrent work more explicit.

They give you a handle to:

* wait for completion
* retrieve a result
* retrieve an exception
* check status
* cancel if possible

Futures also remind you that background work can fail.

Errors do not disappear just because work runs elsewhere.

They must be collected and handled.

---

# Pools

Creating a new thread or process for every tiny task can be expensive.

A pool keeps a set of workers ready.

You submit work to the pool.

The pool assigns work to available workers.

Thread pool:

```text
main program submits I/O tasks
worker threads execute them
main program collects results
```

Process pool:

```text
main program submits CPU tasks
worker processes execute them
main program collects results
```

Pools are useful because they:

* limit concurrency
* reuse workers
* centralize scheduling
* provide result handles
* make shutdown more manageable

Unlimited concurrency is usually a mistake.

If you start 100,000 tasks at once, you may overwhelm:

* your CPU
* your memory
* the remote service
* the database
* the file system
* the operating system

Concurrency needs limits.

Pools are one way to express those limits.

---

# Backpressure

Backpressure means slowing producers when consumers cannot keep up.

Imagine:

```text
producer creates 10,000 tasks per second
consumer handles 500 tasks per second
```

If nothing slows the producer, the queue grows without bound.

Memory rises.

Latency rises.

Eventually the system may fail.

Backpressure asks:

```text
what happens when work arrives faster than it can be handled?
```

Possible answers:

* block the producer
* reject new work
* drop low-priority work
* batch work
* scale consumers
* apply rate limits
* shed load

Backpressure is a professional concurrency concept because it accepts reality:

```text
capacity is finite
```

Good concurrent systems know what happens under pressure.

Bad ones only work when everything is calm.

---

# Timeouts

Concurrent programs often wait.

Whenever a program waits, ask:

```text
could this wait forever?
```

If yes, consider a timeout.

Timeouts are useful for:

* network calls
* subprocesses
* lock acquisition
* queue operations
* database queries
* task completion

Without timeouts, a program may hang indefinitely.

But timeouts are not magic.

When a timeout occurs, the program needs a policy:

* retry?
* cancel?
* report failure?
* use fallback?
* mark task failed?
* continue partially?

Timeouts turn hidden waiting into explicit failure.

That connects directly to Chapter 62:

```text
timeouts are error boundaries for waiting
```

---

# Cancellation

Cancellation means asking work to stop before it completes.

Cancellation is harder than it sounds.

If a task is halfway through writing a file, should cancellation interrupt it?

If a worker holds a lock, can it be stopped safely?

If a database transaction is open, should it roll back?

If a subprocess is running, should it be terminated?

Good cancellation is cooperative.

The worker reaches safe points and checks whether it should stop.

Unsafe cancellation can leave resources inconsistent.

Design cancellation around cleanup:

* close files
* release locks
* roll back transactions
* remove temporary files
* notify dependents
* report partial progress honestly

Cancellation is not just stopping work.

It is stopping work without corrupting the system.

---

# Exceptions in Concurrent Work

In sequential code, an exception travels up the call stack.

In concurrent code, work may happen somewhere else.

That raises a question:

```text
where does the exception go?
```

If a background thread raises an exception, it may not automatically appear where the thread was started.

If a future fails, the exception may be raised when you ask for the result.

If an async task fails and nobody observes it, the event loop may report it later.

If a process crashes, the parent process must detect that failure.

Concurrent programs need explicit error collection.

Do not start background work and ignore its outcome.

Bad:

```text
start background task
never check whether it succeeded
```

Better:

```text
start background task
keep handle
wait or observe completion
handle result or exception
```

Errors are still part of the program.

Concurrency changes where they surface.

It does not remove them.

---

# Ordering

Sequential code has obvious order.

```python
a()
b()
c()
```

`a` runs before `b`, and `b` runs before `c`.

Concurrent code may not have a single obvious order.

If three tasks start together, any of them may finish first.

That affects program design.

If order matters, preserve it deliberately.

If order does not matter, do not accidentally depend on it.

Example:

```text
download files concurrently
write summary in original input order
```

This requires separating:

```text
completion order
input order
output order
```

Common mistake:

```text
assuming concurrent results arrive in the same order work was submitted
```

Sometimes APIs preserve order.

Sometimes they return results as completed.

You must know which behavior you are using.

---

# Determinism

A deterministic program gives the same result for the same input under the same conditions.

Concurrency can reduce determinism.

Not always.

But it often introduces timing-dependent behavior.

Example:

```text
worker A logs first on one run
worker B logs first on another run
```

That may be harmless.

But if output correctness depends on that order, it is a bug.

The goal is not to force every event into a fixed order.

The goal is to know which order matters.

Design deterministic boundaries:

* sort final results if order matters
* attach sequence numbers
* use queues with defined ordering
* avoid shared mutable state
* collect results and assemble them deliberately

Concurrency should not make user-visible behavior randomly wrong.

---

# Throughput and Latency

Performance has different meanings.

Latency is how long one operation takes.

Throughput is how much work completes per unit of time.

Example:

```text
latency: one request takes 200 ms
throughput: server handles 1,000 requests per second
```

Concurrency can improve throughput without improving individual latency.

A web server may handle many requests at once.

Each request may still take 200 ms.

But the server can complete more total requests per second.

Concurrency can sometimes improve latency too.

If a task has independent subtasks, running them concurrently can reduce total waiting time.

Example:

```text
fetch user profile
fetch user permissions
fetch user notifications
```

If independent, these can overlap.

The total time may become closer to the slowest fetch rather than the sum of all fetches.

Always ask:

```text
am I trying to improve latency, throughput, responsiveness, or CPU utilization?
```

The answer affects the design.

---

# Responsiveness

Concurrency is not only for speed.

It is also for responsiveness.

A GUI application should not freeze while saving a file.

A command-line tool may want to show progress while work continues.

A server should keep accepting connections while one request waits.

A monitoring process should keep checking health while a background operation runs.

Responsiveness means:

```text
the program can still react while work is in progress
```

Sometimes this matters more than total runtime.

An operation that takes ten seconds but keeps the interface responsive may feel better than one that takes eight seconds and freezes everything.

Concurrency is often about user experience and operational behavior, not only benchmark numbers.

---

# Concurrency and Resource Limits

Every concurrent worker consumes resources.

Threads consume memory for stacks and scheduling overhead.

Processes consume more memory because each has its own process state.

Async tasks are lighter, but they still consume memory and file descriptors if they hold open connections.

External systems have limits too:

* databases have connection limits
* APIs have rate limits
* operating systems have file descriptor limits
* networks have bandwidth limits
* disks have throughput limits

More concurrency is not always better.

At some point, adding workers increases contention.

The system spends more time coordinating than doing useful work.

This is why production systems tune concurrency.

They ask:

```text
how much parallel work can this system actually support?
```

Not:

```text
how many tasks can I start?
```

---

# Sequential Baseline First

Before adding concurrency, write the simple version.

The sequential version helps you understand:

* correctness
* inputs
* outputs
* error handling
* resource usage
* performance baseline

Then measure.

If the sequential version is fast enough, stop.

If it is too slow, profile or inspect where time is spent.

If time is spent waiting on I/O, concurrency may help.

If time is spent computing, parallelism may help.

If time is spent in inefficient algorithms, concurrency may hide the real problem.

Do not use concurrency to avoid understanding performance.

Concurrency should be a response to a measured or clearly understood bottleneck.

---

# Example: Sequential Downloads

Imagine:

```python
def download_all(urls):
    results = []
    for url in urls:
        results.append(download(url))
    return results
```

This is simple.

It is easy to reason about.

It preserves order.

It has obvious error behavior.

But if each `download(url)` spends most of its time waiting on the network, this can be slow.

A concurrent design might start several downloads at once.

But then questions appear:

* How many downloads at a time?
* What if one fails?
* Should failure stop all downloads?
* Should results preserve input order?
* What timeout should each download use?
* Should retries happen?
* Should remote rate limits be respected?

The concurrent version is not just the sequential version with magic speed added.

It is a different design.

---

# Example: CPU-Heavy Work

Imagine:

```python
def process_all(images):
    return [process_image(image) for image in images]
```

If `process_image()` is CPU-heavy, threads may not speed it up in ordinary CPython.

Processes may help because each process can run on a different core.

But process-based design introduces questions:

* How large are the images?
* How expensive is serialization between processes?
* How many CPU cores are available?
* How much memory does each worker need?
* Can work be split independently?
* How should progress be reported?
* What happens if one worker crashes?

Parallelism helps only if the work can be divided and the overhead is worth it.

If each task is tiny, process overhead may dominate.

If each task is huge, memory may become the limit.

---

# Example: Shared Counter

This looks harmless:

```python
counter = 0

def increment():
    global counter
    counter += 1
```

In concurrent code, this is suspicious.

The problem is not the `global` keyword by itself.

The problem is shared mutable state.

If many workers call `increment()`, the program needs to define how updates are protected.

Options:

* use a lock
* use a queue and a single owner
* avoid shared state
* aggregate local counts and combine later
* use a database atomic update
* use a specialized concurrent primitive

Often the best answer is not:

```text
put locks everywhere
```

It is:

```text
change the design so less state is shared
```

---

# Example: Aggregating Results Safely

Instead of every worker updating a shared counter, each worker can return a local result.

Sequential mental model:

```text
worker 1 counts its chunk
worker 2 counts its chunk
worker 3 counts its chunk
main program adds the results
```

This avoids shared mutation during the work.

Example idea:

```python
def count_words(chunk):
    counts = {}
    for word in chunk:
        counts[word] = counts.get(word, 0) + 1
    return counts
```

Each worker owns its local `counts`.

The main program merges results later.

This design is often easier than locking a shared dictionary on every word.

The broader principle:

```text
local work first
combine results at clear boundaries
```

This pattern appears in map-reduce, data processing, build systems, and batch jobs.

---

# Testing Concurrent Code

Concurrent code is harder to test because timing varies.

A test that passes once does not prove the absence of race conditions.

Bad concurrent tests rely on sleep:

```python
time.sleep(0.1)
assert result == expected
```

This can be flaky.

On a fast machine, it passes.

On a slow machine, it fails.

Under load, it fails randomly.

Better tests use explicit synchronization:

* events
* queues
* futures
* joins
* timeouts
* controlled fake dependencies

Tests should wait for meaningful conditions, not arbitrary time.

Example idea:

```text
start worker
wait until worker reports ready
send input
wait until output arrives
assert output
stop worker
```

Concurrency tests should also include timeouts so failures do not hang forever.

---

# Debugging Concurrent Code

Debugging concurrent code is hard because observing the program changes timing.

Adding a print statement can make a race condition disappear.

Using a debugger can change scheduling.

Logging can serialize operations enough to hide the bug.

This is why design matters more than heroic debugging.

To debug concurrent programs:

* reduce shared state
* add structured logging with task/thread/process identifiers
* use timeouts
* record state transitions
* make ownership explicit
* reproduce with smaller examples
* test under load
* prefer deterministic coordination in tests

Concurrency bugs often become understandable only after you draw the schedule.

Write down:

```text
worker A does this
worker B does that
where can they interleave?
what state is shared?
what ordering does correctness require?
```

The schedule is the bug's hiding place.

---

# Choosing Not to Use Concurrency

Sometimes the best concurrency decision is not to add it.

Avoid concurrency when:

* the sequential version is fast enough
* the task is simple and run rarely
* correctness is more important than speed
* the bottleneck is algorithmic
* the team cannot maintain the complexity
* the external system cannot handle parallel load
* ordering requirements are strict and complex
* shared mutable state would dominate the design

This is not fear.

It is engineering judgment.

Concurrency is a tool.

Tools have costs.

The professional question is:

```text
does the benefit justify the added reasoning burden?
```

---

# A Practical Design Process

When considering concurrency, use this process:

1. Write or understand the sequential version.
2. Identify the bottleneck.
3. Decide whether work is I/O-bound or CPU-bound.
4. Identify independent units of work.
5. Decide what state is shared.
6. Prefer local state and message passing.
7. Choose a concurrency model.
8. Limit concurrency.
9. Define timeout and cancellation behavior.
10. Define error propagation.
11. Preserve ordering only where required.
12. Test with explicit synchronization.
13. Measure again.

This process is slower than adding a thread immediately.

It is also how you avoid turning a performance problem into a correctness problem.

---

# Common Mistakes

Do not confuse concurrency with parallelism.

Concurrency means overlapping task lifetimes.

Parallelism means simultaneous execution.

Do not assume threads make CPU-bound Python code faster.

In ordinary CPython, the GIL limits pure Python bytecode execution across threads.

Do not share mutable state casually.

Shared mutation is where many concurrency bugs begin.

Do not start unbounded work.

Limit concurrency with pools, queues, semaphores, or explicit capacity rules.

Do not ignore background errors.

Keep handles to tasks, futures, threads, or processes and observe their results.

Do not use arbitrary sleeps as synchronization.

Wait for explicit events or results.

Do not hold locks while doing slow I/O unless you have a strong reason.

That can block other workers unnecessarily.

Do not add concurrency before understanding the bottleneck.

You may make the program harder to debug without making it faster.

Do not assume completion order equals input order.

Preserve order deliberately when it matters.

---

# Mental Checklist

Before using concurrency, ask:

* What problem am I solving?
* Is the work CPU-bound or I/O-bound?
* What can happen independently?
* What must happen in order?
* What state is shared?
* Who owns each piece of mutable state?
* How many workers are allowed?
* What happens when work arrives too fast?
* What happens when one task fails?
* What happens when one task hangs?
* What happens when the program shuts down?
* How will tests wait deterministically?
* How will logs show which worker did what?

If you cannot answer these questions, you are not ready to add concurrency safely.

That does not mean you need a huge design document.

It means you need a clear mental model.

---

# Exercises

1. Explain the difference between concurrency and parallelism using your own example.

2. Classify each task as CPU-bound or I/O-bound: image resizing, API requests, reading a large file, calculating prime numbers, database queries, compressing video.

3. Draw a timeline showing three network requests handled sequentially and concurrently.

4. Explain why a single-core machine can run concurrent programs but not true CPU parallelism on that one core.

5. Write down the possible interleaving for two workers incrementing the same counter.

6. Explain why immutable data is easier to share between concurrent workers.

7. Design a worker system that uses message passing instead of shared mutable state.

8. Describe a deadlock using two locks and two workers.

9. Explain why adding more workers can make a program slower.

10. Take a sequential script you have written and decide whether concurrency would help. Justify your answer.

11. Write a timeout policy for a fake API client: what should happen when a request takes too long?

12. Describe how you would test a background worker without using arbitrary sleep durations.

---

# Summary

Concurrency means multiple tasks are in progress during the same time period.

Parallelism means multiple tasks execute at the same physical instant.

Concurrent programs can overlap waiting time.

Parallel programs can use multiple CPU cores for simultaneous work.

I/O-bound work spends most of its time waiting on external systems.

CPU-bound work spends most of its time computing.

Threads share memory inside one process and are often useful for blocking I/O.

Processes have separate memory and can provide true CPU parallelism.

Async tasks are scheduled cooperatively by an event loop and are useful for high-concurrency I/O when libraries support async operations.

Blocking operations stop the current execution path until they complete.

Schedulers decide what runs next.

Shared mutable state creates race-condition risk.

Locks protect critical sections but can cause deadlocks if used carelessly.

Message passing and clear ownership often make concurrent programs easier to reason about.

Futures represent results that may not be ready yet.

Pools limit and manage reusable workers.

Backpressure handles the reality that systems have finite capacity.

Timeouts and cancellation turn indefinite waiting into explicit policy decisions.

Concurrent error handling must collect failures from background work.

Ordering must be preserved deliberately when it matters.

The deep lesson is:

```text
concurrency is a design problem before it is an API problem
```

If you understand the work, waiting, ownership, limits, and failure behavior, Python's concurrency tools become much easier to choose and use.

---

# Preview of Chapter 66

Chapter 65 built the mental model for concurrency.

We studied overlapping tasks, CPU-bound versus I/O-bound work, blocking, scheduling, shared state, race conditions, locks, deadlocks, futures, pools, backpressure, timeouts, cancellation, and error propagation.

Next we study threads, processes, and the GIL directly.

Chapter 66 will connect the mental model to Python's concrete execution tools:

* `threading`
* `multiprocessing`
* `concurrent.futures`
* `ThreadPoolExecutor`
* `ProcessPoolExecutor`
* worker lifecycle
* `start()`
* `join()`
* locks
* queues
* process isolation
* serialization between processes
* the Global Interpreter Lock
* why threads help I/O-bound work
* why processes help CPU-bound work

The transition is:

```text
concurrency foundations explain the problem
threads, processes, and the GIL explain Python's runtime choices
```

Now that we know what can go wrong, we can study the tools without treating them as magic.
