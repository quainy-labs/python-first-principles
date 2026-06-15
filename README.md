# Python: From First Principles to Professional Engineering

> A complete first-principles journey into Python, CPython internals, software engineering, and the Python ecosystem.

## Philosophy

This project aims to be the definitive Python curriculum.

The objective is not merely learning syntax.

The objective is mastery.

Every concept answers:

* What is it?
* Why does it exist?
* What problem does it solve?
* How does it work?
* How is it implemented?
* What are the tradeoffs?
* How does it connect to previous concepts?

The curriculum follows strict dependency order and avoids unexplained magic.

---

# Target Audience

* Beginners seeking deep understanding.
* Intermediate developers wanting solid foundations.
* Professional engineers transitioning into Python.
* Interview preparation.
* Open-source contributors.
* Engineers interested in CPython internals.

---

# Book Structure

## Volume I — Foundations and Core Language

Volume I builds the complete mental model required before advanced Python feels natural. It starts below Python, moves through execution, objects, references, primitive types, control flow, functions, data structures, memory behavior, and finally modules/imports.

### Part I — Foundations of Computing

1. [What Is Software?](Volume-1-Foundations-and-Core-Language/Part-1-Foundations-of-Computing/Chapter-01-What-Is-Software.md)
2. [Computer Architecture](Volume-1-Foundations-and-Core-Language/Part-1-Foundations-of-Computing/Chapter-02-Computer-Architecture.md)
3. [Processes and Execution](Volume-1-Foundations-and-Core-Language/Part-1-Foundations-of-Computing/Chapter-03-Processes-and-Execution.md)
4. [Operating Systems](Volume-1-Foundations-and-Core-Language/Part-1-Foundations-of-Computing/Chapter-04-Operating-Systems.md)

### Part II — Understanding Python

5. [History and Philosophy of Python](Volume-1-Foundations-and-Core-Language/Part-2-Understanding-Python/Chapter-05-History-of-Python.md)
6. [Python Implementations](Volume-1-Foundations-and-Core-Language/Part-2-Understanding-Python/Chapter-06-Python-Implementations.md)
7. [How Python Runs: Source Code to Bytecode](Volume-1-Foundations-and-Core-Language/Part-2-Understanding-Python/Chapter-07-How-Python-Runs.md)
8. [Bytecode and the Python Virtual Machine](Volume-1-Foundations-and-Core-Language/Part-2-Understanding-Python/Chapter-08-Bytecode-and-the-Python-Virtual-Machine.md)

### Part III — Objects and References

9. [Everything Is an Object](Volume-1-Foundations-and-Core-Language/Part-3-Objects-and-References/Chapter-09-Everything-Is-an-Object.md)
10. [Names and References](Volume-1-Foundations-and-Core-Language/Part-3-Objects-and-References/Chapter-10-Names-and-References.md)
11. [Mutability](Volume-1-Foundations-and-Core-Language/Part-3-Objects-and-References/Chapter-11-Mutability.md)
12. [Identity, Equality, and Memory Diagrams](Volume-1-Foundations-and-Core-Language/Part-3-Objects-and-References/Chapter-12-Identity-and-Equality.md)

### Part IV — Primitive Types and Expressions

13. [Numbers](Volume-1-Foundations-and-Core-Language/Part-4-Primitive-Types-and-Expressions/Chapter-13-Numbers.md)
14. [Strings](Volume-1-Foundations-and-Core-Language/Part-4-Primitive-Types-and-Expressions/Chapter-14-Strings.md)
15. [Booleans](Volume-1-Foundations-and-Core-Language/Part-4-Primitive-Types-and-Expressions/Chapter-15-Booleans.md)
16. [Operators](Volume-1-Foundations-and-Core-Language/Part-4-Primitive-Types-and-Expressions/Chapter-16-Operators.md)
17. [Expressions](Volume-1-Foundations-and-Core-Language/Part-4-Primitive-Types-and-Expressions/Chapter-17-Expressions.md)

### Part V — Control Flow

18. [Conditionals](Volume-1-Foundations-and-Core-Language/Part-5-Control-Flow/Chapter-18-Conditionals.md)
19. [Loops](Volume-1-Foundations-and-Core-Language/Part-5-Control-Flow/Chapter-19-Loops.md)

### Part VI — Functions

20. [Functions, Parameters, and Return Values](Volume-1-Foundations-and-Core-Language/Part-6-Functions/Chapter-20-Functions.md)
21. [Scope and Namespaces](Volume-1-Foundations-and-Core-Language/Part-6-Functions/Chapter-21-Scope.md)
22. [Closures](Volume-1-Foundations-and-Core-Language/Part-6-Functions/Chapter-22-Closures.md)
23. [Call Stack and Stack Frames](Volume-1-Foundations-and-Core-Language/Part-6-Functions/Chapter-23-Call-Stack.md)
24. [Recursion](Volume-1-Foundations-and-Core-Language/Part-6-Functions/Chapter-24-Recursion.md)
25. [Functional Programming](Volume-1-Foundations-and-Core-Language/Part-6-Functions/Chapter-25-Functional-Programming.md)

### Part VII — Data Structures

26. [Lists](Volume-1-Foundations-and-Core-Language/Part-7-Data-Structures/Chapter-26-Lists.md)
27. [Tuples](Volume-1-Foundations-and-Core-Language/Part-7-Data-Structures/Chapter-27-Tuples.md)
28. [Dictionaries](Volume-1-Foundations-and-Core-Language/Part-7-Data-Structures/Chapter-28-Dictionaries.md)
29. [Sets](Volume-1-Foundations-and-Core-Language/Part-7-Data-Structures/Chapter-29-Sets.md)
30. [Comprehension Patterns](Volume-1-Foundations-and-Core-Language/Part-7-Data-Structures/Chapter-30-Comprehension-Patterns.md)
31. [Specialized Collections: `deque`, `Counter`, `defaultdict`, `heapq`, and `bisect`](Volume-1-Foundations-and-Core-Language/Part-7-Data-Structures/Chapter-31-Specialized-Collections.md)
32. [Custom Data Structures](Volume-1-Foundations-and-Core-Language/Part-7-Data-Structures/Chapter-32-Custom-Data-Structures.md)

### Part VIII — Memory Management

33. [Stack vs Heap](Volume-1-Foundations-and-Core-Language/Part-8-Memory-Management/Chapter-33-Stack-vs-Heap.md)
34. [Reference Counting](Volume-1-Foundations-and-Core-Language/Part-8-Memory-Management/Chapter-34-Reference-Counting.md)
35. [Garbage Collection](Volume-1-Foundations-and-Core-Language/Part-8-Memory-Management/Chapter-35-Garbage-Collection.md)
36. [Object Lifecycle](Volume-1-Foundations-and-Core-Language/Part-8-Memory-Management/Chapter-36-Object-Lifecycle.md)
37. [Weak References](Volume-1-Foundations-and-Core-Language/Part-8-Memory-Management/Chapter-37-Weak-References.md)

### Part IX — Modules and Imports

38. [Modules](Volume-1-Foundations-and-Core-Language/Part-9-Modules-and-Imports/Chapter-38-Modules.md)
39. [Packages](Volume-1-Foundations-and-Core-Language/Part-9-Modules-and-Imports/Chapter-39-Packages.md)
40. [Import System](Volume-1-Foundations-and-Core-Language/Part-9-Modules-and-Imports/Chapter-40-Import-System.md)
41. [Namespaces](Volume-1-Foundations-and-Core-Language/Part-9-Modules-and-Imports/Chapter-41-Namespaces.md)
42. [Virtual Environments](Volume-1-Foundations-and-Core-Language/Part-9-Modules-and-Imports/Chapter-42-Virtual-Environments.md)

---

## Volume II — Advanced Python and Internals

Volume II moves from using Python correctly to understanding Python's advanced protocols, object model, runtime behavior, and implementation mechanics.

### Part I — Object-Oriented Python

43. [Classes and Instances](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-43-Classes-and-Instances.md)
44. [Attributes and Methods](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-44-Attributes-and-Methods.md)
45. [Encapsulation and Managed Attributes](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-45-Encapsulation-and-Managed-Attributes.md)
46. [Composition](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-46-Composition.md)
47. [Inheritance and Method Overriding](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-47-Inheritance-and-Method-Overriding.md)
48. [MRO and `super()`](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-48-MRO-and-super.md)
49. [Polymorphism and Duck Typing](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-49-Polymorphism-and-Duck-Typing.md)
50. [ABCs and Mixins](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-50-ABCs-and-Mixins.md)
51. [Dataclasses](Volume-2-Advanced-Python-and-Internals/Part-1-Object-Oriented-Python/Chapter-51-Dataclasses.md)

### Part II — The Python Data Model

52. [Dunder Methods](Volume-2-Advanced-Python-and-Internals/Part-2-The-Python-Data-Model/Chapter-52-Dunder-Methods.md)
53. [Operator Overloading](Volume-2-Advanced-Python-and-Internals/Part-2-The-Python-Data-Model/Chapter-53-Operator-Overloading.md)
54. [Descriptors](Volume-2-Advanced-Python-and-Internals/Part-2-The-Python-Data-Model/Chapter-54-Descriptors.md)
55. [Properties, Static Methods, and Class Methods](Volume-2-Advanced-Python-and-Internals/Part-2-The-Python-Data-Model/Chapter-55-Properties-Static-Methods-and-Class-Methods.md)
56. [`__slots__`](Volume-2-Advanced-Python-and-Internals/Part-2-The-Python-Data-Model/Chapter-56-slots.md)
57. [Metaclasses](Volume-2-Advanced-Python-and-Internals/Part-2-The-Python-Data-Model/Chapter-57-Metaclasses.md)

### Part III — Pythonic Abstractions

58. [Iterators](Volume-2-Advanced-Python-and-Internals/Part-3-Pythonic-Abstractions/Chapter-58-Iterators.md)
59. [Generators](Volume-2-Advanced-Python-and-Internals/Part-3-Pythonic-Abstractions/Chapter-59-Generators.md)
60. [Context Managers](Volume-2-Advanced-Python-and-Internals/Part-3-Pythonic-Abstractions/Chapter-60-Context-Managers.md)
61. [Decorators](Volume-2-Advanced-Python-and-Internals/Part-3-Pythonic-Abstractions/Chapter-61-Decorators.md)

### Part IV — Robust Programs and I/O

62. [Exceptions](Volume-2-Advanced-Python-and-Internals/Part-4-Robust-Programs-and-IO/Chapter-62-Exceptions.md)
63. [Files and Serialization](Volume-2-Advanced-Python-and-Internals/Part-4-Robust-Programs-and-IO/Chapter-63-Files-and-Serialization.md)
64. [Standard Library Deep Dive](Volume-2-Advanced-Python-and-Internals/Part-4-Robust-Programs-and-IO/Chapter-64-Standard-Library-Deep-Dive.md)

### Part V — Concurrency and Parallelism

65. [Concurrency Foundations](Volume-2-Advanced-Python-and-Internals/Part-5-Concurrency-and-Parallelism/Chapter-65-Concurrency-Foundations.md)
66. [Threads, Processes, and the GIL](Volume-2-Advanced-Python-and-Internals/Part-5-Concurrency-and-Parallelism/Chapter-66-Threads-Processes-and-the-GIL.md)
67. [Asyncio and Event Loops](Volume-2-Advanced-Python-and-Internals/Part-5-Concurrency-and-Parallelism/Chapter-67-Asyncio-and-Event-Loops.md)

### Part VI — Type System and Internals

68. [Runtime Type System](Volume-2-Advanced-Python-and-Internals/Part-6-Type-System-and-Internals/Chapter-68-Runtime-Type-System.md)
69. [Bytecode Internals](Volume-2-Advanced-Python-and-Internals/Part-6-Type-System-and-Internals/Chapter-69-Bytecode-Internals.md)
70. [CPython Architecture](Volume-2-Advanced-Python-and-Internals/Part-6-Type-System-and-Internals/Chapter-70-CPython-Architecture.md)
71. [C Extensions](Volume-2-Advanced-Python-and-Internals/Part-6-Type-System-and-Internals/Chapter-71-C-Extensions.md)

---

## Volume III — Software Engineering

Volume III turns Python knowledge into production engineering practice, including testing, debugging, packaging, modern project tooling, typing, profiling, design, architecture, APIs, and microservices.

72. [Testing](Volume-3-Software-Engineering/Chapter-72-Testing.md)
73. [Mocking and Monkey Patching](Volume-3-Software-Engineering/Chapter-73-Mocking-and-Monkey-Patching.md)
74. [Debugging](Volume-3-Software-Engineering/Chapter-74-Debugging.md)
75. [Logging](Volume-3-Software-Engineering/Chapter-75-Logging.md)
76. [Packaging](Volume-3-Software-Engineering/Chapter-76-Packaging.md)
77. [Type Hints](Volume-3-Software-Engineering/Chapter-77-Type-Hints.md)
78. [Static Type Checking](Volume-3-Software-Engineering/Chapter-78-Static-Type-Checking.md)
79. [Profiling](Volume-3-Software-Engineering/Chapter-79-Profiling.md)
80. [Design Patterns](Volume-3-Software-Engineering/Chapter-80-Design-Patterns.md)
81. [SOLID Principles](Volume-3-Software-Engineering/Chapter-81-SOLID-Principles.md)
82. [Architecture](Volume-3-Software-Engineering/Chapter-82-Architecture.md)
83. [APIs](Volume-3-Software-Engineering/Chapter-83-APIs.md)
84. [Microservices](Volume-3-Software-Engineering/Chapter-84-Microservices.md)

---

## Volume IV — Ecosystem and Career Paths

Volume IV connects Python mastery to real-world domains, frameworks, libraries, modern LLM tooling, model hubs, AI systems, automation, and professional specialization paths.

### Part I — Web Development

85. [Flask](Volume-4-Ecosystem-and-Career-Paths/Part-1-Web-Development/Chapter-85-Flask.md)
86. [Django](Volume-4-Ecosystem-and-Career-Paths/Part-1-Web-Development/Chapter-86-Django.md)
87. [FastAPI](Volume-4-Ecosystem-and-Career-Paths/Part-1-Web-Development/Chapter-87-FastAPI.md)

### Part II — Data and Scientific Computing

88. [NumPy](Volume-4-Ecosystem-and-Career-Paths/Part-2-Data-and-Scientific-Computing/Chapter-88-NumPy.md)
89. [Pandas](Volume-4-Ecosystem-and-Career-Paths/Part-2-Data-and-Scientific-Computing/Chapter-89-Pandas.md)
90. [Polars](Volume-4-Ecosystem-and-Career-Paths/Part-2-Data-and-Scientific-Computing/Chapter-90-Polars.md)
91. [SciPy](Volume-4-Ecosystem-and-Career-Paths/Part-2-Data-and-Scientific-Computing/Chapter-91-SciPy.md)

### Part III — Machine Learning and AI Engineering

92. [scikit-learn](Volume-4-Ecosystem-and-Career-Paths/Part-3-Machine-Learning-and-AI-Engineering/Chapter-92-scikit-learn.md)
93. [PyTorch](Volume-4-Ecosystem-and-Career-Paths/Part-3-Machine-Learning-and-AI-Engineering/Chapter-93-PyTorch.md)
94. [TensorFlow](Volume-4-Ecosystem-and-Career-Paths/Part-3-Machine-Learning-and-AI-Engineering/Chapter-94-TensorFlow.md)
95. [AI Engineering](Volume-4-Ecosystem-and-Career-Paths/Part-3-Machine-Learning-and-AI-Engineering/Chapter-95-AI-Engineering.md)
96. [RAG Systems](Volume-4-Ecosystem-and-Career-Paths/Part-3-Machine-Learning-and-AI-Engineering/Chapter-96-RAG-Systems.md)
97. [Agents](Volume-4-Ecosystem-and-Career-Paths/Part-3-Machine-Learning-and-AI-Engineering/Chapter-97-Agents.md)

### Part IV — Automation and Integration

98. [APIs and Automation](Volume-4-Ecosystem-and-Career-Paths/Part-4-Automation-and-Integration/Chapter-98-APIs-and-Automation.md)
99. [Scripting for Real Workflows](Volume-4-Ecosystem-and-Career-Paths/Part-4-Automation-and-Integration/Chapter-99-Scripting-for-Real-Workflows.md)

### Part V — Career Paths

100. [Backend Engineering](Volume-4-Ecosystem-and-Career-Paths/Part-5-Career-Paths/Chapter-100-Backend-Engineering.md)
101. [Data Engineering](Volume-4-Ecosystem-and-Career-Paths/Part-5-Career-Paths/Chapter-101-Data-Engineering.md)
102. [Machine Learning Engineering](Volume-4-Ecosystem-and-Career-Paths/Part-5-Career-Paths/Chapter-102-Machine-Learning-Engineering.md)
103. [AI Engineering](Volume-4-Ecosystem-and-Career-Paths/Part-5-Career-Paths/Chapter-103-AI-Engineering.md)
104. [DevOps](Volume-4-Ecosystem-and-Career-Paths/Part-5-Career-Paths/Chapter-104-DevOps.md)
105. [Cybersecurity](Volume-4-Ecosystem-and-Career-Paths/Part-5-Career-Paths/Chapter-105-Cybersecurity.md)

---

# Capstone Projects

Capstones are placed after the learner has the prerequisites needed to build them properly. They should reinforce the book instead of acting as disconnected exercises.

* [Todo CLI](Capstone-Projects/Capstone-01-Todo-CLI.md)
* [File Organizer](Capstone-Projects/Capstone-02-File-Organizer.md)
* [REST API](Capstone-Projects/Capstone-03-REST-API.md)
* [URL Shortener](Capstone-Projects/Capstone-04-URL-Shortener.md)
* [ORM](Capstone-Projects/Capstone-05-ORM.md)
* [Task Queue](Capstone-Projects/Capstone-06-Task-Queue.md)
* [Mini Redis](Capstone-Projects/Capstone-07-Mini-Redis.md)
* [Mini Web Framework](Capstone-Projects/Capstone-08-Mini-Web-Framework.md)
* [Mini Event Loop](Capstone-Projects/Capstone-09-Mini-Event-Loop.md)
* [Toy Python Interpreter](Capstone-Projects/Capstone-10-Toy-Python-Interpreter.md)
* [Distributed Scheduler](Capstone-Projects/Capstone-11-Distributed-Scheduler.md)

---

# Goal

After completing this curriculum, the learner should need only:

* Official documentation.
* Domain-specific resources.

No general Python tutorial should be necessary.
