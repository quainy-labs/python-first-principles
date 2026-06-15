# Chapter 09 — Everything Is an Object

---

# Learning Objectives

By the end of this chapter, you should understand:

* What an object is in Python.
* Why Python says everything is an object.
* Why numbers, strings, lists, functions, classes, modules, and exceptions are all objects.
* The three core properties every Python object has: identity, type, and value.
* What `type()` tells you.
* What `id()` tells you.
* Why an object's type determines its behavior.
* Why an object's value is not the same thing as its identity.
* Why objects live at runtime, not merely in source code.
* How bytecode from Chapter 08 operates on Python objects.
* Why Python's object model makes the language consistent.
* Why this chapter is the foundation for names, assignment, mutability, equality, functions, classes, and data structures.

This chapter begins one of the most important mental models in the entire book:

> Python programs manipulate objects.

---

# Concept Overview

In earlier chapters, we learned how Python code runs.

We saw this flow:

```text
source code
  -> AST
  -> bytecode
  -> Python Virtual Machine
  -> runtime behavior
```

Now we need to understand what runtime behavior operates on.

It operates on objects.

When Python executes:

```python
x = 10
```

the value `10` is not just a raw mark in the source file.

At runtime, Python works with an integer object representing `10`.

When Python executes:

```python
message = "hello"
```

Python works with a string object.

When Python executes:

```python
items = [1, 2, 3]
```

Python works with a list object.

When Python executes:

```python
def greet():
    print("hello")
```

Python creates a function object.

This is what Python means by:

> Everything is an object.

It does not mean every object behaves the same way.

It means every runtime value participates in Python's object model.

---

# The Core Mental Model

Every Python object has three essential properties:

```text
object
├── identity
├── type
└── value
```

These three ideas answer three different questions:

```text
identity:
    Which specific object is this?

type:
    What kind of object is this?

value:
    What data or state does this object represent?
```

Example:

```python
42
```

At runtime, this is an object.

Conceptually:

```text
object
├── identity: unique object identity
├── type: int
└── value: 42
```

Example:

```python
"Python"
```

Conceptually:

```text
object
├── identity: unique object identity
├── type: str
└── value: "Python"
```

Example:

```python
[1, 2, 3]
```

Conceptually:

```text
object
├── identity: unique object identity
├── type: list
└── value: sequence containing 1, 2, 3
```

This chapter explains these ideas carefully.

---

# Why This Mental Model Matters

Many Python confusions come from a weak object model.

For example:

```python
a = [1, 2]
b = a
b.append(3)
print(a)
```

Why does `a` change?

That question cannot be answered well until you understand:

* What a list object is.
* What identity means.
* What names refer to.
* What mutation means.

This chapter handles the first part:

> What is the object?

Chapter 10 handles:

> How do names refer to objects?

Chapter 11 handles:

> What does it mean for an object to change?

Chapter 12 handles:

> How do identity, equality, and memory diagrams work together?

We are building the model in dependency order.

---

# What Is an Object?

An object is a runtime entity that Python can work with.

Objects can represent:

* Numbers
* Text
* Collections
* Functions
* Classes
* Modules
* Files
* Exceptions
* Iterators
* User-defined data

An object is not just data.

An object combines:

* Data or state
* Type information
* Behavior
* Identity

For example, a string object contains text data:

```python
"hello"
```

But it also has behavior:

```python
"hello".upper()
```

Output:

```text
HELLO
```

The object knows how to respond to operations because of its type.

---

# Object as Data Plus Behavior

In Python, values are not just passive data.

They come with operations.

Example:

```python
name = "python"
print(name.upper())
```

The string object has a method named `upper`.

Example:

```python
numbers = [1, 2, 3]
numbers.append(4)
print(numbers)
```

The list object has a method named `append`.

Example:

```python
count = 10
print(count.bit_length())
```

The integer object has a method named `bit_length`.

This surprises many learners.

They expect objects only when they define classes.

But in Python, built-in values are objects too.

---

# Why Python Uses Objects for Everything

Python uses one object model because consistency makes the language easier to understand and extend.

If only some things were objects, Python would need many special rules.

For example:

```python
len("hello")
len([1, 2, 3])
len({"a": 1})
```

These work on different types of objects:

* String object
* List object
* Dictionary object

The behavior differs, but the object model is consistent.

Each object knows enough about itself for Python to ask questions such as:

* What type are you?
* What operations do you support?
* What attributes do you have?
* How should you be represented?
* How should you be compared?

This consistency is one reason Python is expressive.

---

# Objects Exist at Runtime

Source code is text.

Objects exist while the program runs.

Consider:

```python
x = 100
```

In the source file, `100` is characters:

```text
1
0
0
```

During execution, Python creates or loads an integer object representing the value `100`.

The object exists in runtime memory.

Conceptually:

```text
source text:
    x = 100

runtime:
    integer object with value 100
```

This distinction matters.

The source code describes objects that should exist during execution.

The objects themselves are runtime entities.

---

# Objects and Bytecode

Chapter 08 taught that bytecode instructions operate on runtime state.

Now we can say that more precisely:

> Bytecode loads, stores, creates, compares, calls, and manipulates objects.

Example:

```python
x = 10
```

Conceptual bytecode:

```text
LOAD_CONST 10
STORE_NAME x
```

The `LOAD_CONST` operation loads an object representing `10`.

Example:

```python
print("hello")
```

Conceptually:

```text
load print object
load string object "hello"
call print object with string object
```

The runtime world is object-based.

Bytecode is the instruction layer.

Objects are the things being operated on.

---

# Everything Is an Object

Let us test the claim.

Numbers are objects:

```python
print(type(10))
print(type(3.14))
print(type(True))
```

Output:

```text
<class 'int'>
<class 'float'>
<class 'bool'>
```

Strings are objects:

```python
print(type("hello"))
```

Output:

```text
<class 'str'>
```

Lists are objects:

```python
print(type([1, 2, 3]))
```

Output:

```text
<class 'list'>
```

Dictionaries are objects:

```python
print(type({"name": "Ada"}))
```

Output:

```text
<class 'dict'>
```

Functions are objects:

```python
def greet():
    print("hello")

print(type(greet))
```

Output:

```text
<class 'function'>
```

Modules are objects:

```python
import math

print(type(math))
```

Output:

```text
<class 'module'>
```

Classes are objects:

```python
class User:
    pass

print(type(User))
```

Output:

```text
<class 'type'>
```

Instances are objects:

```python
user = User()
print(type(user))
```

Output:

```text
<class '__main__.User'>
```

Everything you work with at runtime is an object.

---

# The Meaning of type()

The built-in function `type()` tells you an object's type.

Example:

```python
print(type(10))
```

Output:

```text
<class 'int'>
```

This means:

```text
The object 10 has type int.
```

Example:

```python
print(type("hello"))
```

Output:

```text
<class 'str'>
```

This means:

```text
The object "hello" has type str.
```

The output uses the word `class` because Python types are themselves represented by class objects.

You do not need to fully understand classes yet.

For now, remember:

> An object's type determines what kind of object it is and what operations it supports.

---

# Type Determines Behavior

The same operator can behave differently depending on object type.

Example:

```python
print(10 + 20)
```

Output:

```text
30
```

Here, `+` performs integer addition.

Now:

```python
print("Py" + "thon")
```

Output:

```text
Python
```

Here, `+` performs string concatenation.

Now:

```python
print([1, 2] + [3, 4])
```

Output:

```text
[1, 2, 3, 4]
```

Here, `+` performs list concatenation.

The symbol is the same.

The behavior depends on the objects involved.

This is possible because objects know their type, and types define behavior.

---

# Type Determines Supported Operations

Some operations make sense for one type but not another.

Example:

```python
print("hello".upper())
```

Output:

```text
HELLO
```

String objects support `upper()`.

Integer objects do not:

```python
print((10).upper())
```

This raises an error:

```text
AttributeError
```

Why?

Because `10` is an `int` object, and `int` objects do not have an `upper` method.

The error is not arbitrary.

It follows from the object model:

```text
object -> type -> available behavior
```

---

# Attributes and Methods

Objects can have attributes.

An attribute is something accessed using dot syntax:

```python
object.attribute
```

A method is a callable attribute usually associated with an object's behavior.

Example:

```python
text = "python"
print(text.upper())
```

Here:

```text
text        -> name referring to a string object
upper       -> method available on that string object
upper()     -> call to that method
```

Example:

```python
items = [1, 2]
items.append(3)
```

Here:

```text
items       -> name referring to a list object
append      -> method available on that list object
append(3)   -> call to that method
```

We will study methods deeply later.

For now, understand:

> Objects carry behavior through their type.

---

# Identity

Every object has an identity.

Identity answers:

> Which specific object is this?

The built-in function `id()` returns an integer representing an object's identity during its lifetime.

Example:

```python
x = [1, 2, 3]
print(id(x))
```

You may see something like:

```text
4372840064
```

Your number will likely be different.

That is normal.

The actual value is not important.

The meaning is important:

```text
This object has a specific identity.
```

Two objects can have the same value but different identities.

Example:

```python
a = [1, 2]
b = [1, 2]

print(a == b)
print(id(a))
print(id(b))
```

Output conceptually:

```text
True
different identity number
different identity number
```

The lists have equal values.

They are not the same object.

Chapter 12 will explore this distinction deeply.

---

# Identity Is Not Value

Identity and value are different questions.

Value asks:

> What data does this object represent?

Identity asks:

> Is this the same specific object?

Consider two passports with the same name printed on them.

The printed name may be the same.

The passports are still two different physical objects.

In Python:

```python
a = [1, 2]
b = [1, 2]
```

Conceptually:

```text
a -> list object #1 with value [1, 2]
b -> list object #2 with value [1, 2]
```

Same value.

Different objects.

Different identities.

---

# Value

An object's value is the data or state it represents.

Examples:

```python
10
```

Value:

```text
10
```

```python
"hello"
```

Value:

```text
the text "hello"
```

```python
[1, 2, 3]
```

Value:

```text
a sequence containing 1, 2, 3
```

For simple objects, value feels obvious.

For richer objects, value can mean internal state.

Example:

```python
from datetime import date

today = date(2026, 6, 11)
```

The object's value represents a calendar date.

The internal implementation is more complex than plain text, but the conceptual value is:

```text
June 11, 2026
```

---

# Object Representation

When you print an object, Python shows a representation of the object.

Example:

```python
print([1, 2, 3])
```

Output:

```text
[1, 2, 3]
```

But the printed text is not the object itself.

It is a representation of the object.

Example:

```python
print(type([1, 2, 3]))
```

Output:

```text
<class 'list'>
```

This output represents type information.

It is not the same thing as the internal object in memory.

This distinction matters:

```text
object:
    runtime entity

printed representation:
    text Python displays to humans
```

---

# Objects and Memory

Objects live in memory while the program runs.

From Chapter 02, we know memory stores data needed during execution.

From Chapter 03, we know a process has its own memory space.

When a Python program runs, Python objects live inside the Python process memory.

Conceptually:

```text
Python process memory
├── integer object 10
├── string object "hello"
├── list object [1, 2, 3]
├── function object greet
└── module object math
```

You do not manually choose memory addresses in normal Python code.

CPython manages object allocation for you.

But objects still exist in memory.

That is why identity and lifetime matter.

---

# id() and Memory Addresses

In CPython, `id()` is closely related to the object's memory address.

But as a Python learner, you should not rely on `id()` as a portable memory-address tool.

The language-level promise is:

> `id()` returns a value that is unique and constant for an object during its lifetime.

This means:

```python
x = [1, 2, 3]
print(id(x))
print(id(x))
```

The two printed values are the same while that object exists.

But after an object is destroyed, Python may reuse identities for new objects.

So `id()` is useful for learning and debugging object identity, not for manual memory management.

---

# Object Lifetime

Objects are created, used, and eventually destroyed or reclaimed.

Example:

```python
def make_list():
    items = [1, 2, 3]
    return items
```

When `make_list()` runs, a list object is created.

If the function returns the list, the object can continue to exist after the function frame finishes.

If no part of the program can reach an object anymore, Python can eventually reclaim it.

We will study reference counting and garbage collection later.

For now, remember:

> Objects have lifetimes. They exist during runtime while Python can still use them.

---

# Literals Create or Load Objects

A literal is a value written directly in source code.

Examples:

```python
10
"hello"
[1, 2, 3]
{"name": "Ada"}
```

During execution, literals produce objects.

Example:

```python
x = "hello"
```

The string literal `"hello"` corresponds to a string object at runtime.

Example:

```python
items = [1, 2, 3]
```

The list literal creates a list object at runtime.

Different literal forms create different types of objects:

| Literal | Object Type |
| --- | --- |
| `10` | `int` |
| `3.14` | `float` |
| `"hello"` | `str` |
| `True` | `bool` |
| `[1, 2]` | `list` |
| `(1, 2)` | `tuple` |
| `{"a": 1}` | `dict` |
| `{1, 2}` | `set` |
| `None` | `NoneType` |

This table will become more meaningful as we study each type later.

---

# Built-in Types Are Classes

When you run:

```python
print(type(10))
```

you see:

```text
<class 'int'>
```

This tells us that `int` is a class.

Likewise:

```python
print(type("hello"))
print(type([1, 2, 3]))
print(type({"a": 1}))
```

Outputs:

```text
<class 'str'>
<class 'list'>
<class 'dict'>
```

Python's built-in types are classes.

Objects are instances of classes.

This means:

```text
10          -> instance of int
"hello"    -> instance of str
[1, 2, 3]  -> instance of list
```

We will study classes later.

For now, the key idea is:

> A type is not just a label. It is the source of an object's behavior.

---

# Functions Are Objects

Functions are not just syntax.

They are objects created at runtime.

Example:

```python
def greet():
    print("hello")

print(type(greet))
```

Output:

```text
<class 'function'>
```

This matters because function objects can be:

* Assigned to names
* Passed to other functions
* Returned from functions
* Stored in lists or dictionaries
* Given attributes in some cases

Example:

```python
def greet():
    print("hello")

say_hello = greet
say_hello()
```

Output:

```text
hello
```

This works because `greet` refers to a function object.

The name `say_hello` can refer to the same function object.

Chapter 10 will explain that name-reference relationship in detail.

---

# Classes Are Objects

Classes are objects too.

Example:

```python
class User:
    pass

print(type(User))
```

Output:

```text
<class 'type'>
```

This is advanced, but important.

In Python, a class is not merely a compile-time blueprint.

It is a runtime object.

That class object can be:

* Assigned to names
* Passed to functions
* Stored in collections
* Inspected
* Used to create instances

Example:

```python
class User:
    pass

u = User()
print(type(u))
```

Output:

```text
<class '__main__.User'>
```

The class `User` is an object.

The instance `u` is also an object.

They are different kinds of objects.

---

# Modules Are Objects

When you import a module, Python creates or reuses a module object.

Example:

```python
import math

print(type(math))
```

Output:

```text
<class 'module'>
```

The module object contains attributes:

```python
print(math.sqrt(16))
```

Output:

```text
4.0
```

Here:

```text
math       -> module object
sqrt       -> attribute of the module object
sqrt(16)   -> function call
```

This connects to Chapter 08:

Importing a module creates and initializes a module object.

---

# Exceptions Are Objects

Errors in Python are represented by exception objects.

Example:

```python
try:
    1 / 0
except ZeroDivisionError as error:
    print(type(error))
    print(error)
```

Output:

```text
<class 'ZeroDivisionError'>
division by zero
```

The exception is not just a message.

It is an object that carries information about what went wrong.

Later, when we study error handling, this will matter a lot.

---

# None Is an Object

`None` represents the absence of a meaningful value.

It is also an object.

Example:

```python
print(type(None))
```

Output:

```text
<class 'NoneType'>
```

Functions that do not explicitly return a value return `None`.

Example:

```python
def say_hi():
    print("hi")

result = say_hi()
print(result)
print(type(result))
```

Output:

```text
hi
None
<class 'NoneType'>
```

Even absence is represented by an object in Python.

---

# Booleans Are Objects

`True` and `False` are boolean objects.

Example:

```python
print(type(True))
print(type(False))
```

Output:

```text
<class 'bool'>
<class 'bool'>
```

Boolean objects are used in conditions:

```python
if True:
    print("runs")
```

Output:

```text
runs
```

Booleans will be studied in detail later.

For now, remember:

> Conditions operate on objects and their truth value.

---

# Operators Work Through Objects

Operators are not separate from the object model.

Example:

```python
10 + 20
```

This works because integer objects support addition.

Example:

```python
"a" + "b"
```

This works because string objects support concatenation.

Example:

```python
[1] + [2]
```

This works because list objects support concatenation.

But:

```python
"a" + 1
```

raises:

```text
TypeError
```

Why?

Because Python does not know how to add a string object and an integer object using `+`.

Again:

```text
object type -> supported behavior
```

---

# Objects Can Be Inspected

Python is highly introspective.

Introspection means a program can examine objects at runtime.

Examples:

```python
value = "hello"

print(type(value))
print(id(value))
print(dir(value))
```

`type(value)` tells you the object's type.

`id(value)` tells you the object's identity.

`dir(value)` shows many available attributes.

You do not need to memorize the full output of `dir()`.

The important lesson is:

> Python objects carry inspectable structure.

This is useful for debugging, learning, interactive exploration, and tooling.

---

# dir()

The built-in function `dir()` returns a list of names available on an object.

Example:

```python
print(dir("hello"))
```

You will see many names, including methods such as:

```text
upper
lower
replace
split
strip
```

These are operations string objects support.

Example:

```python
print("hello".upper())
print("hello".replace("h", "H"))
```

Output:

```text
HELLO
Hello
```

`dir()` is not something you use constantly in production code, but it is helpful for exploration.

---

# Object Model and Error Messages

Many Python error messages are object-model messages.

Example:

```python
number = 10
number.upper()
```

Error:

```text
AttributeError: 'int' object has no attribute 'upper'
```

Read this carefully:

```text
'int' object
```

Python is telling you the type of object.

```text
has no attribute 'upper'
```

Python is telling you that this type does not provide that behavior.

Another example:

```python
"age: " + 30
```

Error:

```text
TypeError
```

Python is telling you the operation is not supported between those object types.

Good Python debugging often starts by asking:

```text
What object do I have?
What type is it?
What operation am I asking it to perform?
Does that type support that operation?
```

---

# Object Model and APIs

APIs often expect certain kinds of objects.

Example:

```python
len("hello")
len([1, 2, 3])
len({"a": 1})
```

These all work because each object supports length.

But:

```python
len(10)
```

raises:

```text
TypeError
```

An integer object does not have a length.

This is not random.

It follows from object behavior.

When reading documentation, you will often see statements like:

```text
This function expects a file-like object.
This function expects an iterable.
This function expects a mapping.
```

Those descriptions are about object behavior.

Python code often cares less about the exact type name and more about what behavior an object supports.

This idea will later lead to duck typing.

---

# Object Model and Duck Typing

Python often follows duck typing.

Duck typing means:

> If an object supports the behavior needed, Python code can often use it.

For example, many functions do not require exactly a list.

They require something iterable.

Example:

```python
for item in [1, 2, 3]:
    print(item)
```

works with a list.

But:

```python
for char in "abc":
    print(char)
```

works with a string.

The objects are different types.

They both support iteration.

Do not worry about iteration yet.

The key idea is:

> Python's object model allows behavior-based programming.

---

# Object Identity and is

Python has an operator named `is`.

`is` checks identity.

It asks:

> Are these two references pointing to the same object?

Example:

```python
a = [1, 2]
b = a

print(a is b)
```

Output:

```text
True
```

Example:

```python
a = [1, 2]
b = [1, 2]

print(a is b)
```

Output:

```text
False
```

The second example has two list objects.

They have equal contents, but different identities.

Chapter 12 will explore `is` and `==` in detail.

For now, remember:

```text
is   -> identity question
==   -> value/equality question
```

---

# Equality at a High Level

Equality asks whether objects should be considered equal in value.

Example:

```python
print([1, 2] == [1, 2])
```

Output:

```text
True
```

Those are two different list objects.

But their values are equal.

Example:

```python
print("hello" == "hello")
```

Output:

```text
True
```

Equality is type-specific behavior.

Different object types can define equality differently.

We will study this more carefully later.

For now, the key distinction is:

```text
identity:
    same object?

equality:
    same value according to type-specific comparison?
```

---

# Avoid Overusing id()

`id()` is useful for learning.

It helps you see identity.

But normal Python programs rarely need to call `id()`.

Most code should reason using:

* Values
* Types
* Behavior
* Equality
* Object relationships

Use `id()` when you are learning, debugging identity, or exploring object behavior.

Do not use it as a normal substitute for program logic.

---

# Small Object Caching and Interning

You may see surprising results with small integers or strings.

Example:

```python
a = 10
b = 10
print(a is b)
```

This may print:

```text
True
```

Does that mean all equal integers are always the same object?

No.

CPython may reuse certain small integer objects and intern some strings as an optimization.

This is an implementation detail.

Do not build program logic around it.

Use `==` when you mean equality.

Use `is` only when you mean identity, especially for singleton objects such as `None`.

Example:

```python
if result is None:
    print("No result")
```

This is correct because `None` is a singleton object.

We will revisit this in Chapter 12.

---

# What Everything Is an Object Does Not Mean

The phrase can be misunderstood.

It does not mean:

```text
All objects have the same behavior.
```

They do not.

An integer object and a list object behave differently.

It does not mean:

```text
All objects can be changed.
```

Some objects are mutable.

Some are immutable.

That is Chapter 11.

It does not mean:

```text
Names are objects.
```

Names are not objects in the same way values are objects.

Names refer to objects.

That is Chapter 10.

It does not mean:

```text
Objects are always created fresh every time you write a literal.
```

Python may reuse some objects.

Object creation and reuse are implementation details unless the language promises specific behavior.

---

# Python vs Languages With Primitive Values

Some languages distinguish between primitive values and objects.

For example, a language may have:

```text
primitive int
object String
object User
```

In those languages, primitive values may not have methods or object identity in the same way.

Python takes a more uniform approach:

```python
print((10).bit_length())
```

Output:

```text
4
```

The integer `10` has behavior because it is an object.

This uniformity makes Python simpler conceptually:

```text
runtime value -> object
```

But it also means Python has overhead compared with lower-level representations.

Integer objects carry more structure than raw CPU integers.

That is part of Python's tradeoff:

* More flexibility and consistency.
* Less raw performance than low-level primitive machine values.

---

# Internal Mechanics at a High Level

In CPython, objects are represented internally by C structures.

You do not need to read C code yet.

But a high-level view is useful.

A Python object needs to carry information such as:

```text
reference count
type pointer
object-specific data
```

Very roughly:

```text
CPython object
├── reference management information
├── pointer to type object
└── data for this specific object
```

For an integer object, the data includes the integer value.

For a list object, the data includes references to contained objects.

For a function object, the data includes a code object and other function metadata.

This is why Python values are richer than raw machine values.

---

# Type Objects

Types themselves are objects.

Example:

```python
print(type(10))
```

Output:

```text
<class 'int'>
```

The object `10` has type `int`.

But `int` itself is also an object:

```python
print(type(int))
```

Output:

```text
<class 'type'>
```

This can feel strange at first.

Do not worry if it feels recursive.

The important beginner model is:

```text
ordinary object -> has a type
type object     -> defines behavior for ordinary objects
```

Later, when we study classes and metaclasses, this will become clearer.

For now, it is enough to know that Python's object model is highly consistent.

---

# Containers Hold References to Objects

Containers such as lists, tuples, dictionaries, and sets hold objects.

More precisely, they hold references to objects.

Example:

```python
items = [10, "hello", True]
```

Conceptually:

```text
list object
├── reference to int object 10
├── reference to str object "hello"
└── reference to bool object True
```

This is why Python containers can hold mixed types.

Example:

```python
items = [1, "two", 3.0, None]
```

The list can contain different kinds of objects because it stores references to objects.

We will study references in Chapter 10.

For now, remember:

> Containers are objects that organize references to other objects.

---

# Nested Objects

Objects can contain or refer to other objects.

Example:

```python
matrix = [[1, 2], [3, 4]]
```

Conceptually:

```text
outer list object
├── reference to inner list object [1, 2]
└── reference to inner list object [3, 4]
```

The integers inside are objects too.

A nested data structure is a network of objects.

This is why memory diagrams are so important in Python.

They help you see relationships between objects.

---

# Object Model and Data Structures

Every data structure we study later is an object.

Lists are objects.

Tuples are objects.

Dictionaries are objects.

Sets are objects.

Each data structure type provides different behavior:

```text
list:
    ordered, indexable, mutable sequence

tuple:
    ordered, indexable, immutable sequence

dict:
    key-value mapping

set:
    unordered collection of unique elements
```

These differences are type-level behavior.

The object model lets us ask:

```text
What type of object is this?
What value does it represent?
What operations does it support?
What other objects does it refer to?
```

---

# Object Model and Functions

Because functions are objects, Python can support powerful patterns.

Example:

```python
def shout(text):
    return text.upper()

def apply(func, value):
    return func(value)

print(apply(shout, "python"))
```

Output:

```text
PYTHON
```

The function `shout` is passed as an object to `apply`.

This is not special magic.

It follows from:

```text
functions are objects
names can refer to function objects
function objects can be passed around
```

We will return to this when studying functions deeply.

---

# Object Model and Classes

Later, when you define your own classes, you create new types of objects.

Example:

```python
class Point:
    pass

p = Point()
```

Here:

```text
Point -> class object
p     -> instance object of type Point
```

The instance has:

```text
identity
type
value/state
```

This is the same pattern as built-in objects.

Custom objects are not a separate universe.

They participate in the same object model as integers, strings, lists, functions, and modules.

---

# Object Model and Real Programs

In real Python applications, almost everything you touch is an object:

```text
HTTP request object
database connection object
user model object
configuration object
logger object
file object
exception object
dataframe object
tensor object
API client object
```

Frameworks and libraries expose objects with behavior.

Example:

```python
response.status_code
response.json()
```

Here, `response` is an object.

It has data:

```text
status code
headers
body
```

It has behavior:

```text
json()
text
raise_for_status()
```

Understanding Python objects prepares you to understand real APIs.

---

# Common Misconceptions

## Misconception 1

### Only class instances are objects.

In Python, integers, strings, lists, functions, modules, classes, exceptions, and `None` are all objects.

Class instances are one category of object, not the only category.

---

## Misconception 2

### Variables contain objects.

This wording often creates the wrong mental model.

Names refer to objects.

The object exists in memory.

The name is a way to access it.

Chapter 10 will explain this carefully.

---

## Misconception 3

### `type()` returns a string.

`type()` returns a type object.

It may print in a text-like way:

```text
<class 'int'>
```

but the result is not merely a string.

Example:

```python
print(type(10) is int)
```

Output:

```text
True
```

---

## Misconception 4

### `id()` tells you the value of an object.

`id()` tells you identity, not value.

Two objects can have equal values but different identities.

---

## Misconception 5

### Equal objects are always the same object.

Objects can be equal without being identical.

Example:

```python
[1, 2] == [1, 2]
```

is true.

But these are usually two different list objects.

---

## Misconception 6

### If everything is an object, everything is mutable.

No.

Some objects are mutable.

Some objects are immutable.

Integers and strings are immutable.

Lists and dictionaries are mutable.

Mutability is Chapter 11.

---

## Misconception 7

### Object identity should be used for normal value comparison.

Usually, use `==` for value comparison.

Use `is` for identity checks, especially with singletons such as `None`.

---

# Real-world Usage

## Debugging

When a program behaves unexpectedly, ask:

```text
What object do I have?
What is its type?
What is its value?
What operation am I performing?
Does that type support that operation?
```

Example:

```python
value = "10"
print(value + 5)
```

This raises an error because `"10"` is a string object and `5` is an integer object.

The problem is not the visible characters.

The problem is the object types.

---

## Reading Error Messages

Python error messages often name object types:

```text
AttributeError: 'list' object has no attribute 'upper'
TypeError: object of type 'int' has no len()
TypeError: unsupported operand type(s)
```

These messages become clearer once you think in objects.

---

## Reading Documentation

Documentation describes object behavior.

Examples:

```text
str.upper()
list.append(x)
dict.get(key)
Path.exists()
Response.json()
DataFrame.groupby()
Tensor.shape
```

These are attributes and methods on objects.

Understanding objects makes documentation easier to read.

---

## Designing Code

When you write Python programs, you often design objects.

Even before writing classes, you choose objects:

* Should this data be a list?
* Should this data be a dictionary?
* Should this behavior be a function?
* Should this state become a class instance later?

Good Python design starts with understanding what objects represent.

---

## Working With Libraries

Libraries expose objects.

Examples:

```python
path.exists()
response.json()
cursor.execute(sql)
df.head()
model.predict(data)
```

Each line uses an object with behavior.

When you understand object behavior, libraries become less mysterious.

---

# Concept Connections

This chapter connects previous concepts:

```text
Chapter 01:
Software manipulates data and instructions.

Chapter 02:
Runtime data lives in memory.

Chapter 03:
A running process has memory and execution state.

Chapter 04:
The OS provides the process environment where Python runs.

Chapter 07:
Source code becomes executable internal structure.

Chapter 08:
Bytecode executes and manipulates runtime values.

Chapter 09:
Those runtime values are Python objects.
```

This chapter also prepares upcoming chapters:

```text
Chapter 10:
Names refer to objects.

Chapter 11:
Some objects can change in place.

Chapter 12:
Identity and equality answer different object questions.

Later:
Functions, classes, modules, exceptions, and data structures are all objects.
```

The larger mental model is:

```text
Python program
    |
    v
bytecode execution
    |
    v
objects in memory
    |
    v
names, references, operations, and behavior
```

---

# Internal Mechanics Summary

At a high level, a Python object has:

```text
object
├── identity
├── type
├── value/state
└── behavior from its type
```

In CPython, an object is implemented with internal runtime structures.

Conceptually:

```text
CPython object
├── reference management information
├── type information
└── object-specific data
```

Examples:

```text
int object
├── type: int
└── value: integer value

str object
├── type: str
└── value: text data

list object
├── type: list
└── value: references to contained objects

function object
├── type: function
└── value/state: code object, globals, defaults, metadata
```

Important built-ins:

| Tool | Purpose |
| --- | --- |
| `type(obj)` | Shows the object's type |
| `id(obj)` | Shows the object's identity |
| `dir(obj)` | Lists many available attributes |
| `is` | Checks identity |
| `==` | Checks equality/value comparison |

---

# Active Recall

## Easy Recall Questions

1. What does it mean to say everything is an object in Python?
2. Name the three essential properties every Python object has.
3. What does `type()` tell you?
4. What does `id()` tell you?
5. Is a string an object?
6. Is a function an object?
7. Is a class an object?
8. Is `None` an object?
9. What does an object's type determine?
10. What is the difference between an object and its printed representation?

---

## Deep Understanding Questions

1. Why does Python use one object model for many different kinds of values?
2. Why is type more than just a label?
3. Why can the same operator behave differently for different object types?
4. Why is identity different from value?
5. Why can two objects be equal but not identical?
6. Why should `id()` not usually be used for normal program logic?
7. Why does the object model help explain error messages?
8. Why does saying "variables contain objects" lead to confusion?
9. Why are functions being objects important for Python programming?
10. Why does this chapter need to come before mutability?

---

## Explain In Your Own Words

1. Explain what an object is without using the word "class."
2. Explain identity, type, and value using an example.
3. Explain why `10`, `"hello"`, and `[1, 2, 3]` are all objects but behave differently.
4. Explain what `type("hello")` tells you.
5. Explain why `[1, 2] == [1, 2]` can be true even when the two lists are not the same object.
6. Explain why `None` being an object fits Python's design.
7. Explain why a function can be assigned to another name.

---

## Predict-the-Output Questions

### Question 1

```python
print(type(10))
print(type("10"))
```

What is printed?

Answer:

```text
<class 'int'>
<class 'str'>
```

Reason:

`10` is an integer object. `"10"` is a string object.

---

### Question 2

```python
print("Py" + "thon")
print([1, 2] + [3])
```

What is printed?

Answer:

```text
Python
[1, 2, 3]
```

Reason:

The `+` operator uses type-specific behavior. Strings concatenate with strings. Lists concatenate with lists.

---

### Question 3

```python
value = 10
print(value.upper())
```

What happens?

Answer:

Python raises `AttributeError`.

Reason:

The object is an `int`, and integer objects do not have an `upper` method.

---

### Question 4

```python
a = [1, 2]
b = [1, 2]

print(a == b)
print(a is b)
```

What is printed?

Answer:

```text
True
False
```

Reason:

The two lists have equal values, but they are different objects.

---

### Question 5

```python
def greet():
    print("hello")

print(type(greet))
```

What is printed?

Answer:

```text
<class 'function'>
```

Reason:

Functions are runtime objects in Python.

---

### Question 6

```python
print(type(None))
```

What is printed?

Answer:

```text
<class 'NoneType'>
```

Reason:

`None` is an object representing absence of a meaningful value.

---

## Mental Model Questions

1. Draw an object with identity, type, and value.
2. Draw two different list objects with the same value.
3. Draw one function object referred to by two different names.
4. Draw a list object containing references to three other objects.
5. Draw the relationship between bytecode execution and objects.
6. Draw the difference between an object and its printed representation.

---

# Practical Exercises

## Exercise 1

Inspect several objects:

```python
values = [
    10,
    3.14,
    "hello",
    True,
    None,
    [1, 2],
    {"a": 1},
]

for value in values:
    print(value, type(value), id(value))
```

For each value, identify:

* Its printed representation.
* Its type.
* Its identity.

Explain why the printed representation is not the object itself.

---

## Exercise 2

Compare identity and equality:

```python
a = [1, 2]
b = [1, 2]
c = a

print(a == b)
print(a is b)
print(a is c)
```

Explain each result using identity and value.

---

## Exercise 3

Explore type-specific behavior:

```python
print(10 + 20)
print("10" + "20")
print([10] + [20])
```

Explain why the same operator produces different kinds of results.

---

## Exercise 4

Use `dir()`:

```python
text = "python"
number = 10

print("upper" in dir(text))
print("upper" in dir(number))
```

Explain why the results differ.

---

## Exercise 5

Show that functions are objects:

```python
def double(x):
    return x * 2

operation = double

print(type(operation))
print(operation(5))
```

Explain why `operation(5)` works.

---

## Exercise 6

Show that modules are objects:

```python
import math

print(type(math))
print(math.sqrt(25))
```

Explain what `math` refers to and what `sqrt` is.

---

## Exercise 7

Read an error message as object information:

```python
items = [1, 2, 3]
items.upper()
```

Run the code.

Then answer:

* What type of object is `items`?
* What attribute was Python looking for?
* Why did the error happen?

---

# Summary

In this chapter we learned:

* Python programs manipulate objects at runtime.
* Every Python object has identity, type, and value.
* `type()` tells you an object's type.
* `id()` tells you an object's identity during its lifetime.
* An object's type determines much of its behavior.
* Numbers, strings, booleans, lists, dictionaries, functions, classes, modules, exceptions, and `None` are all objects.
* Functions and classes are runtime objects, not just syntax.
* Modules are objects created or reused during imports.
* Exception values are objects carrying error information.
* Printed output is a representation of an object, not the object itself.
* Objects live in process memory while the program runs.
* CPython objects have internal structure, including type information and object-specific data.
* Containers hold references to other objects.
* Equality and identity ask different questions.
* The object model explains many Python errors and API behaviors.

The core mental model is:

```text
object
├── identity: which specific object?
├── type: what kind of object?
└── value: what data or state?
```

And the larger runtime model is:

```text
bytecode execution
    |
    v
loads, stores, creates, calls, and compares
    |
    v
Python objects
```

You now have the foundation needed to understand Python variables correctly.

---

# Preview of Chapter 10

Now that we understand objects, we can ask:

> How does Python code refer to those objects?

The answer is names and references.

In the next chapter, we will replace the misleading idea that variables are boxes.

Instead, we will build the correct Python model:

```text
name ─────▶ object
```
