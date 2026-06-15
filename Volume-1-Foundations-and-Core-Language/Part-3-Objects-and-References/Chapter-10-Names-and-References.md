# Chapter 10 — Names and References

---

# Learning Objectives

By the end of this chapter, you should understand:

* Why Python variables are better understood as names.
* Why the "variable as a box" model is misleading in Python.
* What assignment really does.
* What name binding means.
* What rebinding means.
* How one object can have multiple names.
* How names refer to objects rather than containing objects.
* How references connect names to runtime objects.
* How namespaces store name-object relationships.
* Why local names and global names live in different namespaces.
* How function arguments are passed by object reference.
* Why assignment does not copy objects by default.
* Why aliasing matters.
* How to draw basic Python memory diagrams.
* How this chapter prepares you for mutability.

This chapter replaces one of the most common beginner mental models:

```text
variable is a box containing a value
```

with the more accurate Python model:

```text
name ─────▶ object
```

---

# Concept Overview

Chapter 09 taught that Python programs manipulate objects.

Every object has:

```text
identity
type
value
```

Now we need to answer:

> How does our code refer to those objects?

The answer is names.

When you write:

```python
x = 10
```

Python does not create a box named `x` and put `10` inside it.

Instead, Python ensures there is an integer object representing `10`, then binds the name `x` to that object.

The correct mental model is:

```text
x ─────▶ int object 10
```

The name is how your program reaches the object.

The object is the runtime value.

This distinction is the foundation for understanding assignment, function arguments, mutation, equality, memory diagrams, and many surprising Python behaviors.

---

# The Core Mental Model

Python names are labels that refer to objects.

```text
name ─────▶ object
```

Example:

```python
age = 30
```

Mental model:

```text
age ─────▶ int object
             ├── type: int
             └── value: 30
```

Example:

```python
message = "hello"
```

Mental model:

```text
message ─────▶ str object
                ├── type: str
                └── value: "hello"
```

Example:

```python
items = [1, 2, 3]
```

Mental model:

```text
items ─────▶ list object
              ├── type: list
              └── value: [1, 2, 3]
```

The name is not the object.

The name refers to the object.

---

# Why the Box Model Fails

Many beginners imagine:

```text
+-----+
|  x  |
| 10  |
+-----+
```

This suggests that `x` is a container and `10` is inside it.

That model seems harmless for simple code:

```python
x = 10
print(x)
```

But it breaks down quickly.

Consider:

```python
a = [1, 2]
b = a
```

The box model may suggest:

```text
a box contains [1, 2]
b box contains a copy of [1, 2]
```

That is not what Python does.

The better model is:

```text
a ─┐
   ├────▶ list object [1, 2]
b ─┘
```

Both names refer to the same object.

This matters enormously once objects can change.

We will study mutation in Chapter 11, but the name-reference model must come first.

---

# What Assignment Does

Assignment binds a name to an object.

Example:

```python
x = 10
```

This means:

```text
bind name x to int object 10
```

It does not mean:

```text
put 10 inside x
```

Assignment has two sides:

```python
name = expression
```

The right side is evaluated first.

Then the name on the left is bound to the resulting object.

Example:

```python
x = 2 + 3
```

Step by step:

```text
1. Evaluate expression 2 + 3.
2. Result is int object 5.
3. Bind name x to that object.
```

Mental model:

```text
x ─────▶ int object 5
```

---

# The Right Side Runs First

This rule is important.

In assignment:

```python
x = expression
```

Python evaluates the expression first.

Only after that does it bind the name.

Example:

```python
x = 10
x = x + 1
print(x)
```

Step by step:

```text
Initial:
x ─────▶ int object 10

Evaluate right side:
x + 1 -> 11

Rebind x:
x ─────▶ int object 11
```

Output:

```text
11
```

The old integer object `10` was not changed into `11`.

The name `x` was rebound to a different integer object.

This distinction becomes crucial for immutable objects.

---

# Binding

Binding means associating a name with an object.

Example:

```python
language = "Python"
```

This binds the name `language` to a string object.

Mental model:

```text
language ─────▶ str object "Python"
```

The name exists in a namespace.

The object exists in memory.

The binding connects them.

```text
namespace:
    language ─────▶ str object "Python"
```

The namespace is like a mapping:

```text
name -> object
```

We will explain namespaces more deeply later in this chapter.

---

# Rebinding

Rebinding means making an existing name refer to a different object.

Example:

```python
x = 10
x = "hello"
```

After the first assignment:

```text
x ─────▶ int object 10
```

After the second assignment:

```text
x ─────▶ str object "hello"
```

The name `x` did not have a permanent type.

At one moment, it referred to an integer object.

Later, it referred to a string object.

This is why Python is dynamically typed.

Objects have types.

Names do not have fixed declared types in ordinary Python code.

---

# Names Do Not Have Types; Objects Do

This point is essential.

In Python:

```python
x = 10
x = "hello"
x = [1, 2, 3]
```

This is valid.

Why?

Because `x` is a name.

The objects have types:

```text
10          -> int object
"hello"    -> str object
[1, 2, 3]  -> list object
```

The name `x` can be rebound to different objects.

That does not mean Python has no types.

Python is strongly typed at the object level.

It means names are dynamically bound.

Better phrasing:

```text
Python names do not have fixed declared types.
Python objects always have types.
```

---

# Multiple Names Can Refer to One Object

Example:

```python
a = [1, 2, 3]
b = a
```

After the first assignment:

```text
a ─────▶ list object [1, 2, 3]
```

After the second assignment:

```text
a ─┐
   ├────▶ list object [1, 2, 3]
b ─┘
```

The assignment:

```python
b = a
```

does not copy the list.

It binds `b` to the same object that `a` currently refers to.

This is called aliasing.

An alias is another name for the same object.

---

# Aliasing

Aliasing means one object is reachable through multiple names.

Example:

```python
primary = [1, 2, 3]
alias = primary
```

Mental model:

```text
primary ─┐
         ├────▶ list object [1, 2, 3]
alias   ─┘
```

Both names are equally valid ways to reach the same object.

There is no "real" name and "fake" name.

There is just one object and multiple references to it.

This matters because if the object changes, all aliases still point to that same object.

We will explore that fully in Chapter 11.

For now, understand:

> Assignment creates a new binding. It does not copy objects by default.

---

# Assignment Does Not Copy by Default

Consider:

```python
a = [1, 2]
b = a
```

This does not create two lists.

It creates one list and two names referring to it.

```text
a ─┐
   ├────▶ list object [1, 2]
b ─┘
```

If you want a separate list, you must explicitly create one.

Example:

```python
a = [1, 2]
b = list(a)
```

or:

```python
b = a.copy()
```

Now the model is:

```text
a ─────▶ list object [1, 2]

b ─────▶ different list object [1, 2]
```

The values are equal.

The identities are different.

---

# Checking Aliases With is

The `is` operator checks identity.

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

Both names refer to the same object.

Now:

```python
a = [1, 2]
b = [1, 2]

print(a is b)
print(a == b)
```

Output:

```text
False
True
```

The objects have the same value.

They are not the same object.

This chapter uses `is` to reveal name-object relationships.

In normal value comparisons, use `==`.

---

# References

A reference is the connection that allows Python to access an object.

When we draw:

```text
name ─────▶ object
```

the arrow represents a reference.

You do not normally manipulate references directly in Python.

Python manages them for you.

But the concept explains behavior.

Example:

```python
x = "hello"
```

Mental model:

```text
x ─────▶ str object "hello"
```

The name `x` references the string object.

Example:

```python
y = x
```

Mental model:

```text
x ─┐
   ├────▶ str object "hello"
y ─┘
```

The name `y` now references the same object.

---

# References Are Not C Pointers

If you know C or C++, you may be tempted to think of Python references as pointers.

That analogy is useful only up to a point.

Python references are managed by Python.

You cannot do pointer arithmetic.

You cannot manually free normal Python objects.

You cannot ask Python names to point to arbitrary memory addresses.

So use this mental model:

```text
reference:
    a managed connection from a name or container slot to an object
```

This gives the intuition without importing unsafe low-level assumptions.

---

# Namespaces

A namespace is a mapping from names to objects.

Example:

```python
x = 10
message = "hello"
```

Conceptually:

```text
namespace
├── x ───────▶ int object 10
└── message ─▶ str object "hello"
```

Names do not float freely.

They live in namespaces.

Different namespaces can contain the same name referring to different objects.

This is the foundation of scope.

---

# The Global Namespace

At the top level of a module, names live in the module's global namespace.

Example:

```python
x = 10

def show():
    print(x)
```

The name `x` is global to this module.

Conceptually:

```text
module global namespace
└── x ─────▶ int object 10
```

When `show()` uses `x`, Python can look for the name in the appropriate namespaces.

We will study precise scope rules later.

For now, understand:

> A global name is a name bound in a module's top-level namespace.

---

# Local Namespaces

Function calls create local namespaces.

Example:

```python
def greet(name):
    message = "Hello, " + name
    print(message)

greet("Ada")
```

When `greet("Ada")` runs, Python creates a function frame.

That frame has local names:

```text
greet frame local namespace
├── name ─────▶ str object "Ada"
└── message ──▶ str object "Hello, Ada"
```

After the function returns, that local namespace is no longer active.

This is why `message` is not normally available outside the function.

---

# Same Name in Different Namespaces

Different namespaces can use the same name.

Example:

```python
x = "global"

def show():
    x = "local"
    print(x)

show()
print(x)
```

Output:

```text
local
global
```

Mental model:

```text
global namespace
└── x ─────▶ str object "global"

show local namespace
└── x ─────▶ str object "local"
```

The same spelling `x` appears in two namespaces.

They are different bindings.

This is why understanding namespaces prevents confusion.

---

# Name Lookup

When Python sees a name in an expression, it must find the object that name refers to.

Example:

```python
print(x)
```

Python must resolve:

```text
print -> which object?
x     -> which object?
```

Name lookup is the process of finding the object associated with a name.

At a high level, Python searches appropriate namespaces depending on where the code is running.

Inside a function, Python looks in local and surrounding/global/built-in places according to scope rules.

We will study the exact LEGB rule later.

For now:

> Name lookup turns a name in code into an object at runtime.

---

# NameError

If Python cannot find a name, it raises `NameError`.

Example:

```python
print(username)
```

If `username` has not been bound, Python raises:

```text
NameError
```

This is not a syntax problem.

The syntax is valid.

The failure happens during name lookup at runtime.

Mental model:

```text
look up username
    |
    v
no binding found
    |
    v
NameError
```

This connects back to Chapter 07's distinction between syntax errors and runtime errors.

---

# Built-in Names

Names such as `print`, `len`, `type`, and `id` are built-in names.

Example:

```python
print(len("hello"))
```

Python resolves:

```text
print -> built-in function object
len   -> built-in function object
```

These names are available because Python provides a built-in namespace.

You can bind your own name called `len`, but it is usually a bad idea:

```python
len = 10
print(len("hello"))
```

This fails because `len` now refers to an integer object, not the built-in function.

The name has been rebound.

The built-in function still exists, but your local/global name is hiding it.

---

# Names Are Resolved at Runtime

Consider:

```python
def show():
    print(message)

message = "hello"
show()
```

Output:

```text
hello
```

The function body refers to `message`.

The name is resolved when the function runs, not when the function is first written in the source file.

Now:

```python
def show():
    print(message)

show()
message = "hello"
```

This raises `NameError`.

At the time `show()` runs, `message` is not yet bound.

This demonstrates that names are runtime bindings.

---

# Assignment and Bytecode

Chapter 08 showed that assignment becomes bytecode operations.

Example:

```python
x = 10
```

Conceptually:

```text
LOAD_CONST 10
STORE_NAME x
```

This means:

```text
load object 10
bind/store it under name x
```

Example:

```python
y = x
```

Conceptually:

```text
LOAD_NAME x
STORE_NAME y
```

This means:

```text
find the object referenced by x
bind name y to that same object
```

Again, no automatic copy.

---

# Rebinding Does Not Change Other Names

Consider:

```python
a = 10
b = a
a = 20

print(b)
```

Output:

```text
10
```

Step by step:

```text
After a = 10:
a ─────▶ int object 10

After b = a:
a ─┐
   ├────▶ int object 10
b ─┘

After a = 20:
a ─────▶ int object 20

b ─────▶ int object 10
```

Rebinding `a` changes what `a` refers to.

It does not change what `b` refers to.

This is a crucial rule.

---

# Rebinding vs Mutation Preview

This chapter focuses on rebinding.

Mutation is the subject of Chapter 11.

But we need a preview.

Rebinding changes a name-object connection:

```python
x = [1, 2]
x = [3, 4]
```

Mental model:

```text
before:
x ─────▶ list object [1, 2]

after:
x ─────▶ different list object [3, 4]
```

Mutation changes an object:

```python
x = [1, 2]
x.append(3)
```

Mental model:

```text
x ─────▶ same list object, now [1, 2, 3]
```

The name still points to the same object.

The object changed.

Chapter 11 will explore this deeply.

---

# Augmented Assignment Preview

Operators such as `+=` can be subtle.

Example:

```python
x = 10
x += 1
```

For integers, this behaves like rebinding:

```text
x points to int object 10
x is rebound to int object 11
```

But for some mutable objects:

```python
items = [1, 2]
items += [3]
```

the object may be modified in place.

This depends on object type and operation behavior.

Do not worry about all details yet.

The important distinction is:

```text
rebinding:
    name points to a different object

mutation:
    same object changes internally
```

Chapter 11 will make this precise.

---

# Function Arguments

Function arguments are another place where the box model fails.

Example:

```python
def show(value):
    print(value)

x = 10
show(x)
```

When `show(x)` is called, Python does not copy the object by default.

Instead:

```text
1. Evaluate x.
2. Find the object x refers to.
3. Create a local name value inside the function frame.
4. Bind value to the same object.
```

Mental model during the function call:

```text
global namespace:
x ───────▶ int object 10
             ▲
             │
show local namespace:
value ───────┘
```

Both names refer to the same object during the call.

---

# Argument Passing Is Object Reference Passing

Python's argument passing is often described imprecisely.

You may hear:

```text
Python passes by value.
Python passes by reference.
```

Both phrases can mislead if imported from other languages.

A clearer Python-specific description:

> Python passes object references by assignment.

Calling a function binds parameter names to argument objects.

Example:

```python
def describe(obj):
    print(type(obj))

items = [1, 2, 3]
describe(items)
```

During the call:

```text
items ─┐
       ├────▶ list object [1, 2, 3]
obj ───┘
```

The parameter `obj` is a local name.

It refers to the same object as `items`.

---

# Rebinding a Parameter

If a function rebinds a parameter, it does not rebind the caller's name.

Example:

```python
def change(value):
    value = 99

x = 10
change(x)
print(x)
```

Output:

```text
10
```

Why?

During the call:

```text
x ───────▶ int object 10
             ▲
             │
value ───────┘
```

Inside the function:

```python
value = 99
```

rebinds the local name `value`:

```text
x ─────▶ int object 10

value ─▶ int object 99
```

The caller's name `x` is unchanged.

---

# Mutating Through a Parameter Preview

Now compare:

```python
def add_item(values):
    values.append("new")

items = []
add_item(items)
print(items)
```

Output:

```text
['new']
```

Why?

During the call:

```text
items ─┐
       ├────▶ list object []
values ─┘
```

The function does not rebind `items`.

It uses `values` to reach the same list object and asks that object to append an item.

The object changes.

Chapter 11 will explain mutation in detail.

For now, see why the name-reference model is necessary.

---

# Containers Store References

Lists, tuples, dictionaries, and sets contain references to objects.

Example:

```python
items = ["a", "b", "c"]
```

Mental model:

```text
items ─────▶ list object
              ├──▶ str object "a"
              ├──▶ str object "b"
              └──▶ str object "c"
```

The list does not contain raw text characters as independent boxes.

It stores references to string objects.

Example:

```python
a = []
b = []
items = [a, b]
```

Mental model:

```text
items ─────▶ list object
              ├──▶ list object []
              └──▶ list object []
```

Containers create object graphs.

Memory diagrams help us reason about those graphs.

---

# Nested References

Example:

```python
row1 = [1, 2]
row2 = [3, 4]
matrix = [row1, row2]
```

Mental model:

```text
row1 ─────▶ list object [1, 2]
             ▲
             │
matrix ─────▶ outer list object
             │
             ▼
row2 ─────▶ list object [3, 4]
```

More clearly:

```text
matrix ─────▶ outer list
              ├──▶ same list object as row1
              └──▶ same list object as row2
```

The outer list refers to inner list objects.

The names `row1` and `row2` also refer to those same inner list objects.

This structure matters later when mutation enters the picture.

---

# Unreachable Objects

If no name or container can reach an object, the object is no longer useful to the program.

Example:

```python
x = [1, 2, 3]
x = "done"
```

After the first assignment:

```text
x ─────▶ list object [1, 2, 3]
```

After the second assignment:

```text
x ─────▶ str object "done"
```

If no other reference points to the list object, it becomes unreachable.

Python can eventually reclaim its memory.

This connects to later chapters on reference counting and garbage collection.

For now:

> Objects live as long as they are reachable according to Python's memory management.

---

# del Removes a Binding

The statement `del` removes a binding.

Example:

```python
x = [1, 2, 3]
del x
```

After `del x`, the name `x` is no longer bound in that namespace.

If you try:

```python
print(x)
```

Python raises:

```text
NameError
```

Important:

`del x` does not necessarily destroy the object immediately.

It removes one reference.

If other names still refer to the object, the object remains reachable.

Example:

```python
a = [1, 2]
b = a
del a
print(b)
```

Output:

```text
[1, 2]
```

The object still exists because `b` refers to it.

---

# Names Are Not Objects in the Same Sense

This is subtle.

The string `"x"` can be an object.

But the variable name `x` in code is not itself the runtime value.

Example:

```python
x = 10
```

Here:

```text
x     -> name in a namespace
10    -> int object
```

If you write:

```python
"x"
```

that is a string object containing the character `x`.

Do not confuse:

```text
x      name
"x"    string object
```

This distinction becomes useful when studying dictionaries, globals, locals, and reflection.

---

# locals() and globals()

Python lets you inspect namespaces.

At module level:

```python
x = 10
print(globals()["x"])
```

Output:

```text
10
```

`globals()` returns a dictionary representing the module's global namespace.

Inside a function:

```python
def show():
    message = "hello"
    print(locals())

show()
```

You may see:

```text
{'message': 'hello'}
```

`locals()` gives a view of local names.

Do not rely on modifying `locals()` to change local variables.

For now, use these tools for observation.

They reveal the idea:

```text
namespace = mapping from names to objects
```

---

# Object Identity Through Names

Names let us reach objects, but identity belongs to the object.

Example:

```python
a = [1, 2]
b = a

print(id(a))
print(id(b))
```

The two `id()` values are the same.

Why?

Because `a` and `b` refer to the same object.

Now:

```python
a = [1, 2]
b = [1, 2]

print(id(a))
print(id(b))
```

The two identities are different.

Why?

Because two separate list objects were created.

The names are different, but the deeper issue is object identity.

---

# Diagramming Rules

When drawing Python memory diagrams, follow these rules:

1. Draw names separately from objects.
2. Draw arrows from names to objects.
3. Draw object type and value inside the object.
4. If two names point to the same object, draw two arrows to one object.
5. If two objects have equal values but different identities, draw two separate objects.
6. For containers, draw arrows from container slots to contained objects.

Example:

```python
a = [1, 2]
b = a
c = [1, 2]
```

Diagram:

```text
a ─┐
   ├────▶ list object #1 [1, 2]
b ─┘

c ─────▶ list object #2 [1, 2]
```

This diagram captures:

```text
a is b      -> True
a is c      -> False
a == c      -> True
```

---

# Example: Step-by-Step Diagram

Code:

```python
x = "hello"
y = x
x = "world"
```

After:

```python
x = "hello"
```

diagram:

```text
x ─────▶ str object "hello"
```

After:

```python
y = x
```

diagram:

```text
x ─┐
   ├────▶ str object "hello"
y ─┘
```

After:

```python
x = "world"
```

diagram:

```text
x ─────▶ str object "world"

y ─────▶ str object "hello"
```

Rebinding `x` did not affect `y`.

---

# Example: Swapping Names

Python can swap name bindings:

```python
a = "left"
b = "right"

a, b = b, a
```

Before swap:

```text
a ─────▶ str object "left"
b ─────▶ str object "right"
```

The right side is evaluated first:

```text
(b, a) -> references to "right" and "left"
```

Then assignments happen:

```text
a ─────▶ str object "right"
b ─────▶ str object "left"
```

No string contents are changed.

The names are rebound.

---

# Example: Unpacking Assignment

Assignment can bind multiple names.

Example:

```python
x, y = 10, 20
```

Conceptually:

```text
x ─────▶ int object 10
y ─────▶ int object 20
```

Example:

```python
first, second = ["a", "b"]
```

Conceptually:

```text
first  ─────▶ str object "a"
second ─────▶ str object "b"
```

Unpacking still follows the same rule:

> Evaluate objects, then bind names.

---

# Example: Chained Assignment

Python allows chained assignment:

```python
a = b = []
```

This creates one list object and binds both names to it.

Diagram:

```text
a ─┐
   ├────▶ list object []
b ─┘
```

This is very different from:

```python
a = []
b = []
```

Diagram:

```text
a ─────▶ list object []

b ─────▶ different list object []
```

Chained assignment is safe for immutable values in many simple cases:

```python
a = b = 0
```

But it can be surprising with mutable objects.

Chapter 11 will show why.

---

# Example: Attribute Names Preview

Objects can also have attributes.

Example:

```python
user.name = "Ada"
```

This is not the same as assigning a local variable named `name`.

It binds or sets an attribute on the object referenced by `user`.

We will study attributes deeply in the object-oriented programming chapters.

For now, distinguish:

```python
name = "Ada"
```

from:

```python
user.name = "Ada"
```

The first binds a name in a namespace.

The second changes an attribute relationship on an object.

---

# Example: Subscript Assignment Preview

Assignment can target a container location:

```python
items[0] = "changed"
```

This is not rebinding the name `items`.

It modifies the object that `items` refers to.

Again, this belongs mainly to Chapter 11.

But the name-reference model helps:

```text
items ─────▶ list object
```

The name points to the list.

The subscript assignment changes part of the list object.

---

# Common Misconceptions

## Misconception 1

### Variables are boxes that contain values.

In Python, names refer to objects.

The object lives in memory.

The name is a way to reach it.

The better model is:

```text
name ─────▶ object
```

---

## Misconception 2

### Assignment copies objects.

Assignment binds a name to an object.

It does not copy objects by default.

Example:

```python
b = a
```

means:

```text
bind b to the same object a currently refers to
```

---

## Misconception 3

### Rebinding one name changes all names that used to point to the same object.

Rebinding changes only that name's binding.

Example:

```python
a = 10
b = a
a = 20
```

`b` still refers to `10`.

---

## Misconception 4

### Python names have fixed types.

Objects have types.

Names can be rebound to different objects.

This is why:

```python
x = 10
x = "hello"
```

is valid.

---

## Misconception 5

### Function arguments are copied into parameters.

Function calls bind parameter names to argument objects.

Objects are not copied by default.

---

## Misconception 6

### `del x` always destroys the object.

`del x` removes the binding for the name `x`.

If other references still reach the object, the object continues to exist.

---

## Misconception 7

### A name and a string containing that name are the same thing.

`x` is a name.

`"x"` is a string object.

They are different concepts.

---

# Real-world Usage

## Debugging Aliasing Bugs

Many confusing bugs come from multiple names referring to the same object.

Example:

```python
settings = {"debug": True}
config = settings
```

Now `settings` and `config` refer to the same dictionary.

If code changes the dictionary through one name, the other name sees the same object.

When debugging, ask:

```text
Do I have two objects?
Or two names for one object?
```

---

## Understanding Function Arguments

Function calls become clearer with name binding.

Example:

```python
def process(data):
    ...

records = [...]
process(records)
```

The parameter `data` refers to the same object as `records` during the call.

This explains why functions can observe and sometimes modify objects passed to them.

---

## Avoiding Accidental Built-in Shadowing

If you write:

```python
list = [1, 2, 3]
```

you bind the name `list` to a list object.

Now the built-in `list` type is shadowed in that namespace.

Later:

```python
list("abc")
```

will fail because `list` no longer refers to the built-in class.

Understanding names helps avoid this.

---

## Reading Tracebacks

Errors such as `NameError` are about name lookup.

Example:

```text
NameError: name 'user' is not defined
```

This means Python looked for a binding named `user` and did not find one in the relevant namespaces.

It does not mean the source code spelling is invalid.

It means runtime lookup failed.

---

## Designing APIs

When writing functions, know whether your function:

* Rebinds local names only.
* Reads from objects.
* Mutates objects.
* Returns new objects.

This clarity makes APIs easier to reason about.

Example:

```python
def sorted_copy(items):
    return sorted(items)
```

This returns a new object.

Example:

```python
def sort_in_place(items):
    items.sort()
```

This mutates the object passed in.

The difference matters because callers may have aliases to the same object.

---

# Concept Connections

This chapter connects directly to previous chapters:

```text
Chapter 02:
Objects live in memory.

Chapter 03:
Function calls create frames.

Chapter 08:
Bytecode loads and stores names.

Chapter 09:
Runtime values are objects with identity, type, and value.

Chapter 10:
Names refer to those objects.
```

It prepares the next chapters:

```text
Chapter 11:
If multiple names refer to one mutable object, mutation through one name is visible through the others.

Chapter 12:
Identity and equality become easier to reason about with diagrams.

Later:
Scope, closures, functions, modules, classes, and imports all depend on namespaces and bindings.
```

The larger mental model is:

```text
source code name
    |
    v
runtime name lookup
    |
    v
namespace binding
    |
    v
object in memory
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Object | Runtime value with identity, type, and value |
| Name | Identifier used by Python code |
| Binding | Association between a name and an object |
| Rebinding | Changing a name to refer to a different object |
| Reference | Managed connection to an object |
| Namespace | Mapping from names to objects |
| Alias | Another name/reference to the same object |
| Name lookup | Runtime process of resolving a name to an object |

Core diagram:

```text
name ─────▶ object
```

Multiple names:

```text
a ─┐
   ├────▶ object
b ─┘
```

Different objects with equal values:

```text
a ─────▶ list object [1, 2]

b ─────▶ different list object [1, 2]
```

Function call:

```text
caller name ─┐
             ├────▶ argument object
parameter ───┘
```

---

# Active Recall

## Easy Recall Questions

1. What does assignment do in Python?
2. What is a name?
3. What is a binding?
4. What is rebinding?
5. What is a namespace?
6. What is an alias?
7. Does `b = a` copy the object referred to by `a`?
8. What does `is` check?
9. What does `del x` remove?
10. Do names or objects have types?

---

## Deep Understanding Questions

1. Why is the variable-as-box model misleading in Python?
2. Why does rebinding one name not affect another name?
3. Why can two names refer to the same object?
4. Why does assignment not copy objects by default?
5. Why does `NameError` happen at runtime?
6. Why can a local variable and a global variable have the same name without being the same binding?
7. Why is "pass by reference" an incomplete description of Python argument passing?
8. Why does `del a` not necessarily destroy the object `a` referred to?
9. Why can shadowing built-in names cause confusing errors?
10. Why must this chapter come before mutability?

---

## Explain In Your Own Words

1. Explain assignment using the phrase "binds a name to an object."
2. Explain rebinding with a diagram.
3. Explain aliasing using two names and one object.
4. Explain why `a = b` does not copy by default.
5. Explain what happens when a function parameter receives an argument.
6. Explain the difference between a name and a string containing that name.
7. Explain what a namespace is.

---

## Predict-the-Output Questions

### Question 1

```python
a = 10
b = a
a = 20
print(b)
```

What is printed?

Answer:

```text
10
```

Reason:

`b` was bound to the integer object `10`. Rebinding `a` to `20` does not change `b`.

---

### Question 2

```python
a = [1, 2]
b = a

print(a is b)
```

What is printed?

Answer:

```text
True
```

Reason:

`b = a` binds `b` to the same list object that `a` refers to.

---

### Question 3

```python
a = [1, 2]
b = [1, 2]

print(a is b)
print(a == b)
```

What is printed?

Answer:

```text
False
True
```

Reason:

There are two different list objects with equal values.

---

### Question 4

```python
def change(value):
    value = 99

x = 10
change(x)
print(x)
```

What is printed?

Answer:

```text
10
```

Reason:

The function rebinds its local parameter name `value`. It does not rebind the caller's name `x`.

---

### Question 5

```python
x = "global"

def show():
    x = "local"
    print(x)

show()
print(x)
```

What is printed?

Answer:

```text
local
global
```

Reason:

The local `x` inside `show` and the global `x` are different bindings in different namespaces.

---

### Question 6

```python
len = 5
print(len("hello"))
```

What happens?

Answer:

Python raises `TypeError`.

Reason:

The name `len` has been rebound to an integer object, hiding the built-in `len` function in that namespace.

---

## Mental Model Questions

1. Draw `x = 10`.
2. Draw `a = [1, 2]; b = a`.
3. Draw `a = [1, 2]; b = [1, 2]`.
4. Draw `x = "hello"; y = x; x = "world"`.
5. Draw a function call where parameter `value` refers to the same object as caller name `x`.
6. Draw global and local namespaces that both contain a name `x`.
7. Draw `a = b = []` compared with `a = []; b = []`.

---

# Practical Exercises

## Exercise 1

Draw the memory diagram:

```python
x = 10
y = x
x = 20
```

Answer:

After all three lines:

```text
x ─────▶ int object 20
y ─────▶ int object 10
```

Explain why `y` did not change.

---

## Exercise 2

Compare aliasing and copying:

```python
a = [1, 2]
b = a
c = list(a)

print(a is b)
print(a is c)
print(a == c)
```

Explain each result.

---

## Exercise 3

Inspect namespaces:

```python
x = 10

def show():
    y = 20
    print("locals:", locals())

show()
print("x in globals:", "x" in globals())
```

Explain what `locals()` and `globals()` reveal.

---

## Exercise 4

Analyze function argument binding:

```python
def show(value):
    print(id(value))

x = [1, 2, 3]
print(id(x))
show(x)
```

Explain why the printed identities match.

---

## Exercise 5

Analyze parameter rebinding:

```python
def reset(items):
    items = []
    print("inside:", items)

values = [1, 2, 3]
reset(values)
print("outside:", values)
```

Predict the output.

Then explain it using local rebinding.

---

## Exercise 6

Compare chained assignment:

```python
a = b = []
c = []
d = []

print(a is b)
print(c is d)
```

Explain why the first result is `True` and the second is `False`.

---

## Exercise 7

Read a `NameError`:

```python
def show():
    print(message)

show()
message = "hello"
```

Run the code.

Explain why the error happens even though `message` appears later in the file.

---

# Summary

In this chapter we learned:

* Python names refer to objects.
* Assignment binds names to objects.
* The right side of assignment is evaluated before the binding happens.
* Rebinding changes which object a name refers to.
* Rebinding one name does not change other names.
* Objects have types; names can be rebound to objects of different types.
* Multiple names can refer to the same object.
* Assignment does not copy objects by default.
* Aliasing means one object is reachable through multiple names or references.
* Names live in namespaces.
* Global and local namespaces can contain the same name with different bindings.
* Name lookup resolves names to objects at runtime.
* `NameError` happens when lookup fails.
* Function calls bind parameter names to argument objects.
* Rebinding a parameter does not rebind the caller's name.
* Containers hold references to objects.
* `del` removes a binding, not necessarily the object itself.
* Memory diagrams should draw names, objects, and arrows separately.

The core mental model is:

```text
name ─────▶ object
```

And the most important rule is:

```text
Assignment binds or rebinds names.
It does not copy objects by default.
```

You now understand how Python code reaches objects.

---

# Preview of Chapter 11

Now we know:

```text
names refer to objects
```

The next question is:

> What happens when the object itself can change?

That is mutability.

In the next chapter, we will study:

* Mutable objects
* Immutable objects
* In-place change
* Rebinding versus mutation
* Aliasing bugs
* Why lists behave differently from integers and strings

This is where the name-reference model becomes essential.
