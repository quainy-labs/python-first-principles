# Chapter 35 — Garbage Collection

---

# Learning Objectives

By the end of this chapter, you should understand:

* Why reference counting alone is not enough.
* What a reference cycle is.
* What it means for an object to be reachable.
* Why unreachable objects may still have nonzero reference counts.
* What cyclic garbage collection does in CPython.
* Why garbage collection is mostly about containers and object graphs.
* Why simple atomic objects usually do not need cycle tracking.
* How garbage collection relates to memory leaks.
* How Python's `gc` tool can help you inspect collector behavior.
* Why garbage collection timing should not be used for program logic.
* Why context managers are better for deterministic resource cleanup.
* How garbage collection connects to object lifecycle and weak references.

The previous chapter explained reference counting.

Reference counting answers this question:

```text
How many references point to this object?
```

Garbage collection answers a different question:

```text
Can this object still be reached from a live part of the program?
```

That difference matters.

An object can have references and still be useless.

That is the whole reason cyclic garbage collection exists.

---

# Concept Overview

Garbage collection is the process of finding objects that a program can no longer use and making their memory available for reuse.

In CPython, memory cleanup has two cooperating mechanisms:

```text
reference counting
cyclic garbage collection
```

Reference counting handles many objects immediately.

Example:

```python
items = [1, 2, 3]
items = None
```

After `items` is rebound to `None`, the list may have no remaining references.

In CPython, that list can usually be cleaned up immediately.

But reference counting has a weakness.

It cannot solve cycles by itself.

Example:

```python
a = []
b = []

a.append(b)
b.append(a)
```

Conceptually:

```text
a ──▶ list A ──▶ list B
        ▲         │
        │         ▼
        └─────────┘

b ───────────────▶ list B
```

Now remove the names:

```python
del a
del b
```

The names are gone.

But the two lists still refer to each other.

Conceptually:

```text
list A ──▶ list B
  ▲         │
  │         ▼
  └─────────┘
```

No program variable can reach them.

But their reference counts are not zero because each list is still referenced by the other list.

This is cyclic garbage.

Cyclic garbage is memory that is unreachable from the program but kept alive by internal references inside a group of objects.

Garbage collection exists to find and clean up such groups.

---

# The Problem Reference Counting Cannot Solve

Reference counting is simple and powerful.

It works beautifully for objects whose references eventually disappear one by one.

Example:

```python
def build_list():
    result = [10, 20, 30]
    return result

numbers = build_list()
numbers = None
```

The list is created.

It is returned.

The caller keeps it.

Later the caller releases it.

That is straightforward.

The references form a chain that eventually ends.

But cycles do not form a simple chain.

They form loops.

Example:

```python
node = {}
node["self"] = node
```

This dictionary refers to itself.

Conceptually:

```text
node ──▶ dict
          │
          ▼
        "self"
          │
          ▼
         dict
```

The dictionary contains a reference to itself.

Now:

```python
del node
```

The name is gone.

But the dictionary still points to itself.

Its reference count is not zero.

Reference counting alone sees:

```text
dict has a reference
```

But the program reality is:

```text
no live name can reach this dict
```

Those are different facts.

Reference counting measures references.

Garbage collection measures reachability.

---

# Reachability

Reachability is one of the most important ideas in memory management.

An object is reachable if the running program can still get to it through some path of references.

A path can start from:

* Local variables in currently running functions.
* Global variables in modules.
* Class attributes.
* Objects stored in live containers.
* Closures.
* Running stack frames.
* Interpreter-level roots.

Suppose:

```python
data = {"numbers": [1, 2, 3]}
```

Conceptually:

```text
data ──▶ dict ──▶ list ──▶ integers
```

The list is reachable because the program can follow:

```text
data -> dict value -> list
```

Now:

```python
data = None
```

The old dictionary may no longer be reachable.

If no other object points to it, CPython's reference counting can clean it up.

When the dictionary is cleaned up, it releases its reference to the list.

Then the list may also be cleaned up.

This is normal reference counting.

Cycles require a deeper check.

The question becomes:

```text
Is this group reachable from outside the group?
```

If a group only refers to itself, it can be garbage.

---

# Object Graphs

Python objects form graphs.

A graph is a collection of nodes and connections.

In memory-management discussions:

```text
node       -> object
connection -> reference
```

Example:

```python
profile = {
    "name": "Ada",
    "skills": ["Python", "Math"],
}
```

Object graph:

```text
profile ──▶ dict
             │
             ├──▶ "Ada"
             │
             └──▶ list
                   │
                   ├──▶ "Python"
                   └──▶ "Math"
```

This graph has no cycle.

Now:

```python
profile["self"] = profile
```

Graph:

```text
profile ──▶ dict
             │
             ├──▶ "Ada"
             ├──▶ list
             └──▶ dict itself
```

This graph has a cycle.

The dictionary can be reached from itself.

Cycles are not automatically bad.

Live programs sometimes intentionally use cycles.

For example:

```python
parent.children.append(child)
child.parent = parent
```

This creates a two-way relationship.

That may be useful.

The problem is not that a cycle exists.

The problem is when the cycle becomes unreachable from the rest of the program.

---

# A Simple Cycle

Consider this example:

```python
def make_cycle():
    first = []
    second = []

    first.append(second)
    second.append(first)

make_cycle()
```

Inside the function:

```text
first  ──▶ list A ──▶ list B
second ──▶ list B ──▶ list A
```

While the function is running, both lists are reachable from local variables.

When the function returns, the local variables disappear.

Now the graph is:

```text
list A ──▶ list B
  ▲         │
  │         ▼
  └─────────┘
```

No live program name points to either list.

But each list points to the other.

Their reference counts remain nonzero.

Reference counting cannot clean them immediately.

Cyclic garbage collection can detect that this closed group is unreachable.

Then it can collect it.

---

# The Word Garbage

In programming, garbage does not mean worthless in a moral or design sense.

It means:

```text
the program can no longer use this object
```

An object can contain important-looking data and still be garbage.

Example:

```python
def load_user():
    user = {"id": 1, "name": "Ada"}
    return None
```

The dictionary has meaningful information.

But if nothing can reach it after the function returns, it is garbage.

Garbage is about reachability, not value.

Likewise, an object can be large and still not be garbage if the program can reach it.

Example:

```python
cache = {}
cache["huge_result"] = [0] * 1_000_000
```

That list is reachable through `cache`.

It is not garbage.

It may be a memory problem.

But it is not garbage from Python's perspective.

This distinction is essential:

```text
unreachable object -> garbage
reachable but unwanted object -> program-level memory leak
```

The garbage collector cannot know your intention.

It only knows reachability.

---

# Garbage Collection Is Not Magic

Garbage collection does not mean Python understands which objects are semantically useful.

It does not ask:

```text
Will this user need the data later?
Is this cache still valuable?
Is this object logically obsolete?
```

It asks:

```text
Can this object be reached from live roots?
```

If yes, it must keep the object.

If no, it may collect the object.

This is why memory leaks can still happen in Python.

Example:

```python
events = []

def record_event(event):
    events.append(event)
```

If `events` grows forever, Python will not collect its contents.

They are reachable.

The program still holds references to them.

From Python's perspective, this is not garbage.

From your application's perspective, it may be a bug.

Garbage collection solves unreachable memory.

It does not solve bad ownership decisions.

---

# Containers and Cycles

Cycles require objects that can refer to other objects.

The most common cycle-capable objects are containers and user-defined objects.

Examples:

* `list`
* `dict`
* `set`
* `tuple` when it contains references to cycle-capable objects
* class instances
* functions and closures
* frames

Simple atomic objects usually do not create cycles by themselves.

Examples:

* integers
* floats
* booleans
* `None`
* many strings

An integer does not point to a list.

A boolean does not contain a dictionary.

These objects are usually leaves in the object graph.

They can be referenced by containers, but they do not usually create reference paths outward.

That is why cyclic garbage collection mostly concerns objects that can contain references.

---

# Atomic Objects

An atomic object is an object that does not contain references to other Python objects in a way relevant to cycle detection.

For practical beginner-level reasoning:

```text
atomic object -> does not act like a container of other Python objects
```

Examples:

```python
x = 10
y = True
z = None
name = "Ada"
```

These objects do not create object cycles by themselves.

You can store them in containers:

```python
values = [10, True, None, "Ada"]
```

The list points to those objects.

But the integer `10` does not point back to the list.

Conceptually:

```text
list ──▶ int
list ──▶ bool
list ──▶ None
list ──▶ str
```

There is no loop.

For this reason, garbage collectors often treat simple atomic objects differently from containers.

CPython's cyclic collector focuses on objects that can participate in cycles.

---

# Tracked and Untracked Objects

CPython's cyclic garbage collector does not need to track every object in the same way.

An object that cannot participate in a cycle usually does not need cycle tracking.

For example, the collector does not need to spend much effort checking whether an integer points back to a list.

It cannot.

Some containers are tracked because they can hold references.

Example:

```python
items = []
```

A list can contain another list.

So it can participate in a cycle.

Example:

```python
items.append(items)
```

Now `items` refers to itself.

Some objects may be optimized depending on what they contain.

For example, a dictionary containing only simple atomic keys and values may be treated differently from a dictionary containing containers.

The exact rules are implementation details.

The important idea is:

```text
cycle-capable objects are the main concern of cyclic garbage collection
```

You do not need to memorize every tracking rule.

You need to understand why tracking exists.

---

# The `gc` Tool

Python provides a standard tool named `gc`.

It exposes some garbage collector controls and inspection helpers.

We have not deeply studied modules yet, so read this lightly:

```python
import gc
```

This line gives your program access to Python's garbage-collector interface.

You can ask whether an object is tracked:

```python
import gc

print(gc.is_tracked([]))
print(gc.is_tracked(10))
```

On CPython, a list is typically tracked because it can participate in cycles.

An integer is typically not tracked because it does not contain other Python objects.

The exact output can depend on implementation and version.

Use `gc` for learning and debugging.

Do not build normal application logic around its low-level details.

---

# Manual Collection

The `gc` tool can request a collection:

```python
import gc

collected = gc.collect()
print(collected)
```

This asks the garbage collector to run.

The returned number tells you how many unreachable objects were collected, along with objects found uncollectable depending on the version and context.

This is useful for experiments.

Example:

```python
import gc

def make_cycle():
    a = []
    b = []
    a.append(b)
    b.append(a)

make_cycle()

print(gc.collect())
```

The exact number may vary.

Why?

Because the interpreter may have other tracked objects.

Because garbage collection may already have run.

Because implementation details differ.

Because your environment may keep references you do not see.

Do not focus on the exact number.

Focus on the model:

```text
cycle created
local names disappear
objects are unreachable
collector can reclaim them
```

---

# Automatic Collection

CPython can run cyclic garbage collection automatically.

You do not usually call `gc.collect()` yourself.

The interpreter keeps internal counts related to object allocation and deallocation.

When the conditions are right, it runs a collection.

The exact scheduling is an implementation detail.

It can change across Python versions.

It can also differ across Python implementations.

This matters because you should not write code that depends on garbage collection happening at a precise moment.

Bad assumption:

```text
This object will be collected immediately after this line.
```

Better assumption:

```text
If the object is unreachable, Python may collect it when appropriate.
```

Reference counting in CPython often makes cleanup look immediate.

But cyclic garbage collection is not a clock.

It is a memory-management mechanism.

---

# Generational Collection

Many garbage collectors use a generational strategy.

The broad idea is:

```text
new objects are checked more often
older surviving objects are checked less often
```

Why?

Because many objects in programs are short-lived.

Example:

```python
for line in lines:
    parts = line.split(",")
    process(parts)
```

The `parts` list may be created and discarded quickly for each line.

Short-lived temporary objects are common.

Long-lived objects are also common:

```python
settings = load_settings()
routes = build_routes()
```

If an object has survived several checks, it is less likely to become garbage immediately.

So the collector can often save work by checking newer objects more frequently.

CPython exposes generation-related information through `gc`.

However, the exact generation behavior is version-sensitive.

The book-level model is enough for now:

```text
young objects are candidates for frequent checks
long-lived objects are checked less frequently
```

Treat tuning collector generations as an advanced topic.

---

# Why Cycles Are Common

Reference cycles appear naturally in real programs.

They are not rare mistakes.

They often come from two-way relationships.

Example:

```python
class Parent:
    def __init__(self):
        self.children = []


class Child:
    def __init__(self, parent):
        self.parent = parent


parent = Parent()
child = Child(parent)
parent.children.append(child)
```

Graph:

```text
parent ──▶ Parent instance ──▶ children list ──▶ Child instance
   ▲                                           │
   │                                           ▼
   └──────────────────── parent attribute ◀────┘
```

This cycle is not automatically wrong.

It models reality:

```text
parent knows child
child knows parent
```

When `parent` and `child` are no longer reachable from the program, the cycle should be collectable.

Cyclic garbage collection makes this possible.

---

# Self-Referential Objects

An object can refer to itself.

Example:

```python
items = []
items.append(items)
```

This creates:

```text
items ──▶ list
          │
          ▼
         itself
```

Try printing it:

```python
print(items)
```

Python displays something like:

```text
[[...]]
```

The `...` means Python detected recursive containment while building the representation.

If Python tried to print the list by expanding it forever, it would never stop:

```text
[
  [
    [
      [
        ...
      ]
    ]
  ]
]
```

This is a visible sign of a cycle.

Now:

```python
del items
```

The list still refers to itself.

Reference counting alone cannot reduce its count to zero.

The cyclic collector can identify that the self-contained group is unreachable.

---

# Cycles in Dictionaries

Dictionaries can also refer to themselves.

Example:

```python
data = {}
data["self"] = data
```

Graph:

```text
data ──▶ dict
          │
          ▼
        "self" ──▶ same dict
```

Printing:

```python
print(data)
```

You may see:

```text
{'self': {...}}
```

The `{...}` means recursive dictionary content.

This is not a syntax error.

It is a real object graph.

The dictionary contains itself as a value.

When reachable, it is live.

When unreachable, it can become cyclic garbage.

---

# Cycles in Objects

Class instances often create cycles because attributes can point anywhere.

Example:

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None


a = Node("a")
b = Node("b")

a.next = b
b.next = a
```

Graph:

```text
a ──▶ Node("a") ──next──▶ Node("b")
       ▲                  │
       │                  ▼
       └──────next────────┘

b ──▶ Node("b")
```

Now:

```python
del a
del b
```

The two nodes still reference each other.

If no other object can reach them, they are garbage.

This kind of structure appears in:

* Linked structures.
* Trees with parent pointers.
* Graphs.
* Observer systems.
* Caches.
* Callback systems.

The more expressive your object relationships become, the more important garbage collection becomes.

---

# What the Collector Looks For

At a high level, the cyclic collector looks for groups of tracked objects that are unreachable from outside the group.

Imagine:

```text
live root ──▶ object A ──▶ object B

object X ──▶ object Y
   ▲          │
   │          ▼
   └──────────┘
```

`object A` and `object B` are reachable.

They must stay alive.

`object X` and `object Y` refer to each other.

But nothing live points to `object X` or `object Y`.

They can be collected.

The collector must distinguish:

```text
cycle reachable from live program -> keep it
cycle unreachable from live program -> collect it
```

This is why a cycle itself is not enough.

Reachability decides.

---

# Why The Collector Cannot Just Delete Every Cycle

Suppose:

```python
class Person:
    def __init__(self, name):
        self.name = name
        self.friends = []


ada = Person("Ada")
grace = Person("Grace")

ada.friends.append(grace)
grace.friends.append(ada)
```

There is a cycle:

```text
ada ──▶ Ada ──friends──▶ Grace
       ▲                 │
       │                 ▼
       └────friends──────┘
```

But `ada` and `grace` are live names.

The program can still use the objects:

```python
print(ada.friends[0].name)
```

Deleting the cycle would destroy valid program data.

So the garbage collector must not delete cycles merely because they are cycles.

It must delete unreachable cycles.

This distinction is the heart of garbage collection.

---

# A Reachable Cycle

Consider:

```python
team = []
team.append(team)
```

This is a self-referential list.

It is a cycle.

But it is reachable through the name `team`.

So it is not garbage.

You can still use it:

```python
print(len(team))
print(team[0] is team)
```

The output:

```text
1
True
```

The object is unusual, but live.

Now:

```python
team = None
```

If no other references exist, the self-referential list becomes unreachable.

Then it becomes collectable cyclic garbage.

Same cycle.

Different reachability.

---

# Garbage Collection and Program Roots

Garbage collection starts from things the program can definitely reach.

These are often called roots.

A root is a starting point for reachability.

Examples:

* Local variables in active function calls.
* Global variables.
* Module dictionaries.
* Currently executing frames.
* Some interpreter-managed references.

From these roots, the collector can conceptually trace outward.

Example:

```python
global_data = {"users": []}
```

Root:

```text
global_data
```

Reachable graph:

```text
global_data ──▶ dict ──▶ list
```

If an object cannot be reached by following references from roots, it is unreachable.

Unreachable tracked cycles are candidates for collection.

---

# Stack Frames and Reachability

Function calls create stack frames.

Local variables in active frames keep objects reachable.

Example:

```python
def pause_with_cycle():
    a = []
    a.append(a)
    input("cycle exists, but function is still running")

pause_with_cycle()
```

While the function is waiting at `input`, the local variable `a` exists.

The self-referential list is reachable.

It is not garbage.

When the function returns, the frame disappears.

Then `a` disappears.

If no other references exist, the list becomes unreachable.

The collector may then reclaim it.

The cycle did not become garbage when it was created.

It became garbage when it became unreachable.

---

# Closures and Reachability

Closures can keep objects reachable after a function returns.

Example:

```python
def make_reader():
    data = []

    def read():
        return data

    data.append(read)
    return read


reader = make_reader()
```

Here:

* `read` closes over `data`.
* `data` contains `read`.
* `reader` refers to `read`.

Conceptually:

```text
reader ──▶ function read ──▶ closure cell ──▶ data list
              ▲                              │
              │                              ▼
              └──────────── list item ◀──────┘
```

There is a cycle.

But the cycle is reachable through `reader`.

So it is not garbage.

If later:

```python
reader = None
```

Then the cycle may become unreachable.

At that point, cyclic garbage collection can handle it.

Closures are powerful.

They also make object graphs less obvious.

---

# Exceptions and Frames

Exception tracebacks can keep stack frames alive.

Stack frames contain local variables.

Those local variables can refer to large objects.

Example:

```python
saved_error = None

def fail():
    huge = [0] * 1_000_000
    raise RuntimeError("problem")

try:
    fail()
except RuntimeError as error:
    saved_error = error
```

Depending on what is retained, exception objects and tracebacks can keep references to frames.

Frames can keep references to local variables.

That means `huge` may remain reachable longer than expected.

Modern Python has improved many exception cleanup behaviors, but the concept remains important:

```text
debugging information can keep objects alive
```

If you store exception objects, tracebacks, frames, or inspection results, you may also store references to much more memory than you realize.

This is not always garbage.

It may be reachable through your saved debugging object.

---

# Debuggers and Interactive Sessions

Interactive environments can keep extra references.

Examples:

* A REPL may keep the last result.
* A notebook may keep variables in cells.
* A debugger may keep frame objects.
* A test runner may retain failures.
* An IDE may inspect objects.

This can confuse memory experiments.

Example:

```python
data = [0] * 1_000_000
data = None
```

You may expect the list to disappear.

But an interactive environment might still hold a reference through history or inspection tools.

The object is not garbage if something can still reach it.

When learning garbage collection, prefer small scripts when you want clean experiments.

Interactive environments are wonderful for exploration, but they add invisible roots.

---

# Garbage Collection and `del`

The `del` statement removes a reference.

It does not directly destroy an object in the abstract Python language model.

Example:

```python
a = []
b = a

del a
```

The list remains reachable through `b`.

Now:

```python
del b
```

If no other references exist, the list may be cleaned up.

With cycles:

```python
a = []
a.append(a)

del a
```

The name is removed.

The list still refers to itself.

Reference counting alone cannot finish the job.

The cyclic collector may collect it later.

So:

```text
del removes a reference
garbage collection finds unreachable object groups
```

They are related but not the same.

---

# Garbage Collection and `None`

Rebinding a name to `None` is another way to release a reference.

Example:

```python
data = [1, 2, 3]
data = None
```

This does not erase the list directly.

It makes the name `data` refer to `None`.

The previous list loses one reference.

If that was the last useful path to the list, the list may become collectable.

For cycles:

```python
data = []
data.append(data)
data = None
```

The self-referential list no longer has the name `data`.

But it still refers to itself.

The collector is needed to identify it as unreachable.

You will often see code use:

```python
thing = None
```

to release a reference early.

This can help in long-running functions when an object is large and no longer needed.

But it is not a replacement for clear ownership.

---

# Memory Leaks in Python

Python has automatic memory management.

That does not mean Python programs cannot leak memory.

A memory leak in Python usually means:

```text
objects remain reachable even though the program no longer logically needs them
```

Example:

```python
users_seen = []

def handle_request(user):
    users_seen.append(user)
```

If this server runs for months, `users_seen` may grow forever.

The garbage collector will not collect those users.

They are reachable through `users_seen`.

Another example:

```python
cache = {}

def compute(key):
    if key not in cache:
        cache[key] = expensive_result(key)
    return cache[key]
```

This may be intentional.

But if keys are unbounded, memory may grow without limit.

Again, not garbage.

Reachable memory.

The collector cannot know which cached values are no longer useful.

You must design that policy.

---

# Cyclic Garbage vs Logical Leaks

Compare these two cases.

Case one:

```python
def make_unused_cycle():
    a = []
    b = []
    a.append(b)
    b.append(a)

make_unused_cycle()
```

The cycle is unreachable after the function returns.

The garbage collector can collect it.

Case two:

```python
registry = []

def register():
    a = []
    b = []
    a.append(b)
    b.append(a)
    registry.append(a)

register()
```

Now the cycle is reachable through `registry`.

The collector must keep it.

If `registry` grows forever, that is a program design issue.

The collector is doing the correct thing.

The distinction:

```text
unreachable cycle -> garbage collector problem
reachable unwanted data -> application ownership problem
```

---

# Resource Cleanup Is Different

Memory is not the only resource programs use.

Programs also use:

* Files.
* Sockets.
* Database connections.
* Locks.
* Temporary directories.
* Subprocess handles.

Garbage collection is not the right way to manage these resources.

Bad pattern:

```python
file = open("data.txt")
content = file.read()
file = None
```

This relies on cleanup happening indirectly.

Better pattern:

```python
with open("data.txt") as file:
    content = file.read()
```

The `with` statement gives deterministic cleanup.

The file is closed when the block exits.

This is true even if an exception happens inside the block.

Core rule:

```text
Use garbage collection for memory.
Use context managers for external resources.
```

We will study context managers in depth later.

For now, remember that garbage collection timing is not a resource-management API.

---

# Finalizers and Cleanup Timing

Some objects define special cleanup behavior that may run when the object is finalized.

You may see this method:

```python
def __del__(self):
    ...
```

We will study object lifecycle and special methods later.

For now, treat `__del__` carefully.

It is not a substitute for `with`.

Problems with finalizers include:

* The timing may be surprising.
* Cycles involving finalizers are more subtle.
* Objects may be partially torn down during interpreter shutdown.
* Exceptions inside finalizers are hard to handle cleanly.

Modern Python is much better at handling finalizers in cycles than older versions were.

Still, finalizers are advanced tools.

For beginner and professional code, prefer explicit cleanup:

```python
with resource:
    use(resource)
```

or:

```python
try:
    use(resource)
finally:
    resource.close()
```

Garbage collection should not be the primary cleanup plan for important external resources.

---

# A Mental Model for Collection

Here is a simplified mental model of cyclic garbage collection:

```text
1. Track objects that can participate in cycles.
2. Identify references among tracked objects.
3. Ask which tracked objects are reachable from live roots.
4. Find groups that only refer to themselves.
5. Clear unreachable groups safely.
```

This is not the exact implementation.

But it is the right conceptual model for writing Python.

The key question is:

```text
Does anything live outside this group point into it?
```

If yes, the group is reachable.

If no, the group can be garbage.

Example:

```text
live root ──▶ A ──▶ B ──▶ C

X ──▶ Y
▲     │
│     ▼
└─────┘
```

`A`, `B`, and `C` are reachable.

`X` and `Y` form an unreachable group.

The collector can reclaim `X` and `Y`.

---

# Why Collection Has Cost

Garbage collection is useful, but it is not free.

The collector must inspect objects.

It must reason about references.

It must decide which objects are reachable.

That takes time.

For most Python programs, the default behavior is good enough.

But in performance-sensitive systems, garbage collection can matter.

Examples:

* Low-latency services.
* Games.
* Real-time-ish data pipelines.
* Large graph-processing programs.
* Systems that create many temporary containers.

The goal is not to fear garbage collection.

The goal is to understand that allocation patterns have runtime consequences.

If a program constantly creates many cyclic containers, the collector has more work to do.

If a program reuses data structures carefully, it may create less pressure.

Most of the time, write clear code first.

Measure before tuning.

---

# Disabling Garbage Collection

The `gc` tool can disable automatic cyclic collection:

```python
import gc

gc.disable()
```

It can later be enabled:

```python
gc.enable()
```

This is almost never needed in beginner code.

It can be useful in specialized performance experiments or controlled systems that know they are not creating cycles.

But it is dangerous as a casual fix.

If your program creates cycles while automatic collection is disabled, unreachable cycles can accumulate.

Example:

```python
import gc

gc.disable()

for _ in range(100_000):
    item = []
    item.append(item)
```

Each loop creates a self-referential list.

Reference counting alone cannot clean those lists.

If cyclic collection is disabled, they may remain until collection is enabled and run again, or until the process exits.

So:

```text
do not disable garbage collection unless you deeply understand why
```

---

# Debugging With `gc`

The `gc` tool can help debug memory problems.

Some useful questions:

```text
Is automatic collection enabled?
How many objects were collected?
Is this object tracked?
What objects refer to this object?
What objects does this object refer to?
```

Example:

```python
import gc

print(gc.isenabled())
print(gc.get_count())
print(gc.get_threshold())
```

These functions reveal collector state.

But do not confuse visibility with simplicity.

Memory debugging can be difficult because observing objects often creates new references.

Example:

```python
refs = gc.get_referrers(some_object)
```

Now `refs` itself holds references to referrer objects.

Debugging tools can change the picture they are observing.

Use them carefully.

---

# Referrers and Referents

Two useful words:

```text
referent  -> object being referred to
referrer  -> object that holds the reference
```

Example:

```python
items = []
box = {"items": items}
```

`items` is a referent of `box`.

`box` is a referrer of `items`.

Conceptually:

```text
box ──▶ items
```

With `gc`, advanced debugging code can inspect relationships:

```python
import gc

print(gc.get_referents(box))
print(gc.get_referrers(items))
```

These are debugging tools.

They can return surprising objects from the interpreter, stack frames, globals, and the inspection process itself.

Do not use them as normal application logic.

Their main purpose is investigation.

---

# Why Exact Numbers Vary

When you experiment with garbage collection, numbers may surprise you.

Example:

```python
import gc

def make_cycle():
    a = []
    a.append(a)

make_cycle()

print(gc.collect())
```

You might expect:

```text
1
```

But you may see another number.

Reasons include:

* Other garbage may already exist.
* The interpreter may create helper objects.
* The environment may retain references.
* Collection may have happened earlier.
* Version details may differ.
* Debuggers and notebooks may keep extra state.

This is normal.

When learning, focus on the direction:

```text
unreachable cycles are collectable
```

Do not build your understanding around one exact printed count.

---

# Garbage Collection and Performance Myths

Myth:

```text
Python is slow because of garbage collection.
```

Reality:

Python performance depends on many factors:

* Interpreter overhead.
* Dynamic typing.
* Object allocation.
* Attribute lookup.
* Function calls.
* Algorithm choices.
* I/O.
* Data structure design.
* Garbage collection.

Garbage collection can matter.

But it is rarely the first thing to blame.

Another myth:

```text
Calling gc.collect() regularly makes programs faster.
```

Usually false.

Manual collection can add pauses and duplicate work the interpreter would manage automatically.

Use manual collection when you have measured a specific reason.

Professional rule:

```text
Measure first.
Tune second.
```

We will study profiling later.

---

# A Practical Example: Tree With Parent Links

Consider a tree:

```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.parent = None
        self.children = []

    def add_child(self, child):
        child.parent = self
        self.children.append(child)
```

Use it:

```python
root = TreeNode("root")
left = TreeNode("left")

root.add_child(left)
```

Graph:

```text
root ──▶ root node ──children──▶ left node
          ▲                         │
          │                         ▼
          └────────parent───────────┘
```

This cycle is useful.

The child can find its parent.

The parent can find its child.

Now:

```python
root = None
left = None
```

If no other references exist, the nodes and children list become unreachable.

The parent-child cycle should be collectable.

This lets you design natural object models without manually breaking every cycle in ordinary cases.

Still, for large structures, explicitly clearing references can sometimes help release memory sooner:

```python
root.children.clear()
left.parent = None
```

Whether that is necessary depends on the program.

---

# A Practical Example: Observer Callbacks

Cycles can appear in callback systems.

Example:

```python
class Button:
    def __init__(self):
        self.callbacks = []

    def on_click(self, callback):
        self.callbacks.append(callback)


class Screen:
    def __init__(self):
        self.button = Button()
        self.button.on_click(self.handle_click)

    def handle_click(self):
        print("clicked")
```

The bound method `self.handle_click` keeps a reference to `self`.

The screen keeps the button.

The button keeps the callback.

The callback keeps the screen.

Conceptually:

```text
Screen ──▶ Button ──▶ bound method ──▶ Screen
```

This is a cycle.

It may be fine.

When the screen is no longer reachable, the cycle can be collected.

But if some global UI registry keeps the button alive, the screen may also stay alive.

That would be reachable memory, not cyclic garbage.

Understanding the graph helps you debug the behavior.

---

# Weak References Preview

Sometimes you want a reference that does not keep an object alive.

That is called a weak reference.

Weak references are useful for:

* Caches.
* Parent links.
* Observer systems.
* Registries.
* Avoiding ownership cycles.

Example idea:

```text
child can find parent
but child should not keep parent alive
```

Weak references let you express:

```text
I want to refer to this object if it still exists,
but I do not want to be the reason it stays alive.
```

We will study weak references in Chapter 37.

For now, understand the motivation.

Cyclic garbage collection can clean many unreachable cycles.

Weak references help design object relationships with clearer ownership.

---

# Garbage Collection and Object Lifecycle

Garbage collection is part of object lifecycle.

Object lifecycle includes:

```text
creation
use
reference changes
becoming unreachable
finalization
deallocation
```

Reference counting can deallocate many objects immediately when their count reaches zero.

Cyclic garbage collection can reclaim unreachable groups whose reference counts never reached zero.

But cleanup can involve more than memory.

Objects may have finalizers.

Containers may need to release contained references.

External resources may need closing.

Interpreter shutdown has special rules.

This is why the next chapter studies object lifecycle directly.

Garbage collection tells us how unreachable cycles are found.

Object lifecycle tells us what happens around creation, cleanup, and finalization.

---

# Common Mistake: Thinking Cycles Are Always Leaks

Cycles are not always leaks.

Example:

```python
employee.manager = manager
manager.reports.append(employee)
```

This cycle may model a real relationship.

As long as the organization data is live, the cycle is live.

It is not a leak.

A leak happens when the program keeps data reachable after it is no longer needed.

Example:

```python
old_sessions.append(session)
```

If `old_sessions` is never cleaned, memory grows.

The problem is not necessarily a cycle.

The problem is ownership.

Ask:

```text
Who owns this object?
When should that ownership end?
What reference should be removed?
```

Those questions solve more memory problems than fear of cycles.

---

# Common Mistake: Relying on Collection Timing

Do not write code that requires garbage collection to run at a specific moment.

Bad idea:

```python
open("output.txt", "w").write("hello")
```

This creates a file object and does not explicitly close it.

In CPython, reference counting may close it quickly.

But relying on that behavior is poor practice.

Better:

```python
with open("output.txt", "w") as file:
    file.write("hello")
```

This works clearly across Python implementations.

It also works correctly when exceptions happen.

Rule:

```text
Memory cleanup can be automatic.
Resource cleanup should be explicit.
```

---

# Common Mistake: Calling `gc.collect()` Everywhere

If a program uses too much memory, it can be tempting to add:

```python
import gc

gc.collect()
```

in many places.

This is usually not the right first fix.

First ask:

```text
What is holding references?
Are objects actually unreachable?
Is a cache growing?
Is a list or dict accumulating values?
Are callbacks keeping objects alive?
Are exceptions or frames retained?
```

If objects are reachable, `gc.collect()` will not free them.

Manual collection only helps with collectable unreachable objects.

If the issue is reachable memory, you must remove references or change ownership.

Use `gc.collect()` as a diagnostic tool or a measured optimization, not a superstition.

---

# Common Mistake: Confusing Reference Count With Usefulness

An object can have a positive reference count and still be useless.

That is what cycles show.

Example:

```python
a = []
a.append(a)
del a
```

The list has a reference from itself.

But it is not useful because the program cannot reach it.

An object can also have many references and be genuinely useful.

Example:

```python
config = load_config()
services = [Service(config), Worker(config), Logger(config)]
```

The `config` object may have many references.

That is fine.

Reference count tells you how many references exist.

Reachability tells you whether the program can still access the object.

Usefulness is a design question.

These are related but distinct.

---

# Common Mistake: Forgetting Hidden References

References can hide in places that are not obvious.

Examples:

* Lists.
* Dictionaries.
* Sets.
* Closures.
* Default argument values.
* Class attributes.
* Global registries.
* Tracebacks.
* Bound methods.
* Caches.
* Logging or debugging systems.

Example:

```python
handlers = []

class Page:
    def __init__(self):
        handlers.append(self.handle)

    def handle(self):
        pass
```

Each `Page` registers a bound method.

The bound method refers to the `Page` instance.

The global `handlers` list keeps the bound method reachable.

Therefore it keeps the page reachable.

This may look like a leak.

But the garbage collector is correct not to collect it.

The object is reachable.

The fix is to unregister the handler or use a weak-reference-based design.

---

# Design Guidance

Most Python code does not need manual garbage-collector management.

Good memory behavior usually comes from ordinary design discipline:

* Keep ownership clear.
* Let local variables go out of scope naturally.
* Avoid unbounded global lists and dictionaries.
* Clear large containers when they are no longer needed.
* Use context managers for files and external resources.
* Be careful with callbacks and registries.
* Use weak references for non-owning relationships when appropriate.
* Measure memory before tuning the collector.

The collector is a safety net for unreachable cycles.

It is not a replacement for architecture.

When memory grows unexpectedly, draw the object graph.

Ask what is still reachable.

That simple question often reveals the issue.

---

# Exercises

1. Create a self-referential list:

```python
items = []
items.append(items)
```

Print it.

Then check:

```python
print(items[0] is items)
```

Explain why the result is `True`.

---

2. Create a cycle inside a function:

```python
def make_cycle():
    a = []
    b = []
    a.append(b)
    b.append(a)

make_cycle()
```

Explain why `a` and `b` are gone after the function returns, but the lists may still refer to each other.

---

3. Use `gc.collect()` in a small script:

```python
import gc

def make_cycle():
    a = []
    a.append(a)

make_cycle()
print(gc.collect())
```

Do not focus on the exact number.

Explain what the collector is being asked to do.

---

4. Draw the object graph:

```python
data = {}
data["items"] = []
data["items"].append(data)
```

Identify the cycle.

Then explain whether the cycle is garbage while the name `data` exists.

---

5. Compare these two snippets:

```python
def example_one():
    a = []
    a.append(a)

example_one()
```

and:

```python
registry = []

def example_two():
    a = []
    a.append(a)
    registry.append(a)

example_two()
```

Which cycle is unreachable after the function call?

Which cycle remains reachable?

Why?

---

6. Explain why this is better:

```python
with open("notes.txt") as file:
    text = file.read()
```

than relying on garbage collection to close the file.

---

7. Consider:

```python
callbacks = []

class Widget:
    def __init__(self):
        callbacks.append(self.update)

    def update(self):
        pass
```

Explain how `callbacks` can keep `Widget` instances alive.

Is this garbage?

Why or why not?

---

8. Write a short paragraph explaining the difference between:

```text
unreachable cyclic garbage
```

and:

```text
reachable but unwanted memory
```

Use your own example.

---

# Summary

In this chapter we learned:

* Garbage collection finds objects the program can no longer reach.
* Reference counting cannot collect cycles by itself.
* A reference cycle occurs when objects refer back to each other directly or indirectly.
* A cycle is not automatically garbage.
* Reachability decides whether objects are still live.
* Containers and user-defined objects commonly participate in cycles.
* Simple atomic objects usually do not create cycles by themselves.
* CPython's cyclic garbage collector supplements reference counting.
* Automatic collection timing is an implementation detail.
* The `gc` tool is useful for learning and debugging, not ordinary application logic.
* Memory leaks can still happen when unwanted objects remain reachable.
* Resource cleanup should use context managers, not garbage collection timing.
* Weak references help express non-owning relationships.

Core model:

```text
reference counting:
    clean up many objects when reference count reaches zero

cyclic garbage collection:
    find unreachable groups that still reference each other

application ownership:
    decide which reachable objects should be released
```

Garbage collection completes the memory-management picture started by reference counting.

Reference counting asks:

```text
How many references point here?
```

Garbage collection asks:

```text
Can the program still reach this object?
```

Together, they explain why Python can manage most memory automatically while still requiring programmers to design ownership thoughtfully.

---

# Preview of Chapter 36

Next we study object lifecycle.

So far we have seen how objects are allocated, referenced, and collected.

But objects also move through a broader lifecycle:

```text
creation -> initialization -> use -> becoming unreachable -> finalization -> deallocation
```

Chapter 36 explains this lifecycle carefully.

We will study:

* How objects are created.
* How initialization fits into construction.
* What happens when references disappear.
* Why cleanup timing differs between memory and external resources.
* How finalization works conceptually.
* Why resurrecting objects is dangerous.
* How object lifecycle prepares us for advanced object-oriented Python.

The transition is important:

```text
garbage collection explains how unreachable cycles are found
object lifecycle explains what happens around creation and cleanup
```

This prepares us for weak references, descriptors, classes, and advanced Python internals later in the book.
