# Chapter 33 — Stack vs Heap

---

# Learning Objectives

By the end of this chapter, you should understand:

* What "stack" means in program execution.
* What "heap" means in memory management.
* Why Python stack vs heap explanations must be handled carefully.
* How Python function calls create frames.
* How frames contain local names and references.
* Where Python objects conceptually live.
* Why names are not boxes containing objects.
* Why objects can outlive function calls.
* How references connect frames to objects.
* How lists, dictionaries, and custom structures store references.
* How stack frames differ from object memory.
* How recursion grows the call stack.
* Why object lifetime is not the same as variable scope.
* How this prepares you for reference counting and garbage collection.

This chapter begins the memory management part.

The goal is not to teach low-level C memory allocation.

The goal is to build the correct Python mental model.

---

# Concept Overview

In many programming discussions, memory is divided into two broad ideas:

```text
stack -> active function calls and local execution state
heap  -> objects that live beyond a single stack operation
```

That model is useful, but it can mislead Python learners if taught too literally.

In Python, you should think like this:

```text
function call -> frame
frame -> local names
local names -> references
references -> objects
objects -> managed by Python
```

Example:

```python
def make_list():
    values = [1, 2, 3]
    return values

result = make_list()
```

During the function call:

```text
make_list frame
└── values ─────▶ list object [1, 2, 3]
```

After the function returns:

```text
module frame
└── result ─────▶ same list object [1, 2, 3]
```

The function frame is gone.

The list object still exists.

Why?

Because `result` now refers to it.

This is the core distinction:

```text
frames come and go with calls
objects live as long as references keep them alive
```

---

# Why Stack vs Heap Matters

Stack vs heap matters because it explains:

* why local names disappear after a function returns
* why returned objects can still exist
* why mutation inside a function can affect caller-visible objects
* why recursion can hit a recursion limit
* why object lifetime depends on references
* why memory is not automatically freed just because a name went out of scope

Without this model, Python behavior feels inconsistent.

Example:

```python
def add_item(items):
    items.append("new")

values = []
add_item(values)

print(values)
```

Output:

```text
['new']
```

Why did the caller's list change?

Because both `values` and the parameter `items` referred to the same list object.

The function frame had its own local name.

It did not have its own copy of the list.

---

# The Stack

The stack is the structure of active function calls.

When a function is called, Python creates a frame.

That frame is pushed onto the call stack.

When the function returns, the frame is popped.

Example:

```python
def first():
    second()

def second():
    third()

def third():
    print("inside third")

first()
```

While `third()` is running:

```text
top -> third frame
       second frame
       first frame
bottom -> module frame
```

The stack answers:

```text
Which function is currently running?
Who called it?
Where should execution return?
```

This is the same call stack from Chapter 23.

Here we connect it to memory.

---

# Frames

A frame is the runtime record for one active function call.

A frame contains:

* local names
* parameter bindings
* current execution position
* references to global/built-in namespaces
* temporary evaluation state

Example:

```python
def total(a, b):
    result = a + b
    return result

answer = total(2, 3)
```

During the call:

```text
total frame
├── a ───────▶ int object 2
├── b ───────▶ int object 3
└── result ─▶ int object 5
```

Important:

The frame stores references.

It does not store complete Python objects inside local variable boxes.

That is the mental model that keeps Python behavior coherent.

---

# The Heap

The heap is the broad area of memory where objects live.

In Python, you can think:

```text
objects live in Python-managed memory
```

Many explanations call this "the heap."

Example:

```python
name = "Ada"
numbers = [1, 2, 3]
settings = {"debug": True}
```

Conceptually:

```text
module frame
├── name ─────▶ str object "Ada"
├── numbers ─▶ list object [1, 2, 3]
└── settings ▶ dict object {"debug": True}
```

The frame contains names and references.

The objects live separately in managed memory.

This is why multiple names can refer to the same object:

```python
a = [1, 2]
b = a
```

```text
a ─────┐
       v
b ─▶ list object [1, 2]
```

There is one object.

There are two references.

---

# Python Is Not C

In C, stack vs heap often means:

```text
stack allocation vs manual heap allocation
```

In Python, you do not manually allocate and free objects.

You do not write:

```text
malloc
free
```

Python manages objects for you.

That means C-style rules such as:

```text
local variables are on the stack
objects are copied into variables
```

are not the right mental model for Python.

Better Python model:

```text
names live in namespaces
frames hold local namespaces
names refer to objects
objects live in managed memory
references determine lifetime
```

This model explains Python code better than literal stack/heap diagrams copied from lower-level languages.

---

# Names Are Not Boxes

A common beginner model is:

```text
variable box contains value
```

This is misleading in Python.

Example:

```python
a = [1, 2]
b = a
```

Box model might suggest:

```text
a contains a list
b contains a copied list
```

But Python behaves like:

```text
a ─────┐
       v
b ─▶ list object [1, 2]
```

Mutation proves it:

```python
b.append(3)

print(a)
```

Output:

```text
[1, 2, 3]
```

Names are references to objects.

They are not boxes containing independent values.

---

# Local Names Disappear, Objects May Survive

Consider:

```python
def create_user():
    user = {"name": "Ada"}
    return user

account = create_user()
```

During the call:

```text
create_user frame
└── user ─────▶ dict object {"name": "Ada"}
```

After return:

```text
module frame
└── account ─▶ dict object {"name": "Ada"}
```

The local name `user` disappeared when the frame ended.

The dictionary object survived because the return value was bound to `account`.

This is one of the most important memory ideas:

```text
scope controls where names are visible
references control whether objects stay alive
```

Scope and lifetime are related, but not identical.

---

# Returning Objects

When a function returns an object, it returns a reference to that object.

Example:

```python
def make_numbers():
    numbers = [1, 2, 3]
    return numbers

result = make_numbers()
```

No list is copied just because it is returned.

Conceptually:

```text
inside function:
numbers ─▶ list object

after return:
result ─▶ same list object
```

The object continues to exist because something still refers to it.

If the return value is ignored:

```python
make_numbers()
```

then the list may become unreachable immediately after the function returns.

That prepares us for reference counting.

---

# Passing Objects To Functions

When you pass an object to a function, Python passes a reference.

Example:

```python
def mutate(items):
    items.append("x")

values = []
mutate(values)

print(values)
```

Output:

```text
['x']
```

During the call:

```text
module frame
└── values ─▶ list object []

mutate frame
└── items ──▶ same list object []
```

The function mutates the shared list object.

Both frames refer to the same object while the function is active.

After the function returns, the `items` name disappears.

The object remains because `values` still refers to it.

---

# Rebinding A Parameter

Rebinding a parameter does not rebind the caller's name.

Example:

```python
def replace(items):
    items = ["new"]

values = ["old"]
replace(values)

print(values)
```

Output:

```text
['old']
```

During the call initially:

```text
values ─▶ list object ["old"]
items  ─▶ same list object ["old"]
```

After rebinding inside the function:

```text
values ─▶ list object ["old"]
items  ─▶ list object ["new"]
```

The local name `items` now refers to a different object.

The caller's name `values` is unchanged.

This is not stack vs heap magic.

It is name rebinding.

---

# Mutating A Shared Object

Mutation changes the object.

Example:

```python
def clear_items(items):
    items.clear()

values = ["a", "b"]
clear_items(values)

print(values)
```

Output:

```text
[]
```

The name `items` and the name `values` referred to the same list.

The list object was mutated.

The local name disappeared after return.

The mutated object remained.

This pattern is common with:

* lists
* dictionaries
* sets
* custom data structures

Understanding frames and object memory makes it predictable.

---

# Containers Store References

Lists, tuples, dictionaries, sets, and custom structures store references.

Example:

```python
inner = [1, 2]
outer = [inner]

inner.append(3)

print(outer)
```

Output:

```text
[[1, 2, 3]]
```

Why?

```text
inner ─────▶ list object [1, 2, 3]

outer ─────▶ list object
              └── index 0 ─▶ same inner list
```

The outer list contains a reference to the inner list.

It does not contain a deep copy.

This is why nested mutation matters.

---

# Stack Frames And Containers

A frame may contain a local name that refers to a container.

That container may refer to many other objects.

Example:

```python
def build():
    users = [
        {"name": "Ada"},
        {"name": "Grace"},
    ]
    return users

result = build()
```

During the call:

```text
build frame
└── users ─▶ list object
              ├── dict object {"name": "Ada"}
              └── dict object {"name": "Grace"}
```

After return:

```text
module frame
└── result ─▶ same list object
               ├── same dict object {"name": "Ada"}
               └── same dict object {"name": "Grace"}
```

The frame ended.

The object graph survived.

Why?

Because `result` refers to the list, and the list refers to the dictionaries.

---

# Object Graphs

Objects can refer to other objects.

This creates an object graph.

Example:

```python
user = {
    "name": "Ada",
    "roles": ["admin", "editor"],
}
```

Conceptually:

```text
dict object
├── key "name"  ─▶ value "Ada"
└── key "roles" ─▶ list object
                   ├── "admin"
                   └── "editor"
```

Object graphs are central to memory management.

An object may survive not because a variable directly names it, but because another object refers to it.

Example:

```python
roles = user["roles"]
```

Now both `user` and `roles` provide paths to the same list object.

---

# Object Lifetime

An object's lifetime is the period during which it exists in memory.

In Python, object lifetime is tied to reachability and references.

Example:

```python
def make():
    data = [1, 2, 3]
    return data

x = make()
```

The list survives because `x` refers to it.

Example:

```python
def make():
    data = [1, 2, 3]

make()
```

The list becomes unreachable after the function returns.

Python can clean it up.

The exact cleanup mechanism is the next chapter's topic.

For now:

```text
reachable objects stay alive
unreachable objects can be cleaned up
```

---

# Scope vs Lifetime

Scope answers:

```text
Where can this name be used?
```

Lifetime answers:

```text
How long does this object exist?
```

Example:

```python
def make():
    value = [1, 2, 3]
    return value

result = make()
```

The name `value` has function-local scope.

It cannot be used outside `make()`.

But the list object returned by `make()` still exists.

So:

```text
local name ended
object survived
```

This is why scope and lifetime must not be confused.

---

# Recursion And Stack Growth

Every recursive call creates another frame.

Example:

```python
def countdown(n):
    if n == 0:
        return

    countdown(n - 1)

countdown(3)
```

At the deepest point:

```text
countdown(0) frame
countdown(1) frame
countdown(2) frame
countdown(3) frame
module frame
```

Each frame has its own local `n`.

This consumes stack space.

Too much recursion leads to:

```text
RecursionError
```

This is stack-related.

It is different from heap memory used by objects.

---

# Large Objects And Function Calls

Passing a large list to a function does not copy the list.

Example:

```python
def process(items):
    return len(items)

big = list(range(1_000_000))
print(process(big))
```

The function call passes a reference.

It does not duplicate one million integers into the function frame.

During the call:

```text
module frame
└── big ───▶ large list object

process frame
└── items ─▶ same large list object
```

This is efficient.

But it also means mutation is shared.

If `process()` mutates `items`, the caller sees it.

---

# Copying Changes The Object Graph

Copying creates new container objects.

Example:

```python
original = [[1], [2]]
copy = original.copy()
```

Conceptually:

```text
original ─▶ outer list A
             ├── inner list 1
             └── inner list 2

copy ─────▶ outer list B
             ├── same inner list 1
             └── same inner list 2
```

The outer list is new.

The inner lists are shared.

This is shallow copying.

Deep copying would create new inner objects too.

Deep copying is more expensive and not always desired.

The stack vs heap model helps you see what is copied and what is shared.

---

# Global Names And Object Lifetime

Global names live in a module namespace.

Example:

```python
cache = {}

def store(key, value):
    cache[key] = value
```

The name `cache` is global.

The dictionary object can live for as long as the module remains loaded and something refers to it.

Objects referenced by globals often live longer than objects referenced only by temporary function locals.

This is why global caches can grow memory usage over time.

The object is not temporary just because it was mutated inside a function.

It remains reachable through the global name.

---

# Closures And Object Lifetime

Closures can keep objects alive.

Example:

```python
def make_appender():
    items = []

    def append_item(value):
        items.append(value)
        return list(items)

    return append_item

append = make_appender()
```

The `make_appender` frame ends.

But the inner function keeps access to `items`.

Conceptually:

```text
append ─▶ function object
           └── closure reference ─▶ list object []
```

The local name `items` is gone as a normal frame local.

The list object survives because the closure references it.

This is another example where object lifetime outlasts local scope.

---

# Custom Structures And Memory

Custom data structures are objects that refer to internal storage.

Example:

```python
class Stack:
    def __init__(self):
        self._items = []

stack = Stack()
```

Conceptually:

```text
stack ─▶ Stack object
          └── _items ─▶ list object []
```

When you push:

```python
stack.push("a")
```

the internal list changes:

```text
Stack object
└── _items ─▶ list object ["a"]
```

The stack object and its internal list are separate objects connected by references.

This is why object graphs matter.

---

# Common Misleading Diagrams

Avoid diagrams that imply:

```text
local variable contains object directly
```

For Python, prefer:

```text
local name ─▶ object
```

Avoid saying:

```text
small objects are on the stack
large objects are on the heap
```

That is not the useful Python distinction.

Better:

```text
frames manage execution state
objects live in managed memory
names and containers hold references
references keep objects alive
```

This model will remain valid across:

* lists
* dictionaries
* functions
* closures
* custom objects
* modules
* classes

---

# Common Mistakes

## Misconception 1

### Local variables contain objects directly.

In Python, local names refer to objects.

Frames store local name bindings and references.

## Misconception 2

### Returning a local object is unsafe because the local frame disappears.

Returning an object is safe.

The frame disappears, but the returned object can survive if the caller keeps a reference.

## Misconception 3

### Passing a list to a function copies the list.

It passes a reference to the list object.

Mutation inside the function can affect the caller-visible object.

## Misconception 4

### Reassigning a parameter changes the caller's variable.

Reassigning a parameter only rebinds the local name in the function frame.

## Misconception 5

### Scope and lifetime are the same thing.

Scope is about name visibility.

Lifetime is about object existence.

## Misconception 6

### Recursion mainly consumes heap memory.

Recursion grows the call stack because each recursive call creates a new frame.

The objects created inside recursive calls may also use managed memory, but the recursive call chain itself is stack-related.

## Misconception 7

### Stack vs heap in Python works exactly like C.

Python manages memory at a higher level.

The useful model is frames, references, objects, and reachability.

---

# Real-world Usage

## Avoiding Accidental Mutation

```python
def add_item(items, item):
    updated = items.copy()
    updated.append(item)
    return updated
```

This creates a new outer list rather than mutating the caller's list.

## Understanding Returned Objects

```python
def make_config():
    return {"debug": False}

config = make_config()
```

The dictionary survives because `config` refers to it.

## Understanding Caches

```python
cache = {}
```

Objects stored in a global cache stay reachable through the cache.

## Understanding Closures

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

The returned function keeps the needed state alive.

## Understanding Data Structures

```python
graph = {
    "A": ["B", "C"],
    "B": ["A"],
}
```

The dictionary refers to lists, and the lists refer to strings.

This is an object graph.

---

# Internal Mechanics

At a high level:

```text
function call
    -> create frame
    -> bind local names to object references
    -> execute code
    -> return object reference or raise exception
    -> destroy/deactivate frame
```

Objects are allocated in Python-managed memory.

Names and containers refer to objects.

When a frame disappears, its local references disappear too.

If no other references point to an object, that object may become eligible for cleanup.

The next chapter explains the first major cleanup mechanism:

```text
reference counting
```

For now, keep this model:

```text
frames are temporary execution contexts
objects are managed separately
references connect them
```

---

# Concept Connections

Stack vs heap connects to earlier chapters:

* Names and references: names point to objects.
* Mutability: shared mutable objects can change through any reference.
* Identity and equality: identity tells whether references point to the same object.
* Functions: calls create frames.
* Scope: frames contain local namespaces.
* Closures: objects can outlive their creating frame.
* Recursion: recursive calls stack frames.
* Lists, dictionaries, sets, and tuples: containers store references.
* Custom data structures: objects can own internal storage objects.

This chapter prepares you for:

* Reference counting.
* Garbage collection.
* Object lifecycle.
* Weak references.
* Modules and namespaces.
* Classes and instance attributes.
* CPython internals.

---

# Active Recall

## Easy Recall Questions

1. What is the call stack?
2. What is a frame?
3. What do local names refer to?
4. Where do Python objects conceptually live?
5. What happens to a frame when a function returns?
6. Can an object created inside a function survive after the function returns?
7. Does passing a list to a function copy it?
8. What is the difference between rebinding and mutation?
9. What is an object graph?
10. What is the difference between scope and lifetime?

## Deep Understanding Questions

1. Why is "variables are boxes" misleading in Python?
2. Why can a returned object survive after its local name disappears?
3. Why can mutation inside a function affect an object outside the function?
4. Why does rebinding a parameter not affect the caller's name?
5. How can closures keep objects alive?
6. Why does recursion grow the call stack?
7. Why is Python's stack vs heap model different from C-style explanations?

## Predict-the-Output Questions

### Question 1

```python
def make():
    data = [1, 2]
    return data

result = make()
result.append(3)

print(result)
```

### Question 2

```python
def mutate(items):
    items.append("x")

values = []
mutate(values)

print(values)
```

### Question 3

```python
def replace(items):
    items = ["new"]

values = ["old"]
replace(values)

print(values)
```

### Question 4

```python
inner = [1]
outer = [inner]
inner.append(2)

print(outer)
```

### Question 5

```python
def make_counter():
    count = [0]

    def increment():
        count[0] += 1
        return count[0]

    return increment

counter = make_counter()
print(counter())
print(counter())
```

---

# Practical Exercises

## Exercise 1

Draw a frame/object diagram for:

```python
def f():
    x = [1, 2]
    return x

y = f()
```

Show what exists during the function call and after it returns.

## Exercise 2

Write a function that mutates a list argument.

Draw the references from caller frame and function frame to the same list object.

## Exercise 3

Write a function that rebinds a parameter.

Explain why the caller's name is unchanged.

## Exercise 4

Create a nested object graph using a dictionary containing lists and dictionaries.

Draw the references.

## Exercise 5

Create a closure that keeps a list alive after the outer function returns.

Explain which object survives and why.

## Exercise 6

Write a recursive function and draw the stack frames for three recursive calls.

## Exercise 7

Create a shallow copy of a nested list.

Mutate an inner list and explain why both outer lists show the change.

## Exercise 8

Create a custom data structure with internal list storage.

Draw the object graph.

## Exercise 9

Find an example where scope ends but object lifetime continues.

Explain the difference.

## Exercise 10

Explain why this statement is misleading for Python:

```text
The variable contains the object.
```

Rewrite it using names, references, and objects.

---

# Summary

In this chapter we learned:

* The stack is the structure of active function calls.
* A function call creates a frame.
* A frame contains local names and execution state.
* Local names refer to objects.
* Python objects live in Python-managed memory.
* The heap is a useful broad idea for object memory, but Python should be understood through frames, references, objects, and reachability.
* Names are not boxes containing objects.
* Returning a local object is safe if a reference survives.
* Passing an object to a function passes a reference.
* Rebinding a parameter does not rebind the caller's name.
* Mutating a shared object affects all references to that object.
* Containers store references to objects.
* Object graphs can survive after creating frames disappear.
* Scope and lifetime are different.
* Recursion grows the call stack.
* Closures can keep objects alive.

Core model:

```text
frame
    |
    └── local name ─▶ object in managed memory
                         |
                         └── references to other objects
```

Stack vs heap in Python is best understood as:

```text
temporary frames + managed objects + references between them
```

---

# Preview of Chapter 34

Next we study reference counting.

This chapter showed that objects stay alive while references point to them.

Chapter 34 explains how CPython tracks those references.

We will study:

* What a reference count is.
* What increases a reference count.
* What decreases a reference count.
* Why assignment affects references.
* Why passing arguments affects references.
* Why containers affect references.
* How objects become eligible for cleanup.
* Why reference cycles are more complicated.

The transition is direct:

```text
references keep objects alive
reference counting tracks how many references exist
```

Reference counting is the first concrete memory-management mechanism to understand.
