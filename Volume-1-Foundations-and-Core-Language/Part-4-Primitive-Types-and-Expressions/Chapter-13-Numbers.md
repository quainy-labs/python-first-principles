# Chapter 13 — Numbers

---

# Learning Objectives

By the end of this chapter, you should understand:

* What numeric objects are in Python.
* Why numbers are objects, not primitive boxes.
* The main built-in numeric types: `int`, `float`, `complex`, and `bool`.
* Why `bool` is related to `int`.
* How integer objects behave.
* Why Python integers can grow beyond fixed machine-word size.
* How floating-point numbers approximate real numbers.
* Why floating-point arithmetic sometimes produces surprising results.
* How numeric operators create new numeric objects.
* Why numbers are immutable.
* How identity and equality apply to numeric objects.
* How numeric conversion works at a high level.
* How to choose the right numeric type for common tasks.
* Common mistakes with division, rounding, precision, and equality.

This chapter begins Part IV of Volume I.

The key shift is important:

> We are not learning numbers as isolated syntax. We are learning numbers as Python objects with type, value, identity, and behavior.

---

# Concept Overview

Numbers are among the first values most programmers use.

Example:

```python
age = 30
price = 19.99
temperature = -5
```

These look simple.

But in Python, they are still objects.

From Chapter 09:

```text
object
├── identity
├── type
└── value
```

So when Python evaluates:

```python
30
```

it works with an integer object.

Conceptually:

```text
int object
├── identity: specific runtime object
├── type: int
└── value: 30
```

When Python evaluates:

```python
19.99
```

it works with a floating-point object.

Conceptually:

```text
float object
├── identity: specific runtime object
├── type: float
└── value: approximate decimal value 19.99
```

Numbers support arithmetic because their types define numeric behavior.

---

# Why Numbers Matter

Numbers appear everywhere:

* Counting
* Prices
* Measurements
* Indexes
* IDs
* Time durations
* Percentages
* Statistics
* Coordinates
* Scores
* Machine learning tensors
* Financial calculations
* System metrics

But numeric code fails when the programmer does not understand the type and semantics of the values involved.

Examples:

```python
5 / 2
```

does not produce the same type as:

```python
5 // 2
```

This:

```python
0.1 + 0.2 == 0.3
```

may not behave as a beginner expects.

This:

```python
round(2.675, 2)
```

may surprise you.

This:

```python
True + True
```

works because `bool` is related to `int`.

Python's numeric model is convenient, but it is not magic.

This chapter gives you the mental model.

---

# The Main Numeric Types

Python's built-in numeric types include:

| Type | Example | Purpose |
| --- | --- | --- |
| `int` | `42` | Whole numbers |
| `float` | `3.14` | Approximate real numbers |
| `complex` | `2 + 3j` | Complex numbers |
| `bool` | `True` | Truth values, subclass of `int` |

Check with `type()`:

```python
print(type(42))
print(type(3.14))
print(type(2 + 3j))
print(type(True))
```

Output:

```text
<class 'int'>
<class 'float'>
<class 'complex'>
<class 'bool'>
```

Each value is an object.

Each object has a type.

The type determines behavior.

---

# Numbers Are Objects

Numbers have methods and attributes because they are objects.

Example:

```python
number = 10
print(number.bit_length())
```

Output:

```text
4
```

Why `4`?

The integer `10` is:

```text
1010
```

in binary, which requires four bits.

This method exists because `10` is an `int` object.

Another example:

```python
x = 3.5
print(x.is_integer())
```

Output:

```text
False
```

The float object has float-specific behavior.

The object model applies even to values that look small and simple.

---

# Numeric Literals

A numeric literal is a number written directly in source code.

Examples:

```python
10
-5
3.14
1_000_000
0b1010
0o12
0xA
2 + 3j
```

Different literal forms produce numeric objects.

| Literal | Meaning | Type |
| --- | --- | --- |
| `10` | Decimal integer | `int` |
| `-5` | Negative integer expression | `int` |
| `3.14` | Floating-point number | `float` |
| `1_000_000` | Integer with visual separators | `int` |
| `0b1010` | Binary integer literal | `int` |
| `0o12` | Octal integer literal | `int` |
| `0xA` | Hexadecimal integer literal | `int` |
| `2 + 3j` | Complex number expression | `complex` |

Underscores are allowed for readability:

```python
population = 1_428_000_000
```

Python ignores the underscores when computing the value.

---

# Unary Minus Is an Operation

This is subtle.

In Python:

```python
-5
```

is usually understood as a negative integer literal.

Conceptually, it is also useful to see unary minus as an operation applied to `5`.

```text
5      -> int object 5
-5     -> numeric negation result
```

This matters more when expressions get complex:

```python
x = 10
y = -x
```

Python evaluates `x`, applies unary negation, and binds `y` to the result.

---

# Integers

The `int` type represents whole numbers.

Examples:

```python
0
1
-1
42
10_000
```

Integers can be positive, negative, or zero.

Common operations:

```python
print(10 + 3)
print(10 - 3)
print(10 * 3)
print(10 // 3)
print(10 % 3)
print(10 ** 3)
```

Output:

```text
13
7
30
3
1
1000
```

Integer arithmetic with `+`, `-`, `*`, `//`, `%`, and `**` produces numeric objects.

---

# Python Integers Are Arbitrary Precision

Many languages have fixed-size integers.

For example, a 32-bit signed integer has a limited range.

Python integers are arbitrary precision.

That means they can grow as large as memory allows.

Example:

```python
big = 10 ** 100
print(big)
```

Python can represent this integer.

This is convenient.

It avoids many overflow bugs common in lower-level languages.

But it has a tradeoff:

```text
very large integers require more memory and more computation
```

Python integers are not raw fixed CPU integers.

They are Python objects that can manage large values.

---

# Integer Overflow

In many lower-level languages, integer overflow can occur.

For example:

```text
maximum integer + 1 -> wraps around or causes overflow
```

In Python:

```python
x = 999999999999999999999999999999
y = x + 1
print(y)
```

Python produces the mathematically larger integer.

It does not wrap around like a fixed-width machine integer.

This is one reason Python is pleasant for general programming and math exploration.

The cost is object overhead.

---

# Integers Are Immutable

Integers cannot change in place.

Example:

```python
x = 10
x = x + 1
```

This creates or obtains an integer object representing `11`, then rebinds `x`.

Mental model:

```text
before:
x ─────▶ int object 10

after:
x ─────▶ int object 11
```

The object `10` did not become `11`.

This is consistent with Chapter 11.

---

# Integer Identity Warning

You may see:

```python
a = 10
b = 10
print(a is b)
```

print:

```text
True
```

In CPython, small integers are commonly cached and reused.

Do not rely on this.

Numeric value comparison should use `==`:

```python
if count == 10:
    ...
```

not:

```python
if count is 10:
    ...
```

This follows directly from Chapter 12:

```text
is -> identity
== -> value equality
```

---

# Floating-Point Numbers

The `float` type represents approximate real numbers.

Examples:

```python
3.14
-0.5
1.0
2e3
1.5e-4
```

Scientific notation:

```python
2e3
```

means:

```text
2 * 10^3
```

So:

```python
print(2e3)
```

Output:

```text
2000.0
```

Floats are useful for measurements, scientific calculations, and approximate quantities.

They are not exact decimal arithmetic.

---

# Floats Are Approximate

Most decimal fractions cannot be represented exactly in binary floating-point.

Example:

```python
print(0.1 + 0.2)
```

Output often:

```text
0.30000000000000004
```

This is not a Python bug.

It is a property of binary floating-point representation.

Computers store floats in binary.

Some decimal fractions repeat forever in binary, just as `1/3` repeats forever in decimal:

```text
0.3333333333...
```

So Python stores the closest representable binary approximation.

---

# Floating-Point Equality

Because floats are approximate, direct equality can be risky.

Example:

```python
print(0.1 + 0.2 == 0.3)
```

Output often:

```text
False
```

Use `math.isclose()` for approximate comparison:

```python
import math

print(math.isclose(0.1 + 0.2, 0.3))
```

Output:

```text
True
```

Use direct equality only when exact float equality is truly what you mean.

For many measurements, use tolerance-based comparison.

---

# Floating-Point Rounding

Rounding can also surprise you.

Example:

```python
print(round(2.675, 2))
```

You might expect:

```text
2.68
```

But you may see:

```text
2.67
```

Why?

The stored floating-point value may be slightly less than the exact decimal value `2.675`.

Rounding operates on the stored binary approximation.

For financial decimal rounding, use `decimal.Decimal`, not `float`.

---

# Decimal for Exact Decimal Arithmetic

Python's standard library provides `decimal.Decimal`.

Example:

```python
from decimal import Decimal

price = Decimal("19.99")
tax = Decimal("0.08")
total = price + price * tax

print(total)
```

`Decimal` is useful when decimal precision matters, such as money.

Important:

Use strings when creating decimals:

```python
Decimal("0.1")
```

not:

```python
Decimal(0.1)
```

The second form starts from an already approximate float.

`Decimal` is not a built-in numeric literal type, but it is an important practical numeric tool.

---

# Fractions for Rational Arithmetic

Python also provides `fractions.Fraction`.

Example:

```python
from fractions import Fraction

x = Fraction(1, 3)
y = Fraction(1, 6)

print(x + y)
```

Output:

```text
1/2
```

Fractions represent rational numbers exactly.

They are useful for exact mathematical relationships.

They can be slower and more verbose than floats.

Choose them when exact rational arithmetic matters.

---

# Complex Numbers

Python has built-in support for complex numbers.

Example:

```python
z = 2 + 3j

print(type(z))
print(z.real)
print(z.imag)
```

Output:

```text
<class 'complex'>
2.0
3.0
```

Python uses `j` for the imaginary unit.

Complex numbers are common in engineering, signal processing, mathematics, and scientific computing.

Most everyday business applications do not need them.

But they are part of Python's numeric model.

---

# Booleans Are Numeric

`bool` is a subclass of `int`.

Example:

```python
print(isinstance(True, int))
print(True + True)
print(False + 10)
```

Output:

```text
True
2
10
```

This surprises many learners.

Conceptually:

```text
True behaves numerically like 1 in arithmetic contexts
False behaves numerically like 0 in arithmetic contexts
```

But do not overuse this.

Booleans should usually represent truth values, not ordinary numbers.

This relationship exists for historical and practical reasons.

We will study booleans in their own chapter.

---

# Numeric Operators

Common numeric operators:

| Operator | Meaning | Example | Result |
| --- | --- | --- | --- |
| `+` | Addition | `5 + 2` | `7` |
| `-` | Subtraction | `5 - 2` | `3` |
| `*` | Multiplication | `5 * 2` | `10` |
| `/` | True division | `5 / 2` | `2.5` |
| `//` | Floor division | `5 // 2` | `2` |
| `%` | Modulo | `5 % 2` | `1` |
| `**` | Power | `5 ** 2` | `25` |

Operators evaluate expressions.

Expressions produce objects.

Example:

```python
result = 5 + 2
```

Mental model:

```text
evaluate 5 + 2 -> int object 7
bind result -> int object 7
```

---

# True Division

The `/` operator performs true division.

Example:

```python
print(5 / 2)
print(4 / 2)
```

Output:

```text
2.5
2.0
```

Even when the mathematical result is a whole number, `/` produces a `float`.

Example:

```python
print(type(4 / 2))
```

Output:

```text
<class 'float'>
```

Use `/` when you want division that may produce fractional results.

---

# Floor Division

The `//` operator performs floor division.

Example:

```python
print(5 // 2)
```

Output:

```text
2
```

Floor division rounds down toward negative infinity.

This matters for negative numbers:

```python
print(-5 // 2)
```

Output:

```text
-3
```

Why not `-2`?

Because floor means:

```text
greatest integer less than or equal to the exact result
```

The exact result is:

```text
-2.5
```

The floor is:

```text
-3
```

---

# Modulo

The `%` operator gives the remainder associated with floor division.

Example:

```python
print(5 % 2)
```

Output:

```text
1
```

Python maintains this relationship:

```text
a == (a // b) * b + (a % b)
```

Example:

```python
a = -5
b = 2

print(a // b)
print(a % b)
print((a // b) * b + (a % b))
```

Output:

```text
-3
1
-5
```

This explains why modulo with negative numbers may differ from intuition imported from other languages.

---

# divmod()

Python provides `divmod()` to get quotient and remainder together.

Example:

```python
q, r = divmod(17, 5)

print(q)
print(r)
```

Output:

```text
3
2
```

This is equivalent to:

```python
q = 17 // 5
r = 17 % 5
```

Use `divmod()` when you need both.

---

# Exponentiation

The `**` operator performs exponentiation.

Example:

```python
print(2 ** 10)
```

Output:

```text
1024
```

Exponentiation has high precedence.

Example:

```python
print(2 * 3 ** 2)
```

Output:

```text
18
```

because:

```text
3 ** 2 -> 9
2 * 9  -> 18
```

Use parentheses when clarity matters:

```python
print((2 * 3) ** 2)
```

Output:

```text
36
```

---

# Operator Precedence

Python follows precedence rules.

Example:

```python
result = 2 + 3 * 4
print(result)
```

Output:

```text
14
```

Multiplication happens before addition.

Parentheses make intent explicit:

```python
result = (2 + 3) * 4
print(result)
```

Output:

```text
20
```

Do not rely on readers remembering every precedence rule.

Use parentheses when they make code clearer.

---

# Numeric Type Promotion

When numeric types mix, Python chooses a result type that can represent the operation.

Example:

```python
print(1 + 2)
print(type(1 + 2))

print(1 + 2.0)
print(type(1 + 2.0))
```

Output:

```text
3
<class 'int'>
3.0
<class 'float'>
```

`int + float` produces a `float`.

Example:

```python
print(1 + 2j)
print(type(1 + 2j))
```

Output:

```text
(1+2j)
<class 'complex'>
```

Mixed numeric operations follow type-specific rules.

---

# Converting Numeric Types

Python provides constructors for conversion:

```python
int()
float()
complex()
bool()
```

Examples:

```python
print(int("42"))
print(float("3.14"))
print(int(3.9))
print(float(10))
```

Output:

```text
42
3.14
3
10.0
```

Important:

```python
int(3.9)
```

does not round.

It truncates toward zero.

Example:

```python
print(int(3.9))
print(int(-3.9))
```

Output:

```text
3
-3
```

Use `round()` when you want rounding.

---

# round()

`round()` rounds numeric values.

Example:

```python
print(round(3.14159, 2))
```

Output:

```text
3.14
```

But rounding has rules and floating-point limitations.

Example:

```python
print(round(2.5))
print(round(3.5))
```

Output:

```text
2
4
```

Python uses banker's rounding for ties: it rounds to the nearest even number.

This reduces statistical bias over many operations.

Do not assume `round(x)` always rounds `.5` upward.

---

# abs()

`abs()` returns absolute value.

Example:

```python
print(abs(-10))
print(abs(10))
```

Output:

```text
10
10
```

For complex numbers, `abs()` returns magnitude:

```python
print(abs(3 + 4j))
```

Output:

```text
5.0
```

This follows the mathematical magnitude:

```text
sqrt(3^2 + 4^2)
```

---

# pow()

`pow()` computes powers.

Example:

```python
print(pow(2, 10))
```

Output:

```text
1024
```

It can also take a modulus:

```python
print(pow(2, 10, 1000))
```

Output:

```text
24
```

This computes:

```text
(2 ** 10) % 1000
```

efficiently.

The three-argument form is useful in number theory and cryptography.

---

# Numeric Objects and Immutability

Numeric objects are immutable.

Example:

```python
x = 5
y = x
x += 1

print(x)
print(y)
```

Output:

```text
6
5
```

`x += 1` rebinds `x` to a numeric object representing `6`.

It does not mutate the integer object `5`.

Diagram:

```text
before:
x ─┐
   ├────▶ int object 5
y ─┘

after:
x ─────▶ int object 6

y ─────▶ int object 5
```

This is why numeric aliases are less risky than aliases to mutable containers.

---

# Numeric Equality

Numeric equality compares numeric value.

Example:

```python
print(1 == 1.0)
```

Output:

```text
True
```

The objects have different types:

```python
print(type(1))
print(type(1.0))
```

Output:

```text
<class 'int'>
<class 'float'>
```

But they compare equal numerically.

Identity is different:

```python
print(1 is 1.0)
```

Output:

```text
False
```

Use `==` for numeric equality.

---

# Numeric Ordering

Numbers support ordering comparisons:

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
```

Output:

```text
True
True
True
```

Ordering comparisons produce boolean objects:

```python
result = 3 < 5
print(type(result))
```

Output:

```text
<class 'bool'>
```

This connects numbers to the upcoming booleans chapter.

---

# Chained Comparisons

Python supports chained comparisons:

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

but `x` is evaluated only once.

Chained comparisons are readable for ranges:

```python
if 0 <= percentage <= 100:
    print("valid")
```

This is idiomatic Python.

---

# NaN

Floating-point includes special values.

One is NaN: not a number.

Example:

```python
nan = float("nan")

print(nan)
print(nan == nan)
```

Output:

```text
nan
False
```

NaN is not equal to itself.

This is defined by floating-point rules.

Use `math.isnan()`:

```python
import math

print(math.isnan(nan))
```

Output:

```text
True
```

Do not check NaN using `==`.

---

# Infinity

Floating-point also supports infinity:

```python
positive = float("inf")
negative = float("-inf")

print(positive)
print(negative)
print(positive > 10 ** 100)
```

Output:

```text
inf
-inf
True
```

Infinity can appear in numeric algorithms, but it should be handled deliberately.

Use:

```python
import math

math.isinf(positive)
```

to detect it.

---

# Choosing Numeric Types

Use `int` when:

* Counting things
* Working with indexes
* Representing whole quantities
* Needing exact whole-number arithmetic

Use `float` when:

* Working with approximate measurements
* Scientific calculations
* Coordinates
* Ratios where approximation is acceptable

Use `Decimal` when:

* Decimal precision matters
* Money is involved
* You need controlled decimal rounding

Use `Fraction` when:

* Exact rational arithmetic matters
* You want fractions preserved symbolically

Use `complex` when:

* The domain requires complex numbers
* Engineering, signal processing, or mathematical modeling needs them

The type should match the problem domain.

---

# Common Mistakes

## Misconception 1

### Python numbers are raw CPU primitives.

Python numeric values are objects.

They have type, identity, value, and behavior.

CPython implements them with runtime object structures.

---

## Misconception 2

### Integer arithmetic overflows like fixed-width integers.

Python integers are arbitrary precision.

They grow as needed within memory limits.

---

## Misconception 3

### Floats store exact decimal values.

Floats are binary approximations.

Many decimal fractions cannot be represented exactly.

---

## Misconception 4

### `0.1 + 0.2 == 0.3` must be true.

Floating-point approximation can make this false.

Use `math.isclose()` for approximate comparisons.

---

## Misconception 5

### `int()` rounds floats.

`int()` truncates toward zero.

Use `round()` if rounding is intended.

---

## Misconception 6

### `/` always returns an integer when division is exact.

`/` returns a float.

Example:

```python
4 / 2
```

produces:

```text
2.0
```

---

## Misconception 7

### `//` truncates toward zero.

`//` performs floor division.

For negative numbers, floor division rounds down toward negative infinity.

---

## Misconception 8

### `is` is acceptable for numeric comparison.

Use `==` for numeric value comparison.

`is` checks object identity.

---

# Real-world Usage

## Counting and Indexing

Use integers for counts:

```python
attempts = 3
users_count = 128
```

Use integers for indexes:

```python
items[0]
items[1]
```

Indexes are whole-number positions.

---

## Money

Avoid float for money when exact decimal behavior matters.

Problem:

```python
total = 0.1 + 0.2
```

For money, prefer:

```python
from decimal import Decimal

total = Decimal("0.10") + Decimal("0.20")
```

Or store money as integer minor units:

```python
cents = 1999
```

The right choice depends on the application.

---

## Measurements

Floats are appropriate for many measurements:

```python
temperature = 36.6
latitude = 12.9716
ratio = 0.875
```

Measurements are often approximate already.

Use tolerance-based comparisons when needed.

---

## Percentages

Represent percentages carefully.

These are different:

```python
discount_percent = 15
discount_rate = 0.15
```

Name variables clearly.

Avoid mixing percent and rate silently.

---

## Large Integers

Python handles large integers well:

```python
factorial_like = 1
for number in range(1, 101):
    factorial_like *= number
```

The result can be huge.

Python will not overflow in the usual fixed-width sense.

But large integers still consume memory and CPU time.

---

## Data Science and Numeric Libraries

Python's built-in numbers are not the whole numeric ecosystem.

Libraries such as NumPy, pandas, PyTorch, and TensorFlow use specialized numeric types and arrays.

Those types have different performance and memory behavior.

This chapter gives the baseline Python model.

Later ecosystem chapters will explain library-specific numeric systems.

---

# Concept Connections

This chapter builds on the previous object model:

```text
Chapter 09:
Numbers are objects with type, identity, and value.

Chapter 10:
Names refer to numeric objects.

Chapter 11:
Numbers are immutable; arithmetic rebinds names to new numeric objects.

Chapter 12:
Use == for numeric equality, not is.
```

It prepares upcoming chapters:

```text
Strings:
    another immutable built-in type

Booleans:
    truth values produced by comparisons

Operators:
    numeric operators are one category of operator behavior

Expressions:
    numeric expressions evaluate to objects

Control flow:
    numeric comparisons often control branches and loops
```

The core model remains:

```text
numeric expression
    |
    v
numeric object
    |
    v
name may refer to it
    |
    v
operators create or compare numeric objects
```

---

# Internal Mechanics Summary

Important terms:

| Term | Meaning |
| --- | --- |
| `int` | Arbitrary-precision whole number object |
| `float` | Binary floating-point approximation |
| `complex` | Number with real and imaginary parts |
| `bool` | Truth value type related to `int` |
| `Decimal` | Exact decimal arithmetic type from standard library |
| `Fraction` | Exact rational arithmetic type from standard library |
| `/` | True division, returns float for built-in ints |
| `//` | Floor division |
| `%` | Remainder paired with floor division |
| `**` | Exponentiation |
| `math.isclose()` | Approximate float comparison |

Core rules:

```text
Numbers are objects.
Numeric objects are immutable.
Arithmetic creates numeric result objects.
Names can be rebound to those result objects.
Use == for numeric value comparison.
Do not use is for numeric equality.
Floats are approximate.
Use Decimal or Fraction when exactness is required.
```

---

# Active Recall

## Easy Recall Questions

1. What type represents whole numbers in Python?
2. What type represents approximate real numbers?
3. Are Python integers mutable or immutable?
4. What does `/` do?
5. What does `//` do?
6. What does `%` do?
7. What does `**` do?
8. Does `int(3.9)` round or truncate?
9. What should you use to compare floats approximately?
10. Should numeric equality use `is` or `==`?

---

## Deep Understanding Questions

1. Why are Python integers not the same as raw CPU integers?
2. Why do Python integers avoid fixed-width overflow?
3. Why can floats represent some decimal values only approximately?
4. Why can `0.1 + 0.2 == 0.3` be false?
5. Why does `/` return `2.0` for `4 / 2`?
6. Why does `-5 // 2` produce `-3`?
7. Why is `Decimal("0.1")` better than `Decimal(0.1)`?
8. Why does `True + True` produce `2`?
9. Why does `x += 1` rebind `x` for integers?
10. Why should the numeric type match the problem domain?

---

## Explain In Your Own Words

1. Explain what it means for a number to be an object.
2. Explain arbitrary-precision integers.
3. Explain binary floating-point approximation.
4. Explain true division versus floor division.
5. Explain modulo using the relationship between `//` and `%`.
6. Explain why money should usually not be represented with float.
7. Explain why numeric operations produce new objects.

---

## Predict-the-Output Questions

### Question 1

```python
print(5 / 2)
print(type(5 / 2))
```

Answer:

```text
2.5
<class 'float'>
```

Reason:

`/` performs true division.

---

### Question 2

```python
print(5 // 2)
print(-5 // 2)
```

Answer:

```text
2
-3
```

Reason:

`//` performs floor division, not truncation toward zero.

---

### Question 3

```python
print(5 % 2)
print(-5 % 2)
```

Answer:

```text
1
1
```

Reason:

Python's modulo pairs with floor division so `a == (a // b) * b + (a % b)`.

---

### Question 4

```python
x = 5
y = x
x += 1

print(x)
print(y)
```

Answer:

```text
6
5
```

Reason:

Integers are immutable. `x += 1` rebinds `x` to a new numeric result object.

---

### Question 5

```python
print(0.1 + 0.2 == 0.3)
```

Answer:

Often:

```text
False
```

Reason:

Binary floating-point approximations are involved.

---

### Question 6

```python
print(True + True)
print(False + 10)
```

Answer:

```text
2
10
```

Reason:

`bool` is related to `int`; `True` behaves like `1` and `False` like `0` in arithmetic contexts.

---

# Mental Model Questions

1. Draw `x = 10`.
2. Draw `x = 10; y = x; x = x + 1`.
3. Draw why numeric rebinding does not mutate the old integer object.
4. Draw an expression `2 + 3` evaluating to an integer object.
5. Draw the difference between `1` and `1.0` as objects with different types but equal numeric value.
6. Draw why `is` is the wrong question for numeric equality.

---

# Practical Exercises

## Exercise 1

Inspect numeric types:

```python
values = [10, 3.14, 2 + 3j, True]

for value in values:
    print(value, type(value), id(value))
```

Explain each object's type and value.

---

## Exercise 2

Compare division operators:

```python
for value in [5, -5]:
    print(value / 2)
    print(value // 2)
    print(value % 2)
```

Explain the results using true division, floor division, and modulo.

---

## Exercise 3

Investigate float approximation:

```python
import math

result = 0.1 + 0.2

print(result)
print(result == 0.3)
print(math.isclose(result, 0.3))
```

Explain why the equality and `isclose` results differ.

---

## Exercise 4

Use `Decimal`:

```python
from decimal import Decimal

a = Decimal("0.1")
b = Decimal("0.2")

print(a + b)
print(a + b == Decimal("0.3"))
```

Explain why strings are used to create the decimals.

---

## Exercise 5

Compare `int()` and `round()`:

```python
print(int(3.9))
print(int(-3.9))
print(round(3.9))
print(round(-3.9))
```

Explain truncation versus rounding.

---

## Exercise 6

Use `divmod()`:

```python
minutes = 137
hours, remaining_minutes = divmod(minutes, 60)

print(hours)
print(remaining_minutes)
```

Explain why `divmod()` is useful here.

---

## Exercise 7

Choose the right numeric type:

For each case, decide whether you would use `int`, `float`, `Decimal`, `Fraction`, or `complex`:

* Number of login attempts.
* Product price.
* Temperature reading.
* Exact ratio `1/3`.
* Electrical impedance.
* Number of records in a database table.

Explain each choice.

---

# Summary

In this chapter we learned:

* Numbers are Python objects.
* Numeric objects have identity, type, and value.
* `int` represents arbitrary-precision whole numbers.
* Python integers avoid fixed-width overflow within memory limits.
* `float` represents binary floating-point approximations.
* Floating-point arithmetic can produce surprising decimal results.
* Use `math.isclose()` for approximate float comparison.
* Use `Decimal` for exact decimal arithmetic when needed.
* Use `Fraction` for exact rational arithmetic when needed.
* `complex` represents complex numbers.
* `bool` is related to `int`, but booleans should usually represent truth values.
* `/` performs true division.
* `//` performs floor division.
* `%` gives the related remainder.
* `int()` truncates; it does not round.
* Numeric objects are immutable.
* Numeric operations produce result objects and may rebind names.
* Use `==`, not `is`, for numeric equality.

The core model is:

```text
numeric expression
    |
    v
numeric object
    |
    ├── identity
    ├── type
    └── value
```

Numbers are the first core language construct we can now understand through the object model.

---

# Preview of Chapter 14

Next we study strings.

Strings are also objects.

They are immutable, like numbers.

But they represent text, not numeric quantity.

We will study:

* String literals
* Escape sequences
* Indexing and slicing
* String methods
* Immutability
* Formatting
* Unicode
* Text vs bytes

The same object model still applies:

```text
string expression -> str object -> operations create or compare objects
```
