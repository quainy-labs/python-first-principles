# Chapter 36 — Object Lifecycle

---

# Learning Objectives

By the end of this chapter, you should understand:

* What an object lifecycle is.
* How object creation differs from object initialization.
* Why names do not contain objects directly.
* How references keep objects alive.
* What it means for an object to become unreachable.
* How reference counting and garbage collection fit into the lifecycle.
* Why memory cleanup and resource cleanup are different.
* What finalization means.
* Why `__del__` should be used carefully.
* Why object resurrection is dangerous.
* How context managers provide predictable cleanup.
* How object lifecycle prepares you for weak references and advanced object-oriented Python.

So far, we have studied memory from several angles:

```text
stack vs heap
reference counting
garbage collection
```

Now we put those ideas together.

An object is not just "made" and "deleted."

It moves through a lifecycle.

That lifecycle starts when Python creates the object.

It continues while references keep the object reachable.

It ends when the object can no longer be used and its memory can be reclaimed.

Understanding this lifecycle makes Python feel less mysterious.

It also prevents many common mistakes about cleanup, destructors, files, sockets, caches, and long-running programs.

---

# Concept Overview

An object lifecycle is the sequence of stages an object goes through from creation to cleanup.

A simple lifecycle looks like this:

```text
create object
initialize object
use object
release references
object becomes unreachable
finalize if needed
deallocate memory
```

For many objects, this lifecycle is simple and fast.

Example:

```python
total = 10 + 20
```

Python creates or reuses integer objects, performs the operation, and binds the result to `total`.

For a larger object:

```python
items = [1, 2, 3]
```

Python creates a list object, stores references to the elements, and binds the name `items` to the list.

Later:

```python
items = None
```

The name `items` no longer refers to the list.

If no other references exist, the list can be cleaned up.

This is the basic lifecycle.

But real programs add complexity:

* Objects can refer to other objects.
* Objects can be stored inside containers.
* Objects can participate in cycles.
* Objects can manage external resources.
* Objects can define cleanup behavior.
* Objects can be kept alive accidentally.

This chapter is about seeing those stages clearly.

---

# Objects Are Created Before Names Point To Them

When you write:

```python
user = {"name": "Ada"}
```

it is easy to think Python starts with the name `user`.

But conceptually, Python must first create the dictionary object.

Then it binds the name `user` to that object.

Mental model:

```text
1. create dict object
2. create or reuse string objects
3. store references inside dict
4. bind name user to dict
```

Conceptually:

```text
user ──▶ dict ──▶ "Ada"
```

The name is not the object.

The name is a reference to the object.

This matters because an object can exist without the original name.

Example:

```python
user = {"name": "Ada"}
profile = user
del user
```

The dictionary still exists.

Why?

Because `profile` still refers to it.

Lifecycle is about objects, not names.

Names are only one kind of reference that can keep an object alive.

---

# Creation and Initialization

Creation and initialization are related but not identical.

Creation means:

```text
allocate or obtain the object
```

Initialization means:

```text
put the object into a useful starting state
```

For built-in objects, Python handles most of this for you.

Example:

```python
numbers = [1, 2, 3]
```

The list object is created.

Then it is initialized with references to `1`, `2`, and `3`.

For dictionaries:

```python
settings = {"debug": True}
```

The dictionary object is created.

Then the key-value relationship is stored.

For custom classes, these two ideas become more visible.

Example:

```python
class User:
    def __init__(self, name):
        self.name = name


user = User("Ada")
```

We have not deeply studied classes yet, but the idea is readable:

* Python creates a new `User` object.
* Python runs initialization code to attach `name`.
* The name `user` refers to the initialized object.

Later in the book, we will study `__new__` and `__init__` in detail.

For now, remember:

```text
creation obtains the object
initialization prepares the object
```

---

# The Name Binding Stage

After an object exists, names can be bound to it.

Example:

```python
items = []
```

The name `items` points to a list object.

```text
items ──▶ list
```

Another name can point to the same object:

```python
other = items
```

```text
items ──┐
        ▼
       list
        ▲
other ──┘
```

The list did not get copied.

Its lifecycle did not restart.

There is still one list object.

It now has another reference.

This is why mutation through one name is visible through the other:

```python
items.append("Python")
print(other)
```

Output:

```text
['Python']
```

Both names point to the same living object.

---

# The Use Stage

During its use stage, an object participates in computation.

It may be:

* Read.
* Mutated.
* Passed to functions.
* Returned from functions.
* Stored in containers.
* Referenced by closures.
* Attached to other objects.

Example:

```python
def add_item(target, value):
    target.append(value)


items = []
add_item(items, "Python")
```

During the function call:

```text
items  ──▶ list
target ──▶ same list
```

The list is alive because references point to it.

When the function returns, the local name `target` disappears.

But the list remains alive because `items` still points to it.

Object lifecycle is not affected by whether a specific name disappears.

It is affected by whether all relevant references disappear.

---

# References During Function Calls

Function calls temporarily create references.

Example:

```python
def inspect(value):
    print(value)


data = [1, 2, 3]
inspect(data)
```

While `inspect` runs:

```text
data  ──▶ list
value ──▶ same list
```

The parameter `value` keeps the list alive during the call.

When the function returns, `value` disappears.

The list remains if `data` still exists.

Now consider:

```python
inspect([1, 2, 3])
```

The list is created and passed directly.

During the call, the parameter `value` refers to it.

When the call ends, if no other reference was saved, the list can be cleaned up.

This is a tiny lifecycle:

```text
create list
bind parameter
use list
remove parameter reference
clean up list
```

Many temporary objects live very short lives.

---

# Returning Objects

Returning an object can extend its life.

Example:

```python
def build_user():
    user = {"name": "Ada"}
    return user


result = build_user()
```

Inside the function:

```text
user ──▶ dict
```

When the function returns, the local name `user` disappears.

But the returned object is assigned to `result`.

After the call:

```text
result ──▶ dict
```

The object survives the function frame.

This is normal.

Objects do not belong to the stack frame in the way local variables do.

The local name disappears.

The object survives if another reference receives it.

This is why returning newly created objects works.

---

# Containers Extend Lifetimes

Containers are one of the most common ways objects live longer than expected.

Example:

```python
users = []

def add_user(name):
    user = {"name": name}
    users.append(user)


add_user("Ada")
```

Inside `add_user`, the local name `user` points to a dictionary.

Then the dictionary is appended to the global list `users`.

After the function returns, the local name disappears.

But the dictionary remains alive:

```text
users ──▶ list ──▶ dict
```

This may be exactly what you want.

But it can also create memory growth if the container is never cleared.

Object lifecycle is often controlled by containers more than by local names.

Ask:

```text
What containers hold this object?
When are those containers cleared?
```

Those questions explain many real memory issues.

---

# Rebinding and Lifecycle

Rebinding a name changes what the name points to.

Example:

```python
data = [1, 2, 3]
data = [4, 5, 6]
```

The first line creates a list and binds `data` to it.

The second line creates another list and rebinds `data` to the new list.

Conceptually:

```text
before:
data ──▶ [1, 2, 3]

after:
data ──▶ [4, 5, 6]
```

The old list loses the reference from `data`.

If no other references exist, it can be cleaned up.

But:

```python
data = [1, 2, 3]
backup = data
data = [4, 5, 6]
```

Now:

```text
data   ──▶ [4, 5, 6]
backup ──▶ [1, 2, 3]
```

The old list remains alive through `backup`.

Rebinding does not destroy an object.

It removes one reference.

---

# Deleting Names

The `del` statement removes a binding.

Example:

```python
items = [1, 2, 3]
del items
```

After `del`, the name `items` is no longer available.

Trying to use it raises an error:

```python
print(items)
```

The error:

```text
NameError
```

But `del` does not mean "destroy the object immediately" in the abstract language model.

It means:

```text
remove this reference
```

Example:

```python
items = [1, 2, 3]
other = items

del items

print(other)
```

Output:

```text
[1, 2, 3]
```

The list remains alive.

The name `items` disappeared.

The object did not.

---

# Becoming Unreachable

An object becomes unreachable when the program has no path to it from live roots.

Example:

```python
def create():
    data = [1, 2, 3]


create()
```

Inside `create`, `data` points to the list.

After the function returns, `data` disappears.

If nothing else refers to the list, the list is unreachable.

For a non-cyclic object in CPython, reference counting usually cleans it up immediately.

For a cyclic group:

```python
def create_cycle():
    data = []
    data.append(data)


create_cycle()
```

After the function returns, the name `data` disappears.

The list still refers to itself.

It is unreachable from the program, but its reference count is not zero.

The cyclic garbage collector can clean it up later.

The lifecycle stage is the same:

```text
object became unreachable
```

The cleanup mechanism differs.

---

# Cleanup Mechanisms

Python cleanup can involve several ideas:

```text
reference count reaches zero
cyclic garbage collector finds unreachable cycle
object finalization runs if defined
memory is returned to Python's allocator
allocator may reuse memory later
```

These are not all the same event.

For simple objects, the path may look like:

```text
last reference removed
reference count reaches zero
object deallocated
```

For cycles, the path may look like:

```text
outside references removed
cycle remains internally referenced
collector detects unreachable group
objects finalized and cleared safely
memory can be reclaimed
```

For objects managing external resources, there is another concern:

```text
resource must be released predictably
```

That should be handled explicitly with context managers or `try` / `finally`.

Memory lifecycle and resource lifecycle often overlap, but they should not be confused.

---

# Memory Cleanup vs Resource Cleanup

Memory is managed automatically.

External resources should be managed deliberately.

External resources include:

* Files.
* Network sockets.
* Database connections.
* Locks.
* Temporary directories.
* Subprocesses.

Consider:

```python
file = open("notes.txt")
text = file.read()
```

The file object is a Python object.

But it also represents an operating-system resource.

If you wait for the object to be cleaned up, the file may eventually close.

But "eventually" is not the kind of guarantee good programs should rely on.

Better:

```python
with open("notes.txt") as file:
    text = file.read()
```

Now the file is closed when the block exits.

That is deterministic cleanup.

Rule:

```text
Use Python's memory management for memory.
Use context managers for external resources.
```

---

# Context Managers in the Lifecycle

The `with` statement gives an object a controlled use period.

Example:

```python
with open("notes.txt") as file:
    text = file.read()
```

Conceptual lifecycle:

```text
open file
enter context
use file
exit context
close file
```

This happens even if an exception occurs:

```python
with open("notes.txt") as file:
    raise RuntimeError("problem")
```

The context manager still gets a chance to clean up.

This is why context managers are essential for professional Python.

They separate resource cleanup from memory cleanup.

The file object may still exist briefly after the block depending on references and implementation details.

But the file resource has been released.

That is what matters.

We will study context managers in depth later.

For now, remember their role in object lifecycle:

```text
they define a predictable usage boundary
```

---

# Finalization

Finalization means running cleanup logic associated with an object before or during object destruction.

In Python, one visible finalization hook is `__del__`.

Example:

```python
class Example:
    def __del__(self):
        print("cleaning up")
```

When an `Example` object is finalized, Python may call `__del__`.

But this method is subtle.

You should not treat it like a normal function you control.

Reasons:

* You usually do not control exactly when it runs.
* It may run during interpreter shutdown.
* Other objects it needs may already be gone.
* Exceptions inside it cannot be handled normally by the caller.
* Cycles involving finalizers require careful handling.

Modern Python handles finalization much better than older Python versions, but `__del__` is still an advanced tool.

Most code should prefer explicit cleanup.

Example:

```python
class Connection:
    def close(self):
        print("closing")
```

Then:

```python
connection = Connection()
try:
    use(connection)
finally:
    connection.close()
```

or better, design it as a context manager later.

---

# Why `__del__` Is Not `finally`

The `finally` block runs as part of normal control flow.

Example:

```python
resource = acquire()

try:
    use(resource)
finally:
    release(resource)
```

The cleanup is tied to a visible block of code.

By contrast, `__del__` is tied to object finalization.

That depends on object reachability and interpreter behavior.

Bad mental model:

```text
__del__ is a reliable place for normal cleanup
```

Better mental model:

```text
__del__ is a last-chance finalization hook
```

If you need cleanup at a specific time, use:

```text
with
try/finally
explicit close methods
```

Do not make important program correctness depend on finalizer timing.

---

# Object Resurrection

Object resurrection happens when an object being finalized becomes reachable again.

This is unusual, but possible.

Example:

```python
saved = None

class Lazarus:
    def __del__(self):
        global saved
        saved = self
```

Now:

```python
obj = Lazarus()
obj = None
```

When the object is finalized, `__del__` assigns `self` to the global name `saved`.

The object becomes reachable again.

Conceptually:

```text
object was dying
finalizer created a new reference
object became reachable again
```

This is called resurrection.

It makes lifecycle reasoning difficult.

Avoid it.

A finalizer should not usually store `self` somewhere new.

If you see resurrection in real code, treat it as advanced and suspicious unless there is a very specific reason.

---

# Object Deallocation

Deallocation means the object's memory can be reclaimed by Python's memory system.

But there is an important nuance.

When an object is deallocated, memory may return to Python's internal allocator rather than directly to the operating system.

That means process memory shown by your operating system may not immediately decrease.

Example:

```python
data = [0] * 1_000_000
data = None
```

The list may become unreachable.

The list object and its internal storage may be released for reuse.

But your process memory display may not drop immediately.

Why?

Because Python may keep memory arenas or pools for future allocations.

This is normal.

Do not assume:

```text
object freed -> operating system memory immediately decreases
```

Better:

```text
object freed -> Python may reuse that memory later
```

This distinction matters when debugging memory usage.

---

# Small Objects and Reuse

Python may reuse objects or memory internally.

For example, small integers are commonly reused.

Strings may be interned in some cases.

Lists, dictionaries, tuples, and other built-in types may use internal allocation optimizations.

These are implementation details.

They exist for performance.

Example:

```python
a = 10
b = 10
```

You should not build broad lifecycle conclusions from whether `a is b` happens to be true for small values.

The language-level lesson remains:

```text
names refer to objects
objects live while reachable
cleanup is managed by the implementation
```

Implementation optimizations can affect identity, memory reuse, and timing.

They do not change the core model.

---

# Lifecycle of an Immutable Object

Immutable objects cannot be changed after creation.

Example:

```python
name = "Ada"
```

The string object contains text.

If you write:

```python
upper_name = name.upper()
```

Python does not modify the original string.

It creates or returns another string object.

Conceptually:

```text
name       ──▶ "Ada"
upper_name ──▶ "ADA"
```

The original object remains alive while `name` refers to it.

The new object remains alive while `upper_name` refers to it.

When references disappear, objects can be cleaned up or reused according to implementation details.

Immutability affects what can happen during the use stage.

It does not remove the lifecycle.

Immutable objects are still created, referenced, and eventually no longer needed.

---

# Lifecycle of a Mutable Object

Mutable objects can change during their lifetime.

Example:

```python
items = []
items.append("Python")
items.append("Memory")
```

The same list object changes over time.

Conceptually:

```text
start:
items ──▶ []

after append:
items ──▶ ["Python"]

after another append:
items ──▶ ["Python", "Memory"]
```

The list identity remains the same.

Its contents change.

This is different from rebinding:

```python
items = []
items = ["Python"]
```

Here the name is changed to point to a different list object.

Lifecycle distinction:

```text
mutation changes an object
rebinding changes a reference
```

This distinction is central to understanding object lifetime.

---

# Lifecycle of a Container

Containers have lifecycles of their own, and they also affect the lifecycles of contained objects.

Example:

```python
outer = []
inner = {"language": "Python"}

outer.append(inner)
```

Graph:

```text
outer ──▶ list ──▶ dict ──▶ "Python"
inner ───────────▶ dict
```

Now:

```python
inner = None
```

The dictionary remains alive through `outer`.

```text
outer ──▶ list ──▶ dict
```

Now:

```python
outer.clear()
```

The list releases its reference to the dictionary.

If no other references exist, the dictionary can be cleaned up.

Containers are lifetime managers.

They can keep objects alive far beyond the local scope where those objects were created.

---

# Clearing Containers

Sometimes you want a container object to remain alive but release what it contains.

Example:

```python
cache = {}

cache["a"] = [1, 2, 3]
cache["b"] = [4, 5, 6]
```

To release the stored values:

```python
cache.clear()
```

The dictionary object remains.

Its entries are removed.

The values lose references from the dictionary.

If nothing else refers to those lists, they can be cleaned up.

Compare:

```python
cache = {}
```

This rebinds the name `cache` to a new dictionary.

The old dictionary may be cleaned up if no other references exist.

Difference:

```text
cache.clear() -> mutate existing dictionary
cache = {}    -> bind name to a new dictionary
```

Both can release references, but they affect object identity differently.

---

# Lifecycle and Aliasing

Aliasing means multiple references point to the same object.

Example:

```python
a = []
b = a
c = b
```

Graph:

```text
a ──┐
b ──┼──▶ list
c ──┘
```

The list remains alive until all references are gone or no live path can reach it.

```python
del a
del b
```

The list still exists because `c` refers to it.

Only after:

```python
del c
```

can the list become unreachable, assuming no hidden references exist.

Aliasing is why lifecycle reasoning must look at the whole graph.

You cannot decide whether an object is dead by inspecting one name.

---

# Lifecycle and Closures

Closures can extend object lifetimes.

Example:

```python
def make_counter():
    count = {"value": 0}

    def increment():
        count["value"] += 1
        return count["value"]

    return increment


counter = make_counter()
```

The function `make_counter` has returned.

But the dictionary `count` is still alive.

Why?

Because `increment` closes over it.

Conceptually:

```text
counter ──▶ increment function ──▶ closure cell ──▶ count dict
```

The local variable's frame is gone.

The object survives through the closure.

This is not a leak.

It is the intended behavior of closures.

But it is another reason object lifetimes can be longer than local scopes.

---

# Lifecycle and Globals

Global variables often keep objects alive for the duration of a program.

Example:

```python
CONFIG = {"debug": False}
```

The dictionary is reachable through the module global `CONFIG`.

It will usually remain alive as long as the module remains loaded and the name remains bound.

Global registries can be especially important:

```python
REGISTERED_HANDLERS = []

def register(handler):
    REGISTERED_HANDLERS.append(handler)
```

Anything appended to `REGISTERED_HANDLERS` may live for a long time.

This is useful when intentional.

It is dangerous when accidental.

Global containers are common sources of reachable memory growth.

The garbage collector will not collect objects still reachable through globals.

---

# Lifecycle and Caches

Caches intentionally extend object lifetimes.

Example:

```python
cache = {}

def get_user(user_id):
    if user_id not in cache:
        cache[user_id] = load_user(user_id)
    return cache[user_id]
```

The cache keeps loaded users alive.

This may improve performance.

But if `user_id` values are unbounded, memory can grow forever.

The objects are reachable.

They are not garbage.

Lifecycle design question:

```text
When should cached objects expire?
```

Possible policies include:

* Clear the cache manually.
* Limit cache size.
* Expire entries by time.
* Use weak references in appropriate cases.
* Use existing cache utilities.

Automatic memory management does not replace cache policy.

---

# Lifecycle and Cycles

Cycles affect the cleanup stage.

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

While `a` and `b` exist, the cycle is reachable.

Now:

```python
a = None
b = None
```

If no other references exist, the two nodes become unreachable.

But they still reference each other.

Reference counting alone may not clean them immediately.

Cyclic garbage collection can detect the unreachable group.

Lifecycle path:

```text
objects created
objects linked into cycle
external names removed
cycle becomes unreachable
garbage collector finds cycle
objects can be finalized and deallocated
```

This is why Chapter 35 came before this chapter.

Object lifecycle includes both simple and cyclic cleanup paths.

---

# Lifecycle and Weak References

Sometimes you want to refer to an object without extending its lifetime.

That is what weak references are for.

Strong reference:

```text
I point to this object and keep it alive.
```

Weak reference:

```text
I point to this object only if it is still alive.
I do not keep it alive by myself.
```

This is useful for relationships where one object should not own another.

Example:

```text
child can know parent
but child should not force parent to stay alive
```

Weak references are also useful for caches and registries.

They allow lifecycle-sensitive designs:

```text
keep metadata while object exists
let metadata disappear when object disappears
```

Chapter 37 studies this directly.

For now, see weak references as a way to design object lifetimes more precisely.

---

# Hidden References

Objects are sometimes kept alive by references you did not notice.

Common places include:

* Lists.
* Dictionaries.
* Sets.
* Closures.
* Default argument values.
* Class attributes.
* Global variables.
* Tracebacks.
* Bound methods.
* Debuggers.
* Interactive history.

Example:

```python
history = []

def remember(value):
    history.append(value)
```

Every remembered object remains reachable through `history`.

Another example:

```python
callbacks = []

class Widget:
    def __init__(self):
        callbacks.append(self.update)

    def update(self):
        pass
```

The bound method `self.update` refers to `self`.

The global `callbacks` list stores the bound method.

Therefore the `Widget` instance can remain alive.

This is not mysterious once you draw the graph:

```text
callbacks ──▶ bound method ──▶ Widget instance
```

Hidden references are often visible once you know where to look.

---

# Object Lifecycle and Errors

Errors can affect object lifetimes.

Example:

```python
def process():
    data = [0] * 1_000_000
    raise RuntimeError("failed")
```

If the exception is not stored, local references usually disappear as the stack unwinds.

But if you store exception objects, tracebacks, or frames, you may keep local variables alive.

Example:

```python
saved_errors = []

try:
    process()
except RuntimeError as error:
    saved_errors.append(error)
```

Depending on traceback references, this can keep more data reachable than expected.

The exact details can vary, but the lesson is stable:

```text
diagnostic objects can hold references to execution state
```

When debugging memory growth, inspect stored exceptions, frames, and tracebacks.

They can extend object lifetimes.

---

# Object Lifecycle and Interpreter Shutdown

Interpreter shutdown is a special phase.

When a Python program exits, the interpreter cleans up modules, globals, and remaining objects.

This phase can be messy compared with normal execution.

Why?

Because global names may be cleared in an order your code did not choose.

If a finalizer runs during shutdown, objects it wants to use may already be gone or partially unavailable.

Example:

```python
class Reporter:
    def __del__(self):
        print("reporting shutdown")
```

This might work in a simple case.

But finalizers that rely on module globals, open files, logging systems, or network resources can behave unpredictably during shutdown.

This is another reason to avoid putting critical cleanup in `__del__`.

Do important cleanup before shutdown:

```python
with resource:
    run_program()
```

or:

```python
resource = acquire()
try:
    run_program()
finally:
    resource.close()
```

---

# The Lifecycle of a File Object

A file object is useful because it shows both object lifecycle and resource lifecycle.

Example:

```python
file = open("notes.txt")
text = file.read()
file.close()
```

Object lifecycle:

```text
file object created
name file refers to it
program uses it
program closes it
references eventually disappear
object memory is cleaned up
```

Resource lifecycle:

```text
operating system file handle opened
data read
file handle closed
```

These lifecycles overlap but are not identical.

Closing the file releases the operating-system resource.

The Python file object may still exist as a closed object:

```python
file = open("notes.txt")
file.close()
print(file.closed)
```

Output:

```text
True
```

The object still exists.

The resource has been released.

This is why resource cleanup and memory cleanup are separate concepts.

---

# The Lifecycle of a List

A list is a good example of a pure memory object.

Example:

```python
items = []
items.append("a")
items.append("b")
items.pop()
items = None
```

Lifecycle:

```text
list created
name items points to it
list grows
list shrinks
name items releases it
list becomes unreachable
memory can be reclaimed or reused
```

The list does not need to close a file or release a socket.

It only owns memory and references to contained objects.

When the list is cleaned up, it releases references to its elements.

Those elements may then become collectable too.

This cascading release is common:

```text
container cleaned up -> contained references released -> more objects may become unreachable
```

---

# The Lifecycle of a Dictionary

A dictionary stores references to keys and values.

Example:

```python
data = {}
key = "language"
value = ["Python"]

data[key] = value
```

Graph:

```text
data ──▶ dict
          │
          ├──▶ "language"
          └──▶ list ──▶ "Python"

key ─────▶ "language"
value ───▶ list
```

Now:

```python
value = None
```

The list remains alive through the dictionary.

Now:

```python
del data["language"]
```

The dictionary releases references to the key and value.

If no other references exist, those objects may become collectable.

Dictionaries often serve as ownership maps.

The lifecycle of many objects is tied to whether their dictionary entries still exist.

---

# Object Lifecycle Is Graph Lifecycle

Individual objects matter.

But in real Python programs, lifecycles usually happen in graphs.

Example:

```python
app = {
    "routes": [],
    "config": {},
    "cache": {},
}
```

This one global object can keep an entire graph alive:

```text
app ──▶ routes list
app ──▶ config dict
app ──▶ cache dict
cache ──▶ cached objects
routes ──▶ route handlers
handlers ──▶ closures
closures ──▶ captured state
```

When memory grows, do not ask only:

```text
Why was this object not deleted?
```

Ask:

```text
What path still reaches this object?
```

That question moves you from superstition to diagnosis.

Object lifecycle is graph lifecycle.

---

# Common Mistake: Thinking Scope Owns Objects

Beginners often think local scope owns objects.

Example:

```python
def build():
    items = []
    return items


result = build()
```

The list was created inside `build`.

But it survives after `build` returns.

Why?

Because it was returned and assigned to `result`.

Scope owns names.

References keep objects alive.

Better mental model:

```text
function scope controls local names
object lifetime depends on reachability
```

This distinction explains returns, closures, containers, and many memory behaviors.

---

# Common Mistake: Thinking `del` Calls a Destructor

The statement:

```python
del name
```

removes a name binding.

It does not mean:

```text
call destructor now
```

Example:

```python
a = []
b = a
del a
```

The list remains alive through `b`.

No cleanup of the list should be expected just because `a` was deleted.

If `del` removes the last reference in CPython, cleanup may happen immediately for non-cyclic objects.

But that is a consequence of reference counting, not the meaning of `del`.

Meaning:

```text
remove this reference
```

Possible result:

```text
object may become unreachable
```

Those are separate ideas.

---

# Common Mistake: Thinking `None` Deletes Objects

This code:

```python
data = [1, 2, 3]
data = None
```

does not delete the list directly.

It rebinds `data` to `None`.

The old list loses one reference.

If no other references exist, it may be cleaned up.

But:

```python
data = [1, 2, 3]
other = data
data = None
```

The list remains alive:

```text
other ──▶ [1, 2, 3]
```

Assigning `None` is useful when you want to release a reference early.

It is not an object destruction command.

---

# Common Mistake: Depending on CPython Timing

CPython's reference counting often cleans up objects as soon as their reference count reaches zero.

This can make cleanup look deterministic.

Example:

```python
open("notes.txt").read()
```

In CPython, the temporary file object may be cleaned up quickly.

But relying on that behavior is not good Python style.

Other Python implementations may use different memory-management strategies.

Even in CPython, cycles and finalization can make timing less direct.

Write code based on language guarantees and clear resource management:

```python
with open("notes.txt") as file:
    text = file.read()
```

Professional Python avoids depending on accidental cleanup timing.

---

# Common Mistake: Believing Freed Memory Always Shrinks Process Memory

You may run:

```python
data = [0] * 10_000_000
data = None
```

Then check your operating system memory monitor.

You may expect memory usage to drop immediately.

It might not.

Possible reasons:

* Python may keep memory available for reuse.
* The allocator may hold arenas.
* Fragmentation may prevent returning memory directly.
* Other objects may still be alive.
* The environment may keep hidden references.

This does not always mean the object is still reachable.

It means process memory is not the same as one object's lifecycle.

Memory debugging requires careful tools and measurements.

We will return to profiling and memory analysis later in the book.

---

# Common Mistake: Finalizers Doing Too Much

A finalizer should not usually perform complex application logic.

Risky finalizer:

```python
class Report:
    def __del__(self):
        send_network_request()
        write_to_database()
        update_global_state()
```

Problems:

* The network may be unavailable.
* The database connection may already be closed.
* Global state may be partially shut down.
* Exceptions cannot be handled by normal callers.
* Cleanup timing may be surprising.

Better design:

```python
class Report:
    def close(self):
        write_to_database()
```

Then call `close()` explicitly, preferably through a context manager.

Finalizers should be small, defensive, and rare.

---

# Designing With Lifecycle in Mind

Good Python design asks:

```text
Who creates this object?
Who owns this object?
Who else can reference it?
When should it stop being reachable?
Does it manage external resources?
Does it need explicit cleanup?
Could there be cycles?
Should any reference be weak?
```

These questions are practical.

Example:

```python
class Session:
    pass
```

If sessions are stored in a global dictionary:

```python
sessions[user_id] = session
```

then the dictionary controls their lifetime.

You need a policy:

```text
remove session when user logs out
expire inactive sessions
limit total sessions
```

Garbage collection cannot guess that policy.

Lifecycle-aware design makes memory behavior intentional.

---

# A Full Lifecycle Example

Consider this small example:

```python
class Document:
    def __init__(self, title):
        self.title = title
        self.sections = []

    def add_section(self, text):
        self.sections.append(text)


def build_document():
    doc = Document("Python Memory")
    doc.add_section("Objects")
    doc.add_section("References")
    return doc


book = build_document()
```

Lifecycle:

```text
Document class exists
build_document is called
Document object is created
__init__ initializes title and sections
sections list is created
strings are referenced
local name doc refers to Document
doc is returned
global name book refers to Document
function frame disappears
Document survives through book
```

Now:

```python
book = None
```

If no other references exist:

```text
Document becomes unreachable
sections list becomes unreachable
contained strings may lose references
objects can be cleaned up or reused
```

This example contains most lifecycle ideas in one place.

---

# A Full Resource Example

Now consider a resource-owning object:

```python
class LogFile:
    def __init__(self, path):
        self.file = open(path, "a")

    def write(self, message):
        self.file.write(message + "\n")

    def close(self):
        self.file.close()
```

Use it carefully:

```python
log = LogFile("app.log")

try:
    log.write("starting")
finally:
    log.close()
```

Object lifecycle:

```text
LogFile object created
file object created
name log refers to LogFile
program writes
program closes file explicitly
references later disappear
memory cleaned up
```

Resource lifecycle:

```text
file handle opened
file handle used
file handle closed in finally
```

This is better than hoping `LogFile` is collected soon.

Eventually, you would make this a context manager:

```python
with LogFile("app.log") as log:
    log.write("starting")
```

We will learn how later.

---

# Lifecycle Checklist

When you are unsure why an object is still alive, use this checklist:

* Is it stored in a list, dictionary, set, or tuple?
* Is it referenced by a global variable?
* Is it captured by a closure?
* Is it stored in a class attribute?
* Is it part of a cycle?
* Is it referenced by a traceback or frame?
* Is it stored in a cache or registry?
* Is it referenced by a bound method?
* Is an interactive environment keeping it alive?
* Did you confuse rebinding with mutation?
* Did you clear the container that owns it?

The most important question:

```text
What live path can still reach this object?
```

If you can answer that, you can usually understand the lifecycle.

---

# Exercises

1. Explain the lifecycle of this object:

```python
def make_list():
    values = [1, 2, 3]
    return values


items = make_list()
items = None
```

Describe when the list is created, how it survives the function return, and when it can become unreachable.

---

2. Draw the object graph:

```python
a = []
b = a
c = [a]
```

How many paths can reach the first list?

What happens after:

```python
del a
del b
```

Is the list still reachable?

---

3. Compare:

```python
cache.clear()
```

with:

```python
cache = {}
```

How are these different for object identity and references?

---

4. Explain why this object may stay alive:

```python
callbacks = []

class Button:
    def __init__(self):
        callbacks.append(self.click)

    def click(self):
        pass
```

What path reaches the `Button` instance?

---

5. Explain why this is safer:

```python
with open("data.txt") as file:
    text = file.read()
```

than:

```python
file = open("data.txt")
text = file.read()
file = None
```

Focus on resource lifecycle, not only memory lifecycle.

---

6. Write a short example where a local object survives because it is stored in a global list.

Then explain which reference keeps it alive.

---

7. Create a self-referential list:

```python
items = []
items.append(items)
```

Explain its lifecycle while `items` exists.

Then explain what changes after:

```python
items = None
```

---

8. In your own words, explain the difference between:

```text
object is unreachable
```

and:

```text
object's memory has been returned to the operating system
```

Why are these not the same thing?

---

# Summary

In this chapter we learned:

* Object lifecycle describes how objects move from creation to cleanup.
* Creation and initialization are related but different.
* Names refer to objects; they do not contain objects.
* Objects remain alive while reachable through references.
* Function calls create temporary references.
* Returning an object can extend its lifetime beyond a function frame.
* Containers can keep objects alive long after local names disappear.
* Rebinding and `del` remove references; they do not directly mean "destroy this object."
* Objects become unreachable when no live path can reach them.
* Reference counting and garbage collection are cleanup mechanisms.
* Memory cleanup and external resource cleanup are different concerns.
* Context managers provide predictable resource cleanup.
* Finalizers such as `__del__` are advanced and should be used carefully.
* Object resurrection makes lifecycle reasoning difficult.
* Python may reuse freed memory internally instead of returning it immediately to the operating system.
* Lifecycle-aware design requires clear ownership.

Core model:

```text
create object
initialize object
bind references
use object
release references
become unreachable
finalize if needed
deallocate or reuse memory
```

The most useful debugging question is:

```text
What live path can still reach this object?
```

That question connects names, containers, closures, globals, caches, cycles, and garbage collection into one practical model.

---

# Preview of Chapter 37

Next we study weak references.

So far, every normal reference we have discussed keeps an object alive.

That is called a strong reference.

But sometimes a program needs to refer to an object without owning it.

Chapter 37 explains weak references.

We will study:

* What weak references are.
* Why normal references are strong references.
* Why weak references do not keep objects alive.
* How weak references help with caches.
* How weak references help with parent links and observer systems.
* Why not every object supports weak references.
* How weak references connect to object lifecycle.

The transition is natural:

```text
object lifecycle asks when objects should stay alive
weak references let us refer to objects without extending that lifetime
```

Weak references complete Part VIII by giving us a design tool for precise ownership.
