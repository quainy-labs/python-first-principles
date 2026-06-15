# Chapter 04 — Operating Systems

---

# Learning Objectives

By the end of this chapter, you should understand:

* What an operating system is.
* Why operating systems exist.
* The responsibilities of an operating system.
* Processes and scheduling.
* Memory management.
* Files and file systems.
* System calls.
* File descriptors.
* Why programming languages depend heavily on the operating system.
* How Python interacts with the operating system.

---

# Concept Overview

Modern computers are incredibly complex.

Programs need access to:

* CPU
* Memory
* Disk
* Keyboard
* Mouse
* Network
* Monitor
* Speakers

If every application controlled hardware directly, chaos would occur.

Imagine:

* Chrome controlling the keyboard.
* Spotify controlling the speakers.
* VS Code controlling memory.

Programs would interfere with each other constantly.

The operating system exists to coordinate and manage all resources.

---

# Mental Model

Think of a hotel.

Resources:

* Rooms
* Elevators
* Electricity
* Water
* Staff

Guests cannot directly manage these resources.

Instead, they interact with hotel management.

```text
Guest
   ↓
Hotel Management
   ↓
Resources
```

Similarly:

```text
Application
     ↓
Operating System
     ↓
Hardware
```

The operating system acts as the manager of the computer.

---

# What Is an Operating System?

An operating system (OS) is software that manages hardware resources and provides services to programs.

Examples:

* Windows
* Linux
* macOS
* Android
* iOS

The operating system sits between applications and hardware.

```text
Applications
      ↑
Operating System
      ↑
Hardware
```

---

# Why Does an Operating System Exist?

Without an operating system:

Every application would need to understand:

* CPUs
* Memory
* Keyboards
* Monitors
* Disk drives
* Network cards

Writing software would become extremely difficult.

The operating system provides abstraction.

Programs ask the operating system for services instead of controlling hardware directly.

---

# Major Responsibilities

An operating system manages:

```text
Processes
Memory
Files
Devices
Networking
Security
Scheduling
```

---

# Process Management

Programs become processes when they run.

The OS is responsible for:

* Creating processes
* Terminating processes
* Allocating resources
* Switching between processes

Example:

```text
Chrome
Spotify
VS Code
Python
```

All run simultaneously because the OS coordinates them.

---

# CPU Scheduling

Only a few CPU cores exist.

Yet dozens of programs run simultaneously.

The OS rapidly switches between processes:

```text
Chrome
 ↓
Spotify
 ↓
VS Code
 ↓
Python
 ↓
Chrome
```

This process is called:

> Scheduling

It creates the illusion of parallel execution.

---

# Context Switching

When switching between processes, the OS saves:

* Registers
* Program counter
* Stack information

Then restores another process.

```text
Process A
    ↓
Save State
    ↓
Process B
    ↓
Save State
    ↓
Process C
```

This is called:

> Context Switching

---

# Memory Management

Programs need memory.

The operating system:

* Allocates memory
* Protects memory
* Frees memory

Example:

```text
+-------------------+
| Chrome            |
+-------------------+
| Spotify           |
+-------------------+
| Python            |
+-------------------+
| VS Code           |
+-------------------+
```

Processes cannot normally access each other's memory.

This isolation improves security and stability.

---

# Virtual Memory

Programs believe they own continuous memory:

```text
0x0000
0x0001
0x0002
...
```

But physical memory may be scattered.

The OS creates an abstraction called:

> Virtual Memory

```text
Program
    ↓
Virtual Memory
    ↓
Physical Memory
```

This abstraction simplifies program execution.

---

# File Systems

Storage devices contain billions of bytes.

Without organization, finding data would be impossible.

File systems organize storage.

Examples:

* NTFS
* ext4
* APFS

They provide:

* Files
* Directories
* Permissions

Example:

```text
Documents/
    notes.txt

Pictures/
    image.jpg

Python/
    app.py
```

---

# Files

Files are persistent containers for information.

Examples:

```text
data.csv
notes.txt
app.py
video.mp4
```

Programs read and write files through the operating system.

---

# Directories

Directories organize files.

Example:

```text
project/
│
├── app.py
├── config.py
├── data/
│     users.csv
│
└── logs/
      app.log
```

Directories create hierarchy.

---

# Devices

Hardware devices include:

* Keyboard
* Mouse
* Printer
* Network card
* Monitor

Programs do not communicate directly with devices.

Instead:

```text
Application
     ↓
Operating System
     ↓
Device Driver
     ↓
Hardware
```

Drivers translate software requests into hardware operations.

---

# System Calls

Applications cannot directly manipulate hardware.

Instead, they request services from the OS.

These requests are called:

> System Calls

Examples:

* Open file
* Read file
* Write file
* Create process
* Allocate memory

The flow:

```text
Application
      ↓
System Call
      ↓
Operating System
      ↓
Hardware
```

---

# Example: Opening a File

Python code:

```python
with open("hello.txt") as file:
    content = file.read()
```

Internally:

```text
Python
   ↓
CPython
   ↓
System Call
   ↓
Operating System
   ↓
Disk
```

Python itself cannot read disks.

The OS performs the actual operation.

---

# File Descriptors

Operating systems represent open files using integers.

Example:

```text
0 → stdin
1 → stdout
2 → stderr
```

Suppose:

```python
file = open("data.txt")
```

The OS might assign:

```text
3
```

Internally:

```text
3 → data.txt
```

Programs use these descriptors to interact with files.

---

# Standard Streams

Every process starts with three streams.

## stdin

Input stream.

Descriptor:

```text
0
```

Example:

Keyboard input.

---

## stdout

Output stream.

Descriptor:

```text
1
```

Example:

```python
print("Hello")
```

writes to stdout.

---

## stderr

Error stream.

Descriptor:

```text
2
```

Used for error messages.

---

# Networking

Operating systems manage network communication.

Applications use:

```text
Sockets
Ports
Protocols
```

Example:

Browser:

```text
Chrome
   ↓
Operating System
   ↓
Network Card
   ↓
Internet
```

Python libraries like:

```python
requests
socket
httpx
```

ultimately rely on the OS.

---

# Security

Operating systems enforce:

* User permissions
* Process isolation
* File permissions
* Authentication

Example:

One program cannot freely access another process's memory.

This prevents many security problems.

---

# Multitasking

Modern systems run many applications simultaneously.

Examples:

```text
Browser
Music Player
IDE
Python Script
Discord
```

The operating system coordinates all of them.

---

# Why Python Depends On The Operating System

Python itself cannot:

* Read files
* Access the network
* Create processes
* Allocate memory directly

Instead:

```text
Python Program
      ↓
CPython Interpreter
      ↓
Operating System
      ↓
Hardware
```

Many Python modules are thin wrappers around OS services.

Examples:

```python
os
pathlib
socket
subprocess
multiprocessing
threading
```

All eventually interact with the OS.

---

# Common Misconceptions

## Misconception 1

Python talks directly to hardware.

Reality:

```text
Python
 ↓
OS
 ↓
Hardware
```

---

## Misconception 2

Programs run independently.

Reality:

The OS manages all execution.

---

## Misconception 3

Files belong to Python.

Reality:

Files are operating system resources.

Python merely requests access.

---

## Misconception 4

Multiple applications run truly simultaneously.

Reality:

The OS performs scheduling and context switching.

---

# Real World Example

Consider:

```python
import requests

response = requests.get("https://example.com")
```

Internally:

```text
Python Code
      ↓
requests
      ↓
socket module
      ↓
System Calls
      ↓
Operating System
      ↓
Network Card
      ↓
Internet
```

The operating system performs the actual networking.

---

# Concept Connections

```text
Computer
    ↓
CPU + Memory
    ↓
Operating System
    ↓
Processes
    ↓
Files
    ↓
System Calls
    ↓
Python Interpreter
    ↓
Python Program
```

Understanding the OS removes much of the mystery behind Python libraries.

---

# Active Recall

## Easy Questions

1. What is an operating system?
2. Why does an operating system exist?
3. What is a system call?
4. What are file descriptors?
5. What is scheduling?

---

## Deep Understanding Questions

1. Why shouldn't applications control hardware directly?
2. Why does process isolation improve security?
3. Why do programming languages depend on the operating system?
4. Why are file descriptors integers?

---

## Explain In Your Own Words

Explain:

* Operating system
* System calls
* File descriptors

using a hotel analogy.

---

## Mental Model Questions

Complete:

```text
Application
      ↓
?
      ↓
Hardware
```

Why is the middle layer necessary?

---

## Predict The Flow

Suppose:

```python
with open("notes.txt") as file:
    print(file.read())
```

Describe the path from:

* Python
* CPython
* Operating system
* Disk

---

# Exercises

## Exercise 1

List five things managed by the operating system.

---

## Exercise 2

Explain why Python cannot read files without the operating system.

---

## Exercise 3

Draw the layers:

```text
Application
Operating System
Hardware
```

and explain the responsibility of each.

---

# Summary

In this chapter we learned:

* Operating systems manage hardware resources.
* Applications interact with hardware through the OS.
* Processes are scheduled by the OS.
* Memory and files are OS resources.
* System calls provide access to OS services.
* File descriptors represent open resources.
* Python relies heavily on the operating system.
* Many Python libraries are wrappers around OS functionality.

---

# Preview of Part II — Understanding Python

So far we have studied the world beneath Python:

```text
Hardware
     ↓
CPU + Memory
     ↓
Operating System
     ↓
Processes
```

Now we are ready to study Python itself.

In the next chapter, we begin with:

**Chapter 05 — History and Philosophy of Python**

We will answer:

* Why was Python created?
* What problems was it designed to solve?
* Who created Python?
* Why is readability important?
* What is the Zen of Python?
* Why does Python look the way it does?

Understanding Python's philosophy is essential because many design decisions in the language become obvious once you understand its goals.
