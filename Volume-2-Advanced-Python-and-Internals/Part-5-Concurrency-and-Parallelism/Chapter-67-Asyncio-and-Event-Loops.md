# Chapter 67 — Asyncio and Event Loops

Threads and processes are not the only way to write concurrent Python.

Python also has `asyncio`.

`asyncio` is Python's standard library for asynchronous I/O.

It is built around:

* coroutines
* `async def`
* `await`
* tasks
* event loops
* cooperative scheduling
* non-blocking I/O
* cancellation
* async context managers
* async iterators

The most important shift from Chapter 66 is this:

```text
threads depend on operating-system scheduling
asyncio depends on cooperative scheduling at await points
```

In threaded code, the operating system can switch between threads.

In async code, a coroutine runs until it reaches an `await` that yields control.

That difference changes how you reason about concurrency.

Asyncio is not "threads but faster."

It is a different model.

It is excellent for many I/O-heavy programs.

It is poor for some CPU-heavy programs.

It is elegant when every layer cooperates.

It is painful when blocking code sneaks into the event loop.

This chapter explains the model.

---

# The Problem Asyncio Solves

Many programs need to manage many waiting operations.

Examples:

* many network connections
* many API calls
* many websocket clients
* many database queries through async drivers
* many timers
* many subprocess waits
* many background I/O tasks

Threads can handle this.

But thousands of threads can become expensive.

Each thread has operating-system overhead.

Each thread has a stack.

The operating system must schedule them.

Shared memory creates synchronization problems.

Asyncio offers another approach:

```text
use one event loop to coordinate many tasks that voluntarily pause while waiting
```

Instead of one thread per waiting operation, many tasks share an event loop.

When one task waits, the event loop runs another ready task.

The result is high-concurrency I/O without needing one operating-system thread per operation.

---

# The Event Loop

The event loop is the scheduler at the center of asyncio.

It keeps track of:

* tasks that are ready to run
* tasks waiting for I/O
* timers
* callbacks
* cancellations
* completed operations

Conceptually:

```text
while program is running:
    run ready task until it awaits
    remember what it is waiting for
    run another ready task
    wake tasks when their awaited operation is ready
```

The event loop is not a magical parallel CPU engine.

Usually, one event loop runs in one thread.

Only one Python coroutine is actively running on that event loop at a time.

The benefit comes from not blocking that thread while operations wait.

Timeline:

```text
task A starts network request and awaits
event loop runs task B
task B awaits timer
event loop runs task C
network response for A arrives
event loop resumes task A
```

The event loop coordinates waiting.

That is the core.

---

# Coroutines

An `async def` function defines a coroutine function.

Example:

```python
async def greet():
    return "hello"
```

Calling it does not run the function body immediately.

```python
result = greet()
```

`result` is a coroutine object.

It is not the string `"hello"`.

This is one of the most important beginner traps in async Python.

Regular function:

```python
def regular():
    return "hello"

value = regular()
```

The function runs immediately.

Async function:

```python
async def async_regular():
    return "hello"

value = async_regular()
```

The function body does not run yet.

You must run or await the coroutine.

---

# Running a Coroutine

At the top level of an ordinary script, use `asyncio.run()`.

```python
import asyncio

async def main():
    print("hello")

asyncio.run(main())
```

`asyncio.run()`:

* creates an event loop
* runs the coroutine
* handles loop shutdown
* closes the loop

It is designed as the main entry point for asyncio programs.

Use it once near the top of the program.

Common shape:

```python
import asyncio

async def main():
    ...

if __name__ == "__main__":
    asyncio.run(main())
```

Inside already-running async code, use `await`, not another `asyncio.run()`.

---

# await

`await` pauses the current coroutine until an awaitable operation completes.

Example:

```python
import asyncio

async def main():
    print("before")
    await asyncio.sleep(1)
    print("after")

asyncio.run(main())
```

`asyncio.sleep(1)` is non-blocking sleep.

It tells the event loop:

```text
pause this coroutine for about one second
run other ready work meanwhile
```

This is different from:

```python
time.sleep(1)
```

`time.sleep(1)` blocks the thread.

Inside async code, blocking the thread blocks the event loop.

That prevents other tasks from running.

The rule:

```text
inside async code, await async operations instead of blocking the event loop
```

---

# Sequential Awaiting

Awaiting two coroutines one after another is still sequential.

```python
import asyncio

async def say_after(delay, message):
    await asyncio.sleep(delay)
    print(message)

async def main():
    await say_after(1, "hello")
    await say_after(2, "world")

asyncio.run(main())
```

This takes about three seconds.

Why?

Because the second call does not start until the first finishes.

This is async code, but not concurrent code.

Async syntax alone does not create concurrency.

To run coroutines concurrently, schedule them as tasks.

---

# Tasks

A task schedules a coroutine to run on the event loop.

Use `asyncio.create_task()`:

```python
import asyncio

async def say_after(delay, message):
    await asyncio.sleep(delay)
    print(message)

async def main():
    task1 = asyncio.create_task(say_after(1, "hello"))
    task2 = asyncio.create_task(say_after(2, "world"))

    await task1
    await task2

asyncio.run(main())
```

This takes about two seconds, not three.

Both tasks are scheduled before either is awaited to completion.

Task creation means:

```text
please run this coroutine concurrently with other tasks
```

Awaiting a task means:

```text
pause here until this scheduled task finishes
```

Do not confuse creating a coroutine object with creating a task.

```python
coro = say_after(1, "hello")
```

This creates a coroutine object.

```python
task = asyncio.create_task(coro)
```

This schedules it.

---

# Coroutine Objects vs Tasks

Coroutine object:

```text
created by calling an async function
not automatically running
can be awaited or scheduled
```

Task:

```text
wraps a coroutine
scheduled on the event loop
runs concurrently with other tasks
can be cancelled
can be awaited for result
```

Example:

```python
async def fetch():
    return "data"

coro = fetch()
task = asyncio.create_task(coro)
result = await task
```

Most application code does not need to create low-level futures.

It works with coroutines and tasks.

Futures appear more often inside libraries and lower-level event loop integrations.

---

# Awaitables

An awaitable is an object that can be used with `await`.

Main awaitable types in asyncio:

* coroutine objects
* tasks
* futures

This works:

```python
result = await some_coroutine()
```

This works:

```python
task = asyncio.create_task(some_coroutine())
result = await task
```

This may work when a library returns a future:

```python
result = await library_future
```

The shared idea is:

```text
await pauses the current coroutine until the awaitable has a result or exception
```

Awaiting does not block the entire event loop.

It pauses only the current coroutine.

That is why other tasks can run.

---

# Asyncio Is Cooperative

Asyncio scheduling is cooperative.

A coroutine must reach an await point to let other tasks run.

Example:

```python
async def busy():
    while True:
        pass
```

This coroutine never awaits.

If it runs, it can monopolize the event loop.

Better:

```python
async def cooperative():
    while True:
        await asyncio.sleep(0)
```

`await asyncio.sleep(0)` can yield control to the event loop.

But do not sprinkle sleep everywhere as a design substitute.

The real solution is to avoid CPU-heavy loops on the event loop.

For CPU-heavy work, use:

* a process pool
* a thread only if the work releases the GIL or is small
* a native library designed for the workload
* a different architecture

Asyncio is for waiting well.

It is not a CPU parallelism tool.

---

# Blocking Code Poisons the Event Loop

This is one of the most important rules in async Python:

```text
do not block the event loop
```

Bad:

```python
import asyncio
import time

async def bad():
    time.sleep(5)

async def main():
    task1 = asyncio.create_task(bad())
    task2 = asyncio.create_task(asyncio.sleep(1))
    await task1
    await task2

asyncio.run(main())
```

`time.sleep(5)` blocks the event loop thread.

During that time, no other asyncio task can run.

Better:

```python
async def good():
    await asyncio.sleep(5)
```

The async sleep lets the event loop run other tasks.

Blocking operations include:

* `time.sleep()`
* synchronous network calls
* synchronous database drivers
* heavy CPU loops
* large file operations
* blocking subprocess waits

Some blocking work can be moved to a thread using `asyncio.to_thread()`.

But the deeper rule remains:

```text
the event loop must stay free to schedule tasks
```

---

# Running Blocking Work in a Thread

Sometimes async code must call a blocking function.

Example:

```python
def read_config_sync(path):
    return path.read_text(encoding="utf-8")
```

Inside async code, you can use `asyncio.to_thread()`:

```python
import asyncio

async def load_config(path):
    text = await asyncio.to_thread(read_config_sync, path)
    return parse_config(text)
```

This runs the blocking function in a separate thread and waits asynchronously for the result.

Use this for:

* small blocking file operations
* legacy synchronous APIs
* blocking libraries you cannot replace yet

Do not treat it as magic.

It still uses threads.

It still has thread overhead.

It does not turn CPU-heavy pure Python work into efficient parallelism in ordinary CPython.

But it can protect the event loop from being blocked.

---

# gather

`asyncio.gather()` waits for multiple awaitables.

Example:

```python
import asyncio

async def fetch(name, delay):
    await asyncio.sleep(delay)
    return name

async def main():
    results = await asyncio.gather(
        fetch("a", 2),
        fetch("b", 1),
        fetch("c", 3),
    )
    print(results)

asyncio.run(main())
```

The result preserves input order:

```python
["a", "b", "c"]
```

Even though `"b"` finishes before `"a"`, the output order follows the order passed to `gather()`.

Use `gather()` when:

* you have a fixed group of awaitables
* you want all results
* input order matters

But understand error behavior.

If one awaitable raises, `gather()` may propagate the exception depending on options and task state.

For structured task management, `TaskGroup` is often clearer in modern Python.

---

# TaskGroup

`asyncio.TaskGroup` provides structured concurrency.

It was added to make groups of related tasks easier and safer to manage.

Example:

```python
import asyncio

async def fetch(name, delay):
    await asyncio.sleep(delay)
    return name

async def main():
    async with asyncio.TaskGroup() as group:
        task_a = group.create_task(fetch("a", 2))
        task_b = group.create_task(fetch("b", 1))

    print(task_a.result())
    print(task_b.result())

asyncio.run(main())
```

The `async with` block waits for tasks in the group before exiting.

The structure says:

```text
these tasks belong together
leaving this block means they are done or handled
```

This helps avoid loose background tasks that outlive the scope that created them.

Structured concurrency is an important design idea:

```text
the lifetime of concurrent work should be visible in the program structure
```

Task groups make that easier.

---

# Fire-and-Forget Is Usually Not Free

It is tempting to write:

```python
asyncio.create_task(do_background_work())
```

and then forget about it.

This is risky.

Questions:

* What if it fails?
* Who observes the exception?
* Can it outlive the request that started it?
* Should it be cancelled on shutdown?
* Does anything keep a strong reference to it?
* What resources does it hold?

If you truly need background tasks, keep track of them.

Example:

```python
background_tasks = set()

task = asyncio.create_task(do_background_work())
background_tasks.add(task)
task.add_done_callback(background_tasks.discard)
```

But even this only manages references.

You still need error handling and shutdown policy.

Most work should belong to a clear scope, such as a `TaskGroup`.

---

# Cancellation

Asyncio uses cancellation heavily.

Cancelling a task asks it to stop.

The task receives `asyncio.CancelledError` at an await point.

Example:

```python
import asyncio

async def worker():
    try:
        while True:
            await asyncio.sleep(1)
    finally:
        print("cleanup")

async def main():
    task = asyncio.create_task(worker())
    await asyncio.sleep(3)
    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print("cancelled")

asyncio.run(main())
```

Use `try/finally` for cleanup.

Usually, do not swallow `CancelledError` silently.

Cancellation is how timeouts, task groups, and shutdown often work internally.

If you catch cancellation and do not re-raise it, you may break higher-level control flow.

The rule:

```text
clean up on cancellation, then usually let cancellation continue
```

---

# Timeouts

Asyncio provides timeout tools.

Modern style:

```python
import asyncio

async def main():
    try:
        async with asyncio.timeout(5):
            await slow_operation()
    except TimeoutError:
        print("operation timed out")
```

Timeouts are cancellation-based.

When the timeout expires, the awaited operation is cancelled.

That means the operation needs proper cleanup.

Timeout design asks:

* Is partial work safe?
* Should the operation retry?
* Should the caller receive a domain-specific error?
* Are resources released?
* Is the cancellation propagated correctly?

Timeouts are not only performance tools.

They are failure boundaries.

---

# Async Context Managers

Async code often needs resources that require asynchronous setup or cleanup.

Examples:

* network connections
* async database sessions
* websocket connections
* client sessions
* locks

Use `async with`:

```python
async with resource:
    ...
```

An async context manager implements:

```python
__aenter__
__aexit__
```

Conceptually:

```text
await resource setup
run block
await resource cleanup
```

This mirrors normal context managers from Chapter 60, but setup and cleanup can await.

Example shape:

```python
async with asyncio.timeout(5):
    await operation()
```

The context manager can coordinate async cleanup when leaving the block.

---

# Async Iterators

Async iterators produce values asynchronously.

Use `async for`:

```python
async for message in websocket:
    handle(message)
```

An async iterator may wait between values.

Examples:

* messages from a websocket
* rows from an async database cursor
* events from a stream
* chunks from an async file or network response

The protocol uses:

```python
__aiter__
__anext__
```

`__anext__` returns an awaitable.

This means each iteration can pause without blocking the event loop.

Normal `for`:

```text
next value is available synchronously
```

Async `for`:

```text
next value may require waiting asynchronously
```

---

# Async Synchronization

Asyncio has synchronization primitives similar in concept to threading tools:

* `asyncio.Lock`
* `asyncio.Event`
* `asyncio.Condition`
* `asyncio.Semaphore`
* `asyncio.Queue`

They are designed for async tasks, not operating-system threads.

Example lock:

```python
lock = asyncio.Lock()

async with lock:
    shared_state.append(value)
```

Example queue:

```python
queue = asyncio.Queue(maxsize=100)

await queue.put(item)
item = await queue.get()
```

Do not use `threading.Lock` as the normal tool for protecting async task coordination.

Use asyncio primitives inside asyncio code.

They cooperate with the event loop.

---

# Semaphores and Limiting Concurrency

A semaphore limits how many tasks can enter a section at once.

Example:

```python
import asyncio

async def fetch(url, semaphore):
    async with semaphore:
        return await fetch_url(url)

async def main(urls):
    semaphore = asyncio.Semaphore(10)
    results = await asyncio.gather(
        *(fetch(url, semaphore) for url in urls)
    )
    return results
```

This limits concurrent fetches to 10.

Why limit concurrency?

Because external systems have limits:

* API rate limits
* database connection limits
* file descriptor limits
* memory limits
* bandwidth limits

Asyncio makes it easy to create many tasks.

That does not mean you should create unlimited pressure.

Concurrency still needs capacity planning.

---

# Async Queues

`asyncio.Queue` is useful for producer-consumer pipelines.

Example:

```python
import asyncio

STOP = object()

async def producer(queue):
    for item in range(10):
        await queue.put(item)
    await queue.put(STOP)

async def consumer(queue):
    while True:
        item = await queue.get()
        try:
            if item is STOP:
                return
            await process(item)
        finally:
            queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=5)

    async with asyncio.TaskGroup() as group:
        group.create_task(producer(queue))
        group.create_task(consumer(queue))

asyncio.run(main())
```

The queue coordinates work without blocking the event loop.

`maxsize` provides backpressure.

If the queue is full, `await queue.put(item)` pauses the producer until space is available.

This is the async version of the bounded queue idea from Chapter 66.

---

# Asyncio and Threads

Asyncio and threads can coexist, but you should keep the boundary clear.

Common reasons to use both:

* async application needs to call a blocking library
* event loop delegates file or CPU-adjacent work to a thread
* a threaded program needs to submit work to an event loop

For simple blocking calls from async code, use:

```python
await asyncio.to_thread(blocking_function, arg1, arg2)
```

For lower-level integration, event loops can run work in executors.

But do not mix models casually.

If most of your libraries are synchronous, a thread-based design may be simpler.

If most of your stack is async-native, keep the event loop clean.

The boundary between async and sync code is an architectural decision.

---

# Asyncio and CPU-Bound Work

Asyncio is not for CPU-bound parallelism.

This is bad:

```python
async def compute():
    total = 0
    for number in range(100_000_000):
        total += number
    return total
```

The function is marked `async`, but it does not await.

It will block the event loop until the loop finishes.

For CPU-bound work, consider:

* `ProcessPoolExecutor`
* native libraries
* batching work outside the event loop
* separate worker services

Asyncio can coordinate CPU work submitted elsewhere.

It should not run long CPU loops on the event loop itself.

The slogan:

```text
asyncio handles waiting, not heavy computation
```

---

# Error Handling in Async Code

Async errors propagate through `await`.

Example:

```python
async def fail():
    raise ValueError("bad")

async def main():
    try:
        await fail()
    except ValueError:
        print("handled")
```

For tasks:

```python
task = asyncio.create_task(fail())

try:
    await task
except ValueError:
    print("handled")
```

If you create a task and never await it or inspect it, its exception may be reported later by the event loop.

This is another reason loose background tasks are risky.

Structured task management helps.

With task groups, related task failures are surfaced at the scope boundary.

The design rule is the same as with futures:

```text
background work needs an owner that observes success or failure
```

---

# Ordering in Async Code

Async tasks may complete in a different order than they were created.

Example:

```python
async def fetch(name, delay):
    await asyncio.sleep(delay)
    return name
```

If you start:

```python
task_a = asyncio.create_task(fetch("a", 3))
task_b = asyncio.create_task(fetch("b", 1))
```

`task_b` completes first.

But if you write:

```python
result_a = await task_a
result_b = await task_b
```

you wait for `a` first.

If you need completion order, use tools designed for that pattern.

If you need input order, `gather()` can preserve it.

Always distinguish:

```text
start order
completion order
result order
```

Async code makes this distinction visible.

---

# A Small Async Pipeline

Here is a simple producer-consumer shape.

```python
import asyncio

STOP = object()

async def producer(queue, values):
    for value in values:
        await queue.put(value)
    await queue.put(STOP)

async def consumer(queue):
    results = []

    while True:
        value = await queue.get()
        try:
            if value is STOP:
                return results
            await asyncio.sleep(0.1)
            results.append(value * 2)
        finally:
            queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=3)

    async with asyncio.TaskGroup() as group:
        consumer_task = group.create_task(consumer(queue))
        group.create_task(producer(queue, range(10)))

    print(consumer_task.result())

asyncio.run(main())
```

This example shows several async ideas together:

* an event loop via `asyncio.run()`
* async functions
* `await`
* a task group
* an async queue
* a sentinel
* backpressure through `maxsize`
* a result collected from a task

This is not the only way to build a pipeline.

It is a compact mental model.

---

# When Asyncio Is a Good Fit

Asyncio is a good fit when:

* you have many I/O-bound operations
* libraries are async-native
* tasks spend most time waiting
* high connection counts matter
* you want structured task lifetimes
* you can keep blocking code out of the event loop

Examples:

* websocket servers
* async web frameworks
* async API clients
* async database clients
* chat systems
* streaming services
* network protocols
* background I/O pipelines

Asyncio shines when the whole stack cooperates.

One blocking library in the middle can ruin the model.

---

# When Asyncio Is a Poor Fit

Asyncio may be a poor fit when:

* the program is CPU-bound
* most libraries are synchronous
* the task count is small
* sequential code is fast enough
* the team is not comfortable with async semantics
* blocking code cannot be isolated
* the domain requires heavy process parallelism

Async code can infect call chains.

If a low-level function becomes async, callers often need to await it.

That can push async upward through the architecture.

This is not bad when the application is designed around async.

It is painful when async is added casually.

Choose async deliberately.

---

# Async All the Way Down

In async programs, the call chain often becomes async.

Example:

```python
async def fetch_user(id):
    ...

async def build_dashboard(id):
    user = await fetch_user(id)
    ...

async def handle_request(request):
    dashboard = await build_dashboard(request.user_id)
    ...
```

Once an operation needs `await`, its caller usually needs to be async too.

This is sometimes called "async all the way down."

It affects architecture.

You cannot hide an async operation behind a normal synchronous function without making a blocking decision somewhere.

That is why async boundaries matter.

Design modules so it is clear which side is async and which side is sync.

---

# Common Mistakes

Do not assume calling an async function runs it.

Calling an async function creates a coroutine object.

Do not forget to await coroutines.

Unawaited coroutines do not do useful work and may produce warnings.

Do not use `time.sleep()` inside async code.

Use `await asyncio.sleep()`.

Do not call blocking network or database libraries directly on the event loop.

Use async libraries or move blocking calls out of the event loop.

Do not create tasks and ignore them.

Keep ownership of background work.

Do not swallow `CancelledError` casually.

Clean up, then usually re-raise.

Do not use unlimited async concurrency.

Use semaphores, queues, batching, or rate limits.

Do not use async syntax for CPU-bound work and expect parallelism.

Asyncio is not a CPU accelerator.

Do not mix threading locks and asyncio synchronization primitives without understanding the boundary.

Use asyncio primitives for coordinating async tasks.

---

# Mental Checklist

Before using asyncio, ask:

* Is the work mostly I/O-bound?
* Are the libraries async-compatible?
* What operations might block the event loop?
* Where is the top-level `asyncio.run()`?
* Which tasks belong together?
* Should I use `TaskGroup`?
* What happens if one task fails?
* What happens if a task is cancelled?
* Are timeouts defined?
* Is concurrency bounded?
* Does result order matter?
* How will background tasks be observed?
* Where is the async/sync boundary?

Asyncio rewards explicit boundaries.

It punishes accidental blocking.

---

# Exercises

1. Write an `async def main()` function and run it with `asyncio.run()`.

2. Call an async function without awaiting it. Observe what object is returned.

3. Write two coroutines using `asyncio.sleep()` and await them sequentially. Measure the total time.

4. Schedule the same two coroutines with `asyncio.create_task()` and compare the total time.

5. Rewrite the task example using `asyncio.TaskGroup`.

6. Replace `await asyncio.sleep()` with `time.sleep()` inside a coroutine and explain why the event loop becomes blocked.

7. Use `asyncio.gather()` to collect results from several coroutines and observe result ordering.

8. Use `asyncio.Semaphore` to limit concurrency to three tasks.

9. Build a small producer-consumer pipeline with `asyncio.Queue`.

10. Add cancellation to a long-running coroutine and use `try/finally` for cleanup.

11. Wrap a slow coroutine in `asyncio.timeout()` and handle `TimeoutError`.

12. Use `asyncio.to_thread()` to call a small blocking function from async code.

13. Explain why async code often becomes "async all the way down."

14. Decide whether an API client, image processor, CLI script, and websocket server are good asyncio candidates.

---

# Summary

`asyncio` is Python's standard library for asynchronous I/O.

It uses an event loop to coordinate many tasks.

`async def` defines a coroutine function.

Calling an async function returns a coroutine object; it does not run the body immediately.

Use `asyncio.run()` as the top-level entry point for an async program.

Use `await` to pause the current coroutine until an awaitable completes.

Sequential awaits are still sequential.

Use tasks to schedule coroutines concurrently.

`asyncio.create_task()` schedules a coroutine as a task.

`TaskGroup` provides structured concurrency for related tasks.

`asyncio.gather()` waits for multiple awaitables and preserves input result order.

Asyncio is cooperative: tasks yield control at await points.

Blocking code blocks the event loop and prevents other tasks from running.

Use async-compatible libraries or move blocking calls to threads with tools such as `asyncio.to_thread()`.

Cancellation is central to asyncio and should be handled with cleanup, usually through `try/finally`.

Timeouts are cancellation-based failure boundaries.

Async context managers and async iterators extend context-manager and iteration ideas into asynchronous code.

Asyncio synchronization primitives coordinate tasks without blocking the event loop.

Asyncio is excellent for high-concurrency I/O.

It is not a substitute for CPU parallelism.

The deep lesson is:

```text
asyncio works when waiting is explicit and cooperative
```

If your code awaits instead of blocks, owns its tasks, limits concurrency, and handles cancellation, async Python becomes a powerful tool rather than a puzzle.

---

# Preview of Chapter 68

Chapter 67 completes Part V on concurrency and parallelism.

We have studied:

* concurrency foundations
* threads
* processes
* futures
* pools
* the GIL
* asyncio
* event loops
* cooperative scheduling

Next we move into Part VI: Type System and Internals.

Chapter 68 begins with the runtime type system.

Python is dynamically typed, but that does not mean types are vague.

Every object has a type.

Types control behavior.

Classes are type objects.

`isinstance()` and `issubclass()` ask runtime type questions.

Protocols, ABCs, generics, annotations, and static type checkers all build on top of the runtime object model.

Chapter 68 will connect:

* dynamic typing
* runtime types
* class objects
* `type`
* `isinstance`
* `issubclass`
* nominal and structural thinking
* annotations as metadata
* the bridge toward static type checking

The transition is:

```text
concurrency explains how work overlaps
the type system explains how Python understands object behavior
```

We now return from execution scheduling to the object model, but at a deeper level.
