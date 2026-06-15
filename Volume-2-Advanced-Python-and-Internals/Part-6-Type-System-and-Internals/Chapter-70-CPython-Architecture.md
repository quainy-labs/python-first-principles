# Chapter 70 — CPython Architecture

CPython is the reference implementation of Python.

When most people say "Python," they usually mean CPython.

But Python the language and CPython the implementation are not the same thing.

The language defines behavior:

```text
what Python code means
```

CPython is one implementation:

```text
one program that executes Python code according to that behavior
```

There are other implementations:

* PyPy
* Jython
* IronPython
* MicroPython
* GraalPy

They share language ideas, but they can differ internally.

CPython is written primarily in C and Python.

It parses source code, compiles it to bytecode, creates objects, manages memory, runs frames, handles imports, loads extension modules, and provides the C API that much of the Python ecosystem depends on.

Chapter 69 looked at bytecode.

This chapter zooms out.

We now ask:

```text
what larger runtime architecture executes that bytecode?
```

---

# Why CPython Matters

CPython matters because it is:

* the default Python implementation
* the reference implementation
* the target for most Python packages
* the runtime behind most production Python deployments
* the implementation that defines the behavior many developers observe
* the provider of the Python/C API used by native extensions

Many things people call "Python behavior" are actually CPython behavior.

Examples:

* reference counting
* prompt object finalization in many cases
* the Global Interpreter Lock
* `.pyc` cache format details
* bytecode instruction choices
* C extension compatibility

Some of these are not guaranteed by the Python language itself.

This distinction matters.

If you depend on CPython-specific behavior, your code may not behave the same on another implementation.

Most application code does not need to care.

But advanced Python engineers should know where the boundary is.

---

# The Big Pipeline

CPython execution can be viewed as a pipeline:

```text
source code
    -> tokenizer
    -> parser
    -> abstract syntax tree
    -> compiler
    -> code object
    -> bytecode
    -> frame evaluation
    -> object operations
```

This is a simplification.

But it is the right mental map.

Each stage has a job.

Tokenizer:

```text
turn source text into tokens
```

Parser:

```text
turn tokens into syntax structure
```

Compiler:

```text
turn syntax tree into code objects
```

Evaluation loop:

```text
execute bytecode instructions inside frames
```

Object system:

```text
provide runtime behavior for values, attributes, calls, operators, and protocols
```

Memory manager:

```text
allocate, track, and release object memory
```

The interpreter is not one idea.

It is a set of cooperating subsystems.

---

# Source Text to Tokens

Python source code begins as text.

Example:

```python
total = price * quantity
```

CPython first needs to understand the text as meaningful pieces.

Tokens include things like:

* names
* numbers
* strings
* operators
* punctuation
* indentation
* newlines
* keywords

Indentation is part of Python syntax.

So the tokenizer must represent indentation changes too.

Example:

```python
if active:
    send_email()
```

The indentation after `if active:` is not decoration.

It defines the block.

The tokenizer produces tokens that let later stages understand the structure.

If text cannot be tokenized correctly, execution never begins.

---

# Parsing and Syntax Trees

After tokenization, CPython parses the token stream.

Parsing checks whether the tokens form valid Python syntax.

Example:

```python
if active
    send_email()
```

This is invalid because the colon is missing.

The parser raises `SyntaxError` before bytecode exists.

When parsing succeeds, CPython builds an abstract syntax tree, often called an AST.

An AST represents source structure.

Example:

```python
total = price * quantity
```

Conceptually:

```text
assignment
    target: total
    value:
        multiplication
            left: price
            right: quantity
```

The AST is not bytecode.

It is a structured representation of source meaning.

The compiler turns that structure into executable code objects.

---

# Compilation to Code Objects

CPython compiles ASTs into code objects.

Code objects contain bytecode and metadata.

A module has a code object.

Each function has a code object.

Comprehensions may have nested code objects.

Generator and async functions have code objects with special flags.

Example:

```python
def add(a, b):
    return a + b
```

The `def` statement creates a function object at runtime.

That function object references a code object.

The code object contains the compiled instructions for the function body.

This relationship matters:

```text
source defines function
compiler creates code object
runtime creates function object
function object uses code object when called
```

This is how Python can treat functions as objects while still executing compiled bytecode.

---

# Module Execution

When CPython runs a `.py` file, the file is compiled into a module code object.

Then that code object is executed in a module namespace.

Top-level statements run immediately.

Example:

```python
print("loading")

def greet():
    print("hello")
```

When the module executes:

1. `print("loading")` runs.
2. The `def` statement creates a function object.
3. The name `greet` is bound to that function object.

The body of `greet()` does not run until the function is called.

This explains imports too.

Importing a module executes its top-level code the first time it is imported.

The resulting module object is cached in `sys.modules`.

The import system is part of CPython's runtime architecture.

---

# Frames and Evaluation

Bytecode executes inside frames.

A frame contains the runtime state needed to execute a code object.

When a function is called, CPython creates a frame.

The frame knows:

* which code object is executing
* local variables
* global namespace
* builtins
* current instruction position
* evaluation stack
* exception state
* tracing and debugging information

The evaluation loop reads bytecode instructions and performs operations.

Conceptually:

```text
fetch instruction
decode instruction
execute instruction
move to next instruction
```

This loop is the heart of the interpreter.

The implementation is more complex than this simple model, especially in modern CPython with specialization and caches.

But the mental model remains:

```text
frames execute code objects instruction by instruction
```

---

# The Evaluation Stack

CPython bytecode is stack-based.

Inside a frame, temporary values are pushed and popped from an evaluation stack.

Example:

```python
result = a + b
```

Conceptual execution:

```text
load a
push value of a
load b
push value of b
add
pop a and b
push result
store result
```

This stack is not the same as the call stack.

Call stack:

```text
which function calls are active
```

Evaluation stack:

```text
temporary values within one active frame
```

The call stack is made of frames.

Each frame has its own evaluation state.

---

# Object Representation

At the C level, Python objects have a common object header.

The exact details are implementation-specific, but the idea is:

```text
object memory contains bookkeeping plus type-specific data
```

Core bookkeeping includes:

* reference count information
* pointer to the object's type

That type pointer connects every object to its runtime behavior.

When Python evaluates:

```python
value.upper()
```

the runtime uses object and type information to perform attribute lookup and method binding.

When Python evaluates:

```python
len(value)
```

the runtime asks the object's type whether it supports the length protocol.

This is why the object header matters conceptually.

Every object carries a link to its type.

That type controls behavior.

---

# PyObject and PyTypeObject

In CPython's C implementation, `PyObject` is the basic object structure concept.

Specific object types extend that structure with extra fields.

An integer object needs integer storage.

A list object needs storage for references to elements.

A dictionary object needs hash table machinery.

A function object needs references to code, globals, defaults, closure cells, and metadata.

The type object describes behavior.

At a high level, a type object includes slots for operations such as:

* representation
* attribute access
* calling
* numeric operations
* sequence operations
* mapping operations
* deallocation

This connects directly to dunder methods.

Python-level methods and C-level type slots cooperate to make syntax work.

The user sees:

```python
x + y
```

CPython sees:

```text
perform addition according to the runtime types
```

---

# Reference Counting

CPython primarily manages object lifetime with reference counting.

Each object tracks how many strong references point to it.

When a new strong reference is created, the count increases.

When a strong reference is released, the count decreases.

When the count reaches zero, the object can be deallocated.

Example:

```python
a = []
b = a
del a
del b
```

Conceptually:

```text
[] created
a references it
b references it too
del a removes one reference
del b removes final reference
object can be destroyed
```

This is why CPython often finalizes objects promptly.

But reference counting alone cannot collect reference cycles.

That requires cyclic garbage collection.

---

# Reference Counting at the C API Boundary

The C API exposes reference ownership rules.

Extension authors must understand:

* strong references
* borrowed references
* new references
* `Py_INCREF`
* `Py_DECREF`
* `Py_XINCREF`
* `Py_XDECREF`

If C extension code decrements too many times, it can use freed memory.

If it forgets to decrement, it can leak objects.

This is one reason C extensions are powerful but dangerous.

At Python level, you usually do not manually increment or decrement reference counts.

At C level, reference ownership is part of correctness.

Chapter 71 will go deeper into extension boundaries.

For this chapter, remember:

```text
CPython object lifetime depends heavily on reference ownership
```

---

# Cyclic Garbage Collection

Reference counting cannot collect cycles by itself.

Example:

```python
a = []
b = []
a.append(b)
b.append(a)
```

If no outside names refer to `a` or `b`, the two lists still refer to each other.

Their reference counts are not zero.

Reference counting alone would not free them.

CPython includes a cyclic garbage collector to find groups of objects that are unreachable except through cycles.

The garbage collector focuses on container objects that can participate in cycles.

Examples:

* lists
* dictionaries
* sets
* class instances
* frames

Simple objects like many integers and strings do not need cycle tracking in the same way.

The combined model:

```text
reference counting handles most lifetime events promptly
cyclic GC handles unreachable reference cycles
```

---

# Memory Allocation

CPython manages a private heap for Python objects and internal data structures.

The Python memory manager sits between Python object allocation and the operating system's memory allocator.

It uses different allocation domains for different purposes.

At a high level:

```text
raw memory allocation
Python memory allocation
object allocation
```

Object allocation is specialized for Python objects.

Different object types may have different allocation strategies.

Small objects may use specialized allocators for speed.

Some objects may be cached or reused internally.

This is why `id()` values can sometimes appear reused after objects are destroyed.

The memory address becomes available for another object.

Do not build application logic on memory addresses.

---

# The Private Heap

Python objects live in memory managed by CPython's private heap.

This does not mean memory is outside the operating system.

The operating system still provides memory to the process.

But CPython manages Python object allocation within that process.

The private heap lets CPython:

* allocate objects efficiently
* use object-specific allocators
* track memory behavior
* integrate allocation with garbage collection
* support debugging hooks

Extension authors must use the correct allocation APIs.

Mixing allocators can corrupt memory.

For example, memory allocated with one Python memory API should be released with the matching Python memory API.

This becomes important in Chapter 71.

---

# The Global Interpreter Lock

The Global Interpreter Lock, or GIL, is part of ordinary CPython builds.

It ensures that only one thread executes Python bytecode at a time in a given interpreter.

Chapter 66 studied the GIL from the concurrency perspective.

Here we place it architecturally.

The GIL simplifies parts of CPython's runtime:

* object reference counting
* interpreter state access
* many C API assumptions
* internal object mutation safety

Without a GIL, many internal operations would need finer-grained synchronization.

That does not mean the GIL makes application code race-free.

It does not.

The GIL is an interpreter-level lock.

Application invariants still need locks, queues, immutability, or ownership rules.

---

# Interpreter State and Thread State

CPython tracks interpreter state and thread state.

Interpreter state contains runtime-wide information for an interpreter.

Thread state contains information for a thread executing Python code.

Conceptually:

```text
interpreter state
    modules
    builtins
    import machinery
    runtime configuration
    memory/runtime structures

thread state
    current frame
    exception state
    recursion depth
    execution context
```

C extensions that interact with Python must be aware of thread state when calling into the API.

This is especially important when native code releases the GIL and later reacquires it.

At Python level, this machinery is mostly hidden.

But it explains why embedding Python, writing extensions, and free-threaded builds are complex.

---

# Imports and Module Objects

The import system is part of runtime architecture.

When importing a module, CPython:

1. Checks `sys.modules`.
2. Finds a module specification.
3. Loads source or extension code.
4. Creates or reuses a module object.
5. Executes module code if needed.
6. Stores the module in `sys.modules`.
7. Binds names in the importing namespace.

This architecture supports:

* Python source modules
* packages
* built-in modules
* frozen modules
* extension modules
* custom import hooks

Module objects are runtime objects.

They have dictionaries.

Top-level names live in module namespaces.

This is why imports are execution, not only file loading.

---

# Built-in and Extension Modules

Some modules are built into CPython.

Others are written in Python.

Others are extension modules written in C or another native language and exposed through compatible interfaces.

Examples of extension-backed modules and packages are common in the ecosystem.

Native extensions are used for:

* speed
* system integration
* bindings to C libraries
* numerical computing
* compression
* cryptography
* image processing

The extension boundary is powerful because native code can operate close to the system.

It is risky because native code can crash the interpreter if it violates memory or reference rules.

Pure Python exceptions are controlled runtime events.

C memory corruption is not.

---

# The C API

CPython exposes a C API.

The C API lets native code:

* create Python objects
* call Python functions
* define extension types
* raise Python exceptions
* manage references
* access modules
* interact with interpreter state

This API is one reason the Python ecosystem is so powerful.

It allows libraries like numerical packages and system bindings to integrate deeply with Python.

But the C API also ties code to CPython's object model and runtime assumptions.

Some APIs are stable.

Some are version-specific.

Some are internal and should not be used by extensions.

Chapter 71 will focus on C extensions directly.

For now, understand:

```text
the C API is the bridge between Python objects and native code
```

---

# Stable ABI and Limited API

Extension modules can target different levels of compatibility.

Some extensions use the full CPython C API.

That gives power but can tie the extension to specific CPython versions.

The limited API and stable ABI are designed to improve binary compatibility across Python versions.

This matters for package distribution.

If an extension depends on CPython internals, it may need separate builds for different Python versions.

If it uses a stable ABI subset, distribution can be easier.

The tradeoff:

```text
more internal access -> more power, less portability
stable ABI subset -> more compatibility, less access
```

This is a recurring engineering theme.

Boundaries give safety.

They also limit what you can do.

---

# Object-Specific Implementations

Built-in types have specialized implementations.

Lists are dynamic arrays of references.

Dictionaries are hash tables with sophisticated layout and lookup behavior.

Sets are hash-based collections.

Tuples are fixed-size immutable containers.

Strings have Unicode-aware internal representation.

Integers use arbitrary-precision representation.

These details explain why built-in types have different performance characteristics.

Example:

```text
list append at end is usually efficient
dict lookup is usually fast
tuple size cannot change
int does not overflow like fixed-width C integers
```

The language gives high-level behavior.

CPython provides concrete data structures to implement that behavior.

Understanding this helps you choose data structures wisely.

---

# Optimizations and Caches

Modern CPython includes many optimizations.

Examples include:

* bytecode specialization
* inline caches
* optimized local variable access
* object freelists in some areas
* efficient dictionary layouts
* vectorcall calling convention for many call paths
* optimized import caches

You do not need to memorize every optimization.

But you should understand that CPython is not a naive interpreter.

It constantly balances:

* correctness
* compatibility
* performance
* maintainability
* debugging support
* extension compatibility

Some optimizations are invisible at the Python level.

Others show up in performance changes across Python versions.

That is why the same Python code can become faster when run on a newer CPython release.

---

# Error Handling

At Python level, errors are exceptions.

At C level, CPython often represents errors through a combination of:

* return values
* exception state
* error indicators

A C API function may return `NULL` to indicate failure and set a Python exception.

The caller must check.

If extension code ignores error indicators, it can create confusing behavior or crashes.

At Python level:

```python
raise ValueError("bad")
```

At C level, extension code must explicitly set an exception object and return an error signal according to API conventions.

This is another reason extension writing requires discipline.

Python's graceful exception model sits on top of careful C-level conventions.

---

# Tracebacks, Profilers, and Debuggers

CPython exposes enough runtime information for powerful tools.

Tracebacks use frame and code object information.

Debuggers inspect frames.

Profilers observe function calls and timing.

Coverage tools map executed bytecode back to source lines.

`sys.settrace()` and related mechanisms allow tracing execution.

This introspectability is part of Python's personality.

It makes Python pleasant for:

* debugging
* teaching
* profiling
* testing
* interactive exploration
* framework development

But introspection has costs.

The interpreter must preserve metadata and hooks that lower-level runtimes might avoid for speed.

CPython balances transparency and performance.

---

# CPython Is Not a JIT by Default

Traditional CPython is an interpreter.

It compiles source to bytecode, then interprets bytecode.

It does not normally compile hot Python functions into machine code the way a JIT-based runtime might.

This differs from implementations like PyPy, which use JIT techniques.

CPython has added adaptive interpreter optimizations, but that is not the same as saying CPython is generally a traditional JIT compiler.

This matters for performance expectations.

Pure Python loops can be slower than equivalent native code.

Libraries often move heavy work into native extensions.

Python remains productive because:

* high-level operations are expressive
* built-in types are implemented in C
* many libraries use native code internally
* most programs are not bottlenecked on pure Python loops
* integration with C extensions is strong

---

# Why NumPy Can Be Fast

NumPy is a useful example of CPython architecture in practice.

Python-level loop:

```python
result = []
for value in values:
    result.append(value * 2)
```

This runs many Python-level operations.

NumPy-style operation:

```python
array * 2
```

The Python expression dispatches to library code.

The heavy loop runs in optimized native code over contiguous data.

This reduces Python interpreter overhead.

The architecture lesson:

```text
Python coordinates
native extensions can compute
```

This is one reason CPython's C extension ecosystem is so important.

---

# Implementation Details vs Language Guarantees

Some behaviors are language guarantees.

Some are CPython implementation details.

Language-level ideas:

* objects have identity, type, and value
* names bind to objects
* exceptions propagate
* iterators use `__iter__` and `__next__`
* context managers use `__enter__` and `__exit__`
* dictionaries preserve insertion order in modern Python language semantics

CPython-specific or implementation-sensitive details:

* exact bytecode
* reference count timing
* internal object layouts
* memory allocator strategy
* GIL behavior
* specialization details
* C API internals

Professional Python code should rely on language guarantees where possible.

Use implementation details only when the project intentionally targets CPython.

---

# Reading CPython Source

Reading CPython source can be illuminating.

But it can also be overwhelming.

Approach it with questions.

Good questions:

* How is a list represented?
* Where does attribute lookup happen?
* How are frames evaluated?
* How is `dict` lookup implemented?
* How are reference counts updated?
* How does import initialize modules?

Do not try to read the repository linearly.

The source tree is large because CPython includes:

* parser
* compiler
* runtime
* object implementations
* standard library
* tests
* documentation
* build system
* platform support
* tools

Use the developer guide, source comments, and focused exploration.

The goal is not to become a CPython core developer immediately.

The goal is to understand enough architecture to reason correctly.

---

# Why This Matters for Python Engineers

Most Python developers do not need to write C extensions.

But CPython architecture still helps explain everyday behavior.

It explains why:

* local variables are efficient
* globals are looked up dynamically
* object finalization can be prompt in CPython
* cycles need garbage collection
* threads do not speed up pure Python CPU loops
* process pools avoid the GIL by using separate interpreters
* extension modules can be extremely fast
* bytecode changes across versions
* tracebacks contain frame information
* imports execute module code
* memory can grow due to caches, references, or cycles

Architecture turns scattered facts into one model.

That model makes debugging and design easier.

---

# Common Mistakes

Do not confuse Python the language with CPython the implementation.

Do not assume all Python implementations use reference counting.

Do not rely on prompt finalization unless you intentionally target CPython behavior.

Use context managers for resource cleanup.

Do not assume the GIL makes your application data race-free.

It does not protect your logical invariants.

Do not assume bytecode is stable across versions.

It is implementation detail.

Do not use `id()` as a business identity.

It reflects object identity during lifetime, not a permanent identifier.

Do not mix C allocation APIs with Python object allocation APIs in extensions.

Allocator mismatches can corrupt memory.

Do not assume extension modules are safer because they are faster.

Native code can crash the interpreter.

Do not treat CPython internals as public API unless the documentation says so.

---

# Mental Checklist

When thinking about CPython, ask:

* Is this behavior guaranteed by Python or specific to CPython?
* Am I reasoning about source, AST, bytecode, frames, or objects?
* Where does name lookup happen?
* Which namespace is involved?
* Is object lifetime controlled by references, cycles, or external resources?
* Does this code depend on prompt finalization?
* Is the GIL relevant to this workload?
* Is heavy work happening in Python bytecode or native code?
* Is an extension using stable API or CPython internals?
* Could this behavior change across Python versions?

The goal is not to fear implementation details.

The goal is to know when they matter.

---

# Exercises

1. Explain the difference between Python the language and CPython the implementation.

2. Draw the pipeline from source code to bytecode execution.

3. Explain why a syntax error happens before bytecode execution.

4. Describe the difference between a module code object and a function code object.

5. Explain what a frame contains during function execution.

6. Explain why local variable access can be optimized compared with global lookup.

7. Describe the conceptual relationship between `PyObject`, an object's type, and behavior.

8. Explain how reference counting and cyclic garbage collection complement each other.

9. Explain why a reference cycle can survive reference counting alone.

10. Describe why the GIL simplifies CPython internals but affects CPU-bound threading.

11. Explain why process pools can avoid the GIL limitation for CPU-bound work.

12. Explain why native extensions can be fast and risky at the same time.

13. Choose one built-in type and research its CPython implementation at a high level.

14. Explain why relying on exact bytecode is less portable than relying on language semantics.

15. Write a short paragraph explaining when CPython-specific knowledge is useful.

---

# Summary

CPython is the reference implementation of Python and the implementation most developers use.

Python the language and CPython the implementation are related but not identical.

CPython turns source code into tokens, parses tokens into syntax, compiles syntax into code objects, and executes bytecode inside frames.

Frames hold runtime execution state.

The evaluation loop executes bytecode instructions over frame state and Python objects.

Python objects have runtime bookkeeping and a link to their type.

Types describe behavior and operation support.

CPython uses reference counting as its primary lifetime mechanism.

Cyclic garbage collection handles unreachable reference cycles that reference counting alone cannot collect.

CPython manages Python objects in a private heap through specialized memory allocation systems.

The GIL protects interpreter-level execution in ordinary CPython builds but does not make application logic race-free.

Interpreter state and thread state help CPython manage execution across interpreters and threads.

The import system creates, executes, and caches module objects.

The C API lets native code interact with Python objects and the interpreter.

Native extensions can be powerful, fast, and dangerous.

CPython includes many optimizations, but it is not traditionally a general JIT compiler by default.

Some behaviors are language guarantees.

Others are CPython implementation details.

The deep lesson is:

```text
CPython is the runtime machine that turns Python's object model, bytecode, memory management, and extension system into execution
```

Understanding that machine makes advanced Python feel less mysterious.

---

# Preview of Chapter 71

Chapter 70 studied CPython architecture.

We saw how parsing, compilation, code objects, frames, object representation, memory management, reference counting, garbage collection, the GIL, imports, and the C API fit together.

Next we study C extensions.

C extensions are where Python crosses into native code.

They allow Python programs to:

* call C libraries
* define native-backed Python types
* move performance-critical loops out of Python bytecode
* integrate with operating-system APIs
* expose high-performance numerical or systems code

But they also introduce risks:

* segmentation faults
* reference leaks
* use-after-free bugs
* ABI compatibility issues
* platform-specific builds
* GIL management errors
* memory allocator mistakes

Chapter 71 will explain:

* what C extensions are
* why they exist
* how Python objects appear at the C boundary
* reference ownership
* extension module initialization
* native functions exposed to Python
* compiled wheels
* alternatives such as CFFI, ctypes, Cython, pybind11, and Rust bindings

The transition is:

```text
CPython architecture explains the runtime
C extensions explain how native code plugs into it
```

This will complete Volume II's journey from Python language features into Python's implementation boundaries.
