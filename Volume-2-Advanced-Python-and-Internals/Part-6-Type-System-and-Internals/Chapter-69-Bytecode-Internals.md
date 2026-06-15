# Chapter 69 — Bytecode Internals

Volume I introduced bytecode.

We learned that CPython does not execute source code directly.

The path is:

```text
source code
    -> tokens
    -> syntax tree
    -> code object
    -> bytecode
    -> Python Virtual Machine execution
```

That first explanation was enough for foundations.

Now we go deeper.

Bytecode is the instruction format CPython's evaluation loop executes.

It is not CPU machine code.

It is not the Python language specification.

It is an implementation detail of CPython.

But it is an extremely useful implementation detail to understand.

Bytecode helps explain:

* why functions create code objects
* how local variables are loaded
* how globals and builtins are searched
* how constants are stored
* how function calls are represented
* how attribute access becomes runtime operations
* how control flow jumps
* how exceptions are handled
* why Python versions can change performance
* why introspection tools can inspect execution

Do not study bytecode to memorize every instruction.

Study bytecode to sharpen your runtime model.

The goal is:

```text
see Python source code as operations over objects, names, frames, and runtime protocols
```

---

# Bytecode Is CPython's Instruction Language

CPython compiles Python source into bytecode.

Bytecode is a sequence of instructions for the Python Virtual Machine.

Example source:

```python
x = 2 + 3
```

At a high level, CPython needs to:

```text
load constants
perform operation or use folded constant
store result in name x
```

The exact bytecode can vary by Python version.

That warning matters.

Bytecode is not stable across Python versions.

Instructions are added.

Instructions are removed.

Instructions are specialized.

Optimizations change.

Exception handling layout changes.

Function call bytecode changes.

So the right attitude is:

```text
understand the model, not a frozen list of opcodes
```

---

# The dis Module

Python's `dis` module disassembles bytecode.

Example:

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

The output depends on your Python version.

It may look roughly like:

```text
LOAD_FAST
LOAD_FAST
BINARY_OP
RETURN_VALUE
```

The exact formatting and instruction names can differ.

`dis` lets you inspect:

* functions
* methods
* code objects
* classes
* source snippets compiled with `compile()`

Example with a code object:

```python
code = compile("x = 1 + 2", "<string>", "exec")
dis.dis(code)
```

`dis` is a learning tool, debugging tool, and introspection tool.

It is not something you use in ordinary application logic.

---

# Code Objects

A code object represents compiled executable Python code.

Functions contain code objects.

Example:

```python
def greet(name):
    return f"hello {name}"

print(greet.__code__)
```

The function object is not the same as the code object.

The function object contains:

* the code object
* global namespace reference
* default arguments
* closure cells
* annotations
* function name
* other metadata

The code object contains compiled information:

* bytecode
* constants
* variable names
* free variable names
* filename
* first line number
* flags
* exception table information

The relationship:

```text
function object
    -> code object
        -> bytecode instructions
```

This is why two functions can have similar source but different runtime metadata.

---

# Inspecting a Code Object

Code objects expose attributes.

Example:

```python
def add(a, b):
    total = a + b
    return total

code = add.__code__

print(code.co_name)
print(code.co_varnames)
print(code.co_consts)
print(code.co_names)
```

Possible output:

```text
add
('a', 'b', 'total')
(None,)
()
```

`co_varnames` includes local variable names.

`co_consts` includes constants used by the code object.

`co_names` includes names referenced by some instructions, often globals or attributes.

Example:

```python
def shout(text):
    return text.upper()
```

The name `upper` may appear in `co_names` because the bytecode performs attribute lookup.

The exact details vary, but the idea is stable:

```text
code objects store the metadata bytecode needs during execution
```

---

# Constants

Constants used by a code object live in `co_consts`.

Example:

```python
def example():
    return "hello", 42, None

print(example.__code__.co_consts)
```

You may see:

```python
(None, 'hello', 42)
```

The compiler stores constants in the code object.

Bytecode instructions can refer to constants by index.

Conceptually:

```text
LOAD_CONST 1  -> load 'hello'
LOAD_CONST 2  -> load 42
```

This is not unlike a function carrying a small table of values its instructions need.

Constants are part of compiled code metadata.

They are not looked up by name each time.

---

# Names

Code objects also store names.

Example:

```python
value = 10

def read_global():
    return value
```

The name `value` is not a local variable inside `read_global`.

It is referenced as a global name.

The code object may include it in `co_names`.

At runtime, the frame uses that name to search:

```text
globals
builtins
```

This connects back to Volume I's namespaces chapter.

Bytecode does not contain the object `value` refers to.

It contains the instruction to load the name.

The actual object is found at runtime.

That is why rebinding a global can change what a function sees:

```python
value = 10

def read_global():
    return value

value = 99

print(read_global())
```

The function returns `99`.

The bytecode loads the name at runtime.

---

# Local Variables

Local variables are handled differently from globals.

Example:

```python
def compute(a, b):
    total = a + b
    return total
```

Names `a`, `b`, and `total` are local to the function.

The compiler can assign them local variable slots.

Bytecode may use instructions such as:

```text
LOAD_FAST
STORE_FAST
```

These are faster than dictionary-style name lookup.

Why?

Because local variable names in a function are known at compile time.

CPython can store them in an array-like structure inside the frame.

Conceptual model:

```text
local slot 0 -> a
local slot 1 -> b
local slot 2 -> total
```

This is why local variable access is generally faster than global lookup.

The compiler can optimize locals because their scope is known.

---

# Globals and Builtins

Global lookup is more dynamic.

Example:

```python
def length(value):
    return len(value)
```

`len` is not local.

Python looks it up as a global name, then in builtins if not found globally.

Conceptually:

```text
look in function globals for len
if missing, look in builtins for len
call the object found
```

This is why shadowing builtins can change behavior:

```python
len = 10

def length(value):
    return len(value)
```

Now `len` refers to an integer in globals.

Calling it fails.

Bytecode follows name lookup rules.

It does not know that you meant the built-in `len`.

Names are resolved at runtime according to namespaces.

---

# Frames

A frame represents an executing code object.

When a function is called, CPython creates a frame for that call.

The frame contains:

* the code object being executed
* local variables
* global namespace
* builtins
* instruction pointer
* evaluation stack
* exception state
* links needed for tracing/debugging

Conceptually:

```text
function call
    -> new frame
        -> executes code object's bytecode
```

If a function calls another function, another frame is created.

This forms the call stack.

Volume I explained frames conceptually.

Here we connect frames to bytecode:

```text
bytecode is executed inside frames
frames hold the runtime state needed by bytecode
```

---

# The Evaluation Stack

CPython bytecode is stack-based.

Many instructions push values onto an evaluation stack or pop values from it.

Example expression:

```python
a + b
```

Conceptually:

```text
load a      -> push value of a
load b      -> push value of b
add         -> pop two values, push result
```

The evaluation stack is not the same as the call stack.

Call stack:

```text
which functions are currently active
```

Evaluation stack:

```text
temporary values inside one executing frame
```

This distinction is important.

Each frame has its own evaluation state.

Nested function calls create new frames.

Expressions inside a frame use that frame's evaluation stack.

---

# Disassembling a Function

Example:

```python
import dis

def compute(a, b):
    total = a + b
    return total

dis.dis(compute)
```

You might see instructions conceptually like:

```text
RESUME
LOAD_FAST       a
LOAD_FAST       b
BINARY_OP       +
STORE_FAST      total
LOAD_FAST       total
RETURN_VALUE
```

Do not worry if your output differs.

The broad flow is:

```text
start frame
load local a
load local b
perform addition
store local total
load local total
return it
```

Bytecode makes explicit what source code hides.

Source code:

```python
total = a + b
```

Bytecode model:

```text
load values
operate
store result
```

---

# Function Calls

Function calls are more involved than they appear.

Source:

```python
result = func(a, b)
```

At runtime, Python must:

* load the callable object
* load arguments
* arrange the call
* invoke the callable protocol
* receive the return value
* store the result

Conceptual bytecode flow:

```text
load func
load a
load b
call
store result
```

Function calls create frames when the callable is a Python function.

But not every callable creates a Python frame in the same way.

Built-in functions and C extension functions may execute in native code.

Callable objects use `__call__`.

Methods involve descriptor binding before the call.

The bytecode instruction says "call this callable."

The runtime then follows Python's callable machinery.

---

# Method Calls

Method calls combine attribute access and function calling.

Source:

```python
text.upper()
```

The runtime must:

* load `text`
* find attribute `upper`
* bind the method if needed
* call it
* return the result

This connects to descriptors.

Functions stored on classes are descriptors.

When accessed through an instance, they produce bound methods.

Bytecode has evolved over Python versions to optimize method calls.

The exact instruction names may change.

The conceptual flow remains:

```text
object -> attribute lookup -> callable -> call
```

Method calls are not primitive magic.

They are runtime operations over objects and types.

---

# Attribute Access

Source:

```python
value = obj.name
```

The bytecode performs attribute access.

Conceptually:

```text
load obj
load attribute name
store result in value
```

Attribute access may involve:

* instance dictionary
* class dictionary
* descriptors
* properties
* `__getattribute__`
* `__getattr__`
* MRO search
* slots

The bytecode instruction does not contain all of that logic.

It delegates to runtime attribute lookup.

This is the pattern:

```text
bytecode selects an operation
runtime object model performs the operation
```

That is why earlier chapters on descriptors, properties, MRO, and `__slots__` matter.

Bytecode triggers those mechanisms.

It does not replace them.

---

# Control Flow

Conditionals and loops compile into jumps.

Example:

```python
if value:
    result = "yes"
else:
    result = "no"
```

Conceptually:

```text
load value
test truthiness
jump if false to else block
load "yes"
store result
jump past else block
load "no"
store result
```

Loops also use jumps.

Example:

```python
for item in items:
    handle(item)
```

Conceptually:

```text
get iterator from items
get next item
if iterator exhausted, exit loop
store item
call handle(item)
jump back to get next item
```

This connects directly to the iterator protocol.

The bytecode manages control flow.

The object model provides iteration behavior.

---

# Loops and Iterators

`for` loops are not special to lists.

Source:

```python
for item in iterable:
    process(item)
```

Runtime model:

```text
iterator = iter(iterable)
repeat:
    item = next(iterator)
    process(item)
stop when StopIteration occurs
```

Bytecode uses instructions that implement this loop pattern.

When the iterator is exhausted, the loop exits.

This is why any object that implements the iterator protocol can work in a `for` loop.

Bytecode does not care whether the object is a list, tuple, generator, file object, or custom iterator.

It asks for iteration behavior at runtime.

Again:

```text
syntax -> bytecode operation -> runtime protocol
```

---

# Exception Handling

Exception handling also compiles into bytecode and metadata.

Source:

```python
try:
    risky()
except ValueError:
    recover()
```

The runtime must know:

* where protected code begins
* where handlers live
* which exceptions match
* where to continue after handling
* how to unwind stack state

Modern CPython uses exception table information associated with code objects.

The exact representation has changed across versions.

The conceptual model:

```text
execute protected block
if exception occurs, consult handler metadata
match exception type
run handler or propagate
```

Exception handling is not just a high-level idea.

It is represented in compiled code metadata and runtime frame state.

This is one reason traceback information can point to precise source locations.

---

# Line Numbers and Debugging

Code objects contain information that maps bytecode back to source locations.

This supports:

* tracebacks
* debuggers
* coverage tools
* profilers
* tracing hooks

When an exception occurs, Python can show:

```text
file name
line number
function name
source line
```

That information comes from the relationship between:

* code objects
* frames
* line number tables
* traceback objects

This is why tracebacks are more than error strings.

They are structured runtime information about frame execution.

---

# Specialized Bytecode

Modern CPython can specialize bytecode at runtime.

This means instructions may adapt based on the types and operations actually seen during execution.

For example, loading an attribute from a common object shape may become optimized internally.

This is part of CPython's adaptive interpreter work.

The important idea:

```text
bytecode you see may include adaptive behavior or cache entries depending on options and Python version
```

The `dis` module has options to show more detail in some versions.

Do not panic when disassembly includes instructions or cache information you did not expect.

The stable lesson is:

```text
CPython can optimize common runtime patterns while preserving Python semantics
```

The implementation changes.

The language behavior remains the contract.

---

# Bytecode and Performance

Bytecode can help explain performance, but it should not be your only performance tool.

Example:

```python
def use_local(value):
    local_len = len
    return local_len(value)
```

Older advice sometimes suggested binding globals to locals because local lookup can be faster.

Modern CPython optimizations can change whether such micro-optimizations matter.

The lesson:

```text
understand bytecode for insight
measure performance before optimizing
```

Bytecode can reveal:

* repeated global lookups
* function call overhead
* attribute access
* loop structure
* constant folding
* closure access

But real performance depends on:

* algorithm choice
* data structures
* I/O
* allocation behavior
* interpreter version
* C extension behavior
* workload size

Do not optimize from disassembly alone.

Use profiling.

---

# Constant Folding

The compiler may simplify some constant expressions.

Example:

```python
def value():
    return 2 + 3
```

The compiler may store `5` as a constant rather than emitting runtime addition.

Disassembly might show loading `5`.

This is called constant folding.

The compiler can optimize expressions that are safe to compute ahead of time.

But it cannot fold everything.

Example:

```python
def value(x):
    return x + 3
```

The value of `x` is not known at compile time.

The addition must happen at runtime.

The idea:

```text
compile-time known values can sometimes become constants
runtime-dependent values become bytecode operations
```

---

# Closures and Free Variables

Closures require bytecode to access variables from enclosing scopes.

Example:

```python
def make_multiplier(factor):
    def multiply(value):
        return value * factor
    return multiply
```

`factor` is not local to `multiply`.

It is a free variable captured from the enclosing scope.

Code objects expose closure-related names:

```python
fn = make_multiplier(10)

print(fn.__code__.co_freevars)
print(fn.__closure__)
```

`co_freevars` tells you which free variable names the code uses.

`__closure__` contains cell objects holding captured values.

Bytecode uses special mechanisms to load from closure cells.

This connects Volume I closures to runtime internals:

```text
closures are not magic memory
they are code objects plus cell references
```

---

# Comprehensions Have Code Objects

Comprehensions can create their own hidden function-like code objects.

Example:

```python
def squares(values):
    return [value * value for value in values]
```

If you inspect constants:

```python
print(squares.__code__.co_consts)
```

you may see a nested code object for the list comprehension.

This helps explain why comprehension loop variables do not leak into the surrounding scope in modern Python.

The comprehension has its own scope-like execution context.

Again, exact details vary.

The important idea:

```text
some syntax creates hidden compiled code objects
```

Functions are not the only source of code objects.

---

# Generators and Bytecode

Generator functions compile differently from ordinary functions.

Example:

```python
def numbers():
    yield 1
    yield 2
```

Calling `numbers()` returns a generator object.

The function body does not run immediately.

The generator object holds execution state.

Each `next()` resumes the frame until the next `yield`.

Bytecode for generators includes yield-related instructions.

The conceptual difference:

```text
ordinary function:
    call -> run to return or exception

generator function:
    call -> create generator object
    next -> run until yield
    next -> resume after yield
```

The generator chapter explained the behavior.

Bytecode internals show that the behavior is represented in compiled execution machinery.

---

# Async Functions and Bytecode

Async functions also compile into special code objects.

Example:

```python
async def fetch():
    await operation()
```

Calling `fetch()` returns a coroutine object.

The body does not run immediately.

The coroutine is driven by an event loop or by awaiting it.

Bytecode contains await-related operations.

Conceptually:

```text
async function call -> coroutine object
await -> suspend until awaitable completes
resume -> continue execution
```

This connects Chapter 67 to internals.

Async behavior is not a separate universe.

It is another form of compiled code object plus runtime execution state.

---

# Bytecode Is Not a Stable Contract

This point deserves repetition.

Bytecode is an implementation detail.

Do not write normal application code that depends on exact bytecode instruction sequences.

Different Python versions can change:

* instruction names
* instruction arguments
* stack effects
* call conventions
* exception handling representation
* line number tables
* specialization behavior
* optimization choices

Code that inspects bytecode must be version-aware.

Examples:

* debuggers
* profilers
* coverage tools
* linters
* educational tools
* advanced instrumentation

For ordinary applications, bytecode is for understanding, not dependency.

The Python language behavior is the contract.

The bytecode is CPython's current strategy.

---

# Why Bytecode Still Matters

If bytecode is unstable, why learn it?

Because it clarifies the execution model.

Bytecode helps you understand:

* why function bodies do not run at definition time
* why local variables are different from globals
* why closures need cells
* why calls have overhead
* why loops depend on iterator protocol
* why comprehensions have their own scope
* why tracebacks know line numbers
* why interpreter versions can change performance
* why Python can introspect execution

Bytecode is like an X-ray.

You do not build the whole house out of X-rays.

But when you need to understand the structure underneath, it is invaluable.

---

# Common Mistakes

Do not treat bytecode as Python source code.

It is compiled instruction data for CPython's virtual machine.

Do not assume bytecode is CPU machine code.

The CPU does not directly execute Python bytecode.

Do not memorize bytecode instruction lists as if they were stable language syntax.

They change across versions.

Do not assume disassembly from one Python version applies exactly to another.

Check the version.

Do not use bytecode inspection instead of profiling for performance decisions.

It can inform profiling, not replace it.

Do not assume one source line maps to one bytecode instruction.

One line may compile into many instructions, and some instructions may not correspond neatly to a visible line.

Do not forget that bytecode delegates to runtime protocols.

`LOAD_ATTR` or equivalent attribute access still invokes Python's attribute machinery.

Do not forget that code objects are not function objects.

A function object contains a code object plus runtime context and metadata.

---

# Mental Checklist

When inspecting bytecode, ask:

* Which Python version produced this output?
* Am I looking at a function, code object, class, or expression?
* Which names are local?
* Which names are global or builtins?
* Which constants are stored in the code object?
* Where are function calls?
* Where are attribute lookups?
* Where are jumps?
* Does this code create nested code objects?
* Are closures involved?
* Are generators or async functions involved?
* Am I learning a model or depending on an implementation detail?

Bytecode inspection is most valuable when guided by a question.

Do not stare at disassembly hoping meaning appears.

Ask what source behavior you are trying to explain.

---

# Exercises

1. Use `dis.dis()` on a function that returns `a + b`. Identify local loads, the binary operation, and return.

2. Inspect `__code__.co_varnames`, `co_consts`, and `co_names` for a simple function.

3. Compare bytecode for a function using a local variable and a global variable.

4. Disassemble a function that calls `len(value)`. Identify where the name `len` appears.

5. Disassemble a method call such as `text.upper()`. Explain the attribute lookup and call conceptually.

6. Disassemble an `if` statement and identify jump instructions.

7. Disassemble a `for` loop and connect it to the iterator protocol.

8. Inspect a closure's `co_freevars` and `__closure__`.

9. Inspect `co_consts` for a function containing a list comprehension.

10. Disassemble a generator function and compare it with an ordinary function.

11. Disassemble an async function and identify that the output differs from ordinary functions.

12. Try the same disassembly on two Python versions if available. Note differences.

13. Find an example of constant folding by disassembling a function returning a constant expression.

14. Explain why relying on exact bytecode sequences is risky.

---

# Summary

CPython compiles Python source code into code objects containing bytecode.

Bytecode is an instruction format for the Python Virtual Machine, not CPU machine code.

The `dis` module disassembles bytecode for inspection.

Function objects contain code objects.

Code objects store bytecode, constants, names, variable names, flags, source metadata, and other execution information.

Frames execute code objects.

Frames hold local variables, global and builtin namespace references, instruction state, and an evaluation stack.

Local variables can be accessed through optimized local slots.

Global names are resolved through global and builtin namespaces at runtime.

Function calls load callables and arguments, invoke callable behavior, and may create new frames.

Method calls combine attribute lookup, descriptor behavior, and calling.

Attribute access delegates to Python's runtime object model.

Conditionals and loops compile into jumps.

`for` loops compile into iterator protocol operations.

Exception handling uses bytecode plus code-object metadata such as exception tables.

Closures use free variable metadata and cell objects.

Comprehensions can create nested code objects.

Generators and async functions compile into special resumable execution forms.

Bytecode changes across Python versions and should not be treated as stable application API.

The deep lesson is:

```text
bytecode reveals how source syntax becomes runtime operations over frames, names, objects, and protocols
```

Once you see that, Python's execution model becomes less magical and more mechanical.

---

# Preview of Chapter 70

Chapter 69 studied bytecode internals.

We saw how code objects, frames, instructions, constants, names, jumps, calls, closures, generators, async functions, and exception metadata fit together.

Next we zoom out to CPython architecture.

Chapter 70 will study the implementation that makes these pieces work:

* CPython as the reference implementation
* parser and compiler pipeline
* code objects
* frame evaluation
* object representation
* reference counting
* garbage collection
* the GIL
* memory allocation
* interpreter state
* module initialization
* extension boundaries

The transition is:

```text
bytecode explains the instruction stream
CPython architecture explains the machine that executes it
```

The next chapter connects the execution model to CPython's larger runtime design.
