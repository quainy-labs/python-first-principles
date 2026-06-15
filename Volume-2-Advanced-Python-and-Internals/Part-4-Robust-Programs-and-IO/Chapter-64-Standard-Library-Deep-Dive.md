# Chapter 64 — Standard Library Deep Dive

Python is not only a language.

It is also a large, practical toolkit.

That toolkit is the standard library.

The standard library is the collection of modules that come with Python itself. You do not install them from PyPI. You do not add them to `requirements.txt`. If Python is available, these modules are usually available too.

This matters because professional Python is not just knowing how to write a loop, define a class, raise an exception, or open a file.

Professional Python is knowing when the language already gives you the tool you need.

Beginners often solve everything from scratch:

```python
counts = {}
for word in words:
    if word not in counts:
        counts[word] = 0
    counts[word] += 1
```

That code is understandable.

But Python already has a tool for counting:

```python
from collections import Counter

counts = Counter(words)
```

The point is not that shorter code is always better.

The point is that standard-library tools encode common patterns clearly.

When you use `Counter`, you are not merely saving lines.

You are saying:

```text
this is a counting problem
```

When you use `deque`, you are saying:

```text
this needs efficient operations at both ends
```

When you use `itertools`, you are saying:

```text
this sequence can be processed lazily
```

When you use `argparse`, you are saying:

```text
this script has a command-line interface
```

When you use `logging`, you are saying:

```text
this program needs operational visibility
```

The standard library is a vocabulary of solved problems.

This chapter is not an encyclopedia. The official documentation is the encyclopedia.

Our job is different.

We will learn how to think with the standard library.

---

# What Belongs in the Standard Library?

The standard library contains modules for many categories:

* data structures
* iteration
* functional programming helpers
* mathematics
* statistics
* dates and times
* files and paths
* temporary files
* directory operations
* process execution
* command-line interfaces
* logging
* networking
* compression
* databases
* testing
* parsing
* introspection
* concurrency

You do not need to memorize it all.

That would be a poor use of your memory.

Instead, develop a habit:

```text
before inventing a general utility, ask whether the standard library has it
```

This habit saves time and improves design.

Standard-library modules tend to be:

* well tested
* widely understood
* documented
* portable across platforms where possible
* maintained with Python itself

They also have limits.

Some third-party libraries are better for specialized tasks:

* NumPy for numerical arrays
* Pandas or Polars for dataframes
* Requests or HTTPX for higher-level HTTP clients
* Pydantic for structured validation
* Rich for terminal UI
* Click or Typer for advanced command-line interfaces

But reach for the standard library first when the problem is common and the built-in tool is good enough.

---

# Importing Standard-Library Modules

You import standard-library modules the same way you import your own modules.

```python
import math
import json
from pathlib import Path
from collections import Counter
```

Use plain `import module` when the module name makes the call clearer:

```python
import statistics

average = statistics.mean(values)
```

Use `from module import name` when the imported name is the main abstraction:

```python
from pathlib import Path

path = Path("data") / "users.json"
```

Avoid wildcard imports:

```python
from math import *
```

Wildcard imports make it harder to see where names come from.

Prefer:

```python
import math
```

or:

```python
from math import sqrt, isclose
```

Import style is not only taste.

It affects readability.

Readers should be able to answer:

```text
where did this name come from?
```

without detective work.

---

# A Map of This Chapter

We will focus on modules that appear constantly in real Python work:

```text
collections  specialized container types
itertools    lazy iteration building blocks
functools    helpers for functions and callables
math         mathematical functions and constants
statistics   descriptive statistics
datetime     dates, times, durations, and time zones
pathlib      object-oriented paths
tempfile     temporary files and directories
shutil       high-level file operations
subprocess   running external programs
argparse     command-line interfaces
logging      application logs
```

This order is deliberate.

We begin with data containers, because they connect directly to Volume I's data structures.

Then we move to iteration and function tools, connecting to generators, decorators, and functional programming.

Then mathematics and time.

Then filesystem and process tools, building on Chapter 63.

Finally, command-line interfaces and logging, because they turn scripts into usable tools.

---

# collections

The `collections` module provides specialized container datatypes.

Python already has excellent built-in containers:

```text
list
tuple
dict
set
```

The `collections` module adds containers for common patterns that would otherwise require repetitive code.

Important tools include:

```text
Counter
defaultdict
deque
namedtuple
ChainMap
OrderedDict
UserDict
UserList
UserString
```

We will focus on the ones you will use most often.

---

# Counter

`Counter` counts hashable objects.

Basic example:

```python
from collections import Counter

words = ["red", "blue", "red", "green", "blue", "blue"]
counts = Counter(words)

print(counts)
```

Output:

```python
Counter({'blue': 3, 'red': 2, 'green': 1})
```

Without `Counter`, you might write:

```python
counts = {}

for word in words:
    counts[word] = counts.get(word, 0) + 1
```

That works.

But `Counter` says what the code means.

Useful methods:

```python
counts.most_common(2)
```

Output:

```python
[('blue', 3), ('red', 2)]
```

Accessing a missing key returns zero:

```python
counts["purple"]
```

Output:

```python
0
```

This is different from a normal dictionary, which raises `KeyError` for a missing key.

`Counter` is useful for:

* word frequencies
* inventory counts
* vote totals
* histogram-like data
* duplicate detection
* comparing multisets

Example duplicate detector:

```python
from collections import Counter

def find_duplicates(values):
    counts = Counter(values)
    return [value for value, count in counts.items() if count > 1]
```

Counters also support arithmetic:

```python
from collections import Counter

current = Counter(apples=10, oranges=4)
sold = Counter(apples=3, oranges=1)

remaining = current - sold
```

`remaining` is:

```python
Counter({'apples': 7, 'oranges': 3})
```

Use `Counter` when the core operation is tallying.

Do not use it merely because you need a dictionary.

---

# defaultdict

`defaultdict` is a dictionary that creates missing values using a factory function.

Example:

```python
from collections import defaultdict

groups = defaultdict(list)

for name, department in employees:
    groups[department].append(name)
```

Without `defaultdict`, you might write:

```python
groups = {}

for name, department in employees:
    if department not in groups:
        groups[department] = []
    groups[department].append(name)
```

`defaultdict(list)` means:

```text
when a missing key is accessed, create an empty list
```

Common factories:

```python
defaultdict(list)
defaultdict(set)
defaultdict(int)
```

Counting with `defaultdict(int)`:

```python
from collections import defaultdict

counts = defaultdict(int)

for word in words:
    counts[word] += 1
```

This works because `int()` returns `0`.

Grouping with `defaultdict(list)`:

```python
from collections import defaultdict

by_city = defaultdict(list)

for user in users:
    by_city[user.city].append(user)
```

Collecting unique values with `defaultdict(set)`:

```python
from collections import defaultdict

permissions_by_user = defaultdict(set)

for user_id, permission in assignments:
    permissions_by_user[user_id].add(permission)
```

Be careful: accessing a missing key creates it.

```python
groups["unknown"]
```

That line mutates the dictionary if `"unknown"` is missing.

If you only want to check whether a key exists, use:

```python
if "unknown" in groups:
    ...
```

Use `defaultdict` when missing keys should naturally create empty containers or zero values.

Use normal dictionaries when missing keys should be treated as mistakes.

---

# deque

`deque` stands for double-ended queue.

It supports efficient appends and pops from both ends.

```python
from collections import deque

queue = deque()
queue.append("first")
queue.append("second")

print(queue.popleft())
```

Output:

```text
first
```

Lists are efficient at appending and popping at the right end:

```python
items.append(value)
items.pop()
```

But removing from the left of a list is expensive:

```python
items.pop(0)
```

That operation shifts the remaining elements.

For queues, use `deque`:

```python
from collections import deque

def breadth_first(start):
    queue = deque([start])
    seen = {start}

    while queue:
        node = queue.popleft()
        yield node

        for neighbor in node.neighbors:
            if neighbor not in seen:
                seen.add(neighbor)
                queue.append(neighbor)
```

`deque` also supports `appendleft()` and `pop()`:

```python
d = deque([1, 2, 3])
d.appendleft(0)
d.pop()
```

You can limit its maximum length:

```python
recent = deque(maxlen=3)

for event in events:
    recent.append(event)
```

When a bounded deque is full, adding a new item discards one from the opposite end.

This is useful for:

* recent history
* fixed-size windows
* queues
* breadth-first traversal
* simple producer-consumer buffers

Use `deque` when both ends matter.

Use `list` when random indexing and right-end appends dominate.

---

# namedtuple

`namedtuple` creates tuple subclasses with named fields.

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])

p = Point(3, 4)

print(p.x)
print(p.y)
```

Output:

```text
3
4
```

Named tuples are immutable like tuples:

```python
p.x = 10
```

This raises `AttributeError`.

Before dataclasses, named tuples were a common way to create small record-like objects.

Today, for most new code, prefer `dataclass` when you want a domain object:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

Still, `namedtuple` remains useful when:

* you need tuple compatibility
* memory footprint matters
* you are working with older APIs
* the object is a lightweight immutable record

There is also `typing.NamedTuple`, which integrates better with type hints.

The decision is:

```text
need tuple behavior -> namedtuple
need regular class-like data model -> dataclass
```

---

# ChainMap

`ChainMap` presents multiple mappings as one view.

```python
from collections import ChainMap

defaults = {"color": "blue", "debug": False}
env = {"debug": True}
cli = {"color": "green"}

config = ChainMap(cli, env, defaults)

print(config["color"])
print(config["debug"])
```

Output:

```text
green
True
```

Lookups search from left to right.

Writes go to the first mapping.

This is useful for layered configuration:

```text
command-line arguments
environment variables
configuration file
defaults
```

Instead of merging dictionaries and losing where values came from, `ChainMap` keeps the layers visible.

Use it when you need a live view over multiple dictionaries.

Use dictionary unpacking or `|` when you want a new merged dictionary.

---

# UserDict, UserList, and UserString

Subclassing built-in containers directly can be subtle.

`UserDict`, `UserList`, and `UserString` wrap the underlying container and are often easier to customize.

Example:

```python
from collections import UserDict

class LowercaseKeys(UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key.lower(), value)

settings = LowercaseKeys()
settings["HOST"] = "localhost"

print(settings["host"])
```

These classes are useful when you want custom container behavior without fighting every implementation detail of `dict`, `list`, or `str`.

We will return to custom structures in Volume III when design and maintainability become the main focus.

---

# itertools

The `itertools` module provides functions for efficient looping.

Most tools in `itertools` are lazy.

That means they produce values as needed instead of building entire lists immediately.

This connects directly to iterators and generators from Part III.

Consider:

```python
numbers = range(1_000_000)
evens = [number for number in numbers if number % 2 == 0]
```

This builds a list.

Sometimes that is fine.

But if you only need to process values one at a time, laziness is better:

```python
evens = (number for number in numbers if number % 2 == 0)
```

`itertools` gives you building blocks for lazy pipelines.

Important tools include:

```text
count
cycle
repeat
chain
islice
takewhile
dropwhile
groupby
pairwise
batched
product
permutations
combinations
zip_longest
```

You do not need them all every day.

But when you recognize the pattern, they are elegant.

---

# count, cycle, and repeat

`count()` creates an infinite sequence of numbers:

```python
from itertools import count

for number in count(start=10, step=5):
    if number > 30:
        break
    print(number)
```

Output:

```text
10
15
20
25
30
```

`cycle()` repeats values forever:

```python
from itertools import cycle

colors = cycle(["red", "green", "blue"])

for _ in range(5):
    print(next(colors))
```

Output:

```text
red
green
blue
red
green
```

`repeat()` repeats one value:

```python
from itertools import repeat

for value in repeat("yes", times=3):
    print(value)
```

Output:

```text
yes
yes
yes
```

These tools can produce infinite iterators.

That is powerful, but it means you must limit them:

```python
from itertools import islice, count

first_ten = list(islice(count(), 10))
```

Laziness does not remove responsibility.

An infinite iterator in the wrong place can create an infinite loop.

---

# chain and islice

`chain()` joins iterables lazily:

```python
from itertools import chain

combined = chain([1, 2], [3, 4], [5])

print(list(combined))
```

Output:

```python
[1, 2, 3, 4, 5]
```

This avoids creating intermediate lists.

`islice()` slices an iterator:

```python
from itertools import islice

def first_n_lines(path, n):
    with path.open("r", encoding="utf-8") as f:
        return list(islice(f, n))
```

Unlike normal slicing, `islice()` works with any iterator.

This is useful because many streams cannot be indexed:

```python
f[0:10]
```

File objects do not support list-style slicing.

But `islice(f, 10)` works because it consumes the first ten items lazily.

---

# groupby

`groupby()` groups consecutive items with the same key.

This word matters:

```text
consecutive
```

Example:

```python
from itertools import groupby

items = ["a", "a", "b", "a"]

for key, group in groupby(items):
    print(key, list(group))
```

Output:

```text
a ['a', 'a']
b ['b']
a ['a']
```

The final `"a"` is a separate group because it is not adjacent to the first two.

If you want all equal keys grouped together, sort first:

```python
from itertools import groupby

rows = [
    {"department": "sales", "name": "Asha"},
    {"department": "engineering", "name": "Ben"},
    {"department": "sales", "name": "Chen"},
]

rows.sort(key=lambda row: row["department"])

for department, group in groupby(rows, key=lambda row: row["department"]):
    names = [row["name"] for row in group]
    print(department, names)
```

`groupby()` is not the same as SQL `GROUP BY`.

It groups runs in an iterable.

For accumulating groups without sorting, use `defaultdict(list)`.

---

# pairwise and batched

`pairwise()` produces overlapping pairs:

```python
from itertools import pairwise

for left, right in pairwise([10, 20, 30, 40]):
    print(left, right)
```

Output:

```text
10 20
20 30
30 40
```

This is useful for:

* comparing adjacent readings
* detecting changes
* computing deltas
* validating ordered data

Example:

```python
from itertools import pairwise

def is_strictly_increasing(values):
    return all(left < right for left, right in pairwise(values))
```

`batched()` groups items into fixed-size tuples:

```python
from itertools import batched

for batch in batched(range(10), 3):
    print(batch)
```

Output:

```text
(0, 1, 2)
(3, 4, 5)
(6, 7, 8)
(9,)
```

Batches are useful for:

* chunked API calls
* bulk database inserts
* paged processing
* limiting memory usage

If the final batch must be full, validate its length.

---

# product, permutations, and combinations

These functions generate combinatorial iterators.

Cartesian product:

```python
from itertools import product

for size, color in product(["S", "M", "L"], ["red", "blue"]):
    print(size, color)
```

This generates every pair.

Permutations:

```python
from itertools import permutations

list(permutations(["A", "B", "C"], 2))
```

Output:

```python
[('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]
```

Combinations:

```python
from itertools import combinations

list(combinations(["A", "B", "C"], 2))
```

Output:

```python
[('A', 'B'), ('A', 'C'), ('B', 'C')]
```

These tools can grow very quickly.

Before using them on large inputs, ask:

```text
how many results will this produce?
```

Laziness saves memory.

It does not make an enormous search space small.

---

# functools

The `functools` module provides helpers for working with callables.

You have already seen that functions are objects.

`functools` builds on that idea.

Important tools include:

```text
lru_cache
cache
cached_property
partial
wraps
singledispatch
reduce
total_ordering
```

We will focus on the tools that make code meaningfully clearer.

---

# lru_cache and cache

`lru_cache` memoizes function results.

Memoization stores the result of a function call so repeated calls with the same arguments can reuse the answer.

Example:

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

Without caching, recursive Fibonacci repeats enormous amounts of work.

With caching, each `n` is computed once.

Use caching when:

* the function is deterministic
* the arguments are hashable
* repeated calls are likely
* cached results will not become stale unexpectedly

Do not cache functions that depend on hidden changing state:

```python
@lru_cache
def read_config():
    return Path("config.json").read_text(encoding="utf-8")
```

This may keep returning old config after the file changes.

`cache` is a simpler unbounded cache:

```python
from functools import cache

@cache
def expensive_lookup(key):
    ...
```

Unbounded caches can grow forever.

Use them only when the set of inputs is naturally limited or the process lifetime is short.

Caching is not just an optimization.

It changes memory behavior.

---

# partial

`partial()` freezes some arguments of a function and returns a new callable.

```python
from functools import partial

def multiply(a, b):
    return a * b

double = partial(multiply, 2)

print(double(10))
```

Output:

```text
20
```

This is useful when an API expects a callable with a certain shape.

Example:

```python
from functools import partial

def format_price(currency, amount):
    return f"{currency} {amount:.2f}"

format_usd = partial(format_price, "USD")
```

Now:

```python
format_usd(19.99)
```

returns:

```text
USD 19.99
```

`partial` is often cleaner than a tiny wrapper function when the intent is simply binding arguments.

But do not use it when a named function would communicate domain meaning better.

---

# wraps

Decorators wrap functions.

Without care, the wrapper can hide the original function's metadata.

Bad decorator:

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

After decoration, the function's `__name__` becomes `"wrapper"`.

Use `functools.wraps`:

```python
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

`wraps` preserves useful metadata such as:

* `__name__`
* `__doc__`
* `__module__`
* annotations
* access to the wrapped function through `__wrapped__`

For professional decorators, use `wraps` unless you have a specific reason not to.

---

# singledispatch

`singledispatch` creates generic functions that dispatch based on the type of the first argument.

```python
from functools import singledispatch

@singledispatch
def describe(value):
    return f"object: {value!r}"

@describe.register
def _(value: int):
    return f"integer: {value}"

@describe.register
def _(value: list):
    return f"list with {len(value)} items"
```

Now:

```python
describe(10)
describe([1, 2, 3])
describe("hello")
```

returns different behavior based on the first argument's type.

Use `singledispatch` when:

* behavior varies by type
* the variants are naturally part of one operation
* adding new supported types should be clean

Do not use it to avoid ordinary polymorphism.

If the behavior belongs on the object itself, a method may be better.

`singledispatch` is most helpful at boundaries, such as serialization, formatting, conversion, or normalization.

---

# math

The `math` module provides mathematical functions and constants.

It works with real numbers, not complex numbers.

For complex numbers, Python has `cmath`.

Important groups include:

```text
number theory
rounding and floating-point helpers
powers and logarithms
trigonometry
distances and products
constants
```

Examples:

```python
import math

math.sqrt(81)
math.factorial(5)
math.gcd(24, 36)
math.lcm(6, 8)
math.pi
math.e
```

Use `math.sqrt()` when you want a floating-point square root.

Use `math.isqrt()` when you want an exact integer square root:

```python
import math

math.isqrt(10)
```

Output:

```text
3
```

This returns the floor of the exact square root.

For comparing floating-point values, do not rely on direct equality:

```python
0.1 + 0.2 == 0.3
```

This is often `False` because of binary floating-point representation.

Use `math.isclose()`:

```python
import math

math.isclose(0.1 + 0.2, 0.3)
```

Use `math.fsum()` for more accurate summation of floats:

```python
import math

total = math.fsum(values)
```

Use `math.prod()` for products:

```python
import math

math.prod([2, 3, 4])
```

Output:

```text
24
```

The practical lesson:

```text
use math when the operation has mathematical meaning and precision details matter
```

Do not reimplement factorials, greatest common divisors, combinations, or floating-point comparisons by hand.

---

# statistics

The `statistics` module provides descriptive statistics for numeric data.

Examples:

```python
import statistics

values = [10, 20, 20, 30, 40]

statistics.mean(values)
statistics.median(values)
statistics.mode(values)
statistics.stdev(values)
```

Common functions:

```text
mean
fmean
median
median_low
median_high
mode
multimode
variance
stdev
quantiles
correlation
linear_regression
```

Use `mean()` for arithmetic average:

```python
statistics.mean([1, 2, 3])
```

Use `median()` when outliers should not dominate:

```python
statistics.median([1, 2, 100])
```

Output:

```text
2
```

Use `fmean()` for a fast floating-point mean:

```python
statistics.fmean(values)
```

The `statistics` module is good for ordinary descriptive statistics.

It is not a replacement for NumPy, SciPy, Pandas, or Polars when you need large-scale numerical computing or dataframes.

But for scripts, reports, tests, and small datasets, it is exactly the right level of tool.

---

# datetime

Dates and times are famously tricky.

Python's `datetime` module provides core types:

```text
date
time
datetime
timedelta
timezone
tzinfo
```

Basic example:

```python
from datetime import date, datetime, timedelta, timezone

today = date.today()
now = datetime.now(timezone.utc)
tomorrow = today + timedelta(days=1)
```

Use `date` for calendar dates:

```python
from datetime import date

birthday = date(1995, 5, 17)
```

Use `datetime` for a date plus time:

```python
from datetime import datetime, timezone

created_at = datetime.now(timezone.utc)
```

Use `timedelta` for durations:

```python
from datetime import timedelta

timeout = timedelta(seconds=30)
```

The most important professional rule:

```text
be explicit about time zones
```

Naive datetimes do not contain timezone information:

```python
datetime.now()
```

Aware datetimes do:

```python
datetime.now(timezone.utc)
```

For internal timestamps, UTC is usually the best default.

For user-facing display, convert to the user's local timezone.

Python also includes `zoneinfo` for IANA time zones:

```python
from datetime import datetime
from zoneinfo import ZoneInfo

local = datetime.now(ZoneInfo("Asia/Kolkata"))
```

Dates and times become hard because of:

* time zones
* daylight saving changes
* leap years
* ambiguous local times
* serialization formats
* database conventions

Do not treat time as just a string.

Use proper types at the boundary.

Serialize timestamps deliberately, often with ISO 8601-style strings.

---

# pathlib

Chapter 63 introduced `pathlib`.

Here we place it inside the broader standard-library toolbox.

`pathlib.Path` represents filesystem paths as objects.

```python
from pathlib import Path

path = Path("data") / "users.json"

print(path.parent)
print(path.name)
print(path.suffix)
```

Useful methods:

```python
path.exists()
path.is_file()
path.is_dir()
path.read_text(encoding="utf-8")
path.write_text("hello\n", encoding="utf-8")
path.read_bytes()
path.write_bytes(b"data")
path.iterdir()
path.glob("*.json")
path.mkdir(parents=True, exist_ok=True)
```

Example:

```python
from pathlib import Path

def find_markdown_files(root):
    root = Path(root)
    return sorted(root.glob("**/*.md"))
```

`Path` improves code because it keeps path operations as path operations.

Compare:

```python
filename = directory + "/" + name + ".json"
```

with:

```python
path = directory / f"{name}.json"
```

The second form is clearer and more portable.

Use `Path` when manipulating paths.

Use file objects when reading and writing streams.

Use `shutil` when copying or moving larger filesystem trees.

---

# tempfile

The `tempfile` module creates temporary files and directories safely.

Do not invent temporary names manually:

```python
path = Path("temp.txt")
```

That can collide with existing files and can be unsafe in shared directories.

Use `TemporaryDirectory`:

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    path = Path(directory) / "work.txt"
    path.write_text("temporary data\n", encoding="utf-8")
    process(path)
```

When the block exits, the temporary directory is cleaned up.

Use `NamedTemporaryFile` when you need a file with a visible name:

```python
from tempfile import NamedTemporaryFile

with NamedTemporaryFile("w+", encoding="utf-8") as f:
    f.write("hello\n")
    f.seek(0)
    print(f.read())
```

Temporary files are useful for:

* tests
* safe write patterns
* intermediate data
* external tools that require a path
* generated exports before final replacement

Prefer high-level context managers.

Lower-level functions such as `mkstemp()` require manual cleanup.

Manual cleanup is easy to forget.

---

# shutil

`shutil` provides high-level file operations.

Where `pathlib` is about paths and simple file contents, `shutil` handles operations such as copying, moving, removing directory trees, and creating archives.

Examples:

```python
import shutil
from pathlib import Path

source = Path("reports/january.csv")
target = Path("backup/january.csv")

target.parent.mkdir(parents=True, exist_ok=True)
shutil.copy2(source, target)
```

Common functions:

```text
copyfile
copy
copy2
copytree
move
rmtree
make_archive
unpack_archive
disk_usage
which
```

`copyfile()` copies file contents.

`copy()` copies contents and permission mode.

`copy2()` tries to preserve more metadata.

`copytree()` copies directories recursively:

```python
import shutil

shutil.copytree("site", "site-backup")
```

`rmtree()` removes a directory tree:

```python
import shutil

shutil.rmtree("build")
```

Be careful with `rmtree()`.

It is destructive.

Validate paths before removing directory trees.

`shutil.which()` finds executables on the system path:

```python
import shutil

python_path = shutil.which("python")
```

This is useful before calling external tools.

---

# subprocess

The `subprocess` module runs external programs.

It is a boundary between Python and the operating system.

The most common entry point is `subprocess.run()`.

```python
import subprocess

result = subprocess.run(
    ["python", "--version"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

Pass arguments as a list:

```python
["python", "--version"]
```

Do not build shell strings from untrusted input:

```python
command = f"convert {user_file} output.png"
subprocess.run(command, shell=True)
```

That can create command injection vulnerabilities.

Prefer:

```python
subprocess.run(["convert", user_file, "output.png"], check=True)
```

Important parameters:

```text
check=True          raise CalledProcessError on non-zero exit
capture_output=True capture stdout and stderr
text=True           decode output to strings
timeout=seconds     limit runtime
cwd=path            run in a specific directory
env=mapping         control environment variables
```

Example with error handling:

```python
import subprocess

try:
    result = subprocess.run(
        ["git", "status", "--short"],
        capture_output=True,
        text=True,
        check=True,
        timeout=10,
    )
except subprocess.CalledProcessError as error:
    raise RuntimeError(f"command failed: {error.stderr}") from error
except subprocess.TimeoutExpired as error:
    raise RuntimeError("command timed out") from error
```

Use `subprocess` when Python must delegate to another executable.

Do not use it when a standard-library function already does the job directly.

For example, use `shutil.copy2()` instead of calling `cp`.

That makes code more portable.

---

# argparse

`argparse` builds command-line interfaces.

Instead of manually reading `sys.argv`, define the interface:

```python
import argparse

parser = argparse.ArgumentParser(
    prog="wordcount",
    description="Count lines, words, and characters in a text file.",
)

parser.add_argument("path", help="file to read")
parser.add_argument("--json", action="store_true", help="output JSON")

args = parser.parse_args()
```

If the user passes invalid arguments, `argparse` prints a helpful error and exits.

It also generates help text:

```text
wordcount --help
```

Argument types:

```python
parser.add_argument("--limit", type=int, default=10)
```

Choices:

```python
parser.add_argument("--format", choices=["text", "json"], default="text")
```

Boolean flags:

```python
parser.add_argument("--verbose", action="store_true")
```

Subcommands:

```python
parser = argparse.ArgumentParser(prog="notes")
subparsers = parser.add_subparsers(dest="command", required=True)

add_parser = subparsers.add_parser("add")
add_parser.add_argument("text")

list_parser = subparsers.add_parser("list")
list_parser.add_argument("--all", action="store_true")
```

Subcommands are useful when one tool has multiple operations:

```text
notes add "Buy milk"
notes list
notes done 3
```

Use `argparse` for scripts that other people, future you included, will run.

A good command-line interface is part of a program's design, not an afterthought.

---

# logging

`logging` records what a program is doing.

Do not use `print()` as your main observability tool in serious programs.

`print()` writes text.

`logging` records events with levels, names, timestamps, and destinations.

Basic example:

```python
import logging

logging.basicConfig(level=logging.INFO)

logger = logging.getLogger(__name__)

logger.info("application started")
logger.warning("configuration file missing; using defaults")
```

Common levels:

```text
DEBUG     detailed diagnostic information
INFO      normal operational events
WARNING   something unexpected but not fatal
ERROR     an operation failed
CRITICAL  the program may not be able to continue
```

Use module-level loggers:

```python
logger = logging.getLogger(__name__)
```

This gives logs names based on modules.

That allows applications to configure logging by component.

Use structured message arguments instead of f-strings when possible:

```python
logger.info("processed %s records", count)
```

This lets logging defer string formatting until the message is actually emitted.

Logging exceptions:

```python
try:
    process_file(path)
except OSError:
    logger.exception("failed to process file %s", path)
    raise
```

`logger.exception()` logs the traceback for the current exception.

Use it inside an `except` block.

Do not log and suppress errors unless suppression is intentional.

Bad:

```python
try:
    save_data()
except Exception:
    logger.exception("save failed")
```

This logs the error and then continues as if nothing happened.

If the caller needs to know, re-raise:

```python
try:
    save_data()
except Exception:
    logger.exception("save failed")
    raise
```

Logging should reveal behavior.

It should not hide failure.

---

# Combining Standard-Library Tools

The standard library becomes powerful when modules compose.

Example: a command-line tool that reads a JSON Lines file and prints the most common event names.

```python
import argparse
import json
from collections import Counter
from pathlib import Path

def iter_events(path):
    with path.open("r", encoding="utf-8") as f:
        for line_number, line in enumerate(f, start=1):
            line = line.strip()
            if not line:
                continue
            try:
                yield json.loads(line)
            except json.JSONDecodeError as error:
                raise ValueError(f"invalid JSON on line {line_number}") from error

def main(argv=None):
    parser = argparse.ArgumentParser()
    parser.add_argument("path")
    parser.add_argument("--limit", type=int, default=10)
    args = parser.parse_args(argv)

    path = Path(args.path)
    counts = Counter()

    for event in iter_events(path):
        counts[event.get("event", "<missing>")] += 1

    for name, count in counts.most_common(args.limit):
        print(f"{name}: {count}")
```

This uses:

* `argparse` for the interface
* `Path` for file paths
* `json` for parsing
* `Counter` for tallying
* exceptions for clear failure
* iteration for streaming

No third-party package is needed.

The code is not clever.

It is composed.

That is the standard library's real power.

---

# A More Operational Example

Now add logging and subprocess.

Suppose a tool runs an external formatter on generated files.

```python
import argparse
import logging
import shutil
import subprocess
from pathlib import Path

logger = logging.getLogger(__name__)

def require_tool(name):
    path = shutil.which(name)
    if path is None:
        raise RuntimeError(f"required executable not found: {name}")
    return path

def run_formatter(formatter, path):
    logger.info("formatting %s", path)

    try:
        subprocess.run(
            [formatter, str(path)],
            check=True,
            capture_output=True,
            text=True,
            timeout=30,
        )
    except subprocess.CalledProcessError as error:
        logger.error("formatter stderr: %s", error.stderr)
        raise RuntimeError(f"formatter failed for {path}") from error

def main(argv=None):
    parser = argparse.ArgumentParser()
    parser.add_argument("paths", nargs="+")
    parser.add_argument("--verbose", action="store_true")
    args = parser.parse_args(argv)

    logging.basicConfig(
        level=logging.DEBUG if args.verbose else logging.INFO
    )

    formatter = require_tool("black")

    for raw_path in args.paths:
        run_formatter(formatter, Path(raw_path))
```

This example has boundaries:

* command-line input
* filesystem paths
* external executable discovery
* subprocess execution
* timeout handling
* logging
* exception translation

Standard-library modules help each boundary stay explicit.

---

# Standard Library Versus Third-Party Packages

Use the standard library when:

* the problem is common
* the built-in tool is sufficient
* portability matters
* avoiding dependencies is valuable
* the task is part of application plumbing

Use a third-party package when:

* the standard-library tool is too low-level
* the domain requires specialized algorithms
* productivity gain is large
* community conventions strongly favor a package
* correctness would be risky to implement yourself

Examples:

```text
use pathlib before writing path string utilities
use json before adding a JSON package
use argparse for ordinary CLIs
consider Click or Typer for large polished CLIs
use urllib for low-level HTTP
consider Requests or HTTPX for application HTTP clients
use statistics for small descriptive stats
use NumPy/Pandas/Polars for serious data analysis
```

Dependencies are not bad.

Unnecessary dependencies are expensive.

Avoiding a dependency is not automatically virtuous either.

The professional question is:

```text
which tool gives the best reliability, clarity, and maintenance cost for this problem?
```

---

# Common Mistakes

Do not reimplement common containers:

```python
counts = {}
for item in items:
    counts[item] = counts.get(item, 0) + 1
```

Use `Counter` when counting is the point.

Do not parse command-line arguments manually for real tools:

```python
mode = sys.argv[1]
```

Use `argparse`.

Do not use `print()` as a substitute for application logging.

Use `logging` for events that operators or developers need to understand.

Do not use shell commands for work Python can do portably:

```python
subprocess.run(["cp", source, target])
```

Use `shutil.copy2()`.

Do not use `shell=True` with untrusted input.

Pass subprocess arguments as a list.

Do not build large intermediate lists when lazy iteration is enough.

Use iterators, generators, and `itertools`.

Do not cache functions whose results depend on hidden changing state.

Use `lru_cache` only when cached answers remain valid.

Do not use naive datetimes for important timestamps.

Use timezone-aware datetimes when time crosses system or user boundaries.

Do not manually create temporary filenames.

Use `tempfile`.

---

# How to Learn the Standard Library

You do not learn the standard library by reading every page once.

You learn it by building a mental index.

When you face a task, ask:

```text
Is this about counting?
collections.Counter

Is this about grouping?
defaultdict or itertools.groupby

Is this about queues?
collections.deque

Is this about lazy sequence transformations?
itertools

Is this about function wrappers or caching?
functools

Is this about paths?
pathlib

Is this about copying directory trees?
shutil

Is this about temporary files?
tempfile

Is this about running external tools?
subprocess

Is this about command-line arguments?
argparse

Is this about operational events?
logging
```

This mental index is more valuable than memorizing signatures.

Once you know the module likely exists, the documentation becomes useful.

You know where to look.

---

# Exercises

1. Use `Counter` to find the five most common words in a text file.

2. Use `defaultdict(list)` to group users by department.

3. Use `deque` to keep only the last ten events from a stream.

4. Use `itertools.islice()` to print the first twenty lines of a large file without reading the whole file.

5. Use `itertools.groupby()` to group sorted rows by a category field.

6. Use `functools.lru_cache()` to speed up a recursive function, then inspect its cache statistics.

7. Write a decorator using `functools.wraps()`, then compare the decorated function's `__name__` with and without `wraps`.

8. Use `math.isclose()` to compare floating-point calculations.

9. Use `statistics.mean()` and `statistics.median()` on a dataset with outliers and compare the results.

10. Write a script with `argparse` that accepts a file path and an optional `--limit`.

11. Use `logging` instead of `print()` in a file-processing script.

12. Use `TemporaryDirectory` in a test-like script that creates files and automatically cleans them up.

13. Use `shutil.copytree()` to copy a directory, then use `shutil.rmtree()` to remove the copy carefully.

14. Use `subprocess.run()` with `check=True`, `capture_output=True`, `text=True`, and `timeout`.

15. Combine `argparse`, `Path`, `Counter`, and `logging` into a small command-line word counter.

---

# Summary

The standard library is Python's built-in toolkit.

It contains production-ready modules for common programming problems.

`collections` provides specialized containers such as `Counter`, `defaultdict`, `deque`, and `ChainMap`.

`itertools` provides lazy iteration tools for efficient looping and sequence processing.

`functools` provides helpers for callables, including caching, partial application, decorator metadata, and generic functions.

`math` provides real-number mathematical functions and constants.

`statistics` provides descriptive statistics for ordinary datasets.

`datetime` provides types for dates, times, durations, and time zones.

`pathlib` represents filesystem paths as objects.

`tempfile` creates temporary files and directories safely.

`shutil` performs high-level file and directory operations.

`subprocess` runs external programs and must be used carefully at system boundaries.

`argparse` turns scripts into command-line tools with clear interfaces.

`logging` records operational events with levels, names, and configurable output.

The standard library is not something to memorize.

It is something to recognize.

The deeper lesson is:

```text
write less infrastructure and more intention
```

When a standard-library module matches the problem, use it.

Your code becomes shorter, clearer, more conventional, and easier for other Python programmers to understand.

---

# Preview of Chapter 65

Chapter 64 studied practical leverage from the standard library.

We saw how Python's built-in modules solve common problems around data, iteration, functions, mathematics, files, processes, command-line interfaces, and logging.

Next we move into concurrency foundations.

Concurrency is about dealing with multiple tasks whose lifetimes overlap.

That does not always mean running many things at the exact same CPU instant.

It can mean:

* waiting for files
* waiting for network responses
* handling many requests
* coordinating background work
* overlapping I/O-bound operations
* separating responsiveness from slow tasks

Chapter 65 will introduce the core ideas:

* concurrency versus parallelism
* CPU-bound work versus I/O-bound work
* processes
* threads
* tasks
* scheduling
* blocking
* race conditions
* shared state
* synchronization
* when concurrency helps
* when concurrency makes programs worse

The transition is:

```text
the standard library gives tools
concurrency teaches when many operations overlap
```

Concurrency is powerful, but it charges interest.

Before learning APIs, we need the mental model.
