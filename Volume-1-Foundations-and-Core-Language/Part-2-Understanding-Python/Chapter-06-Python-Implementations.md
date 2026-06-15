# Chapter 06 — Python Implementations

---

# Learning Objectives

By the end of this chapter, you should understand:

* The difference between Python and CPython.
* What an interpreter is.
* Why multiple Python implementations exist.
* The architecture and tradeoffs of CPython.
* What PyPy, Jython, IronPython, and MicroPython are.
* Why different implementations optimize for different goals.
* Why language specifications and implementations are separate concepts.

---

# Concept Overview

Many developers use the terms:

* Python
* CPython

as if they mean the same thing.

They do not.

Python is a language.

CPython is an implementation of that language.

This distinction is extremely important because many different programs can implement the same language.

Understanding this separation removes a great deal of mystery about how Python works.

---

# Mental Model

Think about human languages.

English is a language.

Many people can speak English.

Different people may have:

* Different accents
* Different voices
* Different speaking speeds

But they are all speaking English.

Similarly:

```text
Python Language
        ↓
 ┌──────────────┬─────────────┬─────────────┬─────────────┐
 │              │             │             │
CPython       PyPy         Jython      MicroPython
```

All follow Python's rules.

But they implement them differently.

---

# What Is Python?

Python is a specification.

It defines:

* Syntax
* Semantics
* Keywords
* Language rules
* Standard behavior

For example:

```python
x = 10

if x > 5:
    print("Hello")
```

The language specifies what this code means.

It does not specify exactly how it should be executed.

That is the responsibility of an implementation.

---

# What Is an Implementation?

An implementation is software that understands Python code and executes it.

Responsibilities include:

* Parsing source code.
* Compiling bytecode.
* Managing memory.
* Creating objects.
* Executing instructions.

The implementation determines:

> How Python runs.

---

# What Is CPython?

CPython is the reference implementation of Python.

It is:

* Written primarily in C.
* Maintained by the Python Software Foundation.
* The most widely used implementation.

When most people install Python:

```text
python3
```

they are actually installing:

```text
CPython
```

---

# Why Is It Called CPython?

Because it is implemented in:

```text
C
```

Hence:

```text
C + Python
↓
CPython
```

---

# High-Level Architecture

```text
Python Source Code
         ↓
Tokenizer
         ↓
Parser
         ↓
AST
         ↓
Compiler
         ↓
Bytecode
         ↓
Python Virtual Machine
         ↓
Operating System
         ↓
Hardware
```

We will study these stages in later chapters.

---

# Why Is CPython Written In C?

C provides:

* High performance
* Direct memory management
* Operating system access
* Portability

Advantages:

* Mature ecosystem
* Stable
* Extensible

Tradeoff:

* More complex internals

---

# CPython Responsibilities

CPython handles:

## Parsing

Converts:

```python
x = 5
```

into internal structures.

---

## Compilation

Produces bytecode.

---

## Memory Management

Creates and destroys objects.

---

## Garbage Collection

Reclaims unused memory.

---

## Object Model

Everything in Python is an object.

CPython implements this model.

---

## Execution

Runs bytecode instructions.

---

# Alternative Implementations

Why would alternative implementations exist?

Different users have different priorities:

* Speed
* Memory usage
* Embedded systems
* Java integration
* .NET integration

No single implementation optimizes for everything.

---

# PyPy

PyPy focuses on:

> Performance

Unlike CPython, PyPy includes:

```text
JIT Compiler
```

(JIT = Just-In-Time Compiler)

Architecture:

```text
Python Code
      ↓
Bytecode
      ↓
JIT Compiler
      ↓
Machine Code
```

Advantages:

* Often faster than CPython.

Tradeoffs:

* Higher memory usage.
* Some compatibility issues.

---

# Why JIT Compilers Exist

Suppose a loop runs millions of times.

CPython repeatedly interprets instructions.

PyPy detects:

> This code is executed frequently.

and converts it into machine code.

This reduces overhead.

---

# Jython

Jython runs on:

```text
JVM (Java Virtual Machine)
```

Architecture:

```text
Python
   ↓
Java Bytecode
   ↓
JVM
```

Advantages:

* Access to Java libraries.

Tradeoffs:

* Lagging behind newer Python versions.

---

# IronPython

IronPython targets:

```text
.NET CLR
```

(Common Language Runtime)

Advantages:

* Integration with C# and .NET.

Architecture:

```text
Python
   ↓
CLR
```

---

# MicroPython

MicroPython is designed for:

* Embedded systems
* Microcontrollers
* IoT devices

Examples:

* ESP32
* Raspberry Pi Pico

Goals:

* Small memory footprint
* Low power consumption

Tradeoff:

Not every CPython feature is available.

---

# Stackless Python

Stackless Python focuses on:

* Massive concurrency
* Lightweight task scheduling

Useful for:

* Games
* Simulations

---

# GraalPython

Runs inside:

```text
GraalVM
```

Goals:

* Interoperability
* Performance

---

# Comparison

| Implementation   | Goal             |
| ---------------- | ---------------- |
| CPython          | General purpose  |
| PyPy             | Performance      |
| Jython           | Java ecosystem   |
| IronPython       | .NET ecosystem   |
| MicroPython      | Embedded systems |
| Stackless Python | Concurrency      |
| GraalPython      | Polyglot runtime |

---

# Why CPython Dominates

CPython offers:

* Stability
* Compatibility
* Huge ecosystem
* Excellent documentation
* Broad library support

Most third-party libraries target CPython first.

Examples:

* NumPy
* Pandas
* PyTorch

often depend heavily on CPython internals.

---

# Python Specification vs Implementation

Consider:

```python
print("Hello")
```

The language defines:

```text
What should happen.
```

The implementation decides:

```text
How it happens.
```

This separation is fundamental.

---

# C Extensions

One reason CPython is popular:

It allows C code to integrate directly.

Examples:

* NumPy
* Pandas
* SciPy
* PyTorch

Python code often delegates heavy computations to C.

This explains why Python can remain productive while still supporting high-performance libraries.

---

# Why Understanding Implementations Matters

Later, we will study:

* Reference counting
* Garbage collection
* Object model
* GIL
* Bytecode

Most of these concepts belong specifically to:

```text
CPython
```

not Python itself.

This distinction prevents many misconceptions.

---

# Common Misconceptions

## Misconception 1

Python and CPython are the same.

Reality:

Python is the language.

CPython is one implementation.

---

## Misconception 2

Python is interpreted, period.

Reality:

Different implementations use:

* Interpretation
* JIT compilation
* Virtual machines

---

## Misconception 3

All implementations behave identically internally.

Reality:

They share semantics but differ in execution.

---

## Misconception 4

Python is written in Python.

Reality:

CPython is primarily written in C.

---

# Real World Usage

Most developers use:

```text
CPython
```

Data Science:

```text
CPython + NumPy + Pandas
```

Embedded Devices:

```text
MicroPython
```

Java Integration:

```text
Jython
```

Performance-sensitive workloads:

```text
PyPy
```

---

# Concept Connections

```text
Programming Language
        ↓
Language Specification
        ↓
Implementation
        ↓
CPython
        ↓
Parser
Compiler
VM
Memory
Objects
```

Understanding implementations prepares us to study how Python actually executes code.

---

# Active Recall

## Easy Questions

1. What is Python?
2. What is CPython?
3. Why are they different?
4. Why does PyPy exist?
5. What is MicroPython used for?

---

## Deep Understanding Questions

1. Why separate language and implementation?
2. Why isn't there only one Python implementation?
3. Why is CPython written in C?
4. Why can PyPy outperform CPython?

---

## Explain In Your Own Words

Explain:

* Python
* CPython
* PyPy

using spoken languages as an analogy.

---

## Mental Model Questions

Complete:

```text
Python Language
       ↓
?
       ↓
Execution
```

Why is the middle layer necessary?

---

## Predict the Consequence

Suppose NumPy uses CPython internals heavily.

Would it automatically work on every Python implementation?

Why or why not?

---

# Exercises

## Exercise 1

Research:

* PyPy
* MicroPython

Identify one advantage and one tradeoff for each.

---

## Exercise 2

Explain why Python and CPython are not synonyms.

---

## Exercise 3

Draw:

```text
Python Language
     ↓
CPython
PyPy
MicroPython
```

and explain the relationship.

---

# Summary

In this chapter we learned:

* Python is a language specification.
* CPython is the reference implementation.
* Multiple implementations exist because different goals require different tradeoffs.
* PyPy focuses on speed.
* MicroPython targets embedded devices.
* CPython is written primarily in C.
* Many future chapters focus specifically on CPython internals.

---

# Preview of Chapter 07

So far we know:

```text
Python Code
      ↓
CPython
      ↓
Execution
```

But what exactly happens inside CPython?

In the next chapter, we will trace the complete journey:

```text
source.py
      ↓
Tokenizer
      ↓
Parser
      ↓
AST
      ↓
Compiler
      ↓
Bytecode
      ↓
Python Virtual Machine
      ↓
Output
```

For the first time, we will answer:

> What really happens when you run a Python file?
