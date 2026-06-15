# Chapter 02 — Computer Architecture

---

# Learning Objectives

By the end of this chapter, you should understand:

* What a computer really is.
* How CPUs execute instructions.
* The difference between memory and storage.
* What registers are and why they exist.
* Why binary numbers are used.
* How programs are represented internally.
* How instructions flow through a computer.
* Why programming languages and operating systems exist.

---

# Concept Overview

Computers appear capable of performing incredibly sophisticated tasks:

* Playing videos
* Running games
* Hosting websites
* Training machine learning models
* Executing Python programs

However, underneath all of this complexity lies a surprisingly simple machine.

A computer repeatedly performs four actions:

1. Fetch an instruction.
2. Read some data.
3. Execute the instruction.
4. Store the result.

Everything else is built on top of this cycle.

---

# Mental Model

Think of a computer as a kitchen.

| Kitchen Component | Computer Component |
| ----------------- | ------------------ |
| Chef              | CPU                |
| Countertop        | Registers          |
| Pantry            | RAM                |
| Refrigerator      | Storage            |
| Recipe Book       | Program            |
| Ingredients       | Data               |

The chef cannot prepare food directly from the supermarket.

Ingredients must first be brought to the kitchen.

Similarly, CPUs operate on data brought into memory.

---

# What Is a Computer?

A computer is a machine that executes instructions on data.

Its major components are:

```text
                Computer
                    │
     ┌──────────────┼───────────────┐
     │              │               │
    CPU            RAM           Storage
     │
 Registers
```

---

# The CPU

CPU stands for:

> Central Processing Unit

The CPU is the "brain" of the computer.

Its job is to execute instructions.

Examples:

* Add numbers
* Compare values
* Move data
* Perform multiplication
* Jump to another instruction

The CPU does not understand:

* Python
* Java
* English

It only understands machine instructions.

---

# CPU Responsibilities

The CPU repeatedly performs:

```text
Fetch Instruction
       ↓
Decode Instruction
       ↓
Execute Instruction
       ↓
Store Result
       ↓
Repeat
```

This process is called the:

> Fetch-Decode-Execute Cycle

Everything you do on a computer is ultimately reduced to this cycle.

---

# Memory vs Storage

Many beginners confuse these two.

## Memory (RAM)

RAM stands for:

> Random Access Memory

Characteristics:

* Fast
* Temporary
* Volatile
* Stores running programs

Examples:

* Browser tabs
* Python variables
* Running applications

When power is lost, RAM is cleared.

---

## Storage

Storage refers to:

* SSD
* HDD

Characteristics:

* Persistent
* Slower than RAM
* Long-term

Examples:

* Photos
* Videos
* Python files
* Installed applications

---

## Relationship

```text
Storage
    ↓
RAM
    ↓
CPU
```

Programs must first move from storage into RAM before the CPU can execute them.

---

# Registers

Registers are tiny storage locations inside the CPU.

They are extremely fast.

Registers hold:

* Current values
* Intermediate results
* Memory addresses

Think of them as the CPU's desk.

```text
Storage
 ↓
RAM
 ↓
Registers
 ↓
CPU Operations
```

Because registers are very small, the CPU constantly moves data between RAM and registers.

---

# Why Registers Exist

Imagine solving a math problem.

You don't repeatedly walk to your bookshelf.

Instead, you keep numbers on a piece of paper.

Registers serve the same purpose.

Without registers, the CPU would constantly wait for memory.

---

# Binary Numbers

Computers use electrical signals.

Electrical circuits naturally represent two states:

```text
Off → 0
On  → 1
```

Therefore computers use:

> Binary

Base-2 number system.

Example:

Decimal:

```text
13
```

Binary:

```text
1101
```

Because:

```text
1×8 + 1×4 + 0×2 + 1×1 = 13
```

---

# Bits and Bytes

## Bit

Smallest unit of information.

Possible values:

```text
0
1
```

---

## Byte

8 bits.

Example:

```text
01000001
```

A byte can represent:

* Characters
* Numbers
* Colors
* Instructions

---

# Units of Memory

| Unit | Size       |
| ---- | ---------- |
| Byte | 8 bits     |
| KB   | 1024 bytes |
| MB   | 1024 KB    |
| GB   | 1024 MB    |
| TB   | 1024 GB    |

---

# Machine Instructions

CPUs execute machine instructions.

Examples:

```text
LOAD
ADD
STORE
COMPARE
JUMP
```

Suppose we want:

```python
x = 2 + 3
```

The CPU may perform:

```text
LOAD 2
LOAD 3
ADD
STORE RESULT
```

Python itself never reaches the CPU directly.

Eventually it becomes machine instructions.

---

# Programs Are Instructions Stored in Memory

A program is simply:

> Instructions stored in memory.

Example:

```python
print("Hello")
```

Internally:

```text
Program File
      ↓
Loaded into RAM
      ↓
Instructions Read
      ↓
Executed by CPU
```

---

# Cache Memory

RAM is fast.

But CPUs are much faster.

Therefore CPUs have another layer:

> Cache

Hierarchy:

```text
Registers
   ↓
L1 Cache
   ↓
L2 Cache
   ↓
L3 Cache
   ↓
RAM
   ↓
Storage
```

Closer means:

* Smaller
* Faster

Farther means:

* Larger
* Slower

---

# Why Cache Exists

Suppose you're reading a book.

Frequently used pages are kept open.

Less frequently used pages remain closed.

Cache acts similarly.

Frequently accessed data stays near the CPU.

This reduces waiting time.

---

# Memory Addresses

RAM consists of millions of locations.

Each location has an address.

```text
Address      Value

1000         A
1001         B
1002         C
1003         D
```

Variables eventually occupy memory addresses.

Later, when studying Python objects, memory addresses will become very important.

---

# Input Devices

Input devices provide information.

Examples:

* Keyboard
* Mouse
* Webcam
* Microphone

Input:

```text
User Action
      ↓
Computer
```

---

# Output Devices

Output devices present information.

Examples:

* Monitor
* Speaker
* Printer

```text
Computer
      ↓
Output Device
```

---

# The Complete Execution Flow

Suppose we execute:

```python
print(2 + 3)
```

High-level view:

```text
Source File
      ↓
Storage
      ↓
RAM
      ↓
CPU
      ↓
Monitor
```

Low-level view:

```text
Program File
      ↓
Load into Memory
      ↓
Fetch Instruction
      ↓
Decode Instruction
      ↓
Execute
      ↓
Store Result
      ↓
Display Output
```

---

# Why High-Level Languages Exist

Imagine programming using machine instructions:

```text
10110000
11001010
00001011
```

Humans are terrible at reasoning this way.

Programming languages allow us to think at higher levels.

```python
print(2 + 3)
```

is much easier to understand.

Python exists because abstraction improves productivity.

---

# Common Misconceptions

## Misconception 1

### RAM and Storage are the same.

Reality:

RAM is temporary.

Storage is permanent.

---

## Misconception 2

### CPUs execute Python directly.

Reality:

CPUs only understand machine instructions.

---

## Misconception 3

### More RAM makes CPUs faster.

Reality:

RAM capacity and CPU speed are different.

---

## Misconception 4

### Data lives inside variables.

Reality:

Data occupies memory locations.

Variables are abstractions provided by programming languages.

This will become important in later chapters.

---

# Real World Example

Suppose you open Spotify.

```text
SSD
 ↓
Spotify Program Loaded into RAM
 ↓
CPU Executes Instructions
 ↓
Audio Decoded
 ↓
Speaker Produces Sound
```

The same principle applies to:

* Chrome
* VS Code
* Python
* Games
* Databases

---

# Concept Connections

This chapter establishes:

```text
Computer
     ↓
CPU
     ↓
Memory
     ↓
Instructions
     ↓
Programs
     ↓
Processes
     ↓
Operating Systems
     ↓
Python Interpreter
```

Understanding hardware makes software behavior much less mysterious.

---

# Active Recall

## Easy Questions

1. What is a CPU?
2. What is RAM?
3. What is storage?
4. What is a register?
5. Why do computers use binary?

---

## Deep Understanding Questions

1. Why can't CPUs execute Python code directly?
2. Why are registers needed if RAM already exists?
3. Why does cache exist?
4. Why must programs be loaded into RAM before execution?

---

## Explain In Your Own Words

Explain:

* CPU
* RAM
* Storage
* Registers

using the kitchen analogy.

---

## Mental Model Questions

Complete:

```text
Storage
   ↓
?
   ↓
CPU
```

Why can't the CPU work directly from storage?

---

## Predict the Flow

What happens when you execute:

```python
print("Hello")
```

Describe the journey from:

* File on disk
* Memory
* CPU
* Screen

---

# Exercises

## Exercise 1

Classify the following:

* SSD
* RAM
* CPU
* Register
* Monitor

into:

* Storage
* Processing
* Input
* Output

---

## Exercise 2

Explain why cache memory improves performance.

---

## Exercise 3

Draw the hierarchy:

```text
Registers
Cache
RAM
Storage
```

and explain why speed decreases downward.

---

# Summary

In this chapter we learned:

* Computers execute instructions on data.
* CPUs perform the fetch-decode-execute cycle.
* RAM and storage serve different purposes.
* Registers provide extremely fast temporary storage.
* Binary represents electrical states.
* Cache reduces memory latency.
* Programs must be loaded into memory before execution.
* High-level languages exist to hide hardware complexity.

---

# Preview of Chapter 03

In the next chapter we will study:

* Programs vs Processes
* Process creation
* Threads
* Stack memory
* Heap memory
* Execution flow

These concepts will later explain how Python functions, variables, and objects work internally.
