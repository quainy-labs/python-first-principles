# Capstone 09 - Mini Event Loop

## Project Brief

In this project, you will build a small event loop.

An event loop is the engine behind asynchronous programming.

It lets one thread coordinate many pieces of waiting work:

```text
timers
callbacks
coroutines
socket reads
socket writes
background tasks
```

The event loop does not make the CPU run many Python instructions at the same instant.

Instead, it helps the program avoid wasting time while operations wait for something external.

The final project will support:

```text
scheduling callbacks soon
scheduling callbacks later
running a loop until completion
Future objects
Task objects
coroutine stepping
sleep
non-blocking socket readiness
basic echo server
task cancellation concepts
tests for scheduling, timers, futures, tasks, and I/O readiness
```

This project is not a production replacement for `asyncio`, Trio, Curio, uvloop, anyio, or a mature async runtime.

It is a learning implementation.

The goal is to understand the ideas that sit underneath `async def`, `await`, `asyncio.create_task`, event loops, non-blocking sockets, and high-concurrency servers.

By the end, the reader should understand how cooperative multitasking works, why async code must yield control, how futures represent pending results, how tasks drive coroutines forward, and how selectors let one thread wait for many sockets.

---

# Why This Project Matters

Earlier capstones built systems that can wait.

The Task Queue waits for jobs.

The Mini Redis server waits for client commands.

The Mini Web Framework waits for HTTP requests.

Waiting is everywhere in real software.

Programs wait for:

```text
network packets
database responses
file descriptors
timers
user input
remote APIs
message queues
locks
```

A simple program may wait by blocking a thread.

That means the thread stops until the operation is ready.

For one operation, that is fine.

For thousands of connections, it becomes expensive.

One thread per connection can consume too much memory and scheduling overhead.

Async programming offers another model:

```text
when this operation would block, pause this task and let another task run
```

The event loop is the coordinator.

It asks:

```text
which callbacks are ready now?
which timers expired?
which sockets can be read?
which sockets can be written?
which coroutine should resume?
```

This capstone makes those questions concrete.

---

# The Mental Model

An event loop is a scheduler.

It owns collections of work:

```text
ready callbacks
timers
waiting I/O operations
tasks waiting on futures
```

Then it repeatedly:

```text
run ready callbacks
run expired timers
wait for I/O readiness
resume tasks whose futures completed
stop when requested
```

The loop is not mysterious.

It is a loop.

The simplified shape is:

```python
while not stopping:
    run_ready_callbacks()
    move_due_timers_to_ready()
    wait_for_io()
```

The powerful part is what the loop coordinates.

A task that cannot continue yields control.

The loop runs something else.

Later, when the task's awaited operation completes, the loop resumes it.

This is cooperative multitasking.

Tasks cooperate by yielding at await points.

---

# What You Will Build

You will build a package named `miniloop`.

Suggested structure:

```text
miniloop/
    __init__.py
    loop.py
    futures.py
    tasks.py
    sockets.py
    server.py
    errors.py
tests/
    test_callbacks.py
    test_timers.py
    test_futures.py
    test_tasks.py
    test_sockets.py
```

The public API should feel small:

```python
from miniloop import EventLoop, sleep

loop = EventLoop()

async def main():
    print("before")
    await sleep(1)
    print("after")

loop.run_until_complete(main())
```

The project should also support lower-level scheduling:

```python
loop.call_soon(print, "hello")
loop.call_later(2.0, print, "later")
loop.run_forever()
```

And eventually non-blocking sockets:

```python
await loop.sock_recv(sock, 4096)
await loop.sock_sendall(sock, data)
```

This gives the reader a line from callbacks to coroutines to I/O.

---

# Requirements

The mini event loop must:

```text
schedule callbacks to run soon
schedule callbacks to run after a delay
run until stopped
run until a future or task completes
represent pending results with Future
represent running coroutines with Task
resume coroutines when awaited futures complete
provide sleep as an awaitable timer
use selectors for socket readiness
support non-blocking socket receive
support non-blocking socket send
handle task exceptions
explain cancellation even if support is minimal
include tests that avoid slow real-time waiting
```

The loop should be single-threaded.

It should be cooperative.

It should not use `asyncio` internally.

Using `selectors`, `socket`, `heapq`, `collections.deque`, and `time.monotonic` is appropriate.

---

# Non-Requirements

The loop does not need to implement all of `asyncio`.

It does not need subprocess support.

It does not need signal handling.

It does not need SSL.

It does not need thread-safe scheduling.

It does not need task groups.

It does not need full cancellation propagation.

It does not need async generators.

It does not need context variables.

It does not need debug tracing.

It does not need high performance.

These are important in mature runtimes.

They are intentionally outside the first implementation.

The goal is clarity.

---

# Synchronous Blocking

Before async makes sense, blocking must be clear.

Consider:

```python
data = sock.recv(4096)
```

If no data is available, this call blocks.

The current thread cannot do other work until data arrives or an error occurs.

That is simple.

It is also limiting.

If a server has 10,000 clients and uses one blocking thread per client, it may spend most resources managing waiting threads.

Non-blocking I/O changes the question.

Instead of:

```text
wait here until this socket has data
```

the program asks:

```text
tell me when any of these sockets has data
```

That is where selectors enter.

---

# Selectors

Python's `selectors` module provides a high-level interface for waiting on file descriptor readiness.

It can watch many sockets.

It can tell the program which sockets are ready for reading or writing.

The shape is:

```python
import selectors

selector = selectors.DefaultSelector()
selector.register(sock, selectors.EVENT_READ, data=callback)

events = selector.select(timeout)
for key, mask in events:
    callback = key.data
    callback()
```

This is not reading data.

This is waiting until reading is likely to succeed without blocking.

The event loop uses this to avoid getting stuck on one socket.

---

# Time

Use monotonic time for scheduling delays.

```python
import time

def current_time():
    return time.monotonic()
```

Wall-clock time can jump.

Monotonic time is designed for measuring elapsed time.

Timers should use monotonic time:

```text
run callback at loop.time() + delay
```

This mirrors the Mini Redis expiration lesson.

Timers and timeouts should generally use monotonic clocks.

---

# Ready Queue

The event loop needs a queue of callbacks ready to run.

Use `collections.deque`.

```python
from collections import deque

self._ready = deque()
```

`call_soon` appends a callback:

```python
def call_soon(self, callback, *args):
    self._ready.append((callback, args))
```

The loop later runs callbacks:

```python
callback, args = self._ready.popleft()
callback(*args)
```

This is the simplest scheduling primitive.

Everything else grows from it.

Tasks are resumed by scheduling callbacks.

Futures notify waiters by scheduling callbacks.

I/O readiness schedules callbacks.

Timers schedule callbacks.

---

# Timer Heap

Callbacks scheduled for later need ordering by time.

Use `heapq`.

```python
import heapq

self._scheduled = []
```

Each timer can be:

```text
(when, sequence_number, callback, args)
```

The sequence number breaks ties.

Without it, Python may try to compare callback functions when two timers have the same time.

That can raise an error.

Scheduling:

```python
def call_later(self, delay, callback, *args):
    when = self.time() + delay
    self._counter += 1
    heapq.heappush(self._scheduled, (when, self._counter, callback, args))
```

Moving due timers:

```python
now = self.time()
while self._scheduled and self._scheduled[0][0] <= now:
    _, _, callback, args = heapq.heappop(self._scheduled)
    self.call_soon(callback, *args)
```

This teaches priority queues in a real system.

---

# The Loop Tick

One loop iteration can be called a tick.

A tick should:

```text
move due timers to ready
compute how long to wait for I/O
wait for I/O readiness
schedule I/O callbacks
run ready callbacks
```

The timeout matters.

If ready callbacks already exist, the loop should not block in `selector.select`.

Use timeout `0`.

If no callbacks are ready but a timer exists, wait until the next timer.

If no callbacks and no timers exist, wait indefinitely for I/O unless stopping.

The logic is:

```python
if self._ready:
    timeout = 0
elif self._scheduled:
    timeout = max(0, self._scheduled[0][0] - self.time())
else:
    timeout = None
```

This is the heartbeat of the loop.

---

# Future

A `Future` represents a result that may not exist yet.

It has states:

```text
pending
finished with result
finished with exception
cancelled
```

The first version can support:

```text
pending
result
exception
```

Methods:

```text
done()
set_result(value)
set_exception(exc)
result()
add_done_callback(callback)
```

A future is not the work itself.

It is a placeholder for the outcome of work.

That is a critical distinction.

Example:

```python
future = Future(loop)

def complete():
    future.set_result("ready")

loop.call_later(1, complete)
```

The future lets code wait for something that will happen later.

---

# Future Callbacks

When a future completes, it should notify callbacks.

```python
def add_done_callback(self, callback):
    if self.done():
        self._loop.call_soon(callback, self)
    else:
        self._callbacks.append(callback)
```

When setting a result:

```python
def set_result(self, value):
    if self.done():
        raise InvalidStateError("future already done")
    self._result = value
    self._done = True
    for callback in self._callbacks:
        self._loop.call_soon(callback, self)
```

Schedule callbacks with the loop.

Do not call them inline.

Scheduling keeps execution order predictable.

It prevents surprising reentrant behavior.

---

# Awaiting Futures

To make a custom future awaitable, implement `__await__`.

Conceptually:

```python
def __await__(self):
    if not self.done():
        yield self
    return self.result()
```

This is the bridge between the event loop and `await`.

When a coroutine awaits a future, it yields that future to the task.

The task registers a callback on the future.

When the future completes, the task is scheduled to step forward.

This is the heart of async execution.

---

# Coroutines

An `async def` function returns a coroutine object when called.

Example:

```python
async def hello():
    return "hello"

coro = hello()
```

At this moment, the body has not run.

The coroutine is an object that can be driven forward.

The event loop must step it.

Under the hood, a coroutine behaves somewhat like a generator.

The task sends values into it.

When the coroutine awaits something, it yields control.

When the awaited thing completes, the task sends the result back in.

This project does not need to expose every protocol detail.

But it should make the stepping process visible enough that `await` no longer feels magical.

---

# Task

A `Task` wraps a coroutine and drives it forward.

It is also a kind of future.

The task's result is the coroutine's return value.

The task's exception is the coroutine's uncaught exception.

The task needs:

```text
coroutine object
loop
future state
step method
current awaited future
```

Creation:

```python
task = Task(coro, loop)
loop.call_soon(task._step)
```

The first step starts the coroutine.

If the coroutine completes, the task completes.

If the coroutine awaits a future, the task pauses until that future is done.

---

# Stepping A Task

The task step method is the hardest part of the project.

Simplified:

```python
def _step(self, value=None, exc=None):
    try:
        if exc is not None:
            awaited = self._coro.throw(exc)
        else:
            awaited = self._coro.send(value)
    except StopIteration as stop:
        self.set_result(stop.value)
        return
    except Exception as error:
        self.set_exception(error)
        return

    awaited.add_done_callback(self._wakeup)
```

The `StopIteration.value` is the coroutine return value.

When the coroutine says:

```python
return 42
```

the task receives that value from `StopIteration`.

This is subtle, but important.

It connects async coroutines to generator mechanics.

---

# Waking A Task

When an awaited future completes, the task should resume.

```python
def _wakeup(self, future):
    try:
        result = future.result()
    except Exception as exc:
        self._loop.call_soon(self._step, None, exc)
    else:
        self._loop.call_soon(self._step, result, None)
```

If the awaited future finished with an exception, throw that exception into the coroutine.

That lets coroutine code handle it:

```python
try:
    value = await something()
except SomeError:
    ...
```

If the future finished successfully, send the result into the coroutine.

This is how `await` expressions produce values.

---

# Sleep

`sleep` is the simplest useful async operation.

It returns an awaitable that completes after a delay.

```python
def sleep(delay, loop=None):
    loop = loop or get_running_loop()
    future = Future(loop)
    loop.call_later(delay, future.set_result, None)
    return future
```

Usage:

```python
async def main():
    print("before")
    await sleep(1)
    print("after")
```

The coroutine pauses at `await sleep(1)`.

The loop schedules the future to complete later.

Other callbacks or tasks can run during the wait.

---

# Running Until Complete

`run_until_complete` receives a future or coroutine.

If it receives a coroutine, wrap it in a task.

Then run the loop until that task is done.

```python
def run_until_complete(self, awaitable):
    task = self.create_task(awaitable)
    while not task.done():
        self._run_once()
    return task.result()
```

This is the easiest way to run a top-level async program.

It mirrors:

```python
asyncio.run(main())
```

The mini version should keep setup and cleanup simple.

---

# Running Forever

`run_forever` runs until stopped.

```python
def run_forever(self):
    self._stopping = False
    while not self._stopping:
        self._run_once()
```

`stop` requests shutdown:

```python
def stop(self):
    self._stopping = True
```

This supports servers.

A server does not usually have one obvious final future.

It runs until interrupted.

---

# Non-Blocking Sockets

Sockets can be set to non-blocking mode.

```python
sock.setblocking(False)
```

In non-blocking mode, operations that would wait raise an exception such as `BlockingIOError`.

The event loop should register interest with the selector instead of blocking.

For reading:

```text
create a future
register socket for read readiness
when socket is readable, try recv
set future result to received bytes
```

For writing:

```text
create a future
register socket for write readiness
when socket is writable, try send
continue until all bytes are sent
set future result
```

This is how one thread can coordinate many sockets.

---

# sock_recv

`sock_recv` should return a future.

```python
def sock_recv(self, sock, max_bytes):
    future = Future(self)

    def on_readable():
        try:
            data = sock.recv(max_bytes)
        except BlockingIOError:
            return
        except Exception as exc:
            self.remove_reader(sock)
            future.set_exception(exc)
        else:
            self.remove_reader(sock)
            future.set_result(data)

    self.add_reader(sock, on_readable)
    return future
```

The actual implementation must avoid completing a future twice.

It should also unregister the file descriptor when done.

For a learning loop, this is enough.

---

# sock_sendall

Sending all bytes may require multiple `send` calls.

`send` may write only part of the buffer.

The loop should keep track of progress.

```python
def sock_sendall(self, sock, data):
    future = Future(self)
    view = memoryview(data)
    offset = 0

    def on_writable():
        nonlocal offset
        try:
            sent = sock.send(view[offset:])
            offset += sent
        except BlockingIOError:
            return
        except Exception as exc:
            self.remove_writer(sock)
            future.set_exception(exc)
            return

        if offset >= len(view):
            self.remove_writer(sock)
            future.set_result(None)

    self.add_writer(sock, on_writable)
    return future
```

This teaches that socket writes are not automatically complete.

Network programming is full of partial progress.

---

# Reader And Writer Registration

The loop needs to register callbacks with the selector.

The selector API associates one file object with an event mask.

If both read and write callbacks exist for the same socket, the loop must register both interests.

A simple approach:

```text
keep dictionaries:
    readers[fileobj] = callback
    writers[fileobj] = callback

when either changes:
    compute mask
    register or modify selector
```

Pseudo-code:

```python
def _update_selector(self, fileobj):
    mask = 0
    if fileobj in self._readers:
        mask |= selectors.EVENT_READ
    if fileobj in self._writers:
        mask |= selectors.EVENT_WRITE

    if mask:
        try:
            self._selector.modify(fileobj, mask)
        except KeyError:
            self._selector.register(fileobj, mask)
    else:
        try:
            self._selector.unregister(fileobj)
        except KeyError:
            pass
```

When readiness events arrive, schedule the appropriate callbacks with `call_soon`.

Do not run them directly inside selector processing if you want consistent scheduling behavior.

---

# Echo Server

An echo server is the best final demonstration.

It accepts a client connection.

It reads bytes.

It writes the same bytes back.

Async version:

```python
async def handle_client(loop, client):
    while True:
        data = await loop.sock_recv(client, 4096)
        if not data:
            client.close()
            return
        await loop.sock_sendall(client, data)
```

Accept loop:

```python
async def serve(loop, server):
    while True:
        client, address = await loop.sock_accept(server)
        client.setblocking(False)
        loop.create_task(handle_client(loop, client))
```

This example shows concurrency clearly.

One client can be waiting for input while another client is being served.

All in one thread.

---

# sock_accept

Accepting connections can also be non-blocking.

```python
def sock_accept(self, server_sock):
    future = Future(self)

    def on_readable():
        try:
            client, address = server_sock.accept()
        except BlockingIOError:
            return
        except Exception as exc:
            self.remove_reader(server_sock)
            future.set_exception(exc)
        else:
            self.remove_reader(server_sock)
            future.set_result((client, address))

    self.add_reader(server_sock, on_readable)
    return future
```

Then an async accept loop can wait for clients without blocking the whole event loop.

---

# Cancellation

Cancellation means requesting that a task stop.

Real cancellation is subtle.

The first version can explain it without fully implementing every edge case.

A task can have:

```python
def cancel(self):
    self._cancel_requested = True
    self._loop.call_soon(self._step, None, CancelledError())
```

The task throws `CancelledError` into the coroutine.

The coroutine may catch it:

```python
try:
    await sleep(10)
except CancelledError:
    cleanup()
    raise
```

If the coroutine suppresses cancellation, should the task be considered cancelled?

What if it awaits another future during cleanup?

What if the awaited future should also be cancelled?

These are real design questions.

For this capstone, implement minimal cancellation or leave it as an extension.

But explain why it is complicated.

---

# Exceptions In Tasks

If a coroutine raises an exception, the task should store it.

When the caller asks for `task.result()`, re-raise it.

```python
async def bad():
    raise ValueError("boom")

task = loop.create_task(bad())
loop.run_until_complete(task)
```

Depending on design, `run_until_complete` may raise `ValueError`.

That is usually what the user expects.

Unhandled task exceptions are a harder topic.

If a background task fails and nobody awaits it, how should the loop report it?

For a mini implementation, a simple log message is enough.

The chapter should still explain the risk:

```text
background task failures should not disappear silently
```

---

# Fairness

An event loop can be starved.

If callbacks keep adding more callbacks, timers and I/O may not get a chance.

One simple policy is to process only the callbacks that were ready at the start of the tick.

```python
ready_count = len(self._ready)
for _ in range(ready_count):
    callback, args = self._ready.popleft()
    callback(*args)
```

Callbacks scheduled during this tick run in the next tick.

This prevents one callback chain from monopolizing the loop forever.

It is not perfect fairness.

But it is an understandable policy.

---

# Blocking The Event Loop

Async code must not block the event loop.

This is bad:

```python
async def handler():
    time.sleep(10)
```

`time.sleep` blocks the whole thread.

No other task can run during those ten seconds.

Use:

```python
await sleep(10)
```

The difference is not cosmetic.

`await sleep(10)` yields control.

`time.sleep(10)` holds control.

This is one of the most important async lessons.

---

# CPU-Bound Work

Async is not a magic solution for CPU-heavy computation.

If a task spends five seconds doing pure CPU work without awaiting, it blocks the event loop for five seconds.

Async helps most when work waits on I/O.

Examples:

```text
HTTP clients
database drivers
network servers
message brokers
websocket connections
timers
```

For CPU-bound work, use:

```text
process pools
worker threads in some cases
native extensions
vectorized libraries
distributed workers
```

This distinction keeps the reader from overusing async.

---

# Testing Philosophy

Event loop tests can become flaky if they rely on real time.

Design the loop so time is injectable.

```python
class EventLoop:
    def __init__(self, clock=time.monotonic):
        self._clock = clock
```

Tests can use:

```python
class FakeClock:
    def __init__(self):
        self.now = 0.0

    def __call__(self):
        return self.now

    def advance(self, seconds):
        self.now += seconds
```

Then timers can be tested instantly.

The loop can expose `_run_once` for tests, or a public `run_once`.

If exposing `run_once` feels too implementation-specific, keep it private but test behavior through public APIs.

The key is to avoid slow sleeps.

---

# Testing Callbacks

Test `call_soon`.

```python
events = []
loop.call_soon(events.append, "hello")
loop.run_once()
assert events == ["hello"]
```

Test callback order.

```text
first scheduled callback runs first
second scheduled callback runs second
```

Test callbacks scheduled by callbacks.

Decide whether they run in the same tick or next tick.

Pin the behavior with a test.

---

# Testing Timers

Use a fake clock.

Schedule:

```python
loop.call_later(10, events.append, "later")
```

Before advancing time:

```text
event has not run
```

After advancing:

```text
event runs
```

Test ordering:

```text
timer at 1 second runs before timer at 5 seconds
timers at same time run in scheduling order
```

This validates the heap design.

---

# Testing Futures

Tests should cover:

```text
new future is not done
set_result marks done
result returns value
set_exception marks done
result re-raises exception
setting result twice fails
callbacks run after completion
callbacks added after completion are scheduled
```

These tests make future behavior trustworthy.

Tasks depend on futures.

If futures are unreliable, the whole event loop becomes confusing.

---

# Testing Tasks

Test a coroutine that returns a value.

```python
async def main():
    return 42

assert loop.run_until_complete(main()) == 42
```

Test a coroutine that awaits sleep.

Use fake time.

Test a coroutine that awaits a future completed later.

Test a coroutine that raises.

Test that exceptions propagate through `run_until_complete`.

Task tests are the point where the event loop begins to feel like async Python.

---

# Testing Sockets

Socket tests should be focused.

Use `socket.socketpair()` when available.

It creates two connected sockets without binding a real port.

```python
left, right = socket.socketpair()
```

Set both to non-blocking mode.

Use `loop.sock_recv(left, 100)`.

Send data from `right`.

Run the loop until the receive future completes.

This tests socket readiness without a full server.

For systems without `socketpair`, use localhost sockets and port `0`.

---

# Milestone 1 - Callback Scheduling

Create `EventLoop`.

Implement:

```text
call_soon
run_once
run_forever
stop
ready queue
```

Write tests for callback execution and order.

At this stage there are no futures, tasks, timers, or sockets.

The loop is already useful because it can schedule work.

---

# Milestone 2 - Timers

Add:

```text
time
call_later
scheduled heap
moving due timers to ready
selector timeout calculation placeholder
```

Use a fake clock in tests.

Test timer order and tie-breaking.

This milestone introduces delayed scheduling.

---

# Milestone 3 - Future

Implement `Future`.

Support:

```text
done
set_result
set_exception
result
add_done_callback
__await__
```

Test all core states.

Do not implement `Task` until `Future` is stable.

Tasks are built on futures.

---

# Milestone 4 - Task

Implement `Task` as a subclass or companion of `Future`.

It should wrap a coroutine and step it.

Support:

```text
successful coroutine result
coroutine exception
awaiting a Future
resuming when awaited Future completes
```

This is the deepest milestone.

Take it slowly.

Add tests for one behavior at a time.

---

# Milestone 5 - sleep

Implement `sleep`.

It should create a future and complete it with a timer.

Test:

```text
sleep does not complete immediately
sleep completes after time advances
coroutine resumes after sleep
```

This milestone makes async code readable.

---

# Milestone 6 - run_until_complete

Implement:

```text
create_task
run_until_complete
get_running_loop
```

The running loop can be stored simply for this project.

Do not support multiple nested loops.

If someone calls `run_until_complete` while the loop is already running, raise a clear error.

This mirrors real async runtime constraints.

---

# Milestone 7 - Selector Integration

Add:

```text
selectors.DefaultSelector
add_reader
remove_reader
add_writer
remove_writer
_process_io_events
```

The loop tick should include selector polling.

If there are ready callbacks, selector timeout should be zero.

If timers exist, timeout should wait until the next timer.

If nothing is scheduled, timeout can be `None`.

Test readiness with `socketpair`.

---

# Milestone 8 - Socket Helpers

Implement:

```text
sock_recv
sock_sendall
sock_accept
```

These should return futures.

They should use reader and writer registration.

They should unregister when complete.

They should set exceptions on socket errors.

This milestone turns the loop into a practical I/O tool.

---

# Milestone 9 - Echo Server

Build a tiny echo server using the event loop.

It should:

```text
create a non-blocking server socket
accept clients asynchronously
spawn a task per client
receive bytes
send bytes back
close clients on EOF
```

Manual test with `nc` or a small socket client.

The point is not a production server.

The point is seeing the event loop coordinate multiple clients.

---

# Milestone 10 - Cancellation And Cleanup

Add minimal cancellation or document it carefully.

If implemented, support:

```text
Task.cancel()
CancelledError
throw cancellation into coroutine
mark task cancelled when appropriate
```

Also make sure socket futures unregister callbacks when completed or failed.

Cleanup matters in long-running loops.

Forgotten file descriptors are real leaks.

---

# Common Mistake 1 - Calling Future Callbacks Immediately

It is tempting to call callbacks directly inside `set_result`.

That can produce surprising reentrant execution.

Scheduling callbacks with `call_soon` is cleaner.

It lets the event loop control execution order.

It also makes behavior easier to test.

---

# Common Mistake 2 - Blocking Inside Async Code

Do not use blocking calls inside tasks.

This includes:

```text
time.sleep
blocking socket recv
blocking socket accept
long CPU loops
```

If a coroutine blocks the thread, the event loop cannot run other tasks.

Async code must yield.

---

# Common Mistake 3 - Forgetting Partial Writes

`socket.send` does not guarantee it writes the full buffer.

Always track how many bytes were sent.

Use `sock_sendall` semantics.

This is a real network programming lesson.

Small local tests may hide the bug because small writes often complete at once.

Larger or congested writes expose it.

---

# Common Mistake 4 - Timer Ties Without A Counter

If two timers have the same scheduled time, a heap may compare the next tuple elements.

If the next element is a function, Python cannot order functions.

Use a sequence number:

```text
(when, sequence, callback, args)
```

This small detail prevents a strange runtime error.

---

# Common Mistake 5 - Losing Task Exceptions

If a task fails and nobody observes it, the error may disappear.

At minimum, store the exception.

When `result()` is called, re-raise it.

For background tasks, consider logging unhandled exceptions.

Invisible async failures are painful to debug.

---

# Common Mistake 6 - Confusing Concurrency With Parallelism

The event loop provides concurrency.

It lets many tasks make progress over time.

It does not make one Python thread execute multiple CPU-bound tasks simultaneously.

Parallelism requires multiple CPU execution contexts.

That usually means multiple processes, native code that releases the GIL, or other runtime support.

This distinction is essential.

---

# Extension Ideas

Add task cancellation fully.

Add `gather`.

Add `wait`.

Add timeout helpers.

Add task names.

Add debug mode that records where tasks were created.

Add unhandled exception reporting.

Add async locks.

Add async queues.

Add stream reader and writer abstractions.

Add a tiny HTTP server on top of the event loop.

Add a simple async Redis-like client.

Add a scheduler for recurring timers.

Add thread-safe `call_soon_threadsafe`.

Add signal handling.

Add context variable propagation.

Add cancellation scopes.

Add structured concurrency ideas.

Each extension moves closer to mature async runtimes.

But the core loop should stay understandable before adding layers.

---

# What This Project Teaches

This project teaches how asynchronous execution works below the friendly syntax.

It teaches callback scheduling.

It teaches timer heaps.

It teaches futures.

It teaches task stepping.

It teaches coroutine mechanics.

It teaches how `await` pauses and resumes computation.

It teaches non-blocking sockets.

It teaches selectors.

It teaches partial reads and writes.

It teaches why blocking code breaks async systems.

It connects many earlier ideas:

```text
deque becomes the ready queue
heapq becomes timer scheduling
callbacks become scheduled work
generators and coroutines explain await
exceptions propagate through tasks
sockets become non-blocking I/O
tests use fake clocks for deterministic time
```

The reader should leave with this mental model:

```text
An event loop is a scheduler that resumes work only when that work can make progress.
```

Once that sentence is clear, `asyncio` becomes much less mysterious.

---

# Completion Checklist

The project is complete when:

```text
call_soon schedules callbacks.
ready callbacks run in predictable order.
call_later schedules delayed callbacks.
timers use monotonic time.
timer ties are stable.
run_forever runs until stopped.
run_until_complete returns coroutine results.
Future supports result and exception completion.
Future callbacks are scheduled through the loop.
Future is awaitable.
Task wraps and steps coroutines.
Task stores coroutine results.
Task stores coroutine exceptions.
Awaiting a future pauses a task.
Completing a future resumes a task.
sleep works as an awaitable timer.
Selector integration can wait for socket readiness.
sock_recv receives data without blocking the loop.
sock_sendall handles partial writes.
sock_accept accepts clients without blocking.
Echo server demonstrates multiple clients.
Tests cover callbacks, timers, futures, tasks, sleep, and sockets.
Documentation explains concurrency vs parallelism.
Documentation explains cancellation limitations.
```

When all of these are true, the reader has built a real miniature event loop.

---

# Exercises

1. Create the `miniloop` package structure.

2. Implement the ready callback queue.

3. Implement `call_soon`.

4. Implement `run_once`.

5. Implement `run_forever` and `stop`.

6. Add tests for callback order.

7. Add monotonic time support.

8. Implement the timer heap.

9. Implement `call_later`.

10. Test delayed callbacks with a fake clock.

11. Implement the `Future` class.

12. Add `set_result`.

13. Add `set_exception`.

14. Add `result`.

15. Add `add_done_callback`.

16. Implement `Future.__await__`.

17. Implement the `Task` class.

18. Step a coroutine to completion.

19. Resume a coroutine after an awaited future completes.

20. Propagate exceptions through tasks.

21. Implement `create_task`.

22. Implement `run_until_complete`.

23. Implement `sleep`.

24. Add selector registration.

25. Implement reader callbacks.

26. Implement writer callbacks.

27. Implement `sock_recv`.

28. Implement `sock_sendall`.

29. Implement `sock_accept`.

30. Build the echo server example.

31. Add socket tests.

32. Document cancellation limitations.

---

# Preview of Capstone 10

Capstone 09 built a Mini Event Loop.

It introduced callback scheduling, timer heaps, futures, tasks, coroutine stepping, sleep, selectors, non-blocking sockets, and cooperative multitasking.

Capstone 10 will build a Toy Python Interpreter.

The Toy Python Interpreter project will move from scheduling Python work to interpreting a small Python-like language.

It will introduce tokenization, parsing, abstract syntax trees, environments, expression evaluation, statements, functions, scope, and interpreter architecture.

The transition is:

```text
Mini Event Loop schedules Python coroutines
Toy Python Interpreter explains how a language can be executed
```

The next project will make the reader build a tiny language runtime and see Python itself from a new angle.
