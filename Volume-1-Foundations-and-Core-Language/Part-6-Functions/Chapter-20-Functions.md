# Chapter 20 — Functions

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a function is.
* Why functions exist.
* How function definitions work.
* How function calls work.
* Why functions are objects.
* What parameters are.
* What arguments are.
* How return values work.
* Why a function returns `None` by default.
* How local names work at a high level.
* How function calls create frames.
* How arguments are passed by object reference.
* How mutation and rebinding behave inside functions.
* What side effects are.
* How to design clear functions.
* Common mistakes with return values, mutable arguments, defaults, and scope assumptions.

Functions are Python's primary tool for packaging behavior.

---

# Concept Overview

A function is a reusable block of code.

Example:

```python
def greet(name):
    return f"Hello, {name}"
```

Calling the function:

```python
message = greet("Ada")
print(message)
```

Output:

```text
Hello, Ada
```

Functions let you:

* Name behavior.
* Reuse logic.
* Accept inputs.
* Produce outputs.
* Isolate complexity.
* Organize programs.
* Test smaller units.

Core model:

```text
arguments
    |
    v
function call
    |
    v
function frame
    |
    v
return value
```

---

# Why Functions Exist

Without functions, programs become repeated sequences of statements.

Example:

```python
name = "Ada"
print(f"Hello, {name}")

name = "Grace"
print(f"Hello, {name}")

name = "Linus"
print(f"Hello, {name}")
```

With a function:

```python
def greet(name):
    print(f"Hello, {name}")

greet("Ada")
greet("Grace")
greet("Linus")
```

The behavior is named once and reused.

Functions exist to manage complexity.

They let us build larger programs from smaller units.

---

# Function Definition

Syntax:

```python
def function_name(parameters):
    body
```

Example:

```python
def add(a, b):
    return a + b
```

Parts:

```text
def        -> begins function definition
add        -> function name
a, b       -> parameters
return     -> sends result back to caller
body       -> indented block
```

The function body does not run when the function is defined.

It runs when the function is called.

---

# Defining a Function Creates a Function Object

Functions are objects.

Example:

```python
def greet():
    print("hello")

print(type(greet))
```

Output:

```text
<class 'function'>
```

When Python executes a `def` statement, it creates a function object and binds the function name to that object.

Conceptually:

```text
greet ─────▶ function object
```

The function object contains:

* A code object.
* A reference to global namespace.
* Default argument values, if any.
* Metadata such as its name.

This connects to Chapter 08.

---

# Function Body Does Not Run at Definition Time

Example:

```python
def show():
    print("inside")

print("outside")
```

Output:

```text
outside
```

The function body did not run.

Only the function object was created.

To run the body, call the function:

```python
show()
```

Now output:

```text
inside
```

This distinction prevents confusion about when code executes.

---

# Function Calls

A function call executes a function.

Syntax:

```python
function_name(arguments)
```

Example:

```python
def add(a, b):
    return a + b

result = add(2, 3)
print(result)
```

Output:

```text
5
```

Call steps:

```text
1. Evaluate the function object.
2. Evaluate argument expressions.
3. Create a new function frame.
4. Bind parameter names to argument objects.
5. Execute the function body.
6. Return a value.
7. Resume caller.
```

---

# Parameters and Arguments

Parameters are names in the function definition.

Arguments are values supplied during a call.

Example:

```python
def greet(name):
    return f"Hello, {name}"

greet("Ada")
```

Here:

```text
name    -> parameter
"Ada"   -> argument
```

During the call:

```text
name ─────▶ str object "Ada"
```

The parameter is a local name inside the function frame.

---

# Argument Expressions Are Evaluated First

Example:

```python
def show(value):
    print(value)

x = 10
show(x + 5)
```

Evaluation:

```text
1. Evaluate x + 5 -> int object 15.
2. Call show with that object.
3. Bind parameter value -> int object 15.
```

Output:

```text
15
```

Arguments are expressions.

They are evaluated before the function receives them.

---

# Return Values

`return` sends a value back to the caller.

Example:

```python
def square(x):
    return x * x

result = square(5)
print(result)
```

Output:

```text
25
```

The expression:

```python
x * x
```

is evaluated.

The result object is returned.

The caller can bind that object to a name.

---

# return Ends the Function

When Python executes `return`, the function call ends.

Example:

```python
def classify(number):
    if number < 0:
        return "negative"

    return "non-negative"
```

If `number < 0`, the first return runs.

The function exits immediately.

The later return is skipped.

This is common in guard-style function design.

---

# Functions Return None by Default

If a function does not execute a `return` with a value, it returns `None`.

Example:

```python
def say_hi():
    print("hi")

result = say_hi()

print(result)
```

Output:

```text
hi
None
```

The function printed as a side effect.

Its return value was `None`.

This is a common source of bugs.

Printing is not returning.

---

# print Is Not return

Example:

```python
def add(a, b):
    print(a + b)

result = add(2, 3)
print(result)
```

Output:

```text
5
None
```

The function printed `5`, but returned `None`.

Correct if caller needs the value:

```python
def add(a, b):
    return a + b
```

Use `print()` for output.

Use `return` for passing values back to code.

---

# Local Names

Names assigned inside a function are local to that function call.

Example:

```python
def compute():
    result = 10 + 5
    return result

print(compute())
```

The name `result` exists inside the function frame.

Outside:

```python
print(result)
```

raises:

```text
NameError
```

Detailed scope rules are Chapter 21.

For now:

```text
function calls create local name spaces
```

---

# Function Frames

Each function call creates a frame.

A frame contains runtime state for that call:

* Parameter bindings
* Local names
* Current instruction position
* Evaluation stack
* Reference to global namespace

Example:

```python
def add(a, b):
    total = a + b
    return total

result = add(2, 3)
```

During the call:

```text
add frame
├── a ─────▶ int object 2
├── b ─────▶ int object 3
└── total ─▶ int object 5
```

When the function returns, the frame is no longer active.

---

# Multiple Calls Create Multiple Frames

Each call gets its own frame.

Example:

```python
def double(x):
    return x * 2

print(double(2))
print(double(5))
```

First call:

```text
double frame
└── x ─────▶ int object 2
```

Second call:

```text
double frame
└── x ─────▶ int object 5
```

The same function object is reused.

Each call has separate runtime state.

---

# Call Stack

When functions call functions, frames stack.

Example:

```python
def a():
    b()

def b():
    c()

def c():
    print("inside c")

a()
```

While `c()` is running:

```text
c frame
b frame
a frame
module frame
```

The call stack will be covered more deeply in Chapter 23.

For now, understand:

> Function calls create frames, and nested calls create a stack of active frames.

---

# Positional Arguments

Positional arguments are matched by position.

Example:

```python
def describe(name, age):
    return f"{name} is {age}"

print(describe("Ada", 36))
```

Output:

```text
Ada is 36
```

Binding:

```text
name ─────▶ "Ada"
age ──────▶ 36
```

Order matters.

```python
describe(36, "Ada")
```

binds:

```text
name ─────▶ 36
age ──────▶ "Ada"
```

which is likely wrong.

---

# Keyword Arguments

Keyword arguments are matched by parameter name.

Example:

```python
def describe(name, age):
    return f"{name} is {age}"

print(describe(age=36, name="Ada"))
```

Output:

```text
Ada is 36
```

Keyword arguments improve clarity when:

* There are multiple arguments.
* Several arguments have the same type.
* Order may be confusing.
* Defaults are involved.

Example:

```python
connect(host="localhost", port=5432, timeout=30)
```

is clearer than:

```python
connect("localhost", 5432, 30)
```

---

# Default Arguments

Parameters can have default values.

Example:

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}"

print(greet("Ada"))
print(greet("Ada", "Welcome"))
```

Output:

```text
Hello, Ada
Welcome, Ada
```

If the caller does not provide `greeting`, the default is used.

Defaults make common cases concise.

Use them when a value is truly optional and has a sensible default.

---

# Default Arguments Are Evaluated Once

Default argument values are evaluated when the function is defined.

This matters for mutable objects.

Problem:

```python
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item("a"))
print(add_item("b"))
```

Output:

```text
['a']
['a', 'b']
```

The same list object is reused across calls.

Correct pattern:

```python
def add_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items
```

---

# Passing Object References

Function calls bind parameter names to argument objects.

Example:

```python
def show(value):
    print(value)

x = [1, 2]
show(x)
```

During the call:

```text
global:
x ───────▶ list object [1, 2]
             ▲
             │
show frame:
value ───────┘
```

The list is not copied by default.

The parameter name refers to the same object.

---

# Rebinding a Parameter

Rebinding a parameter does not affect the caller's name.

Example:

```python
def reset(value):
    value = []

items = [1, 2]
reset(items)
print(items)
```

Output:

```text
[1, 2]
```

Inside the function, `value = []` rebinds the local name.

It does not rebind `items`.

---

# Mutating an Argument Object

Mutating a passed object can affect the caller-visible object.

Example:

```python
def append_item(values, item):
    values.append(item)

items = [1, 2]
append_item(items, 3)
print(items)
```

Output:

```text
[1, 2, 3]
```

The function mutated the list object.

The caller's name still refers to that same object.

This is not pass-by-reference in the C++ sense.

It is parameter binding to object references.

---

# Side Effects

A function has a side effect when it changes something outside its return value.

Examples:

* Printing
* Mutating an argument
* Writing a file
* Updating a database
* Sending a request
* Modifying global state

Example:

```python
def add_item(values, item):
    values.append(item)
```

This function returns `None` but mutates `values`.

Side effects are not always bad.

They should be intentional and clear.

---

# Pure Functions

A pure function's output depends only on its inputs and it has no side effects.

Example:

```python
def add(a, b):
    return a + b
```

Given the same `a` and `b`, it returns the same result.

It does not mutate anything, print, or depend on external state.

Pure functions are easier to test and reason about.

Not every function can or should be pure.

But pure logic is valuable when possible.

---

# Procedure-Style Functions

Some functions are primarily actions.

Example:

```python
def print_report(report):
    print(report.title)
    print(report.summary)
```

This function's purpose is output.

It may return `None`.

That is acceptable if clear.

The mistake is not returning `None`.

The mistake is returning `None` accidentally when callers expect a value.

---

# Function Naming

Function names should describe behavior.

Good names:

```python
calculate_total
normalize_email
is_valid_email
send_message
load_config
```

Predicate functions should read like questions:

```python
is_active(user)
has_permission(user, action)
can_retry(request)
```

Avoid vague names:

```python
do_stuff
handle
process_data
```

unless the context makes the action specific.

---

# Function Size

A function should usually do one coherent thing.

Too much:

```text
load data
validate data
transform data
write report
send email
update database
```

Better:

```python
data = load_data(path)
validated = validate_data(data)
report = build_report(validated)
save_report(report)
send_report_email(report)
```

Each function has a focused purpose.

This improves testing, reuse, and debugging.

---

# Returning Multiple Values

Python functions can return multiple values using tuples.

Example:

```python
def divide_with_remainder(a, b):
    return a // b, a % b

quotient, remainder = divide_with_remainder(17, 5)

print(quotient)
print(remainder)
```

Output:

```text
3
2
```

The function returns one tuple object.

The caller unpacks it into two names.

Conceptually:

```text
return (3, 2)
```

---

# Early Returns

Early returns can simplify logic.

Example:

```python
def classify_age(age):
    if age < 0:
        return "invalid"

    if age < 18:
        return "minor"

    return "adult"
```

This avoids deep nesting.

Each return handles a case and exits.

Use early returns when they make control flow clearer.

---

# Documentation Strings

A docstring documents a function.

Example:

```python
def add(a, b):
    """Return the sum of a and b."""
    return a + b
```

The docstring is a string literal at the start of the function body.

It becomes function documentation.

You can inspect it:

```python
print(add.__doc__)
```

Docstrings are useful for public functions, APIs, and non-obvious behavior.

Do not write noisy docstrings that merely repeat obvious code.

---

# Type Hints Preview

Python supports type hints.

Example:

```python
def add(a: int, b: int) -> int:
    return a + b
```

Type hints do not enforce types at runtime by default.

They help:

* Readers
* Editors
* Static type checkers
* Documentation

We will study type hints in the software engineering volume.

For now, read the example as:

```text
add expects two ints and returns an int
```

---

# Common Mistakes

## Misconception 1

### Defining a function runs its body.

Defining a function creates a function object.

The body runs when the function is called.

---

## Misconception 2

### print returns the printed value.

`print()` returns `None`.

Printing is a side effect.

Use `return` to send a value back to the caller.

---

## Misconception 3

### Parameters are copied objects.

Parameters are local names bound to argument objects.

Objects are not copied by default.

---

## Misconception 4

### Rebinding a parameter changes the caller's variable.

Rebinding a parameter changes only the local name.

It does not rebind the caller's name.

---

## Misconception 5

### Mutating an argument is impossible.

If the argument object is mutable, the function can mutate it.

Callers will see the mutation if they still reference that object.

---

## Misconception 6

### Mutable default arguments are safe.

Mutable defaults are shared across calls.

Use `None` as a sentinel when a fresh mutable object is needed.

---

## Misconception 7

### A function should always print its result.

Most reusable functions should return values.

Printing belongs at boundaries such as CLI output, logs, or reports.

---

# Real-world Usage

## Reusable Calculations

```python
def calculate_total(price, tax_rate):
    return price + price * tax_rate
```

This packages a calculation and returns a value.

---

## Validation

```python
def is_valid_email(email):
    return bool(email) and "@" in email
```

Predicate functions make control flow clearer:

```python
if is_valid_email(email):
    ...
```

---

## Normalization

```python
def normalize_email(email):
    return email.strip().casefold()
```

This function returns a new string and does not mutate input.

---

## Mutating APIs

```python
def add_error(errors, message):
    errors.append(message)
```

This mutates the passed list.

That may be acceptable if the name and documentation make it clear.

---

## Orchestration

Functions can organize workflows:

```python
def process_file(path):
    text = load_text(path)
    records = parse_records(text)
    valid_records = validate_records(records)
    return build_report(valid_records)
```

Each helper function handles a smaller task.

---

# Concept Connections

This chapter builds on:

```text
Chapter 08:
Function calls create frames.

Chapter 09:
Functions are objects.

Chapter 10:
Parameters are local names bound to argument objects.

Chapter 11:
Mutation through parameters can affect caller-visible objects.

Chapter 17:
Function calls are expressions and return objects.

Chapter 18:
Function bodies contain control flow.
```

It prepares:

```text
Scope:
    where names are looked up inside functions

Closures:
    functions can capture surrounding names

Call Stack:
    nested function calls create stacked frames

Modules:
    functions organize code across files
```

Core model:

```text
function object
    |
call with arguments
    |
new frame
    |
parameters bound to objects
    |
body executes
    |
return object
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Function | Reusable callable object |
| Function definition | `def` statement that creates a function object |
| Function call | Execution of a function object |
| Parameter | Local name in function definition |
| Argument | Object supplied in a function call |
| Return value | Object sent back to caller |
| Frame | Runtime context for one function call |
| Local name | Name bound inside a function frame |
| Side effect | Observable effect beyond return value |
| Default argument | Value used when caller omits an argument |
| Docstring | Function documentation string |

Core rules:

```text
def creates a function object.
Calling executes the function body.
Arguments are evaluated before the call.
Parameters are local names.
Functions return None unless they return another value.
Rebinding parameters does not rebind caller names.
Mutating argument objects can affect caller-visible state.
Mutable defaults are shared across calls.
```

---

# Active Recall

## Easy Recall Questions

1. What does `def` do?
2. Does a function body run when the function is defined?
3. What is a parameter?
4. What is an argument?
5. What does `return` do?
6. What does a function return by default?
7. What does `print()` return?
8. What is a function frame?
9. What is a side effect?
10. Why are mutable default arguments risky?

---

## Deep Understanding Questions

1. Why are functions objects?
2. Why does each function call need a new frame?
3. Why are argument expressions evaluated before the function call?
4. Why does rebinding a parameter not affect the caller's name?
5. Why can mutating a parameter affect the caller-visible object?
6. Why is returning usually better than printing for reusable functions?
7. Why can early returns improve readability?
8. Why are focused functions easier to test?
9. Why are keyword arguments often clearer than positional arguments?
10. Why does the mutable default argument bug happen?

---

## Explain In Your Own Words

1. Explain function definition versus function call.
2. Explain parameters versus arguments.
3. Explain what happens during a function call.
4. Explain return values.
5. Explain why `print` is not `return`.
6. Explain parameter rebinding.
7. Explain argument mutation.

---

## Predict-the-Output Questions

### Question 1

```python
def greet():
    print("hello")

print("before")
```

Answer:

```text
before
```

Reason:

The function body does not run until the function is called.

---

### Question 2

```python
def add(a, b):
    print(a + b)

result = add(2, 3)
print(result)
```

Answer:

```text
5
None
```

Reason:

The function prints but does not return a value, so it returns `None`.

---

### Question 3

```python
def reset(value):
    value = []

items = [1, 2]
reset(items)
print(items)
```

Answer:

```text
[1, 2]
```

Reason:

The local parameter name is rebound. The caller's name is unchanged.

---

### Question 4

```python
def append_item(values):
    values.append(3)

items = [1, 2]
append_item(items)
print(items)
```

Answer:

```text
[1, 2, 3]
```

Reason:

The function mutates the list object passed as an argument.

---

### Question 5

```python
def collect(value, values=[]):
    values.append(value)
    return values

print(collect("a"))
print(collect("b"))
```

Answer:

```text
['a']
['a', 'b']
```

Reason:

The default list is shared across calls.

---

### Question 6

```python
def classify(number):
    if number < 0:
        return "negative"
    return "non-negative"

print(classify(-1))
```

Answer:

```text
negative
```

Reason:

The first return exits the function.

---

# Mental Model Questions

1. Draw a function name pointing to a function object.
2. Draw a call frame for `add(2, 3)`.
3. Draw parameter names bound to argument objects.
4. Draw a function returning an object to the caller.
5. Draw rebinding a parameter.
6. Draw mutating a list argument.
7. Draw two separate calls to the same function creating separate frames.

---

# Practical Exercises

## Exercise 1

Write a function:

```python
def square(number):
    ...
```

It should return the square of `number`.

Explain the parameter, argument, and return value.

---

## Exercise 2

Fix this function:

```python
def add(a, b):
    print(a + b)
```

Change it so callers can use the result in another expression.

---

## Exercise 3

Compare rebinding and mutation:

```python
def rebind(values):
    values = []

def mutate(values):
    values.clear()

items = [1, 2, 3]
rebind(items)
print(items)
mutate(items)
print(items)
```

Predict and explain the output.

---

## Exercise 4

Rewrite a mutable default:

```python
def add_error(message, errors=[]):
    errors.append(message)
    return errors
```

Use `None` as the default and create a new list when needed.

---

## Exercise 5

Write a predicate function:

```python
def is_even(number):
    ...
```

It should return a boolean expression directly.

---

## Exercise 6

Write a function with early returns:

```python
def describe_age(age):
    ...
```

Return:

```text
"invalid" for negative ages
"minor" for ages below 18
"adult" otherwise
```

---

## Exercise 7

Design a small workflow with focused functions:

```python
normalize_email(email)
is_valid_email(email)
format_welcome_message(email)
```

Each function should do one clear thing.

---

# Summary

In this chapter we learned:

* Functions package reusable behavior.
* `def` creates a function object and binds it to a name.
* Function bodies run when called, not when defined.
* Parameters are local names in the function definition.
* Arguments are objects supplied by the caller.
* Argument expressions are evaluated before the call.
* Function calls create frames.
* Each call has separate local runtime state.
* `return` sends an object back to the caller.
* Functions return `None` by default.
* `print` is a side effect, not a return value.
* Rebinding a parameter does not affect the caller's name.
* Mutating a passed mutable object can affect caller-visible state.
* Mutable default arguments are shared across calls.
* Side effects should be intentional.
* Clear functions have focused names, focused behavior, and clear inputs/outputs.

Core model:

```text
function object
    |
    v
call(arguments)
    |
    v
new frame with parameter bindings
    |
    v
body execution
    |
    v
return value
```

Functions are where objects, names, expressions, and control flow become reusable behavior.

---

# Preview of Chapter 21

Next we study scope.

Functions create local names.

But Python programs also have global names, built-in names, and eventually enclosing names.

Chapter 21 will answer:

* Where does Python look for a name?
* What is local scope?
* What is global scope?
* What is built-in scope?
* What is LEGB?
* Why does assigning inside a function affect name lookup?
* When should `global` be avoided?

Scope is the next layer of the function model.
