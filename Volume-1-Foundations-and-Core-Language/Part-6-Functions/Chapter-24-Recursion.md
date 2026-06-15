# Chapter 24 — Recursion

---

# Learning Objectives

By the end of this chapter, you should understand:

* What recursion is.
* Why a function can call itself.
* How recursive calls create separate stack frames.
* Why every recursive algorithm needs a base case.
* What a recursive case is.
* How to trace recursive execution step by step.
* How return values move back through recursive calls.
* How recursion differs from iteration.
* When recursion is useful.
* When recursion is risky in Python.
* What `RecursionError` means.
* Why Python does not optimize tail recursion.
* How to rewrite some recursive algorithms iteratively.
* How recursion applies to nested data and tree-shaped structures.
* Common recursion mistakes.

The previous chapter introduced the call stack.

This chapter uses that model to understand recursive functions deeply.

---

# Concept Overview

Recursion happens when a function calls itself.

Example:

```python
def countdown(n):
    if n == 0:
        print("done")
        return

    print(n)
    countdown(n - 1)

countdown(3)
```

Output:

```text
3
2
1
done
```

At first, this may look strange.

How can a function call itself before it has finished?

The answer is the call stack.

Each call creates a new frame.

So `countdown(3)`, `countdown(2)`, `countdown(1)`, and `countdown(0)` are separate active calls, each with its own local name `n`.

Conceptually:

```text
countdown(0) frame
countdown(1) frame
countdown(2) frame
countdown(3) frame
module frame
```

The same function object is reused.

The frames are different.

---

# Why Recursion Exists

Some problems contain smaller versions of themselves.

Examples:

```text
A countdown from n is:
    print n
    then countdown from n - 1

A folder contains:
    files
    and maybe more folders

A tree contains:
    a value
    and child trees

A nested list contains:
    values
    and maybe more nested lists
```

Recursion is useful when a problem naturally decomposes into:

```text
solve this problem
by solving a smaller version of the same problem
```

That is the key idea.

Recursion is not about making code clever.

It is about matching the structure of a problem.

---

# The Two Required Parts

Every recursive function needs two parts:

* A base case.
* A recursive case.

The base case stops recursion.

The recursive case moves toward the base case.

Example:

```python
def countdown(n):
    if n == 0:
        print("done")
        return

    print(n)
    countdown(n - 1)
```

Base case:

```python
if n == 0:
    print("done")
    return
```

Recursive case:

```python
countdown(n - 1)
```

The recursive call uses a smaller input.

That smaller input eventually reaches `0`.

Without that movement, recursion does not terminate.

---

# Base Case

A base case is the condition where the function can answer directly without another recursive call.

Example:

```python
def factorial(n):
    if n == 0:
        return 1

    return n * factorial(n - 1)
```

For factorial:

```text
0! = 1
```

That is the base case.

Python needs it because otherwise calls continue forever:

```python
factorial(3)
factorial(2)
factorial(1)
factorial(0)
```

At `factorial(0)`, Python can return directly.

Then the pending calls can finish.

---

# Recursive Case

The recursive case handles the general problem by calling the same function on a smaller or simpler problem.

For factorial:

```python
return n * factorial(n - 1)
```

This matches the mathematical definition:

```text
n! = n * (n - 1)!
```

For `factorial(4)`:

```text
factorial(4)
= 4 * factorial(3)
= 4 * 3 * factorial(2)
= 4 * 3 * 2 * factorial(1)
= 4 * 3 * 2 * 1 * factorial(0)
= 4 * 3 * 2 * 1 * 1
= 24
```

The recursive case must make progress.

Progress means the input gets closer to the base case.

---

# Recursive Calls Create New Frames

Consider:

```python
def factorial(n):
    if n == 0:
        return 1

    return n * factorial(n - 1)

result = factorial(3)
```

The active calls build up:

```text
factorial(3)
    needs factorial(2)
        needs factorial(1)
            needs factorial(0)
```

Stack frames:

```text
top -> factorial frame: n = 0
       factorial frame: n = 1
       factorial frame: n = 2
       factorial frame: n = 3
bottom -> module frame
```

Each frame has its own local `n`.

There is not one shared `n`.

This is one of the most important recursion facts.

Recursive calls reuse the same function object, but they do not reuse the same frame.

---

# Returning Back Up The Stack

Recursive calls build downward and return upward.

For:

```python
factorial(3)
```

Calls descend:

```text
factorial(3)
factorial(2)
factorial(1)
factorial(0)
```

Then returns ascend:

```text
factorial(0) returns 1
factorial(1) returns 1 * 1 = 1
factorial(2) returns 2 * 1 = 2
factorial(3) returns 3 * 2 = 6
```

Visual model:

```text
call downward
    factorial(3)
        factorial(2)
            factorial(1)
                factorial(0)

return upward
                1
            1
        2
    6
```

This is why recursion often feels like two phases:

* Going down until the base case.
* Coming back up with results.

---

# Tracing Recursion

When learning recursion, trace it manually.

Example:

```python
def show(n):
    if n == 0:
        print("base")
        return

    print("before", n)
    show(n - 1)
    print("after", n)

show(3)
```

Output:

```text
before 3
before 2
before 1
base
after 1
after 2
after 3
```

Why?

Each call pauses at the recursive call.

It resumes only after the deeper call returns.

Trace:

```text
show(3): print before 3
show(3): call show(2), wait
    show(2): print before 2
    show(2): call show(1), wait
        show(1): print before 1
        show(1): call show(0), wait
            show(0): print base, return
        show(1): print after 1, return
    show(2): print after 2, return
show(3): print after 3, return
```

This pattern appears often.

Code before the recursive call runs while descending.

Code after the recursive call runs while ascending.

---

# Recursion With Return Values

Some recursive functions print.

Many recursive functions return values.

Example:

```python
def sum_to(n):
    if n == 0:
        return 0

    return n + sum_to(n - 1)

print(sum_to(4))
```

Output:

```text
10
```

Expansion:

```text
sum_to(4)
= 4 + sum_to(3)
= 4 + 3 + sum_to(2)
= 4 + 3 + 2 + sum_to(1)
= 4 + 3 + 2 + 1 + sum_to(0)
= 4 + 3 + 2 + 1 + 0
= 10
```

The return value from the base case becomes part of the return value for the caller.

Then that result becomes part of the caller above it.

Return values climb back up the stack.

---

# Recursion With Lists

Recursion can process a list one item at a time.

Example:

```python
def sum_list(values):
    if not values:
        return 0

    first = values[0]
    rest = values[1:]
    return first + sum_list(rest)

print(sum_list([10, 20, 30]))
```

Output:

```text
60
```

Mental model:

```text
sum_list([10, 20, 30])
= 10 + sum_list([20, 30])
= 10 + 20 + sum_list([30])
= 10 + 20 + 30 + sum_list([])
= 10 + 20 + 30 + 0
= 60
```

This is clear as a learning example.

But in real Python, slicing creates new lists.

That can be inefficient for large lists.

For normal list summing, use:

```python
sum(values)
```

or an iterative loop when implementing manually.

---

# Recursion With Indexes

To avoid slicing, recursion can pass an index.

Example:

```python
def sum_from(values, index):
    if index == len(values):
        return 0

    return values[index] + sum_from(values, index + 1)

print(sum_from([10, 20, 30], 0))
```

Output:

```text
60
```

This avoids creating a new list at every call.

The recursive state is:

```text
which index are we processing?
```

The base case is:

```text
index reached the length of the list
```

The recursive case is:

```text
process current item and recurse with index + 1
```

---

# Recursion And Nested Lists

Recursion is especially useful for nested structures.

Example:

```python
data = [1, [2, [3, 4]], 5]
```

This structure contains values and lists inside lists.

An iterative single loop sees only the top level:

```python
for item in data:
    print(item)
```

Output:

```text
1
[2, [3, 4]]
5
```

A recursive function can descend into nested lists:

```python
def flatten(values):
    result = []

    for value in values:
        if isinstance(value, list):
            result.extend(flatten(value))
        else:
            result.append(value)

    return result

print(flatten([1, [2, [3, 4]], 5]))
```

Output:

```text
[1, 2, 3, 4, 5]
```

Why recursion fits:

```text
To flatten a list:
    for each value:
        if value is a list, flatten that list
        otherwise, keep the value
```

The same operation applies at every nesting level.

---

# Recursion And Trees

Trees are naturally recursive.

A tree node can contain child nodes.

Example:

```python
tree = {
    "name": "root",
    "children": [
        {
            "name": "src",
            "children": [
                {"name": "main.py", "children": []},
                {"name": "utils.py", "children": []},
            ],
        },
        {
            "name": "tests",
            "children": [
                {"name": "test_main.py", "children": []},
            ],
        },
    ],
}
```

Each node has the same shape:

```text
name
children
```

A recursive traversal:

```python
def print_tree(node, depth=0):
    indent = "  " * depth
    print(indent + node["name"])

    for child in node["children"]:
        print_tree(child, depth + 1)

print_tree(tree)
```

Output:

```text
root
  src
    main.py
    utils.py
  tests
    test_main.py
```

The function handles one node.

Then it calls itself for each child node.

This is the core recursive tree pattern.

---

# Recursion And Directories

File systems are tree-shaped.

A directory can contain files and directories.

Conceptually:

```python
def walk(path):
    for entry in path:
        if entry is a directory:
            walk(entry)
        else:
            process(entry)
```

This is why directory traversal is often recursive.

The problem structure is recursive:

```text
directory
    file
    directory
        file
        directory
            file
```

Even when library functions hide the recursion, the mental model remains useful.

---

# Recursive Search

Recursion can search nested structures.

Example:

```python
def contains_value(values, target):
    for value in values:
        if isinstance(value, list):
            if contains_value(value, target):
                return True
        elif value == target:
            return True

    return False

data = [1, [2, [3, 4]], 5]

print(contains_value(data, 4))
print(contains_value(data, 9))
```

Output:

```text
True
False
```

The base behavior is:

```text
if current value equals target, return True
if all values are checked and none match, return False
```

The recursive behavior is:

```text
if current value is nested, search inside it
```

---

# Mutual Recursion

Mutual recursion happens when functions call each other.

Example:

```python
def is_even(n):
    if n == 0:
        return True

    return is_odd(n - 1)

def is_odd(n):
    if n == 0:
        return False

    return is_even(n - 1)

print(is_even(4))
print(is_odd(4))
```

Output:

```text
True
False
```

Call chain:

```text
is_even(4)
is_odd(3)
is_even(2)
is_odd(1)
is_even(0)
```

Mutual recursion is less common in everyday Python.

It can be useful in parsers, state machines, and tree processing, but it can also make code harder to follow.

Use it only when the relationship between functions is genuinely recursive.

---

# Recursion vs Iteration

Many recursive functions can be rewritten with loops.

Recursive countdown:

```python
def countdown(n):
    if n == 0:
        print("done")
        return

    print(n)
    countdown(n - 1)
```

Iterative countdown:

```python
def countdown(n):
    while n > 0:
        print(n)
        n -= 1

    print("done")
```

Recursive sum:

```python
def sum_to(n):
    if n == 0:
        return 0

    return n + sum_to(n - 1)
```

Iterative sum:

```python
def sum_to(n):
    total = 0

    while n > 0:
        total += n
        n -= 1

    return total
```

For simple linear repetition, iteration is usually better in Python.

For nested or tree-shaped data, recursion can be clearer.

---

# Explicit Stack Instead Of Recursion

Recursion uses the call stack.

You can sometimes replace recursion with your own stack data structure.

Recursive nested-list flattening:

```python
def flatten(values):
    result = []

    for value in values:
        if isinstance(value, list):
            result.extend(flatten(value))
        else:
            result.append(value)

    return result
```

Iterative version with an explicit stack:

```python
def flatten(values):
    result = []
    stack = list(reversed(values))

    while stack:
        value = stack.pop()

        if isinstance(value, list):
            stack.extend(reversed(value))
        else:
            result.append(value)

    return result
```

This avoids recursive calls.

The list `stack` stores work that still needs to be processed.

This pattern matters when data may be deeply nested.

---

# Recursion Depth

Python limits recursion depth.

Example:

```python
def forever():
    forever()

forever()
```

Eventually Python raises:

```text
RecursionError
```

This protects the interpreter from exhausting the C call stack.

You can inspect the current recursion limit:

```python
import sys

print(sys.getrecursionlimit())
```

You can change it:

```python
sys.setrecursionlimit(2000)
```

But raising the limit is not a general solution.

If recursion gets very deep, consider:

* Rewriting with iteration.
* Using an explicit stack.
* Redesigning the algorithm.
* Processing data in smaller chunks.

---

# Tail Recursion

A tail-recursive function returns the recursive call directly.

Example:

```python
def countdown(n):
    if n == 0:
        return

    print(n)
    return countdown(n - 1)
```

The recursive call is the final action.

Some languages optimize tail recursion by reusing the current frame.

Python generally does not do this.

That means tail-recursive Python still consumes stack frames.

Do not rely on tail recursion for large loops in Python.

Use normal iteration instead.

---

# Recursion And Mutable State

Recursive functions can use mutable objects, but you must be careful.

Example:

```python
def collect_numbers(values, result):
    for value in values:
        if isinstance(value, list):
            collect_numbers(value, result)
        else:
            result.append(value)

    return result

data = [1, [2, [3, 4]], 5]
print(collect_numbers(data, []))
```

Output:

```text
[1, 2, 3, 4, 5]
```

The same `result` list is shared across recursive calls.

That can be useful.

But avoid mutable default arguments:

```python
def collect_numbers(values, result=[]):
    ...
```

The default list is created once when the function is defined.

That mistake was introduced in the functions chapter.

It becomes more dangerous with recursion because many calls may share the same object.

---

# Recursion And Closures

Recursion can also appear inside nested functions.

Example:

```python
def make_counter():
    calls = 0

    def count_nodes(node):
        nonlocal calls
        calls += 1

        total = 1
        for child in node["children"]:
            total += count_nodes(child)

        return total

    return count_nodes
```

Here, `count_nodes` is recursive and also closes over `calls`.

This combines:

* Recursion.
* Enclosing scope.
* `nonlocal`.
* Call frames.

The mental model remains the same:

Each recursive call gets a new frame, but all frames can refer to the same closed-over object or binding when the closure allows it.

---

# Designing Recursive Functions

When designing a recursive function, ask these questions:

1. What is the smallest problem I can solve directly?
2. What is the base case?
3. How does the recursive case make the problem smaller?
4. What value should each call return?
5. What should happen before the recursive call?
6. What should happen after the recursive call?
7. Could the recursion become too deep?
8. Would a loop or explicit stack be clearer?

Template:

```python
def recursive_function(problem):
    if base_case(problem):
        return base_result

    smaller_problem = reduce(problem)
    smaller_result = recursive_function(smaller_problem)
    return combine(problem, smaller_result)
```

Not every recursive function looks exactly like this.

But this template exposes the essential structure.

---

# Debugging Recursive Functions

Recursive bugs are easier to debug when you make call depth visible.

Example:

```python
def factorial(n, depth=0):
    indent = "  " * depth
    print(f"{indent}factorial({n})")

    if n == 0:
        print(f"{indent}return 1")
        return 1

    result = n * factorial(n - 1, depth + 1)
    print(f"{indent}return {result}")
    return result

factorial(4)
```

Output:

```text
factorial(4)
  factorial(3)
    factorial(2)
      factorial(1)
        factorial(0)
        return 1
      return 1
    return 2
  return 6
return 24
```

Indentation makes the call stack visible.

This is often more useful than reading the code repeatedly.

---

# Common Mistakes

## Misconception 1

### Recursion means one function frame calls itself in place.

Each recursive call creates a new frame.

The same function object is used, but each call has separate local state.

## Misconception 2

### The base case is optional if the recursive logic is correct.

Without a reachable base case, recursion does not stop.

Python eventually raises `RecursionError`.

## Misconception 3

### Returning from the base case returns from every call immediately.

The base case returns to its caller.

Then each waiting caller continues and returns in turn.

## Misconception 4

### Recursive code is always better than loops.

In Python, loops are often better for simple repetition.

Recursion is most useful when the data or problem is naturally recursive.

## Misconception 5

### Tail recursion avoids stack growth in Python.

Python generally does not optimize tail recursion.

Tail-recursive Python still consumes stack frames.

## Misconception 6

### A recursive function should always mutate shared state.

Some recursive functions use shared mutable accumulators.

Others return combined values.

Choose the design that makes ownership and data flow clearest.

## Misconception 7

### Increasing the recursion limit fixes recursive algorithms.

It may postpone failure.

It does not fix a missing base case, inefficient design, or overly deep input.

---

# Real-world Usage

## Nested Data

Recursive functions are useful for JSON-like data:

```python
def find_strings(value):
    if isinstance(value, str):
        return [value]

    if isinstance(value, list):
        result = []
        for item in value:
            result.extend(find_strings(item))
        return result

    if isinstance(value, dict):
        result = []
        for item in value.values():
            result.extend(find_strings(item))
        return result

    return []
```

## Tree Traversal

```python
def count_nodes(node):
    total = 1

    for child in node.children:
        total += count_nodes(child)

    return total
```

## Directory Traversal

Directory walking is naturally recursive because directories can contain directories.

## Parsing

Parsers often use recursive structure because expressions can contain smaller expressions.

## Backtracking

Search problems sometimes use recursion to try a choice, recurse, and then undo the choice.

Examples include:

* Solving puzzles.
* Generating combinations.
* Exploring paths.
* Traversing decision trees.

---

# Internal Mechanics

Recursive calls use normal function-call mechanics.

There is no special recursion engine.

For each call, Python:

```text
1. Evaluates the call expression.
2. Creates a new frame.
3. Binds arguments to parameters.
4. Pushes the frame onto the call stack.
5. Runs the function body.
6. Returns a value or raises an exception.
7. Pops or unwinds the frame.
```

Recursive calls differ from ordinary nested calls only in one way:

```text
the callee is the same function object
```

Everything else is the same call-stack behavior from Chapter 23.

---

# Concept Connections

Recursion connects to earlier chapters:

* Functions: recursion is made of function calls.
* Scope: each recursive call has its own local scope.
* Closures: recursive nested functions can close over names.
* Call stack: recursive calls create stacked frames.
* Conditionals: base cases are usually conditional branches.
* Loops: recursion and iteration can express repeated work.
* Mutability: shared mutable accumulators require care.
* Identity and references: recursive structures may contain references to nested objects.

Recursion prepares you for:

* Functional programming.
* Tree-shaped data structures.
* Graph traversal.
* Parsing.
* Backtracking.
* Explicit stack data structures.
* Interpreter implementation.

---

# Active Recall

## Easy Recall Questions

1. What is recursion?
2. What is a base case?
3. What is a recursive case?
4. What happens to the call stack during recursion?
5. Why does each recursive call have its own local names?
6. What is `RecursionError`?
7. Does Python optimize tail recursion?
8. When is recursion clearer than iteration?

## Deep Understanding Questions

1. Why does a recursive function need progress toward a base case?
2. How do return values move back through recursive calls?
3. Why can recursion be risky for very deep input in Python?
4. How can an explicit stack replace recursion?
5. Why are trees naturally recursive?
6. Why can mutable accumulators be both useful and dangerous in recursive code?

## Predict-the-Output Questions

### Question 1

```python
def show(n):
    if n == 0:
        print("base")
        return

    print("before", n)
    show(n - 1)
    print("after", n)

show(2)
```

### Question 2

```python
def f(n):
    if n == 0:
        return 0

    return n + f(n - 1)

print(f(3))
```

### Question 3

```python
def mystery(n):
    if n <= 1:
        return 1

    return n * mystery(n - 1)

print(mystery(4))
```

### Question 4

```python
def walk(values):
    for value in values:
        if isinstance(value, list):
            walk(value)
        else:
            print(value)

walk([1, [2, [3]], 4])
```

### Question 5

```python
def bad(n):
    print(n)
    bad(n + 1)

bad(1)
```

What eventually happens?

---

# Practical Exercises

## Exercise 1

Write a recursive `countdown(n)` function.

It should print numbers from `n` down to `1`, then print `"done"`.

## Exercise 2

Write a recursive `factorial(n)` function.

Handle `0` correctly.

Then trace `factorial(4)` by hand.

## Exercise 3

Write a recursive `sum_to(n)` function that returns:

```text
1 + 2 + ... + n
```

Then write an iterative version.

Compare the two.

## Exercise 4

Write a recursive function that counts how many items are in a nested list.

Example:

```python
count_items([1, [2, [3, 4]], 5])
```

should return:

```text
5
```

## Exercise 5

Write a recursive `flatten(values)` function.

It should convert:

```python
[1, [2, [3, 4]], 5]
```

into:

```python
[1, 2, 3, 4, 5]
```

## Exercise 6

Rewrite your recursive `flatten` function using an explicit stack.

Explain which version is easier to read and which version is safer for deeply nested data.

## Exercise 7

Create a tree represented by dictionaries:

```python
node = {
    "name": "root",
    "children": [],
}
```

Write a recursive function that prints all node names with indentation.

## Exercise 8

Write a recursive search function for a nested list.

It should return `True` if the target exists anywhere inside the nested structure.

## Exercise 9

Add debug indentation to a recursive function so each call depth is visible.

## Exercise 10

Find a recursive function that should not be recursive in Python.

Rewrite it using a loop and explain why the loop is better.

---

# Summary

In this chapter we learned:

* Recursion happens when a function calls itself.
* Recursive calls create normal function frames.
* Each recursive call has its own local names.
* A base case stops recursion.
* A recursive case moves toward the base case.
* Recursive calls often descend first and return upward later.
* Code before a recursive call runs while descending.
* Code after a recursive call runs while ascending.
* Return values move back through caller frames.
* Recursion is useful for nested and tree-shaped data.
* Iteration is usually better for simple linear repetition in Python.
* Explicit stacks can replace recursion for deeply nested data.
* Python limits recursion depth.
* Python generally does not optimize tail recursion.
* Mutable state in recursion must be handled carefully.

Core model:

```text
recursive call
    |
    v
new frame for smaller problem
    |
    v
base case
    |
    v
return values climb back through frames
```

Recursion is call-stack reasoning applied to self-similar problems.

---

# Preview of Chapter 25

Next we study functional programming.

Recursion showed that functions can define behavior in terms of smaller calls.

Functional programming broadens that idea by treating functions as values that can be passed, returned, composed, and used to express transformations.

We will study:

* First-class functions.
* Higher-order functions.
* Pure functions.
* Side effects.
* `map`, `filter`, and `reduce` as concepts.
* Lambdas.
* Function composition.
* When functional style improves clarity.
* When normal loops are better.

The connection is direct:

```text
functions create reusable behavior
recursion reuses a function within itself
functional programming treats functions as data for building behavior
```

Functional programming is not a replacement for ordinary Python style.

It is another tool for expressing clear transformations when the problem fits.
