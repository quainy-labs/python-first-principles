# Chapter 23 — Call Stack

---

# Learning Objectives

By the end of this chapter, you should understand:

* What the call stack is.
* Why function calls need frames.
* How frames are pushed and popped.
* How return values move back to callers.
* How nested function calls execute.
* How tracebacks represent call history.
* What recursion is.
* Why recursion needs a base case.
* What maximum recursion depth means.
* How the call stack relates to local names.
* How the call stack differs from heap/object memory at a high level.
* How to debug errors using stack traces.

The call stack is the execution structure that answers:

> Where am I in the chain of active function calls?

---

# Concept Overview

When a function calls another function, Python must remember where to return.

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

While `third()` is running, Python must remember:

```text
third was called by second
second was called by first
first was called by module-level code
```

This active chain of calls is represented by the call stack.

Conceptually:

```text
top -> third frame
       second frame
       first frame
bottom -> module frame
```

The top frame is the currently executing function.

---

# Why the Call Stack Exists

Function calls need memory of execution state.

When Python calls a function, it must remember:

* Which function is running.
* What arguments were passed.
* What local names exist.
* Where execution currently is.
* Where to return after the function finishes.

This information lives in a frame.

The call stack organizes active frames.

Without a call stack, Python could not correctly resume callers after nested function calls.

---

# Frames

A frame is the runtime context for one active function call.

A frame contains:

* The function's code object.
* Local names.
* Parameter bindings.
* Current instruction position.
* Evaluation stack.
* Reference to global namespace.
* Exception handling information.

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

The frame is active only while the function call is active.

---

# Pushing a Frame

When a function is called, Python pushes a new frame onto the call stack.

Example:

```python
def greet(name):
    return f"Hello, {name}"

message = greet("Ada")
```

Before the call:

```text
module frame
```

During the call:

```text
greet frame
module frame
```

The `greet` frame is on top.

Python executes the function body inside that frame.

---

# Popping a Frame

When a function returns, Python pops its frame from the call stack.

Example:

```python
def greet(name):
    return f"Hello, {name}"

message = greet("Ada")
```

During the call:

```text
greet frame
module frame
```

After return:

```text
module frame
```

The return value is passed back to the caller.

Then the caller continues.

---

# Return Flow

Example:

```python
def add(a, b):
    return a + b

def calculate():
    result = add(2, 3)
    return result * 10

print(calculate())
```

Execution:

```text
module calls calculate
calculate calls add
add returns 5
calculate receives 5
calculate returns 50
module passes 50 to print
print displays 50
```

Call stack while `add` is running:

```text
add frame
calculate frame
module frame
```

The return value moves from the top frame back to the frame below it.

---

# Step-by-Step Call Stack

Code:

```python
def a():
    b()
    print("after b")

def b():
    c()
    print("after c")

def c():
    print("inside c")

a()
```

Execution:

```text
module calls a
a calls b
b calls c
c prints "inside c"
c returns None
b resumes and prints "after c"
b returns None
a resumes and prints "after b"
a returns None
module continues
```

Output:

```text
inside c
after c
after b
```

The stack grows with calls and shrinks with returns.

---

# Local Names Live in Frames

Each frame has its own local names.

Example:

```python
def outer():
    x = "outer"
    inner()

def inner():
    x = "inner"
    print(x)

outer()
```

Output:

```text
inner
```

While `inner` is running:

```text
inner frame
└── x ─────▶ "inner"

outer frame
└── x ─────▶ "outer"
```

The same name spelling can exist in different frames.

They are different local bindings.

---

# Call Stack vs Object Memory

The call stack tracks active calls.

Objects live in memory managed by Python.

Example:

```python
def make_list():
    items = [1, 2, 3]
    return items

result = make_list()
```

During the call:

```text
make_list frame
└── items ─────▶ list object [1, 2, 3]
```

After return:

```text
module frame
└── result ─────▶ list object [1, 2, 3]
```

The function frame is gone.

The list object remains because `result` refers to it.

Frame lifetime and object lifetime are different.

---

# Tracebacks

A traceback is Python's report of the call stack when an exception occurs.

Example:

```python
def divide(a, b):
    return a / b

def calculate():
    return divide(10, 0)

calculate()
```

Python raises:

```text
ZeroDivisionError
```

The traceback shows the call chain:

```text
module called calculate
calculate called divide
divide failed on a / b
```

A traceback is not noise.

It is a map of active calls leading to the error.

---

# Reading a Traceback

A traceback usually shows:

* File name
* Line number
* Function name
* Source line
* Exception type
* Exception message

Example structure:

```text
Traceback (most recent call last):
  File "app.py", line 7, in <module>
    calculate()
  File "app.py", line 5, in calculate
    return divide(10, 0)
  File "app.py", line 2, in divide
    return a / b
ZeroDivisionError: division by zero
```

Read from bottom for the immediate error:

```text
ZeroDivisionError in divide
```

Read upward to see how execution got there.

---

# Exceptions Unwind the Stack

When an exception is not handled in the current frame, Python exits that frame and looks in the caller.

This is called stack unwinding.

Example:

```python
def inner():
    1 / 0

def outer():
    inner()

outer()
```

The exception starts in `inner`.

If `inner` does not handle it, Python returns control abnormally to `outer`.

If `outer` does not handle it, Python continues unwinding.

If no frame handles it, the program terminates with a traceback.

Exception handling will be studied later.

---

# Recursion

Recursion happens when a function calls itself.

Example:

```python
def countdown(n):
    if n <= 0:
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

Each recursive call creates a new frame.

Even though the function is the same, each call has its own `n`.

---

# Recursive Call Stack

For:

```python
countdown(3)
```

the stack grows:

```text
countdown n=3
module
```

then:

```text
countdown n=2
countdown n=3
module
```

then:

```text
countdown n=1
countdown n=2
countdown n=3
module
```

then:

```text
countdown n=0
countdown n=1
countdown n=2
countdown n=3
module
```

When the base case returns, frames pop one by one.

---

# Base Case

A recursive function needs a base case.

The base case stops recursion.

Example:

```python
def countdown(n):
    if n <= 0:
        return

    countdown(n - 1)
```

Base case:

```python
if n <= 0:
    return
```

Without a base case, recursion continues until Python stops it.

---

# RecursionError

Python limits recursion depth.

Example:

```python
def forever():
    forever()

forever()
```

Eventually raises:

```text
RecursionError
```

Why?

Each call adds a frame.

Unbounded recursion would consume memory.

Python protects the program by limiting recursion depth.

This limit can be changed, but doing so is rarely the right first solution.

---

# Factorial Example

Factorial:

```text
5! = 5 * 4 * 3 * 2 * 1
```

Recursive implementation:

```python
def factorial(n):
    if n == 0:
        return 1

    return n * factorial(n - 1)

print(factorial(5))
```

Output:

```text
120
```

Call chain:

```text
factorial(5)
factorial(4)
factorial(3)
factorial(2)
factorial(1)
factorial(0)
```

Then return values multiply on the way back.

---

# Recursion vs Iteration

Many recursive solutions can be written with loops.

Recursive factorial:

```python
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)
```

Iterative factorial:

```python
def factorial(n):
    result = 1

    for number in range(1, n + 1):
        result *= number

    return result
```

Iteration is often more practical in Python for simple repetition.

Recursion is useful for naturally recursive structures such as trees.

Python does not optimize tail recursion like some languages do.

---

# Tail Recursion Note

Tail recursion is recursion where the recursive call is the final action.

Some languages optimize tail recursion to avoid stack growth.

Python generally does not.

So recursive Python code still consumes stack frames for recursive calls.

Do not rely on tail-call optimization in Python.

Use iteration when recursion depth may be large.

---

# Call Stack and Debugging

When debugging, ask:

```text
Which function am I in?
Who called it?
What arguments were passed?
What local names exist?
What line failed?
```

The traceback answers many of these.

Debuggers also show call stack frames.

A good debugging habit is to identify:

```text
failure location
caller chain
bad value source
```

The call stack gives the caller chain.

---

# Call Stack and Return Values

Return values move one frame down the stack.

Example:

```python
def one():
    return 1

def two():
    return one() + 1

def three():
    return two() + 1

print(three())
```

Output:

```text
3
```

Flow:

```text
three calls two
two calls one
one returns 1 to two
two returns 2 to three
three returns 3 to module
print displays 3
```

---

# Call Stack and Side Effects

Not every function returns useful data.

Example:

```python
def log(message):
    print(message)

def run():
    log("starting")
    return "done"

print(run())
```

Output:

```text
starting
done
```

`log` returns `None`.

Its purpose is side effect.

`run` returns `"done"`.

The call stack handles both return flow and side-effecting calls.

---

# Common Mistakes

## Misconception 1

### A function has only one frame forever.

Each function call creates a new frame.

The same function can have many frames active during recursion.

---

## Misconception 2

### Local variables are shared across calls.

Each call has its own local frame.

Local names from one call are separate from local names in another call.

---

## Misconception 3

### Returning destroys returned objects.

Returning removes the frame, but returned objects can live on if the caller references them.

---

## Misconception 4

### Tracebacks should be read only from the top.

The bottom shows the immediate error.

The lines above show the call path.

Both matter.

---

## Misconception 5

### Recursion reuses the same local variables.

Each recursive call has its own frame and its own local names.

---

## Misconception 6

### Recursion can continue forever if logic is correct enough.

Recursive calls consume stack frames.

Python enforces a recursion limit.

---

## Misconception 7

### Python optimizes tail recursion.

Python generally does not perform tail-call optimization.

Use loops when deep recursion would be a problem.

---

# Real-world Usage

## Debugging Tracebacks

Tracebacks are call stack reports.

Use them to answer:

```text
Where did the error happen?
Which calls led there?
Which function received the bad value?
```

Do not ignore the middle frames.

They often reveal where incorrect data entered the path.

---

## Recursive Data

Recursion is useful for recursive structures:

* Trees
* Nested directories
* Nested JSON
* Graph traversal with care
* Expression trees

Example concept:

```python
def walk(node):
    process(node)
    for child in node.children:
        walk(child)
```

The call stack naturally tracks the path through the tree.

---

## Avoiding Deep Recursion

For large inputs, prefer iteration or an explicit stack.

Example:

```python
stack = [root]

while stack:
    node = stack.pop()
    process(node)
    stack.extend(node.children)
```

This uses a list as an explicit stack instead of the Python call stack.

Data structures will be studied next.

---

## Profiling and Performance

Function calls have overhead.

Usually this overhead is acceptable.

But in performance-sensitive loops, excessive tiny function calls can matter.

Optimize only with evidence.

Clear functions are usually worth the small overhead.

---

# Concept Connections

This chapter completes Volume I's function execution model:

```text
Chapter 20:
Functions create frames when called.

Chapter 21:
Frames contain local scopes.

Chapter 22:
Closures can retain data beyond active frames.

Chapter 23:
The call stack organizes active frames.
```

It prepares the data structures part:

```text
Lists:
    can be used as explicit stacks

Data Structures:
    containers hold objects and references

Memory:
    object lifetime differs from frame lifetime

Garbage Collection:
    unreachable objects can be reclaimed
```

Core model:

```text
function call
    |
    v
push frame
    |
    v
execute body
    |
    v
return or raise
    |
    v
pop frame
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Call stack | Stack of active function frames |
| Frame | Runtime context for one active call |
| Push | Add a frame for a new call |
| Pop | Remove a frame when call ends |
| Return value | Object passed back to caller |
| Traceback | Report of call stack at exception time |
| Stack unwinding | Exiting frames due to an exception |
| Recursion | Function calling itself |
| Base case | Condition that stops recursion |
| RecursionError | Error when recursion limit is exceeded |

Core rules:

```text
Each function call creates a frame.
Frames form a stack.
The top frame is currently running.
Return pops the current frame.
Return values go to the caller frame.
Exceptions unwind frames until handled.
Recursive calls create multiple frames for the same function.
Objects can outlive frames if still referenced.
```

---

# Active Recall

## Easy Recall Questions

1. What is the call stack?
2. What is a frame?
3. What happens when a function is called?
4. What happens when a function returns?
5. Which frame is currently executing?
6. What is a traceback?
7. What is recursion?
8. What is a base case?
9. What error occurs when recursion is too deep?
10. Does Python generally optimize tail recursion?

---

## Deep Understanding Questions

1. Why does each function call need a frame?
2. Why can the same function have multiple active frames?
3. Why do local names from one call not affect another call?
4. Why can returned objects outlive the function frame that created them?
5. Why are tracebacks useful for debugging?
6. Why does an exception unwind the stack?
7. Why does recursion need a base case?
8. Why can iteration be safer than recursion for large inputs?
9. Why is the call stack different from object memory?
10. Why does the call stack complete the function model?

---

## Explain In Your Own Words

1. Explain pushing and popping frames.
2. Explain return flow through callers.
3. Explain how to read a traceback.
4. Explain recursion using `countdown`.
5. Explain why recursive calls have separate local names.
6. Explain frame lifetime vs object lifetime.
7. Explain stack unwinding.

---

## Predict-the-Output Questions

### Question 1

```python
def a():
    b()
    print("a")

def b():
    print("b")

a()
```

Answer:

```text
b
a
```

Reason:

`a` calls `b`; after `b` returns, `a` continues.

---

### Question 2

```python
def one():
    return 1

def two():
    return one() + 1

print(two())
```

Answer:

```text
2
```

Reason:

`one` returns `1` to `two`, then `two` returns `2`.

---

### Question 3

```python
def countdown(n):
    if n == 0:
        print("done")
        return
    print(n)
    countdown(n - 1)

countdown(2)
```

Answer:

```text
2
1
done
```

Reason:

Each call decrements `n` until the base case.

---

### Question 4

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

The frame is gone, but the list object remains because `result` refers to it.

---

### Question 5

```python
def forever():
    forever()

forever()
```

Answer:

Python eventually raises `RecursionError`.

Reason:

Each recursive call adds a frame until the recursion limit is exceeded.

---

# Mental Model Questions

1. Draw the stack while `a()` calls `b()` and `b()` calls `c()`.
2. Draw the stack before and after a function return.
3. Draw a return value moving from callee to caller.
4. Draw local names in two separate calls to the same function.
5. Draw recursive calls for `countdown(3)`.
6. Draw a traceback as a call chain.
7. Draw a returned object outliving its function frame.

---

# Practical Exercises

## Exercise 1

Trace the call stack:

```python
def first():
    second()
    print("first done")

def second():
    third()
    print("second done")

def third():
    print("third done")

first()
```

Write the stack at each function call.

---

## Exercise 2

Read a traceback:

```python
def divide(a, b):
    return a / b

def calculate():
    return divide(10, 0)

calculate()
```

Run it and identify each frame in the traceback.

---

## Exercise 3

Write recursive countdown:

```python
def countdown(n):
    ...
```

It should print from `n` down to `1`, then `"done"`.

Draw the recursive call stack for `countdown(3)`.

---

## Exercise 4

Write iterative factorial and recursive factorial.

Compare how each uses state.

Explain which one is safer for large `n` in Python.

---

## Exercise 5

Demonstrate object lifetime:

```python
def make_data():
    data = {"status": "ok"}
    return data

result = make_data()
print(result)
```

Explain why the dictionary still exists after `make_data` returns.

---

## Exercise 6

Create two calls with separate frames:

```python
def show(value):
    local = value * 2
    return local

print(show(2))
print(show(5))
```

Draw each call frame.

---

## Exercise 7

Rewrite recursion with an explicit stack.

Given a nested list:

```python
data = [1, [2, [3, 4]], 5]
```

Sketch how you might process it with:

* recursive calls
* a list used as an explicit stack

You do not need a perfect implementation yet. Focus on the mental model.

---

# Summary

In this chapter we learned:

* The call stack is the stack of active function frames.
* Each function call creates a frame.
* A frame contains local names and execution state.
* Calling a function pushes a frame.
* Returning from a function pops a frame.
* Return values move back to caller frames.
* Nested calls create stacked frames.
* Local names belong to frames.
* Objects can outlive frames if still referenced.
* Tracebacks report call history when exceptions occur.
* Exceptions unwind the stack until handled.
* Recursion is a function calling itself.
* Recursive calls create separate frames.
* Recursion needs a base case.
* Python raises `RecursionError` for excessive recursion.
* Python generally does not optimize tail recursion.
* Iteration or explicit stacks are often better for deep repetition.

Core model:

```text
call function
    |
    v
push frame
    |
    v
execute
    |
    v
return value or exception
    |
    v
pop or unwind frame
```

The call stack explains how active function calls are organized.

---

# Preview of Chapter 24

Next we study recursion.

The call stack makes recursion understandable because each recursive call creates a new frame.

We will study:

* What recursion is.
* Base cases.
* Recursive cases.
* How recursive calls create separate frames.
* How return values move back up the stack.
* How to trace recursive execution.
* Recursion with lists and nested data.
* Recursion with trees.
* Recursion depth and `RecursionError`.
* Tail recursion in Python.
* When iteration or an explicit stack is better.

The transition is direct:

```text
function call -> frame
recursive call -> another frame for the same function
base case -> stop creating frames
return values -> move back through waiting frames
```

Recursion is the next step after understanding the call stack.
