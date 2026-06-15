# Chapter 07 — How Python Runs

---

# Learning Objectives

By the end of this chapter, you should understand:

* What happens when you run a Python file.
* Why Python source code is not executed directly as plain text.
* How CPython transforms source code into an executable internal form.
* What tokenization means.
* What parsing means.
* What an Abstract Syntax Tree is.
* Why Python has a compilation step even though people often call it interpreted.
* What bytecode is at a high level.
* What a code object is at a high level.
* Where syntax errors are detected.
* Why `__pycache__` directories appear.
* How this execution pipeline connects to the Python Virtual Machine.

This chapter answers one of the most important questions in Python:

> What really happens after I type `python program.py`?

---

# Concept Overview

A Python file is text.

For example:

```python
x = 10
y = 20
print(x + y)
```

To a human, this is easy to read.

To the computer, it is just characters stored in a file:

```text
x
space
=
space
1
0
newline
...
```

The CPU does not understand Python source code.

The operating system does not understand Python syntax.

Even CPython does not execute the raw text directly.

Instead, CPython transforms the program through several stages:

```text
program.py
    |
    v
source text
    |
    v
tokens
    |
    v
abstract syntax tree
    |
    v
code object
    |
    v
bytecode
    |
    v
Python Virtual Machine
    |
    v
runtime behavior
```

The purpose of this chapter is to make that pipeline clear.

This matters because many beginner explanations say:

> Python reads your code line by line and executes it.

That explanation is too vague.

It hides the real machinery.

Python does not simply read English-like commands and magically perform them. CPython performs a careful sequence of transformations before execution begins.

---

# Mental Model

Think of running a Python program like translating a book into stage directions.

The `.py` file is written for humans.

CPython must turn it into a form the interpreter can execute.

```text
Human-readable text
        |
        v
Structured program meaning
        |
        v
Low-level interpreter instructions
        |
        v
Runtime execution
```

Another useful mental model:

```text
Source code answers:
"What did the programmer write?"

Tokens answer:
"What are the meaningful pieces?"

The parser answers:
"Do these pieces form valid Python grammar?"

The AST answers:
"What does this program mean structurally?"

The compiler answers:
"What instructions should CPython execute?"

The Python Virtual Machine answers:
"How do those instructions change runtime state?"
```

This chapter focuses mainly on the stages up to bytecode.

Chapter 08 will go deeper into bytecode and the Python Virtual Machine.

---

# Why This Pipeline Exists

Python has a rich, human-friendly syntax.

Humans like writing code such as:

```python
if user.is_active:
    send_email(user)
```

But computers cannot execute that directly.

CPython needs a representation that is:

* Precise
* Structured
* Validated
* Easier to analyze
* Easier to execute
* Independent of formatting details where possible

Raw text is not enough.

For example, consider this code:

```python
total = price * quantity + tax
```

CPython must understand that:

* `total` is a name.
* `=` is assignment.
* `price`, `quantity`, and `tax` are names.
* `*` has higher precedence than `+`.
* `price * quantity` must be evaluated before adding `tax`.

That meaning is not obvious from characters alone.

The pipeline exists to move from text to meaning to executable instructions.

---

# Step 1: The Operating System Starts CPython

When you run:

```bash
python program.py
```

you are asking the operating system to start a process.

From Chapter 03, we know:

* A program is a passive file.
* A process is a running instance of a program.

In this case, the program being started is the Python executable.

The operating system creates a new process for CPython:

```text
Operating System
        |
        v
starts python executable
        |
        v
creates Python process
        |
        v
passes "program.py" as an argument
```

The file `program.py` is not itself a machine-code program.

The Python executable is the program that the operating system runs.

`program.py` is input to that executable.

This distinction is important.

When you run:

```bash
python program.py
```

the CPU is executing CPython's machine code.

CPython then reads and executes your Python program.

---

# Step 2: CPython Reads the Source File

Once the Python process starts, CPython opens the file you provided.

For example:

```bash
python program.py
```

CPython receives:

```text
program.py
```

It reads the contents of that file as source code.

At this point, the program is still text.

For example, the file may contain:

```python
message = "Hello"
print(message)
```

The file content is a sequence of characters:

```text
m e s s a g e   =   " H e l l o " newline
p r i n t ( m e s s a g e ) newline
```

Before CPython can execute anything, it must understand the structure of that text.

---

# Source Code Is Not Runtime Behavior

This is a subtle but important point.

Source code describes behavior.

It is not the behavior itself.

The line:

```python
print("Hello")
```

does not print anything merely by existing in a file.

It prints only after:

* CPython reads the file.
* CPython validates the syntax.
* CPython compiles the code.
* CPython executes the resulting instructions.

A Python file on disk is inert.

Execution begins only when a running Python process interprets it.

This connects directly to the distinction from Chapter 03:

```text
program.py on disk  -> passive source file
python process      -> active execution
```

---

# Step 3: Tokenization

The first major transformation is tokenization.

Tokenization means breaking source text into meaningful pieces called tokens.

For example:

```python
x = 10 + 20
```

The raw text contains characters:

```text
x   =   1 0   +   2 0
```

The tokenizer groups those characters into meaningful units:

```text
NAME        x
OP          =
NUMBER      10
OP          +
NUMBER      20
NEWLINE
```

Tokens are not yet full program meaning.

They are the vocabulary of the program.

Just as a sentence is made of words and punctuation, a Python program is made of tokens.

---

# Why Tokenization Exists

Without tokenization, CPython would have to reason about individual characters all the time.

That would be inefficient and messy.

For example, CPython should understand `total_price` as one name, not as many separate characters:

```text
t o t a l _ p r i c e
```

The tokenizer groups it as:

```text
NAME total_price
```

It also recognizes numbers:

```python
42
3.14
```

strings:

```python
"hello"
'world'
```

operators:

```python
+
-
*
/
==
```

keywords:

```python
if
else
for
while
def
class
return
```

and structural elements:

```python
(
)
:
,
NEWLINE
INDENT
DEDENT
```

Tokenization gives CPython a cleaner input for the next stage.

---

# Indentation Becomes Tokens

Python uses indentation to represent blocks.

For example:

```python
if logged_in:
    print("Welcome")
    print("Dashboard loaded")
print("Done")
```

Humans see indentation visually.

CPython must represent indentation structurally.

The tokenizer produces special indentation tokens:

```text
IF
NAME logged_in
:
NEWLINE
INDENT
NAME print
...
NEWLINE
NAME print
...
NEWLINE
DEDENT
NAME print
...
```

This is one reason Python indentation is not merely style.

Indentation is part of Python syntax.

It changes the structure of the program.

---

# Tokenization Example

Python exposes tokenization through the standard library module `tokenize`.

You do not need to master this module now, but it helps reveal what CPython is doing conceptually.

Example source:

```python
answer = 40 + 2
print(answer)
```

Conceptually, this becomes tokens like:

```text
NAME       answer
OP         =
NUMBER     40
OP         +
NUMBER     2
NEWLINE
NAME       print
OP         (
NAME       answer
OP         )
NEWLINE
```

Notice that CPython is no longer dealing with vague text.

It has meaningful pieces.

---

# Step 4: Parsing

After tokenization, CPython has a stream of tokens.

But tokens alone are not enough.

Consider these tokens:

```text
NAME x
OP =
NUMBER 10
OP +
NUMBER 20
```

CPython still needs to know whether they form a valid Python statement.

Parsing checks whether tokens follow Python's grammar.

Grammar means the rules that define valid program structure.

For example, this is valid Python:

```python
x = 10 + 20
```

This is not:

```python
= x 10 + 20
```

The same pieces are present, but the structure is invalid.

Parsing answers:

> Do these tokens form a valid Python program?

---

# Grammar

Every programming language has grammar rules.

In English, grammar tells us that this is reasonable:

```text
The cat sleeps.
```

and this is not:

```text
Sleeps the the.
```

Python grammar plays a similar role.

It defines valid forms such as:

```python
name = expression

if expression:
    block

def name(parameters):
    block
```

The parser uses grammar rules to organize tokens into a structured representation.

---

# Syntax Errors Happen Before Execution

Parsing explains why some errors happen before any code runs.

Consider:

```python
print("Before")

if True
    print("Inside")

print("After")
```

This program has a syntax error because the `if` statement is missing a colon.

The output is not:

```text
Before
```

followed by an error.

Instead, CPython reports a syntax error before executing the program.

Why?

Because CPython parses the source before executing it.

If parsing fails, execution never begins.

That is why a syntax error near the bottom of a file can prevent the first line from running.

---

# Step 5: The Abstract Syntax Tree

After parsing, CPython builds an Abstract Syntax Tree, usually called an AST.

An AST represents the structure and meaning of the program in tree form.

The word "abstract" means it leaves out unnecessary surface details.

For example:

```python
x = 10 + 20
```

The AST represents the important structure:

```text
Assignment
├── target: Name("x")
└── value: Add
    ├── left: Number(10)
    └── right: Number(20)
```

The AST is not concerned with every character from the original source.

It cares about program meaning.

---

# Why a Tree?

Programs are naturally nested.

For example:

```python
result = (price * quantity) + tax
```

This expression contains smaller expressions:

```text
result = (price * quantity) + tax
          ----------------
                |
          multiplication

         --------------------- + ---
                 |              |
        left expression        tax
```

A tree is a good structure for nested meaning.

The root represents the whole statement.

Branches represent subparts.

Leaves represent simple values or names.

---

# AST Example

Consider:

```python
total = price * quantity + tax
```

The AST must preserve operator precedence.

Multiplication happens before addition.

Conceptually:

```text
Assign
├── target: total
└── value: Add
    ├── left: Multiply
    │   ├── left: price
    │   └── right: quantity
    └── right: tax
```

This tree makes the meaning explicit.

The program is not interpreted merely from left to right as characters.

CPython understands the structure.

---

# Inspecting an AST

Python provides an `ast` module.

Example:

```python
import ast

tree = ast.parse("x = 10 + 20")
print(ast.dump(tree, indent=4))
```

The output is detailed, but the important idea is that Python represents the code as structured nodes.

You will see nodes such as:

```text
Module
Assign
Name
BinOp
Constant
Add
```

These nodes represent program meaning:

* `Module` means the whole file or code block.
* `Assign` means assignment.
* `Name` means a variable name.
* `BinOp` means a binary operation such as addition.
* `Constant` means a literal value such as `10`.
* `Add` means the addition operator.

You do not need to memorize AST node classes now.

The key idea is:

> CPython transforms source code into a structured representation before compiling it.

---

# Step 6: Compilation

After CPython has an AST, it compiles the AST.

This surprises many learners.

Python is often called an interpreted language.

That is true in the sense that CPython does not normally compile your program into a standalone native machine-code executable like C does.

But CPython still has a compilation step.

It compiles Python source into bytecode.

So the simplified statement:

> Python is interpreted.

is incomplete.

A more accurate statement is:

> CPython compiles Python source code to bytecode, then executes that bytecode with the Python Virtual Machine.

---

# What Compilation Means Here

Compilation does not always mean "turning code into CPU machine code."

Compilation means translating code from one representation to another.

In C, compilation usually means:

```text
C source code
    |
    v
native machine code
```

In CPython, compilation means:

```text
Python source code
    |
    v
Python bytecode
```

The target is different.

C compilers often target the CPU directly.

CPython targets its own virtual machine.

---

# Step 7: Code Objects

Before discussing bytecode more directly, we need a high-level idea of code objects.

A code object is CPython's compiled representation of a block of code.

A module has a code object.

A function has a code object.

For example:

```python
def greet(name):
    return "Hello, " + name
```

The function body is compiled into a code object.

That code object contains information CPython needs later, such as:

* Bytecode instructions
* Constants used by the code
* Names referenced by the code
* Variable names
* Line number information
* Metadata about arguments and local variables

The code object is not the same as a function object.

A function object is a runtime object that can be called.

A code object is the compiled instructions and metadata used by that function.

This distinction will matter more when we study functions.

For now, remember:

```text
source code -> AST -> code object -> bytecode execution
```

---

# Step 8: Bytecode

Bytecode is a set of low-level instructions for the Python Virtual Machine.

For example, the source code:

```python
x = 10
y = 20
print(x + y)
```

is compiled into bytecode instructions that roughly mean:

```text
load constant 10
store it under the name x
load constant 20
store it under the name y
load print
load x
load y
add them
call print
```

This is not exact bytecode syntax.

The exact instruction names can change between Python versions.

The point is that bytecode is more explicit than source code.

Source code is friendly for humans.

Bytecode is friendly for CPython's interpreter loop.

---

# Seeing Bytecode With dis

Python provides the `dis` module to inspect bytecode.

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

The exact output depends on the Python version.

But conceptually:

* `LOAD_FAST` loads a local variable.
* `BINARY_OP` performs a binary operation such as addition.
* `RETURN_VALUE` returns from the function.

Do not worry about memorizing bytecode instructions yet.

Chapter 08 will study bytecode more carefully.

For this chapter, the important idea is that CPython does not execute your original source text directly.

It executes compiled bytecode.

---

# Step 9: The Python Virtual Machine Begins Execution

Once bytecode exists, CPython can execute it.

The Python Virtual Machine, often abbreviated as PVM, is the part of CPython that executes Python bytecode.

At a high level:

```text
bytecode instruction
        |
        v
PVM reads instruction
        |
        v
PVM updates runtime state
        |
        v
next instruction
```

For example, bytecode may instruct the PVM to:

* Load a value.
* Store a name.
* Call a function.
* Jump to another instruction.
* Return a value.

This chapter stops at the handoff to the PVM.

Chapter 08 explains how bytecode execution works in more detail.

---

# Full Flow Example

Suppose we have a file named `hello.py`:

```python
name = "Ada"
print("Hello", name)
```

When we run:

```bash
python hello.py
```

the flow is:

```text
1. The operating system starts the Python executable.

2. CPython runs as a process.

3. CPython receives "hello.py" as input.

4. CPython reads the source text:
   name = "Ada"
   print("Hello", name)

5. The tokenizer converts characters into tokens:
   NAME, OP, STRING, NEWLINE, NAME, OP, STRING, OP, NAME, OP

6. The parser checks that the tokens form valid Python grammar.

7. CPython builds an AST representing assignment and a function call.

8. CPython compiles the AST into a code object.

9. The code object contains bytecode.

10. The Python Virtual Machine executes the bytecode.

11. The program prints:
    Hello Ada
```

This is what "running Python code" means in CPython.

---

# What Happens With Syntax Errors?

Now consider this file:

```python
print("Before")

if True
    print("Inside")

print("After")
```

The `if` statement is invalid because it is missing a colon.

When CPython runs this file, it does not execute `print("Before")`.

Instead, the pipeline fails during parsing:

```text
source text
    |
    v
tokens
    |
    v
parser detects invalid grammar
    |
    v
SyntaxError
```

This teaches an important rule:

> Syntax errors prevent execution from starting.

Runtime errors are different.

Consider:

```python
print("Before")
print(10 / 0)
print("After")
```

This code is valid syntax.

CPython can tokenize, parse, compile, and start executing it.

The error happens during execution when division by zero occurs.

The output is:

```text
Before
```

followed by a runtime error.

So:

```text
SyntaxError     -> detected before execution
ZeroDivisionError -> detected during execution
```

This distinction becomes very important when debugging.

---

# What About Imports?

Running a file is not the only time this pipeline happens.

Imports also trigger compilation and execution.

Suppose you have:

```python
import helpers
```

When CPython imports `helpers`, it must find and load the `helpers` module.

If `helpers.py` has not already been loaded, CPython will:

* Locate the file.
* Read its source code.
* Tokenize it.
* Parse it.
* Compile it.
* Execute its top-level code.
* Store the module object so future imports can reuse it.

This explains why top-level code in imported modules runs during import.

Example:

```python
# helpers.py
print("helpers loaded")

def add(a, b):
    return a + b
```

```python
# main.py
import helpers
print(helpers.add(2, 3))
```

When `main.py` imports `helpers`, the top-level print in `helpers.py` runs.

Output:

```text
helpers loaded
5
```

The function body of `add` does not run during import.

But the function definition is executed in the sense that Python creates a function object and binds it to the name `add`.

This will become clearer when we study functions and modules later.

---

# What Is __pycache__?

Sometimes you will see a directory named `__pycache__`.

Inside it, you may find files ending in `.pyc`.

These are cached bytecode files.

When CPython imports a module, it may save the compiled bytecode to disk so that future imports can skip some compilation work.

For example:

```text
project/
├── main.py
├── helpers.py
└── __pycache__/
    └── helpers.cpython-312.pyc
```

The exact filename depends on the Python implementation and version.

Important points:

* `.py` files contain source code.
* `.pyc` files contain cached bytecode.
* `__pycache__` is an optimization.
* Python can recreate cached bytecode when needed.
* You normally do not edit `.pyc` files.

The cache does not mean your program has become a native executable.

It is still Python bytecode for the Python Virtual Machine.

---

# Does Python Run Line By Line?

This question needs a careful answer.

In one sense, execution proceeds through bytecode instructions in order, with jumps for loops, conditions, and function calls.

But saying:

> Python runs line by line.

is misleading.

Before execution begins, CPython has already processed the source file.

It has tokenized, parsed, and compiled the code.

Also, runtime control flow is not simply source-line order.

Consider:

```python
def greet():
    print("Hello")

print("Before")
greet()
print("After")
```

The function body appears before `print("Before")`, but it does not run when Python first sees the `def` statement.

The `def` statement creates a function object.

The body runs only when `greet()` is called.

So a better mental model is:

```text
CPython first prepares the code.
Then the Python Virtual Machine executes bytecode according to program control flow.
```

---

# Expressions and Statements in the Pipeline

Python source code is made of statements and expressions.

An expression produces a value.

Examples:

```python
10
x + y
"hello"
len(name)
```

A statement performs an action or controls execution.

Examples:

```python
x = 10
if x > 5:
    print(x)
def greet():
    print("Hello")
```

The parser and AST preserve this distinction.

For example:

```python
x = 10 + 20
```

The whole line is an assignment statement.

Inside it, `10 + 20` is an expression.

Conceptually:

```text
Assign statement
├── target: x
└── value expression: 10 + 20
```

This matters because Python has rules about where expressions and statements may appear.

For example:

```python
y = if x > 0:
```

is invalid because `if x > 0:` is a statement form, not an expression that can be assigned as a value.

Python does have conditional expressions:

```python
y = "positive" if x > 0 else "not positive"
```

But that is a different grammar form.

The parser is responsible for enforcing these rules.

---

# Names Are Not Resolved During Parsing

Parsing checks syntax.

It does not prove that every name exists at runtime.

Consider:

```python
print(username)
```

This is valid syntax.

CPython can tokenize it, parse it, compile it, and start execution.

If `username` has not been defined, the error happens at runtime:

```text
NameError
```

So:

```text
Invalid grammar       -> SyntaxError before execution
Missing runtime name  -> NameError during execution
```

This is another reason the pipeline matters.

Different stages catch different kinds of problems.

---

# Example: Syntax Error vs Name Error

Program A:

```python
print("start")
if True
    print("inside")
print("end")
```

This fails before execution.

Reason:

```text
The parser cannot form a valid if statement.
```

Program B:

```python
print("start")
print(missing_name)
print("end")
```

This starts executing.

It prints:

```text
start
```

Then it fails with `NameError`.

Reason:

```text
The syntax is valid, but the name is not found during execution.
```

That difference is not random.

It comes directly from the stages of Python execution.

---

# How This Connects to Previous Chapters

Chapter 01 taught that software is instructions and data.

A Python file is instructions written in a human-readable language.

Chapter 02 taught that CPUs execute machine instructions, not Python syntax.

That explains why CPython must exist between your `.py` file and the hardware.

Chapter 03 taught that running software means creating a process.

When you type `python program.py`, the operating system starts a Python process.

Chapter 04 taught that programs depend on the operating system for files, input, output, and process management.

CPython uses the operating system to open your `.py` file, read it, and interact with standard output.

Chapter 05 taught Python's philosophy.

Python source code is designed for readability, but readable source still needs transformation before execution.

Chapter 06 taught the difference between Python the language and CPython the implementation.

This chapter shows what CPython does with Python language source code.

---

# Internal Mechanics Summary

Here is the complete flow again:

```text
python program.py
        |
        v
OS starts CPython process
        |
        v
CPython reads program.py
        |
        v
source text
        |
        v
tokenization
        |
        v
tokens
        |
        v
parsing
        |
        v
AST
        |
        v
compilation
        |
        v
code object containing bytecode
        |
        v
Python Virtual Machine executes bytecode
        |
        v
program behavior
```

Each stage solves a different problem:

| Stage | Problem Solved |
| --- | --- |
| Read source | Get the program text from disk |
| Tokenize | Break text into meaningful pieces |
| Parse | Check grammar and build structure |
| Build AST | Represent program meaning |
| Compile | Translate meaning into bytecode |
| Execute | Run bytecode and produce behavior |

---

# Common Misconceptions

## Misconception 1

### Python directly executes the `.py` file.

The `.py` file is source text.

CPython reads it, parses it, compiles it, and executes bytecode produced from it.

The source file itself is not executed by the CPU.

---

## Misconception 2

### Python has no compilation step.

CPython does compile Python source code.

It compiles to bytecode, not usually to native machine code.

This is why saying "Python is interpreted" is only a simplification.

---

## Misconception 3

### Python always runs line by line from top to bottom.

Python prepares the code before execution.

During execution, control flow determines what runs.

Functions, loops, conditionals, exceptions, imports, and returns all affect execution order.

---

## Misconception 4

### Bytecode is the same as machine code.

Bytecode is for the Python Virtual Machine.

Machine code is for the CPU.

The CPU runs CPython's machine code. CPython runs your program's bytecode.

---

## Misconception 5

### `__pycache__` is required for Python programs to work.

`__pycache__` is a cache.

It can speed up imports by reusing compiled bytecode.

If it is deleted, Python can usually recreate it.

---

## Misconception 6

### Syntax errors happen while the program is running.

Syntax errors are detected before execution begins.

Runtime errors happen while bytecode is executing.

---

# Real-world Usage

Understanding how Python runs helps in several practical situations.

## Debugging Syntax Errors

When you see `SyntaxError`, you know CPython failed before runtime execution.

That means you should inspect grammar, punctuation, indentation, quotes, parentheses, or statement structure.

Example:

```python
if ready
    print("go")
```

The issue is not logic.

The program did not reach logic.

The parser could not build a valid structure.

---

## Debugging Runtime Errors

When you see `NameError`, `TypeError`, `ZeroDivisionError`, or similar runtime errors, the code was syntactically valid.

CPython successfully compiled it.

The failure happened while executing bytecode.

This tells you to inspect runtime state:

* Which names exist?
* What values do they reference?
* What types are those values?
* Which branch or function call was executing?

---

## Understanding Imports

Imports are not simple text inclusion.

When Python imports a module, it loads and executes that module's top-level code.

This explains why import-time side effects can happen.

Example:

```python
# config.py
print("Loading config")
```

```python
# app.py
import config
```

Running `app.py` prints:

```text
Loading config
```

because importing `config` executes the top-level code in `config.py`.

---

## Understanding Tooling

Many tools work with stages of this pipeline.

Formatters inspect and rewrite source code.

Linters analyze source code or ASTs.

Type checkers analyze program structure and annotations.

Coverage tools connect executed bytecode back to source lines.

Debuggers use code objects, frames, and line number information.

Profilers observe runtime execution.

Once you understand the pipeline, these tools feel less mysterious.

---

## Understanding Performance

Python startup and import time can involve reading files, parsing, compiling, and loading modules.

For small scripts, this overhead usually does not matter.

For large applications with many imports, startup time can become noticeable.

Bytecode caching helps reduce repeated compilation work for imported modules.

---

# Concept Connections

This chapter connects several mental models:

```text
Chapter 01:
Software is instructions and data.

Chapter 02:
The CPU does not understand high-level language syntax.

Chapter 03:
Execution happens inside a process.

Chapter 04:
The OS provides file access and process management.

Chapter 05:
Python source prioritizes human readability.

Chapter 06:
CPython is the implementation that runs Python source.

Chapter 07:
CPython transforms source into bytecode.

Chapter 08:
The Python Virtual Machine executes that bytecode.
```

The larger picture is:

```text
Human idea
    |
    v
Python source code
    |
    v
CPython compilation pipeline
    |
    v
bytecode
    |
    v
Python Virtual Machine
    |
    v
runtime behavior
```

---

# Active Recall

## Easy Recall Questions

1. What command is commonly used to run a Python file?
2. When you run `python program.py`, which program does the operating system start?
3. What is source code?
4. What is tokenization?
5. What is parsing?
6. What is an AST?
7. What does CPython compile Python source code into?
8. What is bytecode for?
9. What is `__pycache__` used for?
10. Does the CPU directly execute Python bytecode?

---

## Deep Understanding Questions

1. Why is it inaccurate to say Python simply executes source code line by line?
2. Why does CPython need both tokenization and parsing?
3. Why can a syntax error prevent the first line of a file from running?
4. Why is Python still said to be interpreted even though CPython has a compilation step?
5. Why is bytecode different from machine code?
6. Why does importing a module execute its top-level code?
7. Why can `NameError` happen only during execution, while `SyntaxError` happens before execution?
8. Why does indentation need to become part of the token stream?

---

## Explain In Your Own Words

1. Explain what happens from `python program.py` to bytecode execution.
2. Explain tokenization using an example.
3. Explain parsing using an example.
4. Explain what an AST represents.
5. Explain why `__pycache__` exists.
6. Explain the difference between syntax errors and runtime errors.

---

## Predict-the-Output Questions

### Question 1

```python
print("A")

if True
    print("B")

print("C")
```

What happens?

Answer:

The program raises `SyntaxError` before execution begins. It does not print `A`.

Reason:

The parser cannot build a valid `if` statement because the colon is missing.

---

### Question 2

```python
print("A")
print(missing_name)
print("C")
```

What happens?

Answer:

It prints:

```text
A
```

Then it raises `NameError`.

Reason:

The program is syntactically valid, so execution starts. The missing name is discovered during runtime execution.

---

### Question 3

```python
def show():
    print("inside")

print("before")
print("after")
```

What is printed?

Answer:

```text
before
after
```

Reason:

The `def` statement creates a function object. The function body does not run until the function is called.

---

### Question 4

```python
def show():
    print("inside")

print("before")
show()
print("after")
```

What is printed?

Answer:

```text
before
inside
after
```

Reason:

The function body runs when `show()` is called.

---

## Mental Model Questions

1. Draw the full pipeline from `.py` file to runtime behavior.
2. Draw the difference between source code and bytecode.
3. Draw where `SyntaxError` happens in the pipeline.
4. Draw where `NameError` happens in the pipeline.
5. Draw what happens when one module imports another module.

---

# Practical Exercises

## Exercise 1

Create a file named `example.py`:

```python
x = 10
y = 20
print(x + y)
```

Run it:

```bash
python example.py
```

Then explain each stage:

```text
source text
tokens
parser
AST
bytecode
execution
```

You do not need to list every real token or bytecode instruction yet. Focus on the purpose of each stage.

---

## Exercise 2

Create a syntax error:

```python
print("before")

if True
    print("inside")

print("after")
```

Predict whether `before` prints.

Then run the file and explain why the result happens.

---

## Exercise 3

Create a runtime error:

```python
print("before")
print(10 / 0)
print("after")
```

Predict what prints before the error.

Then run the file and explain why this differs from the syntax error example.

---

## Exercise 4

Use the `ast` module:

```python
import ast

tree = ast.parse("total = price * quantity + tax")
print(ast.dump(tree, indent=4))
```

Find the nodes that represent:

* Assignment
* Multiplication
* Addition
* Names

Explain how the AST preserves the meaning of the expression.

---

## Exercise 5

Use the `dis` module:

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

Do not memorize the output.

Identify instructions that appear to:

* Load values
* Perform an operation
* Return a value

Explain why bytecode is more explicit than source code.

---

## Exercise 6

Create two files:

```python
# helper.py
print("helper imported")

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

Explain why `helper imported` prints before `10`.

---

# Summary

In this chapter we learned:

* A Python file is source text.
* The operating system starts the Python executable, not the `.py` file directly.
* CPython reads the source file.
* Tokenization breaks source text into meaningful tokens.
* Parsing checks whether those tokens follow Python grammar.
* Syntax errors happen before execution begins.
* CPython builds an Abstract Syntax Tree to represent program meaning.
* CPython compiles the AST into code objects containing bytecode.
* Bytecode is executed by the Python Virtual Machine.
* Bytecode is not the same as CPU machine code.
* Runtime errors happen during bytecode execution.
* Imports also involve loading, compiling, and executing module code.
* `__pycache__` stores cached bytecode for imported modules.

The key mental model is:

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

You now understand what happens between writing Python code and seeing the program run.

---

# Preview of Chapter 08

We have reached bytecode.

Next we will study what bytecode looks like, how the Python Virtual Machine executes it, and why this execution model explains many Python behaviors.
