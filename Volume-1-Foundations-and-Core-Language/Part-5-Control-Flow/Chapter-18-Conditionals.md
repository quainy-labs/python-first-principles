# Chapter 18 — Conditionals

---

# Learning Objectives

By the end of this chapter, you should understand:

* What conditional execution means.
* Why programs need decision-making.
* How `if`, `elif`, and `else` work.
* How Python evaluates condition expressions.
* How truthiness connects to booleans, objects, and custom types.
* How indentation creates conditional blocks.
* How branch order affects behavior.
* How nested conditionals work.
* How guard conditions reduce nesting.
* How conditional expressions work.
* How `match` statements fit into branching.
* How to design readable decision logic.
* Common mistakes with truthiness, branch order, equality, identity, and indentation.

Loops are covered in the next chapter. This chapter focuses only on decisions.

---

# Concept Overview

A conditional is a statement that lets a program choose whether a block of code should run.

Without conditionals, Python executes statements from top to bottom:

```python
print("read input")
print("process input")
print("show result")
```

Every line runs.

With a conditional, some lines run only when a condition is satisfied:

```python
age = 20

if age >= 18:
    print("adult")
```

The line inside the `if` block runs because the condition is true.

The basic mental model is:

```text
evaluate condition
        |
        v
truthy or falsy?
        |
        v
choose block
```

Python does not ask whether a condition is literally the boolean object `True`.

Python asks whether the condition is truthy.

That distinction matters because many objects can be used in conditions:

```python
if "Ada":
    print("non-empty string")

if [1, 2, 3]:
    print("non-empty list")

if 42:
    print("non-zero number")
```

Each of these conditions is truthy.

---

# Why Conditionals Exist

Programs need to react to different situations.

Examples:

```text
If the password is correct, allow login.
If the file exists, read it.
If the user is an admin, show admin tools.
If the cart is empty, disable checkout.
If the input is invalid, show an error.
If the network request failed, retry or report failure.
```

Conditionals are the foundation of program behavior that depends on data.

They give programs:

* Validation paths.
* Error paths.
* Permission checks.
* Configuration choices.
* Mode selection.
* Boundary handling.
* Business rules.

Without conditionals, a program cannot adapt.

It can only do the same thing every time.

---

# Blocks and Indentation

Python uses indentation to define blocks.

Example:

```python
if True:
    print("inside")
    print("also inside")

print("outside")
```

Output:

```text
inside
also inside
outside
```

The two indented lines belong to the `if` block.

The unindented line is outside the block.

Indentation is syntax in Python.

It is not visual decoration.

This code is different:

```python
if True:
    print("inside")

print("outside")
```

Only the first `print` belongs to the condition.

This code is invalid:

```python
if True:
print("inside")
```

Python raises an indentation error because a block is required after the colon.

Mental model:

```text
colon starts a block
indentation defines membership
dedentation ends the block
```

---

# The `if` Statement

The `if` statement runs a block only when its condition is truthy.

Syntax:

```python
if condition:
    block
```

Example:

```python
temperature = 38

if temperature > 37:
    print("fever")
```

Output:

```text
fever
```

The condition is:

```python
temperature > 37
```

That expression evaluates to a boolean object:

```python
True
```

Since it is truthy, Python runs the block.

If the condition is falsy, Python skips the block:

```python
temperature = 36

if temperature > 37:
    print("fever")

print("done")
```

Output:

```text
done
```

The conditional block was skipped.

---

# Conditions Are Expressions

The condition after `if` is an expression.

That means Python evaluates it to an object.

Examples:

```python
if age >= 18:
    print("adult")
```

```python
if username:
    print("username provided")
```

```python
if user.is_active:
    print("active user")
```

```python
if len(items) > 0:
    print("items exist")
```

Each condition produces an object.

Python then checks the truthiness of that object.

This connects directly to earlier chapters:

* Operators produce values.
* Comparisons produce booleans.
* Names refer to objects.
* Objects have truthiness.
* Conditionals choose execution paths based on truthiness.

---

# Truthiness

Python objects can be truthy or falsy.

Common falsy values:

```python
False
None
0
0.0
""
[]
()
{}
set()
range(0)
```

Common truthy values:

```python
True
1
-1
3.14
"hello"
[0]
(None,)
{"enabled": False}
object()
```

Important detail:

The contents of a container do not need to be truthy for the container itself to be truthy.

Example:

```python
values = [False]

if values:
    print("the list is non-empty")
```

Output:

```text
the list is non-empty
```

The list is truthy because it has one element.

Python is not checking whether every element inside the list is true.

---

# Truthiness vs Boolean Equality

This is good Python:

```python
if items:
    print("items exist")
```

This is usually weaker:

```python
if len(items) > 0:
    print("items exist")
```

This is worse:

```python
if items == True:
    print("items exist")
```

A list is not equal to `True`.

Example:

```python
items = [1, 2, 3]

print(bool(items))
print(items == True)
```

Output:

```text
True
False
```

Truthiness answers:

```text
Should this object count as true in a condition?
```

Equality answers:

```text
Is this object equal to another object?
```

They are different questions.

---

# The `else` Clause

`else` runs when the `if` condition is falsy.

Example:

```python
age = 15

if age >= 18:
    print("adult")
else:
    print("minor")
```

Output:

```text
minor
```

Only one branch runs.

The model is:

```text
if condition is truthy:
    run if block
otherwise:
    run else block
```

`else` does not have its own condition.

It means:

```text
everything else
```

This matters when the condition is broad.

Example:

```python
score = -20

if score >= 60:
    print("pass")
else:
    print("fail")
```

Output:

```text
fail
```

The code is technically correct, but maybe negative scores should be treated as invalid.

That requires another branch.

---

# The `elif` Clause

`elif` means "else if".

It checks another condition only if the earlier conditions failed.

Example:

```python
score = 84

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "D"

print(grade)
```

Output:

```text
B
```

Python checks branches from top to bottom.

The first truthy branch wins.

After a branch runs, the rest are skipped.

This is not the same as separate `if` statements.

---

# Branch Order Matters

Branch order can change behavior.

Bad order:

```python
score = 95

if score >= 60:
    grade = "pass"
elif score >= 90:
    grade = "excellent"

print(grade)
```

Output:

```text
pass
```

The `score >= 90` branch never runs because `score >= 60` catches it first.

Better order:

```python
score = 95

if score >= 90:
    grade = "excellent"
elif score >= 60:
    grade = "pass"

print(grade)
```

Output:

```text
excellent
```

Rule:

Put more specific conditions before broader conditions.

---

# Independent `if` Statements

Separate `if` statements are independent.

Example:

```python
score = 95

if score >= 60:
    print("pass")

if score >= 90:
    print("excellent")
```

Output:

```text
pass
excellent
```

Both blocks can run.

Use separate `if` statements when multiple facts may all be true.

Use `if` / `elif` / `else` when the branches are mutually exclusive.

Comparison:

```python
# Multiple labels can apply.
if user.is_admin:
    print("admin")

if user.is_active:
    print("active")

if user.has_paid:
    print("paid")
```

```python
# Exactly one role should be selected.
if role == "admin":
    permissions = "all"
elif role == "editor":
    permissions = "write"
else:
    permissions = "read"
```

---

# Nested Conditionals

A nested conditional is a conditional inside another conditional.

Example:

```python
is_logged_in = True
is_admin = False

if is_logged_in:
    if is_admin:
        print("admin dashboard")
    else:
        print("user dashboard")
else:
    print("login page")
```

Output:

```text
user dashboard
```

Nested conditionals are useful when one decision depends on another.

Mental model:

```text
first gate
    second gate
        selected action
```

But nesting increases cognitive load.

Deeply nested code is harder to read because the reader must remember every previous condition.

---

# Reducing Nesting With Combined Conditions

Sometimes nested conditions can be combined.

Nested:

```python
if user.is_active:
    if user.has_permission:
        print("allowed")
```

Combined:

```python
if user.is_active and user.has_permission:
    print("allowed")
```

This works when both conditions must be true and no separate `else` behavior is needed.

But do not combine conditions if it hides important business logic.

Readable code is not always the shortest code.

---

# Guard Conditions

A guard condition handles an invalid or special case early.

Example:

```python
def checkout(cart, user):
    if not cart:
        return "cart is empty"

    if not user.is_active:
        return "inactive user"

    if not user.has_payment_method:
        return "payment method required"

    return "checkout started"
```

Without guards, the same logic may become deeply nested:

```python
def checkout(cart, user):
    if cart:
        if user.is_active:
            if user.has_payment_method:
                return "checkout started"
            else:
                return "payment method required"
        else:
            return "inactive user"
    else:
        return "cart is empty"
```

The guard version is easier to scan.

Guard conditions are especially useful in functions.

They let the main path remain visually clear.

---

# Boolean Operators in Conditions

Conditions often use boolean operators:

* `and`
* `or`
* `not`

Example:

```python
age = 21
country = "US"

if age >= 18 and country == "US":
    print("eligible")
```

`and` requires both sides to be truthy.

Example:

```python
role = "editor"

if role == "admin" or role == "editor":
    print("can edit")
```

`or` requires at least one side to be truthy.

Example:

```python
is_blocked = False

if not is_blocked:
    print("allowed")
```

`not` reverses truthiness.

---

# Short-Circuit Evaluation

Python short-circuits boolean expressions.

For `and`, if the left side is falsy, Python does not evaluate the right side.

Example:

```python
user = None

if user is not None and user.is_active:
    print("active")
```

This is safe because `user.is_active` is checked only if `user is not None`.

For `or`, if the left side is truthy, Python does not evaluate the right side.

Example:

```python
name = provided_name or "Anonymous"
```

If `provided_name` is truthy, it is used.

Otherwise, `"Anonymous"` is used.

Short-circuiting is useful, but it can become confusing if the expressions have side effects.

Keep conditions mostly about checking state, not changing state.

---

# Comparing Values

Conditionals often use comparison operators:

```python
==
!=
<
<=
>
>=
```

Example:

```python
if price <= budget:
    print("buy")
```

Comparisons produce booleans:

```python
print(10 < 20)
print("a" == "a")
print([1, 2] != [1, 2, 3])
```

Output:

```text
True
True
True
```

Python also supports chained comparisons:

```python
if 0 <= percentage <= 100:
    print("valid percentage")
```

This is equivalent to:

```python
if percentage >= 0 and percentage <= 100:
    print("valid percentage")
```

The chained form is usually clearer.

---

# Equality vs Identity in Conditions

Use `==` to compare values.

Use `is` to compare identities.

Example:

```python
status = "done"

if status == "done":
    print("complete")
```

Use `is` for `None`:

```python
result = None

if result is None:
    print("no result")
```

Do not write:

```python
if result == None:
    print("no result")
```

`None` is a singleton object.

Identity is the right check.

Also avoid:

```python
if is_ready is True:
    print("ready")
```

Prefer:

```python
if is_ready:
    print("ready")
```

Unless you specifically need to distinguish the exact boolean object `True` from other truthy values, truthiness is the normal Python style.

---

# Membership Conditions

Use `in` to test membership.

Example:

```python
role = "editor"

if role in {"admin", "editor"}:
    print("can edit")
```

This is clearer than:

```python
if role == "admin" or role == "editor":
    print("can edit")
```

Common membership checks:

```python
if item in items:
    ...

if key in settings:
    ...

if suffix in filename:
    ...
```

Membership uses the container's implementation.

For lists, membership may scan elements.

For sets and dictionaries, membership is usually much faster.

That performance detail is covered later with data structures.

---

# Conditional Assignment

A conditional can choose a value.

Example:

```python
if user_name:
    display_name = user_name
else:
    display_name = "Anonymous"
```

Python also has a conditional expression:

```python
display_name = user_name if user_name else "Anonymous"
```

Syntax:

```python
value_if_true if condition else value_if_false
```

Use conditional expressions for simple choices.

Avoid using them for complex branching.

Readable:

```python
label = "active" if is_active else "inactive"
```

Harder to read:

```python
result = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "D"
```

For multiple branches, use `if` / `elif` / `else`.

---

# The `pass` Statement

`pass` means "do nothing".

It is useful when Python requires a block but you have no code to run yet.

Example:

```python
if feature_enabled:
    pass
else:
    print("feature disabled")
```

`pass` is also used as a placeholder:

```python
if needs_review:
    pass  # TODO: add review workflow
```

Use it sparingly.

Leaving many `pass` blocks in real code usually means the behavior is unfinished.

---

# Pattern Matching With `match`

Python also supports structural pattern matching using `match`.

Example:

```python
command = "start"

match command:
    case "start":
        print("starting")
    case "stop":
        print("stopping")
    case _:
        print("unknown command")
```

Output:

```text
starting
```

`match` is useful when you are selecting behavior based on the structure or shape of data.

Simple `if` / `elif`:

```python
if command == "start":
    print("starting")
elif command == "stop":
    print("stopping")
else:
    print("unknown command")
```

This is fine for simple value checks.

Pattern matching becomes more useful with structured data:

```python
event = {"type": "click", "x": 10, "y": 20}

match event:
    case {"type": "click", "x": x, "y": y}:
        print(f"clicked at {x}, {y}")
    case {"type": "keypress", "key": key}:
        print(f"pressed {key}")
    case _:
        print("unknown event")
```

Do not use `match` just because it looks newer.

Use it when pattern structure makes the code clearer.

---

# Designing Decision Logic

Good conditional logic is not just correct.

It is readable.

Ask these questions:

* Are branches mutually exclusive?
* Should more than one branch be allowed to run?
* Are the most specific conditions checked first?
* Is the happy path buried inside too much nesting?
* Would guard conditions make the function clearer?
* Are names meaningful enough to make the condition readable?

Compare:

```python
if u.a and not u.b and p > 0:
    process()
```

With:

```python
user_can_order = user.is_active and not user.is_blocked
has_positive_balance = balance > 0

if user_can_order and has_positive_balance:
    process_order()
```

The second version gives the reader concepts, not just operations.

---

# Common Mistakes

## Misconception 1

### Indentation is just formatting.

In Python, indentation defines blocks.

Changing indentation changes program meaning or causes syntax errors.

## Misconception 2

### Conditions must be exactly `True` or `False`.

Conditions can use any object.

Python checks truthiness.

## Misconception 3

### `elif` branches are all checked.

Only branches before the first truthy branch are checked.

After one branch runs, the rest are skipped.

## Misconception 4

### `else` means an error happened.

`else` means the earlier condition or conditions did not match.

It is not automatically an error path.

## Misconception 5

### `is` and `==` mean the same thing.

`is` compares identity.

`==` compares value equality.

Use `is None`, but usually use `==` for value comparisons.

## Misconception 6

### A truthy container means all its contents are truthy.

A non-empty container is truthy even if it contains falsy values.

```python
if [False, None, 0]:
    print("non-empty")
```

This prints:

```text
non-empty
```

## Misconception 7

### A compact conditional is always better.

Short code can still be unclear.

Prefer direct, readable decision logic.

---

# Real-world Usage

## Input Validation

```python
if not email:
    error = "email is required"
elif "@" not in email:
    error = "email is invalid"
else:
    error = None
```

## Permissions

```python
if user.is_admin:
    can_delete = True
elif user.owns(resource):
    can_delete = True
else:
    can_delete = False
```

## Configuration

```python
if environment == "production":
    debug = False
elif environment == "development":
    debug = True
else:
    raise ValueError("unknown environment")
```

## Boundary Handling

```python
if index < 0:
    index = 0
elif index >= len(items):
    index = len(items) - 1
```

## Command Routing

```python
if command == "create":
    create_item()
elif command == "update":
    update_item()
elif command == "delete":
    delete_item()
else:
    show_help()
```

---

# Internal Mechanics

At a high level, conditional execution works like this:

```text
1. Evaluate condition expression.
2. Convert the result to truthiness.
3. If truthy, execute the associated block.
4. If falsy, skip that block.
5. For elif chains, repeat until a truthy branch is found.
6. If no branch matches and else exists, execute else.
```

CPython compiles conditionals into bytecode that performs jumps.

Conceptually:

```text
evaluate condition
if false, jump over block
run block
continue after conditional
```

You do not need bytecode details to write conditionals, but the model matters:

Conditionals are not magic.

They are controlled jumps based on truthiness.

---

# Concept Connections

Conditionals connect to earlier chapters:

* Numbers: numeric comparisons often drive branches.
* Strings: empty strings are falsy; string membership is common.
* Booleans: comparisons produce boolean values.
* Operators: comparison, boolean, identity, and membership operators are used in conditions.
* Expressions: every condition is an expression.
* Names and references: conditions inspect objects that names refer to.
* Mutability: conditions may depend on whether a container is empty or has changed.

Conditionals prepare you for:

* Loops.
* Functions.
* Error handling.
* Data validation.
* Object-oriented behavior.
* Testing.

---

# Active Recall

## Easy Recall Questions

1. What does an `if` statement do?
2. What does `else` mean?
3. What does `elif` mean?
4. What is truthiness?
5. Name five falsy values.
6. What does indentation define in Python?
7. When should you use `is None`?
8. What is a guard condition?

## Deep Understanding Questions

1. Why is `if items:` different from `if items == True:`?
2. Why does branch order matter in an `elif` chain?
3. When should separate `if` statements be used instead of `elif`?
4. How can guard conditions improve readability?
5. Why can deeply nested conditionals be hard to maintain?
6. What is short-circuit evaluation, and why is it useful?

## Predict-the-Output Questions

### Question 1

```python
items = [False]

if items:
    print("yes")
else:
    print("no")
```

### Question 2

```python
score = 95

if score >= 60:
    print("pass")
elif score >= 90:
    print("excellent")
```

### Question 3

```python
value = None

if value:
    print("truthy")
elif value is None:
    print("none")
else:
    print("falsy")
```

### Question 4

```python
name = ""
display = name if name else "Anonymous"
print(display)
```

### Question 5

```python
role = "editor"

if role in {"admin", "editor"}:
    print("can edit")

if role != "admin":
    print("not admin")
```

---

# Practical Exercises

## Exercise 1

Write a program that accepts an age variable and prints:

* `"child"` for ages below 13.
* `"teen"` for ages 13 through 17.
* `"adult"` for ages 18 and above.

Add a branch for invalid negative ages.

## Exercise 2

Write a password validation routine that checks:

* Password is not empty.
* Password has at least 8 characters.
* Password contains at least one digit.

Use clear branch names and avoid deeply nested code.

## Exercise 3

Rewrite a nested conditional using guard conditions.

Start with:

```python
if user:
    if user.is_active:
        if user.has_permission:
            print("allowed")
```

## Exercise 4

Create a grade classifier using `if`, `elif`, and `else`.

Make sure branch order is correct.

## Exercise 5

Write examples that demonstrate the difference between:

* `if value:`
* `if value == True:`
* `if value is True:`

Use at least three different values.

## Exercise 6

Use a conditional expression to choose between two labels.

Then rewrite it using a normal `if` / `else`.

Explain which version is clearer.

## Exercise 7

Create a command router using `match`.

Handle at least four commands and one default case.

---

# Summary

Conditionals let Python choose execution paths.

The core pieces are:

* `if` runs a block when a condition is truthy.
* `else` runs when earlier conditions fail.
* `elif` adds ordered alternatives.
* Branch order affects behavior.
* Indentation defines blocks.
* Truthiness is not the same as equality with `True`.
* Guard conditions help reduce nesting.
* `match` can express structured branching when the shape of data matters.

Conditionals are the first major step from straight-line code to adaptive programs.

---

# Preview of Chapter 19

Next we study loops.

Conditionals choose whether code runs.

Loops choose how many times code runs.

Chapter 19 expands the control-flow model from branching to repetition.

We will study:

* `while` loops for condition-driven repetition.
* `for` loops for processing iterables.
* `range()` for integer sequences.
* Forward and reverse looping.
* Nested loops and trace-table reasoning.
* `break` for stopping a loop early.
* `continue` for skipping one iteration.
* Loop `else` for search-style logic.
* `enumerate()` for index-value iteration.
* `zip()` for parallel iteration.
* Mutation hazards while iterating.

The connection is direct:

```text
conditionals choose a path once
loops repeat a path until iteration is complete
```

Loops are where conditionals become part of repeated workflows: search, filtering, validation, retries, and processing collections.
