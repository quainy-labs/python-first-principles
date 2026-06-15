# Chapter 25 — Functional Programming

---

# Learning Objectives

By the end of this chapter, you should understand:

* What functional programming means in Python.
* Why functions are first-class objects.
* How to pass functions as arguments.
* How to return functions from functions.
* What higher-order functions are.
* What pure functions are.
* What side effects are.
* Why immutability is useful for reasoning.
* How `lambda` expressions work.
* How `map`, `filter`, and `reduce` work conceptually.
* Why comprehensions are often preferred over `map` and `filter` in Python.
* How function composition works.
* How closures connect to functional programming.
* When functional style improves code.
* When functional style makes code harder to read.

This chapter completes the functions part.

You already know how functions are defined, called, scoped, closed over, stacked, and recursively invoked.

Now we study functions as values that can be moved around and combined.

---

# Concept Overview

Functional programming is a programming style that emphasizes functions as building blocks.

In functional style, you often think in terms of:

```text
input -> function -> output
```

Example:

```python
def double(number):
    return number * 2

print(double(10))
```

Output:

```text
20
```

That is a normal function.

Functional programming becomes more visible when functions are treated as values:

```python
def double(number):
    return number * 2

operation = double

print(operation(10))
```

Output:

```text
20
```

The name `operation` refers to the same function object as `double`.

This should not feel surprising now.

Earlier chapters established the model:

```text
name -> object
```

A function is an object.

Therefore:

```text
function name -> function object
```

Functional programming uses that fact deliberately.

---

# Why Functional Programming Exists

Functional programming helps manage complexity by making data flow explicit.

Instead of focusing on step-by-step mutation:

```python
result = []

for value in values:
    cleaned = value.strip().casefold()
    if cleaned:
        result.append(cleaned)
```

You can sometimes think in transformations:

```text
raw values
    -> strip whitespace
    -> normalize case
    -> keep non-empty values
    -> result
```

Functional style is useful when code is mostly about:

* Transforming values.
* Filtering values.
* Combining values.
* Passing behavior into reusable operations.
* Avoiding unnecessary shared mutable state.
* Making small operations easy to test.

Python is not a purely functional language.

Python supports multiple styles:

* Procedural programming.
* Object-oriented programming.
* Functional programming.
* Imperative scripting.

Good Python often blends styles.

The goal is not to force functional programming everywhere.

The goal is to know when it makes code clearer.

---

# Functions Are Objects

When you define a function, Python creates a function object and binds it to a name.

Example:

```python
def greet(name):
    return f"Hello, {name}"
```

Conceptually:

```text
greet ─────▶ function object
```

You can inspect this:

```python
print(greet)
print(type(greet))
```

Output resembles:

```text
<function greet at 0x...>
<class 'function'>
```

The exact memory address is not important.

The important point is:

```text
greet is a name
the function is an object
```

Because functions are objects, you can:

* Assign them to variables.
* Store them in lists.
* Store them in dictionaries.
* Pass them to other functions.
* Return them from functions.
* Close over them.

---

# Assigning Functions To Names

Example:

```python
def square(number):
    return number * number

operation = square

print(operation(5))
```

Output:

```text
25
```

No new function is created.

Both names refer to the same function object:

```text
square ────────┐
               v
operation ─▶ function object
```

This is the same name-reference model used everywhere else in Python.

Rebinding one name does not destroy the function object if another reference remains:

```python
def square(number):
    return number * number

operation = square
square = None

print(operation(5))
```

Output:

```text
25
```

The function object still exists because `operation` still refers to it.

---

# Passing Functions As Arguments

A function can receive another function as an argument.

Example:

```python
def apply_twice(function, value):
    first = function(value)
    second = function(first)
    return second

def double(number):
    return number * 2

print(apply_twice(double, 3))
```

Output:

```text
12
```

Execution:

```text
apply_twice(double, 3)
first = double(3) -> 6
second = double(6) -> 12
return 12
```

The parameter `function` is a local name.

It refers to the function object passed by the caller.

Inside `apply_twice`, this:

```python
function(value)
```

calls whatever function object `function` refers to.

---

# Higher-Order Functions

A higher-order function is a function that does at least one of these:

* Accepts a function as an argument.
* Returns a function.

Example:

```python
def apply(function, value):
    return function(value)
```

`apply` is higher-order because it accepts a function.

Example:

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor

    return multiply
```

`make_multiplier` is higher-order because it returns a function.

Higher-order functions are useful when you want to separate:

```text
what should be done
```

from:

```text
when or where it should be done
```

---

# Returning Functions

Functions can create and return other functions.

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

This uses closures.

`double` remembers:

```text
factor = 2
```

`triple` remembers:

```text
factor = 3
```

The function returned by `make_multiplier` retains access to the enclosing variable `factor`.

This is functional programming using the closure model from Chapter 22.

---

# Storing Functions In Data Structures

Since functions are objects, they can be stored in containers.

Example:

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

operations = {
    "add": add,
    "subtract": subtract,
    "multiply": multiply,
}

operation = operations["multiply"]
print(operation(6, 7))
```

Output:

```text
42
```

This pattern is common in command dispatch:

```python
commands = {
    "start": start_server,
    "stop": stop_server,
    "restart": restart_server,
}
```

Instead of writing a long chain:

```python
if command == "start":
    start_server()
elif command == "stop":
    stop_server()
elif command == "restart":
    restart_server()
```

you can look up the behavior:

```python
handler = commands[command]
handler()
```

This is not always better.

For simple branching, `if` / `elif` may be clearer.

For many commands, a dispatch dictionary can be cleaner.

---

# Pure Functions

A pure function has two main properties:

* Given the same inputs, it returns the same output.
* It does not cause side effects.

Example:

```python
def add(a, b):
    return a + b
```

This is pure.

Same inputs:

```python
add(2, 3)
```

always return:

```text
5
```

The function does not mutate external data, print, write files, send requests, or change global state.

Pure functions are easy to test:

```python
assert add(2, 3) == 5
```

They are easy to reason about because the output depends only on the inputs.

---

# Side Effects

A side effect is any observable change outside the returned value.

Examples:

* Printing.
* Mutating a list.
* Writing a file.
* Sending a network request.
* Updating a database.
* Changing a global variable.
* Reading user input.

Example:

```python
def add_item(items, item):
    items.append(item)
```

This function has a side effect.

It mutates the list passed by the caller.

Example:

```python
items = ["a"]
add_item(items, "b")
print(items)
```

Output:

```text
['a', 'b']
```

Side effects are not bad.

Programs need side effects to do real work.

But uncontrolled side effects make code harder to reason about.

Functional style often tries to isolate side effects.

---

# Pure Version vs Mutating Version

Mutating version:

```python
def add_item(items, item):
    items.append(item)
    return items
```

Pure-style version:

```python
def with_item(items, item):
    return items + [item]
```

Example:

```python
original = ["a"]
updated = with_item(original, "b")

print(original)
print(updated)
```

Output:

```text
['a']
['a', 'b']
```

The pure-style version returns a new list.

The original list is unchanged.

Tradeoff:

* Pure-style code can be easier to reason about.
* Mutating code can be more memory-efficient.

Good engineering chooses based on the situation.

---

# Immutability And Reasoning

Immutability helps because values do not change unexpectedly.

Example:

```python
def normalize_name(name):
    return name.strip().casefold()
```

Strings are immutable.

This function cannot alter the caller's original string object.

It returns a new string.

For mutable containers, you must choose:

```python
def sort_in_place(values):
    values.sort()
    return values
```

or:

```python
def sorted_copy(values):
    return sorted(values)
```

The names communicate different behavior.

Functional style favors the second version when practical.

It avoids hidden mutation.

---

# Lambda Expressions

A `lambda` expression creates a small anonymous function.

Example:

```python
double = lambda number: number * 2

print(double(10))
```

Output:

```text
20
```

This is similar to:

```python
def double(number):
    return number * 2
```

Lambda syntax:

```python
lambda parameters: expression
```

Important limitations:

* A lambda contains one expression.
* It does not contain statements.
* It is best used for small, local behavior.

Good use:

```python
names = ["Ada", "grace", "Linus"]
print(sorted(names, key=lambda name: name.casefold()))
```

Avoid using lambdas for complex logic.

If the function needs a meaningful name or multiple steps, use `def`.

---

# `sorted()` With `key`

One of the most common higher-order patterns in Python is passing a key function to `sorted()`.

Example:

```python
users = [
    {"name": "Ada", "age": 36},
    {"name": "Linus", "age": 28},
    {"name": "Grace", "age": 85},
]

by_age = sorted(users, key=lambda user: user["age"])

print(by_age)
```

The `key` function receives each item and returns the value to sort by.

Mental model:

```text
for each user:
    compute user["age"]
sort by those computed values
```

Using a named function:

```python
def get_age(user):
    return user["age"]

by_age = sorted(users, key=get_age)
```

The named version is often clearer if the key logic matters.

---

# `map()` Concept

`map()` applies a function to every item in an iterable.

Example:

```python
def double(number):
    return number * 2

numbers = [1, 2, 3]
result = map(double, numbers)

print(list(result))
```

Output:

```text
[2, 4, 6]
```

Conceptually:

```text
[1, 2, 3]
    -> double each value
[2, 4, 6]
```

Equivalent loop:

```python
result = []

for number in numbers:
    result.append(double(number))
```

Equivalent list comprehension:

```python
result = [double(number) for number in numbers]
```

In Python, list comprehensions are often preferred because they are explicit and readable.

`map()` is useful when the function already exists and the transformation is direct.

---

# `filter()` Concept

`filter()` keeps items where a function returns truthy.

Example:

```python
def is_even(number):
    return number % 2 == 0

numbers = [1, 2, 3, 4]
result = filter(is_even, numbers)

print(list(result))
```

Output:

```text
[2, 4]
```

Conceptually:

```text
[1, 2, 3, 4]
    -> keep only even numbers
[2, 4]
```

Equivalent loop:

```python
result = []

for number in numbers:
    if is_even(number):
        result.append(number)
```

Equivalent list comprehension:

```python
result = [number for number in numbers if is_even(number)]
```

The comprehension is usually clearer in Python.

---

# `reduce()` Concept

`reduce()` combines items into one result.

It lives in the `functools` module.

Example:

```python
from functools import reduce

def add(a, b):
    return a + b

numbers = [1, 2, 3, 4]
total = reduce(add, numbers, 0)

print(total)
```

Output:

```text
10
```

Conceptually:

```text
start with 0
combine 0 and 1 -> 1
combine 1 and 2 -> 3
combine 3 and 3 -> 6
combine 6 and 4 -> 10
```

Equivalent loop:

```python
total = 0

for number in numbers:
    total = add(total, number)
```

For common operations, Python usually has clearer tools:

```python
sum(numbers)
max(numbers)
min(numbers)
any(values)
all(values)
```

Use `reduce()` carefully.

It can hide intent if the combining logic is not obvious.

---

# Comprehensions vs Functional Tools

Python programmers often prefer comprehensions over `map()` and `filter()`.

Example with `map()`:

```python
names = ["Ada", "Linus", "Grace"]
normalized = list(map(lambda name: name.casefold(), names))
```

Comprehension:

```python
normalized = [name.casefold() for name in names]
```

The comprehension is usually easier to read.

Example with `filter()`:

```python
active = list(filter(lambda user: user.is_active, users))
```

Comprehension:

```python
active = [user for user in users if user.is_active]
```

The comprehension makes the data flow visible:

```text
make a list of user
for each user in users
if user is active
```

Functional tools are part of Python.

But Python style values readability over forcing a particular paradigm.

---

# Function Composition

Function composition means connecting functions so the output of one becomes the input of another.

Example:

```python
def strip_text(text):
    return text.strip()

def normalize_case(text):
    return text.casefold()

def remove_spaces(text):
    return text.replace(" ", "")

value = remove_spaces(normalize_case(strip_text("  Hello World  ")))
print(value)
```

Output:

```text
helloworld
```

The nested call works, but it reads inside-out.

You can make the steps explicit:

```python
text = "  Hello World  "
text = strip_text(text)
text = normalize_case(text)
text = remove_spaces(text)

print(text)
```

This is often clearer in Python.

Functional composition is useful, but do not sacrifice readability.

---

# A Simple Compose Function

You can write a function that composes two functions.

Example:

```python
def compose(first, second):
    def composed(value):
        return second(first(value))

    return composed

strip_then_lower = compose(str.strip, str.casefold)

print(strip_then_lower("  PYTHON  "))
```

Output:

```text
python
```

This combines higher-order functions and closures.

`compose` receives functions.

It returns a new function.

The returned function remembers `first` and `second`.

This is powerful, but it can become abstract quickly.

Use named intermediate steps when that is clearer.

---

# Pipelines

A pipeline applies a sequence of transformations.

Example:

```python
def strip_text(text):
    return text.strip()

def normalize_case(text):
    return text.casefold()

def remove_spaces(text):
    return text.replace(" ", "")

def apply_pipeline(value, functions):
    for function in functions:
        value = function(value)

    return value

pipeline = [strip_text, normalize_case, remove_spaces]

print(apply_pipeline("  Hello World  ", pipeline))
```

Output:

```text
helloworld
```

This pattern is useful when transformations are configurable.

Examples:

* Data cleaning.
* Request processing.
* Validation steps.
* Text normalization.
* Build steps.

Do not build a pipeline abstraction when a few direct lines are clearer.

---

# Predicates

A predicate is a function that returns a truthy or falsy value.

Example:

```python
def is_non_empty(text):
    return bool(text.strip())
```

Predicates are often used for filtering:

```python
values = ["Ada", "", "  ", "Grace"]
clean = [value for value in values if is_non_empty(value)]

print(clean)
```

Output:

```text
['Ada', 'Grace']
```

Good predicate names often start with:

* `is_`
* `has_`
* `can_`
* `should_`

Examples:

```python
is_valid_email(email)
has_permission(user, action)
can_retry(response)
should_skip(record)
```

Clear predicates make conditionals and filters easier to read.

---

# Partial Application

Partial application means fixing some arguments of a function ahead of time.

Python provides `functools.partial`.

Example:

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))
print(cube(5))
```

Output:

```text
25
125
```

`square` is a callable object that remembers:

```text
exponent = 2
```

You can also write this with closures:

```python
def make_power(exponent):
    def apply(base):
        return base ** exponent

    return apply
```

Both approaches create specialized behavior from a general function.

---

# Closures In Functional Style

Closures are a foundation for functional programming in Python.

Example:

```python
def minimum_length(length):
    def validator(text):
        return len(text) >= length

    return validator

at_least_8 = minimum_length(8)

print(at_least_8("python"))
print(at_least_8("pythonista"))
```

Output:

```text
False
True
```

The returned `validator` function remembers `length`.

This lets you create configured functions.

Use cases:

* Validators.
* Formatters.
* Filters.
* Sorting keys.
* Authorization checks.
* Retry policies.

---

# Decorators As Functional Programming

Decorators are covered later in depth, but they are a functional programming idea.

A decorator is a function that takes a function and returns a function.

Simple example:

```python
def trace(function):
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        return function(*args, **kwargs)

    return wrapper

def greet(name):
    return f"Hello, {name}"

greet = trace(greet)

print(greet("Ada"))
```

Output:

```text
calling greet
Hello, Ada
```

This is a higher-order function.

It receives behavior, wraps it, and returns new behavior.

The later decorators chapter will explain the syntax and production details.

For now, understand the functional idea:

```text
function in -> modified function out
```

---

# Functional Style And Testing

Pure functions are easy to test because they do not need external setup.

Example:

```python
def calculate_discount(price, rate):
    return price * rate
```

Test:

```python
assert calculate_discount(100, 0.2) == 20
```

Compare a side-effect-heavy function:

```python
def apply_discount_to_database(order_id, rate):
    order = database.get_order(order_id)
    order.discount = order.price * rate
    database.save(order)
```

This needs:

* A database.
* A real or fake order.
* Setup.
* Cleanup.
* Error handling.

Better design often separates pure calculation from side effects:

```python
def calculate_discount(price, rate):
    return price * rate

def apply_discount_to_database(order_id, rate):
    order = database.get_order(order_id)
    order.discount = calculate_discount(order.price, rate)
    database.save(order)
```

The pure part is easy to test.

The side-effect part is smaller.

---

# Functional Style And State

Functional style often reduces shared mutable state.

Stateful approach:

```python
total = 0

def add_to_total(value):
    global total
    total += value
```

Pure-style approach:

```python
def add(total, value):
    return total + value
```

The second version makes state flow explicit:

```python
total = add(total, 10)
total = add(total, 5)
```

This is more verbose, but easier to reason about.

Global mutable state is convenient at first.

It becomes difficult when programs grow.

---

# Functional Style In Real Python

Python's standard style is pragmatic.

Python does not require pure functions.

Python does not ban mutation.

Python does not force map/filter/reduce.

Python encourages readable code.

Good Python functional style usually means:

* Use small focused functions.
* Avoid hidden mutation when practical.
* Prefer clear data flow.
* Use comprehensions for simple transformations.
* Use higher-order functions when they remove duplication.
* Use named functions when lambdas become unclear.
* Keep side effects at the edges of the program.

Functional style becomes un-Pythonic when it hides simple logic behind too many abstractions.

---

# Example: Validation Pipeline

Suppose we need to validate a username.

Direct version:

```python
def validate_username(username):
    if not username:
        return "username is required"

    if len(username) < 3:
        return "username is too short"

    if not username.isidentifier():
        return "username must be a valid identifier"

    return None
```

Functional-style validators:

```python
def required(value):
    if not value:
        return "value is required"

    return None

def minimum_length(length):
    def validator(value):
        if len(value) < length:
            return f"value must be at least {length} characters"

        return None

    return validator

def identifier(value):
    if not value.isidentifier():
        return "value must be a valid identifier"

    return None

def validate(value, validators):
    for validator in validators:
        error = validator(value)

        if error:
            return error

    return None

validators = [required, minimum_length(3), identifier]

print(validate("py", validators))
print(validate("python", validators))
```

This style is useful when validation rules are reused or configured dynamically.

For one-off validation, the direct version may be better.

---

# Example: Data Transformation

Raw input:

```python
names = [" Ada ", "", "LINUS", "  Grace  "]
```

Loop version:

```python
cleaned = []

for name in names:
    normalized = name.strip().title()

    if normalized:
        cleaned.append(normalized)
```

Comprehension version:

```python
cleaned = [
    name.strip().title()
    for name in names
    if name.strip()
]
```

The comprehension is concise, but it repeats `name.strip()`.

Clear function version:

```python
def normalize_name(name):
    return name.strip().title()

cleaned = [
    normalize_name(name)
    for name in names
    if name.strip()
]
```

This is often the best balance:

* The transformation has a name.
* The comprehension remains readable.
* The helper function is testable.

---

# When Functional Style Helps

Functional style helps when:

* You are transforming data.
* You can express behavior as small functions.
* You want to reduce hidden mutation.
* You need configurable behavior.
* You want testable units.
* You want to pass behavior into a reusable process.
* You are building pipelines.

Example:

```python
def process_records(records, normalize, is_valid):
    result = []

    for record in records:
        normalized = normalize(record)

        if is_valid(normalized):
            result.append(normalized)

    return result
```

The caller controls behavior:

```python
process_records(records, normalize_user, is_active_user)
```

This separates the framework of processing from the specific operations.

---

# When Functional Style Hurts

Functional style hurts when it becomes abstract without benefit.

Harder than necessary:

```python
result = reduce(
    lambda acc, fn: fn(acc),
    [str.strip, str.casefold, lambda text: text.replace(" ", "")],
    value,
)
```

Clearer:

```python
text = value.strip()
text = text.casefold()
text = text.replace(" ", "")
```

Functional style also hurts when:

* Lambdas become long.
* Composition hides order of execution.
* Debugging intermediate values becomes difficult.
* A simple loop would be clearer.
* The team is not using that style consistently.

Clarity wins.

---

# Common Mistakes

## Misconception 1

### Functional programming means never using loops.

Functional programming emphasizes functions and transformations.

Python still uses loops heavily.

Often, a clear loop is better than a forced functional expression.

## Misconception 2

### Lambda is better than `def`.

Lambda is useful for small local expressions.

Use `def` when the behavior deserves a name or needs multiple steps.

## Misconception 3

### Pure functions are always more efficient.

Pure-style code may create new objects instead of mutating existing ones.

That can use more memory.

The benefit is often clarity, not raw speed.

## Misconception 4

### Side effects are bad.

Side effects are necessary.

Programs need to read input, write output, update state, and communicate with external systems.

The goal is to control side effects, not eliminate them.

## Misconception 5

### `map`, `filter`, and `reduce` are always more Pythonic.

In Python, comprehensions and loops are often clearer.

Use the tool that communicates intent best.

## Misconception 6

### Passing functions is advanced magic.

Functions are objects.

Passing a function is the same object-reference model used throughout Python.

## Misconception 7

### Functional style means unreadable one-liners.

Good functional style uses named functions, clear transformations, and controlled state.

Unreadable one-liners are a style problem, not a requirement.

---

# Real-world Usage

## Sorting

```python
users = sorted(users, key=lambda user: user.created_at)
```

## Validation

```python
validators = [required, minimum_length(8), contains_digit]
```

## Data Cleaning

```python
cleaned = [normalize(record) for record in records if is_valid(record)]
```

## Command Dispatch

```python
handlers = {
    "create": create_item,
    "delete": delete_item,
}

handlers[command]()
```

## Testing

Pure helper functions make business rules easier to test.

## Configuration

Higher-order functions can create customized behavior:

```python
retry_three_times = make_retry_policy(max_attempts=3)
```

## Decorators

Decorators wrap functions to add behavior such as logging, timing, authorization, caching, and validation.

---

# Internal Mechanics

Functional programming in Python relies on ordinary object mechanics.

There is no separate functional runtime.

When you pass a function:

```python
apply_twice(double, 3)
```

Python passes a reference to the function object.

Inside the callee:

```python
function(value)
```

Python calls the object referenced by `function`.

When you return a function:

```python
return multiply
```

Python returns a reference to the function object.

When that function is a closure, it carries references to needed enclosing variables.

Core model:

```text
name -> function object
function object -> callable
call -> frame
closure -> function object + remembered environment
```

This is the same model developed across the functions part.

---

# Concept Connections

Functional programming connects to earlier chapters:

* Names and references: function names refer to function objects.
* Mutability: pure-style code often avoids mutating inputs.
* Expressions: function calls are expressions that return objects.
* Conditionals: predicates drive filtering and validation.
* Loops: many functional transformations can be written as loops.
* Functions: functional programming depends on function objects.
* Scope: returned functions use scope rules.
* Closures: higher-order functions often return closures.
* Call stack: every function call still creates a frame.
* Recursion: recursion is one functional technique, but not the only one.

Functional programming prepares you for:

* Comprehensions.
* Iterators.
* Generators.
* Decorators.
* Data pipelines.
* Testing pure business logic.
* Designing reusable APIs.

---

# Active Recall

## Easy Recall Questions

1. What does it mean that functions are first-class objects?
2. What is a higher-order function?
3. What is a pure function?
4. What is a side effect?
5. What does a lambda expression create?
6. What does `map()` do conceptually?
7. What does `filter()` do conceptually?
8. What does `reduce()` do conceptually?
9. Why are comprehensions often preferred in Python?
10. What is function composition?

## Deep Understanding Questions

1. Why does passing a function as an argument follow the same model as passing any other object?
2. Why are pure functions easier to test?
3. Why can avoiding mutation make code easier to reason about?
4. Why can functional style become harder to read when overused?
5. How do closures support configured functions?
6. Why does Python not require functional style even though it supports it?

## Predict-the-Output Questions

### Question 1

```python
def double(number):
    return number * 2

operation = double
print(operation(4))
```

### Question 2

```python
def apply(function, value):
    return function(value)

def shout(text):
    return text.upper()

print(apply(shout, "python"))
```

### Question 3

```python
def make_adder(amount):
    def add(value):
        return value + amount

    return add

add_10 = make_adder(10)
print(add_10(5))
```

### Question 4

```python
values = [" Ada ", "", "Linus"]
result = [value.strip() for value in values if value.strip()]
print(result)
```

### Question 5

```python
def add_item(items, item):
    items.append(item)
    return items

values = ["a"]
updated = add_item(values, "b")

print(values)
print(updated)
```

---

# Practical Exercises

## Exercise 1

Write a function called `apply_twice(function, value)`.

Test it with:

* A function that doubles a number.
* A function that adds one.
* A function that strips whitespace from a string.

## Exercise 2

Write `make_multiplier(factor)` so it returns a function.

Use it to create:

* `double`
* `triple`
* `times_ten`

## Exercise 3

Create a dispatch dictionary for four commands:

* `create`
* `read`
* `update`
* `delete`

Each command should call a different function.

## Exercise 4

Write a pure function that normalizes a username.

It should:

* Strip whitespace.
* Convert to lowercase.
* Replace spaces with underscores.

## Exercise 5

Write both versions of a function that adds an item to a list:

* One mutating version.
* One pure-style version that returns a new list.

Explain the tradeoff.

## Exercise 6

Use a list comprehension to transform a list of names into normalized names.

Then write the same transformation with `map()`.

Compare readability.

## Exercise 7

Use a list comprehension to filter numbers greater than 10.

Then write the same filtering with `filter()`.

Compare readability.

## Exercise 8

Use `functools.reduce()` to multiply a list of numbers.

Then write the same logic with a loop.

Explain which one is clearer.

## Exercise 9

Create a validation pipeline using a list of validator functions.

Each validator should return either:

* `None` for success.
* An error message string for failure.

## Exercise 10

Write a simple `compose(first, second)` function.

Use it to combine two text transformations.

---

# Summary

In this chapter we learned:

* Functions are first-class objects in Python.
* Function names are names bound to function objects.
* Functions can be assigned to variables.
* Functions can be passed as arguments.
* Functions can be returned from functions.
* Higher-order functions accept or return functions.
* Pure functions return values based only on inputs and avoid side effects.
* Side effects are necessary but should be controlled.
* Lambda expressions create small anonymous functions.
* `map()` transforms items.
* `filter()` keeps items based on a predicate.
* `reduce()` combines items into one value.
* Comprehensions are often clearer than `map()` and `filter()` in Python.
* Function composition connects transformations.
* Closures support configured functions.
* Functional style is useful when it clarifies data flow.
* Functional style is harmful when it hides simple logic behind abstraction.

Core model:

```text
function object
    |
    v
can be named, passed, returned, stored, and called
    |
    v
higher-order behavior
```

Functional programming in Python is not a separate world.

It is the natural result of functions being objects.

---

# Preview of Part VII — Data Structures

The functions part is now complete.

We have studied:

* Function objects.
* Parameters and arguments.
* Scope and namespaces.
* Closures.
* Call stack and stack frames.
* Recursion.
* Functional programming.

Next, Part VII begins with Chapter 26, Lists.

So far, we have used lists as convenient containers in examples.

Now we will study them directly:

* What lists are.
* Why lists exist.
* How list indexing works.
* How list mutation works.
* How list methods behave.
* How lists store references.
* How aliasing affects lists.
* How list growth works conceptually.
* Common list mistakes.

The transition is direct:

```text
functions organize behavior
data structures organize objects
lists are Python's most common mutable sequence
```

Part VII begins the detailed study of how Python stores and organizes groups of objects.
