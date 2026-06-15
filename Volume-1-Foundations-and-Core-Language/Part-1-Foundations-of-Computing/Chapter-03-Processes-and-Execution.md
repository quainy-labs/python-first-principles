# Chapter 03 — Processes and Execution

---

# Learning Objectives

By the end of this chapter, you should understand:

* The difference between a program and a process.
* What happens when a program starts running.
* How the operating system creates processes.
* What threads are.
* What stack and heap memory are.
* Why multiple processes can run simultaneously.
* How execution flow works.
* Why these concepts matter for Python.

---

# Concept Overview

In the previous chapter, we learned that programs are stored on disk and must be loaded into memory before they can execute.

However, a program sitting on disk is passive.

Execution begins only when the operating system creates a running instance of that program.

That running instance is called a process.

Understanding processes and memory layout is fundamental because Python programs themselves run inside processes.

---

# Mental Model

Think of a recipe book.

The recipe itself is not cooking.

Cooking begins when someone starts following the recipe.

```text
Recipe Book
      ↓
Cook Starts Working
      ↓
Active Cooking Session
```

Similarly:

```text
Program File
      ↓
Operating System Starts It
      ↓
Running Process
```

A program is passive.

A process is active.

---

# What Is a Program?

A program is a collection of instructions stored on disk.

Examples:

```text
chrome.exe
python.exe
spotify.exe
calculator.exe
```

Programs do not consume CPU resources while sitting on storage.

They simply exist as files.

---

# What Is a Process?

A process is a running instance of a program.

Examples:

```text
Program
-------
chrome.exe

Process
--------
Chrome running in memory
```

Multiple processes can originate from the same program.

Example:

```text
chrome.exe

        ↓
+-----------------+
| Chrome Process 1|
+-----------------+

+-----------------+
| Chrome Process 2|
+-----------------+

+-----------------+
| Chrome Process 3|
+-----------------+
```

Each process has:

* Its own memory
* Its own stack
* Its own heap
* Its own execution state

---

# Program vs Process

| Program                 | Process                  |
| ----------------------- | ------------------------ |
| Passive                 | Active                   |
| Stored on disk          | Lives in memory          |
| Instructions only       | Instructions + state     |
| Static                  | Dynamic                  |
| Can have many processes | Represents one execution |

---

# Process Creation

Suppose you run:

```bash
python hello.py
```

The operating system performs:

```text
Program File
       ↓
Load into RAM
       ↓
Allocate Memory
       ↓
Create Process
       ↓
Assign Process ID
       ↓
Start Execution
```

The process begins executing instructions.

---

# Process ID (PID)

Every process receives a unique identifier.

Example:

```text
PID 1023 → Chrome

PID 1840 → Spotify

PID 2251 → Python
```

The operating system uses PIDs to manage processes.

---

# Process Memory Layout

A process contains multiple regions of memory.

```text
+------------------+
|     Stack         |
+------------------+
|                  |
|       Heap        |
|                  |
+------------------+
| Global Data       |
+------------------+
| Program Code      |
+------------------+
```

Each region has a different purpose.

---

# Program Code Section

Contains instructions.

Example:

```python
print("Hello")
```

The machine instructions corresponding to this code are stored here.

This section is generally read-only.

---

# Global Data Section

Stores:

* Global variables
* Constants

Example:

```python
PI = 3.14
```

Later, Python objects complicate this picture because everything becomes heap allocated.

---

# Stack Memory

The stack manages function execution.

Think of it as a stack of plates.

```text
Top
------
Function C
------
Function B
------
Function A
------
Bottom
```

Whenever a function is called:

* A stack frame is created.

When the function returns:

* The frame is removed.

---

# Example

```python
def a():
    b()

def b():
    c()

def c():
    pass

a()
```

Execution:

```text
Call a()

Stack:
-------
a
-------

Call b()

-------
b
-------
a
-------

Call c()

-------
c
-------
b
-------
a
-------
```

Returning removes frames:

```text
c finishes

-------
b
-------
a
-------
```

Eventually:

```text
Empty Stack
```

---

# Heap Memory

Heap memory stores dynamically allocated objects.

Examples:

* Lists
* Dictionaries
* Strings
* Objects

Unlike the stack, heap objects may survive after function calls.

---

# Stack vs Heap

| Stack             | Heap               |
| ----------------- | ------------------ |
| Function calls    | Objects            |
| Automatic cleanup | Managed by runtime |
| Fast              | Slower             |
| LIFO structure    | Dynamic            |
| Limited size      | Large              |

---

# Execution Flow

Consider:

```python
print("A")
print("B")
print("C")
```

Execution proceeds sequentially:

```text
Instruction 1
      ↓
Instruction 2
      ↓
Instruction 3
```

The CPU maintains an instruction pointer that indicates which instruction to execute next.

---

# Branching

Example:

```python
age = 20

if age >= 18:
    print("Adult")
```

Flow:

```text
Start
  ↓
Condition
  ↓
True?
 / \
Yes No
 |   |
Print Skip
```

Programs do not always execute linearly.

---

# Loops

Example:

```python
for i in range(3):
    print(i)
```

Execution:

```text
Start
 ↓
Condition
 ↓
Body
 ↓
Repeat
```

Loops repeatedly execute instructions.

---

# Threads

A process can contain multiple threads.

Thread:

> A sequence of execution inside a process.

Example:

Chrome process:

```text
Chrome Process
│
├── Thread 1
├── Thread 2
├── Thread 3
└── Thread 4
```

Threads share:

* Heap memory
* Global variables

But each thread has its own stack.

---

# Why Threads Exist

Suppose a browser:

* Downloads data
* Plays video
* Responds to clicks

Using one thread, these tasks would block each other.

Threads allow multiple activities simultaneously.

---

# Multiprocessing

Processes are isolated.

Example:

```text
Spotify Process

Heap A
Stack A

Chrome Process

Heap B
Stack B
```

Processes cannot directly access each other's memory.

Isolation improves stability.

---

# Context Switching

CPUs execute only a few instructions at a time.

The operating system rapidly switches between processes.

```text
Process A
 ↓
Process B
 ↓
Process C
 ↓
Process A
```

This creates the illusion that everything runs simultaneously.

---

# Process Lifecycle

```text
Created
    ↓
Ready
    ↓
Running
    ↓
Waiting
    ↓
Running
    ↓
Terminated
```

The operating system manages this lifecycle.

---

# Real Example

Suppose you run:

```bash
python app.py
```

The sequence becomes:

```text
app.py
    ↓
python.exe
    ↓
OS creates process
    ↓
Memory allocated
    ↓
Stack created
    ↓
Heap created
    ↓
CPU begins execution
```

Everything inside Python happens inside this process.

---

# Why This Matters For Python

Later we will learn:

* Function calls use stack frames.
* Objects live on the heap.
* Threads affect concurrency.
* Processes affect multiprocessing.
* Garbage collection operates on heap objects.

Without understanding processes, Python internals become mysterious.

---

# Common Misconceptions

## Misconception 1

Program and process are the same.

Reality:

Program:

```text
Passive file
```

Process:

```text
Active execution
```

---

## Misconception 2

Stack and heap are physical locations.

Reality:

They are logical regions within process memory.

---

## Misconception 3

Threads are independent processes.

Reality:

Threads share memory inside one process.

---

## Misconception 4

Multiple cores are required for multiple processes.

Reality:

One CPU can rapidly switch between processes.

---

# Real World Usage

VS Code:

```text
Main Process
    ↓
Extensions
    ↓
Renderer Process
    ↓
Terminal Process
```

Chrome:

```text
Browser Process
    ↓
Tab Processes
    ↓
GPU Process
```

Python Web Server:

```text
Master Process
    ↓
Worker Processes
    ↓
Threads
```

---

# Concept Connections

```text
Program
    ↓
Process
    ↓
Memory Layout
    ↓
Stack
Heap
    ↓
Functions
Objects
    ↓
Threads
Concurrency
```

These concepts will become essential when studying Python variables and objects.

---

# Active Recall

## Easy Questions

1. What is a process?
2. What is the difference between a program and a process?
3. What is a thread?
4. What is stack memory?
5. What is heap memory?

---

## Deep Understanding Questions

1. Why can one program create multiple processes?
2. Why does each thread need its own stack?
3. Why are processes isolated?
4. Why is heap memory necessary?

---

## Explain In Your Own Words

Explain:

* Program
* Process
* Thread
* Stack
* Heap

using a restaurant analogy.

---

## Mental Model Questions

Complete:

```text
Program
    ↓
?
    ↓
Stack + Heap
```

---

## Predict the Execution Stack

```python
def x():
    y()

def y():
    z()

def z():
    pass

x()
```

What does the stack look like when `z()` is executing?

---

# Exercises

## Exercise 1

Draw process memory:

```text
Code
Globals
Heap
Stack
```

and explain the purpose of each region.

---

## Exercise 2

Explain why multiple Chrome tabs can crash independently.

---

## Exercise 3

Draw the stack for:

```python
def a():
    b()

def b():
    c()

def c():
    pass

a()
```

while `c()` is running.

---

# Summary

In this chapter we learned:

* Programs are passive files.
* Processes are active executions.
* Processes have their own memory.
* Stack memory manages function calls.
* Heap memory stores dynamically allocated objects.
* Threads are execution paths inside processes.
* Context switching creates concurrency.
* These concepts form the foundation for understanding Python functions and objects.

---

# Preview of Chapter 04

In the next chapter, we will study Operating Systems:

* What an operating system really does
* Files and file systems
* System calls
* Scheduling
* Memory management
* File descriptors
* Why Python depends heavily on the OS

Understanding the operating system will complete our foundations of computing and prepare us to finally begin studying Python itself.
