# Chapter 17 — Expressions

---

# Learning Objectives

By the end of this chapter, you should understand:

* What an expression is.
* Why expressions evaluate to objects.
* How literals, names, operators, function calls, attribute access, indexing, and slicing can be expressions.
* How nested expressions are evaluated.
* What evaluation order means.
* How operator precedence shapes expressions.
* How expression result objects connect to assignment.
* The difference between expressions and statements.
* What conditional expressions are.
* What side effects inside expressions are.
* Why readable expressions matter.
* Common mistakes with over-complex expressions, assignment, truthiness, and side effects.

Expressions are the building blocks of computation.

They are how Python code produces values.

---

# Concept Overview

An expression is code that evaluates to an object.

Examples:

```python
10
"hello"
x
x + 1
name.upper()
items[0]
text[1:4]
age >= 18
"admin" if is_admin else "user"
```

Each expression produces a result object.

Examples:

```python
2 + 3
```

evaluates to:

```text
int object 5
```

```python
"py" + "thon"
```

evaluates to:

```text
str object "python"
```

```python
age >= 18
```

evaluates to:

```text
bool object True or False
```

Core model:

```text
expression
    |
    v
object
```

---

# Why Expressions Matter

Python programs do work by evaluating expressions and executing statements.

Expressions produce objects.

Statements perform actions.

Example:

```python
x = 2 + 3
```

The expression is:

```python
2 + 3
```

It evaluates to:

```text
int object 5
```

The assignment statement binds:

```text
x ─────▶ int object 5
```

Without expressions, there are no values to assign, compare, pass, return, or print.

---

# Expressions Produce Objects

Every expression produces an object.

Example:

```python
result = 10 * 2
```

The expression:

```python
10 * 2
```

produces:

```text
int object 20
```

Example:

```python
message = "hello".upper()
```

The expression:

```python
"hello".upper()
```

produces:

```text
str object "HELLO"
```

Example:

```python
allowed = age >= 18
```

The expression:

```python
age >= 18
```

produces:

```text
bool object
```

This is the link between syntax and runtime objects.

---

# Literal Expressions

A literal is an expression that directly represents a value.

Examples:

```python
10
3.14
"hello"
True
None
[1, 2, 3]
{"name": "Ada"}
```

Each literal evaluates to an object.

Examples:

```python
10
```

evaluates to an `int` object.

```python
"hello"
```

evaluates to a `str` object.

```python
[1, 2, 3]
```

evaluates to a `list` object.

Container literals may evaluate inner expressions too.

Example:

```python
[1, 2 + 3, "a".upper()]
```

evaluates inner expressions before constructing the list.

---

# Name Expressions

A name can be an expression.

Example:

```python
x
```

When Python evaluates a name expression, it performs name lookup.

Example:

```python
x = 10
print(x)
```

Inside `print(x)`, the expression `x` is evaluated.

Python looks up the name `x` and finds the object it refers to.

If the name is not found:

```python
print(missing)
```

Python raises:

```text
NameError
```

Name expressions connect Chapter 10 to expression evaluation.

---

# Operator Expressions

Operators combine expressions.

Example:

```python
2 + 3
```

The operands are expressions:

```python
2
3
```

The operator is:

```python
+
```

Evaluation:

```text
evaluate 2 -> int object 2
evaluate 3 -> int object 3
apply + -> int object 5
```

Example:

```python
age >= 18
```

Evaluation:

```text
evaluate age -> object
evaluate 18 -> int object 18
apply >= -> bool object
```

Operators are expression builders.

---

# Function Call Expressions

A function call is an expression.

Example:

```python
len("hello")
```

evaluates to:

```text
int object 5
```

Example:

```python
print("hello")
```

is also a function call expression.

It prints text as a side effect and returns:

```text
None
```

This surprises many learners.

Try:

```python
result = print("hello")
print(result)
```

Output:

```text
hello
None
```

The call expression produced `None`.

The printing was a side effect.

---

# Attribute Access Expressions

Attribute access is an expression.

Example:

```python
text.upper
```

This evaluates to a method object associated with `text`.

Calling it:

```python
text.upper()
```

evaluates to a new string.

Example:

```python
import math

math.pi
```

evaluates to the object stored as the `pi` attribute of the `math` module.

Attribute access asks:

```text
Get this named attribute from this object.
```

If the attribute does not exist, Python raises:

```text
AttributeError
```

---

# Indexing Expressions

Indexing is an expression.

Example:

```python
items[0]
```

If:

```python
items = ["a", "b", "c"]
```

then:

```python
items[0]
```

evaluates to:

```text
str object "a"
```

Indexing involves two expressions:

```python
items[index]
```

Python evaluates:

```text
items
index
```

then asks the object to provide the item at that index.

---

# Slicing Expressions

Slicing is an expression.

Example:

```python
text[1:4]
```

If:

```python
text = "Python"
```

then:

```python
text[1:4]
```

evaluates to:

```text
str object "yth"
```

For lists:

```python
items = [1, 2, 3, 4]
part = items[1:3]
```

`part` refers to a new list:

```text
[2, 3]
```

Slicing produces objects.

The exact type depends on the sliced object.

---

# Container Display Expressions

Container literals are expressions.

List:

```python
[1, 2, 3]
```

Tuple:

```python
(1, 2, 3)
```

Dictionary:

```python
{"name": "Ada", "age": 36}
```

Set:

```python
{1, 2, 3}
```

Each creates an object.

Inner expressions are evaluated first.

Example:

```python
x = 10
items = [x, x + 1, x + 2]
```

Evaluation:

```text
x      -> 10
x + 1  -> 11
x + 2  -> 12
create list object [10, 11, 12]
```

---

# Conditional Expressions

Python has conditional expressions.

Syntax:

```python
value_if_true if condition else value_if_false
```

Example:

```python
status = "adult" if age >= 18 else "minor"
```

This is an expression.

It evaluates to one of two objects.

Equivalent statement form:

```python
if age >= 18:
    status = "adult"
else:
    status = "minor"
```

Use conditional expressions for simple value selection.

Do not use them for complex branching.

---

# Conditional Expression Evaluation

Example:

```python
result = "yes" if condition else "no"
```

Evaluation:

```text
1. Evaluate condition.
2. If truthy, evaluate "yes".
3. Otherwise, evaluate "no".
4. Bind result to the chosen object.
```

Only one branch expression is evaluated.

Example:

```python
value = "safe" if denominator != 0 else "cannot divide"
```

The selected branch depends on truthiness of the condition.

---

# Nested Expressions

Expressions can contain expressions.

Example:

```python
result = (2 + 3) * (4 + 5)
```

Evaluation:

```text
2 + 3 -> 5
4 + 5 -> 9
5 * 9 -> 45
```

Example:

```python
message = name.strip().title()
```

Evaluation:

```text
name -> string object
strip() -> new string object
title() -> new string object
bind message
```

Nested expressions are powerful.

They can also become unreadable if overused.

---

# Evaluation Order

Evaluation order is the order in which Python evaluates parts of an expression.

For many expressions, Python evaluates from left to right.

Example:

```python
result = f() + g()
```

Python evaluates `f()` before `g()`.

This matters if functions have side effects.

Example:

```python
def f():
    print("f")
    return 1

def g():
    print("g")
    return 2

print(f() + g())
```

Output:

```text
f
g
3
```

The function calls produce values and print as side effects.

---

# Precedence vs Evaluation Order

Precedence determines grouping.

Evaluation order determines when subexpressions run.

Example:

```python
result = f() + g() * h()
```

Precedence groups it as:

```python
f() + (g() * h())
```

But evaluation of calls still proceeds left to right:

```text
f()
g()
h()
```

Then multiplication and addition are applied according to the expression structure.

Do not confuse:

```text
which operation groups first
```

with:

```text
which subexpression is evaluated first
```

---

# Short-Circuit Evaluation in Expressions

Boolean expressions can short-circuit.

Example:

```python
user is not None and user.is_active
```

If:

```python
user is None
```

then Python does not evaluate:

```python
user.is_active
```

This expression safely avoids an attribute error.

Example with `or`:

```python
name or "Anonymous"
```

If `name` is truthy, Python does not evaluate the fallback expression further.

Short-circuiting is part of expression evaluation.

---

# Expressions and Assignment

Assignment uses expressions on the right side.

Example:

```python
x = 2 + 3
```

The expression:

```python
2 + 3
```

evaluates first.

Then assignment binds `x`.

This rule matters:

```python
x = 10
x = x + 1
```

Evaluation:

```text
1. Evaluate x + 1 using current x.
2. Produce int object 11.
3. Rebind x to 11.
```

The left-side name is not rebound until the right-side expression has been evaluated.

---

# Multiple Assignment Evaluation

Example:

```python
a, b = b, a
```

This swaps names.

Why does it work?

The right side is evaluated first:

```text
b, a
```

Then the left-side targets are assigned.

Example:

```python
a = "left"
b = "right"

a, b = b, a

print(a)
print(b)
```

Output:

```text
right
left
```

The right-side expression objects are determined before rebinding the left-side names.

---

# Expression Statements

An expression can be used as a statement.

Example:

```python
print("hello")
```

This is an expression statement.

The function call expression is evaluated.

Its return value is discarded.

The visible output comes from the side effect.

Another example:

```python
"hello".upper()
```

This evaluates to:

```text
"HELLO"
```

But if the result is not used, it is discarded.

This is why:

```python
text = "hello"
text.upper()
print(text)
```

prints:

```text
hello
```

The expression result was ignored.

---

# Expressions vs Statements

An expression produces an object.

A statement performs an action in the program structure.

Examples of expressions:

```python
2 + 3
name
func()
items[0]
"yes" if condition else "no"
```

Examples of statements:

```python
x = 10
if condition:
    ...
while condition:
    ...
def function():
    ...
return value
import math
```

Some statements contain expressions.

Example:

```python
if age >= 18:
    print("adult")
```

The condition:

```python
age >= 18
```

is an expression inside an `if` statement.

---

# Why if Is Not an Expression

This is invalid:

```python
status = if age >= 18:
```

`if` statements do not evaluate to objects.

Use a conditional expression for value selection:

```python
status = "adult" if age >= 18 else "minor"
```

Use an `if` statement for control flow:

```python
if age >= 18:
    status = "adult"
else:
    status = "minor"
```

Choose based on clarity.

---

# Calls With Arguments

Function arguments are expressions.

Example:

```python
print(2 + 3)
```

Before calling `print`, Python evaluates:

```python
2 + 3
```

to:

```text
int object 5
```

Then `print` receives that object.

Example:

```python
max(len(name), fallback_length)
```

Python evaluates the argument expressions before calling `max`.

This matters when arguments have side effects.

---

# Keyword Argument Expressions

Keyword argument values are expressions.

Example:

```python
print("hello", end="\n")
```

The keyword argument:

```python
end="\n"
```

passes the string object `"\n"` as the value for `end`.

Another example:

```python
connect(timeout=default_timeout * 2)
```

The expression:

```python
default_timeout * 2
```

is evaluated before the function receives the argument.

---

# Attribute and Method Chains

Method chains are nested expressions.

Example:

```python
result = text.strip().lower().replace(" ", "-")
```

Evaluation:

```text
text
strip()
lower()
replace()
bind result
```

Each method call returns an object used by the next call.

This can be readable for simple transformations.

It can become difficult to debug when too long.

If a chain becomes hard to understand, split it:

```python
cleaned = text.strip()
lowered = cleaned.lower()
result = lowered.replace(" ", "-")
```

---

# Side Effects

A side effect is an effect beyond producing a return value.

Examples:

* Printing
* Mutating a list
* Writing to a file
* Sending a network request
* Updating a database
* Changing object state

Example:

```python
items = []
result = items.append("a")

print(items)
print(result)
```

Output:

```text
['a']
None
```

The expression:

```python
items.append("a")
```

mutated the list as a side effect and returned `None`.

---

# Side Effects Inside Larger Expressions

Side effects inside larger expressions can make code harder to reason about.

Example:

```python
items = []
if items.append("a"):
    print("truthy")
```

This does not print:

```text
truthy
```

Why?

`append()` mutates the list and returns `None`, which is falsy.

Better:

```python
items.append("a")
if items:
    print("truthy")
```

Separate mutation from the condition.

This is clearer.

---

# Expressions Should Be Readable

Python allows compact expressions.

Compact is not always better.

Hard to read:

```python
result = "ok" if user and user.is_active and not user.is_blocked else "denied"
```

Sometimes acceptable.

But if logic grows:

```python
is_allowed = (
    user is not None
    and user.is_active
    and not user.is_blocked
)

result = "ok" if is_allowed else "denied"
```

This names the intermediate idea.

Good code exposes meaning.

---

# Constant Expressions

Some expressions contain only constants.

Example:

```python
2 + 3
```

Python may optimize some constant expressions internally.

You do not need to rely on this.

Write clear code.

Let the interpreter handle implementation details.

The important model remains:

```text
expression evaluates to object
```

---

# Generator and Comprehension Preview

Python has expression forms for building collections.

Example:

```python
[number * 2 for number in range(5)]
```

This is a list comprehension.

It evaluates to a list object:

```text
[0, 2, 4, 6, 8]
```

Generator expressions:

```python
(number * 2 for number in range(5))
```

produce generator objects.

These are advanced enough to study later.

For now, know that Python has rich expression forms beyond simple arithmetic.

---

# Lambda Preview

`lambda` creates a function object using expression syntax.

Example:

```python
double = lambda x: x * 2
print(double(5))
```

Output:

```text
10
```

`lambda` is an expression.

It evaluates to a function object.

We will study functions properly later.

For now, note:

```text
not all function creation uses def statements
```

But prefer `def` for named, non-trivial functions.

---

# Common Mistakes

## Misconception 1

### Expressions are the same as statements.

Expressions evaluate to objects.

Statements control program structure or perform actions.

Statements can contain expressions.

---

## Misconception 2

### Assignment produces a normal value like arithmetic.

Plain assignment is a statement.

It binds names.

Use assignment expressions only when specifically appropriate.

---

## Misconception 3

### A method call always returns the changed object.

Some methods mutate and return `None`.

Example:

```python
list.append()
list.sort()
```

Know the API behavior.

---

## Misconception 4

### If an expression has a visible effect, that effect is its value.

Printing is a side effect.

Mutation is a side effect.

The expression still has a return value, often `None`.

---

## Misconception 5

### Precedence and evaluation order are the same thing.

Precedence determines grouping.

Evaluation order determines when subexpressions run.

---

## Misconception 6

### Conditional expressions are always better than if statements.

Conditional expressions are good for simple value selection.

Use `if` statements for multi-step logic.

---

## Misconception 7

### Longer expressions are more Pythonic.

Readable expressions are Pythonic.

Split complex expressions into named intermediate values when that improves clarity.

---

# Real-world Usage

## Data Transformation

Expressions transform data:

```python
normalized = email.strip().casefold()
```

This is readable because each step is simple and sequential.

---

## Validation

Boolean expressions validate conditions:

```python
is_valid = bool(name.strip()) and "@" in email
```

The result is a boolean object.

Use clear variable names for complex predicates.

---

## Function Arguments

Expressions are often used as arguments:

```python
send_email(user.email.strip().casefold())
```

This is fine when simple.

If the expression grows, split it:

```python
email = user.email.strip().casefold()
send_email(email)
```

---

## Configuration Defaults

Conditional expressions can express defaults:

```python
timeout = 30 if timeout is None else timeout
```

This is clearer than:

```python
timeout = timeout or 30
```

when `0` is a valid timeout.

---

## Debugging

When debugging an expression, break it apart.

Instead of:

```python
result = transform(load(path).strip().casefold())
```

use:

```python
raw = load(path)
cleaned = raw.strip()
normalized = cleaned.casefold()
result = transform(normalized)
```

This exposes intermediate objects.

---

# Concept Connections

This chapter builds on:

```text
Chapter 13:
Numeric expressions evaluate to numeric objects.

Chapter 14:
String expressions evaluate to string objects.

Chapter 15:
Boolean expressions evaluate to bool objects or use truthiness.

Chapter 16:
Operators combine expressions.
```

It prepares:

```text
Control Flow:
    if and while use expressions as conditions

Functions:
    arguments and return values are expressions

Scope:
    name expressions resolve through namespaces

Call Stack:
    function call expressions create frames
```

Core model:

```text
source expression
    |
    v
evaluation
    |
    v
result object
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| Expression | Code that evaluates to an object |
| Literal expression | Expression written directly as a value |
| Name expression | Name lookup producing an object |
| Operator expression | Expression using operators and operands |
| Call expression | Function or callable invocation producing a result |
| Attribute access | Getting an attribute from an object |
| Index expression | Getting an item by index/key |
| Slice expression | Producing a subsequence or slice result |
| Conditional expression | `a if condition else b` |
| Evaluation order | Order subexpressions are evaluated |
| Side effect | Effect beyond returning a value |

Core rules:

```text
Expressions evaluate to objects.
Names are looked up during expression evaluation.
Operators combine expression results.
Function arguments are expressions.
Assignment evaluates the right side first.
Expression statements discard unused results.
Side effects should be used deliberately.
```

---

# Active Recall

## Easy Recall Questions

1. What is an expression?
2. Does an expression evaluate to an object?
3. Is `10` an expression?
4. Is `x + 1` an expression?
5. Is `print("hello")` an expression?
6. What does `print()` return?
7. What is a conditional expression?
8. What is a side effect?
9. Does assignment evaluate the right side first?
10. What is the difference between precedence and evaluation order?

---

## Deep Understanding Questions

1. Why is a name considered an expression?
2. Why does ignoring a string method result often cause bugs?
3. Why can function calls be expressions even when they have side effects?
4. Why does `a, b = b, a` work?
5. Why should complex expressions often be split into intermediate names?
6. Why is `append()` inside an `if` condition usually a mistake?
7. Why are conditional expressions best for simple value selection?
8. Why do expressions matter for function arguments?
9. Why are statements and expressions distinct in Python?
10. Why does expression evaluation connect syntax to runtime objects?

---

## Explain In Your Own Words

1. Explain expression evaluation.
2. Explain how `2 + 3 * 4` is grouped and evaluated.
3. Explain why `print("hello")` produces `None`.
4. Explain right-side evaluation in assignment.
5. Explain conditional expressions.
6. Explain side effects inside expressions.
7. Explain why readability matters for expressions.

---

## Predict-the-Output Questions

### Question 1

```python
result = print("hello")
print(result)
```

Answer:

```text
hello
None
```

Reason:

`print()` has the side effect of printing and returns `None`.

---

### Question 2

```python
text = " hello "
text.strip()
print(repr(text))
```

Answer:

```text
' hello '
```

Reason:

`strip()` returns a new string. The result was ignored.

---

### Question 3

```python
a = "left"
b = "right"
a, b = b, a

print(a)
print(b)
```

Answer:

```text
right
left
```

Reason:

The right side is evaluated before the left-side names are rebound.

---

### Question 4

```python
age = 20
status = "adult" if age >= 18 else "minor"
print(status)
```

Answer:

```text
adult
```

Reason:

The condition is truthy, so the first branch expression is selected.

---

### Question 5

```python
items = []
result = items.append("x")

print(items)
print(result)
```

Answer:

```text
['x']
None
```

Reason:

`append()` mutates the list and returns `None`.

---

### Question 6

```python
def f():
    print("f")
    return 1

def g():
    print("g")
    return 2

print(f() + g())
```

Answer:

```text
f
g
3
```

Reason:

The call expressions are evaluated left to right, then the addition result is printed.

---

# Mental Model Questions

1. Draw `2 + 3` evaluating to an `int` object.
2. Draw `name.upper()` evaluating to a new string object.
3. Draw `age >= 18` evaluating to a bool object.
4. Draw right-side expression evaluation before assignment.
5. Draw a conditional expression selecting one branch.
6. Draw a function call expression with a side effect and return value.

---

# Practical Exercises

## Exercise 1

Identify expressions:

```python
x = 10
y = x + 1
print(y)
```

List every expression and explain what object it evaluates to.

---

## Exercise 2

Break down evaluation:

```python
result = (2 + 3) * (4 + 5)
```

Write each evaluation step and final object.

---

## Exercise 3

Investigate expression statements:

```python
text = "python"
text.upper()
print(text)

text = text.upper()
print(text)
```

Explain why the two prints differ.

---

## Exercise 4

Conditional expression:

```python
def label(age):
    return "adult" if age >= 18 else "minor"
```

Test several ages and explain the result.

---

## Exercise 5

Side effects:

```python
items = []

if items.append("a"):
    print("truthy")
else:
    print("falsy")

print(items)
```

Predict and explain the output.

---

## Exercise 6

Evaluation order:

```python
def show(name, value):
    print(name)
    return value

result = show("first", 1) + show("second", 2)
print(result)
```

Explain the output order.

---

## Exercise 7

Refactor a complex expression:

```python
result = transform(user.email.strip().casefold()) if user and user.email else None
```

Rewrite it using intermediate names and an `if` statement for clarity.

---

# Summary

In this chapter we learned:

* Expressions are code that evaluate to objects.
* Literals, names, operators, calls, attributes, indexes, slices, and conditional expressions are expressions.
* Name expressions perform runtime name lookup.
* Operator expressions combine operands and produce results.
* Function calls are expressions and return objects.
* Some expressions have side effects.
* `print()` returns `None`.
* Methods such as `append()` can mutate and return `None`.
* Assignment evaluates the right side first.
* Multiple assignment evaluates the right side before rebinding left-side names.
* Expression statements discard unused results.
* Expressions and statements are distinct.
* Conditional expressions are useful for simple value selection.
* Precedence controls grouping.
* Evaluation order controls when subexpressions run.
* Readability matters more than compressing logic.

Core model:

```text
expression
    |
    v
evaluation
    |
    v
object
```

Expressions are the value-producing layer of Python code.

---

# Preview of Chapter 18

Next we study conditionals.

Expressions produce values.

Conditionals use expression results to decide which branch of code runs.

We will study:

* `if`
* `elif`
* `else`
* Truthiness in conditions
* Branch ordering
* Nested conditionals
* Guard conditions
* Conditional expressions
* Pattern matching with `match`

The connection is direct:

```text
condition expression -> truthiness -> control-flow decision
```

After conditionals, Chapter 19 will study loops: repeated execution with `while`, `for`, `range`, `break`, `continue`, and nested iteration.
