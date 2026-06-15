# Chapter 15 — Booleans

---

# Learning Objectives

By the end of this chapter, you should understand:

* What booleans are.
* Why `True` and `False` are objects.
* Why `bool` is related to `int`.
* How comparisons produce boolean objects.
* What truthiness means.
* Which common values are truthy or falsy.
* How `bool()` converts objects to truth values.
* How `and`, `or`, and `not` work.
* What short-circuit evaluation means.
* Why `and` and `or` return operands, not always `True` or `False`.
* How booleans control `if`, `while`, and other conditional contexts.
* Why `is True` and `is False` are usually not appropriate.
* Common mistakes with boolean expressions.

Booleans are small, but they are central to program flow.

Without booleans, programs could compute values but not make decisions.

---

# Concept Overview

Python has two boolean values:

```python
True
False
```

Their type is `bool`.

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

Booleans answer yes/no questions.

Examples:

```python
age >= 18
username == "admin"
"@" in email
items == []
```

Each expression produces a boolean result:

```python
print(10 > 3)
print("py" in "python")
print([1, 2] == [1, 2])
```

Output:

```text
True
True
True
```

Booleans connect expressions to control flow.

```python
if age >= 18:
    print("allowed")
```

The condition decides whether the block runs.

---

# Booleans Are Objects

`True` and `False` are objects.

They have:

```text
identity
type
value
```

Conceptually:

```text
bool object
├── identity: specific runtime object
├── type: bool
└── value: True
```

and:

```text
bool object
├── identity: specific runtime object
├── type: bool
└── value: False
```

There is one `True` object and one `False` object in a Python process.

They are singletons.

But normal boolean logic usually cares about truth value, not object identity.

---

# bool Is Related to int

`bool` is a subclass of `int`.

Example:

```python
print(isinstance(True, int))
print(isinstance(False, int))
```

Output:

```text
True
True
```

In arithmetic contexts:

```python
print(True + True)
print(False + 10)
```

Output:

```text
2
10
```

`True` behaves like `1`.

`False` behaves like `0`.

This exists for historical and practical compatibility.

But in most code, booleans should represent truth values, not ordinary numbers.

Do not write unclear code like:

```python
total += is_active
```

unless counting truth values is explicitly intended.

---

# Why Booleans Exist

Programs need decisions.

Examples:

```python
if password_is_valid:
    login()
```

```python
if temperature > 100:
    warn()
```

```python
while queue_has_items:
    process_next()
```

Booleans represent the result of a condition.

They answer:

```text
Should this branch run?
Should this loop continue?
Is this condition satisfied?
```

Without booleans, programs would be straight-line calculations.

With booleans, programs can branch, validate, filter, and control behavior.

---

# Boolean Literals

The two boolean literals are:

```python
True
False
```

Capitalization matters.

These are correct:

```python
is_active = True
is_deleted = False
```

These are not Python boolean literals:

```python
true
false
TRUE
FALSE
```

Python will treat those as names and raise `NameError` if they are not defined.

---

# Comparisons Produce Booleans

Comparison operators produce boolean results.

Examples:

```python
print(5 > 3)
print(5 < 3)
print(5 == 5)
print(5 != 3)
```

Output:

```text
True
False
True
True
```

Each comparison expression evaluates to a `bool` object.

Example:

```python
result = 5 > 3

print(result)
print(type(result))
```

Output:

```text
True
<class 'bool'>
```

The comparison creates a boolean value that can be stored, passed, returned, or used in control flow.

---

# Equality Comparisons

Equality operators:

```python
==
!=
```

Examples:

```python
print("python" == "python")
print("python" != "java")
print([1, 2] == [1, 2])
```

Output:

```text
True
True
True
```

As Chapter 12 explained:

```text
== checks equality
is checks identity
```

Boolean results often come from equality comparisons.

---

# Ordering Comparisons

Ordering operators:

```python
<
<=
>
>=
```

Examples:

```python
print(3 < 5)
print(3 <= 3)
print(10 > 2)
print(10 >= 10)
```

Output:

```text
True
True
True
True
```

Ordering is type-dependent.

Numbers support numeric ordering.

Strings support lexicographic ordering.

Some objects do not support ordering with each other.

Example:

```python
3 < "5"
```

raises:

```text
TypeError
```

Python does not invent arbitrary ordering between unrelated types.

---

# Chained Comparisons

Python supports chained comparisons.

Example:

```python
age = 25

print(18 <= age < 65)
```

Output:

```text
True
```

This means:

```text
18 <= age and age < 65
```

but `age` is evaluated once.

Chained comparisons are useful for ranges:

```python
if 0 <= percentage <= 100:
    print("valid")
```

This is clearer than:

```python
if percentage >= 0 and percentage <= 100:
    print("valid")
```

---

# Identity Comparisons Produce Booleans

The `is` operator produces a boolean.

Example:

```python
value = None

print(value is None)
```

Output:

```text
True
```

`is` checks identity.

It asks:

```text
Is this the exact same object?
```

Common correct use:

```python
if value is None:
    ...
```

Avoid:

```python
if value is True:
    ...
```

unless exact identity with the boolean object is specifically required.

Most conditionals should use truthiness directly.

---

# Membership Tests Produce Booleans

The `in` operator checks membership.

Example:

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

Membership tests produce boolean objects.

They are common in validation:

```python
if "@" in email:
    print("has at sign")
```

Remember that membership checks may be too simple for full validation.

---

# Truthiness

Python does not require every condition to be exactly `True` or `False`.

Objects have truth values.

This is called truthiness.

Example:

```python
if "hello":
    print("runs")
```

Output:

```text
runs
```

The string `"hello"` is not the boolean object `True`.

But it is truthy.

Example:

```python
if "":
    print("does not run")
```

An empty string is falsy.

Truthiness lets Python write concise conditions.

---

# bool()

The built-in `bool()` converts an object to its truth value.

Examples:

```python
print(bool(""))
print(bool("hello"))
print(bool(0))
print(bool(42))
print(bool([]))
print(bool([1, 2]))
print(bool(None))
```

Output:

```text
False
True
False
True
False
True
False
```

`bool()` does not necessarily create a new meaningful domain value.

It asks:

```text
How does this object behave in a boolean context?
```

---

# Common Falsy Values

Common falsy values:

| Value | Type |
| --- | --- |
| `False` | `bool` |
| `None` | `NoneType` |
| `0` | `int` |
| `0.0` | `float` |
| `0j` | `complex` |
| `""` | `str` |
| `[]` | `list` |
| `{}` | `dict` |
| `set()` | `set` |
| `()` | `tuple` |
| `range(0)` | `range` |

Most other objects are truthy by default.

Rule of thumb:

```text
empty or zero-like values are usually falsy
non-empty or non-zero values are usually truthy
```

---

# Truthiness of Containers

Containers are falsy when empty and truthy when non-empty.

Examples:

```python
print(bool([]))
print(bool([1]))
print(bool({}))
print(bool({"a": 1}))
```

Output:

```text
False
True
False
True
```

This makes conditions concise:

```python
if items:
    print("items exist")
```

This is usually preferred over:

```python
if len(items) > 0:
    print("items exist")
```

Both can work.

The first is idiomatic Python.

---

# Truthiness of Numbers

Zero is falsy.

Non-zero numbers are truthy.

Examples:

```python
print(bool(0))
print(bool(1))
print(bool(-1))
print(bool(0.0))
print(bool(0.1))
```

Output:

```text
False
True
True
False
True
```

This is useful, but be careful.

Sometimes zero is a valid value, not the same as missing data.

Example:

```python
age = 0
```

For a newborn, this may be valid.

Do not write:

```python
if age:
    ...
```

if you need to distinguish `0` from missing.

Use:

```python
if age is not None:
    ...
```

when `None` represents missing.

---

# Truthiness of Strings

Empty strings are falsy.

Non-empty strings are truthy.

Examples:

```python
print(bool(""))
print(bool(" "))
print(bool("hello"))
```

Output:

```text
False
True
True
```

Important:

```python
" "
```

is not empty.

It contains a space.

If user input may contain only whitespace, use `strip()`:

```python
if name.strip():
    print("non-empty after trimming")
```

---

# not

`not` negates truthiness.

Example:

```python
is_active = False

print(not is_active)
```

Output:

```text
True
```

With containers:

```python
items = []

if not items:
    print("no items")
```

Output:

```text
no items
```

`not` always returns a boolean object:

```python
print(type(not []))
```

Output:

```text
<class 'bool'>
```

---

# and

`and` evaluates expressions from left to right.

It returns the first falsy operand, or the last operand if all are truthy.

Example:

```python
print(True and True)
print(True and False)
```

Output:

```text
True
False
```

But `and` does not always return a `bool`.

Example:

```python
print("hello" and 123)
print("" and 123)
```

Output:

```text
123

```

The second print displays an empty line because the result is the empty string.

Rules:

```text
x and y:
    if x is falsy, return x
    otherwise, return y
```

---

# or

`or` evaluates expressions from left to right.

It returns the first truthy operand, or the last operand if all are falsy.

Example:

```python
print(True or False)
print(False or True)
```

Output:

```text
True
True
```

But `or` does not always return a `bool`.

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

Rules:

```text
x or y:
    if x is truthy, return x
    otherwise, return y
```

This behavior is useful, but it can also hide bugs if used carelessly.

---

# Short-Circuit Evaluation

`and` and `or` short-circuit.

Short-circuiting means Python may skip evaluating the right side.

For `and`:

```python
False and expensive_call()
```

Python does not call `expensive_call()`.

The result is already known to be falsy.

For `or`:

```python
True or expensive_call()
```

Python does not call `expensive_call()`.

The result is already known to be truthy.

This matters for performance and safety.

---

# Short-Circuiting for Safety

Example:

```python
if user is not None and user.is_active:
    print("active")
```

If `user is None`, Python does not evaluate:

```python
user.is_active
```

This prevents an error.

The order matters.

This is unsafe:

```python
if user.is_active and user is not None:
    ...
```

If `user` is `None`, accessing `user.is_active` fails before the `None` check.

---

# Boolean Contexts

Python uses truthiness in boolean contexts.

Common boolean contexts:

```python
if condition:
    ...
```

```python
while condition:
    ...
```

```python
value if condition else other_value
```

```python
assert condition
```

```python
filter(function, iterable)
```

The object in the condition is converted to a truth value.

It does not need to be exactly `True` or `False`.

---

# if Conditions

Example:

```python
items = ["a", "b"]

if items:
    print("non-empty")
```

Output:

```text
non-empty
```

Python evaluates the truthiness of `items`.

Because the list is non-empty, the block runs.

If:

```python
items = []
```

the block does not run.

---

# while Conditions

`while` loops also use truthiness.

Example:

```python
items = [1, 2, 3]

while items:
    item = items.pop()
    print(item)
```

This loop continues while the list is non-empty.

When the list becomes empty, it is falsy and the loop stops.

Output:

```text
3
2
1
```

This depends on mutability and truthiness together.

---

# Boolean Variables

Boolean variables should usually be named as predicates.

Good names:

```python
is_active = True
has_permission = False
can_retry = True
should_send_email = False
```

These names read naturally in conditions:

```python
if is_active:
    ...
```

Avoid vague names:

```python
flag = True
status = False
```

unless the context makes them clear.

Names should communicate the question the boolean answers.

---

# Boolean Expressions Should Be Clear

Avoid unnecessary comparison to `True`.

Instead of:

```python
if is_active == True:
    ...
```

write:

```python
if is_active:
    ...
```

Instead of:

```python
if is_active == False:
    ...
```

write:

```python
if not is_active:
    ...
```

This is clearer and idiomatic.

Exception:

If a value can be truthy but not exactly `True`, and exact boolean identity matters, be explicit. That is less common.

---

# is True and is False

Avoid this in normal code:

```python
if value is True:
    ...
```

Why?

It checks identity with the specific boolean object `True`.

It does not merely check truthiness.

Example:

```python
value = 1

print(bool(value))
print(value is True)
```

Output:

```text
True
False
```

The integer `1` is truthy, but it is not the object `True`.

Usually use:

```python
if value:
    ...
```

or if exact type matters:

```python
if value == True:
    ...
```

But exact boolean comparisons are uncommon in clean Python code.

---

# None Checks Are Different

For `None`, identity checks are correct.

Example:

```python
if value is None:
    ...
```

Why?

`None` is a singleton sentinel.

You are asking whether the value is the exact `None` object.

This is different from truthiness.

Example:

```python
value = 0

print(value is None)
print(bool(value))
```

Output:

```text
False
False
```

`0` is falsy, but it is not missing.

Use `is None` when missingness is the question.

---

# Default Values With or

You may see:

```python
name = provided_name or "Anonymous"
```

This means:

```text
if provided_name is truthy, use it
otherwise use "Anonymous"
```

This can be useful.

But it can be wrong when falsy values are valid.

Example:

```python
timeout = provided_timeout or 30
```

If `provided_timeout` is `0`, this chooses `30`.

That may be a bug if `0` is valid.

Safer:

```python
timeout = 30 if provided_timeout is None else provided_timeout
```

The correct pattern depends on whether falsy values should be treated as missing.

---

# Boolean Arithmetic

Because booleans are related to integers, this works:

```python
values = [True, False, True]
print(sum(values))
```

Output:

```text
2
```

This can be useful for counting conditions:

```python
passed = sum(score >= 50 for score in scores)
```

But use it when the intent is clear.

If readability suffers, write more explicit code.

---

# any()

`any()` returns `True` if any item is truthy.

Example:

```python
values = [0, "", "hello"]

print(any(values))
```

Output:

```text
True
```

Because `"hello"` is truthy.

Useful:

```python
if any(user.is_admin for user in users):
    print("at least one admin")
```

`any()` short-circuits.

It stops once it finds a truthy value.

---

# all()

`all()` returns `True` if all items are truthy.

Example:

```python
values = [1, "x", True]

print(all(values))
```

Output:

```text
True
```

With a falsy item:

```python
print(all([1, "", True]))
```

Output:

```text
False
```

Useful:

```python
if all(score >= 50 for score in scores):
    print("everyone passed")
```

`all()` short-circuits.

It stops once it finds a falsy value.

---

# Boolean Results Are Values

Because booleans are objects, they can be assigned, returned, and passed.

Example:

```python
def is_adult(age):
    return age >= 18

result = is_adult(20)

print(result)
print(type(result))
```

Output:

```text
True
<class 'bool'>
```

This is often better than writing:

```python
def is_adult(age):
    if age >= 18:
        return True
    else:
        return False
```

The comparison already produces a boolean.

---

# De Morgan's Laws

Boolean logic has transformation rules.

Two useful ones:

```text
not (A and B) == (not A) or (not B)
not (A or B)  == (not A) and (not B)
```

Example:

```python
if not (is_admin or is_owner):
    print("access denied")
```

Equivalent:

```python
if not is_admin and not is_owner:
    print("access denied")
```

Choose the clearer form.

Do not transform boolean expressions mechanically if readability gets worse.

---

# Common Mistakes

## Misconception 1

### Conditions must be exactly True or False.

Conditions use truthiness.

Many objects can be truthy or falsy.

---

## Misconception 2

### `and` and `or` always return booleans.

They return operands.

Example:

```python
"hello" or "fallback"
```

returns:

```text
hello
```

---

## Misconception 3

### `is True` checks truthiness.

It checks identity with the `True` object.

Use:

```python
if value:
```

for truthiness.

---

## Misconception 4

### Falsy always means missing.

Falsy values include `0`, `""`, `[]`, and `False`.

These can be valid data.

Use `is None` when checking for missing values represented by `None`.

---

## Misconception 5

### `not x == y` is always clearer than `x != y`.

Usually:

```python
x != y
```

is clearer.

Use direct comparison operators when available.

---

## Misconception 6

### Boolean expressions should compare to True explicitly.

Usually avoid:

```python
if condition == True:
```

Use:

```python
if condition:
```

---

## Misconception 7

### `or` defaults are always safe.

`x or default` treats all falsy values as missing.

That is wrong when `0`, `""`, or `False` are valid explicit values.

---

# Real-world Usage

## Validation

Booleans are used to validate data.

Example:

```python
has_email = "@" in email
has_name = bool(name.strip())

if has_email and has_name:
    print("valid enough for this simple check")
```

In real systems, validation rules should be precise.

---

## Permissions

Permission checks are boolean logic.

Example:

```python
can_edit = is_admin or is_owner

if can_edit:
    show_edit_button()
```

Good boolean names make code readable.

---

## Feature Flags

Feature flags are often booleans.

Example:

```python
if enable_new_checkout:
    use_new_checkout()
else:
    use_old_checkout()
```

Boolean names should make the enabled behavior clear.

---

## Input Defaults

Be careful with falsy values.

Risky:

```python
limit = user_limit or 100
```

Safer if `0` is valid:

```python
limit = 100 if user_limit is None else user_limit
```

The logic should match the domain.

---

## Filtering

Booleans often filter data.

Example:

```python
active_users = [user for user in users if user.is_active]
```

The expression after `if` is evaluated in a boolean context.

List comprehensions will be studied later.

---

# Concept Connections

This chapter builds on:

```text
Chapter 13:
Numeric comparisons produce booleans.

Chapter 14:
String comparisons, membership checks, and emptiness produce or use booleans.

Chapter 12:
Identity checks such as is None produce booleans.
```

It prepares:

```text
Operators:
    and, or, not, ==, <, in, is are operators with boolean behavior

Expressions:
    boolean expressions evaluate to objects

Control flow:
    if and while depend on truthiness

Functions:
    predicates return booleans
```

Core model:

```text
expression
    |
    v
object
    |
    v
truth value used by condition
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| `bool` | Boolean type |
| `True` | Boolean true object |
| `False` | Boolean false object |
| Truthiness | How an object behaves in a boolean context |
| Falsy | Object treated as false in a condition |
| Truthy | Object treated as true in a condition |
| `bool(obj)` | Convert object to truth value |
| `and` | Return first falsy operand or last operand |
| `or` | Return first truthy operand or last operand |
| `not` | Return boolean negation |
| Short-circuit | Skip evaluating unnecessary expressions |

Core rules:

```text
Comparisons produce bool objects.
Conditions use truthiness.
not always returns a bool.
and and or return operands.
Use is None for None checks.
Use if value for truthiness.
Do not confuse falsy with missing.
```

---

# Active Recall

## Easy Recall Questions

1. What are Python's two boolean values?
2. What is the type of `True`?
3. What does `bool("")` return?
4. What does `bool(" ")` return?
5. What does `not []` return?
6. Does `and` always return a boolean?
7. Does `or` always return a boolean?
8. What does short-circuiting mean?
9. What is the idiomatic way to check for `None`?
10. Why should `if condition == True` usually be avoided?

---

## Deep Understanding Questions

1. Why do conditions use truthiness instead of requiring exact booleans?
2. Why can `x or default` be incorrect when `x` is `0`?
3. Why does `and` return the first falsy operand?
4. Why does `or` return the first truthy operand?
5. Why is `user is not None and user.is_active` safer than the reverse order?
6. Why is `is True` different from checking truthiness?
7. Why is `None` checking different from truthiness checking?
8. Why can booleans be added like integers?
9. Why should boolean names read like predicates?
10. Why do `any()` and `all()` short-circuit?

---

## Explain In Your Own Words

1. Explain truthiness.
2. Explain the difference between `bool(value)` and `value is True`.
3. Explain short-circuit evaluation.
4. Explain why `and` and `or` return operands.
5. Explain why `is None` is preferred.
6. Explain why falsy does not always mean missing.
7. Explain how booleans connect expressions to control flow.

---

## Predict-the-Output Questions

### Question 1

```python
print(bool(""))
print(bool(" "))
print(bool("hello"))
```

Answer:

```text
False
True
True
```

Reason:

Only the empty string is falsy.

---

### Question 2

```python
print("hello" and 123)
print("" and 123)
```

Answer:

```text
123

```

Reason:

`and` returns the first falsy operand, otherwise the last operand.

---

### Question 3

```python
print("hello" or 123)
print("" or 123)
```

Answer:

```text
hello
123
```

Reason:

`or` returns the first truthy operand, otherwise the last operand.

---

### Question 4

```python
value = 0

print(value is None)
print(bool(value))
```

Answer:

```text
False
False
```

Reason:

`0` is falsy, but it is not `None`.

---

### Question 5

```python
print(True + True)
print(False + 5)
```

Answer:

```text
2
5
```

Reason:

`bool` is related to `int`.

---

### Question 6

```python
values = [0, "", [], "ok"]
print(any(values))
print(all(values))
```

Answer:

```text
True
False
```

Reason:

At least one value is truthy, but not all values are truthy.

---

# Mental Model Questions

1. Draw a comparison expression evaluating to a bool object.
2. Draw `value is None` as an identity comparison producing a bool.
3. Draw `x and y` returning `x` when `x` is falsy.
4. Draw `x or y` returning `x` when `x` is truthy.
5. Draw why `0` is falsy but not missing.
6. Draw short-circuiting for `user is not None and user.is_active`.

---

# Practical Exercises

## Exercise 1

Inspect truthiness:

```python
values = [False, None, 0, 1, "", " ", [], [1], {}, {"a": 1}]

for value in values:
    print(repr(value), bool(value))
```

Explain each result.

---

## Exercise 2

Test `and` and `or` return values:

```python
print("a" and "b")
print("" and "b")
print("a" or "b")
print("" or "b")
```

Explain why each expression returns that operand.

---

## Exercise 3

Fix a default bug:

```python
def set_limit(limit=None):
    limit = limit or 100
    return limit

print(set_limit(0))
```

Rewrite it so `0` remains valid and only `None` uses the default.

---

## Exercise 4

Use short-circuiting safely:

```python
user = None

if user is not None and user.is_active:
    print("active")
```

Explain why this does not raise an error.

Then reverse the condition and explain why it fails.

---

## Exercise 5

Use `any()` and `all()`:

```python
scores = [80, 70, 45, 90]

print(any(score < 50 for score in scores))
print(all(score >= 50 for score in scores))
```

Explain both results.

---

## Exercise 6

Write predicate functions:

```python
def is_even(number):
    ...

def is_non_empty(text):
    ...
```

Each should return a boolean expression directly.

---

## Exercise 7

Analyze missing vs falsy:

```python
values = [None, 0, "", [], False]

for value in values:
    print(value is None, bool(value))
```

Explain why only one value is missing if `None` is your sentinel.

---

# Summary

In this chapter we learned:

* `True` and `False` are boolean objects.
* Their type is `bool`.
* `bool` is related to `int`, so booleans can behave numerically.
* Comparisons produce boolean objects.
* Conditions use truthiness.
* Empty and zero-like values are usually falsy.
* Non-empty and non-zero values are usually truthy.
* `bool()` shows an object's truth value.
* `not` returns a boolean negation.
* `and` returns the first falsy operand or the last operand.
* `or` returns the first truthy operand or the last operand.
* `and` and `or` short-circuit.
* Short-circuiting can prevent unsafe evaluation.
* Use `is None` for missing-value checks.
* Do not confuse falsy values with missing values.
* Avoid unnecessary comparisons to `True` and `False`.
* Use clear predicate names for boolean variables.

The core model is:

```text
expression
    |
    v
object
    |
    v
truthiness
    |
    v
control decision
```

Booleans are the bridge between values and program decisions.

---

# Preview of Chapter 16

Next we study operators.

We have already used many operators:

```python
+
-
*
/
==
is
in
and
or
not
```

Chapter 16 will organize them into a complete model:

* Arithmetic operators
* Comparison operators
* Boolean operators
* Identity operators
* Membership operators
* Assignment-related operators
* Operator precedence
* Type-driven operator behavior

Operators are not just symbols.

They are syntax for object behavior.
