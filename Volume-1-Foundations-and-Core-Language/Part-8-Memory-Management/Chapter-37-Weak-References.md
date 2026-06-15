# Chapter 37 — Weak References

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a weak reference is.
* How weak references differ from normal references.
* Why normal references are called strong references.
* What a referent is.
* Why weak references do not keep objects alive.
* How weak references connect to object lifecycle.
* How weak references help with caches.
* How weak references help with registries.
* How weak references help with parent links and observer-style relationships.
* Why not every object supports weak references.
* What `WeakValueDictionary`, `WeakKeyDictionary`, and `WeakSet` are for.
* Why bound methods require special care.
* Why `weakref.finalize()` is often safer than `__del__`.
* When weak references are useful and when they are the wrong tool.

This chapter completes Part VIII.

We have already studied:

```text
stack vs heap
reference counting
garbage collection
object lifecycle
```

Now we add one more idea:

```text
not every reference needs to own the object it points to
```

Weak references let a program point to an object without keeping that object alive.

That sounds small.

It is not.

It gives us a way to design ownership more precisely.

---

# Concept Overview

A normal Python reference keeps an object alive.

Example:

```python
class User:
    pass


user = User()
```

The name `user` refers to the `User` object.

As long as the name remains reachable, the object remains alive.

This is a strong reference.

Most references in Python are strong references.

Examples:

* A name referring to an object.
* A list containing an object.
* A dictionary value pointing to an object.
* An instance attribute pointing to another object.
* A closure capturing an object.

Strong reference:

```text
I point to this object, and I help keep it alive.
```

Weak reference:

```text
I point to this object if it still exists, but I do not keep it alive.
```

Weak references are provided by Python's `weakref` module.

We have not studied modules deeply yet.

For now, read:

```python
import weakref
```

as:

```text
make Python's weak-reference tools available
```

---

# A First Weak Reference

Weak references work with many user-defined objects.

Example:

```python
import weakref


class User:
    pass


user = User()
ref = weakref.ref(user)
```

Here:

* `user` is a strong reference to the `User` object.
* `ref` is a weak reference object.

Conceptually:

```text
user ─────────────▶ User object

ref  - - - - - - -▶ User object
```

The solid arrow is strong.

The dashed arrow is weak.

The weak reference does not keep the object alive.

To get the object from a weak reference, call the weak reference:

```python
same_user = ref()
print(same_user is user)
```

Output:

```text
True
```

As long as the object is alive, calling `ref()` returns it.

If the object is gone, calling `ref()` returns `None`.

---

# The Referent

The object pointed to by a weak reference is called the referent.

Example:

```python
import weakref


class Document:
    pass


document = Document()
document_ref = weakref.ref(document)
```

In this example:

```text
document_ref -> weak reference object
document     -> strong reference
Document instance -> referent
```

The referent is the object being referred to.

This word matters because weak-reference APIs often talk about:

```text
referent is alive
referent has been collected
referent no longer exists
```

When the referent is alive:

```python
document_ref()
```

returns the object.

When the referent has been collected:

```python
document_ref()
```

returns `None`.

Weak references are not containers of objects.

They are handles that may or may not still be able to retrieve an object.

---

# Strong References

Before weak references feel natural, strong references must be clear.

Example:

```python
user = User()
users = [user]
current = user
```

All of these are strong references:

```text
user    ──▶ User object
users   ──▶ list ──▶ User object
current ──▶ User object
```

The object remains alive while any strong path can reach it.

If:

```python
user = None
current = None
```

the object is still alive because the list still contains it:

```text
users ──▶ list ──▶ User object
```

Only after:

```python
users.clear()
```

can the object become unreachable, assuming no other strong references exist.

Strong references express ownership or at least lifetime participation.

They say:

```text
this object should remain alive while I point to it
```

Weak references do not say that.

---

# Weak References Do Not Own

A weak reference does not keep its referent alive.

Example:

```python
import weakref


class User:
    pass


user = User()
ref = weakref.ref(user)

print(ref() is user)

user = None

print(ref())
```

In many normal script runs on CPython, the output is:

```text
True
None
```

Why?

Because after `user = None`, no strong reference to the `User` object remains.

The weak reference alone is not enough.

The object can be cleaned up.

Then `ref()` returns `None`.

Important:

```text
weak reference object can remain alive
referent object can be gone
```

The weak reference is like a note that says:

```text
I used to know where this object was.
If it is still alive, I can get it.
If it is gone, I return None.
```

---

# Why Weak References Exist

Weak references exist because not every relationship should extend lifetime.

Suppose you have a cache:

```python
cache = {}
```

You store a large object:

```python
cache["image-1"] = image
```

Now the cache keeps the image alive.

That may be good.

But sometimes you want a different policy:

```text
Keep this object in the cache only while something else is already using it.
Do not let the cache be the only reason the object stays alive.
```

That is a weak-reference use case.

Another example:

```text
child knows parent
```

If the child strongly references the parent, the child can keep the parent alive.

Sometimes that is correct.

Sometimes it is not.

Weak references let you represent:

```text
I can find this object if it is still around,
but I do not own it.
```

This is an ownership design tool.

---

# Weak References and Object Lifecycle

Chapter 36 gave us this lifecycle model:

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

Weak references affect the "bind references" stage.

They create a reference-like relationship without extending the object's lifetime.

Example:

```python
import weakref


class Session:
    pass


session = Session()
session_ref = weakref.ref(session)
```

Lifecycle:

```text
Session object created
strong name session keeps it alive
weak ref session_ref observes it
```

Now:

```python
session = None
```

The weak reference does not prevent cleanup.

If no strong references remain:

```text
Session object can be cleaned up
session_ref remains but returns None
```

Weak references let us observe object lifetime without owning the object.

---

# Checking a Weak Reference Safely

To use a weak reference, call it and check the result.

Pattern:

```python
obj = ref()
if obj is None:
    print("object is gone")
else:
    obj.do_something()
```

Do not write this as two separate calls:

```python
if ref() is not None:
    ref().do_something()
```

Why?

Between the first call and the second call, the object could disappear in a threaded program.

Even if you are not using threads yet, it is better to learn the correct idiom:

```python
obj = ref()
if obj is not None:
    obj.do_something()
```

The local name `obj` temporarily becomes a strong reference.

While `obj` exists, it keeps the object alive for the operation.

This pattern is simple and professional.

---

# Weak Reference Returning `None`

Calling a weak reference can return `None`.

That is not an error.

It is the normal signal that the referent is gone.

Example:

```python
import weakref


class Item:
    pass


item = Item()
item_ref = weakref.ref(item)

print(item_ref())

item = None

print(item_ref())
```

The first `print` shows an `Item` object.

The second may show:

```text
None
```

This is the central weak-reference behavior:

```text
weak ref alive
referent gone
weak ref returns None
```

Your code must be prepared for this.

If your code cannot handle the object disappearing, a weak reference may be the wrong tool.

---

# Weak References Are Not Faster References

Weak references are not an optimization you sprinkle everywhere.

They are not "lighter normal references" in everyday code.

They are more complex than strong references because:

* The object may disappear.
* Every use must handle `None`.
* Not every object supports weak references.
* Weak containers have special behavior.
* Debugging ownership can become subtler.

Use weak references when they express the relationship correctly.

Good reason:

```text
this object should not keep that object alive
```

Weak reason:

```text
maybe weak references are more advanced
```

Professional code uses simple strong references by default.

It uses weak references when ownership requires them.

---

# Caches With Strong References

A normal dictionary keeps its keys and values alive.

Example:

```python
cache = {}

class Image:
    pass


image = Image()
cache["logo"] = image

image = None
```

The `Image` object remains alive.

Why?

Because the dictionary value is a strong reference:

```text
cache ──▶ dict ──▶ Image object
```

This may be desirable.

A cache often exists specifically to keep objects available.

But sometimes a cache should not own objects.

You may want:

```text
if another part of the program still uses the object, let the cache find it
if no one else uses the object, let it disappear
```

That is where weak dictionaries help.

---

# Weak Value Dictionaries

`weakref.WeakValueDictionary` stores weak references to its values.

Example:

```python
import weakref


class Image:
    pass


cache = weakref.WeakValueDictionary()

image = Image()
cache["logo"] = image

print("logo" in cache)

image = None

print("logo" in cache)
```

The first check is likely:

```text
True
```

After `image = None`, if no other strong references to the object remain, the image can be collected.

Then the weak dictionary entry can disappear.

The second check may be:

```text
False
```

Conceptually:

```text
cache key "logo" -> weak reference -> Image object
```

The cache knows about the image only while the image is alive somewhere else.

Use `WeakValueDictionary` when:

```text
keys should find objects
but the dictionary should not keep those objects alive
```

---

# Weak Value Dictionary Example

Imagine an object registry:

```python
import weakref


class Document:
    def __init__(self, title):
        self.title = title


documents_by_id = weakref.WeakValueDictionary()


def remember(document):
    documents_by_id[id(document)] = document
    return id(document)


doc = Document("Memory")
doc_id = remember(doc)

print(documents_by_id[doc_id].title)
```

The registry can find the document while it is alive.

Now:

```python
doc = None
```

If no other strong references exist, the document may disappear.

Then its registry entry can disappear too.

This avoids a common problem:

```text
registry accidentally keeps every object alive forever
```

Weak registries are useful when the registry should observe objects, not own them.

---

# Weak Key Dictionaries

`weakref.WeakKeyDictionary` stores weak references to its keys.

Its values are ordinary strong values.

Example:

```python
import weakref


class User:
    pass


metadata = weakref.WeakKeyDictionary()

user = User()
metadata[user] = {"last_seen": "today"}

print(user in metadata)

user = None
```

When the `User` object is no longer strongly referenced elsewhere, the key can disappear from the dictionary.

Then the associated metadata entry disappears too.

Use `WeakKeyDictionary` when:

```text
I want to attach extra information to objects
without making that extra information keep the objects alive
```

This is useful for frameworks, libraries, and tools that need to associate data with objects they do not own.

---

# Weak Key Dictionary Example

Suppose you are building a validation system.

You want to remember validation results for objects.

But you do not want the validation cache to keep those objects alive forever.

Example:

```python
import weakref


class Form:
    pass


validation_cache = weakref.WeakKeyDictionary()


def validate(form):
    if form not in validation_cache:
        validation_cache[form] = {"valid": True}
    return validation_cache[form]


form = Form()
print(validate(form))
```

The cache uses the form object as the key.

If the form disappears elsewhere, the cache entry can disappear too.

This is different from a normal dictionary:

```python
normal_cache = {}
normal_cache[form] = {"valid": True}
```

The normal dictionary would keep `form` alive as a key.

The weak key dictionary does not.

---

# Weak Sets

`weakref.WeakSet` stores weak references to its elements.

Example:

```python
import weakref


class Connection:
    pass


active_connections = weakref.WeakSet()

connection = Connection()
active_connections.add(connection)

print(len(active_connections))

connection = None
```

When no strong references to the connection remain, it can disappear from the weak set.

Use `WeakSet` when:

```text
I want to know which objects are currently alive,
but I do not want this set to keep them alive
```

Common use cases:

* Tracking live instances.
* Observer lists.
* Registries.
* Debugging tools.
* Framework bookkeeping.

Weak sets are especially useful when membership should follow object lifetime automatically.

---

# Tracking Live Instances

Here is a common pattern:

```python
import weakref


class Window:
    live_windows = weakref.WeakSet()

    def __init__(self, title):
        self.title = title
        Window.live_windows.add(self)
```

Now:

```python
main = Window("Main")
settings = Window("Settings")

print(len(Window.live_windows))
```

The weak set tracks the windows.

But it does not own them.

If:

```python
settings = None
```

and no other strong references exist, the settings window can be collected.

The weak set entry can disappear.

This is cleaner than a normal set:

```python
live_windows = set()
```

A normal set would keep every window alive until explicitly removed.

That can create memory growth if removal is forgotten.

---

# Parent Links

Weak references are often useful for parent links.

Consider a tree:

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.children = []
        self.parent = None
```

If parent and child strongly reference each other:

```python
parent.children.append(child)
child.parent = parent
```

Graph:

```text
parent ──▶ parent node ──▶ children list ──▶ child node
   ▲                                             │
   │                                             ▼
   └──────────────── parent attribute ◀─────────┘
```

This creates a cycle.

Cycles are not automatically bad.

The garbage collector can handle many unreachable cycles.

But sometimes the ownership model is:

```text
parent owns child
child only observes parent
```

In that case, a weak parent reference can express the relationship better.

The child can ask for its parent if the parent still exists.

But the child does not keep the parent alive.

---

# Parent Link Example

Using a weak parent link:

```python
import weakref


class Node:
    def __init__(self, value):
        self.value = value
        self.children = []
        self._parent_ref = None

    @property
    def parent(self):
        if self._parent_ref is None:
            return None
        return self._parent_ref()

    def add_child(self, child):
        child._parent_ref = weakref.ref(self)
        self.children.append(child)
```

Use it:

```python
root = Node("root")
leaf = Node("leaf")

root.add_child(leaf)

print(leaf.parent is root)
```

Output:

```text
True
```

The child can find the parent.

But the child does not keep the parent alive by itself.

Design meaning:

```text
root owns leaf through children
leaf has a non-owning link back to root
```

This is a clearer ownership model than a strong two-way relationship when the parent should control lifetime.

---

# Observer Systems

Observer systems often accidentally keep objects alive.

Example:

```python
callbacks = []

class Screen:
    def __init__(self):
        callbacks.append(self.refresh)

    def refresh(self):
        print("refresh")
```

The bound method `self.refresh` refers to `self`.

The global `callbacks` list stores the bound method.

Graph:

```text
callbacks ──▶ bound method ──▶ Screen instance
```

The screen may remain alive even if the rest of the program no longer uses it.

This is not garbage from Python's perspective.

It is reachable through `callbacks`.

Possible solutions:

* Explicitly unregister callbacks.
* Store weak references to observers.
* Use a weak method helper for bound methods.

Weak references do not remove the need for design.

They give you another way to express non-ownership.

---

# Bound Methods Are Special

A bound method is created when you access a method through an instance.

Example:

```python
screen.refresh
```

This produces a method object that combines:

* The function defined on the class.
* The instance to use as `self`.

Bound method objects are temporary.

This makes plain weak references to bound methods surprising.

Python provides `weakref.WeakMethod` for this case.

Example:

```python
import weakref


class Screen:
    def refresh(self):
        print("refresh")


screen = Screen()
method_ref = weakref.WeakMethod(screen.refresh)
```

To call it:

```python
method = method_ref()
if method is not None:
    method()
```

`WeakMethod` can recreate the bound method while the instance and function are still alive.

Use it when you need weak references to bound methods.

---

# Weak References and Callbacks

A weak reference can have a callback.

The callback runs when the referent is about to be finalized.

Example:

```python
import weakref


class Item:
    pass


def removed(ref):
    print("item is gone")


item = Item()
item_ref = weakref.ref(item, removed)

item = None
```

The callback receives the weak reference object.

It does not receive the referent.

Why?

Because the referent is being finalized.

You should not try to recover it from the callback.

The callback is for notification, cleanup of external bookkeeping, or logging.

It is not a way to keep using the dying object.

---

# Callback Caution

Weak-reference callbacks are advanced.

They run during object finalization.

That means they have some of the same cautions as finalizers:

* Timing may be surprising.
* Exceptions cannot be handled by a normal caller.
* The referent is no longer available.
* The callback should be small and defensive.
* The callback should not accidentally keep the referent alive.

Bad idea:

```python
ref = weakref.ref(obj, lambda ref: obj.cleanup())
```

This callback closes over `obj`.

That creates a strong reference to `obj`.

Now the weak reference design is broken.

The callback itself may keep the object alive.

Better:

```python
def cleanup(ref):
    print("object removed")
```

The callback should not own the referent.

---

# `weakref.finalize`

Python provides `weakref.finalize()` for registering cleanup behavior tied to object collection.

Example:

```python
import weakref


class Resource:
    pass


def cleanup(name):
    print("cleaning", name)


resource = Resource()
finalizer = weakref.finalize(resource, cleanup, "temporary resource")
```

When `resource` is collected, the finalizer can call:

```python
cleanup("temporary resource")
```

`finalize()` is often easier and safer than writing a raw weak-reference callback.

It is also often preferable to `__del__` for certain cleanup patterns.

But it still requires care.

The cleanup function and arguments must not keep the object alive.

Bad:

```python
weakref.finalize(resource, resource.close)
```

The bound method `resource.close` refers to `resource`.

That can keep the resource alive.

Better:

```python
weakref.finalize(resource, cleanup_external_handle, handle)
```

Pass only the external data needed for cleanup, not the object itself.

---

# `finalize` Is Not a Replacement for `with`

`weakref.finalize()` can be useful.

But it is not the first choice for ordinary resource management.

If you need predictable cleanup, use a context manager.

Best pattern:

```python
with open("notes.txt") as file:
    text = file.read()
```

This closes the file at a clear point.

Finalizers are useful for last-chance cleanup or library-level safety nets.

They are not as clear as:

```python
try:
    use(resource)
finally:
    resource.close()
```

Rule:

```text
with / finally -> deterministic cleanup
finalize       -> cleanup when object is collected
```

If correctness depends on cleanup happening now, do not rely on collection timing.

---

# Objects That Support Weak References

Not every object can be weakly referenced.

This works for many user-defined class instances:

```python
import weakref


class User:
    pass


user = User()
ref = weakref.ref(user)
```

But this fails:

```python
import weakref

items = []
ref = weakref.ref(items)
```

You get:

```text
TypeError
```

Built-in `list` objects do not directly support weak references.

Similarly, many simple built-in objects do not support weak references.

Examples commonly include:

* `int`
* `tuple`
* `list`
* `dict`
* many strings

The exact support rules are implementation details, but the practical rule is:

```text
user-defined objects usually support weak references
many built-in value/container objects do not directly support them
```

---

# Subclassing Built-ins

Some built-in types that do not directly support weak references can support them when subclassed.

Example:

```python
import weakref


class WeakList(list):
    pass


items = WeakList([1, 2, 3])
ref = weakref.ref(items)
```

Now the weak reference can work.

Similarly:

```python
class WeakDict(dict):
    pass
```

Instances of the subclass can support weak references.

But not every built-in can be made weak-referenceable this way.

For example, CPython documents that some built-in types such as `tuple` and `int` do not support weak references even when subclassed.

Do not rely on guesswork.

If weak-reference support matters, test it or check the documentation for the type.

---

# `__slots__` and Weak References

When classes use `__slots__`, weak-reference support needs special attention.

We will study `__slots__` later in the book.

For now, understand the basic issue.

This class usually supports weak references:

```python
class User:
    pass
```

This class uses slots:

```python
class User:
    __slots__ = ("name",)

    def __init__(self, name):
        self.name = name
```

Instances may not support weak references unless `__weakref__` is included:

```python
class User:
    __slots__ = ("name", "__weakref__")

    def __init__(self, name):
        self.name = name
```

Now instances can be weakly referenced.

This matters for memory-optimized classes.

If you use slots and also need weak references, remember:

```text
include "__weakref__" in __slots__
```

We will return to this when we study `__slots__` in advanced Python.

---

# Weak References Do Not Break All Cycles Automatically

Weak references can help avoid cycles, but they do not magically fix every memory design.

Example with a strong cycle:

```python
parent.children.append(child)
child.parent = parent
```

If `child.parent` becomes a weak reference, the parent link no longer keeps the parent alive.

That can avoid one ownership cycle.

But your program may still have other strong references:

```python
registry.append(parent)
callbacks.append(child.handle)
cache[parent.id] = parent
```

Those references still matter.

Weak references only affect the relationship where you use them.

They are not a global memory cleanup command.

Always draw the object graph.

Ask:

```text
Which arrows are strong?
Which arrows are weak?
Which strong paths still reach this object?
```

That is the correct model.

---

# Weak References and Garbage Collection

Weak references interact with garbage collection.

When an object is only weakly reachable, it can be collected.

Example:

```text
weak cache - - -> object
```

If there are no strong references:

```text
object can be reclaimed
weak cache entry can disappear
```

In a weak value dictionary:

```python
cache = weakref.WeakValueDictionary()
cache["item"] = obj
```

The dictionary does not strongly own `obj`.

When `obj` is collected, the entry can be removed.

This is why weak containers are useful.

They cooperate with object lifecycle.

They let object lifetime be controlled by strong owners elsewhere.

---

# Weak References and Reference Counting

Weak references do not increase the referent's strong reference count in the ownership sense.

Example:

```python
import weakref


class Item:
    pass


item = Item()
ref = weakref.ref(item)
```

The object is alive because of `item`, not because of `ref`.

If:

```python
item = None
```

the weak reference does not keep the object alive.

This fits the reference-counting model:

```text
strong references keep objects alive
weak references observe without ownership
```

Do not use `sys.getrefcount()` experiments to over-interpret weak references.

Inspection itself can create temporary references.

The conceptual model is enough:

```text
weak references are not strong ownership references
```

---

# Weak References and Reachability

Garbage collection is about reachability through strong references.

Weak references do not create the same kind of reachability.

Example:

```python
import weakref


class Item:
    pass


item = Item()
item_ref = weakref.ref(item)

item = None
```

After `item = None`, the object is not considered reachable just because `item_ref` exists.

The weak reference object exists.

The referent may not.

This distinction is the whole point.

Strong path:

```text
root ──▶ object
```

Weak path:

```text
root ──▶ weak reference - - -> object
```

Only the strong path keeps the object alive.

---

# Weak References and Identity

Weak references are often used with object identity.

Example:

```python
import weakref


class Entity:
    pass


entity = Entity()
ref = weakref.ref(entity)
```

While the object is alive:

```python
print(ref() is entity)
```

Output:

```text
True
```

The weak reference retrieves the same object.

It does not create a copy.

It does not rebuild the object.

It either returns the living object or returns `None`.

Model:

```text
weak reference does not own
weak reference does not copy
weak reference does not recreate
weak reference only retrieves if alive
```

This makes weak references useful for registries that track objects by identity.

---

# Weak References and Equality

Weak-reference behavior around equality can be subtle.

At a beginner level, do not build complex equality logic around raw weak references.

Use them to retrieve objects:

```python
obj = ref()
if obj is not None:
    ...
```

Weak dictionaries and weak sets handle most common cases for you.

If you are tempted to compare weak reference objects directly, ask whether you should instead compare the live referents:

```python
left = left_ref()
right = right_ref()

if left is not None and right is not None:
    print(left == right)
```

For advanced uses, read the documentation carefully.

For this book's current level, the practical rule is:

```text
call the weak reference, check for None, then use the object
```

---

# Weak Containers Are Usually Better Than Raw Weak References

Most code should not need many raw `weakref.ref()` objects.

The standard weak containers are often clearer:

```text
WeakValueDictionary -> weak values
WeakKeyDictionary   -> weak keys
WeakSet             -> weak elements
WeakMethod          -> weak bound methods
```

Use raw weak references when you have a specific one-to-one non-owning relationship.

Use weak containers when you are building:

* Caches.
* Registries.
* Metadata maps.
* Live instance sets.
* Observer collections.

The container names document intent.

Example:

```python
cache = weakref.WeakValueDictionary()
```

This clearly says:

```text
the cache should not own values
```

That is better than hiding raw weak references inside a normal dictionary unless you need custom behavior.

---

# Normal Dictionary of Weak References

Sometimes you may see:

```python
refs = {}
refs["main"] = weakref.ref(obj)
```

This works.

But every lookup must handle the weak reference manually:

```python
ref = refs["main"]
obj = ref()

if obj is None:
    del refs["main"]
else:
    obj.use()
```

A `WeakValueDictionary` can do this housekeeping for many cases.

Instead:

```python
refs = weakref.WeakValueDictionary()
refs["main"] = obj
```

Now the mapping can remove entries when values disappear.

Use the higher-level weak containers unless raw weak references express something more precise.

---

# Weak Reference Proxies

The `weakref` module also provides proxy objects.

A proxy lets code access the referent more directly.

Example idea:

```python
proxy = weakref.proxy(obj)
```

Then:

```python
proxy.some_method()
```

can behave like using `obj.some_method()`.

But if the referent has been collected, using the proxy raises `ReferenceError`.

This can be convenient, but it can also hide the fact that the object may disappear.

For learning and most ordinary code, explicit weak references are clearer:

```python
obj = ref()
if obj is not None:
    obj.some_method()
```

That pattern forces you to handle the missing-object case.

Proxies are useful in advanced designs, but they are not the first weak-reference tool to reach for.

---

# Weak References in Real Design

Weak references are most valuable when ownership is clear.

Example ownership model:

```text
Application owns windows.
Windows own widgets.
Widgets may know their window weakly.
Global registry observes windows weakly.
```

This could become:

```text
app ──▶ windows list ──▶ Window ──▶ Widget
                         ▲          │
                         └ - - - - -┘ weak parent link

registry - - -> Window
```

Now lifetime is clear:

* The application strongly owns windows.
* Windows strongly own widgets.
* Widgets do not keep windows alive.
* Registry does not keep windows alive.

This is the kind of design weak references support.

They are not about avoiding thought.

They are about making thought visible in the object graph.

---

# When Weak References Are the Wrong Tool

Do not use weak references when the object must stay alive.

Example:

```python
class OrderProcessor:
    def __init__(self, database):
        self.database_ref = weakref.ref(database)
```

If the processor requires the database to function, a weak reference may be wrong.

The database could disappear.

Then every operation needs missing-object handling.

If `OrderProcessor` depends on `database`, use a strong reference:

```python
class OrderProcessor:
    def __init__(self, database):
        self.database = database
```

Weak references are for non-owning relationships.

Dependencies are often owning or at least lifetime-extending relationships.

Do not weaken a reference just to avoid thinking about lifecycle.

Clarify ownership first.

---

# When Explicit Removal Is Better

Sometimes explicit removal is better than weak references.

Example:

```python
listeners = []

def add_listener(listener):
    listeners.append(listener)


def remove_listener(listener):
    listeners.remove(listener)
```

This is clear.

The program controls listener registration.

If listeners must remain active until explicitly removed, a normal list is appropriate.

Weak references would change the behavior:

```text
listener may disappear if no one else strongly references it
```

That may surprise users.

Use weak references only when disappearing automatically is correct.

Sometimes the right fix is not weak references.

Sometimes the right fix is:

```text
register explicitly
unregister explicitly
document ownership clearly
```

---

# Weak References and Testing

Testing weak-reference behavior can be tricky.

Example:

```python
obj = ref()
```

This line creates a temporary strong reference to the object.

As long as `obj` exists, the object is alive.

Tests can accidentally keep objects alive through:

* Local variables.
* Assertion helpers.
* Tracebacks from failed assertions.
* Debug print variables.
* Test framework internals.
* Interactive sessions.

When testing weak containers, use small helper functions to limit local references:

```python
def create(cache):
    item = Item()
    cache["x"] = item


cache = weakref.WeakValueDictionary()
create(cache)
```

After `create` returns, the local name `item` is gone.

Then the weak dictionary may remove the entry.

If needed in experiments, `gc.collect()` can force a collection attempt, but exact timing and counts remain implementation details.

---

# A Complete Cache Example

Suppose loading images is expensive.

You want a cache, but you do not want the cache to keep images alive forever.

Example:

```python
import weakref


class Image:
    def __init__(self, path):
        self.path = path


image_cache = weakref.WeakValueDictionary()


def load_image(path):
    image = image_cache.get(path)

    if image is None:
        image = Image(path)
        image_cache[path] = image

    return image
```

Use it:

```python
logo = load_image("logo.png")
again = load_image("logo.png")

print(logo is again)
```

Output:

```text
True
```

As long as `logo` or `again` exists, the image is alive.

If all strong references disappear:

```python
logo = None
again = None
```

the cache does not keep the image alive by itself.

Later:

```python
new_logo = load_image("logo.png")
```

may create a new image object.

This cache improves reuse without becoming the final owner of every image.

---

# A Complete Metadata Example

Suppose you want to store metadata about objects without modifying the objects themselves.

Example:

```python
import weakref


class Model:
    pass


metadata = weakref.WeakKeyDictionary()


def set_label(obj, label):
    metadata[obj] = label


def get_label(obj):
    return metadata.get(obj)
```

Use it:

```python
model = Model()
set_label(model, "important")

print(get_label(model))
```

The metadata dictionary does not keep `model` alive.

When `model` disappears, the metadata can disappear too.

This is useful when:

* You do not own the object's class.
* You do not want to add attributes.
* You want metadata lifetime to follow object lifetime.

This pattern appears in frameworks and tooling.

---

# A Complete Observer Example

Here is a simple weak observer collection for objects with an `update` method.

```python
import weakref


class EventSource:
    def __init__(self):
        self._observers = weakref.WeakSet()

    def add_observer(self, observer):
        self._observers.add(observer)

    def notify(self):
        for observer in list(self._observers):
            observer.update()
```

Observers:

```python
class Screen:
    def update(self):
        print("screen updated")
```

Use it:

```python
source = EventSource()
screen = Screen()

source.add_observer(screen)
source.notify()
```

The source does not keep the screen alive.

If:

```python
screen = None
```

and no other strong references exist, the screen can disappear from the weak set.

This is right only if observer lifetime is owned elsewhere.

If the event source should own observers, use strong references instead.

---

# A Weak Method Observer Sketch

If you want to store bound methods directly, use `WeakMethod`.

Sketch:

```python
import weakref


class EventSource:
    def __init__(self):
        self._callbacks = []

    def add_callback(self, callback):
        self._callbacks.append(weakref.WeakMethod(callback))

    def notify(self):
        alive = []

        for callback_ref in self._callbacks:
            callback = callback_ref()
            if callback is not None:
                callback()
                alive.append(callback_ref)

        self._callbacks = alive
```

This stores weak references to bound methods.

Dead callbacks are removed during notification.

This is more complex than a `WeakSet` of observer objects.

Prefer the simpler design when possible.

Use `WeakMethod` when method-level registration is really what you need.

---

# Common Mistake: Weak Reference to a Temporary Object

This does not work the way beginners expect:

```python
import weakref


class Item:
    pass


ref = weakref.ref(Item())
print(ref())
```

The `Item()` object has no strong reference after the expression ends.

The weak reference does not keep it alive.

So `ref()` may immediately return `None`.

Correct:

```python
item = Item()
ref = weakref.ref(item)

print(ref())
```

Now `item` keeps the object alive.

Weak references need some other strong owner to be useful.

Without a strong owner, the object can disappear immediately.

---

# Common Mistake: Weakening Required Dependencies

Suppose:

```python
class Service:
    def __init__(self, database):
        self.database_ref = weakref.ref(database)
```

If `Service` cannot function without `database`, this is fragile.

Every use becomes:

```python
database = self.database_ref()
if database is None:
    raise RuntimeError("database is gone")
```

That is usually a design smell.

Use a strong reference for required dependencies:

```python
class Service:
    def __init__(self, database):
        self.database = database
```

Weak references are not for things you need to stay alive.

They are for things someone else owns.

---

# Common Mistake: Forgetting to Handle `None`

A weak reference can return `None`.

This is wrong:

```python
ref().run()
```

If the referent is gone, this becomes:

```python
None.run()
```

which raises:

```text
AttributeError
```

Correct:

```python
obj = ref()
if obj is not None:
    obj.run()
```

or:

```python
obj = ref()
if obj is None:
    return

obj.run()
```

Every weak-reference use must consider the object-gone case.

That is the price of non-ownership.

---

# Common Mistake: Callback Captures the Referent

This is a subtle bug:

```python
import weakref


class Item:
    def cleanup(self):
        print("cleanup")


item = Item()
ref = weakref.ref(item, lambda ref: item.cleanup())
```

The lambda closes over `item`.

That closure can keep the `Item` object alive.

The weak reference no longer behaves as intended.

Better:

```python
def cleanup(ref):
    print("item removed")


ref = weakref.ref(item, cleanup)
```

If cleanup needs external information, capture only that information:

```python
name = "temporary item"
ref = weakref.ref(item, lambda ref: print(name, "removed"))
```

Do not capture the referent itself.

---

# Common Mistake: Confusing Weak Containers With Normal Containers

A weak container can lose entries automatically.

Example:

```python
cache = weakref.WeakValueDictionary()
cache["x"] = obj
```

If `obj` is not strongly referenced elsewhere, `"x"` can disappear from the cache.

That means this may fail:

```python
cache["x"]
```

with:

```text
KeyError
```

Use safer access when appropriate:

```python
obj = cache.get("x")
if obj is None:
    obj = create_object()
```

Weak containers are dynamic.

Their contents follow object lifetime.

That is their purpose.

---

# Common Mistake: Expecting Weak References for All Types

This fails:

```python
weakref.ref(10)
```

This also fails:

```python
weakref.ref([])
```

Many built-in objects do not support weak references directly.

If you need weak-reference behavior, use a wrapper object:

```python
class Box:
    def __init__(self, value):
        self.value = value
```

Then:

```python
box = Box([1, 2, 3])
ref = weakref.ref(box)
```

The weak reference points to the box, not directly to the list.

This is often a better design anyway because it gives the object a clear identity.

---

# Common Mistake: Solving Every Cycle With Weak References

Weak references are not required for every cycle.

Python's cyclic garbage collector can handle many ordinary unreachable cycles.

Example:

```python
a = []
b = []
a.append(b)
b.append(a)
```

When this group becomes unreachable, the garbage collector can collect it.

You do not need weak references just because a cycle exists.

Use weak references when they express ownership:

```text
parent owns child
child observes parent
```

Do not use them merely because:

```text
cycle scary
```

Cycles are normal in object models.

Ownership clarity is the real issue.

---

# Design Questions

Before using a weak reference, ask:

```text
Who owns the object?
Who is allowed to keep it alive?
Can this reference become None?
What should happen if the object disappears?
Would explicit unregistering be clearer?
Would a weak container express this better?
Does the object type support weak references?
Will callbacks accidentally capture the object?
Is this a resource cleanup problem instead?
```

These questions prevent misuse.

A weak reference is not a memory-management shortcut.

It is a design statement:

```text
this relationship is non-owning
```

Use it when that statement is true.

---

# Relationship to Part VIII

This part began with stack and heap.

Then we learned that names and stack frames refer to heap objects.

Reference counting showed:

```text
strong references keep objects alive
```

Garbage collection showed:

```text
unreachable cycles can be collected
```

Object lifecycle showed:

```text
objects move through creation, use, reachability, cleanup
```

Weak references now add:

```text
some relationships can observe objects without owning them
```

Together:

```text
strong references define lifetime
weak references observe lifetime
garbage collection cleans unreachable cycles
context managers clean external resources predictably
```

That is a complete beginner-to-intermediate memory model for Python.

---

# Exercises

1. Explain the difference between:

```text
strong reference
```

and:

```text
weak reference
```

Use an example involving a user-defined class.

---

2. Create a weak reference:

```python
import weakref


class Item:
    pass


item = Item()
ref = weakref.ref(item)
```

Call `ref()` while `item` exists.

Then set:

```python
item = None
```

Call `ref()` again.

Explain the result.

---

3. Why is this code usually wrong?

```python
ref().process()
```

Rewrite it safely.

---

4. Explain why a normal dictionary cache can keep objects alive:

```python
cache[key] = obj
```

Then explain how `WeakValueDictionary` changes the ownership relationship.

---

5. Build a small `WeakValueDictionary` example with a class named `Image`.

Store an image in the cache.

Remove the strong reference.

Explain why the cache entry may disappear.

---

6. Build a `WeakKeyDictionary` example where metadata is attached to an object.

Explain why the metadata should disappear when the object disappears.

---

7. Explain why a weak parent link can be useful in a tree.

Use these terms:

```text
owning relationship
non-owning relationship
strong reference
weak reference
```

---

8. Why does this fail?

```python
weakref.ref([])
```

What is one design alternative?

---

9. Explain why this callback is dangerous:

```python
ref = weakref.ref(obj, lambda ref: obj.cleanup())
```

What reference is hidden inside the callback?

---

10. Decide whether weak references are appropriate:

```text
A service depends on a database connection and cannot work without it.
```

Should the service store a strong or weak reference?

Explain why.

---

# Summary

In this chapter we learned:

* Normal Python references are strong references.
* Strong references keep objects alive.
* Weak references do not keep objects alive.
* The object pointed to by a weak reference is called the referent.
* Calling a weak reference returns the object if it is alive.
* Calling a weak reference returns `None` if the object is gone.
* Weak references are useful for non-owning relationships.
* Weak references are commonly used in caches, registries, metadata maps, parent links, and observer systems.
* `WeakValueDictionary` stores weak values.
* `WeakKeyDictionary` stores weak keys.
* `WeakSet` stores weak elements.
* `WeakMethod` handles weak references to bound methods.
* `weakref.finalize()` can register cleanup tied to object collection, but it does not replace context managers.
* Not every object supports weak references.
* Classes using `__slots__` need `__weakref__` if instances should support weak references.
* Weak references should be used to express ownership, not as a general memory shortcut.

Core model:

```text
strong reference:
    points to object
    keeps object alive

weak reference:
    points to object if it is still alive
    does not keep object alive
    returns None after object is gone
```

Ownership model:

```text
owning relationship     -> strong reference
non-owning relationship -> weak reference
external resource       -> context manager or explicit cleanup
```

Weak references complete our memory-management foundation.

They let us design object graphs where lifetime is intentional instead of accidental.

---

# Preview of Chapter 38

Next we begin Part IX: Modules and Imports.

So far, most examples have lived in one file or one small snippet.

Real Python programs are split across files.

Chapter 38 introduces modules.

We will study:

* What a module is.
* Why Python files become modules.
* How names live inside module namespaces.
* What happens when code imports a module.
* Why modules help organize larger programs.
* How module-level code differs from function-level code.
* Why imports are execution, not just textual inclusion.

The transition is important:

```text
memory management explains how objects live
modules explain where program names live at file scale
```

Part IX moves us from object lifetime to program organization.
