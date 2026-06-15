# Chapter 21 — Scope

---

# Learning Objectives

By the end of this chapter, you should understand:

* What scope means.
* Why scope exists.
* How Python finds names.
* What local scope is.
* What global scope is.
* What built-in scope is.
* What enclosing scope is at a high level.
* The LEGB name lookup rule.
* Why assignment inside a function creates a local name.
* Why `UnboundLocalError` happens.
* How shadowing works.
* What `global` does.
* Why `global` should usually be avoided.
* What `nonlocal` means at a preview level.
* How to reason about names in functions.
* Common scope mistakes.

Scope answers one question:

> When Python sees a name, where does it look for the object?

---

# Concept Overview

In Chapter 10, we learned:

```text
names refer to objects
```

In Chapter 20, we learned:

```text
function calls create frames with local names
```

Now we need to understand name lookup.

Example:

```python
x = 10

def show():
    print(x)

show()
```

Output:

```text
10
```

How did `show()` find `x`?

Python looked beyond the function's local names and found `x` in the module's global namespace.

Scope defines where names are visible and how Python resolves them.

---

# Why Scope Exists

Without scope, every name in a program would compete in one shared space.

That would cause constant conflicts.

Example:

```python
def calculate_total(price):
    tax = price * 0.1
    return price + tax

def calculate_discount(price):
    tax = 0
    return price * 0.9
```

Both functions use a name `tax`.

Those names should not interfere with each other.

Scope lets each function have its own local names.

It provides isolation.

---

# Namespaces and Scope

A namespace is a mapping from names to objects.

Scope is the region of code where a namespace is directly accessible for name lookup.

Example:

```python
x = 10
```

At module level:

```text
global namespace
└── x ─────▶ int object 10
```

Inside a function:

```python
def add(a, b):
    total = a + b
    return total
```

During a call:

```text
local namespace for add
├── a ─────▶ argument object
├── b ─────▶ argument object
└── total ─▶ result object
```

Scope controls which namespace Python checks first.

---

# Local Scope

Local scope is the scope inside a function call.

Example:

```python
def greet(name):
    message = f"Hello, {name}"
    return message
```

During:

```python
greet("Ada")
```

the local scope contains:

```text
name ─────▶ "Ada"
message ──▶ "Hello, Ada"
```

After the function returns, that function call's local scope is gone.

Trying to access `message` outside:

```python
print(message)
```

raises:

```text
NameError
```

---

# Global Scope

Global scope is the module-level scope.

Example:

```python
app_name = "Python Mastery"

def show_app_name():
    print(app_name)
```

The name `app_name` is global within that module.

When `show_app_name()` runs, Python can find it in global scope if it is not local.

Global does not mean:

```text
visible to every program everywhere
```

It means:

```text
module-level namespace
```

Each module has its own global namespace.

---

# Built-in Scope

Built-in scope contains names Python provides automatically.

Examples:

```python
print
len
type
id
range
str
int
bool
Exception
```

When you write:

```python
print(len("hello"))
```

Python finds `print` and `len` in built-in scope if they are not shadowed by local or global names.

Built-ins are available almost everywhere.

But they can be shadowed.

---

# Enclosing Scope

Enclosing scope appears with nested functions.

Example:

```python
def outer():
    message = "hello"

    def inner():
        print(message)

    inner()
```

The name `message` is not local to `inner`.

It is not global.

It is in the enclosing function scope: `outer`.

This is the foundation of closures.

Closures are Chapter 22.

For now, understand:

> A nested function can look in the scopes of functions that enclose it.

---

# LEGB Rule

Python name lookup follows LEGB:

```text
L -> Local
E -> Enclosing
G -> Global
B -> Built-in
```

When Python evaluates a name, it looks in this order:

```text
1. Local scope
2. Enclosing function scopes
3. Global module scope
4. Built-in scope
```

Example:

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)

    inner()

outer()
```

Output:

```text
local
```

Python found `x` in local scope first.

---

# LEGB Example Without Local Name

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        print(x)

    inner()

outer()
```

Output:

```text
enclosing
```

`inner` has no local `x`.

Python checks enclosing scope and finds `x` in `outer`.

It does not continue to global once it finds a match.

---

# LEGB Example With Global Name

```python
x = "global"

def outer():
    def inner():
        print(x)

    inner()

outer()
```

Output:

```text
global
```

There is no local `x` in `inner`.

There is no enclosing `x` in `outer`.

Python finds `x` in global scope.

---

# LEGB Example With Built-in Name

```python
def show_length(value):
    print(len(value))

show_length("hello")
```

Python resolves:

```text
print -> built-in
len   -> built-in
value -> local
```

The local name `value` comes from the function parameter.

The names `print` and `len` are built-ins.

---

# Shadowing

Shadowing happens when a name in an inner scope hides a name in an outer scope.

Example:

```python
x = "global"

def show():
    x = "local"
    print(x)

show()
print(x)
```

Output:

```text
local
global
```

The local `x` shadows the global `x` inside `show`.

It does not overwrite the global binding.

---

# Shadowing Built-ins

Avoid naming variables after built-ins.

Bad:

```python
list = [1, 2, 3]
print(list("abc"))
```

This raises:

```text
TypeError
```

Why?

The name `list` now refers to a list object, not the built-in `list` class.

Other risky names:

```python
str
int
dict
set
sum
max
min
len
id
type
input
```

Use more specific names:

```python
names_list
total_sum
user_input
```

---

# Assignment Creates Local Names

If a name is assigned anywhere inside a function, Python treats it as local to that function unless declared otherwise.

Example:

```python
x = 10

def show():
    x = 20
    print(x)

show()
print(x)
```

Output:

```text
20
10
```

The assignment:

```python
x = 20
```

creates a local `x`.

It does not modify the global `x`.

---

# UnboundLocalError

This code is a common surprise:

```python
x = 10

def show():
    print(x)
    x = 20

show()
```

Python raises:

```text
UnboundLocalError
```

Why?

Because assigning to `x` anywhere in the function makes `x` local to that function.

So this line:

```python
print(x)
```

tries to read the local `x` before it has been assigned.

Python does not read the global `x` in this case.

---

# Reading Globals Is Allowed

You can read a global name inside a function.

Example:

```python
tax_rate = 0.1

def calculate_tax(price):
    return price * tax_rate
```

This works.

Python looks for `tax_rate`:

```text
local -> not found
enclosing -> not applicable
global -> found
```

Reading globals is common for constants and configuration.

But heavy dependence on global state can make functions harder to test.

---

# Assigning Globals Requires global

If you want to assign to a global name inside a function, you need `global`.

Example:

```python
count = 0

def increment():
    global count
    count += 1

increment()
print(count)
```

Output:

```text
1
```

`global count` tells Python:

```text
when assigning count, use the module-level binding
```

Without `global`, Python treats `count` as local and raises `UnboundLocalError`.

---

# Avoid global When Possible

`global` is sometimes necessary, but often avoidable.

Instead of:

```python
count = 0

def increment():
    global count
    count += 1
```

prefer returning a new value:

```python
def increment(count):
    return count + 1

count = 0
count = increment(count)
```

This makes data flow explicit.

Global mutation creates hidden dependencies.

Hidden dependencies make testing and debugging harder.

---

# Constants in Global Scope

Global constants are common.

Example:

```python
DEFAULT_TIMEOUT = 30
MAX_RETRIES = 3

def connect(timeout=DEFAULT_TIMEOUT):
    ...
```

Uppercase names conventionally indicate constants.

Python does not enforce constantness.

The convention tells readers:

```text
this value should not be reassigned
```

Using global constants is different from mutating global state.

Constants are generally safer.

---

# nonlocal Preview

`nonlocal` is used with enclosing scopes.

Example:

```python
def outer():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

`nonlocal count` tells Python:

```text
assign to count from the nearest enclosing function scope
```

This is closure-related.

Chapter 22 will explain it properly.

For now, know:

```text
global -> module scope
nonlocal -> enclosing function scope
```

---

# Scope Is Lexical

Python uses lexical scope.

Lexical scope means scope is determined by where code is written, not by where a function is called.

Example:

```python
x = "global"

def show():
    print(x)

def caller():
    x = "caller local"
    show()

caller()
```

Output:

```text
global
```

`show` was defined at module level.

It does not look inside `caller` just because `caller` called it.

Scope follows code structure.

---

# Scope vs Lifetime

Scope and lifetime are related but not identical.

Scope:

```text
where a name can be looked up
```

Lifetime:

```text
how long an object exists
```

Example:

```python
def make_list():
    items = [1, 2, 3]
    return items

result = make_list()
```

The local name `items` is gone after the function returns.

But the list object continues to exist because `result` refers to it.

Do not confuse name lifetime with object lifetime.

---

# locals()

`locals()` returns a mapping of local names.

Example:

```python
def show():
    x = 10
    y = 20
    print(locals())

show()
```

Output may include:

```text
{'x': 10, 'y': 20}
```

Use `locals()` for inspection and debugging.

Do not rely on modifying it to change local variables.

---

# globals()

`globals()` returns the module global namespace dictionary.

Example:

```python
x = 10

print(globals()["x"])
```

Output:

```text
10
```

This reveals that global scope is a namespace mapping.

Normal code should usually access names directly, not through `globals()`.

Use it for introspection when appropriate.

---

# Name Lookup Is Runtime Behavior

Name lookup happens when code runs.

Example:

```python
def show():
    print(message)

message = "hello"
show()
```

Output:

```text
hello
```

The name `message` is found when `show()` executes.

Now:

```python
def show():
    print(message)

show()
message = "hello"
```

This raises:

```text
NameError
```

At call time, `message` is not yet bound.

---

# Scope and Imports

Imported names are bindings in a namespace.

Example:

```python
import math

print(math.sqrt(16))
```

The name `math` is bound in the current namespace.

Example:

```python
from math import sqrt

print(sqrt(16))
```

The name `sqrt` is bound in the current namespace.

Imports do not create magical global availability everywhere.

They bind names in the module where the import statement runs.

Modules are covered later.

---

# Scope and Comprehensions

In modern Python, comprehensions have their own local scope for loop variables.

Example:

```python
items = [number * 2 for number in range(3)]
```

The comprehension variable `number` does not leak into the surrounding scope.

This differs from normal `for` loops:

```python
for number in range(3):
    pass

print(number)
```

prints:

```text
2
```

This distinction matters, but comprehensions will be studied later in more detail.

---

# Common Mistakes

## Misconception 1

### Global means visible across all files automatically.

In Python, global means module-level.

Each module has its own global namespace.

---

## Misconception 2

### Assigning inside a function updates the global name.

Assignment inside a function creates a local name unless declared `global` or `nonlocal`.

---

## Misconception 3

### Python decides local/global line by line.

Python determines local names for a function based on assignments in the function body.

This is why `UnboundLocalError` can happen.

---

## Misconception 4

### Function callers affect the callee's scope.

Python uses lexical scope.

Where a function is defined matters, not who calls it.

---

## Misconception 5

### Shadowing built-ins is harmless.

Shadowing built-ins can break later code and confuse readers.

Avoid names such as `list`, `str`, `sum`, and `type`.

---

## Misconception 6

### local variables and local objects are the same thing.

Local names can refer to objects.

Objects may outlive local names if returned or stored elsewhere.

---

## Misconception 7

### global is the normal way to share data.

Prefer arguments, return values, objects, or explicit configuration.

Use `global` only when the design really calls for module-level mutation.

---

# Real-world Usage

## Constants

Module-level constants are common:

```python
DEFAULT_LIMIT = 100

def fetch(limit=DEFAULT_LIMIT):
    ...
```

This is readable and testable when constants are not mutated.

---

## Avoiding Hidden State

Harder to test:

```python
current_user = None

def can_edit():
    return current_user is not None and current_user.is_admin
```

Better:

```python
def can_edit(user):
    return user is not None and user.is_admin
```

Passing data explicitly makes dependencies visible.

---

## Debugging NameError

When you see:

```text
NameError: name 'x' is not defined
```

ask:

```text
Where should x be bound?
Which scope should contain it?
Did that code run before this lookup?
Was the name misspelled?
```

Name errors are scope and execution-order problems.

---

## Debugging UnboundLocalError

When you see:

```text
UnboundLocalError
```

look for assignment to the same name inside the function.

Example pattern:

```python
x = x + 1
```

inside a function may be trying to read a local `x` before assignment.

Decide whether the function should:

* Receive `x` as an argument.
* Return a new value.
* Use `global`.
* Use `nonlocal`.

Most often, arguments and return values are cleaner.

---

# Concept Connections

This chapter builds on:

```text
Chapter 10:
Names refer to objects through bindings.

Chapter 20:
Function calls create frames with local names.

Chapter 17:
Name expressions require runtime lookup.
```

It prepares:

```text
Closures:
    nested functions can remember enclosing names

Call Stack:
    active frames contain local namespaces

Modules:
    module globals are namespaces

Classes:
    class bodies introduce another namespace pattern
```

Core model:

```text
name expression
    |
    v
LEGB lookup
    |
    v
object
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Scope | Region where a name can be resolved |
| Namespace | Mapping from names to objects |
| Local | Function-call scope |
| Enclosing | Scope of surrounding function |
| Global | Module-level scope |
| Built-in | Names provided by Python |
| LEGB | Local, Enclosing, Global, Built-in |
| Shadowing | Inner name hiding outer name |
| `global` | Declare assignment targets module-level name |
| `nonlocal` | Declare assignment targets enclosing function name |
| Lexical scope | Scope based on where code is written |

Core rules:

```text
Python resolves names using LEGB.
Assignment in a function creates a local name by default.
Reading globals from functions is allowed.
Assigning globals from functions requires global.
Nested functions can access enclosing names.
Scope is lexical, not based on caller.
Avoid shadowing built-ins.
Prefer explicit arguments and returns over mutable global state.
```

---

# Active Recall

## Easy Recall Questions

1. What is scope?
2. What is a namespace?
3. What does LEGB stand for?
4. What is local scope?
5. What is global scope?
6. What is built-in scope?
7. What is enclosing scope?
8. What does `global` do?
9. What does shadowing mean?
10. What error happens when a local name is read before assignment?

---

## Deep Understanding Questions

1. Why does Python need scope?
2. Why does assignment inside a function create a local name?
3. Why does `UnboundLocalError` happen?
4. Why is global state harder to test?
5. Why is lexical scope based on where code is written?
6. Why does a caller's local variable not affect a called function's scope?
7. Why can objects outlive local names?
8. Why should built-in names not be shadowed?
9. Why is `global` often avoidable?
10. Why does closure behavior require enclosing scope?

---

## Explain In Your Own Words

1. Explain LEGB.
2. Explain local vs global names.
3. Explain shadowing.
4. Explain why `x = x + 1` can fail inside a function.
5. Explain lexical scope.
6. Explain scope vs lifetime.
7. Explain why imports create name bindings.

---

## Predict-the-Output Questions

### Question 1

```python
x = "global"

def show():
    x = "local"
    print(x)

show()
print(x)
```

Answer:

```text
local
global
```

Reason:

The local `x` shadows the global `x` inside the function.

---

### Question 2

```python
x = 10

def show():
    print(x)

show()
```

Answer:

```text
10
```

Reason:

There is no local `x`, so Python finds `x` in global scope.

---

### Question 3

```python
x = 10

def show():
    print(x)
    x = 20

show()
```

Answer:

Python raises `UnboundLocalError`.

Reason:

Assignment to `x` in the function makes `x` local. The local `x` is read before assignment.

---

### Question 4

```python
x = "global"

def show():
    print(x)

def caller():
    x = "caller"
    show()

caller()
```

Answer:

```text
global
```

Reason:

Scope is lexical. `show` does not look in `caller`'s local scope.

---

### Question 5

```python
len = 5
print(len("hello"))
```

Answer:

Python raises `TypeError`.

Reason:

The name `len` now refers to an integer object, shadowing the built-in `len`.

---

### Question 6

```python
def make_list():
    items = [1, 2]
    return items

result = make_list()
print(result)
```

Answer:

```text
[1, 2]
```

Reason:

The local name `items` is gone after return, but the list object lives on because `result` refers to it.

---

# Mental Model Questions

1. Draw LEGB lookup for a local name.
2. Draw LEGB lookup for a global name used inside a function.
3. Draw local shadowing of a global name.
4. Draw why `UnboundLocalError` happens.
5. Draw scope vs object lifetime for a returned list.
6. Draw a nested function reading an enclosing name.

---

# Practical Exercises

## Exercise 1

Identify scopes:

```python
x = 10

def add(y):
    z = x + y
    return z
```

Classify `x`, `y`, and `z` as global or local in the context of `add`.

---

## Exercise 2

Trigger and fix `UnboundLocalError`:

```python
count = 0

def increment():
    count = count + 1
    return count
```

First explain why it fails.

Then rewrite it using arguments and return values instead of `global`.

---

## Exercise 3

Shadow a built-in:

```python
list = [1, 2, 3]
```

Explain why this is risky.

Rename the variable.

---

## Exercise 4

Explore lexical scope:

```python
message = "global"

def show():
    print(message)

def run():
    message = "local to run"
    show()

run()
```

Explain why the output is not `"local to run"`.

---

## Exercise 5

Use `locals()`:

```python
def inspect(a, b):
    total = a + b
    print(locals())

inspect(2, 3)
```

Explain the mapping that prints.

---

## Exercise 6

Use a global constant safely:

```python
TAX_RATE = 0.1

def calculate_total(price):
    return price + price * TAX_RATE
```

Explain why reading a global constant is different from mutating global state.

---

## Exercise 7

Preview enclosing scope:

```python
def outer():
    message = "hello"

    def inner():
        print(message)

    inner()

outer()
```

Explain where `inner` finds `message`.

---

# Summary

In this chapter we learned:

* Scope controls where Python looks for names.
* A namespace maps names to objects.
* Local scope belongs to a function call.
* Global scope belongs to a module.
* Built-in scope contains names Python provides.
* Enclosing scope appears with nested functions.
* Python name lookup follows LEGB: Local, Enclosing, Global, Built-in.
* Assignment inside a function creates a local name by default.
* `UnboundLocalError` happens when Python treats a name as local but it is read before assignment.
* Shadowing hides an outer name with an inner name.
* Shadowing built-ins is usually a mistake.
* `global` lets assignment target module-level names.
* `global` should usually be avoided in favor of explicit arguments and return values.
* `nonlocal` targets enclosing function scope and leads into closures.
* Python uses lexical scope.
* Scope and object lifetime are different.

Core model:

```text
name
    |
    v
LEGB lookup
    |
    v
object
```

Scope is the name-resolution layer of Python execution.

---

# Preview of Chapter 22

Next we study closures.

Closures build directly on enclosing scope.

They answer:

> How can an inner function remember names from an outer function after the outer function has returned?

We will study:

* Nested functions
* Enclosing names
* Function objects retaining state
* `nonlocal`
* Closure use cases
* Closure pitfalls

Closures are where functions and scope combine into a more powerful abstraction.
