# Chapter 22 — Closures

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a closure is.
* Why closures exist.
* How nested functions work.
* How inner functions access enclosing names.
* How a function can remember data after an outer function returns.
* What free variables are.
* What `nonlocal` does.
* Why rebinding enclosing names requires `nonlocal`.
* How closures can store state.
* How closures are used for function factories.
* How closures support decorators later.
* What the late binding pitfall is.
* When closures are useful.
* When a class may be clearer than a closure.

Closures combine functions, scope, and object references.

---

# Concept Overview

A closure is a function that remembers names from an enclosing scope even after that enclosing function has finished executing.

Example:

```python
def make_greeter(greeting):
    def greet(name):
        return f"{greeting}, {name}"

    return greet

hello = make_greeter("Hello")
print(hello("Ada"))
```

Output:

```text
Hello, Ada
```

The outer function `make_greeter` has already returned.

But the inner function `greet` still remembers `greeting`.

That is the closure.

Core model:

```text
outer function call
    |
    v
creates inner function
    |
    v
inner function keeps references to needed enclosing names
    |
    v
inner function can use them later
```

---

# Why Closures Exist

Closures let functions carry context.

They solve problems such as:

* Creating specialized functions.
* Remembering configuration.
* Preserving state without global variables.
* Building callbacks.
* Implementing decorators.
* Delaying work while keeping needed data.

Example:

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor

    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(10))
print(triple(10))
```

Output:

```text
20
30
```

Each returned function remembers a different `factor`.

---

# Nested Functions

A nested function is a function defined inside another function.

Example:

```python
def outer():
    def inner():
        print("inside")

    inner()

outer()
```

Output:

```text
inside
```

The name `inner` is local to `outer`.

Outside:

```python
inner()
```

raises:

```text
NameError
```

The nested function object exists because `outer` creates it when `outer` runs.

---

# Inner Functions Can Read Enclosing Names

Example:

```python
def outer():
    message = "hello"

    def inner():
        print(message)

    inner()

outer()
```

Output:

```text
hello
```

When `inner` evaluates `message`, Python uses LEGB:

```text
Local: inner has no message
Enclosing: outer has message
```

So `inner` finds `message` in the enclosing scope.

---

# Returning an Inner Function

The key closure behavior appears when the inner function is returned.

Example:

```python
def outer():
    message = "hello"

    def inner():
        return message

    return inner

fn = outer()
print(fn())
```

Output:

```text
hello
```

`outer()` has finished.

Its normal frame is gone.

But the returned function still has access to `message`.

Python keeps the needed value alive for the inner function.

---

# Free Variables

A free variable is a name used inside a function but not defined inside that function.

Example:

```python
def make_greeter(greeting):
    def greet(name):
        return f"{greeting}, {name}"

    return greet
```

Inside `greet`:

```text
name       -> local variable
greeting   -> free variable
```

`greeting` is free in `greet` because `greet` uses it but does not define it.

Python resolves it from the enclosing function scope.

---

# Closure Memory Model

Conceptually:

```text
make_greeter("Hello")
    |
    v
greeting ─────▶ str object "Hello"
    |
    v
creates greet function object
    |
    v
greet keeps access to greeting
```

After `make_greeter` returns:

```text
hello ─────▶ function object greet
              └── closure reference to greeting -> "Hello"
```

The name `greeting` is no longer a normal active local name in a running frame.

But the function object retains access to the object through closure machinery.

At a high level:

> A closure keeps needed enclosing bindings alive.

---

# Closures Keep References, Not Copies

Closures remember references to objects.

Example:

```python
def make_reader():
    items = []

    def read_items():
        return items

    return read_items

reader = make_reader()
print(reader())
```

Output:

```text
[]
```

The inner function remembers the list object through the enclosing name.

This matters when the object is mutable.

Closures do not automatically copy objects.

They keep access to the captured objects.

---

# Closures and Mutable Objects

Example:

```python
def make_collector():
    items = []

    def add_item(item):
        items.append(item)
        return items

    return add_item

collect = make_collector()

print(collect("a"))
print(collect("b"))
```

Output:

```text
['a']
['a', 'b']
```

The closure remembers the list object `items`.

Each call mutates the same list.

This is state without a global variable.

---

# Rebinding Enclosing Names

Reading an enclosing name works.

Mutating an object referenced by an enclosing name can work.

But rebinding an enclosing name requires `nonlocal`.

Problem:

```python
def make_counter():
    count = 0

    def increment():
        count = count + 1
        return count

    return increment

counter = make_counter()
counter()
```

This raises:

```text
UnboundLocalError
```

Why?

Assignment to `count` inside `increment` makes `count` local to `increment`.

Then `count + 1` tries to read that local `count` before assignment.

---

# nonlocal

Use `nonlocal` to rebind a name from an enclosing function scope.

Example:

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count = count + 1
        return count

    return increment

counter = make_counter()

print(counter())
print(counter())
print(counter())
```

Output:

```text
1
2
3
```

`nonlocal count` tells Python:

```text
do not create a local count
rebind count from the nearest enclosing function scope
```

---

# nonlocal Is Not global

`global` targets module-level names.

`nonlocal` targets enclosing function names.

Example:

```python
value = "global"

def outer():
    value = "outer"

    def inner():
        nonlocal value
        value = "changed"

    inner()
    return value

print(outer())
print(value)
```

Output:

```text
changed
global
```

The enclosing `value` changed.

The global `value` did not.

---

# Function Factories

A function factory is a function that creates and returns functions.

Example:

```python
def make_power(exponent):
    def power(base):
        return base ** exponent

    return power

square = make_power(2)
cube = make_power(3)

print(square(5))
print(cube(5))
```

Output:

```text
25
125
```

Each returned function remembers a different `exponent`.

This is a direct practical use of closures.

---

# Closures for Configuration

Closures can store configuration.

Example:

```python
def make_validator(min_length):
    def validate(text):
        return len(text) >= min_length

    return validate

is_valid_username = make_validator(3)
is_valid_password = make_validator(8)

print(is_valid_username("ada"))
print(is_valid_password("secret"))
```

Output:

```text
True
False
```

The validation logic is shared.

The configuration differs.

---

# Closures for Delayed Work

Closures can delay work while retaining data.

Example:

```python
def make_message_sender(user, message):
    def send():
        return f"Sending {message!r} to {user}"

    return send

send_later = make_message_sender("Ada", "Welcome")

print(send_later())
```

Output:

```text
Sending 'Welcome' to Ada
```

The returned function keeps the `user` and `message` objects available.

This pattern appears in callbacks and event-driven systems.

---

# Closures and Decorators Preview

Decorators rely heavily on closures.

Example pattern:

```python
def decorator(func):
    def wrapper():
        print("before")
        result = func()
        print("after")
        return result

    return wrapper
```

The inner `wrapper` remembers `func`.

This is a closure.

Decorators are a later chapter.

For now:

> A decorator works because a function can return another function that remembers the original function.

---

# Inspecting Closures

Function objects expose closure information through `__closure__`.

Example:

```python
def outer():
    message = "hello"

    def inner():
        return message

    return inner

fn = outer()
print(fn.__closure__)
```

You may see closure cell objects.

You do not need to depend on this in normal code.

It shows that Python stores closure data as part of the function object.

This is introspection, not everyday application logic.

---

# Late Binding Pitfall

Closures capture variables, not immediate copied values.

Common pitfall:

```python
functions = []

for number in range(3):
    def show():
        return number
    functions.append(show)

for function in functions:
    print(function())
```

Many expect:

```text
0
1
2
```

Actual output:

```text
2
2
2
```

Why?

Each function refers to the same `number` variable from the loop scope.

After the loop ends, `number` is `2`.

---

# Fixing Late Binding With Defaults

A common fix uses default arguments:

```python
functions = []

for number in range(3):
    def show(number=number):
        return number
    functions.append(show)

for function in functions:
    print(function())
```

Output:

```text
0
1
2
```

Why?

Default argument values are evaluated when the function is defined.

Each function gets its own default value.

This is one case where default argument evaluation timing is useful.

---

# Fixing Late Binding With a Function Factory

Another fix:

```python
def make_show(number):
    def show():
        return number
    return show

functions = []

for number in range(3):
    functions.append(make_show(number))

for function in functions:
    print(function())
```

Output:

```text
0
1
2
```

Each call to `make_show` creates a separate enclosing scope with its own `number`.

This is often clearer than the default-argument trick when teaching or designing APIs.

---

# Closures vs Classes

Closures can store state.

Classes can also store state.

Closure:

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

Class:

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count
```

Use a closure when:

* The state is small.
* The behavior is focused.
* You need one callable.

Use a class when:

* There are multiple operations.
* State has a richer structure.
* The object needs a clear public interface.
* You need inheritance or type-based organization.

---

# Closures and Encapsulation

Closures can hide state.

Example:

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment

counter = make_counter()
```

The caller can call:

```python
counter()
```

But cannot directly access `count` as a normal attribute.

This can be useful.

It can also make debugging harder if overused.

Use closures intentionally.

---

# Common Mistakes

## Misconception 1

### Inner functions cannot use outer function names.

They can read names from enclosing scopes.

This is the E in LEGB.

---

## Misconception 2

### Outer local variables disappear completely when the outer function returns.

Normal local names disappear.

But objects/bindings needed by returned inner functions can be kept alive through closure machinery.

---

## Misconception 3

### Closures copy values automatically.

Closures keep references to variables/objects.

Late binding can surprise you.

---

## Misconception 4

### Rebinding an enclosing variable works without `nonlocal`.

It does not.

Assignment inside the inner function creates a local name unless `nonlocal` is used.

---

## Misconception 5

### nonlocal modifies global variables.

`nonlocal` targets enclosing function scopes.

`global` targets module-level scope.

---

## Misconception 6

### Closures are always better than classes.

Closures are useful for small, focused stateful callables.

Classes are clearer for richer state and multiple behaviors.

---

## Misconception 7

### The late binding loop behavior is a Python bug.

It follows from how closures resolve names.

The functions look up the variable when called, after the loop has completed.

---

# Real-world Usage

## Function Factories

```python
def make_prefixer(prefix):
    def add_prefix(text):
        return prefix + text

    return add_prefix
```

This creates specialized functions from configuration.

---

## Callbacks

Closures are useful for callbacks that need context.

Example:

```python
def make_handler(user_id):
    def handle(event):
        return f"Handling {event} for {user_id}"

    return handle
```

The returned function remembers `user_id`.

---

## Decorators

Decorators use closures to wrap functions.

Example concept:

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print("calling", func.__name__)
        return func(*args, **kwargs)

    return wrapper
```

The wrapper remembers `func`.

---

## Avoiding Globals

Closures can replace small global state.

Instead of:

```python
count = 0
```

with global mutation, use:

```python
counter = make_counter()
```

This keeps state scoped to the returned function.

---

# Concept Connections

This chapter builds on:

```text
Chapter 20:
Functions are objects and can be returned.

Chapter 21:
Nested functions can access enclosing scope.

Chapter 10:
Names refer to objects through bindings.

Chapter 11:
Closures can retain mutable objects and mutate them later.
```

It prepares:

```text
Call Stack:
    closures preserve data beyond active frames

Decorators:
    wrappers remember original functions

Iterators and generators:
    functions can preserve execution-related state

Object-Oriented Programming:
    closures and classes are both tools for bundling state with behavior
```

Core model:

```text
outer call creates inner function
    |
    v
inner function references enclosing names
    |
    v
returned function keeps needed references alive
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Nested function | Function defined inside another function |
| Enclosing scope | Scope of an outer function |
| Closure | Function plus retained access to enclosing names |
| Free variable | Name used in a function but defined outside it |
| `nonlocal` | Allows rebinding a name in an enclosing function scope |
| Late binding | Closure name lookup happens when function is called |
| Function factory | Function that creates and returns functions |

Core rules:

```text
Inner functions can read enclosing names.
Returning an inner function can create a closure.
Closures keep needed enclosing references alive.
Rebinding enclosing names requires nonlocal.
Closures capture references, not automatic value copies.
Late binding matters in loops.
Use closures for focused stateful callables.
Use classes for richer state and multiple behaviors.
```

---

# Active Recall

## Easy Recall Questions

1. What is a nested function?
2. What is a closure?
3. What is an enclosing scope?
4. What is a free variable?
5. What does `nonlocal` do?
6. What is a function factory?
7. Can a function return another function?
8. Do closures copy values automatically?
9. What is late binding?
10. What later Python feature relies heavily on closures?

---

## Deep Understanding Questions

1. Why can an inner function use a name from an outer function?
2. Why can a returned inner function still access outer data?
3. Why does rebinding an enclosing variable require `nonlocal`?
4. Why does mutating an enclosed list not require `nonlocal`?
5. Why do loop-created closures often return the final loop value?
6. Why does a function factory fix late binding?
7. Why can closures replace some uses of global state?
8. Why might a class be clearer than a closure?
9. Why are closures important for decorators?
10. Why does closure behavior depend on the name/reference model?

---

## Explain In Your Own Words

1. Explain a closure without using implementation details.
2. Explain free variables.
3. Explain `nonlocal`.
4. Explain late binding with a loop example.
5. Explain closure state.
6. Explain closure vs class tradeoffs.
7. Explain how decorators depend on closures.

---

## Predict-the-Output Questions

### Question 1

```python
def outer():
    message = "hello"

    def inner():
        return message

    return inner

fn = outer()
print(fn())
```

Answer:

```text
hello
```

Reason:

`inner` is a closure that remembers `message`.

---

### Question 2

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor
    return multiply

double = make_multiplier(2)
print(double(5))
```

Answer:

```text
10
```

Reason:

`double` remembers `factor = 2`.

---

### Question 3

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment

counter = make_counter()
print(counter())
print(counter())
```

Answer:

```text
1
2
```

Reason:

The closure retains `count`, and `nonlocal` allows rebinding it.

---

### Question 4

```python
functions = []

for number in range(3):
    def show():
        return number
    functions.append(show)

for function in functions:
    print(function())
```

Answer:

```text
2
2
2
```

Reason:

The closures refer to the same loop variable, whose final value is `2`.

---

### Question 5

```python
def make_collector():
    items = []

    def collect(item):
        items.append(item)
        return items

    return collect

collect = make_collector()
print(collect("a"))
print(collect("b"))
```

Answer:

```text
['a']
['a', 'b']
```

Reason:

The closure retains and mutates the same list object.

---

# Mental Model Questions

1. Draw `outer` returning `inner`.
2. Draw an inner function retaining an enclosing name.
3. Draw a closure that remembers `factor`.
4. Draw a counter closure with `count`.
5. Draw late binding in a loop.
6. Draw the fixed version using a function factory.

---

# Practical Exercises

## Exercise 1

Write a function factory:

```python
def make_adder(amount):
    ...
```

It should return a function that adds `amount` to its argument.

Test:

```python
add_five = make_adder(5)
print(add_five(10))
```

---

## Exercise 2

Write a counter closure using `nonlocal`.

It should return `1`, then `2`, then `3` on repeated calls.

---

## Exercise 3

Create two independent counters:

```python
a = make_counter()
b = make_counter()
```

Call them in alternating order and explain why their state is independent.

---

## Exercise 4

Reproduce the late binding bug with functions created in a loop.

Then fix it using default arguments.

---

## Exercise 5

Fix the same late binding bug using a function factory.

Explain why the function factory creates separate enclosing scopes.

---

## Exercise 6

Write a closure for validation:

```python
def min_length_validator(length):
    ...
```

It should return a function that checks whether text has at least that length.

---

## Exercise 7

Compare closure and class design.

Implement a simple counter with:

* A closure
* A class

Write one paragraph explaining which is clearer and why.

---

# Summary

In this chapter we learned:

* A nested function is defined inside another function.
* Inner functions can read names from enclosing scopes.
* A closure is a function that retains access to enclosing names.
* Closures let functions carry context.
* Functions can return functions.
* Free variables are names used inside a function but defined outside it.
* Closures keep references alive, not automatic copies.
* `nonlocal` allows rebinding names from enclosing function scopes.
* `global` targets module scope; `nonlocal` targets enclosing function scope.
* Closures can store state without globals.
* Function factories create specialized functions.
* Late binding can surprise when closures are created in loops.
* Default arguments or function factories can fix late binding.
* Closures are useful for callbacks, configuration, and decorators.
* Classes may be clearer for richer state and multiple behaviors.

Core model:

```text
outer function call
    |
    v
enclosing names
    |
    v
inner function object
    |
    v
closure retains needed references
```

Closures are the result of functions plus lexical scope.

---

# Preview of Chapter 23

Next we study the call stack.

Closures showed that function objects can retain data beyond an outer call.

The call stack explains active calls:

* Which function is currently running
* Who called it
* Where return values go
* How recursion works
* How tracebacks represent call history

Chapter 23 will complete Volume I's execution model for functions.
