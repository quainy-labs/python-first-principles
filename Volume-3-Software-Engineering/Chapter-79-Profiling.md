# Chapter 79 — Profiling

Profiling is the practice of measuring where a program spends time, memory, or other resources.

Testing asks:

```text
is the behavior correct?
```

Static type checking asks:

```text
are values used consistently?
```

Profiling asks:

```text
where is the program actually spending its resources?
```

That word actually matters.

Performance guesses are often wrong.

The slow part is not always the part that looks complicated.

The expensive function is not always the function with the most code.

The bottleneck may be:

* a tiny function called millions of times
* an accidental database query in a loop
* a slow network call
* repeated JSON serialization
* a large temporary allocation
* a lock that forces waiting
* a missing cache
* a poor algorithm
* unnecessary object creation
* logging too much
* converting data back and forth

Profiling replaces suspicion with evidence.

The central rule is:

```text
measure before optimizing
```

---

# Why Profiling Matters

Optimization without measurement is risky.

You may spend hours improving code that contributes almost nothing to total runtime.

Suppose a request takes two seconds.

You optimize a function from:

```text
10 milliseconds
```

to:

```text
2 milliseconds
```

That is an 80 percent improvement for that function.

But the whole request improves from:

```text
2000 milliseconds
```

to:

```text
1992 milliseconds
```

Users will not notice.

Now suppose the request makes a database query that takes:

```text
1200 milliseconds
```

Reducing that query to 200 milliseconds changes the user experience.

Profiling helps you find the second kind of opportunity.

It keeps performance work honest.

---

# Profiling Is Not Benchmarking

Profiling and benchmarking are related, but they are not the same.

Profiling asks:

```text
where does time or memory go?
```

Benchmarking asks:

```text
how fast is this operation under defined conditions?
```

Profiling gives a breakdown.

Benchmarking gives a comparison.

Example profiling question:

```text
which functions dominate this request?
```

Example benchmarking question:

```text
is implementation A faster than implementation B for this input size?
```

Python's `cProfile` is mainly for profiling.

Python's `timeit` is for timing small code snippets more carefully.

Do not use one tool for every performance question.

Use the tool that matches the question.

---

# Start With a Real Symptom

Do not begin performance work with:

```text
this code looks slow
```

Begin with a symptom:

```text
this API endpoint takes 3 seconds at p95
this import job takes 40 minutes
this CLI command feels slow on large directories
memory grows until the worker restarts
CPU stays at 100 percent during report generation
database load spikes when this feature is used
```

The symptom defines the target.

Without a target, you can optimize endlessly.

Good performance work starts by stating:

* what is slow?
* for whom?
* under what workload?
* how slow is it now?
* how fast does it need to be?
* what resource is constrained?

Only then should you profile.

---

# Reproduce the Workload

Profiling requires a workload.

The workload should resemble the real problem.

If production is slow for 50,000 records, profiling with 10 records may hide the bottleneck.

If a web endpoint is slow only when the user has many permissions, profile that case.

If a report is slow only for a certain customer shape, reproduce that shape.

Example:

```text
small input: 100 rows
realistic input: 100,000 rows
worst-case input: 1,000,000 rows
```

Different input sizes can reveal different bottlenecks.

An algorithm that is fine for 100 rows may collapse at 100,000.

Profiling the wrong workload creates false confidence.

---

# Measure a Baseline

Before changing code, measure the current behavior.

Record:

* input size
* environment
* command
* runtime
* memory usage if relevant
* important configuration
* commit or version

Example:

```text
Command: python -m reports monthly --customer c-123
Input: 84,000 invoices
Runtime: 38.4 seconds
Peak memory: 1.2 GB
Python: 3.12
Commit: abc123
```

The baseline matters because after optimization you need to answer:

```text
did it improve?
by how much?
did anything get worse?
```

Without a baseline, you are relying on feeling.

Feeling is not a performance metric.

---

# CPU, I/O, Memory, Waiting

Before choosing a tool, ask what kind of bottleneck you suspect.

CPU-bound work spends time executing instructions.

Examples:

* parsing
* compression
* encryption
* numerical loops
* image processing
* pure Python transformations

I/O-bound work waits on external systems.

Examples:

* database queries
* network calls
* file reads
* file writes
* subprocesses

Memory-bound work allocates, copies, or retains too much data.

Waiting can also come from:

* locks
* queues
* rate limits
* thread pools
* connection pools
* async scheduling

Different bottlenecks need different profiling methods.

`cProfile` can show function call time.

It may not fully explain why a database query is slow.

`tracemalloc` can show Python memory allocations.

It may not explain native memory used by an extension.

Know what you are measuring.

---

# Wall Time vs CPU Time

Wall time is real elapsed time.

If a request starts at 10:00:00 and ends at 10:00:03, wall time is 3 seconds.

CPU time is time spent actively executing on the CPU.

A program can have high wall time but low CPU time if it is waiting on I/O.

Example:

```text
wall time: 3 seconds
CPU time: 50 milliseconds
```

This suggests waiting, not computation.

Maybe the program is waiting for a database or network response.

Another example:

```text
wall time: 3 seconds
CPU time: 2.9 seconds
```

This suggests CPU-bound work.

Optimization strategy depends on which time dominates.

Do not optimize Python loops when the program is mostly waiting for a remote API.

---

# cProfile

`cProfile` is Python's standard deterministic profiler implemented in C.

It is recommended for most standard-library profiling use.

Run a script:

```bash
python -m cProfile script.py
```

Run a module:

```bash
python -m cProfile -m package.module
```

Sort output:

```bash
python -m cProfile -s cumulative script.py
```

Save output:

```bash
python -m cProfile -o profile.stats script.py
```

`cProfile` records function calls and timing information.

It answers questions like:

* which functions were called?
* how often?
* how much time did each function spend internally?
* how much cumulative time includes subcalls?

This is often the first profiler to reach for in Python.

---

# Reading cProfile Output

Typical columns include:

```text
ncalls
tottime
percall
cumtime
percall
filename:lineno(function)
```

`ncalls` is the number of calls.

`tottime` is time spent inside that function, excluding subcalls.

`cumtime` is cumulative time spent in that function and all subcalls.

`percall` divides time by number of calls.

Example:

```text
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
100000    0.800    0.000    1.200    0.000 parser.py:22(parse_row)
```

This tells you `parse_row` was called 100,000 times.

Its total cumulative cost matters because it is repeated.

A small per-call cost can become large when multiplied.

Profiling teaches you to care about both:

```text
expensive once
cheap but repeated many times
```

---

# tottime vs cumtime

Sort by `tottime` when you want functions that spend time in their own body.

Sort by `cumtime` when you want functions whose full call tree is expensive.

Example:

```text
main -> build_report -> load_data -> query_database
```

`build_report` may have high `cumtime` because it calls expensive functions.

`query_database` may have high `tottime` if the call itself waits.

If you only look at `tottime`, you may miss high-level operations that organize expensive work.

If you only look at `cumtime`, you may blame wrapper functions that are not the true bottleneck.

Use both views.

Ask:

```text
is this function expensive itself, or does it call expensive things?
```

---

# pstats

The `pstats` module reads and formats profiler output.

Example:

```python
import pstats
from pstats import SortKey


stats = pstats.Stats("profile.stats")
stats.strip_dirs().sort_stats(SortKey.CUMULATIVE).print_stats(20)
```

This prints the top 20 entries by cumulative time.

Sort by internal time:

```python
stats.strip_dirs().sort_stats(SortKey.TIME).print_stats(20)
```

Print callers:

```python
stats.print_callers()
```

Print callees:

```python
stats.print_callees()
```

`pstats` helps you analyze saved profiling data after the run.

Saving profile data is useful when runs are expensive or need comparison.

---

# Profiling a Function

You can profile a specific function programmatically.

Example:

```python
import cProfile
import pstats
from pstats import SortKey


def run_profile():
    profiler = cProfile.Profile()
    profiler.enable()
    build_report()
    profiler.disable()

    stats = pstats.Stats(profiler)
    stats.strip_dirs().sort_stats(SortKey.CUMULATIVE).print_stats(20)
```

This is useful when:

* you do not want to profile startup
* you only care about one workflow
* setup is expensive but not part of the performance target
* you want profiling inside a test-like harness

Profile the region that matches the question.

Do not include unrelated setup unless setup is part of the problem.

---

# Profiling With a Context Manager

`cProfile.Profile` can be used as a context manager.

Example:

```python
import cProfile
import pstats
from pstats import SortKey


with cProfile.Profile() as profiler:
    build_report()

pstats.Stats(profiler).sort_stats(SortKey.CUMULATIVE).print_stats(20)
```

This keeps the profiled region clear.

The code says:

```text
measure only this block
```

That clarity matters.

Profiling too much can drown the interesting signal in startup, imports, test setup, and framework machinery.

---

# Profiling Overhead

Profilers add overhead.

`cProfile` changes runtime because it records function call events.

That is usually acceptable for finding relative hotspots.

It is not ideal for precise benchmarking.

This is why the official docs distinguish profiling from benchmarking.

When profiling, focus on proportions and hotspots:

```text
where is most time going?
which functions dominate?
what is called too often?
```

When benchmarking, use tools designed for timing comparisons.

Do not treat profiled runtime as exact production runtime.

The profiler is an instrument.

Instruments affect measurements.

Good engineers account for that.

---

# Deterministic Profiling

`cProfile` and `profile` are deterministic profilers.

They trace function call and return events.

This provides detailed call statistics.

Another style is statistical profiling, where a profiler samples the program periodically and estimates where time is spent.

Statistical profilers often have lower overhead and can be useful for production-like systems.

The standard library's main built-in profiler is deterministic.

Understanding the style helps interpret results.

Deterministic profiling is excellent for function-call-heavy Python code.

It may distort very small functions or code dominated by C extensions and external waiting.

---

# timeit

`timeit` measures execution time of small code snippets.

Example:

```bash
python -m timeit '"-".join(str(i) for i in range(100))'
```

In Python:

```python
import timeit


duration = timeit.timeit(
    '"-".join(str(i) for i in range(100))',
    number=10_000,
)
print(duration)
```

`timeit` is useful for comparing small alternatives:

```python
"".join(parts)
```

versus:

```python
result = ""
for part in parts:
    result += part
```

But microbenchmarks can mislead.

A faster snippet may not matter in the real program.

Use `timeit` after profiling identifies a small operation worth investigating.

---

# Microbenchmarks

A microbenchmark measures a tiny operation.

Microbenchmarks are useful when:

* the operation is truly hot
* alternatives are small and isolated
* input sizes are representative
* you understand setup cost

They are dangerous when:

* the measured code is not a bottleneck
* the input is unrealistic
* setup dominates the measurement
* caching changes behavior
* interpreter warmup or system noise distorts results
* you generalize too broadly

Example mistake:

```text
I found this expression is 20% faster in timeit, so the application will be faster.
```

Maybe.

Maybe not.

Only end-to-end measurement can confirm application improvement.

---

# Use Representative Data

Performance depends on data shape.

Sorting 10 items is different from sorting 10 million.

Parsing one JSON object is different from parsing a 200 MB file.

Checking membership in a list of 5 items is different from a list of 500,000.

When profiling, record input characteristics:

* size
* distribution
* missing values
* nesting depth
* duplicate rate
* worst-case cases
* common-case cases

An optimization for rare worst-case input may not help normal users.

An optimization for average input may still leave worst-case failures.

Know which case you are optimizing.

---

# Algorithmic Complexity

Profiling tells you where time goes.

Algorithmic thinking explains how time grows.

Example:

```python
def has_duplicates(values):
    for i, left in enumerate(values):
        for right in values[i + 1:]:
            if left == right:
                return True
    return False
```

This compares many pairs.

For large lists, it becomes expensive.

A set-based version:

```python
def has_duplicates(values):
    seen = set()
    for value in values:
        if value in seen:
            return True
        seen.add(value)
    return False
```

Profiling may show the nested loop is slow.

Complexity explains why it gets worse as input grows.

Optimization is not only about faster syntax.

Often it is about better algorithms.

---

# Call Counts

Sometimes the problem is not that a function is slow.

The problem is that it is called too often.

Example:

```text
normalize_email called 4,000,000 times
```

Maybe the code normalizes the same email repeatedly.

Possible fixes:

* normalize once at input
* cache repeated results
* move work outside a loop
* batch operations
* change data structure

Call count is one of the most important profiler columns.

A function with tiny `tottime` per call can dominate runtime if called enough times.

Read `ncalls`.

Do not only read time.

---

# The N Plus One Problem

The N plus one problem happens when code performs one query to get N items and then one additional query per item.

Example:

```python
orders = get_orders()
for order in orders:
    order.customer = get_customer(order.customer_id)
```

If there are 1,000 orders, this may run 1,001 queries.

The Python profiler may show time in database call wrappers.

Application logs or database logs may show the repeated queries.

The fix may be:

* join data
* prefetch related records
* batch lookup
* cache within request

This is a common performance issue in web applications and ORMs.

Profiling should include external boundaries, not just Python function timing.

---

# I/O Profiling

If a program waits on I/O, Python-level CPU profiling may not tell the full story.

For I/O-heavy code, collect:

* request duration
* database query timing
* network call timing
* file read/write timing
* retry counts
* timeout counts
* queue wait time

Example instrumentation:

```python
start = time.perf_counter()
response = client.get(url)
duration_ms = (time.perf_counter() - start) * 1000
logger.info("external_call service=%s duration_ms=%.2f", service, duration_ms)
```

This is not a replacement for full tracing.

It is a practical start.

If wall time is high but CPU time is low, look at I/O.

---

# Database Profiling

Database performance needs database evidence.

Look at:

* query duration
* query count
* query plan
* indexes
* row counts
* locks
* connection pool waits
* transaction duration

Python profiling may show:

```text
db.execute
```

taking a lot of cumulative time.

That tells you Python is waiting in database calls.

It does not tell you whether the SQL needs an index, the query returns too many rows, or the connection pool is exhausted.

Use database tools for database problems.

Profiling points to the boundary.

Then the boundary needs its own tools.

---

# Memory Profiling

Performance is not only time.

Memory matters too.

Symptoms:

* process grows over time
* worker is killed by the operating system
* garbage collection becomes expensive
* large inputs fail
* swapping slows the machine
* containers hit memory limits

Memory issues may come from:

* loading everything at once
* retaining references
* unbounded caches
* large temporary lists
* repeated copying
* reference cycles
* queues growing faster than workers process
* native extension allocations

Memory profiling asks:

```text
where are allocations happening?
what objects stay alive?
what data could stream instead of accumulate?
```

---

# tracemalloc

`tracemalloc` traces Python memory allocations.

Example:

```python
import tracemalloc


tracemalloc.start()

run_workload()

snapshot = tracemalloc.take_snapshot()
top = snapshot.statistics("lineno")

for stat in top[:10]:
    print(stat)
```

This shows allocation hotspots by line.

You can compare snapshots:

```python
before = tracemalloc.take_snapshot()
run_workload()
after = tracemalloc.take_snapshot()

for stat in after.compare_to(before, "lineno")[:10]:
    print(stat)
```

This helps find where memory increased during a workload.

`tracemalloc` tracks Python allocations.

It may not capture all memory used by native extensions or external libraries.

Use it as evidence, not as the entire truth.

---

# Peak Memory

Sometimes peak memory matters more than final memory.

Example:

```python
data = path.read_text()
rows = parse(data)
```

This may hold both raw text and parsed rows at the same time.

A streaming approach may reduce peak memory:

```python
with path.open() as file:
    for row in parse_rows(file):
        process(row)
```

Peak memory matters in:

* data imports
* report generation
* file processing
* ETL jobs
* ML preprocessing
* web requests with large payloads

Profiling should match the resource constraint.

If the process is killed for memory, shaving CPU time may not help.

---

# Allocation Is Work

Creating objects costs time and memory.

Example:

```python
normalized = [
    normalize(row)
    for row in rows
]
```

This builds a full list.

If you only need to stream results:

```python
for row in rows:
    process(normalize(row))
```

or:

```python
normalized = (normalize(row) for row in rows)
```

Generators can reduce memory.

But they are not automatically faster.

They trade memory behavior for lazy execution.

Profile both if performance matters.

---

# Caches

Caching can improve performance by avoiding repeated work.

Example:

```python
from functools import lru_cache


@lru_cache(maxsize=1024)
def load_country(code: str) -> Country:
    ...
```

Caching helps when:

* inputs repeat
* computation is expensive
* results are stable enough
* memory cost is acceptable

Caching hurts when:

* inputs rarely repeat
* values become stale
* memory grows without bound
* invalidation is hard
* cached results are huge

Profiling can show repeated calls.

That suggests caching might help.

But caching is a design decision, not a reflex.

Measure after adding it.

---

# The GIL and Profiling

In CPython, the Global Interpreter Lock affects threading behavior.

For CPU-bound Python code, multiple threads may not speed up execution.

Profiling may show CPU-bound work inside Python functions.

Options include:

* better algorithm
* vectorized library
* multiprocessing
* native extension
* Cython or Rust extension
* moving work to a service

For I/O-bound work, threads or async can help because the program spends time waiting.

Profiling should reveal whether the work is CPU-bound or waiting.

Do not add threads before knowing which problem you have.

Concurrency is not a seasoning.

It is a design response to a specific bottleneck.

---

# Async Profiling

Async programs introduce waiting and scheduling.

A slow async flow may be slow because:

* a task is CPU-bound and blocks the event loop
* a network call is slow
* too many tasks run concurrently
* too few tasks run concurrently
* a semaphore is too restrictive
* a task is never awaited
* a connection pool is exhausted

Function profilers can help, but async systems often need:

* timing around awaits
* logs with request or task IDs
* tracing
* event loop diagnostics
* external service timings

If the event loop is blocked by CPU work, other tasks wait.

A profiler may show the CPU-heavy function.

If the event loop is waiting on network, trace timings matter more.

---

# Profiling Tests

Tests can help create repeatable profiling workloads.

But do not turn ordinary unit tests into performance tests by accident.

A profiling harness may look like:

```python
def make_large_input() -> list[Row]:
    ...


def run_workload() -> None:
    process_rows(make_large_input())
```

Then profile `run_workload`.

Keep performance experiments separate from correctness tests unless the project intentionally has benchmark tests.

Unit tests should be fast and deterministic.

Performance tests may need larger inputs and different infrastructure.

---

# Performance Regression Tests

Sometimes performance becomes part of the contract.

Example:

```text
importing 100,000 rows should complete under 10 seconds on CI hardware
```

Be cautious.

Timing tests can be flaky because CI machines vary.

Better regression checks may assert:

* query count does not exceed a threshold
* algorithm does not call expensive dependency repeatedly
* memory growth stays under a broad limit
* function call count remains reasonable

Example:

```python
assert query_counter.count <= 3
```

This may be more stable than:

```python
assert duration < 0.2
```

Performance regression tests should be designed carefully.

---

# Optimize the Bottleneck

After profiling, choose the bottleneck that matters.

Ask:

* does this hotspot affect the user-visible symptom?
* is it on the critical path?
* how much total time or memory does it represent?
* can it be improved safely?
* will the improvement be measurable?
* what complexity will the optimization add?

Do not optimize a function just because it appears in the profile.

Some functions are naturally central.

For example, a framework request dispatcher may show high cumulative time because all requests pass through it.

You may not be able or need to optimize it.

Find the actionable bottleneck.

---

# Keep Optimizations Honest

Every optimization should be measured after implementation.

Record before and after:

```text
Before: 38.4 seconds
After: 9.7 seconds
Input: 84,000 invoices
```

Also check correctness:

```bash
pytest
```

Optimization that changes behavior is a bug.

Optimization that helps one case but hurts another may still be wrong.

Measure:

* common case
* worst case
* memory
* correctness
* maintainability impact

Fast wrong code is not an improvement.

---

# Simple Wins

Common performance wins include:

* avoid repeated work
* move invariant work outside loops
* use appropriate data structures
* batch database queries
* stream large files
* avoid unnecessary conversions
* use built-in functions where clear
* avoid repeated regex compilation
* cache stable expensive results
* reduce logging in hot paths
* avoid loading unused data

Example:

```python
for row in rows:
    pattern = re.compile(r"\d+")
    pattern.search(row.text)
```

Better:

```python
pattern = re.compile(r"\d+")

for row in rows:
    pattern.search(row.text)
```

This is simple and clear.

The best optimizations often remove waste rather than add cleverness.

---

# Data Structures Matter

Choosing the right data structure can dominate performance.

Membership in a list is linear:

```python
if user_id in user_ids_list:
    ...
```

Membership in a set is usually much faster for large collections:

```python
user_ids = set(user_ids_list)

if user_id in user_ids:
    ...
```

But building the set has a cost.

If you check membership once, a set may not help.

If you check thousands of times, it likely will.

Profiling and complexity thinking work together.

---

# Built-ins and Libraries

Python built-ins and standard-library tools are often implemented efficiently.

Examples:

* `sum`
* `min`
* `max`
* `sorted`
* `collections.Counter`
* `collections.defaultdict`
* `itertools`
* `bisect`
* `heapq`
* `array`

Using them can improve clarity and performance.

But do not contort code to use a built-in if the result becomes unreadable.

Performance is one design constraint.

Maintainability is another.

Professional optimization balances both.

---

# Vectorization

For numerical and tabular workloads, Python loops may be the bottleneck.

Libraries like NumPy, pandas, Polars, and similar tools can move work into optimized native code.

Example concept:

```python
for row in rows:
    row["total"] = row["price"] * row["quantity"]
```

In a dataframe library, the operation may be expressed over whole columns.

This can be much faster for large data.

But vectorization is not free.

It may require different data structures, memory layout, and mental model.

Use it when the workload justifies it.

Volume IV will study data libraries more deeply.

---

# When Not to Optimize

Do not optimize when:

* there is no performance problem
* the code is not on a hot path
* the optimization makes code much harder to understand
* the improvement is not measurable
* the workload is unrealistic
* the bottleneck is elsewhere
* correctness is not protected by tests

Optimization has cost.

It can add:

* complexity
* caching bugs
* invalidation problems
* concurrency issues
* memory usage
* maintenance burden

The fastest code is not always the best code.

The best code meets requirements with appropriate clarity.

---

# Profiling in Production

Production profiling requires care.

You may not be able to attach heavy profilers freely.

Concerns include:

* overhead
* privacy
* security
* service reliability
* representative traffic
* permissions
* data retention

Production-safe approaches may include:

* metrics
* tracing
* sampled profiling
* slow query logs
* request duration logs
* feature-specific instrumentation
* profiling a canary instance

Do not experiment recklessly on production systems.

But do not ignore production evidence either.

Performance problems often appear only under real traffic and real data.

---

# Profiling Checklist

Before optimizing, ask:

* What is the performance symptom?
* What workload reproduces it?
* What is the baseline?
* Is the bottleneck CPU, I/O, memory, or waiting?
* Which profiler or measurement tool matches that bottleneck?
* Is the input representative?
* Which functions dominate time or memory?
* Which call counts are suspicious?
* Are external systems involved?
* Is the proposed optimization measurable?
* Are correctness tests in place?
* Did the optimization improve the real workload?
* Did it harm readability, memory, or another path?

This checklist prevents performance work from becoming superstition.

---

# Common Profiling Mistakes

Common mistakes include:

* optimizing before measuring
* profiling unrealistic inputs
* confusing profiling with benchmarking
* reading only the top line of profiler output
* ignoring call counts
* ignoring I/O and database timing
* treating profiler runtime as exact benchmark runtime
* optimizing startup when the symptom is request latency
* optimizing CPU when the program waits on network
* ignoring memory peaks
* adding caches without invalidation strategy
* making code unreadable for tiny gains
* failing to measure after changes
* failing to run correctness tests after optimization

The cure is discipline:

```text
measure
change
measure again
verify behavior
```

---

# Chapter Summary

Profiling measures where a program spends time, memory, or other resources.

Performance work without profiling is often guesswork.

Profiling is different from benchmarking.

Profiling identifies where resources go.

Benchmarking compares performance under defined conditions.

Start with a real symptom and a representative workload.

Measure a baseline before changing code.

Identify whether the bottleneck is CPU, I/O, memory, or waiting.

`cProfile` is Python's standard deterministic profiler and is useful for function call timing.

`pstats` helps analyze saved profiling data.

`timeit` measures small snippets and is useful for microbenchmarks after a real hotspot is identified.

`tracemalloc` traces Python memory allocations and can compare snapshots.

Read `ncalls`, `tottime`, and `cumtime` carefully.

Small functions called many times can dominate runtime.

High cumulative time may point to expensive subcalls.

Python profilers may not fully explain database, network, native extension, or operating-system bottlenecks.

Use boundary-specific tools when needed.

Algorithmic complexity and profiling complement each other.

Caching, batching, streaming, better data structures, and built-ins can all help when applied to real bottlenecks.

Every optimization should be measured after implementation.

Correctness must be protected.

The central lesson is:

```text
profiling turns performance work from guessing into investigation
```

Optimization should be evidence-driven, measured, and justified by the real workload.

---

# Exercises

1. Use `python -m cProfile` on a small script and identify the top five functions by cumulative time.

2. Save profiler output to a file and read it with `pstats`.

3. Compare sorting by `tottime` and `cumtime` on the same profile.

4. Use `timeit` to compare two small string-building approaches.

5. Create a function with an accidental nested loop and profile it with increasing input sizes.

6. Use `tracemalloc` to find the largest allocation lines in a workload.

7. Replace repeated list membership checks with a set and measure before and after.

8. Add timing logs around a fake external API call.

9. Create a profiling baseline note for a command-line workload.

10. Optimize one measured bottleneck, then rerun tests and record before/after measurements.

---

# Preview of Chapter 80

Chapter 79 studied profiling.

We learned how to measure real workloads, read profiler output, distinguish CPU, I/O, memory, and waiting bottlenecks, use `cProfile`, `pstats`, `timeit`, and `tracemalloc`, and verify optimizations with before-and-after measurements.

Next we study design patterns.

Design patterns are named solutions to recurring design problems.

They are not recipes to force into every codebase.

They are vocabulary.

They help engineers discuss structure:

* factory
* strategy
* adapter
* facade
* repository
* observer
* command
* dependency injection
* context object
* unit of work

Profiling helps you understand runtime behavior.

Design patterns help you organize change.

The transition is:

```text
profiling improves how code performs
design patterns improve how code evolves
```

Chapter 80 will explain patterns in Pythonic terms, with care not to turn them into ceremony.
