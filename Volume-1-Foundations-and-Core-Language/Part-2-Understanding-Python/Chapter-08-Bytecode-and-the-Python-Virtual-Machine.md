# Chapter 08 — Bytecode and the Python Virtual Machine

---

# Learning Objectives

By the end of this chapter, you should understand:

* What Python bytecode is.
* Why CPython compiles source code into bytecode.
* Why bytecode is not the same as CPU machine code.
* What the Python Virtual Machine does.
* How bytecode instructions produce runtime behavior.
* Why CPython uses a stack-based execution model.
* What a code object contains.
* What a frame is at a high level.
* How function calls create new execution frames.
* Why `dis` is useful for understanding Python internals.
* Why exact bytecode instructions may differ across Python versions.
* How bytecode connects source code to objects, names, functions, and control flow.

This chapter answers the question left open by Chapter 07:

> Once CPython has bytecode, how does that bytecode actually run?

---

# Concept Overview

In Chapter 07, we followed Python source code through this pipeline:

```text
.py file
  -> source text
  -> tokens
  -> AST
  -> code object
  -> bytecode
  -> Python Virtual Machine
  -> behavior
```

This chapter focuses on the last major part:

```text
code object
  -> bytecode
  -> Python Virtual Machine
  -> runtime behavior
```

Python bytecode is a lower-level instruction format used by CPython.

It is not written for humans.

It is not written for the CPU directly.

It is written for the Python Virtual Machine.

For example, source code like this:

```python
x = 10
y = 20
print(x + y)
```

is compiled into instructions that conceptually say:

```text
load constant 10
store it as x
load constant 20
store it as y
load print
load x
load y
add x and y
call print
discard the return value
```

The exact bytecode names depend on the Python version.

But the idea is stable:

> CPython translates readable Python source into smaller interpreter instructions, then executes those instructions.

---

# Mental Model

Think of source code as a recipe written for a human:

```text
Make tea:
1. Boil water.
2. Put tea leaves in a cup.
3. Pour water.
4. Wait.
```

Bytecode is closer to detailed kitchen actions:

```text
LOAD kettle
LOAD water
CALL boil
LOAD cup
LOAD tea_leaves
CALL place
...
```

The Python Virtual Machine is the worker following those lower-level instructions.

Another mental model:

```text
Python source code
    Human-friendly description

Bytecode
    CPython-friendly instruction sequence

Python Virtual Machine
    The executor of those instructions
```

The CPU is still involved, but indirectly.

The CPU executes CPython itself.

CPython executes your bytecode.

```text
CPU
  executes
CPython machine code
  which executes
Python bytecode
  which represents
your Python program
```

This distinction removes a lot of confusion.

---

# Why Bytecode Exists

CPython could theoretically interpret source text directly.

But that would be inefficient and complicated.

Source text contains details that are useful to humans:

* Whitespace
* Comments
* Formatting
* Literal spelling
* High-level syntax

The interpreter does not want to repeatedly reason about raw text.

It wants a compact set of instructions.

Bytecode exists because CPython needs a representation that is:

* Easier to execute than source code.
* More explicit than source code.
* Independent of most formatting details.
* Connected to runtime operations.
* Portable across machines running compatible Python versions.
* Easier to cache for imported modules.

Source code is designed for people.

Bytecode is designed for the interpreter.

---

# Bytecode Is an Intermediate Representation

An intermediate representation is a form between the original source language and the final execution machinery.

In CPython:

```text
Python source
     |
     v
bytecode
     |
     v
Python Virtual Machine execution
```

Bytecode is not the final thing the CPU executes.

It is the final thing CPython's interpreter loop executes.

This is different from a C program:

```text
C source
     |
     v
machine code
     |
     v
CPU execution
```

For CPython:

```text
Python source
     |
     v
Python bytecode
     |
     v
CPython interpreter
     |
     v
CPU executes CPython's machine code
```

That is why Python bytecode is not generally a standalone executable program.

It needs a compatible Python runtime.

---

# Bytecode Is Version-Specific

Python bytecode is an implementation detail.

It can change between Python versions.

For example, Python 3.10, Python 3.11, Python 3.12, and later versions may show different bytecode for similar source code.

The exact instruction names and optimizations are not the main thing to memorize.

The important mental model is:

```text
source code
  -> compiled interpreter instructions
  -> executed by the Python Virtual Machine
```

When this book shows bytecode examples, focus on the meaning:

* Loading values
* Storing names
* Calling functions
* Performing operations
* Returning values
* Jumping for control flow

The exact printed output from `dis` may differ on your machine.

That is normal.

---

# Seeing Bytecode With dis

Python includes a standard library module named `dis`.

`dis` means disassembler.

A disassembler shows a lower-level instruction representation.

Example:

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

You may see output containing instructions such as:

```text
LOAD_FAST
BINARY_OP
RETURN_VALUE
```

The output may include additional instructions depending on your Python version.

Conceptually:

* `LOAD_FAST` loads a local variable.
* `BINARY_OP` performs an operation such as addition.
* `RETURN_VALUE` returns a value from the function.

The source code:

```python
return a + b
```

is human-friendly.

The bytecode says more operationally:

```text
load a
load b
add them
return the result
```

That is the key idea.

---

# A First Bytecode Walkthrough

Consider:

```python
def add(a, b):
    return a + b
```

At a high level, CPython needs to do this when `add(2, 3)` runs:

```text
1. Create a function call frame.
2. Bind a to 2.
3. Bind b to 3.
4. Load a.
5. Load b.
6. Add them.
7. Return the result.
8. Destroy or release the frame when the call is done.
```

Bytecode represents the middle part:

```text
load a
load b
add
return
```

Notice what bytecode does not look like.

It does not look like the original source:

```python
return a + b
```

It also does not look like CPU assembly for a specific processor.

It is its own instruction language for the Python runtime.

---

# What Is the Python Virtual Machine?

The Python Virtual Machine is the part of CPython that executes bytecode.

It is not a separate program you usually launch manually.

It is part of the CPython runtime.

At a high level, the PVM does this:

```text
while there are bytecode instructions:
    read the next instruction
    perform the operation
    update runtime state
```

This is called an interpreter loop.

The PVM repeatedly:

* Fetches an instruction.
* Decodes what it means.
* Executes it.
* Moves to the next instruction.

This is similar in spirit to a CPU instruction cycle:

```text
fetch
decode
execute
```

But the CPU executes machine instructions.

The Python Virtual Machine executes Python bytecode instructions.

---

# Why It Is Called a Virtual Machine

A physical machine has:

* Instructions
* Memory
* Execution state
* Control flow

The Python Virtual Machine is "virtual" because it is implemented in software.

It provides an execution environment for Python bytecode.

It has:

* Bytecode instructions
* A value stack
* Frames
* Namespaces
* Exception handling state
* Instruction pointers

It is not virtual in the same way as a full operating-system virtual machine such as VirtualBox or VMware.

The PVM does not emulate an entire computer.

It is a language virtual machine.

It exists to execute Python programs.

---

# Stack-Based Execution

CPython bytecode uses a stack-based execution model.

A stack is a last-in, first-out structure.

You can think of it like a stack of plates:

```text
push plate A
push plate B
pop  -> gets plate B first
pop  -> gets plate A
```

In bytecode execution, the PVM uses an evaluation stack.

Instructions push values onto the stack and pop values from the stack.

For example, to compute:

```python
2 + 3
```

the bytecode model is conceptually:

```text
push 2
push 3
pop 3
pop 2
add them
push 5
```

The result is left on the stack.

---

# Stack Walkthrough: 2 + 3

Source:

```python
result = 2 + 3
```

Conceptual bytecode:

```text
LOAD_CONST 2
LOAD_CONST 3
BINARY_OP +
STORE_NAME result
```

Stack behavior:

```text
start:
[]

LOAD_CONST 2:
[2]

LOAD_CONST 3:
[2, 3]

BINARY_OP +:
[5]

STORE_NAME result:
[]

namespace:
result -> 5
```

The stack is temporary working space.

The name `result` is stored in a namespace.

This distinction will become important in later chapters:

* Stack: temporary execution values.
* Namespace: mapping from names to objects.
* Object: actual runtime value.

---

# Stack Walkthrough: Function Call

Consider:

```python
print(10 + 20)
```

Conceptually:

```text
load print
load 10
load 20
add them
call print with the result
```

Stack behavior:

```text
start:
[]

load print:
[print]

load 10:
[print, 10]

load 20:
[print, 10, 20]

add:
[print, 30]

call:
[]
```

In reality, modern CPython has specific call-related instructions that can vary by version.

The important idea is that function calls are built from bytecode operations that load the callable, load arguments, and perform the call.

---

# Code Objects

Chapter 07 introduced code objects briefly.

Now we can look more closely.

A code object is a compiled representation of a block of Python code.

Blocks of code include:

* A module
* A function body
* A class body
* A comprehension body
* Code passed to `eval()` or `exec()`

For example:

```python
def square(x):
    return x * x
```

The function `square` has a code object.

You can access it using:

```python
square.__code__
```

That code object contains information such as:

* The bytecode instruction sequence.
* Constants used by the code.
* Names referenced by the code.
* Local variable names.
* Argument information.
* Filename information.
* Function name information.
* Line number information for tracebacks and debugging.

The code object is not executing by itself.

It is data that the Python runtime can execute inside a frame.

---

# Function Object vs Code Object

This distinction is very important.

Consider:

```python
def greet(name):
    return "Hello, " + name
```

When Python executes the `def` statement, it creates a function object.

The function object contains or references:

* A code object
* A global namespace
* Default argument values, if any
* Closure variables, if any
* Metadata such as the function name

The code object contains the compiled instructions.

The function object is the callable runtime object.

Mental model:

```text
function object
    |
    v
code object
    |
    v
bytecode instructions
```

When you call the function, Python uses the function object to create a new execution frame from the code object.

---

# Inspecting a Code Object

Example:

```python
def square(x):
    return x * x

code = square.__code__

print(code.co_name)
print(code.co_varnames)
print(code.co_consts)
```

You may see output like:

```text
square
('x',)
(None,)
```

The exact details may vary.

Important ideas:

* `co_name` stores the code object's name.
* `co_varnames` stores local variable names.
* `co_consts` stores constants used by the code.

For example:

```python
def answer():
    return 42
```

The constant `42` is stored in the code object's constants.

This means constants are not rediscovered from source text every time the function runs.

They are part of the compiled representation.

---

# Constants and Names in Code Objects

Consider:

```python
def show():
    message = "Hello"
    print(message)
```

The code object must know:

* The string constant `"Hello"`.
* The local name `message`.
* The referenced name `print`.

Conceptually:

```text
constants:
    None
    "Hello"

local variables:
    message

names:
    print
```

The bytecode then refers to these tables.

Instead of storing the full string `"Hello"` inside every instruction, bytecode can say:

```text
load constant at index 1
```

Instead of storing all name data directly inside every instruction, bytecode can refer to name tables.

This makes execution more structured.

---

# Frames

To execute a code object, CPython creates a frame.

A frame represents one active execution context.

When a function is running, it has a frame.

When a module is executing top-level code, it has a frame.

A frame contains runtime state such as:

* The code object being executed.
* The current instruction position.
* The evaluation stack.
* Local variables.
* Global variables reference.
* Builtins reference.
* Exception handling state.

If a code object is the recipe, a frame is the active cooking session.

The same recipe can be used many times.

Each active run needs its own state.

---

# Function Calls Create Frames

Consider:

```python
def double(x):
    return x * 2

result = double(5)
```

At a high level:

```text
module frame starts
    |
    v
function object double is created
    |
    v
double(5) is called
    |
    v
new frame for double is created
    |
    v
x is bound to 5 inside that frame
    |
    v
bytecode for double executes
    |
    v
return value goes back to module frame
```

The module and the function do not share one flat execution space.

The function call gets its own frame.

This is why local variables inside a function do not automatically become global variables.

---

# Frame Stack

When functions call other functions, frames stack up.

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

While `c()` is running, the call stack is conceptually:

```text
c frame
b frame
a frame
module frame
```

The top frame is currently executing.

When `c()` returns, its frame is removed.

Then execution resumes in `b()`.

This connects directly to Chapter 03, where we introduced stack memory and function calls.

Python frames are the language-level execution records that make function calls work.

---

# Namespaces During Execution

Bytecode often works with names.

For example:

```python
x = 10
print(x)
```

The bytecode must:

* Store `10` under the name `x`.
* Later load the value associated with `x`.
* Load the callable named `print`.
* Call it.

Names live in namespaces.

A namespace is a mapping from names to objects.

At a high level:

```text
namespace:
    x -> 10
    print -> built-in print function
```

Later chapters will deeply explain names and references.

For now, the important connection is:

> Bytecode instructions do not merely compute numbers. They also load and store names in namespaces.

---

# LOAD and STORE Operations

Many bytecode instructions are about moving values.

Common conceptual categories:

```text
LOAD    -> put a value onto the evaluation stack
STORE   -> take a value from the stack and bind/store it somewhere
CALL    -> call a callable object
RETURN  -> return from the current frame
JUMP    -> move execution to another instruction
```

Example:

```python
x = 10
```

Conceptual operations:

```text
LOAD_CONST 10
STORE_NAME x
```

Example:

```python
print(x)
```

Conceptual operations:

```text
LOAD_NAME print
LOAD_NAME x
CALL
```

The exact names differ between module scope, function scope, and Python versions.

But the categories remain useful.

---

# Local Variables Are Optimized

Inside functions, local variables are handled efficiently.

Example:

```python
def add(a, b):
    total = a + b
    return total
```

The names `a`, `b`, and `total` are local variables.

CPython can access many local variables through fast internal storage rather than a normal dictionary lookup.

That is why you may see bytecode names like:

```text
LOAD_FAST
STORE_FAST
```

Conceptually:

* `LOAD_FAST` loads a local variable.
* `STORE_FAST` stores a local variable.

The word "fast" hints that local variables are optimized.

This also explains why Python needs to know which names are local in a function.

Compilation analyzes the function body and determines local variable layout.

---

# Example: Assignment

Source:

```python
x = 10
```

Conceptual bytecode:

```text
LOAD_CONST 10
STORE_NAME x
```

Execution:

```text
LOAD_CONST 10
    stack becomes [10]

STORE_NAME x
    stack becomes []
    namespace gets x -> 10
```

Visible result:

```python
print(x)
```

prints:

```text
10
```

The bytecode explains that assignment is not putting a value inside a box.

It is binding a name to an object.

Chapter 10 will develop this mental model fully.

---

# Example: Arithmetic

Source:

```python
result = 2 + 3 * 4
```

Python must respect operator precedence.

Multiplication happens before addition.

Conceptual operations:

```text
load 2
load 3
load 4
multiply 3 and 4
add 2 and 12
store result
```

Stack walkthrough:

```text
[]
[2]
[2, 3]
[2, 3, 4]
[2, 12]
[14]
[]

namespace:
result -> 14
```

The AST already represented the correct structure.

The compiler turns that structure into bytecode that produces the correct result.

---

# Example: Conditional Execution

Source:

```python
if temperature > 30:
    status = "hot"
else:
    status = "comfortable"
```

Bytecode must support branching.

Conceptually:

```text
load temperature
load 30
compare >
if false, jump to else branch
load "hot"
store status
jump past else branch
load "comfortable"
store status
```

This is how high-level control flow becomes instruction-level control flow.

The source code shows a clean `if` statement.

The bytecode contains conditional jumps.

---

# Example: Loop Execution

Source:

```python
count = 0

while count < 3:
    print(count)
    count = count + 1
```

Conceptually:

```text
store count as 0

loop_start:
    load count
    load 3
    compare <
    if false, jump to loop_end
    load print
    load count
    call print
    load count
    load 1
    add
    store count
    jump to loop_start

loop_end:
```

Loops are not magic.

They are repeated jumps and tests.

This connects Python control flow back to the execution concepts from Chapter 03.

---

# Example: Function Definition

Source:

```python
def greet(name):
    return "Hello, " + name
```

This often surprises learners:

Defining a function does not run the function body.

When Python executes the `def` statement, it creates a function object and binds it to the function name.

Conceptually:

```text
load code object for greet
create function object
store function object under name greet
```

The function body bytecode exists, but it runs later when the function is called.

This is why:

```python
def greet():
    print("hello")

print("done")
```

prints:

```text
done
```

The body of `greet` does not run.

The function object is created.

---

# Example: Function Call

Source:

```python
def greet(name):
    print("Hello", name)

greet("Ada")
```

At runtime:

```text
1. The module frame creates function object greet.
2. The name greet is bound to that function object.
3. The call greet("Ada") loads the function object.
4. The argument "Ada" is loaded.
5. CPython creates a new frame for the call.
6. Inside that frame, name is bound to "Ada".
7. The bytecode inside greet executes.
8. The frame returns.
9. Execution resumes in the module frame.
```

Function calls are frame transitions.

This idea will become central when we study scope, call stacks, recursion, closures, and exceptions.

---

# Return Values

A function returns by sending a value from its frame back to the caller.

Example:

```python
def add(a, b):
    return a + b

result = add(2, 3)
```

Conceptually:

```text
module frame:
    call add(2, 3)

add frame:
    load a
    load b
    add
    return 5

module frame:
    receive 5
    store result -> 5
```

The return value crosses from the callee frame back to the caller frame.

If a function does not explicitly return a value, Python returns `None`.

That is why:

```python
def say_hi():
    print("hi")

value = say_hi()
print(value)
```

prints:

```text
hi
None
```

The call to `say_hi()` produces the return value `None`.

---

# Exceptions and Bytecode

Exceptions are also part of runtime execution.

Consider:

```python
print("before")
1 / 0
print("after")
```

The source is valid.

CPython can compile it.

The error happens when bytecode attempts division.

At that point, Python raises an exception.

Execution does not continue normally to the next instruction.

Instead, Python looks for exception-handling logic.

If no handler is found, the program terminates and prints a traceback.

Tracebacks are possible because frames contain information about:

* The current code object.
* The current line number.
* The active call stack.

This is why Python can tell you where an error happened.

---

# Tracebacks and Frames

Consider:

```python
def divide(a, b):
    return a / b

def run():
    return divide(10, 0)

run()
```

The traceback shows a chain of calls:

```text
module code called run
run called divide
divide attempted division by zero
```

This mirrors the frame stack:

```text
divide frame
run frame
module frame
```

Tracebacks are not random error messages.

They are reports of active or recently active frames at the moment an exception occurred.

---

# The Evaluation Loop

At the heart of bytecode execution is an evaluation loop.

Conceptually:

```text
while frame is active:
    instruction = next bytecode instruction
    execute instruction
    update stack, names, frame, or instruction pointer
```

Different instructions do different things:

```text
LOAD_CONST
    push a constant onto the stack

STORE_NAME
    pop a value and bind it to a name

BINARY_OP
    pop operands, perform operation, push result

CALL
    call a callable object

RETURN_VALUE
    return from the current frame

JUMP
    move to another instruction
```

The real CPython evaluation loop is implemented in C and is much more complex.

It handles:

* Function calls
* Exceptions
* Generators
* Async execution
* Tracing
* Debugging hooks
* Adaptive optimizations
* Reference counting
* The Global Interpreter Lock

But the beginner mental model remains:

> The PVM repeatedly reads bytecode instructions and updates runtime state.

---

# Adaptive Interpreter Note

Modern CPython versions include adaptive interpreter optimizations.

This means CPython may specialize some bytecode execution paths at runtime based on observed behavior.

For example, if Python sees that a particular addition operation repeatedly adds integers, it may optimize that operation internally.

You do not need to master these details now.

The important point is:

* The language meaning stays the same.
* The implementation may optimize execution.
* Bytecode details can vary by Python version.

This is another reason not to memorize every instruction name too early.

Learn the model first.

---

# Bytecode and Objects

Bytecode operates on objects.

When bytecode loads the constant `10`, it loads a Python object representing the integer value `10`.

When bytecode loads `"hello"`, it loads a string object.

When bytecode calls `print`, it loads a function-like object and calls it.

This prepares us for Chapter 09:

> Runtime execution is object manipulation.

Source code:

```python
x = 10
```

Conceptually:

```text
load integer object 10
bind name x to that object
```

Python does not treat values as raw unstructured data.

Python values are objects.

Bytecode is the instruction layer that creates, loads, stores, and manipulates those objects.

---

# Bytecode and Names

Bytecode also explains why names matter.

Source:

```python
x = 10
y = x
```

Conceptually:

```text
load 10
store name x
load name x
store name y
```

This does not mean `x` is a box containing `10`.

It means:

* There is an object representing `10`.
* The name `x` refers to that object.
* The name `y` can be bound to the same object.

Chapter 10 will explain names and references in full.

But bytecode already hints at the truth:

> Python execution loads objects and binds names to objects.

---

# Bytecode and Mutability

Consider:

```python
items = []
items.append("a")
```

Conceptually:

```text
create/load empty list object
store name items
load items
load append method
load "a"
call append
```

The second line does not reassign `items`.

It calls a method on the object that `items` refers to.

That method mutates the list.

This will matter later when we study mutability.

Bytecode helps us separate:

* Binding names
* Loading objects
* Calling methods
* Mutating objects

Those are different operations.

---

# Bytecode and Control Flow

High-level Python control flow becomes bytecode jumps.

Examples:

```python
if condition:
    do_this()
else:
    do_that()
```

becomes conceptually:

```text
evaluate condition
if false, jump to else
call do_this
jump to after if
call do_that
after if
```

Loops are also jumps:

```python
while condition:
    body()
```

becomes conceptually:

```text
loop start
evaluate condition
if false, jump after loop
call body
jump to loop start
after loop
```

This connects Python to general computer science:

> Structured control flow is implemented using conditional and unconditional jumps.

Python syntax makes control flow readable.

Bytecode makes it executable by the interpreter.

---

# Bytecode Caching

Chapter 07 introduced `__pycache__`.

Now we can understand it more clearly.

When Python imports a module, CPython may save the compiled bytecode in a `.pyc` file.

Example:

```text
project/
├── main.py
├── tools.py
└── __pycache__/
    └── tools.cpython-312.pyc
```

The `.pyc` file stores compiled bytecode and metadata.

The purpose is to avoid recompiling unchanged imported modules every time.

Important:

* `.pyc` files are an optimization.
* They are version-specific.
* They are not meant to be edited by humans.
* They do not replace the need for the Python runtime.
* They can usually be deleted and regenerated.

Bytecode caching improves startup/import behavior, especially in projects with many modules.

---

# Why Scripts May Not Always Leave __pycache__

Learners sometimes expect every executed Python file to create `__pycache__`.

The behavior depends on how code is run and what is imported.

Most commonly, cached bytecode is created for imported modules.

If you run:

```bash
python main.py
```

Python may not create a `.pyc` for `main.py` in the same way it does for imported modules.

But if `main.py` imports `helper.py`, you may see cached bytecode for `helper.py`.

The practical rule is:

> `__pycache__` is normal. Do not panic when you see it. Do not depend on editing it.

---

# Why Bytecode Matters

You can write Python for a long time without reading bytecode.

So why learn it?

Because bytecode explains the machinery behind many behaviors:

* Why syntax errors happen before runtime.
* Why function bodies do not run at definition time.
* Why local variables are different from globals.
* Why loops and conditionals are jumps.
* Why function calls create frames.
* Why tracebacks show call chains.
* Why imports execute top-level module code.
* Why `__pycache__` exists.
* Why Python is not simply "line by line text execution."

You do not need bytecode for every daily programming task.

But understanding it gives you a deeper mental model of Python.

That is the goal of this book.

---

# Reading dis Output Without Fear

When you first see `dis` output, it may look intimidating.

Example output might include columns such as:

```text
line number
instruction offset
instruction name
instruction argument
argument meaning
```

You do not need to memorize the table.

Ask these questions instead:

1. What values are being loaded?
2. What names are being stored?
3. What operation is being performed?
4. Is a function being called?
5. Is execution jumping somewhere?
6. Where does the function return?

This turns bytecode from noise into a readable execution story.

---

# Example: Reading dis Conceptually

Source:

```python
def example(x):
    y = x + 1
    return y
```

Conceptual bytecode story:

```text
load local x
load constant 1
add them
store local y
load local y
return it
```

Visible behavior:

```python
print(example(4))
```

Output:

```text
5
```

The source tells you what the function means.

The bytecode tells you the execution steps CPython follows.

---

# Example: Function Body Is Compiled Before It Runs

Consider:

```python
def broken():
    return 1 / 0

print("function created")
```

Output:

```text
function created
```

No error occurs yet.

Why?

The function body is compiled and stored in a code object.

But it is not executed until the function is called.

Now:

```python
def broken():
    return 1 / 0

print("function created")
broken()
```

Output:

```text
function created
```

then a `ZeroDivisionError`.

The runtime error occurs when the function's bytecode executes inside a call frame.

---

# Example: Syntax Error Inside Function Body

Now compare:

```python
def broken():
    if True
        return 1

print("function created")
```

This does not print:

```text
function created
```

Why?

The syntax error prevents compilation.

The function object is never created.

The module does not begin normal execution.

This reinforces the distinction:

```text
syntax error:
    detected before bytecode execution

runtime error:
    detected during bytecode execution
```

---

# Common Misconceptions

## Misconception 1

### Bytecode is machine code.

Bytecode is not CPU machine code.

It is an instruction format for the Python Virtual Machine.

The CPU executes CPython.

CPython executes Python bytecode.

---

## Misconception 2

### Learning bytecode means memorizing every instruction.

The goal is not memorization.

The goal is understanding execution.

Instruction names can change across Python versions.

The mental model is more important:

```text
load values
operate on values
store names
call functions
jump for control flow
return values
```

---

## Misconception 3

### The Python Virtual Machine is a separate app.

The PVM is part of the Python runtime implementation.

When using CPython, it is part of CPython.

You usually do not launch it separately.

---

## Misconception 4

### A function body runs when Python sees `def`.

Executing `def` creates a function object.

The function body runs when the function is called.

The function's code object already exists, but execution waits until call time.

---

## Misconception 5

### Local variables are stored exactly like globals.

CPython can optimize local variable access.

That is why local variable bytecode may use fast local storage.

Global names require different lookup behavior.

---

## Misconception 6

### Bytecode explains every Python implementation.

This chapter focuses on CPython.

Other Python implementations may use different execution strategies.

For example, PyPy uses a JIT compiler.

The Python language is shared, but implementation internals can differ.

---

# Real-world Usage

## Debugging

Bytecode helps explain why errors occur where they do.

Syntax errors happen before bytecode execution.

Runtime errors happen while bytecode is executing.

Tracebacks show frame information because Python execution happens inside frames.

---

## Understanding Function Calls

When a function is called, Python creates a frame.

That frame has local variables, an evaluation stack, and an instruction position.

This explains:

* Local variables
* Recursion
* Tracebacks
* Return values
* Call stack behavior

---

## Understanding Imports

Imports compile and execute module code.

Cached bytecode may be stored in `__pycache__`.

This explains:

* Import-time side effects
* Startup time
* `.pyc` files
* Why module top-level code should be written carefully

---

## Understanding Performance

Bytecode helps explain why Python has overhead compared with native compiled languages.

For many operations, CPython must:

* Interpret bytecode.
* Work with Python objects.
* Perform dynamic type checks.
* Manage references.
* Handle possible exceptions.

This flexibility is powerful, but it has costs.

Later chapters will return to performance with better foundations.

---

## Understanding Tools

Many tools connect to Python's execution model.

Debuggers inspect frames.

Profilers observe function calls and execution time.

Coverage tools map executed bytecode back to source lines.

Linters and type checkers often work before bytecode execution.

The `dis` module exposes compiled instruction structure.

---

# Concept Connections

This chapter connects directly to earlier chapters:

```text
Chapter 01:
Software is instructions and data.

Chapter 02:
CPUs execute machine instructions.

Chapter 03:
Execution uses processes, stacks, and function calls.

Chapter 04:
The OS starts the Python process and provides file access.

Chapter 05:
Python source is designed for readability.

Chapter 06:
CPython is the implementation this chapter studies.

Chapter 07:
CPython compiles source code into bytecode.

Chapter 08:
The Python Virtual Machine executes bytecode.
```

This chapter also prepares the next phase:

```text
Bytecode loads and stores objects.
Objects have type, value, and identity.
Names refer to objects.
Mutation changes objects.
Equality compares objects.
```

That is exactly where the book goes next.

---

# Internal Mechanics Summary

The full runtime picture is:

```text
source code
    |
    v
AST
    |
    v
code object
    |
    v
bytecode instructions
    |
    v
execution frame
    |
    v
evaluation stack + namespaces
    |
    v
Python Virtual Machine loop
    |
    v
runtime behavior
```

Important terms:

| Term | Meaning |
| --- | --- |
| Bytecode | CPython instruction format for the PVM |
| PVM | Runtime machinery that executes bytecode |
| Code object | Compiled code plus metadata |
| Frame | Active execution context for a code object |
| Evaluation stack | Temporary stack used by bytecode execution |
| Namespace | Mapping from names to objects |
| Instruction pointer | Position of the next instruction to execute |

---

# Active Recall

## Easy Recall Questions

1. What is Python bytecode?
2. What executes Python bytecode in CPython?
3. Is bytecode the same as CPU machine code?
4. What module can show Python bytecode?
5. What is a code object?
6. What is a frame?
7. What is the evaluation stack used for?
8. What happens when a function is called?
9. What is `__pycache__` used for?
10. Why can bytecode output differ between Python versions?

---

## Deep Understanding Questions

1. Why does CPython use bytecode instead of repeatedly interpreting raw source text?
2. Why is bytecode called an intermediate representation?
3. Why does a function body not run when the `def` statement is executed?
4. Why does a function call need a new frame?
5. Why is stack-based execution useful for evaluating expressions?
6. Why are local variables often accessed differently from global names?
7. Why can syntax errors prevent bytecode execution entirely?
8. Why do tracebacks reflect the call stack?
9. Why is `__pycache__` an optimization rather than a source of truth?
10. Why should beginners learn the bytecode model without memorizing every instruction?

---

## Explain In Your Own Words

1. Explain bytecode to someone who knows only source code.
2. Explain the Python Virtual Machine.
3. Explain why bytecode is not machine code.
4. Explain how `2 + 3` can be evaluated using a stack.
5. Explain what happens when a function is called.
6. Explain how a traceback relates to frames.
7. Explain why importing a module may create cached bytecode.

---

## Predict-the-Output Questions

### Question 1

```python
def greet():
    print("hello")

print("done")
```

What is printed?

Answer:

```text
done
```

Reason:

The `def` statement creates a function object. The body does not run until the function is called.

---

### Question 2

```python
def greet():
    print("hello")

greet()
print("done")
```

What is printed?

Answer:

```text
hello
done
```

Reason:

Calling `greet()` creates a frame and executes the function body's bytecode.

---

### Question 3

```python
def broken():
    return 1 / 0

print("created")
```

What is printed?

Answer:

```text
created
```

Reason:

The function body is compiled but not executed.

---

### Question 4

```python
def broken():
    return 1 / 0

print("created")
broken()
print("done")
```

What happens?

Answer:

```text
created
```

Then Python raises `ZeroDivisionError`.

Reason:

The error occurs when the function's bytecode executes inside a call frame.

---

### Question 5

```python
x = 10
y = x
print(y)
```

What is printed?

Answer:

```text
10
```

Reason:

The bytecode stores the object represented by `10` under `x`, then loads `x` and stores the same value under `y`.

---

## Mental Model Questions

1. Draw the relationship between source code, bytecode, and the PVM.
2. Draw how `2 + 3` is evaluated using a stack.
3. Draw a function object pointing to a code object.
4. Draw the frame stack while function `a()` calls `b()` and `b()` calls `c()`.
5. Draw where local variables live during a function call.
6. Draw how a return value moves from a callee frame back to a caller frame.

---

# Practical Exercises

## Exercise 1

Disassemble a simple function:

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

Identify instructions that appear to:

* Load local variables.
* Perform an operation.
* Return a value.

Do not memorize the instruction names.

Write the execution story in plain English.

---

## Exercise 2

Inspect a code object:

```python
def answer():
    return 42

code = answer.__code__

print(code.co_name)
print(code.co_consts)
print(code.co_varnames)
```

Explain what each printed value represents.

---

## Exercise 3

Trace stack execution for:

```python
result = 2 + 3 * 4
```

Draw the conceptual stack after each operation:

```text
load 2
load 3
load 4
multiply
add
store result
```

Explain why the result is `14`, not `20`.

---

## Exercise 4

Compare definition time and call time:

```python
def show():
    print("inside")

print("outside")
```

Then:

```python
def show():
    print("inside")

show()
print("outside")
```

Explain the difference using function objects, code objects, and frames.

---

## Exercise 5

Create a traceback:

```python
def divide(a, b):
    return a / b

def run():
    return divide(10, 0)

run()
```

Read the traceback and identify:

* The module frame.
* The `run` frame.
* The `divide` frame.
* The line where runtime execution failed.

---

## Exercise 6

Create two files:

```python
# helper.py
def double(x):
    return x * 2
```

```python
# main.py
import helper

print(helper.double(5))
```

Run:

```bash
python main.py
```

Look for `__pycache__`.

Explain why cached bytecode may appear for `helper.py`.

---

# Summary

In this chapter we learned:

* CPython compiles Python source code into bytecode.
* Bytecode is an instruction format for the Python Virtual Machine.
* Bytecode is not CPU machine code.
* The CPU executes CPython; CPython executes Python bytecode.
* The `dis` module can show bytecode.
* Exact bytecode output can vary across Python versions.
* Code objects contain bytecode and metadata.
* Function objects reference code objects.
* Executing code requires a frame.
* Function calls create new frames.
* Frames contain runtime state such as local variables, an evaluation stack, and an instruction position.
* CPython bytecode uses stack-based execution.
* Assignments, arithmetic, conditionals, loops, function definitions, and function calls all become bytecode operations.
* Tracebacks are connected to frames and the call stack.
* `__pycache__` stores cached bytecode for imported modules.

The core mental model is:

```text
source code
  -> code object
  -> bytecode
  -> frame
  -> evaluation stack and namespaces
  -> Python Virtual Machine
  -> runtime behavior
```

You now understand how CPython moves from compiled instructions to actual execution.

---

# Preview of Chapter 09

Bytecode does not operate on vague values.

It operates on Python objects.

When bytecode loads `10`, it loads an integer object.

When bytecode loads `"hello"`, it loads a string object.

When bytecode calls a function, it calls a function object.

In the next chapter, we begin the most important Python mental model:

> Everything is an object.
