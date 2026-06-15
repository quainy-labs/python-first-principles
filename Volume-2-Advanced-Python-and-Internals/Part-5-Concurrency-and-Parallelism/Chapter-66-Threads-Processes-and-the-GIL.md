# Chapter 66 — Threads, Processes, and the GIL

Chapter 65 gave us the mental model.

Concurrency means multiple tasks are in progress during the same time period.

Parallelism means multiple tasks execute at the same physical instant.

Now we move from concepts to Python's concrete tools.

Python gives you several ways to run work concurrently:

* threads
* processes
* thread pools
* process pools
* futures
* queues
* locks
* events
* async tasks

This chapter focuses on threads, processes, and the Global Interpreter Lock.

Asyncio gets its own chapter next because it has a different execution model.

The practical question for this chapter is:

```text
when should I use threads, when should I use processes, and why does the GIL matter?
```

The short answer is:

```text
threads are usually useful for overlapping blocking I/O
processes are usually useful for CPU-bound parallelism
the GIL explains why that distinction matters in CPython
```

But short answers are dangerous when they become slogans.

This chapter will build the real model.

---

# The Three Layers

Think of Python concurrency at three layers:

```text
operating system layer
Python runtime layer
application design layer
```

The operating system layer knows about processes and threads.

The Python runtime layer knows about interpreters, frames, bytecode, objects, and the GIL.

The application design layer knows about tasks, errors, timeouts, state ownership, ordering, and shutdown.

Good Python concurrency requires all three.

If you only think at the operating system layer, you may assume threads always use multiple cores for Python code.

That misses the GIL.

If you only think at the Python runtime layer, you may forget that file and network operations wait outside Python.

That misses why threads are still useful.

If you only think at the application design layer, you may choose a nice-looking model that performs poorly because data must be serialized too often.

That misses process overhead.

Concurrency is where layers meet.

---

# Threads Recap

A thread is a path of execution inside a process.

One Python process can have multiple threads.

Those threads share the same memory space.

That means they can access the same module globals, objects, lists, dictionaries, file handles, caches, and class instances.

Shared memory is convenient:

```text
thread A can put a result in a shared queue
thread B can read from that queue
```

Shared memory is also risky:

```text
thread A mutates a dictionary
thread B reads the dictionary at the same time
```

Threads are lightweight compared with processes, but they are not free.

Each thread has:

* an operating-system thread
* a stack
* scheduling overhead
* lifecycle state
* failure behavior

Threads are most natural when the work is blocked on I/O and the data being shared is controlled carefully.

---

# The threading Module

Python's `threading` module provides a higher-level interface over lower-level thread support.

The central class is `threading.Thread`.

Basic example:

```python
import threading

def work():
    print("worker running")

thread = threading.Thread(target=work)
thread.start()
thread.join()
```

This program:

1. Creates a `Thread` object.
2. Starts a new thread of control.
3. Runs `work()` in that thread.
4. Waits for the thread to finish with `join()`.

Creating a `Thread` object does not start execution.

This line only creates the object:

```python
thread = threading.Thread(target=work)
```

This line starts it:

```python
thread.start()
```

This line waits for it:

```python
thread.join()
```

That lifecycle matters.

---

# start and join

`start()` begins the thread's activity.

`join()` waits until the thread terminates.

Example:

```python
import threading
import time

def slow_task(name):
    print(f"{name} started")
    time.sleep(1)
    print(f"{name} finished")

threads = [
    threading.Thread(target=slow_task, args=("A",)),
    threading.Thread(target=slow_task, args=("B",)),
    threading.Thread(target=slow_task, args=("C",)),
]

for thread in threads:
    thread.start()

for thread in threads:
    thread.join()

print("all done")
```

The first loop starts all threads.

The second loop waits for all of them.

Do not do this:

```python
for thread in threads:
    thread.start()
    thread.join()
```

That starts one thread and immediately waits for it before starting the next.

The result is mostly sequential.

The ordering matters:

```text
start all workers
then wait for all workers
```

not:

```text
start one worker
wait for it
start next worker
wait for it
```

---

# Passing Arguments to Threads

Use `args` and `kwargs` to pass arguments.

```python
import threading

def greet(name, punctuation="!"):
    print(f"hello {name}{punctuation}")

thread = threading.Thread(
    target=greet,
    args=("Asha",),
    kwargs={"punctuation": "."},
)

thread.start()
thread.join()
```

`args` is a tuple of positional arguments.

`kwargs` is a dictionary of keyword arguments.

A common mistake is forgetting the comma in a one-item tuple:

```python
args=("Asha")
```

That is just a string in parentheses.

Use:

```python
args=("Asha",)
```

The comma creates the tuple.

This is the same tuple rule from Volume I.

---

# Thread Names

Threads can have names.

```python
thread = threading.Thread(
    target=work,
    name="report-worker",
)
```

Names are useful for:

* logging
* debugging
* tracebacks
* monitoring
* understanding worker roles

In concurrent programs, logs without worker identity become confusing.

Compare:

```text
started
started
finished
finished
```

with:

```text
report-worker started
email-worker started
report-worker finished
email-worker finished
```

When multiple workers exist, names reduce fog.

---

# Daemon Threads

A daemon thread does not keep the Python program alive.

When only daemon threads remain, the program can exit.

Example:

```python
thread = threading.Thread(target=background_work, daemon=True)
```

Daemon threads are sometimes used for background helpers.

But they are dangerous for important work.

If the interpreter exits while a daemon thread is running, that thread may be stopped abruptly.

It may not:

* close files
* flush buffers
* release resources
* finish transactions
* write final results

For important work, prefer non-daemon threads and a graceful shutdown signal.

The rule:

```text
daemon threads are for disposable background work, not critical cleanup
```

---

# Thread Exceptions

If a thread raises an unhandled exception, that exception does not automatically raise in the thread that created it.

Example:

```python
import threading

def fail():
    raise RuntimeError("worker failed")

thread = threading.Thread(target=fail)
thread.start()
thread.join()

print("main continues")
```

The worker fails.

The main thread may still continue after `join()`.

This is one reason raw threads can be awkward.

If the parent needs the result or exception, you must communicate it.

Common approaches:

* use a `queue.Queue`
* store results in a shared structure protected by a lock
* use `concurrent.futures.ThreadPoolExecutor`
* design worker objects that capture errors

For many application tasks, `ThreadPoolExecutor` is cleaner because exceptions are attached to futures and re-raised when you call `future.result()`.

---

# Shared Data in Threads

Threads share memory.

This means this object is visible to both threads:

```python
items = []
```

If two threads call:

```python
items.append(value)
```

you are sharing mutable state.

Some operations on built-in types are protected internally enough not to corrupt the interpreter's memory.

That does not mean your program logic is correct.

The important question is not only:

```text
will Python crash?
```

It is:

```text
does this sequence of operations preserve the program's invariant?
```

Example:

```python
if user_id not in active_users:
    active_users.add(user_id)
```

Even if individual set operations are safe at a low level, the check-then-act sequence can race.

Thread safety is about whole invariants, not only single method calls.

---

# Locks

Use a lock to protect a critical section.

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    with lock:
        counter += 1
```

The `with lock:` block ensures only one thread at a time runs that section.

Use locks to protect:

* shared counters
* shared dictionaries
* shared caches
* multi-step invariants
* read-modify-write operations
* state transitions

Keep critical sections small.

Bad:

```python
with lock:
    data = download_from_network()
    cache[key] = data
```

This holds the lock while waiting on the network.

Better:

```python
data = download_from_network()

with lock:
    cache[key] = data
```

Only protect the shared state update.

Do slow work outside the lock when possible.

---

# RLock

`threading.RLock` is a reentrant lock.

The same thread can acquire it multiple times.

This can be useful when one locked method calls another locked method on the same object.

Example:

```python
import threading

class Counter:
    def __init__(self):
        self._value = 0
        self._lock = threading.RLock()

    def increment(self):
        with self._lock:
            self._value += 1

    def increment_twice(self):
        with self._lock:
            self.increment()
            self.increment()
```

With a normal `Lock`, this design could deadlock because `increment_twice()` holds the lock and `increment()` tries to acquire it again in the same thread.

With `RLock`, the same thread can re-enter.

Do not use `RLock` automatically.

Sometimes needing it is a sign that the locking design is tangled.

Use it when reentrancy is part of the design, not as a reflex.

---

# Events

`threading.Event` is a simple signaling tool.

One thread can wait for an event.

Another thread can set it.

Example:

```python
import threading
import time

ready = threading.Event()

def worker():
    print("worker preparing")
    time.sleep(1)
    ready.set()

thread = threading.Thread(target=worker)
thread.start()

ready.wait()
print("worker is ready")

thread.join()
```

Events are useful for:

* startup coordination
* shutdown signals
* test synchronization
* one-time readiness notifications

An event is clearer than sleeping for a guessed amount of time.

Bad:

```python
time.sleep(1)
```

Better:

```python
ready.wait(timeout=5)
```

The event communicates intent.

The timeout prevents waiting forever.

---

# Queues

`queue.Queue` is designed for safe communication between threads.

Producer-consumer example:

```python
import queue
import threading

work_queue = queue.Queue()

def worker():
    while True:
        item = work_queue.get()
        try:
            if item is None:
                return
            process(item)
        finally:
            work_queue.task_done()

thread = threading.Thread(target=worker)
thread.start()

for item in items:
    work_queue.put(item)

work_queue.put(None)
work_queue.join()
thread.join()
```

The sentinel `None` tells the worker to stop.

The queue handles synchronization around putting and getting items.

Queues are often better than shared lists because they express ownership transfer:

```text
producer gives work to queue
consumer takes work from queue
```

This reduces direct shared mutation.

---

# Thread Pools

Raw threads are useful for learning and low-level control.

For many real tasks, a thread pool is better.

A pool:

* limits the number of workers
* reuses threads
* provides futures
* manages shutdown
* collects results and exceptions

Python provides `ThreadPoolExecutor` in `concurrent.futures`.

Example:

```python
from concurrent.futures import ThreadPoolExecutor

def fetch(url):
    return download(url)

urls = ["https://example.com/a", "https://example.com/b"]

with ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(fetch, url) for url in urls]

    for future in futures:
        result = future.result()
        handle(result)
```

`submit()` schedules work and returns a future.

`future.result()` waits for the result and re-raises the worker exception if the worker failed.

The `with` block shuts down the executor cleanly.

This is usually preferable to manually creating many `Thread` objects.

---

# Futures

A future represents work that may not be complete yet.

With `concurrent.futures`, futures can come from thread pools or process pools.

Important methods:

```python
future.done()
future.result()
future.exception()
future.cancel()
```

`result()` is important because it retrieves either:

* the returned value
* the exception raised by the worker

Example:

```python
from concurrent.futures import ThreadPoolExecutor

def fail():
    raise ValueError("bad input")

with ThreadPoolExecutor(max_workers=1) as executor:
    future = executor.submit(fail)

    try:
        future.result()
    except ValueError as error:
        print("caught:", error)
```

The exception is not lost.

It is stored in the future and raised when the result is requested.

This is one of the biggest practical reasons to use executors instead of raw background threads.

---

# map vs submit

Executors support both `map()` and `submit()`.

`map()` is convenient when applying one function to many inputs:

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=5) as executor:
    for result in executor.map(fetch, urls):
        handle(result)
```

The results are yielded in input order.

That is useful when order matters.

`submit()` gives more control:

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor(max_workers=5) as executor:
    future_to_url = {
        executor.submit(fetch, url): url
        for url in urls
    }

    for future in as_completed(future_to_url):
        url = future_to_url[future]
        try:
            result = future.result()
        except Exception as error:
            report_failure(url, error)
        else:
            handle(url, result)
```

`as_completed()` yields futures as they finish.

This is useful when:

* completion order matters
* you want early results
* you need per-task error handling
* tasks have very different durations

Use `map()` for simple ordered transformations.

Use `submit()` when each task needs identity, error handling, or flexible result collection.

---

# Processes Recap

A process is an executing program with its own memory space.

When Python uses multiple processes, each process has its own interpreter and object memory.

This means normal Python objects are not automatically shared.

If a child process modifies a list, the parent process does not see that mutation unless you use explicit communication.

Process isolation helps avoid shared-memory races.

It also creates overhead.

Data must usually be serialized to move between processes.

In Python, that often means pickling.

The model:

```text
parent process serializes input
child process receives input
child process computes result
child process serializes result
parent process receives result
```

For large data, this cost can matter.

Processes are powerful, but not free.

---

# The multiprocessing Module

The `multiprocessing` module provides process-based parallelism.

Basic example:

```python
from multiprocessing import Process

def work(name):
    print("hello", name)

if __name__ == "__main__":
    process = Process(target=work, args=("Asha",))
    process.start()
    process.join()
```

The shape resembles threads:

```text
create worker
start worker
join worker
```

But the underlying behavior is different.

This starts a separate process.

That process has separate memory.

The `if __name__ == "__main__":` guard is important for multiprocessing, especially on platforms that use the `spawn` start method.

Without it, child processes may re-import the main module and accidentally start new child processes recursively.

The guard says:

```text
only run process-starting code when this file is the main program
```

Not when it is imported by a child process.

---

# Process Start Methods

Different platforms can start processes in different ways.

Important start methods include:

```text
spawn
fork
forkserver
```

`spawn` starts a fresh Python interpreter process.

The child imports the main module and receives the necessary data.

`fork` creates a child process by copying the current process state at the operating-system level.

`forkserver` uses a server process to fork new child processes.

You do not need to master all details immediately.

But you must understand one practical consequence:

```text
process code must be import-safe
```

Functions submitted to child processes usually need to be defined at module top level.

This is risky:

```python
def main():
    def work(x):
        return x * x
```

A process may not be able to import that nested function.

Prefer:

```python
def work(x):
    return x * x

def main():
    ...
```

This is a serialization and import boundary, not just style.

---

# Pickling and Process Boundaries

Processes do not share ordinary Python memory.

To send work to a child process, Python often pickles arguments.

To send results back, Python often pickles results.

This means arguments and return values must be picklable.

Usually picklable:

* numbers
* strings
* lists of picklable values
* dictionaries of picklable values
* top-level functions
* many dataclass instances

Often not picklable:

* open file objects
* sockets
* locks
* nested functions
* lambdas
* database connections
* generator objects
* many objects tied to system resources

The process boundary forces clarity.

You cannot casually pass a live runtime object to another process.

You must pass data.

That is often a good design pressure.

---

# Process Pools

For many CPU-bound tasks, use `ProcessPoolExecutor`.

Example:

```python
from concurrent.futures import ProcessPoolExecutor

def count_primes(limit):
    count = 0
    for number in range(2, limit):
        if is_prime(number):
            count += 1
    return count

if __name__ == "__main__":
    limits = [50_000, 60_000, 70_000, 80_000]

    with ProcessPoolExecutor() as executor:
        results = list(executor.map(count_primes, limits))

    print(results)
```

The pool manages worker processes.

The executor interface is similar to `ThreadPoolExecutor`.

That is intentional.

The same high-level pattern works:

```text
submit work
receive futures
collect results
handle exceptions
shutdown cleanly
```

The difference is the execution backend.

Thread pool:

```text
workers are threads in one process
```

Process pool:

```text
workers are separate processes
```

---

# Why Processes Help CPU-Bound Work

CPU-bound Python code spends most of its time executing Python bytecode.

In ordinary CPython, the GIL allows only one thread to execute Python bytecode at a time.

Multiple threads may exist.

Only one runs Python bytecode at a given instant.

Separate processes have separate Python interpreters and separate GILs.

That means CPU-bound work can run truly in parallel across processes.

Conceptually:

```text
process 1 has interpreter 1 and GIL 1
process 2 has interpreter 2 and GIL 2
process 3 has interpreter 3 and GIL 3
```

Each process can run on a different CPU core.

This is why process pools are a common answer for CPU-bound pure Python workloads.

But the overhead matters.

If tasks are tiny, process overhead may dominate.

If data is huge, serialization may dominate.

If memory is limited, multiple processes may be too expensive.

Parallelism helps when the work is large enough and independent enough.

---

# The Global Interpreter Lock

The Global Interpreter Lock, usually called the GIL, is a lock inside CPython.

It protects execution of Python bytecode so that only one thread executes Python bytecode at a time in a given interpreter.

This is an implementation detail of CPython, not a rule of the Python language specification itself.

The GIL simplifies parts of CPython's memory management and object model.

Remember that CPython uses reference counting.

Reference counts change constantly:

```text
bind a name
pass an argument
store in a list
return from a function
delete a reference
```

If multiple threads could freely mutate interpreter internals at the same time, CPython would need much more fine-grained synchronization around object state and reference counts.

The GIL is one large coordination mechanism.

It has benefits.

It also has performance consequences.

---

# What the GIL Does Not Mean

The GIL does not mean Python cannot do concurrency.

Threads can still overlap I/O waiting.

The GIL does not mean Python cannot use multiple cores.

Python can use multiple cores through multiple processes, native extensions, libraries that release the GIL, and emerging free-threaded builds.

The GIL does not make your data structures logically race-free.

You can still have race conditions in your application logic.

The GIL does not mean locks are unnecessary.

If your program has a multi-step shared-state invariant, you still need synchronization.

The GIL does not make all operations atomic at the level your program cares about.

It is a runtime implementation lock, not an application correctness proof.

---

# Why Threads Still Help I/O

If only one thread can execute Python bytecode at once, why are threads useful?

Because many operations wait outside Python bytecode execution.

When a thread waits for I/O, it may release the GIL or block in operating-system code while another thread can run Python.

Examples:

* waiting for network
* waiting for disk
* waiting for subprocess output
* sleeping
* waiting on locks or queues

Suppose three threads fetch URLs.

Each spends most time waiting for network.

While thread A waits, thread B can run.

While thread B waits, thread C can run.

The program can overlap waiting time even though Python bytecode execution is still constrained.

This is why threads often help I/O-bound workloads.

They are not magic CPU accelerators.

They are a way to avoid wasting time while operations block.

---

# Libraries That Release the GIL

Some C extensions release the GIL while doing long-running native work.

This can allow parallelism even with threads.

Examples may include parts of numerical, compression, hashing, image-processing, or scientific libraries, depending on implementation.

The exact behavior is library-specific.

Do not assume.

Measure.

The general model:

```text
pure Python CPU loop -> usually constrained by the GIL
C extension that releases GIL -> may run in parallel across threads
I/O wait -> threads can overlap waiting
```

This is why performance advice must be specific.

The question is not only:

```text
is this CPU-bound?
```

It is also:

```text
where is the CPU work happening?
```

Pure Python code and native extension code can behave differently.

---

# Free-Threaded Python

Recent Python versions have introduced experimental or optional free-threaded builds that can disable the GIL.

This is important for Python's future.

But it is not the default assumption for most Python environments today.

For most ordinary CPython installations, you should still understand the GIL model.

Even in a free-threaded future, concurrency does not become simple.

Removing the GIL can increase true parallelism for threads.

It can also make data-race discipline even more important.

The application-level lessons remain:

* avoid shared mutable state when possible
* protect invariants
* use queues and ownership
* handle cancellation and shutdown
* measure performance

The GIL is a major runtime detail.

It is not the only concurrency problem.

---

# ThreadPoolExecutor vs ProcessPoolExecutor

Both executors share a similar interface.

Thread pool:

```python
from concurrent.futures import ThreadPoolExecutor
```

Process pool:

```python
from concurrent.futures import ProcessPoolExecutor
```

Both support:

```python
executor.submit(...)
executor.map(...)
future.result()
```

The choice depends on the work.

Use `ThreadPoolExecutor` when:

* tasks are I/O-bound
* tasks share process resources naturally
* data passed to workers is not easily pickled
* startup overhead should be low
* you need many lightweight workers

Use `ProcessPoolExecutor` when:

* tasks are CPU-bound
* work is independent
* data is picklable
* tasks are large enough to justify process overhead
* using multiple CPU cores matters

The shared interface is convenient.

The tradeoffs are not the same.

---

# A Thread Pool Example

Imagine checking many URLs.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
from urllib.request import urlopen

def fetch_size(url):
    with urlopen(url, timeout=10) as response:
        return len(response.read())

urls = [
    "https://www.python.org/",
    "https://docs.python.org/",
    "https://peps.python.org/",
]

with ThreadPoolExecutor(max_workers=5) as executor:
    future_to_url = {
        executor.submit(fetch_size, url): url
        for url in urls
    }

    for future in as_completed(future_to_url):
        url = future_to_url[future]
        try:
            size = future.result()
        except Exception as error:
            print(f"{url} failed: {error}")
        else:
            print(f"{url}: {size} bytes")
```

This is I/O-bound.

The program spends most time waiting for network responses.

Threads can overlap that waiting.

Important details:

* timeout is set
* failures are handled per URL
* `as_completed()` processes results as they finish
* the executor is used as a context manager
* concurrency is limited by `max_workers`

That is professional shape.

---

# A Process Pool Example

Imagine CPU-heavy computation.

```python
from concurrent.futures import ProcessPoolExecutor

def is_prime(number):
    if number < 2:
        return False
    for candidate in range(2, int(number ** 0.5) + 1):
        if number % candidate == 0:
            return False
    return True

def count_primes(limit):
    return sum(1 for number in range(limit) if is_prime(number))

if __name__ == "__main__":
    limits = [100_000, 110_000, 120_000, 130_000]

    with ProcessPoolExecutor() as executor:
        results = list(executor.map(count_primes, limits))

    print(results)
```

This is CPU-bound pure Python work.

A process pool can distribute work across cores.

Important details:

* functions are defined at module top level
* the `if __name__ == "__main__":` guard is present
* inputs and results are simple picklable values
* work chunks are large enough to justify process overhead

This is the kind of workload where processes may beat threads.

---

# Queue-Based Worker Threads

Executors are convenient.

But sometimes you need long-running workers that consume tasks from a queue.

Example:

```python
import queue
import threading

STOP = object()

def worker(name, tasks):
    while True:
        item = tasks.get()
        try:
            if item is STOP:
                return
            process(item)
        finally:
            tasks.task_done()

tasks = queue.Queue()
threads = [
    threading.Thread(target=worker, args=(f"worker-{i}", tasks))
    for i in range(4)
]

for thread in threads:
    thread.start()

for item in items:
    tasks.put(item)

for _ in threads:
    tasks.put(STOP)

tasks.join()

for thread in threads:
    thread.join()
```

This pattern is useful when:

* workers are long-lived
* work arrives over time
* you want backpressure through queue size
* workers own some local setup
* shutdown needs explicit signaling

The sentinel object is important.

It tells each worker to exit.

One sentinel is needed per worker because each sentinel is consumed by one worker.

---

# Bounded Queues

An unbounded queue can grow forever if producers are faster than consumers.

A bounded queue adds backpressure.

```python
tasks = queue.Queue(maxsize=100)
```

If the queue is full, `put()` blocks until space is available.

This prevents producers from flooding memory.

Bounded queues are a practical way to express capacity:

```text
at most 100 pending tasks
```

This is better than pretending infinite memory exists.

For production code, think carefully about:

* queue size
* producer timeout
* consumer failure
* shutdown behavior
* what happens when the queue is full

Backpressure is part of design.

---

# Shutdown

Concurrent programs need explicit shutdown behavior.

Questions:

* How do workers know no more work is coming?
* Should current work finish?
* Should pending work be cancelled?
* How long should shutdown wait?
* What happens if a worker is stuck?
* Are files closed?
* Are queues drained?
* Are subprocesses terminated?

For executors, the context manager handles normal shutdown:

```python
with ThreadPoolExecutor(max_workers=5) as executor:
    ...
```

Leaving the block shuts down the executor and waits for submitted work by default.

For custom threads, you usually need:

* a stop event
* sentinel messages
* queue draining
* joins with timeouts
* clear cleanup in workers

Shutdown is not an afterthought.

It is where many concurrency bugs surface.

---

# Timeouts

Use timeouts around operations that might wait indefinitely.

Examples:

```python
thread.join(timeout=5)
event.wait(timeout=5)
queue.get(timeout=5)
future.result(timeout=5)
```

A timeout is not a complete solution.

It is a decision point.

After a timeout, ask:

* should the program retry?
* should it log and continue?
* should it cancel pending work?
* should it fail the whole operation?
* should it alert an operator?

Timeouts turn silent hanging into explicit policy.

No timeout means:

```text
I am willing to wait forever
```

Sometimes that is true.

Often it is not.

---

# Deadlocks with Futures

Deadlocks can happen even with high-level executors.

Example idea:

```text
thread pool has one worker
task A runs in that worker
task A submits task B to the same pool
task A waits for task B
task B cannot run because the only worker is busy running A
```

The program is stuck.

This is a pool starvation deadlock.

Avoid having tasks wait on other tasks submitted to the same limited pool unless you understand the capacity.

General rule:

```text
worker tasks should not casually block waiting for work from the same saturated worker pool
```

Pools are not magic.

They are limited resources.

---

# Process Pool Pitfalls

Process pools have their own pitfalls.

Common issues:

* forgetting `if __name__ == "__main__":`
* submitting non-picklable functions
* submitting non-picklable arguments
* passing huge data repeatedly
* using too many workers
* relying on global state mutation
* assuming child logs appear in order
* forgetting that startup overhead exists

Example bad pattern:

```python
with ProcessPoolExecutor() as executor:
    results = executor.map(lambda x: x * x, values)
```

The lambda may not be picklable.

Use a top-level function:

```python
def square(x):
    return x * x
```

Process pools reward simple, explicit function boundaries.

---

# Globals in Threads and Processes

Globals behave very differently with threads and processes.

Threads share process memory.

If one thread mutates a global list, other threads see the same list.

Processes have separate memory.

If one process mutates a global list, the parent process usually does not see that mutation.

Example concept:

```text
threads:
    global cache is shared

processes:
    each process has its own cache copy
```

This matters for:

* caches
* counters
* configuration
* database connections
* logging setup
* random seeds
* open files

Do not assume a design that works with threads behaves the same with processes.

The memory model is different.

---

# Logging from Workers

Concurrent logging needs context.

Useful fields include:

* thread name
* process ID
* task ID
* request ID
* input identifier

For threads, thread names help.

For processes, process IDs help.

For task-oriented systems, task IDs help.

Without context, logs from concurrent work interleave:

```text
started
started
failed
finished
```

With context:

```text
worker-1 url=https://example.com/a started
worker-2 url=https://example.com/b started
worker-2 url=https://example.com/b failed timeout
worker-1 url=https://example.com/a finished
```

Concurrency makes structured logs more valuable.

Logs become the timeline you use to understand execution.

---

# Testing Threads and Processes

Testing concurrent code requires deterministic coordination.

Avoid arbitrary sleeps.

Bad:

```python
thread.start()
time.sleep(0.1)
assert state.ready
```

Better:

```python
thread.start()
assert ready_event.wait(timeout=5)
```

Use:

* events
* queues
* futures
* joins with timeouts
* temporary directories
* fake slow dependencies
* small worker counts

Tests should fail quickly if a worker hangs.

That means timeouts belong in tests.

For process tests, also remember:

* functions must be importable
* process startup is slower
* failures may appear through futures or exit codes
* child output may be buffered

Good tests make synchronization explicit.

---

# Measuring

Concurrency advice without measurement is fragile.

Measure:

* sequential baseline
* thread version
* process version
* overhead
* memory usage
* error behavior
* external-system load

Example outcomes:

```text
thread version faster for API calls
process version faster for CPU loops
sequential version faster for tiny tasks
thread version overloads remote service
process version uses too much memory
```

All of these are realistic.

The right answer depends on the workload.

Do not ask:

```text
are threads faster?
```

Ask:

```text
for this workload, on this machine, with this data, under this limit, what happens?
```

That is engineering.

---

# Choosing a Tool

Use raw `threading.Thread` when:

* you need explicit thread lifecycle control
* you have a small number of long-lived threads
* you are building a worker around queues or events
* executor abstraction is too simple for your needs

Use `ThreadPoolExecutor` when:

* you have many similar I/O-bound tasks
* you want futures
* you want simple result collection
* you want managed shutdown

Use raw `multiprocessing.Process` when:

* you need explicit process lifecycle control
* workers are long-lived
* processes have specialized setup
* you need manual IPC

Use `ProcessPoolExecutor` when:

* you have many independent CPU-bound tasks
* arguments and results are picklable
* task size justifies process overhead
* you want futures and simpler result collection

Use no concurrency when:

* the sequential program is fast enough
* the complexity is not justified
* ordering and shared state would dominate
* the bottleneck is not waiting or CPU parallelism

---

# Common Mistakes

Do not call `join()` immediately after each `start()` when you mean to run many threads together.

Start all workers first, then join them.

Do not use threads as a default answer for CPU-bound pure Python loops.

The GIL usually prevents true parallel bytecode execution in ordinary CPython.

Do not ignore exceptions from futures.

Call `future.result()` or otherwise inspect outcomes.

Do not assume daemon threads finish cleanup.

They may be stopped abruptly at interpreter shutdown.

Do not share mutable state without protecting invariants.

The GIL does not make application logic race-free.

Do not forget the `if __name__ == "__main__":` guard for multiprocessing entry points.

Do not pass lambdas, nested functions, open files, locks, or database connections to process pools casually.

They may not be picklable or safe across process boundaries.

Do not submit unlimited work.

Use worker limits, bounded queues, batching, or backpressure.

Do not hold locks while doing slow I/O unless the design truly requires it.

Do not assume output or logs from concurrent workers arrive in meaningful order.

Add context.

---

# Mental Checklist

Before using threads, ask:

* Is this mostly I/O-bound?
* What state is shared?
* Which invariants need locks?
* How will workers stop?
* How will exceptions be reported?
* How many threads are allowed?
* Do logs identify the thread or task?

Before using processes, ask:

* Is this CPU-bound enough to justify process overhead?
* Are arguments and results picklable?
* Are functions importable from the child process?
* Is the `__main__` guard present?
* How large is the data being copied?
* How much memory will workers use?
* How will failures be collected?

Before using an executor, ask:

* Do I need ordered results or completion-order results?
* Should I use `map()` or `submit()`?
* What should happen on timeout?
* What should happen if one task fails?
* How will the executor shut down?

Concurrency tools are easier when the design questions are answered first.

---

# Exercises

1. Write a program that starts three threads, each sleeping for one second, and compare it with running the sleeps sequentially.

2. Modify the thread program so `join()` happens immediately after each `start()`. Explain why the behavior changes.

3. Write a thread worker that communicates completion through a `queue.Queue`.

4. Write a thread worker that signals readiness with `threading.Event`.

5. Create a shared counter protected by `threading.Lock`.

6. Remove the lock and explain why the result may become unreliable under enough concurrency.

7. Use `ThreadPoolExecutor` with `submit()` and collect results with `as_completed()`.

8. Use `ThreadPoolExecutor.map()` and observe that results are returned in input order.

9. Write a CPU-heavy function and run it sequentially, with a thread pool, and with a process pool. Measure each version.

10. Try submitting a lambda to `ProcessPoolExecutor`. Observe and explain the failure.

11. Write a process-pool example with a proper `if __name__ == "__main__":` guard.

12. Add timeouts to `future.result()` and decide what your program should do when a timeout occurs.

13. Create a bounded queue and observe how producers block when consumers are slow.

14. Add thread names to logging output in a concurrent program.

15. Explain in your own words why the GIL does not eliminate the need for locks.

---

# Summary

Threads are execution paths inside one process.

Threads share memory, which makes communication easy and shared-state bugs possible.

Use `threading.Thread` when you need explicit thread lifecycle control.

Use `start()` to begin a thread and `join()` to wait for it.

Use locks to protect shared invariants.

Use events for signaling.

Use queues for safer producer-consumer communication.

Use `ThreadPoolExecutor` for many similar I/O-bound tasks where futures and managed shutdown are helpful.

Processes are independent executing programs with separate memory.

Use processes when CPU-bound work can be split into independent chunks and process overhead is justified.

Use `multiprocessing.Process` for explicit process lifecycle control.

Use `ProcessPoolExecutor` for many independent CPU-bound tasks.

Process workers usually need picklable arguments and return values.

Multiprocessing entry points should use the `if __name__ == "__main__":` guard.

The GIL is a CPython runtime lock that allows only one thread to execute Python bytecode at a time in a given interpreter.

The GIL limits CPU-bound pure Python threading.

Threads remain useful for I/O-bound concurrency because waiting operations can overlap.

Processes can use multiple CPU cores because each process has its own interpreter and GIL.

Executors provide a shared future-based interface over thread pools and process pools.

The deep lesson is:

```text
threads share state, processes isolate state, and the GIL shapes CPU parallelism in CPython
```

Choose the tool that matches the work.

Then design ownership, error handling, timeouts, shutdown, and measurement deliberately.

---

# Preview of Chapter 67

Chapter 66 studied threads, processes, pools, futures, and the GIL.

We saw how Python can overlap I/O with threads and use processes for CPU-bound parallelism.

Next we study `asyncio` and event loops.

Asyncio is Python's standard model for cooperative asynchronous I/O.

It uses:

* `async def`
* `await`
* coroutines
* tasks
* event loops
* non-blocking I/O
* cancellation
* async context managers
* async iterators

The important shift is:

```text
threads rely on operating-system scheduling
asyncio relies on cooperative scheduling through await points
```

Chapter 67 will explain how async execution works, why blocking code harms event loops, when async is a good fit, and how it compares with thread-based concurrency.

The transition is:

```text
threads and processes use OS-level workers
asyncio uses one event loop to coordinate many waiting tasks
```

Once you understand both models, Python's concurrency ecosystem becomes much less mysterious.
