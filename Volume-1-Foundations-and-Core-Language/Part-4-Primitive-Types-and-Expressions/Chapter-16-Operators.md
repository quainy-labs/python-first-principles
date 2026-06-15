# Chapter 16 — Operators

---

# Learning Objectives

By the end of this chapter, you should understand:

* What operators are.
* Why operators are syntax for operations on objects.
* How arithmetic operators work.
* How comparison operators work.
* How boolean operators work.
* How identity operators work.
* How membership operators work.
* How assignment-related operators work.
* How bitwise operators work at a practical level.
* What operator precedence means.
* What associativity means.
* Why parentheses improve clarity.
* Why operator behavior depends on object type.
* Why some operators create new objects while others mutate existing objects.
* Common mistakes with `is`, `and`, `or`, `+=`, precedence, and chained comparisons.

Operators are not just symbols.

They are compact syntax for object behavior.

---

# Concept Overview

An operator is a symbol or keyword that performs an operation.

Examples:

```python
1 + 2
name == "Ada"
value is None
"py" in "python"
not is_active
x and y
```

Operators combine objects, inspect objects, compare objects, or bind names.

From earlier chapters:

```text
expression -> object
operator -> operation involving objects
```

Example:

```python
result = 1 + 2
```

Python evaluates:

```text
int object 1
operator +
int object 2
```

and produces:

```text
int object 3
```

Then assignment binds `result` to that object.

---

# Operators Are Type-Driven

The same operator can mean different things for different types.

Example:

```python
print(1 + 2)
print("Py" + "thon")
print([1, 2] + [3])
```

Output:

```text
3
Python
[1, 2, 3]
```

The symbol `+` is the same.

The behavior depends on object types:

```text
int + int      -> numeric addition
str + str      -> string concatenation
list + list    -> list concatenation
```

This is not arbitrary.

Objects have types.

Types define supported behavior.

---

# Operators and Methods

Many operators correspond to special methods internally.

For example:

```python
a + b
```

uses addition behavior defined by the objects involved.

At a high level, Python asks:

```text
Does the left object know how to add this right object?
If not, can the right object handle the operation?
If not, raise TypeError.
```

You do not need to learn special methods yet.

Later, when studying classes, we will see methods such as:

```python
__add__
__eq__
__lt__
```

For now, understand:

> Operators are syntax that dispatches to type-defined behavior.

---

# Arithmetic Operators

Arithmetic operators work with numeric objects and sometimes other sequence-like objects.

Common arithmetic operators:

| Operator | Meaning |
| --- | --- |
| `+` | Addition or concatenation |
| `-` | Subtraction |
| `*` | Multiplication or repetition |
| `/` | True division |
| `//` | Floor division |
| `%` | Modulo |
| `**` | Exponentiation |
| `+x` | Unary plus |
| `-x` | Unary minus |

Examples:

```python
print(10 + 3)
print(10 - 3)
print(10 * 3)
print(10 / 3)
print(10 // 3)
print(10 % 3)
print(10 ** 3)
```

Output:

```text
13
7
30
3.3333333333333335
3
1
1000
```

---

# Addition

For numbers:

```python
print(2 + 3)
```

Output:

```text
5
```

For strings:

```python
print("hello" + " world")
```

Output:

```text
hello world
```

For lists:

```python
print([1, 2] + [3, 4])
```

Output:

```text
[1, 2, 3, 4]
```

Invalid mixed types raise errors:

```python
"age: " + 30
```

raises:

```text
TypeError
```

Python does not silently convert the integer to a string for `+`.

Be explicit:

```python
"age: " + str(30)
```

or:

```python
f"age: {30}"
```

---

# Subtraction

Subtraction is primarily numeric.

Example:

```python
print(10 - 4)
```

Output:

```text
6
```

Strings do not support subtraction:

```python
"hello" - "he"
```

raises:

```text
TypeError
```

If you want to remove text, use string methods:

```python
"hello".removeprefix("he")
```

or:

```python
"hello".replace("he", "", 1)
```

The operator must be supported by the type.

---

# Multiplication

For numbers:

```python
print(4 * 3)
```

Output:

```text
12
```

For strings:

```python
print("ha" * 3)
```

Output:

```text
hahaha
```

For lists:

```python
print([0] * 3)
```

Output:

```text
[0, 0, 0]
```

Be careful with nested mutable objects:

```python
matrix = [[0] * 3] * 3
matrix[0][0] = 1
print(matrix)
```

Output:

```text
[[1, 0, 0], [1, 0, 0], [1, 0, 0]]
```

The outer list repeats references to the same inner list.

---

# Division Operators

True division:

```python
print(5 / 2)
print(4 / 2)
```

Output:

```text
2.5
2.0
```

`/` returns a float for built-in integers.

Floor division:

```python
print(5 // 2)
print(-5 // 2)
```

Output:

```text
2
-3
```

`//` floors toward negative infinity.

Modulo:

```python
print(5 % 2)
```

Output:

```text
1
```

Modulo is paired with floor division:

```text
a == (a // b) * b + (a % b)
```

---

# Exponentiation

`**` computes powers.

Example:

```python
print(2 ** 10)
```

Output:

```text
1024
```

Exponentiation is right-associative:

```python
print(2 ** 3 ** 2)
```

This means:

```python
2 ** (3 ** 2)
```

not:

```python
(2 ** 3) ** 2
```

Output:

```text
512
```

Use parentheses when this could surprise a reader.

---

# Unary Operators

Unary operators operate on one operand.

Examples:

```python
x = 5

print(+x)
print(-x)
```

Output:

```text
5
-5
```

`+x` usually returns numeric positive form.

`-x` returns numeric negation.

Unary `not` is boolean negation:

```python
print(not True)
```

Output:

```text
False
```

Bitwise unary `~` is covered later in this chapter.

---

# Comparison Operators

Comparison operators produce boolean results.

| Operator | Meaning |
| --- | --- |
| `==` | Equal |
| `!=` | Not equal |
| `<` | Less than |
| `<=` | Less than or equal |
| `>` | Greater than |
| `>=` | Greater than or equal |

Examples:

```python
print(3 == 3)
print(3 != 4)
print(3 < 4)
print(3 <= 3)
print(5 > 2)
print(5 >= 5)
```

Output:

```text
True
True
True
True
True
True
```

Comparison results are `bool` objects.

---

# Equality vs Identity

Equality:

```python
a == b
```

asks:

```text
Are these values equal?
```

Identity:

```python
a is b
```

asks:

```text
Are these the same object?
```

Example:

```python
a = [1, 2]
b = [1, 2]

print(a == b)
print(a is b)
```

Output:

```text
True
False
```

Use `==` for normal value comparison.

Use `is` for identity checks such as:

```python
value is None
```

---

# Chained Comparisons

Python supports chained comparisons.

Example:

```python
x = 5
print(1 < x < 10)
```

Output:

```text
True
```

This means:

```text
1 < x and x < 10
```

but `x` is evaluated once.

Chained comparisons are useful for ranges:

```python
if 0 <= score <= 100:
    print("valid")
```

Do not overcomplicate chains.

Readable range checks are the main use case.

---

# Boolean Operators

Boolean operators:

```python
and
or
not
```

They use truthiness.

Examples:

```python
print(True and False)
print(True or False)
print(not True)
```

Output:

```text
False
True
False
```

As Chapter 15 explained:

```text
not always returns bool
and returns first falsy operand or last operand
or returns first truthy operand or last operand
```

---

# and Returns Operands

Example:

```python
print("hello" and 123)
print("" and 123)
```

Output:

```text
123

```

`x and y`:

```text
if x is falsy, return x
otherwise, return y
```

This is why:

```python
user is not None and user.is_active
```

can safely check `user.is_active` only when `user` is not `None`.

---

# or Returns Operands

Example:

```python
print("hello" or 123)
print("" or 123)
```

Output:

```text
hello
123
```

`x or y`:

```text
if x is truthy, return x
otherwise, return y
```

This is useful for fallback values:

```python
display_name = name or "Anonymous"
```

But be careful.

If `name` can be an intentionally empty string, this may be wrong.

---

# Identity Operators

Identity operators:

```python
is
is not
```

Examples:

```python
value = None

print(value is None)
print(value is not None)
```

Output:

```text
True
False
```

Use identity operators when identity matters.

Common cases:

```python
value is None
value is not None
value is sentinel
```

Do not use identity operators for numeric or string equality.

---

# Membership Operators

Membership operators:

```python
in
not in
```

Examples:

```python
print("py" in "python")
print(3 in [1, 2, 3])
print("name" in {"name": "Ada"})
```

Output:

```text
True
True
True
```

For dictionaries, membership checks keys:

```python
user = {"name": "Ada", "age": 36}

print("name" in user)
print("Ada" in user)
```

Output:

```text
True
False
```

To check values:

```python
"Ada" in user.values()
```

---

# Assignment

Assignment uses:

```python
=
```

Example:

```python
x = 10
```

Assignment is not an expression in the same way as arithmetic expressions.

It binds a name to an object.

From Chapter 10:

```text
evaluate right side
bind left-side name to resulting object
```

Example:

```python
x = 2 + 3
```

Steps:

```text
evaluate 2 + 3 -> int object 5
bind x -> int object 5
```

---

# Augmented Assignment

Augmented assignment operators:

```python
+=
-=
*=
/=
//=
%=
**=
```

Examples:

```python
x = 10
x += 5
print(x)
```

Output:

```text
15
```

For immutable numbers, this rebinds the name.

For mutable objects, augmented assignment may mutate in place.

Example:

```python
a = [1, 2]
b = a
a += [3]

print(a)
print(b)
print(a is b)
```

Output:

```text
[1, 2, 3]
[1, 2, 3]
True
```

The list was modified in place.

---

# Assignment Expressions

Python has an assignment expression operator:

```python
:=
```

It is often called the walrus operator.

Example:

```python
text = "hello"

if (length := len(text)) > 3:
    print(length)
```

Output:

```text
5
```

This both assigns and returns a value inside an expression.

Use it sparingly.

It can make code clearer when it avoids repeated work.

It can make code worse when it compresses too much logic into one line.

---

# Bitwise Operators

Bitwise operators operate on integer bits.

| Operator | Meaning |
| --- | --- |
| `&` | Bitwise AND |
| `|` | Bitwise OR |
| `^` | Bitwise XOR |
| `~` | Bitwise invert |
| `<<` | Left shift |
| `>>` | Right shift |

Examples:

```python
print(0b1100 & 0b1010)
print(0b1100 | 0b1010)
print(0b1100 ^ 0b1010)
```

Output:

```text
8
14
6
```

In binary:

```text
1100 & 1010 -> 1000
1100 | 1010 -> 1110
1100 ^ 1010 -> 0110
```

Bitwise operators are common in flags, masks, permissions, compression, cryptography, and low-level protocols.

---

# Bit Shifts

Left shift:

```python
print(1 << 3)
```

Output:

```text
8
```

This shifts bits left by three positions:

```text
1 -> 1000
```

Right shift:

```python
print(8 >> 1)
```

Output:

```text
4
```

Shifts are roughly related to multiplying or dividing by powers of two for non-negative integers.

Do not use shifts for ordinary arithmetic unless bit-level intent is clear.

---

# Bitwise Operators Are Not Boolean Operators

Do not confuse:

```python
and
or
not
```

with:

```python
&
|
~
```

Examples:

```python
print(True and False)
print(True & False)
```

Both print:

```text
False
```

But they are not the same operation.

`and` and `or` short-circuit and return operands.

`&` and `|` call bitwise/operator behavior and do not short-circuit.

This matters especially with arrays, pandas, and custom objects.

---

# Operator Precedence

Precedence determines which operators bind first.

Example:

```python
print(2 + 3 * 4)
```

Output:

```text
14
```

Multiplication has higher precedence than addition.

Use parentheses to change grouping:

```python
print((2 + 3) * 4)
```

Output:

```text
20
```

Precedence is a parsing rule.

It determines the structure of the expression before execution.

---

# Common Precedence Order

You do not need to memorize the full table immediately.

But know the common order:

```text
()
**
+x, -x, ~x
*, /, //, %
+, -
shifts
&
^
|
comparisons, is, in
not
and
or
```

Assignment is not part of ordinary expression precedence in the same way, except for assignment expressions.

Practical rule:

> If precedence is not obvious to a human reader, use parentheses.

---

# Associativity

Associativity determines how operators of the same precedence group.

Most binary operators are left-associative.

Example:

```python
print(10 - 3 - 2)
```

means:

```python
(10 - 3) - 2
```

Output:

```text
5
```

Exponentiation is right-associative:

```python
print(2 ** 3 ** 2)
```

means:

```python
2 ** (3 ** 2)
```

Output:

```text
512
```

---

# Parentheses

Parentheses are not just for changing behavior.

They are also for clarity.

Example:

```python
if (age >= 18) and (country == "IN"):
    ...
```

The parentheses are not required, but they may help readability.

Avoid excessive parentheses that add noise.

Use them when they make grouping clear.

Good code optimizes for correct human understanding, not just parser acceptance.

---

# Operator Result Types

Operators produce objects.

The result type depends on the operation and operand types.

Examples:

```python
print(type(1 + 2))
print(type(1 + 2.0))
print(type(5 / 2))
print(type(5 // 2))
print(type("a" + "b"))
print(type(3 < 5))
```

Output:

```text
<class 'int'>
<class 'float'>
<class 'float'>
<class 'int'>
<class 'str'>
<class 'bool'>
```

Do not assume an operator always returns the same type.

Look at the operator and operands together.

---

# Operators Can Raise Errors

Unsupported operations raise exceptions.

Examples:

```python
"age: " + 30
```

raises:

```text
TypeError
```

```python
10 / 0
```

raises:

```text
ZeroDivisionError
```

```python
[1, 2] < 3
```

raises:

```text
TypeError
```

Errors are not random.

They mean the requested operation is invalid for the objects involved or the runtime values provided.

---

# Operator Overloading Preview

Different types can define how operators work.

This is called operator overloading.

Example:

```python
1 + 2
```

and:

```python
"a" + "b"
```

use the same operator symbol but different type behavior.

Later, when you define classes, you can define operator behavior for your own types.

Example concept:

```python
point1 + point2
```

could add two coordinate objects if the class defines that behavior.

Operator overloading should be used only when the meaning is clear and natural.

---

# Common Mistakes

## Misconception 1

### Operators have one universal meaning.

Operators are type-driven.

`+` means addition for numbers, concatenation for strings and lists.

---

## Misconception 2

### `is` is a stronger version of `==`.

It is not.

`is` checks identity.

`==` checks equality.

They answer different questions.

---

## Misconception 3

### `and` and `or` return only booleans.

They return operands.

This matters for fallback expressions and truthiness.

---

## Misconception 4

### `&` and `|` are the same as `and` and `or`.

They are different operators.

`and` and `or` are boolean operators with short-circuiting.

`&` and `|` are bitwise/operator-overload operators and do not short-circuit.

---

## Misconception 5

### `+=` always means rebinding.

For immutable objects, it often results in rebinding.

For mutable objects, it may mutate in place.

Example:

```python
items += [1]
```

can modify the existing list.

---

## Misconception 6

### Precedence should be memorized instead of clarified.

Know the common rules.

Use parentheses when readability benefits.

---

## Misconception 7

### Assignment is just another arithmetic-like operator.

Assignment binds names.

It is part of Python's name-object model, not just value computation.

---

# Real-world Usage

## Validation

Operators express validation rules:

```python
if 0 <= score <= 100:
    print("valid")
```

Use chained comparisons for readable ranges.

---

## Defaults

`or` can provide fallback values:

```python
display_name = name or "Anonymous"
```

Use this only when all falsy values should use the fallback.

If only `None` means missing:

```python
display_name = "Anonymous" if name is None else name
```

---

## Membership

Membership checks are common:

```python
if role in allowed_roles:
    ...
```

For dictionaries:

```python
if key in data:
    ...
```

checks keys.

---

## Bit Flags

Bitwise operators can represent sets of flags.

Example:

```python
READ = 0b001
WRITE = 0b010
EXECUTE = 0b100

permission = READ | WRITE

print(bool(permission & READ))
print(bool(permission & EXECUTE))
```

Output:

```text
True
False
```

This is a compact representation, but use it only when bit flags are appropriate.

For many applications, named sets or enums are clearer.

---

## Data Libraries

In libraries such as pandas or NumPy, `&` and `|` are often used for elementwise boolean operations.

Example concept:

```python
(ages >= 18) & (countries == "IN")
```

This is not the same as Python's `and`.

Those libraries define operator behavior for array-like objects.

This reinforces the key point:

```text
operator behavior depends on object type
```

---

# Concept Connections

This chapter builds on:

```text
Chapter 13:
Numeric operators create and compare numeric objects.

Chapter 14:
String operators concatenate, compare, and test membership.

Chapter 15:
Boolean operators use truthiness and short-circuiting.

Chapter 12:
Identity and equality operators answer different questions.
```

It prepares:

```text
Expressions:
    operators combine values into expressions

Control flow:
    comparison and boolean operators control branching

Functions:
    operators can appear in arguments, returns, and predicates

Classes:
    custom objects can define operator behavior
```

Core model:

```text
operand object
    |
operator
    |
operand object
    |
    v
result object or side effect
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Operator | Syntax for an operation |
| Operand | Object an operator acts on |
| Precedence | Which operator groups first |
| Associativity | Grouping direction for same-precedence operators |
| Arithmetic operator | Numeric or sequence operation |
| Comparison operator | Produces boolean result |
| Boolean operator | Uses truthiness |
| Identity operator | Checks object sameness |
| Membership operator | Checks containment |
| Bitwise operator | Operates on integer bits or overloaded behavior |
| Augmented assignment | Assignment combined with operation |

Core rules:

```text
Operators act on objects.
Operator behavior depends on type.
Comparison operators produce bool objects.
and and or return operands.
is checks identity.
== checks equality.
in checks membership.
+= may mutate or rebind depending on type.
Use parentheses for clarity.
```

---

# Active Recall

## Easy Recall Questions

1. What is an operator?
2. What is an operand?
3. What does `+` do for integers?
4. What does `+` do for strings?
5. What does `==` check?
6. What does `is` check?
7. What does `in` check?
8. Does `and` always return a boolean?
9. What does operator precedence determine?
10. What does `+=` do for lists?

---

## Deep Understanding Questions

1. Why does the same operator behave differently for different types?
2. Why is `is` not a stronger form of `==`?
3. Why do `and` and `or` return operands instead of always returning booleans?
4. Why does `&` not replace `and` in normal Python conditions?
5. Why can `+=` mutate a list but rebind an integer?
6. Why does Python raise `TypeError` for unsupported operations?
7. Why is precedence a parsing concern?
8. Why are parentheses often better than relying on obscure precedence rules?
9. Why should operator overloading be used carefully?
10. Why does membership in a dictionary check keys?

---

## Explain In Your Own Words

1. Explain operators as syntax for object behavior.
2. Explain type-driven operator behavior using `+`.
3. Explain equality versus identity operators.
4. Explain boolean operator short-circuiting.
5. Explain augmented assignment with immutable and mutable objects.
6. Explain precedence and associativity.
7. Explain why bitwise operators are separate from boolean operators.

---

## Predict-the-Output Questions

### Question 1

```python
print(2 + 3 * 4)
print((2 + 3) * 4)
```

Answer:

```text
14
20
```

Reason:

Multiplication has higher precedence than addition. Parentheses change grouping.

---

### Question 2

```python
print("Py" + "thon")
print("ha" * 3)
```

Answer:

```text
Python
hahaha
```

Reason:

Strings define `+` as concatenation and `*` as repetition.

---

### Question 3

```python
a = [1, 2]
b = a
a += [3]

print(b)
print(a is b)
```

Answer:

```text
[1, 2, 3]
True
```

Reason:

List augmented assignment mutates the existing list in place.

---

### Question 4

```python
print("hello" or "fallback")
print("" or "fallback")
```

Answer:

```text
hello
fallback
```

Reason:

`or` returns the first truthy operand, otherwise the last operand.

---

### Question 5

```python
user = {"name": "Ada"}

print("name" in user)
print("Ada" in user)
```

Answer:

```text
True
False
```

Reason:

Dictionary membership checks keys.

---

### Question 6

```python
print(2 ** 3 ** 2)
```

Answer:

```text
512
```

Reason:

Exponentiation is right-associative: `2 ** (3 ** 2)`.

---

# Mental Model Questions

1. Draw `1 + 2` producing an `int` object.
2. Draw `"a" + "b"` producing a new `str` object.
3. Draw `a == b` versus `a is b`.
4. Draw `x and y` returning `x` when `x` is falsy.
5. Draw `items += [3]` when `items` has an alias.
6. Draw operator precedence for `2 + 3 * 4`.

---

# Practical Exercises

## Exercise 1

Compare operator behavior:

```python
print(1 + 2)
print("1" + "2")
print([1] + [2])
```

Explain how the same operator produces different behavior.

---

## Exercise 2

Test identity and equality:

```python
a = [1, 2]
b = [1, 2]
c = a

print(a == b)
print(a is b)
print(a is c)
```

Explain each result.

---

## Exercise 3

Investigate `and` and `or`:

```python
values = ["", "hello", 0, 42]

print(values[0] or values[1])
print(values[2] and values[3])
print(values[1] and values[3])
```

Explain operand-return behavior.

---

## Exercise 4

Compare augmented assignment:

```python
x = 10
y = x
x += 1

print(x)
print(y)
```

Then:

```python
a = []
b = a
a += [1]

print(a)
print(b)
```

Explain rebinding versus mutation.

---

## Exercise 5

Use membership correctly:

```python
user = {"name": "Ada", "role": "admin"}

print("role" in user)
print("admin" in user)
print("admin" in user.values())
```

Explain dictionary membership.

---

## Exercise 6

Bit flags:

```python
READ = 0b001
WRITE = 0b010
EXECUTE = 0b100

permission = READ | EXECUTE

print(bool(permission & READ))
print(bool(permission & WRITE))
print(bool(permission & EXECUTE))
```

Explain each result.

---

## Exercise 7

Add parentheses for clarity:

```python
result = a or b and c
```

Research or reason about how Python groups it.

Then rewrite it with parentheses to make the intended grouping explicit.

---

# Summary

In this chapter we learned:

* Operators are syntax for operations on objects.
* Operands are the objects operators act on.
* Operator behavior depends on object type.
* Arithmetic operators perform numeric operations and some sequence operations.
* Comparison operators produce boolean objects.
* `==` checks equality.
* `is` checks identity.
* Boolean operators use truthiness.
* `and` and `or` return operands and short-circuit.
* Membership operators check containment.
* Dictionary membership checks keys.
* Assignment binds names.
* Augmented assignment may rebind or mutate depending on object type.
* Bitwise operators operate on integer bits or overloaded behavior.
* Precedence determines grouping.
* Associativity determines grouping direction for operators of the same precedence.
* Parentheses should be used when they improve clarity.
* Unsupported operations raise exceptions.

The core model is:

```text
operand object
    |
operator
    |
operand object
    |
    v
result object
```

or, for assignment:

```text
right-side expression -> object -> bind name
```

Operators are the grammar of object interaction.

---

# Preview of Chapter 17

Next we study expressions.

Operators are pieces of expressions.

Expressions are code fragments that evaluate to objects.

Examples:

```python
2 + 3
name.upper()
age >= 18
"admin" if is_admin else "user"
```

Chapter 17 will explain:

* What expressions are
* How expressions are evaluated
* Expression result objects
* Nested expressions
* Conditional expressions
* Evaluation order
* Side effects inside expressions

This will prepare us for control flow and functions.
